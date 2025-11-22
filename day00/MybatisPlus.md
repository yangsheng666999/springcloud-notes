# 知识1：

先了解mybatisplus是干嘛的。

（mybatis是流行的持久层框架，主要做数据库的CRUD）

引入MyBatis-Plus不会对现有的工程产生影响，原来基于MyBatis的代码依然可以照常运行。

![](D:\ProgramData\Typora Data\springcloud-notes\img\MyBatis-Plus.png)

后面分为4部分：1.快速入门，2.核心功能，3.扩展功能，4.插件功能，

来讲解MyBatis-Plus

1. 快速入门

   1. 入门案例

      1. 学会MP的基本用法

         1. 引入MybatisPlus的起步依赖

            ![](D:\ProgramData\Typora Data\springcloud-notes\day00\img\引入mp依赖.png)
            原来的mybatis依赖删掉

         2. 定义mapper
            ![](D:\ProgramData\Typora Data\springcloud-notes\day00\img\定义mapper.png)

      2. 体会MP的无侵入和方便快捷的特点

   2. 常见注解

      ![](D:\ProgramData\Typora Data\springcloud-notes\day00\img\常用注解1.png)

      接着讲了几个常用的注解

      ![](D:\ProgramData\Typora Data\springcloud-notes\day00\img\常用注解2.png)

      - 几个问题
        - MybatisPlus是如何获取实现CRUD的数据库表信息的？
          - 默认以类名驼峰转下划线作为表名
          - 默认把名为id的字段作为主键
          - 默认把变量名驼峰转下划线作为表的字段名
        - MybatisPlus的常用注解有哪些？
          - @TableName:指定表名称及全局配置
          - @TableId:指定id字段及相关配置
          - @TableFeld:指定普通字段及相关配置
        - idType的常见类型有哪些？
          - AUTO、ASSIGN_ID、INPUT
        - 使用@TableField的常见场景是？
          - 成员变量名与数据库字段名不一致
          - 成员变量名以is开头，且是布尔值
          - 成员变量名与数据库关键字冲突
          - 成员变量名不是数据库字段

   3. 常见配置
      ![](D:\ProgramData\Typora Data\springcloud-notes\day00\img\mp配置.png)

      1. 查询配置的方法

         1. 查询官网

            具体可参考官方文档：使用配置|MyBatis-Plus(baomidou.com)
            ![](D:\ProgramData\Typora Data\springcloud-notes\day00\img\官方mp配置.png)

         2. 走IDEA的默认

      2. MyBatisPlus使用的基本流程是什么？

         1. 引入起步依赖
         2. 自定义Mapper基础BaseMapper
         3. 在实体类上添加注解声明 表信息
         4. 在application.yml中根据需要添加配置

