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

# 知识3： 常见配置

## 一、我要改哪些配置？（尤其是没有默认值的）

**1. MyBatis-Plus 的配置到底写在哪儿？**

全部写在 `application.yml`（或 application.properties）里，固定前缀是：

```yaml
mybatis-plus
```

**2. 真正常用的配置**

| 配置项                                     | 是否有默认值                          | 强烈建议你配吗？     | 推荐写法与说明                                        |
| ------------------------------------------ | ------------------------------------- | -------------------- | ----------------------------------------------------- |
| type-aliases-package                       | 无默认值                              | 强烈建议配           | 实体类包扫描 com.xxx.domain.po 或 com.xxx.entity      |
| global-config.db-config.id-type            | 有默认值（3.3.0+ 起默认是 ASSIGN_ID） | 老项目建议显式配     | 新项目推荐 assign_id（雪花算法） 单库 MySQL 推荐 auto |
| mapper-locations                           | 有默认值                              | 一般不用改           | 默认就是 classpath*:/mapper/**/*.xml                  |
| configuration.map-underscore-to-camel-case | 有默认值 true                         | 一般不用改           | 驼峰转下划线，默认开启就行                            |
| configuration.log-impl                     | 无                                    | 建议配（开发看 SQL） | org.apache.ibatis.logging.stdout.StdOutImpl           |

推荐的最简洁 application.yml（新项目直接抄）

```yaml
mybatis-plus:
  # 1. 实体类包扫描（没有这个会报错！很多新手就卡在这）
  type-aliases-package: com.itheima.mp.domain.po

  # 2. 全局主键策略（强烈建议显式写一个）
  global-config:
    db-config:
      id-type: assign_id    # 分布式雪花算法（推荐）
      # id-type: auto       # 如果你是单机MySQL自增，就写这个

  # 3. 手写mapper.xml的路径（默认就是这个，一般不用改）
  mapper-locations: classpath*:/mapper/**/*.xml

  # 4. 开发时打印SQL（生产环境删掉或改成日志实现）
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```



## 二、我想手写 SQL，怎么放 mapper.xml 文件才会被 MyBatis-Plus 自动加载？

**1. 手写 mapper.xml 的核心规则**

![](E:\Program Data\BlackHorse Data\SpringCloud Data\springcloud-notes\day00\img\usermap.png)


MyBatis-Plus 完全兼容原生 MyBatis 的 mapper.xml 文件，只要你遵守下面规则，它就会自动加载：

1. 文件必须放在 resources/mapper/ 目录（或其子目录）下
   常见结构：

   ```text
   src/main/resources/
   └── mapper/
       ├── UserMapper.xml
       └── user/
           └── UserMapper.xml   都可以！
   ```

2. 默认扫描规则就是 classpath*:/mapper/**/*.xml
   → 所以你只要放在 resources/mapper 下面任意层级，都一定会被加载！

3. xml 内容和原生 MyBatis 完全一样

你图片里的例子（完美正确）：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.itheima.mp.mapper.UserMapper">

    <!-- 手写一个查询方法 -->
    <select id="queryById" resultType="User">
        SELECT * FROM user WHERE id = #{id}
    </select>

</mapper>
```

对应的接口要这样写：

```java
public interface UserMapper extends BaseMapper<User> {

    // 手写的方法必须加 @Select 或者放在 xml 里
    // 这里选择放在 xml，所以接口里只要声明就行
    User queryById(Long id);
}
```

测试一下就出结果：

```java
@Test
void testQuery() {
    User user = userMapper.queryById(1L);
    System.out.println(user);
}
```

**2. 总结**

新手最容易踩的 3 个坑（你现在看完就不会踩了）

| 坑                        | 现象                                              | 解决办法                          |
| ------------------------- | ------------------------------------------------- | --------------------------------- |
| 没配 type-aliases-package | 启动就报错 Cannot find entity                     | 加上实体包路径                    |
| mapper.xml 放错位置了     | 启动没报错，但手写 SQL 不生效                     | 必须放在 resources/mapper/ 目录下 |
| 全局 id-type 不配         | 老版本默认是 AUTO，新版本默认 ASSIGN_ID，容易混乱 | 显式配一个，团队统一              |

记住这张图就一辈子不会忘：

```text
resources/
└── mapper/                ← 手写 xml 只能放这里！
    └── UserMapper.xml
