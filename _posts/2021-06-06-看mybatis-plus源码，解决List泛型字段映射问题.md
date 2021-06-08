---
layout: post
title: 看mybatis-plus源码<br>解决List&lt;泛型&gt;字段映射问题
summary: 
featured-img: 273138.jpg
labels: [mybatis, mybatis-plus]
---

### 前言
&emsp; &emsp; 开发的过程中，个人不太喜欢创建过多的数据库表，所以有一些简单的外键关联表的场景时， 我都会习惯于
将外键id放在类的属性中List&lt;Long&gt;或者是关联表的字段都放在类的属性List<自定义类>，
数据库表创建一个mediumtext字段存放着，然后用mybatis-plus的注解
@TableField(typeHandler = JacksonTypeHandler.class)修饰，通过json的格式交互，例如：

```hgignore
    @TableField(typeHandler = JacksonTypeHandler.class)
    private List<Long> ids;
    
    @TableField(typeHandler = JacksonTypeHandler.class)
    private List<业务BO> bos;
    
    -------------------------------------------------------------------
    
    @Data
    public class 业务BO {

        private String xxx;
        
        private String xxx;

    }
```

### 遇到的问题
```hgignore
1.List<Long>查询出来却是List<Integer>
2.List<业务BO>查询出来却是List<LinkedHashMap>

如果不对该字段进行任何操作直接返回给前端的话是没什么问题的，
因为接口与前端交互本身就是json的格式，是不存在泛型的概念，
但如果要对查询数据再进行封装处理时，则会抛异常ClassCastException

java.lang.ClassCastException: class java.lang.Integer cannot be cast to class java.lang.Long
java.lang.ClassCastException: class java.util.LinkedHashMap cannot be cast to class com.业务BO

```

### 分析
```hgignore
项目启动时mybatis-plus会通过反射，将实体类的属性，注解等信息通过TableInfoHelper类的initTableFields方法处理并
存放在TableInfo类中的List<TableFieldInfo>，但没有属性的泛型，也就是说List<xxx>会变成List的方式存在，

/**
 * Jackson 实现 JSON 字段类型处理器
 *
 * @author hubin
 * @since 2019-08-25
 */
@Slf4j
@MappedTypes({Object.class})
@MappedJdbcTypes(JdbcType.VARCHAR)
public class JacksonTypeHandler extends AbstractJsonTypeHandler<Object> {
    private static ObjectMapper objectMapper = new ObjectMapper();
    private Class<?> type;

    public JacksonTypeHandler(Class<?> type) {
        if (log.isTraceEnabled()) {
            log.trace("JacksonTypeHandler(" + type + ")");
        }
        Assert.notNull(type, "Type argument cannot be null");
        this.type = type;
    }

    @Override
    protected Object parse(String json) {
        try {
            return objectMapper.readValue(json, type);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }

    @Override
    protected String toJson(Object obj) {
        try {
            return objectMapper.writeValueAsString(obj);
        } catch (JsonProcessingException e) {
            throw new RuntimeException(e);
        }
    }

    public static void setObjectMapper(ObjectMapper objectMapper) {
        Assert.notNull(objectMapper, "ObjectMapper should not be null");
        JacksonTypeHandler.objectMapper = objectMapper;
    }
}

通过查看JacksonTypeHandler源码不难看出，
JacksonTypeHandler（mybatis-plus） 继承 AbstractJsonTypeHandler（mybatis-plus） 继承 BaseTypeHandler（mybatis） 
重写toJson，将数据存入库中
重写parse，将数据映射到类中

this.type = type启动的时候就将List<xxx>变成List的形式赋值给了Class<?> type;
当使用parse转换时就会出现
1.List<Long>查询出来却是List<Integer>
2.List<业务BO>查询出来却是List<LinkedHashMap>

如果字段不是List<xxx> 而是一个类的话，JacksonTypeHandler映射是没问题的
```

