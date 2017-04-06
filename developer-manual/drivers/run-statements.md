## 4.4. 명령문 실행
###### <table width="1200"><td width=1200>*Neo4j 데이터베이스와의 모든 상호 작용은 세션 내에서 발생합니다. 각 세션은 데이터베이스에 대한 작업의 하나 이상의 트랜잭션에 대한 컨텍스트입니다.*</td></table>

세션은 일련의 트랜잭션에 대한 경량 컨테이너입니다. 세션은 드라이버 객체에서 가져옵니다. 그것은 개별 트랜잭션에 대한 작업 컨텍스트를 제공합니다.

라우팅 드라이버를 사용할 때는 세션 액세스 모드를 <font color="red">READ</font> 또는 <font color="red">WRITE</font>로 선언하십시오. 드라이버는 선택된 작업 모드에 적합한 클러스터 멤버에 대해 세션을 생성합니다.

일반적인 경우, 트랜잭션은 명시적으로 관리됩니다. 이것은 트랜잭션이 세션에서 생성되고 하나 이상의 명령문을 실행하는 데 사용됨을 의미합니다. 각 명령문에 대해 결과가 처리되고 예외가 다루어질 수 있습니다. 트랜잭션 시퀀스가 모두 실행되면 세션이 닫힙니다.

<font color="gray">예제 4.10. 명령문 실행하기</font>

