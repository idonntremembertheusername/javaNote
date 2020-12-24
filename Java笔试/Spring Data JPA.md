#### ORM

Object-Relational Mapping ，表示对象关系映射，映射的不只是对象的值还有对象之间的关系，通过 ORM 就可以把对象映射到关系型数据库中。操作实体类就相当于操作数据库表，可以不再重点关注 SQL 语句。

------

#### JPA 的使用

只需要持久层接口继承 JpaRepository 即可，泛型参数列表中第一个参数是实体类类型，第二个参数是主键类型。

运行时通过 `JdkDynamicAopProxy` 的 `invoke` 方法创建了一个动态代理对象 `SimpleJpaRepository`，`SimpleJpaRepository` 中封装了 JPA 的操作，通过 hibernate 完成数据库操作。

------

#### 实体类相关注解

`@Entity`：表明当前类是一个实体类。

`@Table` ：关联实体类和数据库表。

`@Column` ：关联实体类属性和数据库表中字段。

`@Id` ：声明当前属性为数据库表主键对应的属性。

`@GeneratedValue`： 配置主键生成策略。

`@OneToMany` ：配置一对多关系，mappedBy 属性值为主表实体类在从表实体类中对应的属性名。

`@ManyToOne` ：配置多对一关系，targetEntity 属性值为主表对应实体类的字节码。

`@JoinColumn`：配置外键关系，name 属性值为外键名称，referencedColumnName 属性值为主表主键名称。