---
title: Spark Thrift 原理
tagline: "spark"
category : spark
layout: post
tags : [Spark thrift]
---
hive jdbc client


客户端执行sql入口statement.execute(sql);
```
HiveStatement.java
public boolean execute(String sql) throws SQLException {
  runAsyncOnServer(sql);
  TGetOperationStatusResp status = waitForOperationToComplete();

  // The query should be completed by now
  if (!status.isHasResultSet() && !stmtHandle.isHasResultSet()) {
    return false;
  }
  resultSet = new HiveQueryResultSet.Builder(this).setClient(client)
      .setStmtHandle(stmtHandle).setMaxRows(maxRows).setFetchSize(fetchSize)
      .setScrollable(isScrollableResultset)
      .build();
  return true;
}

private void runAsyncOnServer(String sql) throws SQLException {
  checkConnection("execute");

  reInitState();

  TExecuteStatementReq execReq = new TExecuteStatementReq(sessHandle, sql);
  /**
   * Run asynchronously whenever possible
   * Currently only a SQLOperation can be run asynchronously,
   * in a background operation thread
   * Compilation can run asynchronously or synchronously and execution run asynchronously
   */
  execReq.setRunAsync(true);
  execReq.setConfOverlay(sessConf);
  execReq.setQueryTimeout(queryTimeout);
  try {
    TExecuteStatementResp execResp = client.ExecuteStatement(execReq);
    Utils.verifySuccessWithInfo(execResp.getStatus());
    stmtHandle = execResp.getOperationHandle();
    isExecuteStatementFailed = false;
  } catch (SQLException eS) {
    isExecuteStatementFailed = true;
    isLogBeingGenerated = false;
    throw eS;
  } catch (Exception ex) {
    isExecuteStatementFailed = true;
    isLogBeingGenerated = false;
    throw new SQLException(ex.toString(), "08S01", ex);
  }
}
```
thrift client侧通过sendBase发送ExecuteStatement给服务端
```
//TCLIService.Iface
public TExecuteStatementResp ExecuteStatement(TExecuteStatementReq req) throws org.apache.thrift.TException
{
  send_ExecuteStatement(req);
  return recv_ExecuteStatement();
}

public void send_ExecuteStatement(TExecuteStatementReq req) throws org.apache.thrift.TException
{
  ExecuteStatement_args args = new ExecuteStatement_args();
  args.setReq(req);
  sendBase("ExecuteStatement", args);
}
```

服务端相关启动的类是CLASS="org.apache.spark.sql.hive.thriftserver.HiveThriftServer2"，入口函数main

```
  val server = new HiveThriftServer2(SparkSQLEnv.sqlContext)
  server.init(executionHive.conf)
  server.start()
```

init()会添加两种service,cliService，还有个thriftCLIService

```
override def init(hiveConf: HiveConf) {
  val sparkSqlCliService = new SparkSQLCLIService(this, sqlContext)
  setSuperField(this, "cliService", sparkSqlCliService)
  addService(sparkSqlCliService)

  val thriftCliService = if (isHTTPTransportMode(hiveConf)) {
    new ThriftHttpCLIService(sparkSqlCliService)
  } else {
    new ThriftBinaryCLIService(sparkSqlCliService)
  }

  setSuperField(this, "thriftCLIService", thriftCliService)
  addService(thriftCliService)
  initCompositeService(hiveConf)
}
```
cliService就是SparkSQLCLIService,thriftCLIService这里会有两种可选择,ThriftBinaryCLIService以及tcp模式的ThriftHttpCLIService

这两个都是封装的thrift相关的，按理thrift server服务直接看processor就好了，

先看ThriftHttpCLIService，在其run函数里边启动jettyserver,processor是TCLIService.Processor

```
 TProcessor processor = new TCLIService.Processor<Iface>(this);
 TServlet thriftHttpServlet = new ThriftHttpServlet(processor, protocolFactory, authType,
    serviceUGI, httpUGI);
// Context handler
final ServletContextHandler context = new ServletContextHandler(
    ServletContextHandler.SESSIONS);
context.setContextPath("/");
String httpPath = getHttpPath(hiveConf
    .getVar(HiveConf.ConfVars.HIVE_SERVER2_THRIFT_HTTP_PATH));
httpServer.setHandler(context);
context.addServlet(new ServletHolder(thriftHttpServlet), httpPath);
```
ThriftBinaryCLIService里边的
