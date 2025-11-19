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

