---
title: MyBatis 拦截器介绍及自定义示例
date: 2024.02.01
tags: 
   - Mybatis
   - Java
categories: Technology  
mathjax: true 
comments: true
toc: true
description: 本文探讨了 MyBatis 拦截器在处理敏感数据中的重要性。通过深入分析，我们提供了一种有效的方法来确保数据安全，同时保持 MyBatis 框架的灵活性和性能。文章详细介绍了拦截器的机制、设计思路、实现方法以及其在特定场景中的应用。
---

# 引言
`MyBatis` 作为一个流行的持久层框架，提供了拦截器 `Interceptor` 机制，允许开发者在 `SQL` 执行过程中插入自定义逻辑。本文将深入探讨如何利用 `MyBatis` 的拦截器 `Interceptor` 来处理敏感数据，确保数据在传输和存储过程中的安全性。

# Interceptor 介绍
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

# 处理敏感数据的场景

## 场景描述
在处理敏感数据时，我们希望数据在数据库中加密存储，而在系统中以明文展示。同时，为了保护隐私，我们计划对数据进行脱敏处理，例如隐藏部分敏感信息。为了降低对现有代码的修改，我们将通过配置文件来管理哪些字段需要加密或脱敏。

## 设计思路
为了实现上述需求，我们可以使用 `MyBatis` 的拦截器机制，自定义一个拦截器来处理敏感数据。选择拦截 `Executor` 的 `update` 方法和 `ResultSetHandler` 的 `handleResultSets` 方法作为拦截点。

## 原因
- `Executor` 的 update 方法用于执行 `insert、update 和 delete` 操作，可以在这里对写入数据库的数据进行加密处理。
- `ResultSetHandler` 的 `handleResultSets` 方法用于处理查询结果集，可以在这里对从数据库中读取的数据进行解密处理。

这两个方法可以满足我们对敏感数据处理的需求，包括加密写入、解密读取，加密写入、不解密读取，以及不加密写入、加密读取的场景。通过自定义拦截器，我们可以在不修改原有代码的情况下，实现对敏感数据的处理。同时，通过配置文件来配置敏感字段，增加了系统的灵活性和可维护性。

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

   /**
    * 获取配置文件中需要加密的属性
    *
    * @param config DataSensitiveConfig
    */
   public DataSensitiveInterceptor(DataSensitiveConfig config) {
      this.configs = config.getSensitive();
   }

   /**
    * 通过拦截器处理
    *
    * @param invocation invocation
    * @return Object
    * @throws Throwable 异常
    */
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
                    wrapper.getPropertyValue(property) == null ? null :
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

   /**
    * 在写入数据时对数据做的处理
    * 默认为不进行任何操作
    *
    * @param str 将要写入的数据
    * @return 实际写入的数据
    */
   default String encrypt(String str) {
      return str;
   }

   /**
    * 在读取数据时对数据做的处理
    * 默认为不进行任何操作
    *
    * @param str 实际读到的数据
    * @return 返回给调用者的数据
    */
   default String decrypt(String str) {
      return str;
   }

}
```

### 3.内置敏感数据处理器 abb、sm4hex、md5
目前内置三种敏感数据处理器：

- `abb`：对字符串中间部分使用 `*` 遮挡，仅对读取到的数据执行操作
- `md5`：对字符串进行 md5 摘要，仅在写入数据时执行操作
- `sm4hex`：使用国密 SM4 算法进行对称加解密，以 16 进制表示加密结果，在写入及读取时均执行

```java
/**
 * 敏感数据处理器：使用 * 遮挡明文中间部分，保留前后内容。
 * <p>
 * 仅在读取明文数据并返回时，进行遮挡处理；
 * 存入明文数据时，不对存入内容进行变更。
 * <p>
 * 遮挡方式为：<p>
 * 1. 明文长度为 1 时，不遮挡<p>
 * 2. 明文长度为 2 时，遮挡第二位<p>
 * 3. 明文长度大于 2 时，将明文分成三部分，遮挡中间部分；不能整除时，尽可能使遮挡部分较多<p>
 * 遮挡后内容长度与明文保持一致。
 * 如：<p>
 * 明文                   | 遮挡后<p>
 * 'a'                   | 'a'<p>
 * 'ab'                  | 'a*'<p>
 * 'abc'                 | 'a*c'<p>
 * 'abcd'                | 'a**d'<p>
 * '13012345678'         | '130*****678'<p>
 * '123456789012345678'  | '123456******345678'<p>
 * '这是一段测试文字'       | '这是****文字'<p>
 * 注册 bean 使用 ”dataSensitiveHandler-“ 固定前缀，abb 为后缀，意为 abbreviation
 */
@Component("dataSensitiveHandler-abb")
public class DataSensitiveAbbHandler implements DataSensitiveHandler {

   private static final char MASK = '*';

