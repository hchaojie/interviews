## 目录

- 使用mybatis有什么好处？
- mybatis的工作原理知道吗？（难）
- 什么叫mybatis的接口绑定？有哪几种方式？分别什么时候用？
- 查询结果是如何封装成对象的？
- mybatis有哪些动态sql标签？
- 类的属性名和表的字段名不一致，如何处理？
- 每个sql文件都有一个dao或者mapper接口，这个接口文件的工作原理是什么？
- mapper接口里面能有重载的方法吗？
- #{}和${}的区别？什么时候用${}?
- 模糊查询如何做？
- mybatis里面如何批量插入数据？
- mybatis如何获取自动生成的主键？
- mapper映射文件里面如何传递多个参数？
- mapper接口和mapper映射文件需要如何对应？
- 一对一，一对多的关联查询，如何作封装查询结果？
- mybatis是如何进行分页的？
- mybatis的缓存机制？
- mybatis有什么不好的地方？

## 使用mybatis有什么好处？

1. 可以将sql语句和代码分离，便于修改。
2. 可以直接将查询结果映射成pojo。
3. 主要是简化了jdbc的使用，省去了繁琐的操作。

## mybatis的工作原理知道吗？（难）

主要是用了JDK的动态代理技术，生成mapper接口的代理对象，来执行数据库的查询。然后在查询完成后，使用反射将结果封装成pojo。

它启动时，会加载所有的mapper映射文件，里面所有sql语句，会被封装成MappedStatement对象。

在查询时，它使用SessionFactory（工厂模式），去创建Session（会话），一个Session代表一个数据库连接。然后使用Excecutor（执行器）来执行sql语句。

执行完成之后，使用ResultSetHandler来进行结果的封装。

## 什么叫mybatis的接口绑定？

接口绑定：就是把mapper接口里面的方法，映射到某个sql语句。

分两种方式：1）使用注解、2）在xml里面写对应的sql语句。

简单查询可以直接使用@Select, @Delete, @Update注解。

复杂的关联查询通常在xml里面写sql语句和resultMap结果映射。

## 查询结果是如何封装成对象的？

可以用ResultType, 也可以用ResultMap。列名需要和对象的属性一一对应。列如果是下划线形式，可以自动映射成java里面驼峰形式的属性。除此之外，如果列和属性不匹配，需要使用别名查询。比如 `select username name from xxx`

封装成对象时，mybatis使用反射创建pojo，设置对应的属性值。

## mybatis有哪些动态sql标签？

动态sql语句，可以用来作条件查询、循环等。

常见的有<where> <set> <if> <foreach> 等。

## 类的属性名和表的字段名不一致，如何处理？

使用别名查询

使用ResultMap手动映射。

<result column="username" property="name" />

## 每个sql文件都有一个dao或者mapper接口，这个接口的工作原理是什么？

接口的工作原理为JDK动态代理，运行时会为mapper接口生成proxy代理对象，代理对象会拦截接口方法的调用，去执行对应的sql语句。

JDK动态代理和cglib代替代理的区别？

JDK动态代理是基于接口的，cglib是基于继承的。

## mapper接口里面能有重载的方法吗？

不能重载。因为找接口方法对应的sql语句，是用的接口全限定名 + 方法名称。跟接口参数无关。

## {}和${}有什么区别？

mybatis在处理#{}时，会将sql中的#{}替换为?号，调用jdbc里面的PreparedStatement里面的set方法来设置查询参数。这个叫jdbc的参数化查询，而PreparedStatement会对sql语句做编译预处理，过滤非法的sql语句，防止sql注入。

mybatis处理${}时，会直接把${}替换成变量值。因此不会经过jdbc的编译预处理。有sql注入的风险。

## 模糊查询如何做？

1. 可以在java里面拼接通配符，通过#{}引用参数

   利用了jdbc的参数化查询，可以防止sql注入。

2. 也可以在sql语句里面拼接字符串，使用${}引用参数。  '%${value}%'

   有sql注入的风险。

## mybatis里面如何批量插入数据？

使用foreach循环， 生成一个`insert into xxx () values (), (), ()`形式的sql语句。

## mybatis如何获取自动生成的主键？

1. 在<insert>语句里面使用<selectKey>标签

2. 也可以在配置文件里面设置useGeneratedKeys为true

## mapper映射文件里面如何传递多个参数？

1. 使用@Param注解
2. 也可以在sql语句里面使用#{0}, #{1}来获取（可读性差，几乎不用）

## mapper接口和mapper映射文件需要如何对应？

1. xml里面的namespace对应mapper接口的全限定名（包名+接口名）
2. xml里面sql语句的id对应方法名
3. xml里面的resultType对应方法返回值
4. xml里面的parameterType对应方法参数

## 一对一，一对多的关联查询，如何作封装查询结果？

一对一查询，可以定义一个dto，属性跟返回的查询字段一一对应即可，使用ResultType映射。

也可以定义关联对象，比如每个课题有个老师，就可以在Course类里面定义一个Teacher属性。使用ResultMap映射查询结果，ResultMap里面使用association映射关联的表。字段相同的话，需要给重名的列取别名。

一对多查询，需要顶一个嵌套的关联对象，比如一个用户有多个角色，可以在User类里面定义一个List<Role>属性。使用ResultMap映射查询结果，ResultMap里面使用collections映射关联的表。

## mybatis是如何进行分页的？

查询语句后面加上limit就可以分页。（当然前提是使用mysql数据库，oracle是没有limit关键字的）

当然，项目大了的话，每个需要分页的地方都写limit也不科学。mybatis提供了插件机制，可以拦截所有的sql语句执行，添加额外的功能。

插件是不需要我们写的，这么通用的需求，已经有人给我们做好了。可以使用PageHelper这个分页插件，也可以使用mybatis-plus这个开源库。

## mybatis的缓存机制？

mybatis里面的缓存分为一级缓存和二级缓存。

一级缓存默认开启，也叫session级别的缓存，二级缓存需要额外配置，一般还需要结合第三方缓存框架，比如ehcache或者spring cache。

一级缓存：session级别的缓存，只在一次事务里面有效。表现就是，如果你在一次事务里面，多次执行同样的sql语句，mybatis只发送一次查询。

二级缓存：一般是缓存应用级别的东西。

比如京东首页的商品分类，假设分类信息都存在数据库里，这样每个用户访问首页的时候，都需要从数据库里面查询一遍。但是分类信息一般变化比较小，因此我们可以在应用启动的时候，把分类信息预先查询出来，缓存到内存。后面所有的用户访问首页的时候，都不需要查询数据，而是从缓存里面取，减少了数据库的压力。

为什么需要二级缓存？因为一级缓存生命周期太短，一次请求完成，session关闭后，缓存就被清空了。前一个连接的查询结果，不能被后续查询重用。

## mybatis有什么不好的地方？

写sql语句的工作量大。尤其是字段多，关联的表多的情况下。

sql语句依赖底层的数据库，不同的数据库语法有差别，因此可移植性较差。