C#
<table><tr><td width="1200">
<font color="purple">using</font> (<font color="purple">var</font> <font color="blue">tx</font> = session.BeginTransaction())
<br>{
<br>&nbsp;&nbsp;&nbsp;&nbsp;tx.Run(<font color="red">"CREATE (:Person {name: {name}})"</font>,
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">new</font> Dictionary<<font color="green">string, object</font>> {{<font color="red">"name", "Guinevere"</font>}});
<br>&nbsp;&nbsp;&nbsp;&nbsp;tx.Success();
<br>}
</td></tr></table>

Java
<table><tr><td width="1200">
<font color="purple">try</font> ( Transaction tx = session.beginTransaction() )
<br>{
<br>&nbsp;&nbsp;&nbsp;&nbsp;tx.run( <font color="red">"CREATE (:Person {name: {name}})"</font>,
<br><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Values.parameters( <font color="red">"name", "Guinevere"</font> ) );
<br>&nbsp;&nbsp;&nbsp;&nbsp;tx.success();
<br>}
</td></tr></table>

JavaScript
<table><tr><td width="1200">
<font color="purple">var</font> <font color="blue">tx</font> = session.beginTransaction();
<br>tx.run( <font color="red">"CREATE (:Person {name: {name}})"</font>, {name: <font color="red">"Guinevere"</font>} );
<br>tx.commit().then(<font color="purple">function</font>() {session.close()});
</td></tr></table>

Python
<table><tr><td width="1200">
<font color="purple">with</font> session.begin_transaction() <font color="purple">as</font> tx:
<br>&nbsp;&nbsp;&nbsp;&nbsp;tx.run(<font color="red">"CREATE (:Person {name: 'Guinevere'})"</font>)
<br>&nbsp;&nbsp;&nbsp;&nbsp;tx.success = <font color="blue">True</font>
</td></tr></table>

### 4.4.1. 세션
세션은 공통 엔드포인트로 작업 단위를 그룹화하는 데 사용할 수 있는 연결 래퍼입니다. 새로운 세션을 취득하려면, <font color="red">driver.session</font> 메소드를 사용하십시오. 세션 생성은 저비용 프로세스입니다. 기본 연결은 구성시 드라이버 연결 풀에서 빌려 오고 닫을 때 풀로 반환됩니다.

세션은 스레드로부터 안전하지 않습니다.

#### 4.4.1.1. 액세스 모드
드라이버에서 라우팅이 활성화되면 원하는 액세스 모드 (<font color="red">READ</font> 또는 <font color="red">WRITE</font>)를 제공하십시오. 만일 생략되면 <font color="red">WRITE</font>가 역 호환성을 위해 사용됩니다.

라우팅 드라이버는 각각 쓰기 또는 읽기에 적합한 엔드포인트에 대한 세션을 만듭니다. Causal Cluster에 대한 배포는 드라이버가 읽기 및 쓰기를 효율적으로 라우팅 할 수 있음을 의미합니다. 읽기는 읽기 복제본으로 보내는 반면 쓰기는 컨센서스 커밋 및 보관을 위해 클러스터의 코어로 이동합니다

세션이 생성 된 인스턴스가 선언 된 액세스 모드에서 더 이상 사용할 수 없는 경우 어플리케이션은 <font color="red">SessionExpiredException</font>을 수신합니다.
이 경우 어플리케이션은 드라이버에서 다른 세션을 가져 와서 실패한 트랜잭션을 재생해야 합니다.

#### 4.4.1.2. 연결 풀링
드라이버에는 연결 풀이 있으며 세션은 연결을 빌려 작업을 수행합니다. 세션이 닫히면 연결은 풀로 반환됩니다. 클라이언트에서 결과 및 오류를 완전히 수신하고 연결을 다시 사용할 수 있도록 세션을 올바르게 닫는 것이 중요합니다.

<table width=1200><tr><td><img src="./img/caution.gif"></td><td width=90%>예외가 발생하는 경우 세션 닫기를 제공하는 것이 중요합니다. 예상되는 실행 경로 뿐만아니라 예외적인 실행 경로가 세션을 닫는 지 확인 하십시오.</td></tr><table>

### 4.4.2. 트랜잭션 롤백
트랜잭션도 롤백 될 수 있습니다.

<font color="gray">예제 4.11. 트랜잭션 롤백하기</font>

C#
<table><tr><td width="1200">
<font color="purple">using</font> (<font color="purple">var</font> <font color="blue">tx</font> = session.BeginTransaction())
<br>{
<br>&nbsp;&nbsp;&nbsp;&nbsp;tx.Run(<font color="red">"CREATE (:Person {name: {name}})"</font>,
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">new</font> Dictionary<<font color="green">string, object</font>> {{<font color="red">"name", "Merlin"</font>}});
<br>&nbsp;&nbsp;&nbsp;&nbsp;tx.Failure(); <font color="brown">// This step is optional. If not called,  tx.Failure() is implicit.</font>
<br>}
</td></tr></table>

Java
<table><tr><td width="1200">
<font color="purple">try</font> ( Transaction tx = session.beginTransaction() )
<br>{
<br>&nbsp;&nbsp;&nbsp;&nbsp;tx.run( <font color="red">"CREATE (:Person {name: {name}})"</font>,
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Values.parameters( <font color="red">"name", "Merlin"</font> ) );
<br>&nbsp;&nbsp;&nbsp;&nbsp;tx.failure();
<br>}
</td></tr></table>

JavaScript
<table><tr><td width="1200">
<font color="purple">var</font> <font color="blue">tx</font> = session.beginTransaction();
<br>tx.run( <font color="red">"CREATE (:Person {name: {name}})"</font>, {name: <font color="red">"Merlin"</font>});
<br>tx.rollback().then(<font color="purple">function</font>() {session.close()});
</td></tr></table>

Python
<table><tr><td width="1200">
<font color="purple">with</font> session.begin_transaction() <font color="purple">as</font> tx:
<br>&nbsp;&nbsp;&nbsp;&nbsp;tx.run(<font color="red">"CREATE (:Person {name: 'Merlin'})"</font>)
<br>&nbsp;&nbsp;&nbsp;&nbsp;tx.success = <font color="blue">False</font>
</td></tr></table>

### 4.4.3. 자덩 커밋 트랜잭션
일반적인 경우는 명시적인 트랜잭션을 사용하는 것이지만, 세션에서 직접 명령문을 실행할 수도 있습니다. 세션의 <font color="red">run()</font> 메소드는 명령문이 실행되는 트랜잭션을 암묵적으로 시작하고 커밋합니다. 이것은 개별 읽기 전용 쿼리를 위한 편리한 방법입니다.

자동 커밋 트랜잭션은 북마킹을 지원하지 않으며 파이프라인 될 때 실패시 진단하기 어렵습니다.

<font color="gray">예제 4.12. 자동 커밋 트랜잭션에서 명령문 실행하기 </font>

C#
<table><tr><td width="1200">
<font color="purple">var</font> <font color="blue">result</font> = session.Run(<font color="red">"CREATE (person:Person {name: {name}})"</font>,
<br>&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">new</font> Dictionary<<font color="green">string, object</font>> {{<font color="red">"name", "Arthur"</font>}});
</td></tr></table>

Java
<table><tr><td width="1200">
StatementResult result =
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;transaction.run( <font color="red">"CREATE (person:Person {name: {name}})"</font>, Values.parameters( <font color="red">"name"</font>, <font color="red">"Arthur"</font> ) );
<br>transaction.success();
</td></tr></table>

JavaScript
<table><tr><td width="1200">
session
<br>&nbsp;&nbsp;&nbsp;&nbsp;.run( <font color="red">"CREATE (person:Person {name: {name}})"</font>, {name: <font color="red">"Arthur"</font>} )
</td></tr></table>

Python
<table><tr><td width="1200">
result = session.run(<font color="red">"CREATE (person:Person {name: {name}})"</font>, {<font color="red">"name"</font>: <font color="red">"Arthur"</font>})
</td></tr></table>

### 4.4.4. 쿼리 파라미터
항상 파라미터를 사용하는 것이 좋습니다. 파라미터를 사용하면 중요한 성능 및 보안 이점을 얻을 수 있습니다:
- 쿼리 플래너는 쿼리를 훨씬 효율적으로 만들어 계획을 재사용할 수 있습니다.
- 잘못 입력되거나 필터링 된 입력에서 악의적인 쿼리 절이 추가되는 Cypher 삽입 공격으로부터 데이터베이스를 보호합니다.

그러나 문자 그대로의 값을 사용하여 쿼리를 실행할 수도 있습니다.

<font color="gray">예제 4.13. 파라미터 없이 명령문 실행하기</font>

C#
<table><tr><td width="1200">
<font color="purple">var</font> <font color="blue">result</font> = session.Run(<font color="red">"CREATE (p:Person {name: 'Arthur'})"</font>);
</td></tr></table>

Java
<table><tr><td width="1200">
StatementResult result = transaction.run( <font color="red">"CREATE (p:Person { name: 'Arthur' })"</font> );
<br>transaction.success();
</td></tr></table>

JavaScript
<table><tr><td width="1200">
session
<br>&nbsp;&nbsp;&nbsp;&nbsp;.run( <font color="red">"CREATE (p:Person {name: 'Arthur'})"</font> )
</td></tr></table>

Python
<table><tr><td width="1200">
result = session.run(<font color="red">"CREATE (person:Person {name: 'Arthur'})"</font>)
</td></tr></table>

### 4.4.5. 북마킹
Causal Cluster와 함께 볼트 드라이버를 사용하면 어플리케이션에 인과 관계의 일관성을 제공합니다. 이는 어플리케이션이 항상 관련 데이터를 일관되게 볼 수 있음을 의미합니다. 이것은 북마크를 통해 어플리케이션에 노출됩니다.

세션에서 작업을 수행 한 후 북마크가 생성됩니다. 북마크는 작업에 대한 포인터입니다. 다른 관련 작업에 대한 새 세션이 만들어질 때 북마크가 사용됩니다.북마크를 사용하면 이전의 작업에 대한 종속성이 유지됩니다. 데이터가 클러스터의 모든 코너에 복제되었는지 여부에 관계없이 북마크는 이전 작업으로 다음 세션이 최신 상태가 될 모든 인스턴스 서비스를 보장합니다.

클러스터에 쓰기를 하면, 작업의 논리적 위치를 나타내는 북마크가 생성됩니다.

<font color="gray">예제 4.14. 북마크 생성하기</font>
<table><tr><td width="1200">
Driver driver = GraphDatabase.driver( seedUri, Authtokens.basic( <font color="red">"neo4j", "neo4j"</font> ) );
<br><font color="green">String</font> bookmark;
<br><br>
<font color="purple">try</font> ( Session session = driver.session( AccessMode.WRITE ) )
<br>{
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">try</font> ( Transaction tx = session.beginTransaction() )
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;tx.run( <font color="red">"CREATE (person:Person {name: {name}, title: {title}})"</font>,
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;parameters( <font color="red">"name", "Arthur", "title", "king"</font> ) );
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;tx.success();
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;tx.close();
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">finally</font>
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;bookmark = session.lastBookmark();
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
<br>}
<font color="purple">return</font> bookmark;
</td></tr></table>

북마크가 생성되어지면, 클러스터에서 읽을 때 이후 세션에서 북마크를 사용합니다. 북마크 된 작업을 인식하는 인스턴스가 그 세션을 처리합니다. 데이터 보기는 어플리케이션에 대해 일관적이 됩니다.

<font color="gray">예제 4.15. 북마크 사용하기</font>
<table><tr><td width="1200">
<font color="purple">try</font> ( Session session = driver.session( AccessMode.READ ) )
<br>{
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;StatementResult result;
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">try</font> ( Transaction tx = session.beginTransaction( bookmark ) )
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;result = tx.run( <font color="red">"MATCH (person:Person {name: {name}}) RETURN person.title"</font>,
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;parameters( <font color="red">"name"</font>, <font color="red">"Arthur"</font> ) );
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;tx.success();
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;tx.close();
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">while</font> ( result.hasNext() )
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Record record = result.next();
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System.out.println( <font color="red">"Arthur's title is "</font> + record.get( <font color="red">"person.title"</font> ) );
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
<br>}
</td></tr></table>