2. 核心功能

   1. 条件构造器

      1. 什么是条件构造器
         在真实项目中，CRUD的条件往往比较复杂。所以mp构造了一个条件构造器，来帮我们构建复杂的条件。
         ![](D:\ProgramData\Typora Data\springcloud-notes\day00\img\BaseMapper1.png)
         ![](D:\ProgramData\Typora Data\springcloud-notes\day00\img\Wrapper.png)
         AbstractWrapper
         ![](D:\ProgramData\Typora Data\springcloud-notes\day00\img\AbstractWrapper.png)
         QueryWrapper
         拓展了select的部分，where由父类AbstractWrapper实现了。
         ![](D:\ProgramData\Typora Data\springcloud-notes\day00\img\QueryWrapper.png)

         UpdateWrapper
         ![](D:\ProgramData\Typora Data\springcloud-notes\day00\img\UpdateWrapper.png)

      2. 案例演示

         1. 案例1：基于 QueryWrapper 的查询
            需求：

            1. 查询出名字中带o的，存款大于等于1000元的人的id、username、info、balance字段
               ![](D:\ProgramData\Typora Data\springcloud-notes\day00\img\user表.png)

               ```sql
               select id,username,info,balance
               from user
               where username like ? and balance >= ?
               ```

               

            2. 更新用户名为 jack 的用户的余额为 2000
               ```sql
               update user
               	set balance = 2000
               	where (username = "jack")
               ```

         2. 案例2：基于id为1，2，4的用户的余额，扣200
            需求：

            1. 更新id为1，2，4的用户的余额，扣200
               ```sql
               update user
               	set balance = balance - 200
               	where id in (1,2,4)
               ```

      3. 总结：

         - 条件构造器的用法：
           - QueryWrapper和LambdaQueryWrapper通常用来构建select 、delete、 update的where条件部分
           - UpdateWrapper和LambdaUpdateWrapper通常只有在set语句比较特殊才使用
           - 尽量使用LambdaQueryWrapper和LambdaUpdateWrapper，避免硬编码

   2. 自定义SQL
      *我们可以利用MyBatisPlus的Wrapper来构建复杂的Where条件，然后自己定义SQL语句中剩下的部分。*

      - 案例

        - 需求：将id在指定范围的用户（例如1、2、4）的余额扣减指定值。

          ```java
          <update id="updateBalanceByIds">
              UPDATE user
              SET balance = balance - #{amount}
          	WHERE id IN
                  <foreach collection="ids" separator="," item="id" open="(" close=")">
                  #{id}
          		</foreach>
          </update>            
          ```

          ```java
          @Test
          void testUpdateWrapper() {
              List<Long> ids = List.of(1L,2L,4L);
              UpdateWrapper<User> wrapper = new UpdateWrapper<User>()
                  .setSql("balance = balance - 200")
                  .in("id", ids);
              userMapper.update(null, wrapper);
          }
          ```

          ![](D:\ProgramData\Typora Data\springcloud-notes\day00\img\自定义sql.png)
          ![](D:\ProgramData\Typora Data\springcloud-notes\day00\img\自定义sql2.png)

      - 总结

        我们可以利用MyBatisPlus的Wrapper来构建复杂的Where条件，然后自己定义SQL语句中剩下的部分。

        1.  基于Wrapper构建where条件
           ```java
           List<Long> ids = list.of(1L,2L,4L);
           int amount = 200;
           // 1.构建条件
           LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<User>().in(User::getId,ids);
           // 2.自定义SQL方法调用
           userMapper.updateBalanceByIds(wrapper,amount);
           ```

        2. 在mapper方法参数中用Param注解声明wrapper变量名称，必须是ew

           ```java
           void updateBalanceIds(@Param("ew")LambdaQueryWrapper<User> wrapper,@Param("amount") int amount);
           ```

        3. 自定义SQL,并使用Wrapper条件
           ```xml
           <update id="updateBalanceByIds">
           	UPDATE tb_user SET balance = balance - #{amount} ${ew.customSQLSegment}
           </update>
           ```

           

        

   3. Service接口
      ![](D:\ProgramData\Typora Data\springcloud-notes\day00\img\Service提供的方法.png)
      ![](D:\ProgramData\Typora Data\springcloud-notes\day00\img\IService2.png)

      - 总结：MP的Service接口使用流程是怎样的？

        - 自定义Service接口继承IService接口
          ```java
          public interface IUserService extends IService<User> {}
          ```

        - 自定义Service实现类，实现自定义接口并继承ServiceImpl类
          ```java
          public class UserServiceImpl
              extends ServiceImpl<UserMapper,User>
              implements IUserService {}
          ```

      - 案例

        - 基于Restful风格实现下列接口
          - 需求：基于Restful风格实现下面的接口：
            ![](D:\ProgramData\Typora Data\springcloud-notes\day00\img\IService3.png)
            

3. 扩展功能

4. 插件功能

# 知识2： 常用注解

## @TableName

###  Q1: 注解的 6 个属性到底怎么用？尤其是 keepGlobalPrefix

