---
layout: post
title: mybatis-plus逻辑删除与唯一索引的坑
summary: 记一次mybatis-plus逻辑删除与唯一索引的填坑记录
featured-img: 205659.jpg
labels: [记录]
---

### 配置
```hgignore
  global-config:
    #数据库相关配置
    db-config:
      #主键类型  AUTO:"数据库ID自增", INPUT:"用户输入ID", ID_WORKER:"全局唯一ID (数字类型唯一ID)", UUID:"全局唯一ID UUID";
      id-type: AUTO
      logic-delete-field: status  # 全局逻辑删除的实体字段名(版本 3.3.0,配置后可以忽略实体类字段上加上@TableLogic注解)
      logic-delete-value: 1 # 逻辑已删除值(默认为 1)
      logic-not-delete-value: 0 # 逻辑未删除值(默认为 0)
    banner: false
```

### 问题描述
在yml中配置了全局逻辑删除的字段status，未删除为0，删除为1  
logic-delete-value: 1 # 逻辑已删除值(默认为 1)  
logic-not-delete-value: 0 # 逻辑未删除值(默认为 0)  

<br>

### 问题1：
表中name字段设置为唯一索引时，删除后再插入一条相同的数据时会报错：  
Caused by: java.sql.SQLIntegrityConstraintViolationException: Duplicate entry '' for key 'product.product_name_uindex'

**解决：**  
因为id是唯一的，所以将status设置删除后等于id，再将status和name设置成联合唯一索引，这样就解决了问题1  
logic-delete-value: id # 逻辑已删除值(默认为 1)  
logic-not-delete-value: 0 # 逻辑未删除值(默认为 0)   
这里需要注意的是，数据库设置status字段默认值为0，或者插入时设置为0  

<br>

### 问题2：
由于mybatis-plus不支持联合主键，当表需要用到联合主键时，我将两个联合的字段加起来MD5生成唯一的串作为用户自定义的varchar类型的id  
```hgignore
@TableId(type = IdType.INPUT)  
private String id;  
```
    
这时就出现了第二个问题，status字段相应的也是varchar类型，当逻辑删除后，mybatis-plus并没有将id的值存入status，而是把id当成了字符串存入了status，
这样问题1的问题又出现了。
<img class="imgclass" src="/assets/img/posts/mybatis-plus/img.png"/>

这就非常奇怪了，最后通过查看mybatis-plus源码得到答案，原来作者是在反射时做了个判断（TableInfo类409行）  
logicDeleteFieldInfo.isCharSequence() ? "'%s'" : "%s"  
如果字段是字符类型，则用"'%s'"，否则"%s"，我不太清楚作者为什么要对字符类型单独处理，可能是防sql注入？  
<img class="imgclass" src="/assets/img/posts/mybatis-plus/img_1.png"/>

**定位到问题后就好解决了**  
将status设置成datetime类型，全局逻辑删除修改为如下，删除则用数据库的当前时间函数，未删除则不设置时间  
logic-delete-value: now() # 逻辑已删除值(默认为 1)  
logic-not-delete-value: null # 逻辑未删除值(默认为 0)  

如果想继续用第一个方案（logic-delete-value: id），则可以对字段单独配置逻辑删除
```hgignore
/**
* 是否删除  null-正常  时间-删除(删除后显示时间)
*/
@TableLogic(delval = "now()",value = "null")
private Date status;
```
  
个人还是建议全局配置成第二个方案，一劳永逸

<script src="{{ '/assets/js/jquery.min.js' | relative_url }}"></script>
<script src="{{ '/assets/js/viewer.min.js' | relative_url }}"></script>
<link rel="stylesheet" href="{{ '/assets/css/viewer.min.css' | prepend: site.baseurl }}">
<script>
$('.imgclass').viewer();
</script>