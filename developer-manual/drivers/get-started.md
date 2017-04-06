## 4.2. 시작하기
###### *<table width=1200><td width=1200>최소한의 작업 예제로 빠르게 시작하고 실행할 수 있습니다.</td></table>*

### 4.2.1. 드라이버 구하기
#### 4.2.1.1. 버전
특정 버전의 Neo4j에서 사용할 드라이버 버전을 확인하려면 버전 표를 참조하십시오.

<font color="gray">표 4.1. 지원 버전</font>

| 드라이버 버전 | Neo4j 버전 | 볼트 프로토콜 버전 |
|--------------|------------|---------------------|
|1.1|3.1|1|
|1.1|3.0 <*>|1|
|1.0|3.1 <*>|1|
|1.0|3.0|1|

<\*> 드라이버 버전 1.0과 1.1은 Neo4j 3.0 및 3.1에서 모두 작동합니다. Neo4j 3.1에서 Causal Cluster 모드의 장점을 활용하고 북마킹 및 라우팅과 같은 기능을 사용하려면 버전 1.1의 드라이버를 사용해야합니다.

#### 4.2.1.2. 드라이버 얻기
다운로드하거나 해당 언어의 dependency 관리자 중 하나를 사용하여 드라이버 소스를 얻을 수 있습니다.

<font color="gray">예제 4.1. 드라이버 얻기</font>

C#
Visual Studio에서 NuGet 사용하기:
<table><tr><td width="1200">
PM> Install-Package Neo4j.Driver -Version 1.2.0
</td></tr></table>

java
Maven을 사용할 때, *pom.xml* 파일에 다음을 추가하십시오 :
<table><tr><td width="1200">
<<font color="green">dependencies</font>>
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<<font color="green">dependency</font>>
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<<font color="green">groupId</font>>org.neo4j.driver<<font color="green">/groupId</font>>
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<<font color="green">artifactId</font>>neo4j-java-driver<<font color="green">/artifactId</font>>
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<<font color="green">version</font>>1.2.0<<font color="green">/version</font>>
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<<font color="green">/dependency</font>>
<br><<font color="green">/dependencies</font>>
</td></tr></table>

Gradle 또는 Grails의 경우, 종속성을 추가하는 방법은 다음과 같습니다.
<table><tr><td width="1200">
compile <font color="red">'org.neo4j.driver:neo4j-java-driver:1.2.0'</font>
</td></tr></table>

다른 빌드 시스템의 경우, [Maven Central에서 사용가능한 정보](http://search.maven.org/#artifactdetails%7Corg.neo4j.driver%7Cneo4j-java-driver%7C1.2.0%7Cjar)를 참조하십시오.

javascript
<table><tr><td width="1200">
<font color="purple">npm</font> install neo4j-driver@1.1.0
</td></tr></table>

python
<table><tr><td width="1200">
shellpip install <font color="blue">neo4j-driver</font>==1.1.0
</td></tr></table>

### 4.2.2. 드라이버 사용하기
각 Neo4j 드라이버에는 드라이버를 만들기 위한 데이터베이스 객체가 있습니다. 드라이버를 사용하려면 다음 패턴을 따르십시오.

1. 데이터베이스 객체에 새 드라이버를 요청하십시오.
2. 드라이버 객체에 새 세션을 요청하십시오.
3. 세션 객체에 트랜잭션 생성을 요청하십시오.
4. 명령문을 실행하려면 트랜잭션 객체를 사용하십시오. 그것은 결과를 나타내는 객체를 반환합니다.
5. 결과를 처리하십시오.
6. 세션을 닫으십시오.

<font color="gray">예제 4.2. 드라이버 사용하기</font>

C#
최소 작업 예제를 위해, 다음과 같은 가져오기가 필요합니다.
<table><tr><td width="1200">
<font color="purple">using</font> Neo4j.Driver.V1;
</td></tr></table>

그러면 이들은 서버에 대해 명령문을 실행하기 위해 사용될 수 있습니다.
<table><tr><td width="1200">
<font color="purple">using</font> (<font color="purple">var</font> <font color="blue">driver</font> = GraphDatabase.Driver(<font color="red">"bolt://localhost:7687"</font>, AuthTokens.Basic(<font color="red">"neo4j", "neo4j"</font>)))
<br><font color="purple">using</font> (<font color="purple">var</font> <font color="blue">session</font> = driver.Session())
<br>{
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;session.Run(<font color="red">"CREATE (a:Person {name: {name}, title: {title}})"</font>,
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">new</font> Dictionary<<font color="green">string, object</font>> { {<font color="red">"name", "Arthur"</font>}, {<font color="red">"title", "King"</font>} });

<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">var</font> <font color="blue">result</font> = session.Run(<font color="red">"MATCH (a:Person) WHERE a.name = {name} "</font> +
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="red">"RETURN a.name AS name, a.title AS title"</font>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">new</font> Dictionary<<font color="green">string, object</font>> { {<font color="red">"name", "Arthur"</font>} });
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">foreach</font> (<font color="purple">var</font> <font color="blue">record</font> <font color="purple">in</font> result)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Output.WriteLine($<font color="red">"{record["</font>title<font color="red">"].As<<font>string</font>>()} {record["</font>name<font color="red">"].As<<font>string</font>>()}"</font>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
}
</td></tr></table>

java
최소 작업 예제를 위해, 다음과 같은 가져오기가 필요합니다.
<table><tr><td width="1200">
<font color="purple">import</font> org.neo4j.driver.v1.AuthTokens;
<br><font color="purple">import</font> org.neo4j.driver.v1.Driver;
<br><font color="purple">import</font> org.neo4j.driver.v1.GraphDatabase;
<br><font color="purple">import</font> org.neo4j.driver.v1.Record;
<br><font color="purple">import</font> org.neo4j.driver.v1.Session;
<br><font color="purple">import</font> org.neo4j.driver.v1.StatementResult;
<br><font color="purple">import</font> org.neo4j.driver.v1.Transaction;

<br><font color="purple">import</font> static org.neo4j.driver.v1.Values.parameters;
</td></tr></table>

그러면 이들은 서버에 대해 명령문을 실행하기 위해 사용될 수 있습니다.
<table><tr><td width="1200">
Driver driver = GraphDatabase.driver( <font color="red">"bolt://localhost:7687"</font>, AuthTokens.basic( <font color="red">"neo4j", "neo4j"</font> ) );
<br><br>
<font color="purple">try</font> ( Session session = driver.session() )
<br>{
<br><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">try</font> ( Transaction tx = session.beginTransaction() )
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;tx.run( <font color="red">"CREATE (a:Person {name: {name}, title: {title}})"</font>,
<br>                parameters( <font color="red">"name", "Arthur", "title", "King"</font> ) );
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;tx.success();
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}

<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">try</font> ( Transaction tx = session.beginTransaction() )
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;StatementResult result = tx.run( <font color="red">"MATCH (a:Person) WHERE a.name = {name} "</font> +
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="red">"RETURN a.name AS name, a.title AS title"</font>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;parameters( <font color="red">"name", "Arthur"</font> ) );
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">while</font> ( result.hasNext() )
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Record record = result.next();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System.out.println( <font color="green">String</font>.format( <font color="red">"%s %s"</font>, record.get( <font color="red">"title"</font> ).asString(), record.get( <font color="red">"name"</font> ).asString() ) );
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}

}