|       属性       | 类型     | 是否必须 | 默认值 | 实际作用 + 推荐写法（重点！）                                |
| :--------------: | -------- | :------- | ------ | :----------------------------------------------------------- |
|      value       | String   | 否       | ""     | 指定数据库表名。如果你的实体类名和表名不一致，必须写这个！ 例如：表名是 sys_user，类名是 User，必须加 @TableName("sys_user") |
|      schema      | String   | 否       | ""     | 指定数据库的 schema（就是数据库名），MySQL一般不用，PostgreSQL/SQLServer 常用。 例如：@TableName(value = "sys_user", schema = "public") |
| keepGlobalPrefix | boolean  | 否       | false  | 是否保留全局配置的 tablePrefix 前缀！ 这个超级重要！！！ 场景举例： 你在生成器里配了全局前缀 tb_，数据库表全是 tb_user、tb_order → 生成的类默认会变成 User、Order（自动去掉 tb_） → 如果你某个表不想去掉前缀（比如表就是叫 tb_log，你希望类名也叫 TbLog），就把这个设成 true： @TableName(value = "tb_log", keepGlobalPrefix = true) |
|    resultMap     | String   | 否       | ""     | 指定自定义 resultMap 的 id（一般不用） 只有你 xml 里手写了复杂 resultMap 才填 |
|  autoResultMap   | boolean  | 否       | false  | 是否自动为这个表生成 resultMap（用于处理字段和属性类型不一致的情况） 一般不用开，开了会多生成一个 resultMap |
| excludeProperty  | String[] | 否       | {}     | 生成实体时排除某些字段（比如你不想生成 create_time、update_time） 用法： @TableName(excludeProperty = {"createTime", "updateTime"}) |

### Q2: 全局怎么配 table-prefix？（yml 和 Java 两种方式）

```yaml
# application.yml  或 application-prod.yml 等
mybatis-plus:
  global-config:
    db-config:
      table-prefix: t_          # ← 这里配你表的前缀！！！
      # table-prefix: tb_       # 很多公司用 tb_
      # table-prefix: sys_      # 也有人用 sys_

      # 下面这几行顺便一起配上，后面用得着
      id-type: auto             # 主键自增
      logic-delete-field: deleted   # 逻辑删除字段名（全局）
      logic-delete-value: 1
      logic-not-delete-value: 0
```

配完这句后，只要你的表名是` t_user、t_role、t_menu、t_order`，代码生成器会自动把前缀 t_ 去掉，生成类名就是 `User、Role、Menu、Order`，一个 `@TableName` 注解都不用加！！！

### Q3: GlobalConfig、DbConfig 这种“套娃代码”到底怎么读？

```java
GlobalConfig globalConfig = new GlobalConfig();           // 拿出一张外卖总单
GlobalConfig.DbConfig dbConfig = new GlobalConfig.DbConfig(); // 从总单口袋里掏出“数据库配置卡”
dbConfig.setTablePrefix("t_");                             // 在卡片上写：表前缀是 t_
globalConfig.setDbConfig(dbConfig);                        // 把写好字的卡片塞回总单的口袋
```

## @TableId

### 1. @Table的作用

`@TableId` 用来标注实体类中的哪个字段是数据库表的主键。如果不加这个注解，MyBatis-Plus 默认会把字段名叫 `id` 的当成主键，但是一旦你的主键字段名不是 `id`（比如叫 `user_id、uid`），或者你想明确指定主键生成策略，就必须显式加上 `@TableId`。

### 2. 基本用法示例

```java
@TableName("t_user")  // 表名
public class User {
    
    @TableId(type = IdType.AUTO)  // 主键，数据库自增
    private Long id;

    // 或者不写 type，使用默认策略（推荐后面会说为什么）
    @TableId
    private Long id;

    private String name;
    private Integer age;
}
```

### 3. @TableId 的两个属性

| 属性  | 类型            | 是否必填 | 默认值      | 说明                             |
| ----- | --------------- | -------- | ----------- | -------------------------------- |
| value | String          | 否       | ""          | 对应数据库中的主键列名（很少用） |
| type  | IdType 枚举类型 | 否       | IdType.NONE | 指定主键生成策略，最重要！       |

### 4. IdType所有类型详解