application.yml            ← 配置 type-aliases-package 和 id-type
```

全配好后，你的 MyBatis-Plus 就真正“开箱即用”+“手写 SQL 随便用”了！

# 知识4： 核心功能

## 1.条件构造器

除了新增以外，修改、删除、查询的SQL语句都需要指定`where`条件。

因此`BaseMapper`中提供的相关方法除了以`id`作为`where`条件以外，还支持更加复杂的`where`条件。

![](D:\ProgramData\Typora Data\springcloud-notes\day00\img\BaseMapper1.png)

记住规律： 只要方法的最后一个参数是 Wrapper，那就一定支持任意复杂 where 条件！

| 方法                        | 含义                      | 推荐 Wrapper 类型   |
| --------------------------- | ------------------------- | ------------------- |
| delete(Wrapper)             | 删                        | LambdaQueryWrapper  |
| update(entity, Wrapper)     | 改（推荐 entity 传 null） | LambdaUpdateWrapper |
| selectCount(Wrapper)        | 查总数                    | LambdaQueryWrapper  |
| selectList / selectMaps     | 查列表                    | LambdaQueryWrapper  |
| selectOne                   | 查一条                    | LambdaQueryWrapper  |
| selectPage / selectMapsPage | 分页查                    | LambdaQueryWrapper  |
| exists(Wrapper)             | exists 查询               | LambdaQueryWrapper  |

---

参数中的`Wrapper`就是条件构造的抽象类，其下有很多默认实现，继承关系如图：

![](E:\Program Data\BlackHorse Data\SpringCloud Data\springcloud-notes\day00\img\Wrapper.png)

```text
记住一句话：想查 → 用 QueryWrapper 或 LambdaQueryWrapper  
想改 → 用 UpdateWrapper 或 LambdaUpdateWrapper  
强烈推荐永远使用 Lambda 版本（LambdaQueryWrapper / LambdaUpdateWrapper），因为可以防止字段名写错 + 重构安全！
```

---

`Wrapper`的子类`AbstractWrapper`提供了where中包含的所有条件构造方法：

![](E:\Program Data\BlackHorse Data\SpringCloud Data\springcloud-notes\day00\img\AbstractWrapper.png)

1. 基本比较

```java
eq      → = 
ne      → !=
gt      → >
ge      → >=
lt      → <
le      → <=
between → BETWEEN ? AND ?
notBetween
in      → IN (.....)
notIn
like    → LIKE '%值%'
likeLeft  → LIKE '%值'
likeRight → LIKE '值%'
notLike
```

2. 空值处理（超级实用！）

```java
isNull      → IS NULL
isNotNull   → IS NOT NULL
```

3. 嵌套、or、and

```java
and(wrapper -> wrapper.gt("age", 18).eq("status", 1))
or(wrapper -> wrapper.lt("age", 18).or().eq("status", 2))
```

4. 排序、分组、having

```java
orderByAsc("create_time")
orderByDesc("id")
groupBy("dept_id")
having("count(*) > {0}", 5)
```

5. select 指定列（不想查所有列时非常有用）

```java
.select("id", "name", "age")                 // 字符串版
.select(User::getId, User::getName)         // Lambda 版（推荐！）
```

---

而QueryWrapper在AbstractWrapper的基础上拓展了一个select方法，允许指定查询字段：

![](E:\Program Data\BlackHorse Data\SpringCloud Data\springcloud-notes\day00\img\QueryWrapper.png)
而UpdateWrapper在AbstractWrapper的基础上拓展了一个set方法，允许指定SQL中的SET部分：

![](E:\Program Data\BlackHorse Data\SpringCloud Data\springcloud-notes\day00\img\UpdateWrapper.png)

---

QueryWrapper vs UpdateWrapper 的最大区别（最后一张图）

| Wrapper             | 能写 select 列？                                         | 能写 set 字段？                                              | 典型用途                           |
| ------------------- | -------------------------------------------------------- | ------------------------------------------------------------ | ---------------------------------- |
| QueryWrapper        | ![✅](https://abs-0.twimg.com/emoji/v2/svg/2705.svg) 可以 | ![❌](https://abs-0.twimg.com/emoji/v2/svg/274c.svg) 不行     | 专门用来 WHERE + SELECT            |
| LambdaQueryWrapper  | ![✅](https://abs-0.twimg.com/emoji/v2/svg/2705.svg) 可以 | ![❌](https://abs-0.twimg.com/emoji/v2/svg/274c.svg) 不行     | 同上，但防字段写错                 |
| UpdateWrapper       | ![❌](https://abs-0.twimg.com/emoji/v2/svg/274c.svg) 不行 | ![✅](https://abs-0.twimg.com/emoji/v2/svg/2705.svg) 可以 set | 专门用来 WHERE + SET（更新）       |
| LambdaUpdateWrapper | ![❌](https://abs-0.twimg.com/emoji/v2/svg/274c.svg) 不行 | ![✅](https://abs-0.twimg.com/emoji/v2/svg/2705.svg) 可以 set | 同上，但防字段写错（最推荐更新用） |

---

实际开发中最最最常用的 4 种写法（直接抄就行）

```java
// 1. 复杂条件查询 + 分页（日常 80% 场景）
userMapper.selectPage(
    new Page<>(pageNum, pageSize),
    new LambdaQueryWrapper<User>()
        .eq(User::getDeleted, 0)
        .like(StringUtils.hasText(name), User::getName, name)
        .ge(birthStart != null, User::getBirthday, birthStart)
        .le(birthEnd != null, User::getBirthday, birthEnd)
        .in(User::getDeptId, deptIds)
        .orderByDesc(User::getCreateTime)
);

// 2. 复杂条件更新（超级常用！）
userMapper.update(null,
    new LambdaUpdateWrapper<User>()
        .set(User::getStatus, 2)
        .set(User::getUpdateTime, LocalDateTime.now())
        .in(User::getId, idList)
);

// 3. 复杂条件删除
userMapper.delete(
    new LambdaQueryWrapper<User>()
        .in(User::getId, ids)
);