### 解决
```hgignore
通过分析得出，造成问题的主要的原因有两个
1.mybatis-plus在反射的时候没有将泛型信息存放起来而是直接使用了属性List
2.JacksonTypeHandler的使用场景是不带泛型的属性

解决方案如下：
1.再转换一次
  拿到List<Integer>再转回List<Long>
  拿到List<LinkedHashMap>再转回List<业务BO>
2.自定义处理器
3.改源码重写打包
```
**1.再转换一次**
```hgignore
public class JsonUtils {
    public static ObjectMapper objectMapper = new ObjectMapper();
    
    @SneakyThrows
    public static <T> T parse(Object fromValue, TypeReference<T> typeReference) {
        return objectMapper.convertValue(fromValue, typeReference);
    }
}

1.List<Long>查询出来却是List<Integer>
List<Long> ids = JsonUtils.parse(实体类.getIds(), new TypeReference<List<Long>>() {});

2.List<业务BO>查询出来却是List<LinkedHashMap>
List<业务BO> bos = JsonUtils.parse(实体类.getBos(), new TypeReference<List<业务BO>>() {});
```
**2.自定义处理器**
```hgignore
第一种方式显然很笨，转来转去的也麻烦，所以有了第二种，只需自定义一次处理器，从根上解决

1.List<Long>查询出来却是List<Integer>

自定义专门处理List<Long>类型的处理器，继承mybatis-plus的AbstractJsonTypeHandler，泛型List<Long>
public class HtLongTypeHandler extends AbstractJsonTypeHandler<List<Long>> {

    @Override
    protected List<Long> parse(String json) {
        return JsonUtils.toList(json, Long.class);
    }

    @Override
    protected String toJson(List<Long> obj) {
        return JsonUtils.toJson(obj);
    }
}

typeHandler引用自定义的处理器
@TableField(typeHandler = HtLongTypeHandler.class)
private List<Long> ids;

----------------------------------------------------------------------------------------------------

2.List<业务BO>查询出来却是List<LinkedHashMap>
这个就不能使用mybatis-plus的AbstractJsonTypeHandler了，这还需要继承上一级BaseTypeHandler，TypeReference<T>则是泛型类型

public class HtTypeHandler<T> extends BaseTypeHandler<T> {

    private final TypeReference<T> typeReference;

    public HtTypeHandler(TypeReference<T> typeReference) {
        if (typeReference == null) {
            throw new HuaTengSystemException("Type argument cannot be null");
        }
        this.typeReference = typeReference;
    }

    @Override
    public void setNonNullParameter(PreparedStatement ps, int i, Object parameter, JdbcType jdbcType) throws SQLException {
        ps.setString(i, JsonUtils.format(parameter));
    }

    @Override
    public T getNullableResult(ResultSet rs, String columnName) throws SQLException {
        return JsonUtils.parse(rs.getString(columnName), typeReference);
    }


    @Override
    public T getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
        return JsonUtils.parse(rs.getString(columnIndex), typeReference);

    }

    @Override
    public T getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
        return JsonUtils.parse(cs.getString(columnIndex), typeReference);
    }

}

再根据List<业务BO> 创建一个专门的TypeHandler，继承自定义好的HtTypeHandler，泛型List<业务BO>，构造方法中初始化父类的TypeReference
public class 业务BOTypeHandler extends HtTypeHandler<List<业务BO>> {

    public 业务BOTypeHandler() {
        super(new TypeReference<List<业务BO>>() {
        });
    }
}

typeHandler引用自定义的处理器
@TableField(typeHandler = 业务BOTypeHandler.class)
private List<业务BO> bos;
```

**3.改源码重写打包**
```hgignore
第二种方式List<Long>还好，但是List<业务BO>每次都需要创建一个指定的类去指定TypeReference，显然也不是最优方案
如果可以写成通用的，自动识别泛型就最好了，但我理解的是BaseTypeHandler是处理数据返回的层面，反射拿泛型在项目启动的时候就执行了，所以用类型处理器是没办法写成通用的

所以最优的方案还是从修改源码，设置一个javaType类型，mybatis-plus的javaType是布尔类型，只是一个校验功能，并不能像mybatis那样自定义类作为List里的泛型类


```