| IdType 值     | 说明                                                         | 适用场景                               | 推荐程度 |
| ------------- | ------------------------------------------------------------ | -------------------------------------- | -------- |
| AUTO          | 数据库自增（依赖数据库的 AUTO_INCREMENT）                    | 单机项目 + MySQL + 主键是自增的        | ★★★★★    |
| NONE          | 不设置主键类型，遵循全局配置（全局默认是 ASSIGN_ID）         | 不确定时先用这个                       | ★★★★     |
| INPUT         | 插入前必须自己手动 set id 值，否则报错或插入 null            | 业务自己生成主键（如订单号、业务单号） | ★★★      |
| ASSIGN_ID     | 自动分配雪花算法生成的 Long 或 String 类型 ID（当前版本推荐方式） | 分布式系统、最常用、最推荐！           | ★★★★★★★  |
| ASSIGN_UUID   | 自动生成 UUID（去掉 - 的32位字符串）                         | 需要字符串主键，不关心顺序和长度       | ★★★★     |
| ID_WORKER     | 老版本分布式 ID（Long 类型）已过时 → 请改用 ASSIGN_ID        | 老项目还在用                           | 不推荐   |
| ID_WORKER_STR | 老版本分布式 ID（String 类型）已过时 → 请改用 ASSIGN_ID      | 老项目还在用                           | 不推荐   |
| UUID          | 老版本 UUID 已过时 → 请改用 ASSIGN_UUID                      | 老项目还在用                           | 不推荐   |

### 5. 2025年当前最佳实践

```java
// 情况1：单表使用数据库自增（传统单体应用）
@TableId(type = IdType.AUTO)
private Long id;

// 情况2：分布式系统、微服务、分库分表（99% 新项目都应该这样）
@TableId(type = IdType.ASSIGN_ID)  // 或者干脆不写 type，因为全局默认就是这个
private Long id;

// 情况3：想用字符串雪花 ID（部分公司喜欢）
@TableId(type = IdType.ASSIGN_ID)
private String id;

// 情况4：最简洁写法（推荐！）
@TableId   // 什么都不写，自动走雪花算法，完美适配分布式环境
private Long id;
```

### 6. 全局配置（application.yml)

你还可以在配置文件里设置全局默认策略，这样实体类里可以完全不写 type：

```yaml
mybatis-plus:
  global-config:
    db-config:
      id-type: assign_id   # 全局默认使用雪花算法
```

这样所有实体类都可以简化为：

```java
@TableId
private Long id;
```

非常清爽！

### 7. 总结：你该怎么选？

| 你的项目类型           | 推荐的 IdType     | 写法建议                            |
| ---------------------- | ----------------- | ----------------------------------- |
| 传统单体应用 + MySQL   | AUTO              | @Table(type=IdType.AUTO)            |
| 新项目、微服务、分布式 | ASSIGN_ID（默认） | @TableId 或者完全不写 type          |
| 主键要用 String        | ASSIGN_ID         | private String id + @TableId        |
| 需要UUID               | ASSIGN_UUID       | @TableId(type = IdType.ASSIGN_UUID) |
| 业务自己生成主键       | INPUT             | 插入前手动 setId()                  |

## **@TableField**

### 1.@TableField的核心作用

用来解决「Java 实体字段」和「数据库表字段」之间的各种“不一致”问题。

正常情况（驼峰命名）可以完全不写这个注解：

```java
private String userName;     → 自动映射到 user_name（默认开启驼峰转下划线）
private Integer age;         → 自动映射到 age
```

### 2.99%情况下会用到的3个经典场景

| 场景                               | 问题                                               | 正确写法                                                     | 说明               |
| ---------------------------------- | -------------------------------------------------- | ------------------------------------------------------------ | ------------------ |
| 1. 字段名不一致                    | Java 是 realName，数据库是 real_name2              | @TableField("real_name2") private String realName;           | 最常见！           |
| 2. 布尔类型以 is 开头              | Java 是 isMarried，MP 会误认为数据库字段是 married | @TableField("is_married") private Boolean isMarried;         | 超级容易踩坑！     |
| 3. 数据库字段是 SQL 关键字或保留字 | 数据库字段叫 order、desc、concat、key 等           | @TableField("order") private String order; @TableField("concat") private String concat; | 必须用反引号包起来 |

示例：

```java
@TableName("t_user")
public class User {
    @TableId
    private Long id;

    private String userName;                    // 自动映射 user_name，OK

    @TableField("real_name")                    // 场景1
    private String realName;

    @TableField("is_married")                   // 场景2 重点！
    private Boolean isMarried;

    @TableField("`desc`")                       // 场景3 重点！
    private String desc;

    @TableField("`concat`")                     // 场景3
    private String concat;
}
```