// 4. 统计 + 复杂条件
Long count = userMapper.selectCount(
    new LambdaQueryWrapper<User>()
        .eq(User::getStatus, 1)
        .gt(User::getCreateTime, yesterday)
```

---

终极结论（背下来就无敌了）

1. 看到方法最后参数是 Wrapper → 就支持任意复杂 where  
2. 查东西 → 用 LambdaQueryWrapper  
3. 改东西 → 用 LambdaUpdateWrapper（第一个参数一般传 null）  
4. 永远使用 Lambda 版本，永不写错字段名！

---

接下来，我们就来看看如何利用`Wrapper`实现复杂查询。

### 1.1.QueryWrapper

无论是修改、删除、查询，都可以使用QueryWrapper来构建查询条件。接下来看一些例子：

**查询**：查询出名字中带`o`的，存款大于等于1000元的人。代码如下：

```Java
@Test
void testQueryWrapper() {
    // 1.构建查询条件 where name like "%o%" AND balance >= 1000
    QueryWrapper<User> wrapper = new QueryWrapper<User>()
            .select("id", "username", "info", "balance")
            .like("username", "o")
            .ge("balance", 1000);
    // 2.查询数据
    List<User> users = userMapper.selectList(wrapper);
    users.forEach(System.out::println);
}
```

生成的 SQL 类似：

```sql
SELECT id,username,info,balance 
FROM user 
WHERE username LIKE ? AND balance >= ?
-- 参数: %o%, 1000
```

重点技巧：

- .select(...) 可以只查需要的列，性能更高（不写就查所有列）
- .like("username", "o") → 自动变成 %o%（两边模糊）
- 想左模糊用 .likeLeft("username","o") → %o
- 想右模糊用 .likeRight("username","o") → o%

---

**更新**：更新用户名为jack的用户的余额为2000，代码如下：

```Java
@Test
void testUpdateByQueryWrapper() {
    // 1.构建查询条件 where name = "Jack"
    QueryWrapper<User> wrapper = new QueryWrapper<User>().eq("username", "Jack");
    // 2.更新数据，user中非null字段都会作为set语句
    User user = new User();
    user.setBalance(2000);
    userMapper.update(user, wrapper);
}
```

生成的 SQL 类似：

```sql
UPDATE user 
SET balance = 2000 
WHERE username = 'Jack'
```

超级重要细节：

- 实体 user 中哪些字段为 null 就不会出现在 SET 里 → 这就是 MyBatis-Plus 的“选择性更新”
- 如果你想强制某些字段设为 null，用 UpdateWrapper 更方便（后面会讲）

---

**QueryWrapper 是 MyBatis-Plus 中最最最最最最常用的条件构造器**，几乎所有“带复杂 where”的操作都能用它搞定！一、QueryWrapper 能干 3 件大事（记住这三句话就够了）

| 场景 | 怎么用 QueryWrapper                | 实际执行的 SQL 片段                      |
| ---- | ---------------------------------- | ---------------------------------------- |
| 查询 | userMapper.selectXXX(wrapper)      | 只生成 WHERE 条件                        |
| 更新 | userMapper.update(entity, wrapper) | 生成 WHERE 条件 + SET（set 来自 entity） |
| 删除 | userMapper.delete(wrapper)         | 只生成 WHERE 条件                        |

一句话总结：QueryWrapper 专门负责生成 WHERE 子句，不管你后面是要查、改、删，它都行！

---

**QueryWrapper 常用方法速查表（直接抄走）**

| 方法                                  | 生成的 SQL            | 例子                                 |
| ------------------------------------- | --------------------- | ------------------------------------ |
| eq(col, val)                          | col = val             | .eq("status", 1)                     |
| ne(col, val)                          | col <> val            |                                      |
| gt(col, val)                          | col > val             |                                      |
| ge(col, val)                          | col >= val            | .ge("balance", 1000)                 |
| lt(col, val) / le(col, val)           | < / <=                |                                      |
| between(col, v1, v2)                  | col BETWEEN v1 AND v2 |                                      |
| like(col, val)                        | col LIKE '%val%'      | .like("name","o")                    |
| likeLeft / likeRight                  | %val / val%           |                                      |
| in(col, list)                         | col IN (?,?,?)        | .in("id", ids)                       |
| isNull(col) / isNotNull(col)          | IS NULL / IS NOT NULL |                                      |
| or()                                  | 加一个 OR             | .eq("age",18).or().eq("age",65)      |
| and(Consumer<Param>)                  | 括号嵌套              | .and(i->i.gt("age",18).lt("age",60)) |
| orderByAsc(col) / orderByDesc         | ORDER BY col ASC/DESC | .orderByDesc("create_time")          |
| having(sql, params...)                | HAVING 子句           |                                      |
| apply("date_format(...) > {0}", date) | 拼原生 SQL（慎用）    |                                      |

---

**强烈推荐升级写法：**用 LambdaQueryWrapper 代替 QueryWrapper字符串写法容易写错字段、重构后报错：

```java
new QueryWrapper<User>().eq("usename", "Jack")  // 拼错了！运行时才报错
```

Lambda 写法编译期就报错，彻底杜绝字段写错：

```java
new LambdaQueryWrapper<User>()
    .like(User::getUsername, "o")
    .ge(User::getBalance, 1000)
    .orderByDesc(User::getCreateTime);
```

### 1.2.UpdateWrapper

基于BaseMapper中的update方法更新时只能直接赋值，对于一些复杂的需求就难以实现。 

例如：更新id为`1,2,4`的用户的余额，扣200，对应的SQL应该是：

```Java
UPDATE user SET balance = balance - 200 WHERE id in (1, 2, 4)
```

SET的赋值结果是基于字段现有值的，这个时候就要利用UpdateWrapper中的setSql功能了：

```Java
@Test
void testUpdateWrapper() {
    List<Long> ids = List.of(1L, 2L, 4L);
    // 1.生成SQL
    UpdateWrapper<User> wrapper = new UpdateWrapper<User>()
            .setSql("balance = balance - 200") // SET balance = balance - 200
            .in("id", ids); // WHERE id in (1, 2, 4)
        // 2.更新，注意第一个参数可以给null，也就是不填更新字段和数据，
    // 而是基于UpdateWrapper中的setSQL来更新
    userMapper.update(null, wrapper);
}
```

---

**超级重点来了！这才是 MyBatis-Plus 真正“无敌”的地方**

—— UpdateWrapper + setSql() 实现“字段自增、自减、拼接、函数计算”等复杂更新

你举的这个例子，完美戳中了 QueryWrapper + 实体更新的最大痛点：

| 场景                                   | 能用实体 update(entity, wrapper) 实现吗？ | 必须用 UpdateWrapper + setSql 吗？ |
| -------------------------------------- | ----------------------------------------- | ---------------------------------- |
| 普通赋值：balance = 2000               | 可以                                      | 不需要                             |
| 自增：balance = balance + 100          | 不行（实体只能写死值）                    | 必须！                             |
| 自减：balance = balance - 200          | 不行                                      | 必须！                             |
| 拼接字符串：name = concat(name,'_vip') | 不行                                      | 必须！                             |
| 时间计算：update_time = now()          | 不行（实体会带时区问题）                  | 推荐！                             |

**一、核心结论（背下来就无敌）**

```text
想实现 “SET 字段 = 字段 ±/* 某个值” 或 “SET 字段 = 函数()” 这类操作
→ 必须用 UpdateWrapper + setSql() / set()
→ 第一个参数传 null
```

**二、UpdateWrapper 三种 set 写法的对比（超级清晰表格）**

| 写法                                    | 生成的 SQL                  | 推荐场景                       |
| --------------------------------------- | --------------------------- | ------------------------------ |
| .set("balance", 2000)                   | SET balance = 2000          | 普通赋值（等价于实体更新）     |
| .setSql("balance = balance - 200")      | SET balance = balance - 200 | 自增自减、复杂计算（最常用！） |
| .setSql("balance = balance - {0}", 200) | 同上，但支持占位符更安全    | 参数较多时推荐                 |

**三、经典实战案例（直接抄到项目里）**

```java
// 1. 余额扣200（你举的例子，最经典！）
userMapper.update(null,
    new LambdaUpdateWrapper<User>()
        .setSql("balance = balance - 200")
        .in(User::getId, List.of(1L,2L,4L))
);

// 2. 积分 +100
userMapper.update(null,
    new LambdaUpdateWrapper<User>()
        .setSql("score = score + 100")
        .eq(User::getId, 5)
);

// 3. 库存扣减（电商必备）
userMapper.update(null,
    new LambdaUpdateWrapper<Product>()
        .setSql("stock = stock - ?", quantity)  // 推荐用占位符
        .eq(Product::getId, productId)
        .gt("stock", quantity)  // 防止库存变成负数（乐观锁思想）
);

// 4. 字符串拼接
userMapper.update(null,
    new LambdaUpdateWrapper<User>()
        .setSql("name = concat(name, '_VIP')")
        .eq(User::getId, 1)
);

// 5. 多字段同时计算（一条 SQL 搞定！）
userMapper.update(null,
    new LambdaUpdateWrapper<User>()
        .setSql("balance = balance - 200, score = score + 50, update_time = now()")
        .in(User::getId, ids)
);
```

**四、推荐永远用 LambdaUpdateWrapper（防字段写错 + 重构友好）**

```java
new LambdaUpdateWrapper<User>()
    .setSql("balance = balance - 200")
    .in(User::getId, ids)
    .eq(User::getStatus, 1)
    .ge(User::getBalance, 200); // 防止余额变成负数
```

**五、终极总结口诀（背下来，面试装逼 + 日常开发都无敌）**

```text
普通赋值 → 用实体 update(user, wrapper)
复杂计算 → 必须 UpdateWrapper + setSql + 第一个参数传 null
推荐写法 → 永远使用 LambdaUpdateWrapper + setSql
```

记住这张图就够了：

```text
update(null, 
    new LambdaUpdateWrapper<T>()
        .setSql("字段 = 字段 ± 值、函数、拼接...")
        .where条件...
);
```

掌握了这个技巧，你就已经进入了 MyBatis-Plus 高阶玩家行列
—— 再也不用为了“库存扣减”“积分增减”“版本号+1”去写 XML 或者原生 SQL 了！一条 Java 代码，优雅暴击！

### 1.3.LambdaQueryWrapper

无论是QueryWrapper还是UpdateWrapper在构造条件的时候都需要写死字段名称，会出现字符串`魔法值`。这在编程规范中显然是不推荐的。 那怎么样才能不写字段名，又能知道字段名呢？

其中一种办法是基于变量的`gettter`方法结合反射技术。因此我们只要将条件对应的字段的`getter`方法传递给MybatisPlus，它就能计算出对应的变量名了。而传递方法可以使用JDK8中的`方法引用`和`Lambda`表达式。 因此MybatisPlus又提供了一套基于Lambda的Wrapper，包含两个：

- LambdaQueryWrapper
- LambdaUpdateWrapper

分别对应QueryWrapper和UpdateWrapper

其使用方式如下：

```Java
@Test
void testLambdaQueryWrapper() {
    // 1.构建条件 WHERE username LIKE "%o%" AND balance >= 1000
    QueryWrapper<User> wrapper = new QueryWrapper<>();
    wrapper.lambda()
            .select(User::getId, User::getUsername, User::getInfo, User::getBalance)
            .like(User::getUsername, "o")
            .ge(User::getBalance, 1000);
    // 2.查询
    List<User> users = userMapper.selectList(wrapper);
    users.forEach(System.out::println);
}
```

---

**LambdaQueryWrapper —— 真正“无敌”的写法（2025 年你还写字符串字段名就落伍了）你已经看到了问题的核心：**

```java
// 老古董写法（坚决淘汰！）
new QueryWrapper<User>()
    .like("username", "o")     // ← 字符串魔法值！重构改字段名？炸！
    .ge("balance", 1000)       // ← 运行时才报错，线上事故预定
```

MyBatis-Plus 官方自己都说：“字符串写列名是历史遗留问题，我们强烈不推荐”所以从 3.0 开始就推出了 Lambda 系列 Wrapper，彻底终结魔法值！

**一、核心结论（背下来就起飞）**

| Wrapper 类型        | 字符串写法（已淘汰） | Lambda 写法（2025 主流）     | 是否防字段名写错 |
| ------------------- | -------------------- | ---------------------------- | ---------------- |
| QueryWrapper        | .like("name","o")    | .like(User::getName, "o")    | ×                |
| LambdaQueryWrapper  | 不存在               | .like(User::getName, "o")    | √（编译期报错）  |
| UpdateWrapper       | .set("balance",2000) | .set(User::getBalance, 2000) | ×                |
| LambdaUpdateWrapper | 不存在               | .set(User::getBalance, 2000) | √                |

**二、官方例子深度升级（你贴的代码还能再简洁）你写的这坨其实还能再优雅：**

```java
@Test
void testLambdaQueryWrapper() {
    // 错误写法！你还是用了 QueryWrapper，只不过是调用了 .lambda()
    QueryWrapper<User> wrapper = new QueryWrapper<>();
    wrapper.lambda()
           .select(User::getId, User::getUsername, User::getInfo, User::getBalance)
           .like(User::getUsername, "o")
           .ge(User::getBalance, 1000);

    List<User> users = userMapper.selectList(wrapper);
}
```

正确写法（2025 年标配，直接 new LambdaQueryWrapper）：

```java
@Test
void testLambdaQueryWrapper() {
    // 正确！直接使用 LambdaQueryWrapper
    List<User> users = userMapper.selectList(
        new LambdaQueryWrapper<User>()
            .select(User::getId, User::getUsername, User::getInfo, User::getBalance)
            .like(User::getUsername, "o")
            .ge(User::getBalance, 1000)
    );
    
    users.forEach(System.out::println);
}
```

甚至可以再简化成一行（生产里都这么写）：

```java
List<User> users = userMapper.selectList(new LambdaQueryWrapper<User>()
    .like(User::getUsername, "o")
    .ge(User::getBalance, 1000));
```

**三、Lambda 系列完整对照表（直接抄走）**

| 场景                     | QueryWrapper 写法（×）                            | Lambda 写法（强烈推荐）                                  |
| ------------------------ | ------------------------------------------------- | -------------------------------------------------------- |
| 等值                     | .eq("status", 1)                                  | .eq(User::getStatus, 1)                                  |
| 大于等于                 | .ge("balance", 1000)                              | .ge(User::getBalance, 1000)                              |
| like 模糊                | .like("username","o")                             | .like(User::getUsername, "o")                            |
| in                       | .in("id", ids)                                    | .in(User::getId, ids)                                    |
| 排序                     | .orderByDesc("create_time")                       | .orderByDesc(User::getCreateTime)                        |
| 指定查询列               | .select("id","name")                              | .select(User::getId, User::getName)                      |
| 复杂更新（set）          | .set("score", 100)                                | .set(User::getScore, 100)                                |
| 复杂更新（setSql）       | .setSql("score = score + 100")                    | .setSql("score = score + 100")（Lambda 版也支持）        |
| 条件判断（防止拼空条件） | .like(StringUtils.isNotBlank(name), "name", name) | .like(StringUtils.isNotBlank(name), User::getName, name) |

**四、为什么 Lambda 写法能彻底碾压字符串？**

| 对比项     | 字符串写法                           | Lambda 写法                           |
| ---------- | ------------------------------------ | ------------------------------------- |
| 字段改名后 | 运行时报错（Column 'xxx' not found） | IDEA 直接红，编译都过不了             |
| 重构支持   | 不支持                               | 完全支持（改字段名 → 自动改所有引用） |
| 代码提示   | 无                                   | 全程有提示                            |
| 可读性     | 差                                   | 极高（看到方法引用就知道是哪个字段）  |
| 性能       | 略慢（要解析字符串）                 | 略快（反射缓存）                      |

**五、2025 年最新最佳实践（直接套用就行）**

```java
// 1. 普通查询
userMapper.selectList(new LambdaQueryWrapper<User>()
    .eq(User::getStatus, 1)
    .like(StringUtils.hasText(name), User::getName, name)
    .orderByDesc(User::getCreateTime));

// 2. 分页查询（万年不变模板）
IPage<User> page = userMapper.selectPage(new Page<>(current, size),
    new LambdaQueryWrapper<User>()
        .eq(User::getDeleted, 0)
        .like(StringUtils.hasText(keyword), User::getName, keyword)
        .or()
        .like(StringUtils.hasText(keyword), User::getEmail, keyword)
);

// 3. 复杂更新（库存扣减、积分+100 等）
userMapper.update(null, new LambdaUpdateWrapper<User>()
    .setSql("balance = balance - 200, score = score + 50")
    .in(User::getId, ids));
```

**终极结论（面试 + 日常开发必背）**

从今天起，看到任何人还在用 QueryWrapper/UpdateWrapper 写字符串字段名，你就可以理直气壮地说：“老哥，2025 年了，你还在写魔法字符串？
LambdaQueryWrapper / LambdaUpdateWrapper 才是王道！
编译期检查 + 重构友好 + 代码可读性拉满，香得一批！”

记住这三行代码，你就已经是公司里 MyBatis-Plus 最靓的仔：

```java
new LambdaQueryWrapper<T>()      // 查
new LambdaUpdateWrapper<T>()     // 改（普通赋值）
new LambdaUpdateWrapper<T>().setSql(...) // 改（复杂计算）
```

—— 从此告别 Column 'xxx' not found，永远不写错字段名！

## 2.自定义SQL

在演示UpdateWrapper的案例中，我们在代码中编写了更新的SQL语句：

![](E:\Program Data\BlackHorse Data\SpringCloud Data\springcloud-notes\day00\img\test1.png)

这种写法在某些企业也是不允许的，因为SQL语句最好都维护在持久层，而不是业务层。就当前案例来说，由于条件是in语句，只能将SQL写在Mapper.xml文件，利用foreach来生成动态SQL。 这实在是太麻烦了。假如查询条件更复杂，动态SQL的编写也会更加复杂。

所以，MybatisPlus提供了自定义SQL功能，可以让我们利用Wrapper生成查询条件，再结合Mapper.xml编写SQL

---

这个老师的笔记讲的是 MyBatis-Plus 里“自定义 SQL”这一小节（2.2 自定义SQL） 的核心精华，属于整个 MyBatis-Plus 课程里“压轴”级别的知识点。用大白话总结一下，他其实就想告诉你下面这几句话（这就是整页笔记的灵魂）：

1. Wrapper 再牛，也有搞不定的复杂 SQL（比如多表联查、子查询、动态 UNION、窗口函数等）。

2. 这时候不要轻易去写 XML！MyBatis-Plus 给你留了好几条“后门”，尽量让代码还保持“无 XML”的优雅。

3. 最推荐的后门就是：在 LambdaUpdateWrapper / LambdaQueryWrapper 里直接用 .setSql() 或 .apply() 塞一段原生 SQL 片段就行了。

4. 你看代码里那句：

   ```java
   .setSql("balance = balance - 200")
   ```

   就是最典型的用法——实现“余额扣 200”这种实体 + Wrapper 完全搞不定的需求，而且还不写 XML。

5. 老师最后还补了一句大实话： “虽然还有 @Select 和 XML 的方式，但只要能用 Wrapper + setSql/apply 搞定，就别写 XML，保持全 Java 代码才是 MyBatis-Plus 的正确打开方式。”

一句话总结这个知识点：“当 Wrapper 不够用了，别慌，先尝试在 Wrapper 里塞原生 SQL（setSql / apply），99% 的复杂需求都能不写 XML 就解决，这才叫真正的 MyBatis-Plus 高手。”这页笔记就是老师在告诉你：
真正的 MyBatis-Plus 大师，从来不轻易写 XML，而是把 SQL 片段“偷偷塞进 Wrapper”里就完事了。

### 2.1.基本用法

以当前案例来说，我们可以这样写：

```Java
@Test
void testCustomWrapper() {
    // 1.准备自定义查询条件
    List<Long> ids = List.of(1L, 2L, 4L);
    QueryWrapper<User> wrapper = new QueryWrapper<User>().in("id", ids);

    // 2.调用mapper的自定义方法，直接传递Wrapper
    userMapper.deductBalanceByIds(200, wrapper);
}
```

然后在UserMapper中自定义SQL：

```Java
package com.itheima.mp.mapper;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.itheima.mp.domain.po.User;
import org.apache.ibatis.annotations.Param;
import org.apache.ibatis.annotations.Update;
import org.apache.ibatis.annotations.Param;

public interface UserMapper extends BaseMapper<User> {
    @Select("UPDATE user SET balance = balance - #{money} ${ew.customSqlSegment}")
    void deductBalanceByIds(@Param("money") int money, @Param("ew") QueryWrapper<User> wrapper);
}
```

这样就省去了编写复杂查询条件的烦恼了。

---

这段笔记讲的是 MyBatis-Plus 中一个非常高级、非常实用的骚操作，属于“真正的老司机写法”，很多教材和老师都不敢讲，因为它有点“超纲”，但在真实企业项目里使用率极高，几乎是 MyBatis-Plus 高阶玩家的标配技巧。

**这段笔记的核心知识点叫：**

**“${ew.customSqlSegment} 万能占位符 + Wrapper 复用”**
—— 让自定义 SQL 也能完美复用 Wrapper 条件，再也不用重复写 where 条件！

**用大白话翻译一下这段代码在干嘛**

你原来的痛点：

- 我想扣 id 为 1、2、4 的用户的余额 200
- 用 UpdateWrapper.setSql() 可以搞定，但每次都要 new 一个 Wrapper
- 如果很多地方都要“根据不同条件扣余额”，岂不是要写很多重复的 Wrapper？

这段代码的牛逼之处在于： 

它只用写一次 Wrapper，然后把这个 Wrapper 像“参数”一样直接塞进自定义 SQL 里，SQL 自动帮你把 where 条件拼好！

**具体怎么做到的？一句话就说透：**

MyBatis-Plus 提供了一个魔法字符串：  

```sql
${ew.customSqlSegment}
```

- ew 是 “entity wrapper” 的缩写
- 只要你在注解里写 ${ew.customSqlSegment}，MyBatis-Plus 就会自动把你传进来的 QueryWrapper/LambdaQueryWrapper 的 WHERE 条件部分 完整地拼接进去（自动加 WHERE 关键字，自动防 SQL 注入）

**你这段代码最终生成的真实 SQL 是：**

```sql
UPDATE user 
SET balance = balance - #{money} 
WHERE id IN (1,2,4)        ← 这句就是 ew.customSqlSegment 自动拼出来的！
```

**对比一下传统写法 vs 这段骚操作**

| 传统写法（麻烦）                    | 这段骚操作（优雅）                             |
| ----------------------------------- | ---------------------------------------------- |
| 每次都要 new UpdateWrapper + setSql | 只 new QueryWrapper 拼条件                     |
| 条件和 SQL 写两处，容易漏条件       | 条件只写一处，SQL 里只管写 SET                 |
| 多个地方扣余额要重复写很多 Wrapper  | 所有扣余额逻辑都调用同一个方法，Wrapper 随意传 |

**真实项目里会怎么用（直接抄）**

```java
// Service 层随便写条件，复用率拉满
public void deduct(Long userId, int money) {
    userMapper.deductBalanceByIds(money, new LambdaQueryWrapper<User>()
        .eq(User::getId, userId)
        .ge(User::getBalance, money)); // 防止余额不足
}

public void batchDeduct(List<Long> userIds, int money) {
    userMapper.deductBalanceByIds(money, new LambdaQueryWrapper<User>()
        .in(User::getId, userIds));
}
```

Mapper 只写一个方法就够了：

```java
@Update("UPDATE user SET balance = balance - #{money} ${ew.customSqlSegment}")
void deductBalanceByIds(@Param("money") int money, @Param(Constants.WRAPPER) Wrapper<User> wrapper);
```

**总结：这段笔记真正想教给你的是**

“别把 Wrapper 只能用在 BaseMapper 的方法上！
 自定义 SQL 也可以通过 ${ew.customSqlSegment} 把 Wrapper 当参数传进来，
 从而实现：条件全用 Wrapper 拼，SQL 只管写 SET/SELECT，彻底解耦，复用性拉满！”

这才是 MyBatis-Plus 真正的“巅峰用法”，学会了这个，你写出来的代码能让面试官直接喊：“兄弟，你这 MyBatis-Plus 玩得飞起啊！”

### 2.2.多表关联

理论上来讲MyBatisPlus是不支持多表查询的，不过我们可以利用Wrapper中自定义条件结合自定义SQL来实现多表查询的效果。 例如，我们要查询出所有收货地址在北京的并且用户id在1、2、4之中的用户 要是自己基于mybatis实现SQL，大概是这样的：

```XML
<select id="queryUserByIdAndAddr" resultType="com.itheima.mp.domain.po.User">
      SELECT *
      FROM user u
      INNER JOIN address a ON u.id = a.user_id
      WHERE u.id
      <foreach collection="ids" separator="," item="id" open="IN (" close=")">
          #{id}
      </foreach>
      AND a.city = #{city}
  </select>
```

可以看出其中最复杂的就是WHERE条件的编写，如果业务复杂一些，这里的SQL会更变态。

但是基于自定义SQL结合Wrapper的玩法，我们就可以利用Wrapper来构建查询条件，然后手写SELECT及FROM部分，实现多表查询。

查询条件这样来构建：

```Java
@Test
void testCustomJoinWrapper() {
    // 1.准备自定义查询条件
    QueryWrapper<User> wrapper = new QueryWrapper<User>()
            .in("u.id", List.of(1L, 2L, 4L))
            .eq("a.city", "北京");

    // 2.调用mapper的自定义方法
    List<User> users = userMapper.queryUserByWrapper(wrapper);

    users.forEach(System.out::println);
}
```

然后在UserMapper中自定义方法：

```Java
@Select("SELECT u.* FROM user u INNER JOIN address a ON u.id = a.user_id ${ew.customSqlSegment}")
List<User> queryUserByWrapper(@Param("ew")QueryWrapper<User> wrapper);
```

当然，也可以在`UserMapper.xml`中写SQL：

```XML
<select id="queryUserByIdAndAddr" resultType="com.itheima.mp.domain.po.User">
    SELECT * FROM user u INNER JOIN address a ON u.id = a.user_id ${ew.customSqlSegment}
</select>
```

---

这段笔记讲的正是 MyBatis-Plus 里最最最最最最最实用的“伪多表查询”终极骚操作，也是 2025 年真实企业项目里 使用频率最高、性价比最高 的多表查询写法。一句话概括这页笔记的灵魂：

“MyBatis-Plus 官方确实不提供原生的多表联查功能，但我们完全可以用 ${ew.customSqlSegment} 这个万能占位符，把 Wrapper 的 where 条件强行塞进手写 JOIN SQL 里，从而实现：  SELECT + FROM + JOIN 自己手写（简单清晰）  WHERE 条件全部交给 Wrapper 自动生成（动态、防注入、重构安全）
→ 既拥有了多表联查能力，又保留了 MyBatis-Plus 无 XML、Lambda 条件构造器的全部优点！”

这段写法到底有多牛？对比三代写法你就秒懂了

| 代数  | 写法                           | WHERE 条件怎么写？               | 缺点                                         | 是否推荐（2025 年） |
| ----- | ------------------------------ | -------------------------------- | -------------------------------------------- | ------------------- |
| 第0代 | 纯 MyBatis XML                 | 手写 <if> <foreach> 等动态标签   | 又臭又长、易出错、不能重构                   | 完全淘汰            |
| 第1代 | MyBatis-Plus Wrapper + apply() | 手写 apply("a.city = '北京'")    | 还是魔法字符串，改字段名要全项目搜索         | 不推荐              |
| 第2代 | 就是你这段笔记的写法           | Wrapper 用 .eq("a.city", "北京") | 条件用 Wrapper，字段改名编译就报错，超级安全 | 强烈推荐！！！      |

这段代码最终生成的真实 SQL（自动拼出来的）

```sql
SELECT u.* 
FROM user u 
INNER JOIN address a ON u.id = a.user_id 
WHERE u.id IN (1,2,4) AND a.city = '北京'     ← 这整个 WHERE 部分都是 ${ew.customSqlSegment} 自动生成的！
```

升级版（2025 年真实项目里都这么写）

```java
// 测试代码（超级简洁）
@Test
void testBestPractice() {
    List<User> users = userMapper.queryUserWithAddress(
        new LambdaQueryWrapper<User>()
            .in(User::getId, List.of(1L, 2L, 4L))
            .eq(Address::getCity, "北京")        // 注意：这里甚至可以直接写 Address 的字段！
            .ge(User::getBalance, 1000)
    );
}
```

```java
// Mapper 接口（只需要这一行！）
@Select("""
    SELECT DISTINCT u.* 
    FROM user u 
    LEFT JOIN address a ON u.id = a.user_id 
    ${ew.customSqlSegment}
    """)
List<User> queryUserWithAddress(@Param(Constants.WRAPPER) Wrapper<User> wrapper);
```

注意：用了 LambdaQueryWrapper 后，你甚至可以直接写别的表的字段（如 Address::getCity），MyBatis-Plus 会自动解析列名！（前提是实体类加了 

@TableName

 和 

@TableField

）

**终极结论（这段笔记真正的含金量）**

这页笔记教给你的不是“如何做多表联查”，而是：

“在 MyBatis-Plus 项目里，遇到任何多表查询，永远不要去写 XML 的 <where>、<if> 标签！
直接手写 SELECT ... FROM ... JOIN ... ${ew.customSqlSegment}
然后把 LambdaQueryWrapper 当参数传进去就完事了！”这才是 2025 年 MyBatis-Plus 多表查询的标准答案、主流写法、面试最爱考的点。记住这张图你就无敌了：

```text
多表查询 = 手写 SELECT + FROM + JOIN 
        + 一个 ${ew.customSqlSegment}
        + 传入 LambdaQueryWrapper 拼 WHERE
```

学会了这个，你再也不用为“MyBatis-Plus 不会联查”而烦恼
—— 实际上，它比原生 MyBatis 写联查还优雅 10 倍！

## 3.Service接口

### 3.1.CRUD

### 3.2.基本用法

### 3.3.Lambda

### 3.4.批量新增

