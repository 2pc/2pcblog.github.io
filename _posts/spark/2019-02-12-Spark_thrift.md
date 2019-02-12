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
thrift client侧通过sendBase发送ExecuteStatement
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