### 3. @TableField 所有重要属性全解析

| 属性           | 类型                         | 常用度 | 默认值    | 说明与最佳实践                                               |
| -------------- | ---------------------------- | ------ | --------- | ------------------------------------------------------------ |
| value          | String                       | ★★★★★  | ""        | 指定数据库列名（上面3大场景全靠它）                          |
| exist          | boolean                      | ★★★★   | true      | false 表示这个字段根本不存在于表中（常用于逻辑字段、计算字段） 例：private String notInDb; + @TableField(exist = false) |
| select         | boolean                      | ★★★    | true      | false 表示查询时不 select 这个字段（用于超大文本、敏感字段提升性能） |
| fill           | FieldFill                    | ★★★★★  | DEFAULT   | 自动填充！常用于 createTime、updateTime 配合 @TableField(fill = FieldFill.INSERT_UPDATE) |
| insertStrategy | FieldStrategy                | ★★★    | DEFAULT   | 控制 insert 时是否包含该字段 常用 NOT_NULL：null 就不插了    |
| updateStrategy | FieldStrategy                | ★★★★   | DEFAULT   | 控制 update 时是否 set 该字段 常用 IGNORED：无论 null 不 null 都更新 |
| whereStrategy  | FieldStrategy                | ★★     | DEFAULT   | 控制查询条件拼接时是否加入该字段 常用 NOT_EMPTY：空字符串也不加入 |
| jdbcType       | JdbcType                     | ★★★    | UNDEFINED | 明确指定 JDBC 类型，解决某些 null 插入警告                   |
| typeHandler    | Class<? extends TypeHandler> | ★★     | -         | 自定义类型处理器（如 JSON 字段、枚举等）                     |
| numericScale   | String                       | ★      | ""        | 指定小数位数，一般不用                                       |

### 4.FieldStrategy 策略枚举

| 值        | 含义                            | 典型用途                               |
| --------- | ------------------------------- | -------------------------------------- |
| IGNORED   | 完全忽略判断，字段永远参与      | update 时强制更新某个字段（包括 null） |
| NOT_NULL  | 非 null 才参与                  | insert/update 最常用，避免插 null      |
| NOT_EMPTY | 非空（null 和 "" 都不行）才参与 | 字符串查询条件常用                     |
| DEFAULT   | 走全局配置（默认就是 NOT_NULL） | 一般不用显式写                         |

推荐全局配置（application.yml）：

```yaml
mybatis-plus:
  global-config:
    db-config:
      insert-strategy: NOT_NULL
      update-strategy: NOT_NULL   # 或者改成 IGNORED 看团队习惯
      select-strategy: NOT_NULL
```

### 5.自动填充

```java
// 创建时间 + 更新时间
@TableField(fill = FieldFill.INSERT)
private LocalDateTime createTime;

@TableField(fill = FieldFill.INSERT_UPDATE)
private LocalDateTime updateTime;

// 必须实现一个 MetaObjectHandler
@Component
public class MyMetaObjectHandler implements MetaObjectHandler {
    @Override
    public void insertFill(MetaObject metaObject) {
        this.strictInsertFill(metaObject, "createTime", LocalDateTime::now, LocalDateTime.class);
        this.strictUpdateFill(metaObject, "updateTime", LocalDateTime::now, LocalDateTime.class);
    }

    @Override
    public void updateFill(MetaObject metaObject) {
        this.strictUpdateFill(metaObject, "updateTime", LocalDateTime::now, LocalDateTime.class);
    }
}
```

### 6.2025年推荐的实体类最佳写法总结

```java
@TableName("t_user")
@Data
public class User {
    @TableId(type = IdType.ASSIGN_ID)
    private Long id;

    // 普通字段啥都不用写
    private String username;
    private String phone;

    // 特殊字段才加注解
    @TableField("real_name")
    private String realName;

    @TableField("is_vip")
    private Boolean isVip;

    @TableField("`desc`")
    private String desc;

    // 自动填充
    @TableField(fill = FieldFill.INSERT)
    private LocalDateTime createTime;

    @TableField(fill = FieldFill.INSERT_UPDATE)
    private LocalDateTime updateTime;

    // 不在数据库的字段
    @TableField(exist = false)
    private String tempData;
}
```