driver.close();
</td></tr></table>

javascript
최소 작업 예제를 위해, 다음과 같은 가져오기가 필요합니다.
<table><tr><td width="1200">
<font color="purple">var</font> <font color="blue">neo4j</font> = require(<font color="red">'neo4j-driver'</font>).v1;
</td></tr></table>
그러면 이들은 서버에 대해 명령문을 실행하기 위해 사용될 수 있습니다.
<table><tr><td width="1200">
<font color="purple">var</font> <font color="blue">driver</font> = neo4j.driver(<font color="red">"bolt://localhost:7687"</font>, neo4j.auth.basic(<font color="red">"neo4j", "neo4j"</font>));
<br><font color="purple">var</font> <font color="blue">session</font> = driver.session();
<br>session
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.run( <font color="red">"CREATE (a:Person {name: {name}, title: {title}})"</font>, {name: <font color="red">"Arthur"</font>, title: <font color="red">"King"</font>})
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.then( <font color="purple">function</font>()
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">return</font> session.run( <font color="red">"MATCH (a:Person) WHERE a.name = {name} RETURN a.name AS name, a.title AS title"</font>,
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{name: <font color="red">"Arthur"</font>})
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;})
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.then( <font color="purple">function</font>( <font color="blue">result</font> ) {
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;console.log( <font color="blue">result</font>.records[0].get(<font color="red">"title"</font>) + " " + <font color="blue">result</font>.records[0].get(<font color="red">"name"</font>) );
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;session.close();
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;driver.close();
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;});
  </td></tr></table>

python
최소 작업 예제를 위해, 다음과 같은 가져오기가 필요합니다.
<table><tr><td width="1200">
<font color="purple">from</font> neo4j.v1 <font color="purple">import</font> GraphDatabase, basic_auth
</td></tr></table>
그러면 이들은 서버에 대해 명령문을 실행하기 위해 사용될 수 있습니다.
<table><tr><td width="1200">
driver = GraphDatabase.driver(<font color="red">"bolt://localhost:7687"</font>, auth=basic_auth(<font color="red">"neotest", "neotest"</font>))
<br>session = driver.session()
<br><br>
session.run(<font color="red">"CREATE (a:Person {name:'Arthur', title:'King'})"</font>)
<br><br>
result = session.run(<font color="red">"MATCH (a:Person) WHERE a.name = 'Arthur' RETURN a.name AS name, a.title AS title"</font>)
<br><font color="purple">for</font> record <font color="purple">in</font> result:
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">print</font>(<font color="red">"%s %s"</font> % (record[<font color="red">"title"</font>], record[<font color="red">"name"</font>]))

<br>session.close()
</td></tr></table>

또한 세션 객체에서 직접 명령문을 실행할 수도 있습니다. 그러면 세션이 트랜잭션 열기 및 닫기를 처리할 것입니다. 이는 개별 읽기 전용 쿼리를 위해 편리할 수 있습니다.
