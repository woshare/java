# Mybaits 学习

## idea导入mybatis
>1，git clone
>2，idea open
>3，config set maven project
>4，修改jdk 为1.8
>5，可以maven 打包和运行test下的文件

## 看官方文档，看源码
>1，抓主放次，梳理核心流程，整体到细节
>2，源码调试，核心数据结构和关键类
### JDBC操作数据库一般步骤
>1，注册数据库驱动类，明确指定数据库 URL 地址、数据库用户名、密码等连接信息。 
>2，通过 DriverManager 打开数据库连接
>3，通过数据库连接创建 Statement 对象
>4，通过 Statement 对象执行 SQL 语句，得到 ResultSet 对象。 
>5，通过 ResultSet 读取数据，并将数据转换成 JavaBean 对象。 
>6，关闭 ResultSet、 Statement 对象以及数据库连接，释放相关资源

>PS:ORM (Object Relational Mapping，对象－关系映射)框架是为了解决减少重复步骤，因为每次操作数据库都需要1，2，3，4，6步骤

>SAX 是基于事件模型的 XML 解析方式，它并不需要将整个 XML 文档加载到内存中，而 只需将 XML 文档的一部分加载到内存中，即可开始解析，在处理过程中井不会在内存中记录 XML 中的数据，所以占用的资源比较小

### 整体框架
![Alt text](./mybaits-struct.png "整体框架")

![Alt text](./mybatis-struct.png "整体框架")

![Alt text](./excutor-cache-based.png "整体框架")

>SqlSessionFactory 有两个实现类，一个是 SqlSessionManager 类，一个是 DefaultSqlSessionFactory 类

>DefaultSqlSessionFactory : SqlSessionFactory 的默认实现类，是真正生产会话的工厂类，这个类的实例的生命周期是全局的，它只会在首次调用时生成一个实例（单例模式），就一直存在直到服务器关闭。

>SqlSessionManager ： 已被废弃，原因大概是: SqlSessionManager 中需要维护一个自己的线程池，而使用MyBatis 更多的是要与 Spring 进行集成，并不会单独使用，所以维护自己的 ThreadLocal 并没有什么意义，所以 SqlSessionManager 已经不再使用。

### 初始化阶段-》代理阶段-》数据读写阶段
![Alt text](./mybaits-workflow.png "三大核心流程")

![Alt text](./mybatis-config.png "config类")

#### connection

>1，java.sql.Connection [Interface] -> com.mysql.jdbc.MySQLConnection [Interface] -> com.mysql.jdbc.ConnectionImpl [Class]
>2，在jdk里没有mysql.jdbc connection的实现，需要额外看mysql-connector-java这个项目源码

#### Executor的三个重要组件
>1，StatementHandler：它的作用是使用数据库的Statement或PrepareStatement执行操作，启承上启下作用；
>2，ParameterHandler：对预编译的SQL语句进行参数设置
>3，ResultSetHandler：对数据库返回的结果集（ResultSet）进行封装，返回用户指定的实体类型

### SqlSession->DefaultSqlSession->excutor
![Alt text](./mybatis-sql-excutor.png "sql-excutor")

>4，mybatis大概的一个结构或过程：SqlSessionFactory->SqlSession->DefaultSqlSession{configuration,executor,autocommit,dirty,cursorlist}->executor{transaction,exectuor(this),configuration,deferredLoads,localCache,localOutout,paramterCache}->transaction{connection,autocommit,level,datasource}

>5,connection--->statement--->execute

>6，查询的一个大概过程：query{statementHandle->statement.execute()->connection.execute()}

![Alt text](./mybaits-sql-select.png "sql-select")

