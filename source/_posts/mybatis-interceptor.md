---
title: MyBatis Interceptor 工作原理及示例
date: 2024.02.01
tags: 
   - Mybatis
   - Java
categories: Technology  
mathjax: true 
comments: true
toc: true
description: 本文介绍如何基于拦截处理敏感数据并支持在配置文件中可配置实体类中字段进行处理。
---

# 引言
在软件开发中，保护敏感数据是至关重要的。
特别是在处理敏感信息，如个人身份信息、财务数据等时，开发者必须确保数据在存储和传输过程中的安全性。
`MyBatis` 作为一个流行的持久层框架，提供了拦截器 `Interceptor` 机制，允许开发者在 `SQL` 执行过程中插入自定义逻辑，从而实现对敏感数据的加密和解密处理的同时支持通过实体字段可配置。

# Interceptor 工作原理
[MyBatis 官网中 Interceptor 的介绍：](https://mybatis.org/mybatis-3/zh_CN/configuration.html#%E6%8F%92%E4%BB%B6%EF%BC%88plugins%EF%BC%89)

> MyBatis 允许你在映射语句执行过程中的某一点进行拦截调用。默认情况下，MyBatis 允许使用插件来拦截的方法调用包括：
> * Executor (update, query, flushStatements, commit, rollback, getTransaction, close, isClosed)
> * ParameterHandler (getParameterObject, setParameters)
> * ResultSetHandler (handleResultSets, handleOutputParameters)
> * StatementHandler (prepare, parameterize, batch, update, query)
>
>通过 MyBatis 提供的强大机制，使用插件是非常简单的，只需实现 Interceptor 接口，并指定想要拦截的方法签名即可。

可见这些方法是在不同的执行阶段被调用的—[MyBatis从入门到精通](https://book.douban.com/subject/27074809/)：
1. **Executor**：
    - update: 当执行 INSERT、UPDATE、DELETE 操作时调用。
    - query: 在 SELECT 查询方法执行时调用。
    - flushStatements: 在通过 SqlSession 方法调用 flushStatements 方法或执行的接口方法中带有 @Flush 注解时才被调用。
    - commit: 在通过 SqlSession 方法调用 commit 方法时被调用。
    - rollback: 在通过 SqlSession 方法调用 rollback 方法时被调用。
    - getTransaction: 在通过 SqlSession 方法获取数据库连接时被调用。
    - close: 在延迟加载获取新的 Executor 后才会被执行。
    - isClosed: 在延迟加载执行查询方法前被执行。

2. **ParameterHandler**：
    - getParameterObject: 在执行存储过程处理出参的时候被调用。
    - setParameters: 在所有数据库方法设置 SQL 参数时被调用。

3. **ResultSetHandler**：
    - handleResultSets: 处理查询结果集。
    - handleOutputParameters: 使用存储过程处理出参时被调用。

4. **StatementHandler**：
    - prepare: 在数据库执行前被调用，优先于当前接口中其他方法而被执行。
    - parameterize: 在 prepare 方法后执行，用于处理参数信息。
    - batch: 在全局设置配置 defaultExecutorType="BATCH" 时执行数据操作才会调用。
    - update: 用于执行更新类型的 SQL 语句。
    - query: 用于获取查询返回的结果集。

通过实现 `Interceptor` 接口并指定想要拦截的方法签名，可以轻松地使用这一机制。
`Interceptor` 接口包含以下方法：（Mybatis-3.5.1 版本）

```java
public interface Interceptor {
    Object intercept(Invocation invocation) throws Throwable;

    Object plugin(Object var1);

    void setProperties(Properties var1);
}
```

* `intercept(Invocation invocation)`: 这个方法用于拦截目标方法并执行自定义逻辑。获取目标方法的参数、方法等信息，并进行处理。
* `plugin(Object target)`: 这个方法用于生成一个代理对象。需要判断目标对象是否需要被拦截，如果需要则返回一个代理对象，否则返回 null。
* `setProperties(Properties properties)`: 这个方法用于设置属性。获取配置文件中的属性，并进行相应的设置。

在 `MyBatis` 的配置文件中注册 `Interceptor` 后，框架会在执行相应的操作时自动调用自定义逻辑。

## Simple Example

1. 创建并实现 `org.apache.ibatis.plugin.Interceptor` 接口的类：
```
// ExamplePlugin.java
@Intercepts({@Signature(
  type= Executor.class,
  method = "update",
  args = {MappedStatement.class,Object.class})})
public class ExamplePlugin implements Interceptor {
  private Properties properties = new Properties();

  @Override
  public Object intercept(Invocation invocation) throws Throwable {
    // implement pre processing if need
    Object returnObject = invocation.proceed();
    // implement post processing if need
    return returnObject;
  }

  @Override
  public void setProperties(Properties properties) {
    this.properties = properties;
  }
}
```

**Tips**：在 [Mybatis-3.5.2 版本后 Interceptor](https://github.com/mybatis/mybatis-3/blob/mybatis-3.5.2/src/main/java/org/apache/ibatis/plugin/Interceptor.java) 接口中定义的方法已给出了默认实现，只需按需求实现 `intercept` 方法，这是 `Java 8` 默认方法特性的一种应用，旨在简化接口的实现。

```java
public interface Interceptor {

  Object intercept(Invocation invocation) throws Throwable;

  default Object plugin(Object target) {
    return Plugin.wrap(target, this);
  }

  default void setProperties(Properties properties) {
    // NOP
  }

}
```

2. 在 `MyBatis` 的配置文件中注册 `Interceptor`：
```
<!-- mybatis-config.xml -->
<plugins>
  <plugin interceptor="org.mybatis.example.ExamplePlugin">
    <property name="someProperty" value="100"/>
  </plugin>
</plugins>
```

3. 插件将会拦截在 `Executor` 实例中所有的 `update` 方法调用，这里的 `Executor` 是负责执行底层映射语句的内部对象。

# 敏感数据处理的场景

## 场景描述
在处理敏感数据时，我们希望在数据库中以密文形式存储，而在系统中以明文形式展示。同时，也希望能够对数据进行脱敏处理，如对部分内容进行遮挡。为了减少对原有代码的修改，我们希望通过配置文件来配置敏感字段。

## 设计思路
为了实现上述需求，我们可以使用 `MyBatis` 的拦截器机制，自定义一个拦截器来处理敏感数据。选择拦截 `Executor` 的 `update` 方法和 `ResultSetHandler` 的 `handleResultSets` 方法作为拦截点。

## 原因
- `Executor` 的 update 方法用于执行 `insert、update 和 delete` 操作，可以在这里对写入数据库的数据进行加密处理。
- `ResultSetHandler` 的 `handleResultSets` 方法用于处理查询结果集，可以在这里对从数据库中读取的数据进行解密处理。

这两个方法可以满足我们对敏感数据处理的需求，包括加密写入、解密读取，加密写入、不解密读取，以及不加密写入、加密读取的场景。通过自定义拦截器，我们可以在不修改原有代码的情况下，实现对敏感数据的处理。同时，通过配置文件来配置敏感字段，增加了系统的灵活性和可维护性。

代码如下：完整实例可见[仓库](https://github.com/wyiyi/bronze)

## 实现
### 1.自定义敏感数据处理器

```java
@Slf4j
@Intercepts({
        @Signature(type = Executor.class, method = "update", args = {MappedStatement.class, Object.class}),
        @Signature(type = ResultSetHandler.class, method = "handleResultSets", args = {Statement.class})
})
@Component
public class DataSensitiveInterceptor implements Interceptor {

    private final Map<String, String> configs;

    public DataSensitiveInterceptor(DataSensitiveConfig config) {
        this.configs = config.getSensitive();
    }

    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        if (invocation.getTarget() instanceof Executor) {
            Object object = invocation.getArgs()[1];
            if (object instanceof Map) {
                Map<String, Object> map = (Map<String, Object>) object;
                for (Map.Entry<String, Object> entry : map.entrySet()) {
                    if (!entry.getKey().startsWith("param")) {
                        handleEncrypt(entry.getValue());
                    }
                    continue;
                }
            } else {
                handleEncrypt(object);
            }
            return invocation.proceed();
        } else if (invocation.getTarget() instanceof ResultSetHandler) {
            ResultSetHandler resultSetHandler = (ResultSetHandler) invocation.getTarget();
            Statement statement = (Statement) invocation.getArgs()[0];
            List<Object> resultList = resultSetHandler.handleResultSets(statement);
            resultList.forEach(this::handleDecrypt);
            return resultList;
        }
        return invocation.proceed();
    }

    private void handleEncrypt(Object object) {
        handleObject(object, true);
    }

    private void handleDecrypt(Object object) {
        handleObject(object, false);
    }

    private void handleObject(Object object, boolean encrypt) {
        for (Map.Entry<String, String> config : configs.entrySet()) {
            int lastPoint = config.getKey().lastIndexOf('.');
            String className = config.getKey().substring(0, lastPoint);
            if (object.getClass().getName().equals(className)) {
                String property = config.getKey().substring(lastPoint + 1);
                String handlerName = config.getValue();
                DataSensitiveHandler handler = SpringContextHolder.getBean("dataSensitiveHandler-" + handlerName);
                BeanWrapper wrapper = new BeanWrapperImpl(object);
                wrapper.setPropertyValue(property,
                        encrypt ? handler.encrypt(String.valueOf(wrapper.getPropertyValue(property))) :
                                handler.decrypt(String.valueOf(wrapper.getPropertyValue(property)))
                );
            }
        }
    }
}
```

```java
@Component
@ConfigurationProperties("com.amber.common")
public class DataSensitiveConfig {

    private Map<String, String> sensitive = new HashMap<>();

    public Map<String, String> getSensitive() {
        return sensitive;
    }
}
```

### 2.实现敏感数据加密和解密接口

实现 `DataSensitiveHandler` 接口，定义具体的加密和解密方法。

```java
public interface DataSensitiveHandler {

    String encrypt(String str);

    String decrypt(String str);

}
```

### 3.内置敏感数据处理器 abb、sm4、md5
目前内置三种敏感数据处理器：

- `abb`：对字符串中间部分使用 `*` 遮挡，仅对读取到的数据执行操作
- `md5`：对字符串进行 md5 摘要，仅在写入数据时执行操作
- `sm4`：使用国密 SM4 算法进行对称加解密，在写入及读取时均执行

```java
/**
 * 自定义处理器注册 bean，”dataSensitiveHandler-“ 为固定前缀
 * 自定义后缀：abb 对字符串中间部分使用 `*` 遮挡，仅对读取到的数据执行操作
 */
@Component("dataSensitiveHandler-abb")
public class DataSensitiveAbbHandler implements DataSensitiveHandler {

    @Override
    public String encrypt(String str) {
        return str;
    }

    @Override
    public String decrypt(String str) {
       int length = str.length();
       if (length < 2){
          return str;
       } else if (length % 2 == 0) {
          int midIndex = length / 2 - 1;
          return str.substring(0, midIndex) + "****" + str.substring(midIndex + 2);
       }else {
          int midIndex = length / 2;
          return str.substring(0, midIndex - 2) + "****" + str.substring(midIndex + 2);
       }
    }
}
```

```java
/**
 * 自定义处理器注册 bean，”dataSensitiveHandler-“ 为固定前缀
 * 自定义后缀：对字符串进行 md5 摘要，仅在写入数据时执行操作
 */
@Component("dataSensitiveHandler-md5")
public class DataSensitiveMd5Handler implements DataSensitiveHandler {

    @Override
    public String encrypt(String str) {
        return DigestUtils.md5Hex(str);
    }

    @Override
    public String decrypt(String str) {
        return str;
    }
}
```

```java
/**
 * 自定义处理器注册 bean，”dataSensitiveHandler-“ 为固定前缀
 * 自定义后缀：sm4 使用国密 SM4 算法进行对称加解密，在写入及读取时均执行
 * 条件注册 Bean
 */
@ConditionalOnClass(name = "org.bouncycastle.crypto.Digest")
@Component("dataSensitiveHandler-sm4")
public class DataSensitiveSm4Handler implements DataSensitiveHandler {

    private static final SymmetricCrypto SM4 = SmUtil.sm4();

    @Override
    public String encrypt(String str) {
        return SM4.encryptHex(str);
    }

    @Override
    public String decrypt(String str) {
        return SM4.decryptStr(str, CharsetUtil.CHARSET_UTF_8);
    }
}
```

### 4.配置

配置参数以 `com.amber.common.sensitive` 为前缀，后面配置格式为 `key : value`，其中：

- `key` 为 MyBatis Mapper 实体类全名及要处理敏感数据的属性，如 `UserDO` 的 `phone` 属性设置为：`com.amber.common.sensitive.entity.UserDO.phone`。目前仅支持字符串类型的属性。
- `value` 为敏感数据处理类 bean name 的后缀，加上 `dataSensitiveHandler-` 前缀组成完整 bean name。

支持 `yml` 和 `properties` 文件格式：

```yml
com:
  amber:
    common:
      sensitive:
        com.amber.common.sensitive.entity.UserDO.phone: abb
        com.amber.common.sensitive.entity.UserDO.idCard: sm4
```

```properties
com.amber.common.sensitive.com.amber.common.sensitive.entity.UserDO.password=md5
```

### 5.单元测试
- 加密和解密：确保敏感数据在写入数据库时被正确加密，并在从数据库读取时被正确解密。
- 配置解析：验证配置文件中的敏感字段是否正确地被解析并应用到拦截器中。
- 不同处理器的应用：测试不同的敏感数据处理器（如 abb、md5、sm4）是否按照预期工作。
- 非敏感字段的不处理：确保非敏感字段在拦截器中不被错误地处理。

```groovy
@Sql
class DataSensitiveTest extends BaseApplicationTests {

    @Autowired
    UserDAO userDAO

    @Autowired
    RoleServiceImpl roleService

   @Autowired
   JdbcTemplate jdbcTemplate

   @Test
   void test() {
      assert jdbcTemplate.queryForObject('select count(*) from userinfo', Integer) == 0

      // Create
      UserDO user = new UserDO()
      user.setName('user name')
      user.setPhone('12345678901')
      user.setIdCard('234098uzxcv')
      user.setPassword('123456')

      assert userDAO.insert(user) == 1
      assert user.getId() > ''
      assert user.getPhone() == '12345678901'
      assert user.getPassword() == 'e10adc3949ba59abbe56e057f20f883e'
      assert user.getIdCard() != '234098uzxcv'
      // str.length <15  sm4 length 32
      // str.length >15  sm4 length 64
      // str.length >32 sm4 length 96
      // sm4 length 128 160 192 ...
      def sm4ValueLen = 32
      assert user.getIdCard().length() == sm4ValueLen

      assert jdbcTemplate.queryForObject('select count(*) from userinfo', Integer) == 1
      assert jdbcTemplate.queryForObject('select phone from userinfo', String) == '12345678901'
      assert jdbcTemplate.queryForObject('select password from userinfo', String) == 'e10adc3949ba59abbe56e057f20f883e'
      assert jdbcTemplate.queryForObject('select id_card from userinfo', String) != '234098uzxcv'
      assert jdbcTemplate.queryForObject('select id_card from userinfo', String).length() == sm4ValueLen

      // Retrieve

      UserDO retrievedUser = userDAO.selectById(user.getId())
      assert retrievedUser.getPhone() == '123****8901'
      assert retrievedUser.getIdCard() == '234098uzxcv'

      // Update
      retrievedUser.setPhone('01234567890')
      retrievedUser.setIdCard('210103')
      userDAO.updateById(retrievedUser)
      assert retrievedUser.getPhone() == '01234567890'
      assert retrievedUser.getIdCard() != '210103'
      assert retrievedUser.getIdCard().length() == sm4ValueLen

      assert jdbcTemplate.queryForObject('select phone from userinfo', String) == '01234567890'
      assert jdbcTemplate.queryForObject('select id_card from userinfo', String) != '210103'

      retrievedUser = userDAO.selectById(user.getId())
      assert retrievedUser.getPhone() == '012****7890'
      assert retrievedUser.getIdCard() == '210103'
   }
}
```

### 注意事项

- 通过 MyBatis 新增或保存实体时，传入的实体在方法调用后，配置为敏感数据的属性会变成应用了敏感处理器 `encrypt` 方法之后的值
- 通过 MyBatis 查询实体时，检索出的实体对象中，配置了敏感数据的属性会变成应用了敏感处理器 `decrypt` 方法之后的值
- 不通过 MyBatis 操作的数据，不会应用敏感数据处理器处理数据
- **存入数据库中的数据在执行了敏感处理后将丧失按照处理前的数据进行查询的能力，只能按照处理后的数据进行查询**

# 结论
通过 MyBatis 的 Interceptor 机制，开发者可以有效地处理敏感数据，提高应用程序的安全性。合理地配置和使用 Interceptor，可以在不改变原有业务逻辑的情况下，增强数据保护措施。
