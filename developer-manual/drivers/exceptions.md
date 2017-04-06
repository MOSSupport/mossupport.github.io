## 4.7. 예외
<font color="gray">예외 목록</font>

#### ClientException
><font color="red">ClientException</font>은 클라이언트가 조작을 잘못 수행했음을 나타냅니다. 제공된 오류 코드는 문제에 대한 세부 사항을 판별하는데 사용될 수 있습니다.

#### ConnectionFailureException
><font color="red">ConnectionFailureException</font>는 기본 연결에 문제가 있음을 나타냅니다. 연결이 종결되었을 가능성이 있습니다.

#### DatabaseException
><font color="red">DatabaseException</font>는 기본 데이터베이스 내에 문제가 있음을 나타냅니다. 제공된 오류 코드는 문제에 대한 세부 사항을 판별하는데 사용될 수 있습니다.

#### NoSuchRecordException
>클라이언트가 사용할 수 없는 레코드를 읽을 것으로 예상 될 때마다 (즉, 서버에서 반환되지 않았기 때문에) 발생됩니다. 이것은 일반적으로 클라이언트 코드와 데이터베이스 어플리케이션 로직 사이의 예상 불일치를 나타냅니다.

#### ServiceUnavailableException
><font color="red">ServiceUnavailableException</font>은 드라이버가 생성시 제공된 엔드 포인트와 통신 할 수 없음을 나타냅니다. 라우팅 URI (볼트 + 라우팅)의 경우 이 오류를 해결하기 위해 드라이버를 다시 만들어야 할 수도 있습니다.

#### SessionExpiredException

><font color="red">SessionExpiredException</font>은 세션이 더 이상 획득 된 기준을 충족시킬 수 없음을 나타냅니다. 예를 들어, 서버가 더 이상 쓰기 요청을 허용하지 않는 경우가 이에 해당합니다. 이 경우 드라이버에서 새 세션을 가져와 만료 된 세션에서 수행 한 모든 작업을 재수행 해야만 합니다.

#### TransientException
><font color="red">TransientException</font>은 다시 시도하면 해결할 수 있는 일시적인 오류를 알립니다. 제공된 오류 코드는 문제에 대한 세부 사항을 판별하는데 사용될 수 있습니다.

<font color="gray">예제 4.23. 클라이언트 오류 처리하기</font>

C#
<table><tr><td width="1200">
<font color="purple">try</font>
<br>{
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;session.Run(<font color="red">"This will cause a syntax error"</font>).Consume();
<br>}
<br><font color="purple">catch</font> (ClientException)
<br>{
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">throw new</font> InvalidOperationException(<font color="red">"Something really bad has happened!"</font>);
<br>}
</td></tr></table>

Java
<table><tr><td width="1200">
<font color="purple">try</font>
<br>{
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">try</font> ( Transaction tx = session.beginTransaction() )
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;tx.run( <font color="red">"This will cause a syntax error"</font> ).consume();
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
<br>}
<br><font color="purple">catch</font> ( ClientException e )
<br>{
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">throw new</font> RuntimeException( <font color="red">"Something really bad has happened!"</font> );
<br>}
</td></tr></table>

javascript
<table><tr><td width="1200">
session
<br>&nbsp;&nbsp;&nbsp;&nbsp;.run(<font color="red">"This will cause a syntax error"</font>)
<br>&nbsp;&nbsp;&nbsp;&nbsp;.catch( <font color="purple">function</font>(<font color="blue">err</font>) {
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;console.log(<font color="blue">err</font>);
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;session.close();
<br>&nbsp;&nbsp;&nbsp;&nbsp;});
</td></tr></table>

python
<table><tr><td width="1200">
<font color="purple">try</font>:
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;session.run(<font color="red">"This will cause a syntax error"</font>).consume()
<br><font color="purple">except</font> CypherError:
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">raise</font> RuntimeError(<font color="red">"Something really bad has happened!"</font>)
<br><font color="purple">finally</font>:
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;session.close()
</td></tr></table>
