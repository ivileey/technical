# Mybatis

## 1.Mybatis底层实现

## 2.Mybatis工作流程

1. 读取核心配置文件，也就是xml文件，并返回`InputStream`流对象。

2. 根据`InputStream`流对象解析出`Configuration`对象，然后创建`SqlSessionFactory`工厂对象

   -Resources.getResourceAsStream(resource);//Resources是mybatis提供的一个加载资源文件的工具类，获取到自身的ClassLoadere对象，然后交给ClassLoader来加载，返回一个InputStream对象。

   - new sqlSessionFactoryBuilder().build(inputStream);//通过Document对象来解析，然后返回InputStream对象，然后交给XMLConfigBuilder构造成Configuration对象，然后交给build()方法构成成SqlSessionFactory。

3. 根据一系列属性从`SqlSessionFactory`工厂中创建`SqlSession`

   > SqlSession完全包含了面向数据库执行SQL命令所需的所有方法，可以通过SqlSession实例来直接执行已映射的SQL语句。

   调用自身的openSessionFromDataSource方法。

   1. getDefaultEecutorType()默认是SIMPLE。

   2. 构建步骤：Environment>>TransactionFactory+autoCommit+txlevel>>Transaction+ExecType>>

      Executor+Condiguration+autoCommit>>sqlSession

      ```
      private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {
          Transaction tx = null;
      
          DefaultSqlSession var8;
          try {
              Environment environment = this.configuration.getEnvironment();
              // 根据Configuration的Environment属性来创建事务工厂
              TransactionFactory transactionFactory = this.getTransactionFactoryFromEnvironment(environment);
              // 从事务工厂中创建事务，默认等级为null，autoCommit=false
              tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);
              // 创建执行器
              Executor executor = this.configuration.newExecutor(tx, execType);
              // 根据执行器创建返回对象 SqlSession
              var8 = new DefaultSqlSession(this.configuration, executor, autoCommit);
          } catch (Exception var12) {
              this.closeTransaction(tx);
              throw ExceptionFactory.wrapException("Error opening session.  Cause: " + var12, var12);
          } finally {
              ErrorContext.instance().reset();
          }
          return var8;
      }
      ```

      

4. 从`SqlSession`中调用`Executor`执行数据库操作&&生成具体SQL指令

   1. 在拿到SqlSession对象后，调用它的insert方法，会调用update方法。

   ```
   public int update(String statement, Object parameter) {
       int var4;
       try {
           this.dirty = true;
           MappedStatement ms = this.configuration.getMappedStatement(statement);
           // wrapCollection(parameter)判断 param对象是否是集合
           var4 = this.executor.update(ms, this.wrapCollection(parameter));
       } catch (Exception var8) {
           throw ExceptionFactory.wrapException("Error updating database.  Cause: " + var8, var8);
       } finally {
           ErrorContext.instance().reset();
       }
   
       return var4;
   }
   ```

   mappedStatement就是平时我们所说的sql映射对象，mappingxml的namespce和id信息就会存放为mappedStatements的key，对应的sql语句就是value。

   2. 然后调用BaseExecutor中的update方法，调用doUpdate()真正做执行操作的方法。

   ```
   public int doUpdate(MappedStatement ms, Object parameter) throws SQLException {
       Statement stmt = null;
   
       int var6;
       try {
           Configuration configuration = ms.getConfiguration();
           // 创建StatementHandler对象，从而创建Statement对象
           StatementHandler handler = configuration.newStatementHandler(this, ms, parameter, RowBounds.DEFAULT, (ResultHandler)null, (BoundSql)null);
           // 将sql语句和参数绑定并生成SQL指令
           stmt = this.prepareStatement(handler, ms.getStatementLog());
           var6 = handler.update(stmt);
       } finally {
           this.closeStatement(stmt);
       }
   
       return var6;
   }
   ```

   3. prepareStatement()方法，将sql拼接

   ```
   private Statement prepareStatement(StatementHandler handler, Log statementLog) throws SQLException {
       Connection connection = this.getConnection(statementLog);
       // 准备Statement
       Statement stmt = handler.prepare(connection);
       // 设置SQL查询中的参数值
       handler.parameterize(stmt);
       return stmt;
   }
   ```

   4. parameterize()方法

   ```
   1 public void parameterize(Statement statement) throws SQLException {
   2 
   3     this.parameterHandler.setParameters((PreparedStatement)statement);
   4 
   5 }
   ```

   这里把statement转换成preparedStatement对象，它比statement更快更安全。

   jdbc中的对象。其实就是对jdbc的拼接

   从ParameterMapping中读取参数值和类型，设置到sql语句中

5. 对执行结果进行二次封装

我们是插入操作，返回的是一个int类型的值，所以这里mybatis给我们直接返回int。

如果是query操作，返回的是一个ResultSet，mybatis将查询结果包装程`ResultSetWrapper`类型，然后一步步对应java类型赋值。

6. 提交与事务

   回去判断，如果dirty是false，则进行回滚，如果是true则正常提交。

   调用CachingExecutor的commit方法。

   调用BaseExecutor的commit方法。

   最后调用JDBCTransaction方法。