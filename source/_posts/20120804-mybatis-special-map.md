---
layout: post
title: "Mybatis返回特殊Map"
date: 2012-08-04 23:00:00
tags: [Java]
urlname: mybatis-special-map
---

想返回个特殊实体，一个Map，key是一个表的一个字段的值，value是另一个表的所有记录。参考了下网上的“攻略”，不过和攻略里面讲的有点不同，那里面key的值value的实体实在同一个表内，如果不同表，会报缺少set方法的异常。解决方法很简单，就是在实体里面加个字段对应的属性。但是并不愿意为了一个查询语句就要去改变实体。  
   无聊看了下session的select，提供了通过 `ResultHandler` 来处理返回的结果集。所以尝试着解决这个问题。

mapper.xml里面的配置如下：  
```xml
    <resultMap type="HashMap" id="testMap">  
        <result column="UA_INFO" property="key" />  
        <association property="value" 
            resultMap="com.xxx.xxx.BaseResultMap">
        </association>  
      </resultMap>  
      
      
    <select id="getUaMapByTimestamp" 
        parameterType="Map" resultMap="testMap">  
      SQL语句  
    </select>  
```

`com.xxx.xxx.BaseResultMap`是另一个实体的mapper的resultMap。这个查询，返回的每一条记录都是 `{key=..., value=...}`

这个结果集并不符合要求。不过通过ResultHandler来处理每一条记录就可以达到要求了。

看下Mybatis源码里面有关继承 `ResultHandler` 的 `DefaultMapResultHandler`类。

```java
public class DefaultMapResultHandler<K, V>
    implements ResultHandler {  
  
  private final Map<K, V> mappedResults;  
  private final String mapKey;  
  
  @SuppressWarnings("unchecked")  
  public DefaultMapResultHandler(String mapKey,
    ObjectFactory objectFactory) {  
    this.mappedResults = objectFactory.create(Map.class);  
    this.mapKey = mapKey;  
  }  
  
  public void handleResult(ResultContext context) {  
    // TODO is that assignment always true?  
    final V value = (V) context.getResultObject();  
    final MetaObject mo = MetaObject.forObject(value);  
    // TODO is that assignment always true?  
    final K key = (K) mo.getValue(mapKey);  
    mappedResults.put(key, value);  
  }  
  
  public Map<K, V> getMappedResults() {  
    return mappedResults;  
  }  
}
```

这个 `DefaultMapResultHandler` 实现 `handleResult` 接口，处理每条数据。

可以模仿这个类写个类来装配自己所需要的Map。

```java
private class MapResultHandler implements ResultHandler {  
      
        @SuppressWarnings("rawtypes")  
        private final Map mappedResults = new HashMap();  
              
        @SuppressWarnings({ "unchecked", "rawtypes" })  
        @Override  
        public void handleResult(ResultContext context) {  
            Map map = (Map) context.getResultObject();  
            mappedResults.put(map.get("key"), map.get("value"));  
        }  
              
        public Map getMappedResults() {  
            return mappedResults;  
        }  
            
}
```

使用：

```java
this.getSqlSession().select(
    getWholeSqlId("getUaMapByTimestamp"),handler);  
```

这样就可以得到需要的结果集了。