记住一句话： 

能不写 `@TableField`就不写，只有“名字对不上”或“特殊需求”才加！

### 7.为什么这么多属性

1. 核心功能

```java
@TableField("real_name")
private String realName;
```

→ 就是告诉 MyBatis-Plus：“别傻乎乎地驼峰转下划线了，这个 Java 字段对应数据库里叫 real_name”
包括 isVip → is_vip、order → order 这类特殊命名都靠它。

2. 真正让它变“复杂”的 5 大类功能（每类解决一类痛点）

| 分类                                | 属性           | 解决什么痛点                                          | 实际场景举例                                                 |
| ----------------------------------- | -------------- | ----------------------------------------------------- | ------------------------------------------------------------ |
| 1. 这个字段到底存不存在数据库？     | exist          | 有的字段只在 Java 里用，不想建列                      | 临时计算字段、VO 字段、前端传的额外参数 private String passwordAgain; → @TableField(exist = false) |
| 2. 查询时要不要 select 这一列？     | select         | 有的列特别大，或者是敏感字段，日常查询不想查出来      | 超大 text、blob、用户密码、身份证号 @TableField(select = false) private String idCard; |
| 3. 插入时要不要把这个字段塞进 SQL？ | insertStrategy | 想让 null 的字段干脆就不出现在 insert 语句里          | 所有字段默认用 NOT_NULL（全局配），但个别字段想连 null 也插进去，就写 IGNORED |
| 4. 修改时要不要 set 这个字段？      | updateStrategy | 最常见需求：想把 null 也更新成数据库 null，而不是跳过 | 清除某个字段（如清除头像） @TableField(updateStrategy = FieldStrategy.IGNORED) private String avatar; |
| 5. 自动填充（创建时间、更新人等）   | fill           | 每次 insert/update 自动填值，再也不用手动 set         | createTime、updateTime、createBy、updateBy 四大时间字段      |

3. 举一个真实项目里几乎所有属性都会用上的实体例子

```java
@TableName("t_user")
@Data
public class User {

    @TableId(type = IdType.ASSIGN_ID)
    private Long id;

    // 普通字段，什么都不用写
    private String username;

    // 1. 数据库列名对不上
    @TableField("real_name")
    private String realName;

    // 2. 布尔值 is 开头
    @TableField("is_vip")
    private Boolean isVip;

    // 3. 字段是关键字
    @TableField("`desc`")
    private String desc;

    // 4. 自动填充创建时间
    @TableField(fill = FieldFill.INSERT)
    private LocalDateTime createTime;

    // 5. 自动填充更新时间
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private LocalDateTime updateTime;

    // 6. 密码只更新，不查询
    @TableField(select = false, updateStrategy = FieldStrategy.IGNORED)
    private String password;

    // 7. 头像允许清空（传 null 也要更新）
    @TableField(updateStrategy = FieldStrategy.IGNORED)
    private String avatar;

    // 8. 身份证号不查询（敏感信息）
    @TableField(select = false)
    private String idCard;

    // 9. 临时字段，前端传的确认密码，完全不存数据库
    @TableField(exist = false)
    private String passwordAgain;
}
```

*看到没有？一个实体里可能同时出现上面 9 种需求，`@TableField`必须把这些能力全包进来。*

4. 总结：为什么 `@TableField` 这么“多”属性？

| 需求                           | 如果没有对应属性会怎样？ | 所以必须有这个属性        |
| ------------------------------ | ------------------------ | ------------------------- |
| 列名不一致                     | 映射失败，报错           | value                     |
| 字段不在表里                   | 自动生成列或报错         | exist = false             |
| 大字段拖慢查询                 | 每次都查出来很慢         | select = false            |
| 想把 null 更新成数据库 null    | 默认跳过不更新           | updateStrategy = IGNORED  |
| 想自动填时间                   | 每次都要手动 set         | fill                      |
| 想让 null 不出现在 insert 语句 | 产生一大堆 null          | insertStrategy = NOT_NULL |