   /**
    * 实现解密方法，对字符串中间部分使用 `*` 遮挡
    *
    * @param str str
    * @return String
    */
   @Override
   public String decrypt(String str) {
      if (StringUtils.isBlank(str)) {
         return str;
      }
      int len = str.length();
      switch (len) {
         case 1:
            return str;
         case 2:
            return str.substring(0, 1) + MASK;
         default:
            int oriLen = len / 3;
            int maskLen = len - oriLen * 2;
            return str.substring(0, oriLen) +
                    StringUtils.repeat(MASK, maskLen) +
                    str.substring(oriLen + maskLen);
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
@ConditionalOnClass(name = {"org.bouncycastle.crypto.Digest", "org.bouncycastle.asn1.gm.GMNamedCurves"})
@Component("dataSensitiveHandler-sm4hex")
public class DataSensitiveSm4HexHandler implements DataSensitiveHandler {

   private static final SymmetricCrypto SM4 = SmUtil.sm4();

   /**
    * 使用国密 SM4 算法进行加密
    *
    * @param str 明文
    * @return 密文 16 进制表示
    */
   @Override
   public String encrypt(String str) {
      return SM4.encryptHex(str);
   }

   /**
    * 使用国密 SM4 算法进行解密
    *
    * @param str 密文
    * @return 明文
    */
   @Override
   public String decrypt(String str) {
      return SM4.decryptStr(str, CharsetUtil.CHARSET_UTF_8);
   }
}
```

### 4.配置

配置参数以 `com.amber.common.sensitive` 为前缀，后面配置格式为 `key : value`，其中：

- `key` 为 MyBatis Mapper 实体类全名及要处理敏感数据的属性，如 `UserDO` 的 `phone` 属性设置为：`com.amber.common.sensitive.mock.entity.UserDO.phone`。目前仅支持字符串类型的属性。
- `value` 为敏感数据处理类 bean name 的后缀，加上 `dataSensitiveHandler-` 前缀组成完整 bean name。

支持 `yml` 和 `properties` 文件格式：

```yml
com:
  amber:
    common:
      sensitive:
        com.amber.common.sensitive.mock.entity.UserDO.phone: abb
        com.amber.common.sensitive.mock.entity.UserDO.idCard: sm4hex
```

```properties
com.amber.common.sensitive.com.amber.common.sensitive.mock.entity.UserDO.password=md5
```

### 5.单元测试
- 加密和解密：确保敏感数据在写入数据库时被正确加密，并在从数据库读取时被正确解密。
- 配置解析：验证配置文件中的敏感字段是否正确地被解析并应用到拦截器中。
- 不同处理器的应用：测试不同的敏感数据处理器（如 abb、md5、sm4）是否按照预期工作。
- 非敏感字段的不处理：确保非敏感字段在拦截器中不被错误地处理。

```groovy
@Sql
@Transactional
class DataSensitiveTest extends BaseApplicationTests {

   @Autowired
   UserDAO userDAO
   
   @Autowired
   RoleDAO roleDAO
   
   @Autowired
   UserService userService
   
   @Autowired
   RoleService roleService
   
   @Autowired
   UserRoleService userRoleService
   
   @Autowired
   UserHistoryService userHistoryService

   @Autowired
   JdbcTemplate jdbcTemplate

   def static final MD5_LEN = 32
   def name = 'user name'
   def phone = '12345678901'
   def idCard = '234098uzxcv'
   def pwd = '123456'

   @Test
   void cruTest() {
      def user = testCreate()
      def retrievedUser = testRetrieve(user.getId())
      testUpdate(retrievedUser)
   }

   def testCreate() {
      assert jdbcTemplate.queryForObject('select count(*) from userinfo', Integer) == 0

      UserDO user = new UserDO()
      user.setName(name)
      user.setPhone(phone)
      user.setIdCard(idCard)
      user.setPassword(pwd)

      assert userDAO.insert(user) == 1
      assert user.getId() > ''
      // abb handler
      assert user.getPhone() == phone
      // md5 handler
      assert user.getPassword().length() == MD5_LEN
      // sm4hex handler
      assert user.getIdCard() != idCard && user.getIdCard().length() == getSm4HexLen(idCard)

      assert jdbcTemplate.queryForObject('select count(*) from userinfo', Integer) == 1
      assert jdbcTemplate.queryForObject('select phone from userinfo', String) == phone
      assert jdbcTemplate.queryForObject('select password from userinfo', String).length() == MD5_LEN
      assert jdbcTemplate.queryForObject('select id_card from userinfo', String) != '234098uzxcv'
      assert jdbcTemplate.queryForObject('select id_card from userinfo', String).length() == getSm4HexLen(idCard)

      return user
   }

   static int getSm4HexLen(String str) {
      // SM4算法的块大小为16字节
      int blockSize = 16
      str > '' ?
              ((int) (str.getBytes(StandardCharsets.UTF_8).length / blockSize) + 1) * blockSize * 2 :
              0
   }

   def testRetrieve(String userId) {
      UserDO retrievedUser = userDAO.selectById(userId)
      assert retrievedUser.getPhone() != phone
      assert retrievedUser.getPhone() == '123*****901'
      assert retrievedUser.getIdCard() == idCard
      return retrievedUser
   }

   def testUpdate(UserDO userToUpdate) {
      def newPhone = '01234567890'
      def newIdCard = '12345678901234567'

      userToUpdate.setPhone(newPhone)
      userToUpdate.setIdCard(newIdCard)
      userDAO.updateById(userToUpdate)

      assert userToUpdate.getPhone() == newPhone
      assert userToUpdate.getIdCard() != newIdCard
      assert userToUpdate.getIdCard().length() == getSm4HexLen(newIdCard)

      assert jdbcTemplate.queryForObject('select phone from userinfo', String) == newPhone
      assert jdbcTemplate.queryForObject('select id_card from userinfo', String) != newIdCard

      def retrievedUser = userDAO.selectById(userToUpdate.getId())
      assert retrievedUser.getPhone() == '012*****890'
      assert retrievedUser.getIdCard() == newIdCard
   }

   @Test
   void batchTest() {
      testBatchInsert()
      List<UserDO> retrievedUsers = testBatchRetrieve()
      testBatchUpdate(retrievedUsers)
   }

   def testBatchInsert() {
      assert jdbcTemplate.queryForObject('select count(*) from userinfo', Integer) == 0

      List<UserDO> users = new ArrayList<>()
      3.times {
         UserDO user = new UserDO()
         user.setName("${name}${it+1}")
         user.setPhone("${phone}${it+1}")
         user.setPassword("${pwd}${it+1}")
         user.setIdCard("${idCard}${it+1}")
         users.add(user)
      }
      userService.saveBatch(users)

      assert jdbcTemplate.queryForObject('select count(*) from userinfo', Integer) == 3
      // sm4hex handler
      assert users.get(0).getIdCard() != idCard
      assert users.get(0).getIdCard().length() == getSm4HexLen(idCard)
      // abb handler
      assert jdbcTemplate.queryForList('select phone from userinfo', String) == ["${phone}1", "${phone}2", "${phone}3"]
      assert users.get(0).getPhone() == "${phone}1"
      // sm4hex handler
      def queriedIdCard = jdbcTemplate.queryForObject("select id_card from userinfo where user_name='${name}2'", String)
      assert queriedIdCard != "${idCard}2"
      assert queriedIdCard.length() == getSm4HexLen("${idCard}2")
   }

   def testBatchRetrieve() {
      List<UserDO> retrievedUsers = userService.list()
      assert retrievedUsers.size() == 3
      assert retrievedUsers[0].getPhone() == '1234****9011'
      assert retrievedUsers[1].getPhone() == '1234****9012'
      assert retrievedUsers[2].getPhone() == '1234****9013'
      assert retrievedUsers[0].getIdCard() == "${idCard}1"
      assert retrievedUsers[1].getIdCard() == "${idCard}2"
      assert retrievedUsers[2].getIdCard() == "${idCard}3"
      return retrievedUsers
   }

   def testBatchUpdate(List<UserDO> retrievedUsers) {
      List<UserDO> usersToUpdate = new ArrayList<UserDO>()
      for (UserDO user : retrievedUsers) {
         user.setPhone(phone.reverse())
         user.setIdCard(idCard.reverse())
         usersToUpdate.add(user)
      }
      userService.updateBatchById(usersToUpdate)

      assert jdbcTemplate.queryForObject("select count(*) from userinfo where phone = '${phone.reverse()}'", Integer) == 3
      assert jdbcTemplate.queryForObject("select id_card from userinfo where user_name='${name}3'", String).length() == getSm4HexLen("${idCard.reverse()}")

      QueryWrapper<UserDO> wrapper = new QueryWrapper()
      wrapper.eq('phone', phone.reverse())
      retrievedUsers = userService.list(wrapper)
      assert retrievedUsers.size() == 3
      assert retrievedUsers[0].getPhone() == '109*****321'
      assert retrievedUsers[1].getIdCard() == idCard.reverse()
   }
   
   // ...
}
```

完整实例可见[仓库](https://github.com/wyiyi/bronze)

### 注意事项

- 通过 MyBatis 新增或保存实体时，传入的实体在方法调用后，配置为敏感数据的属性会变成应用了敏感处理器 `encrypt` 方法之后的值
- 通过 MyBatis 查询实体时，检索出的实体对象中，配置了敏感数据的属性会变成应用了敏感处理器 `decrypt` 方法之后的值
- 不通过 MyBatis 操作的数据，不会应用敏感数据处理器处理数据
- **存入数据库中的数据在执行了敏感处理后将丧失按照处理前的数据进行查询的能力，只能按照处理后的数据进行查询**

# 结论
通过 `MyBatis` 的 `Interceptor` 机制，开发者能够增强对敏感数据的保护，提升应用的安全性。
`Interceptor` 允许在不改动原有业务逻辑的情况下，实施额外数据保护措施。
在处理个人身份信息、财务数据等敏感信息时，`Interceptor` 尤为关键，确保数据存储和传输的安全。
