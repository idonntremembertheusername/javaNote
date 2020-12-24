#### XML 标签

`select`、`insert`、`update`、`delete` 标签分别对应查询、添加、更新、删除操作。

`parameterType` 属性表示参数的数据类型，包括基本数据类型和对应的包装类型、String 和 Java Bean 类型，当有多个参数时可以使用 `#{argn}` 的形式表示第 n 个参数。除了基本数据类型都要以全限定类名的形式指定参数类型。

`resultType` 表示返回的结果类型，包括基本数据类型和对应的包装类型、String 和 Java Bean 类型。还可以使用把返回结果封装为复杂类型的 `resultMap` 。

------

#### 一级缓存

一级缓存是 SqlSession 级别，默认开启。

SqlSession 对象中有一个 HashMap 缓存数据，不同 SqlSession 间缓存数据互不影响。同一个 SqlSession 中执行两次相同的 SQL 语句时，第一次执行完毕会将结果保存在缓存中，第二次查询直接从缓存中获取。

如果 SqlSession 执行了 DML 操作（insert、update、delete），Mybatis 必须将缓存清空保证数据有效性。 

------

#### 二级缓存

二级缓存是Mapper 级别，默认关闭。

相比于一级缓存，缓存范围更大，多个 SqlSession 可以共用二级缓存，作用域是 Mapper 的同一个 namespace，不同 SqlSession 两次执行相同的 namespace 下的 SQL 语句，参数也相等，则第一次执行成功后会将数据保存在二级缓存中。

**开启mybatis的二级缓存：**

在全局配置文件中配置 `<setting name="cacheEnabled" value="true"/>` ，并在对应的映射文件`mapper.xml`中配置 `<cache/>` 标签。

------



#### mybatis中的#号与$符号的区别

`#{}`: 解析为一个 JDBC 预编译语句（prepared statement）的参数标记符，一个 #{ } 被解析为一个参数占位符 。

`${}`: 仅仅为一个纯粹的 string 替换，在动态 SQL 解析阶段将会进行变量替换。

> #{}占位符，用于参数的传递
>
>  ${}用于sql语句拼接
>

  #{} 预编译，防止sql注入 不需要关心数据类型， 一个参数时，任意参数名都可以接受参数。

  ${}非预编译 直接字符串的拼接 不可以防止sql注入 需要关系数据类型一个参数时，必须是value。