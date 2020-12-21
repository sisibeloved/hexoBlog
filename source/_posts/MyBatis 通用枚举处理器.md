---
title: MyBatis 通用枚举处理器
date: 2020-12-21 18:06:00
categories: Backend
tags:
- MyBatis
---

##  背景

mybatis在3.4.5及之后版本中，新增了一个指定全局默认枚举类型处理器的配置项；

+ 需要将mybatis-spring-boot-starter更新到2.1.4
+ 对应mybatis-spring版本2.0.6
+ 对应mybatis版本3.5.6
+ 相应地需要更新druid到1.1.21以上

<!-- more -->

##  配置

```java
    public SqlSessionFactory sqlSessionFactory(DataSource dataSource, PageHelper pageHelper, MybatisInterceptor mybatisInterceptor) throws Exception {

		 // ...

        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDefaultEnumTypeHandler(CodeIdentifyEnumHandler.class);

        // ...

        bean.setTypeHandlersPackage("com.xxx.model.enums");
        return bean.getObject();
    }
```

##  源码解析

```java
  private Map<JdbcType, TypeHandler<?>> getJdbcHandlerMap(Type type) {
    Map<JdbcType, TypeHandler<?>> jdbcHandlerMap = typeHandlerMap.get(type);
    if (NULL_TYPE_HANDLER_MAP.equals(jdbcHandlerMap)) {
      return null;
    }
    if (jdbcHandlerMap == null && type instanceof Class) {
      Class<?> clazz = (Class<?>) type;
      if (Enum.class.isAssignableFrom(clazz)) {
        Class<?> enumClass = clazz.isAnonymousClass() ? clazz.getSuperclass() : clazz;
        jdbcHandlerMap = getJdbcHandlerMapForEnumInterfaces(enumClass, enumClass);
        if (jdbcHandlerMap == null) {
          register(enumClass, getInstance(enumClass, defaultEnumTypeHandler));
          return typeHandlerMap.get(enumClass);
        }
      } else {
        jdbcHandlerMap = getJdbcHandlerMapForSuperclass(clazz);
      }
    }
    typeHandlerMap.put(type, jdbcHandlerMap == null ? NULL_TYPE_HANDLER_MAP : jdbcHandlerMap);
    return jdbcHandlerMap;
  }
```

判断是Enum类型，且没有在全局map中拿到handler，会调用`defaultEnumTypeHandler`，在初始化时会将 enumClass 传入，因此生成的就是子类的 typeHandler。

```java
    public CodeIdentifyEnumHandler(Class<T> type) {
        if (type == null) {
            throw new IllegalArgumentException("Type argument cannot be null");
        }
        this.type = type;
    }
```

##  问题

###  1. 初始化报错

`CodeIdentifyEnumHandler` 类需要去掉abstract修饰，且添加无参构造函数，否则拿不到合适的构造器，初始化会报错。

###  2. JSR支持

MyBatis 3.4.1 之后内置了对 jsr310 的支持，导致调用`LocalDateTimeTypeHandler`等处理器报错，需要升级druid包版本到1.1.21以上。

因为内置的处理器会调用 ResultSet 的`getObject()` 方法：
```java
  @Override
  public LocalDateTime getNullableResult(ResultSet rs, String columnName) throws SQLException {
    return rs.getObject(columnName, LocalDateTime.class);
  }
```

而因为使用druid，实际使用中使用的实现类是 `DruidPooledResultSet`，在1.1.21版本以前`getObject()`方法并未实现，会直接抛出`SQLFeatureNotSupportedException`异常。