* [mybatis源碼簡要介紹](https://juejin.im/post/5c04e6325188252e4c2e94ca)

#### interface没有具体实现，在别的类里拓展实现的，如何用interface调用到了具体的方法，如果有多个类拓展实现呢？
>1，和父类是一样的道理，就有类B拓展实现（implements）了接口A，我们可以把A=B，用A去调用B中的实现的A的方法

## datasource


### DataSource什么时候创建Connection对象
>1,MyBatis创建了DataSource实例后，会将其放到Configuration对象内的Environment对象中， 供以后使用。当我们需要创建SqlSession对象并需要执行SQL语句时，这时候MyBatis才会去调用dataSource对象来创建java.sql.Connection对象。也就是说，java.sql.Connection对象的创建一直延迟到执行SQL语句的时候。

![Alt text](./UnpooledDataSource.png "UnpooledDataSource")

* [mybatis数据源与链接池](https://yq.aliyun.com/articles/641401?spm=a2c4e.11153940.0.0.430d396fCekMrD)


## 设计模式

* [设计模式-可以看看](http://c.biancheng.net/view/1351.html)
* [设计模式-可以看看](https://www.liaoxuefeng.com/wiki/1252599548343744/1281319214514210)

### 设计模式六原则
>1，单一职责原则：不要存在多于一个导致类变更的原因，简单来说，一个类只负责唯一项职责。 
>2，里氏替换原则：如果对每一个类型为Tl的对象 tl，都有类型为 T2 的对象 t2，使得以 Tl定义的所有程序 P 在所有的对象 tl 都代换成 t2 时，程序 P的行为没有发生变化，那么类型T2是类型Tl的子类型。遵守里氏替换则，可以帮助我们设计出更为合理的继承体系。 
>3，依赖倒置原则：系统的高层模块不应该依赖低层模块的具体实现，二者都应该依赖其抽象类或接口，抽象接口不应该依赖具体实现类，而具体实现类应该于依赖抽象。简单来说，我们要 **面向接口编程**。当需求发生变化时对外接口不变，只要提供新的实现类即可。 
>4，接口隔离原则：一个类对另一个类的依赖应该建立在最小的接口上。简单来说，我们在设计接口时，不要设计出庞大膝肿的接口，因为实现这种接口时需要实现很多不必要的方法。 我们要尽量设计出功能单一的接口，这样也能保证实现类的职责单一。 
>5，迪米特法则： 一个对象应该对其他对象保持最少的了解。 简单来说，就是要求我们减低类间耦合。 
>6，开放－封闭原则：程序要对扩展开放，对修改关闭。简单来说，当需求发生变化时，我们可以通过添加新的模块满足新需求，而不是通过修改原来的实现代码来满足新需求。 
>7，在这六条原则中，开放－封闭原则是最基础的原则，也是其他原则以及后文介绍的所有设计模式的最终目标。

### 工厂模式Factory

### 建造者模式Builder
>1,cache模块就是用的建造者模式设计的
>2，MapperBuilderAssistant.useNewCache（）这个方法里，演示了如何理解和使用建造者模式


### 适配器模式Adapter
>1,主要目的是解决由于接口不能兼容而导致类无法使用的问题，适配器模式会将需要适配的类转换成调用者能够使用的目标接口。

![Alt text](./DesigModule-Adapter.png  "适配器设计模式")

### 代理模式 proxy：静态代理和动态代理

### 装饰器模式

### 责任链模式

### 单例模式

### 策略模式

### 工厂模式

### 观察者模式

### mybatis模块介绍
|名称|作用|
| :----: | :----: |
|SqlSession|作为MyBatis工作的主要顶层API，表示和数据库交互的会话，完成必要数据库增删改查功能|
|Executor|MyBatis执行器,是MyBatis调度的核心，负责SQL语句的生成和查询缓存的维护|
|StatementHandler|封装了JDBC Statement操作，负责对JDBC statement的操作，如设置参数、将Statement结果集转换成List集合|
|ParameterHandler|负责对用户传递的参数转换成JDBC Statement 所需要的参数|
|ResultSetHandler|负责将JDBC返回的ResultSet结果集对象转换成List类型的集合|
|TypeHandler|负责java数据类型和jdbc数据类型之间的映射和转换|
|MappedStatement|MappedStatement维护了一条select|
|SqlSource|	负责根据用户传递的parameterObject，动态地生成SQL语句，将信息封装到BoundSql对象中，并返回|
|BoundSql|表示动态生成的SQL语句以及相应的参数信息|
|Configuration|MyBatis所有的配置信息都维持在Configuration对象之中|


## 插件

### 分页插件
>1，分页插件主要是通过拦截器修改当前线程指定的sql，添加limit参数
>2，pagehelper是通过ThreadLocal实现的分页标识传递（隐式传参），如果处理清理不当，可能导致线程污染
>3，因为有可能导致线程污染，PageHelper.start(offset,size)函数应该和mapper函数紧挨着，避免两者之间出异常，导致线程污染

### 延迟加载
>1，减轻DB服务器的压力,因为我们延迟加载只有在用到需要的数据才会执行查询操作
>2，在关联查询时，我们可能先只需要主表数据，而关联表的数据，等需要的时候，再查询数据库
>3，这个需要，这个时机怎么控制和实现    
```
<resultMap id="BaseResultMap" type="com.redstar.basemapper.pojo.User">
        <id column="id" jdbcType="VARCHAR" property="id"/>
        <result column="name" jdbcType="VARCHAR" property="name"/>
        <result column="age" jdbcType="INTEGER" property="age"/>
        <result column="role_id" jdbcType="INTEGER" property="roleId"/>
</resultMap>
<resultMap id="userRoleMapSelect" type="com.redstar.basemapper.pojo.UserVo">
        <association property="user" resultMap="BaseResultMap"/>
        <association property="role" fetchType="lazy" column="{id=role_id}"  //lazy:延迟加载
                     select="com.redstar.basemapper.dao.RoleMapper.getRoleById"/>//引用另外一个查询方法
</resultMap>
<sql id="Base_Column_List">
    id, `name`, age, role_id
</sql>
<select id="getUserVo" resultMap="userRoleMapSelect">
      select * from user where id=#{userId}
</select>
```
* [延迟加载举例](https://juejin.im/post/5c3d518df265da614171cabe)
>ps：许多对延迟加载原理不太熟悉的朋友会经常遇到一些莫名其妙的问题：有些时候延迟加载
可以得到数据，有些时候延迟加载就会报错，为什么会出现这种情况呢？
MyBatis 延迟加载是通过动态代理实现的，当调用配直为延迟加载的属性方法时， 动态代
理的操作会被触发，这些额外的操作就是通过 MyBatis 的 SqlSessio口去执行嵌套 SQL 的 。
由于在和某些框架集成时， SqlSession 的生命周期交给了框架来管理，因此当对象超出
SqlSession 生命周期调用时，会由于链接关闭等问题而抛出异常 。 在和 Spring 集成时，要
确保只能在 Service 层调用延迟加载的属性 。 当结果从 Service 层返回至 Controller 层时， 如果
获取延迟加载的属性值，会因为 SqlSessio口已经关闭而抛出异常 


## JDK sql
|接口或类|	作用|
| :----: | :----: |
|java.sql|	所有与JDBC访问数据库相关的接口和类|
|javax.sql|	数据库扩展包，提供数据库额外的功能。如：连接池|
|数据库的驱动|	由各大数据库厂商提供，需要额外去下载，是对JDBC接口实现的类，像我们常用的mysql，在pom中引用依赖mysql-connector-java，具体实现也在这个包里|

>1，JDBC规范定义接口，具体的实现由各大数据库厂商来实现

![Alt text](./jdbc-interface.png "JDBC接口")

### JDBC规范定义接口
>1，具体的实现由各大数据库厂商来实现，通过数据库厂商提供的DriverManager类，使用接口以多态的形式调用对应的实现类

|接口或类|	作用|
| :----: | :----: |
|DriverManager类|管理和注册数据库驱动；得到数据库连接对象Connection的实现类|
|Connection接口|一个连接对象，可用于创建Statement和PreparedStatement对象|
|Statement接口|	一个SOL语句对象，用于将SQL语句发送给数据库服务器|
|PreparedStatement接口|	一个SQL语句对象，是Statement的子接口|
|ResultSet接口|	用于封装数据库查询的结果集，返回给客户端Java程序|

### 加载和注册驱动

### 事务提交
>1，mysql-connector：ConnectionImpl.setAutoCommit(){autoCommitFlag ? "SET autocommit=1" : "SET autocommit=0"}

### autocommit与显性事务的关系
>1,autocommit=1，自动提交;autocommit=0,手动提交；设置autocommit=0，执行语句，需要再执行commit语句，而autocommit=1，则是隐式自动提交了
>2,使用START TRANSACTION，自动提交将保持禁用状态，直到你使用COMMIT或ROLLBACK结束事务。 自动提交模式然后恢复到之前的状态（如果start transaction 前 autocommit = 1，则完成本次事务后 autocommit 还是 1。如果 start transaction 前 autocommit = 0，则完成本次事务后 autocommit 还是 0）

```
show session variables like 'autocommit';//会话变量，只对当前实例有效
show global variables like 'autocommit';//区分全局还是会话变量

session 1:
1:set autocommit =0;
2:update table1 set col=1 where id=1;
3:commit;

session 2:
1:select col from table1 where id=1;

ps:执行1.1,1.2，再执行2.1，是看不到1.2的操作的，只有再执行1.3之后，2.1才看看到修改
```

### ThreadLocal和Thread,ThreadLocal可以用来隐士传参数，线程间数据隔离   
>1，ThreadLocal内存泄漏的根源是：由于ThreadLocalMap的生命周期跟Thread一样长，如果没有手动删除对应key就会导致内存泄漏，而不是因为弱引用。想要避免内存泄露就要手动remove()掉

![Alt text](./ThreadLocal-Thread.png "ThreadLocal-Thread对象关系引用")


##

* [mybatis-不错](https://github.com/nero520/mybatis/tree/master/search-mybatis-mybatis/doc)


## resultSetHandle

> DefaultObjectFactory 这个会根据mapper returnType，创建出对象，


## 缓存：两级

>1，每个SqlSession中持有了Executor，每个Executor中有一个LocalCache，存储了一级缓存数据，且还区分SESSION（会话），STATMENT（sql语句级，如果配置缓存作用域localCacheScope是STATEMENT则会每次清空缓存）   
>2，使用一级缓存的时候作用在同一个SqlSession下，因为缓存不能跨会话共享，不同的会话之间对于相同的数据可能有不一样的缓存。当在有多个会话或者分布式环境下，可能会存在脏数据的问题。那这种问题如何解决呢？可以将一级缓存级别设置为Statement的级别    
>3，由于二级缓存是从MappedStatement中获取，是存在于全局配置，如果被多个CachingExecutor获取到，则一定会出现线程安全问题导致脏读，所以MyBatis为解决这个问题，在查询的过程中提供了TransactionalCacheManager作为事务缓存管理器   
>4，二级缓存构建在一级缓存之上，一级缓存是和SqlSession绑定，而二级缓存是和Mapper具体的命名空间绑定，二级缓存是全局性的，一个Mapper中有一个Cache，相同的Mapper中多个不同的MappedStatement共用一个Cache      
>5，二级缓存-》一级缓存-》数据库：CachingExecutor#query()       
>6，一级缓存，在同一的sqlSession作用范围下，如果中间操作sqlSession执行了commit操作，其实也就是（删除，更新，新增）那么会清除sqlSession的缓存，保障缓存中拿到的数据一定是最新的，避免脏读。但是是局限在同一sqlSession下，当sqlSession2更新了数据，sqlSession1就可能还会是脏读  
>7，二级缓存是基于namespace的，在相同的namespace下不同的sqlSession更新数据库并提交事务之后，sqlSession1会导致sqlSession2刷新二级缓存，sqlSession不会存在脏读问题，但是要是不同的namespace下，则sqlSession1的更新操作事务提交，不会导致sqlSession2的刷新缓存，会存在脏读              
>装饰链：SynchronizedCache -> LoggingCache -> SerializedCache -> LruCache -> PerpetualCache   

![](excutor-local-cache.awebp "")

![](level1-cache-level2-cache-sqlSession.png "")

* [mybatis缓存-美团](https://tech.meituan.com/2018/01/19/mybatis-cache.html)


## 写代码不仅要写的人很喜欢（逻辑可读性），机器也很喜欢（高性能）

![](mybatis-debug-column1.png "")

![](mybatis-debug-query-eof.png "")



MysqlIO

ResultSetInternalMethods sqlQueryDirect


 //发送查询sql packet string buff
	    	Buffer resultPacket = sendCommand(MysqlDefs.QUERY, null, queryPacket,
	    			false, null, 0);


	ResultSetInternalMethods rs = readAllResults(callingStatement, maxRows, resultSetType,
	    			resultSetConcurrency, streamResults, catalog, resultPacket,
	    			false, -1L, cachedMetadata);
					
					
					
rowBytes[i] = rowPacket.readBytes(StringSelfDataType.STRING_LENENC);					
NativePacketPayload rowPacket ：					
01 31 04 75 74 66 38 04     . 1 . u t f 8 .
75 74 66 38 04 75 74 66     u t f 8 . u t f
38 06 6c 61 74 69 6e 31     8 . l a t i n 1
11 6c 61 74 69 6e 31 5f     . l a t i n 1 _
73 77 65 64 69 73 68 5f     s w e d i s h _
63 69 0f 75 74 66 38 5f     c i . u t f 8 _
67 65 6e 65 72 61 6c 5f     g e n e r a l _
63 69 00 05 32 38 38 30     c i . . 2 8 8 0
30 03 47 50 4c 01 30 08     0 . G P L . 0 .
31 30 34 38 35 37 36 30     1 0 4 8 5 7 6 0
02 36 30 01 31 07 31 30     . 6 0 . 1 . 1 0
34 38 35 37 36 03 4f 46     4 8 5 7 6 . O F
46 16 4e 4f 5f 45 4e 47     F . N O _ E N G
49 4e 45 5f 53 55 42 53     I N E _ S U B S
54 49 54 55 54 49 4f 4e     T I T U T I O N
03 43 53 54 06 53 59 53     . C S T . S Y S
54 45 4d 0f 52 45 50 45     T E M . R E P E
41 54 41 42 4c 45 2d 52     A T A B L E - R
45 41 44 05 32 38 38 30     E A D . 2 8 8 0
30                          0
					
					
					
					AbstractResultsetRow
					
					
					
					0，先 this.session.execSQL(null, autoCommitFlag ? "SET autocommit=1" : "SET autocommit=0", -1, null, false, this.nullStatementResultSetFactory,
                            null, false);
						先做了一些前期的基础工作
					1，先获取column的信息
					
```					
03 73 65 6c 65 63 74 20     . s e l e c t  
2a 20 66 72 6f 6d 20 65     *   f r o m   e
64 75 5f 61 63 63 6f 75     d u _ a c c o u
6e 74 5f 69 6e 66 6f 20     n t _ i n f o  
77 68 65 72 65 20 75 69     w h e r e   u i
64 20 3d 20 27 34 31 31     d   =   ' 4 1 1
30 32 33 37 27 20 20 20     0 2 3 7 '      
6f 72 64 65 72 20 62 79     o r d e r   b y
20 69 64 20 61 73 63 20       i d   a s c  
6c 69 6d 69 74 20 31        l i m i t   1
```


## 数据库列值如何赋值给结果实例
```
DefaultResultSetHandler

private Object getRowValue(ResultSetWrapper rsw, ResultMap resultMap, String columnPrefix) throws SQLException {
    final ResultLoaderMap lazyLoader = new ResultLoaderMap();
    Object rowValue = createResultObject(rsw, resultMap, lazyLoader, columnPrefix);
    if (rowValue != null && !hasTypeHandlerForResultObject(rsw, resultMap.getType())) {
      final MetaObject metaObject = configuration.newMetaObject(rowValue);
      boolean foundValues = this.useConstructorMappings;
      if (shouldApplyAutomaticMappings(resultMap, false)) {
        foundValues = applyAutomaticMappings(rsw, resultMap, metaObject, columnPrefix) || foundValues;
      }
      foundValues = applyPropertyMappings(rsw, resultMap, metaObject, lazyLoader, columnPrefix) || foundValues;
      foundValues = lazyLoader.size() > 0 || foundValues;
      rowValue = foundValues || configuration.isReturnInstanceForEmptyRow() ? rowValue : null;
    }
    return rowValue;
  }
  
    private Object createResultObject(ResultSetWrapper rsw, ResultMap resultMap, ResultLoaderMap lazyLoader, String columnPrefix) throws SQLException {
    this.useConstructorMappings = false; // reset previous mapping result
    final List<Class<?>> constructorArgTypes = new ArrayList<>();
    final List<Object> constructorArgs = new ArrayList<>();
    Object resultObject = createResultObject(rsw, resultMap, constructorArgTypes, constructorArgs, columnPrefix);
    if (resultObject != null && !hasTypeHandlerForResultObject(rsw, resultMap.getType())) {
      final List<ResultMapping> propertyMappings = resultMap.getPropertyResultMappings();
      for (ResultMapping propertyMapping : propertyMappings) {
        // issue gcode #109 && issue #149
        if (propertyMapping.getNestedQueryId() != null && propertyMapping.isLazy()) {
          resultObject = configuration.getProxyFactory().createProxy(resultObject, lazyLoader, configuration, objectFactory, constructorArgTypes, constructorArgs);
          break;
        }
      }
    }
    this.useConstructorMappings = resultObject != null && !constructorArgTypes.isEmpty(); // set current mapping result
    return resultObject;
  }
  
   private Object createResultObject(ResultSetWrapper rsw, ResultMap resultMap, List<Class<?>> constructorArgTypes, List<Object> constructorArgs, String columnPrefix)
      throws SQLException {
    final Class<?> resultType = resultMap.getType();
    final MetaClass metaType = MetaClass.forClass(resultType, reflectorFactory);
    final List<ResultMapping> constructorMappings = resultMap.getConstructorResultMappings();
    if (hasTypeHandlerForResultObject(rsw, resultType)) {
      return createPrimitiveResultObject(rsw, resultMap, columnPrefix);
    } else if (!constructorMappings.isEmpty()) {
      return createParameterizedResultObject(rsw, resultType, constructorMappings, constructorArgTypes, constructorArgs, columnPrefix);
    } else if (resultType.isInterface() || metaType.hasDefaultConstructor()) {
      return objectFactory.create(resultType); //根据类名，创建对象
    } else if (shouldApplyAutomaticMappings(resultMap, false)) {
      return createByConstructorSignature(rsw, resultType, constructorArgTypes, constructorArgs);
    }
    throw new ExecutorException("Do not know how to create an instance of " + resultType);
  }

private boolean applyAutomaticMappings(ResultSetWrapper rsw, ResultMap resultMap, MetaObject metaObject, String columnPrefix) throws SQLException {
    List<UnMappedColumnAutoMapping> autoMapping = createAutomaticMappings(rsw, resultMap, metaObject, columnPrefix);
    boolean foundValues = false;
    if (!autoMapping.isEmpty()) {
      for (UnMappedColumnAutoMapping mapping : autoMapping) {
        final Object value = mapping.typeHandler.getResult(rsw.getResultSet(), mapping.column);
        if (value != null) {
          foundValues = true;
        }
        if (value != null || (configuration.isCallSettersOnNulls() && !mapping.primitive)) {
          // gcode issue #377, call setter on nulls (value is not 'found')
          metaObject.setValue(mapping.property, value);
        }
      }
    }
    return foundValues;
  }


```

## mybatis查询逻辑，解析mysql应答数据，生成mapper returnType类实例，并对应列和类属性赋值

>1，DefaultSqlSession.selectOne(String statement, Object parameter)->selectList(statement, parameter)->CachingExecutor.query(ms, wrapCollection(parameter), rowBounds, handler)->BaseExecutor.query->queryFromDatabase->SimpleExecutor.doQuery->RoutingStatmentHandler.query->PreparedStatementHandler.query->ps.execute()[执行sql]->DefaultResultSetHandler.handleResultSets->handleRowValues->handleRowValuesForSimpleResultMap->getRowValue{Object rowValue = createResultObject(rsw, resultMap, lazyLoader, columnPrefix); foundValues = applyAutomaticMappings(rsw, resultMap, metaObject, columnPrefix) || foundValues;}->applyAutomaticMappings{metaObject.setValue(mapping.property, value);//metaObject是由createResultObject方法根据mapper的returnType类生成的，mapping.property是mapper的列对应的返回类对象的属性名，value是通过解析mysql返回的buffer的十六进制数据得到的对应列的数值，此处是把数据库返回的列的值，设置给返回类对象的相应属性}

>2，DefaultResultSetHandler.createResultObject
```
    final Class<?> resultType = resultMap.getType();
    final MetaClass metaType = MetaClass.forClass(resultType, reflectorFactory);
    objectFactory.create(resultType);
```

![](mybatis-get-row-value-set-returnType-value.png "")

![](mybatis-DefaultSqlSession-selectOne.png  "")

![](mybatis-applyAutomaticMappings-mapping-property.png "")

## column，value，property
>1，column->valueIndex-十六进制value->解析成正常数据value
>2，column->property->设置返回对象属性的值
>3，column是数据表列名信息，property是DTO类属性名

![](mybatis-autoMapping-column-property.png "")

![](mybatis-DefaultColumnDefinition-columnLabelToIndex-for-value.png "")

## 发送select的mysql packet
```					
03 73 65 6c 65 63 74 20     . s e l e c t  
2a 20 66 72 6f 6d 20 65     *   f r o m   e
64 75 5f 61 63 63 6f 75     d u _ a c c o u
6e 74 5f 69 6e 66 6f 20     n t _ i n f o  
77 68 65 72 65 20 75 69     w h e r e   u i
64 20 3d 20 27 34 31 31     d   =   ' 4 1 1
30 32 33 37 27 20 20 20     0 2 3 7 '      
6f 72 64 65 72 20 62 79     o r d e r   b y
20 69 64 20 61 73 63 20       i d   a s c  
6c 69 6d 69 74 20 31        l i m i t   1
```

![](mybatis-select-mysql-packet.png "应答0x15，其实是列数21")


![](mybatis-set-autocommit-1-mysql-packet.png "开启事务隐式自动提交")

![](mybatis-set-sql-mode-mysql-packet.png "")

## #{}防止sql注入
动态 SQL 是 mybatis 的强大特性之一，也是它优于其他 ORM 框架的一个重要原因。mybatis 在对 sql 语句进行预编译之前，会对 sql 进行动态解析，解析为一个 BoundSql 对象，也是在此处对动态 SQL 进行处理的。

在动态 SQL 解析阶段， #{ } 和 ${ } 会有不同的表现：

>1，#{ } 解析为一个 JDBC 预编译语句（prepared statement）的参数标记符。
>2，${ } 仅仅为一个纯碎的 string 替换，在动态 SQL 解析阶段将会进行变量替换  

例如，sqlMap 中如下的 sql 语句
```
select * from user where name = #{name};
```
解析为：
```
select * from user where name = ?;
```
一个 #{ } 被解析为一个参数占位符 ? 。

而，

${ } 仅仅为一个纯碎的 string 替换，在动态 SQL 解析阶段将会进行变量替换

例如，sqlMap 中如下的 sql
```
select * from user where name = '${name}';
```
当我们传递的参数为 "ruhua" 时，上述 sql 的解析为：
```
select * from user where name = "ruhua";
```
预编译之前的 SQL 语句已经不包含变量 name 了。

综上所得， ${ } 的变量的替换阶段是在动态 SQL 解析阶段，而 #{ }的变量的替换是在 DBMS 中。

![](mybatis-CachingExecutor-query-boundSql-CacheKey.png "#{}和${}预编译")