## 4.5. 결과 처리
###### <table width="1200"><td width=1200>*Neo4j로부터 반환 된 결과는 레코드 스트림으로 표시됩니다.*</td></table>

명령문이 실행되면 결과는 레코드 스트림으로 반환됩니다. 스트림 자체는 개별 레코드에 액세스하는 데 사용되는 결과 커서로 표시됩니다. 개별 레코드는 한 번에 하나씩 액세스 될 수도 있고 전체가 한 번에 액세스 될 수도 있습니다. 결과는 무엇이 먼저인지에 따라 다음 명령문이 실행될 때까지 또는 현재 트랜잭션이 끝날 때까지 유효합니다. 추가 명령문을 실행하는 동안 결과를 유지하거나 현재 트랜잭션의 범위를 벗어난 결과를 사용하려면 결과를 변수에 할당하여 유지해야 합니다.

### 4.5.1. 결과 커서
결과 커서는 결과를 구성하는 레코드 스트림에 대한 액세스를 제공합니다. 커서는 스트림을 통해 진행되어 각 레코드를 차례로 가리킵니다. 커서가 첫 번째 레코드로 이동하기 전에 그것은 첫 번째 레코드 이전을 가리킵니다. 요약 정보와 메타 데이터는 커서가 결과 스트림의 끝까지 이동하면 사용할 수 있습니다.

<table width="1200"><tr><td align="center"><img src="./img/note.gif">
</td><td width="95%">
결과 레코드는 커서가 스트림을 통해 진행될 때 느리게 로드됩니다. 이것은 이 결과를 사용하려면 먼저 커서를 첫 번째 결과로 이동해야만 한다는 것을 의미합니다. 그것은 또한 요약 정보와 메타 데이터가 사용 가능하기 전에 전체 스트림을 명시적으로 소비해야 함을 뜻합니다. 일반적으로, 특히 업데이트 명령문을 실행할 때 결과룰 명시적으로 소비하고 세션을 닫는 것이 가장 좋습니다. 요약 정보는 필요하지 않더라도 적용됩니다. 서버가 Cypher 문을 보고 처리했다고 보장 할 수 없으므로 결과의 소비가 실패하면 예기치 않은 동작이 발생할 수 있습니다.
</td></tr></table>

<font color="gray">예제 4.16. 결과 커서 사용하기</font>

C#
<table><tr><td width="1200">
<font color="purple">var</font> <font color="blue">searchTerm</font> = <font color="red">"Sword"</font>;
<br><font color="purple">var</font> <font color="blue">result</font> = session.Run(<font color="red">"MATCH (weapon:Weapon) WHERE weapon.name CONTAINS {term} "</font> +
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="red">"RETURN weapon.name"</font>,
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">new</font> Dictionary<<font color="green">string, object</font>> {{<font color="red">"term"</font>, searchTerm}});
<br>
<br>Output.WriteLine($<font color="red">"List of weapons called {searchTerm}:"</font>);
<br><font color="purple">foreach</font> (<font color="purple">var</font> <font color="blue">record</font> <font color="purple">in</font> result)
<br>{
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Output.WriteLine(record[<font color="red">"weapon.name"</font>].As<<font color="green">string</font>>());
<br>}
</td></tr></table>

Java
<table><tr><td width="1200">
<font color="green">String</font> searchTerm = <font color="red">"Sword"</font>;
<br><font color="purple">try</font> ( Transaction tx = session.beginTransaction() )
<br>{
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;StatementResult result =
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;tx.run( <font color="red">"MATCH (weapon:Weapon) WHERE weapon.name CONTAINS {term} RETURN weapon.name"</font>,
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Values.parameters( <font color="red">"term"</font>, searchTerm ) );
<br>
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System.out.println( <font color="red">"List of weapons called "</font> + searchTerm + <font color="red">":"</font> );
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">while</font> ( result.hasNext() )
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Record record = result.next();
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System.out.println( record.get( <font color="red">"weapon.name"</font> ).asString() );
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
<br>}
</td></tr></table>

Javascript
<table><tr><td width="1200">
<font color="purple">var</font> <font color="blue">searchTerm</font> = <font color="red">"Sword"</font>;
<br>session
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.run( <font color="red">"MATCH (weapon:Weapon) WHERE weapon.name CONTAINS {term} RETURN weapon.name"</font>, {term: searchTerm} )
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.subscribe({
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;onNext: <font color="purple">function</font>(<font color="blue">record</font>) {
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;console.log(<font color="red">""</font> + <font color="blue">record</font>.get(<font color="red">"weapon.name"</font>));
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;onCompleted: <font color="purple">function</font>() {
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;session.close();
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;onError: <font color="purple">function</font>(<font color="blue">error</font>) {
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;console.log(<font color="blue">error</font>);
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;});
</td></tr></table>

Python
<table><tr><td width="1200">
search_term = <font color="red">"Sword"</font>
<br>result = session.run(<font color="red">"MATCH (weapon:Weapon) WHERE weapon.name CONTAINS {term} "
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"RETURN weapon.name"</font>, {<font color="red">"term"</font>: search_term})
<br><font color="purple">print</font>(<font color="red">"List of weapons called %r:"</font> % search_term)
<br><font color="purple">for</font> record <font color="purple">in</font> result:
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">print</font>(record[<font color="red">"weapon.name"</font>])
</td></tr></table>

### 4.5.2. 레코드
결과 레코드 스트림은 개별 레코드로 구성됩니다. 레코드는 결과의 일부에 대한 변경할 수 없는 뷰를 제공합니다. 그것은 키와 값의 정렬 된 맵입니다. 이러한 키-값 쌍을 필드라고 칭합니다. 필드가 Key화 되어있고 정렬되어 있으므로, 필드 값은 위치 *(0부터 시작하는 정수)* 또는 key *(문자열)* 로 액세스 할 수 있습니다.


<table width="1200"><tr><td align="center"><img src="./img/note.gif"></td>
<td width="95%">
• 위치에 따라 값에 액세스 하려면 0부터 시작하는 정수를 사용하십시오.
<br>• key에 따라 값에 액세스 하려면 문자열을 사용하십시오.
</td></tr></table>

 <font color="gray">예제 4.17. 레코드 사용하기</font>

 C#
<table><tr><td width="1200">
<font color="purple">var</font> <font color="blue">searchTerm</font> = <font color="red">"Arthur"</font>;
<br><font color="purple">var</font> <font color="blue">result</font> = session.Run(<font color="red">"MATCH (weapon:Weapon) WHERE weapon.owner CONTAINS {term} "</font> +
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="red">"RETURN weapon.name, weapon.material, weapon.size"</font>,
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">new</font> Dictionary<<font color="green">string, object</font>> {{<font color="red">"term"</font>, searchTerm}});
<br>
<br>Output.WriteLine($<font color="red">"List of weapons owned by {searchTerm}:"</font>);
<br><font color="purple">foreach</font> (<font color="purple">var</font> <font color="blue">record</font> <font color="purple">in</font> result)
<br>{
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">var</font> <font color="blue">list</font> = record.Keys.Select(key => $<font color="red">"{key}: {record[key]}"</font>).ToList();
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Output.WriteLine(<font color="green">string</font>.Join(<font color="red">", "</font>, list));
<br>}
</td></tr></table>

Java
<table><tr><td width="1200">
<font color="green">String</font> searchTerm = <font color="red">"Arthur"</font>;
<br>
<br><font color="purple">try</font> ( Transaction tx = session.beginTransaction() )
<br>{
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;StatementResult result = tx.run(
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="red">"MATCH (weapon:Weapon) WHERE weapon.owner CONTAINS {term} RETURN weapon.name, weapon.material, "</font> +
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="red">"weapon.size"</font>,
<br>
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Values.parameters( <font color="red">"term"</font>, searchTerm ) );
<br>
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System.out.println( <font color="red">"List of weapons owned by "</font> + searchTerm + <font color="red">":"</font> );
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">while</font> ( result.hasNext() )
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Record record = result.next();
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;List<<font color="green">String</font>> sword = <font color="purple">new</font> ArrayList<>();
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">for</font> ( <font color="green">String</font> key : record.keys() )
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;sword.add( key + <font color="red">": "</font> + record.get( key ) );
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System.out.println( sword );
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
<br>}
</td></tr></table>

Javascript
<table><tr><td width="1200">
<font color="purple">var</font> <font color="blue">searchTerm</font> = <font color="red">"Arthur"</font>;
<br>session
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.run( <font color="red">"MATCH (weapon:Weapon) WHERE weapon.owner CONTAINS {term} RETURN weapon.name, weapon.material, weapon.size"</font>,
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{term: searchTerm} )
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.subscribe({
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;onNext: <font color="purple">function</font>(<font color="blue">record</font>) {
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">var</font> <font color="blue">sword</font> = [];
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="blue">record</font>.forEach(<font color="purple">function</font>(<font color="blue">value, key</font>)
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="blue">sword</font>.push(<font color="blue">key</font> + <font color="red">": "</font> + <font color="blue">value</font>);
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;});
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;console.log(<font color="blue">sword</font>);
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;onCompleted: <font color="purple">function</font>() {
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;session.close();
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;onError: <font color="purple">function</font>(<font color="blue">error</font>) {
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;console.log(<font color="blue">error</font>);
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;});
</td></tr></table>

  Python
<table><tr><td width="1200">
search_term = <font color="red">"Arthur"</font>
<br>result = session.run(<font color="red">"MATCH (weapon:Weapon) WHERE weapon.owner CONTAINS {term} "
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"RETURN weapon.name, weapon.material, weapon.size"</font>, {<font color="red">"term"</font>: search_term})
<br><font color="purple">print</font>(<font color="red">"List of weapons owned by %r:"</font> % search_term)
<br><font color="purple">for</font> record <font color="purple">in</font> result:
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">print</font>(<font color="red">", "</font>.join(<font color="red">"%s: %s"</font> % (key, record[key]) <font color="purple">for</font> key <font color="purple">in</font> record.keys()))
</td></tr></table>

#### 4.5.2.1. 결과 유지
결과 레코드 스트림은 다른 명령문이 세션에서 실행되거나 현재 트랜잭션이 닫힐 때까지 사용 가능합니다. 이 범위를 벗어난 결과를 유지하려면, 결과를 명시적으로 유지되도록 해야합니다. 결과를 유지하기 위해, 각 드라이버는 결과 스트림을 수집하고 이를 각 언어의 표준 데이터 구조로 변환하는 메소드를 제공합니다. 세션이 다음 작업량을 자유롭게 처리하는 동안 보유 된 결과는 처리 될 수 있습니다. 저장된 결과는 새 명령문을 실행하는 데 직접 사용될 수도 있습니다.

<font color="gray">예제 4.18. 추가 처리를 위한 결과 유지하기

C#
<table><tr><td width="1200">
<font color="purple">var</font> <font color="blue">result</font> = session.Run(<font color="red">"MATCH (knight:Person:Knight) WHERE knight.castle = {castle} "</font> +
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="red">"RETURN knight.name AS name"</font>,
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">new</font> Dictionary<<font color="green">string, object</font>> {{<font color="red">"castle", "Camelot"</font>}});
<br><br><font color="purple">var</font> <font color="blue">records</font> = result.ToList();
<br><br><font color="purple">foreach</font> (<font color="purple">var</font> <font color="blue">record</font> <font color="purple">in</font> records)
<br>{
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Output.WriteLine($<font color="red">"{record["</font>name<font color="red">"].As<string>()} is a knight of Camelot"</font>);
<br>}
</td></tr></table>

Java
<table><tr><td width="1200">
List< Record> records;
<br><font color="purple">try</font> ( Session session = driver.session() )
<br>{
<br>
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">try</font> ( Transaction tx = session.beginTransaction() )
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;StatementResult result = tx.run(
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="red">"MATCH (knight:Person:Knight) WHERE knight.castle = {castle} RETURN knight.name AS name"</font>,
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Values.parameters( <font color="Red">"castle", "Camelot"</font> ) );
<br>
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;records = result.list();
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
<br>}
<br>
<br><font color="purple">for</font> ( Record record : records )
<br>{
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System.out.println( record.get( <font color="Red">"name"</font> ).asString() + <font color="Red">" is a knight of Camelot"</font> );
<br>}
</td></tr></table>

Javascript
<table><tr><td width="1200">
session
<br>&nbsp;&nbsp;&nbsp;&nbsp;.run(<font color="red">"MATCH (knight:Person:Knight) WHERE knight.castle = {castle} RETURN knight.name AS name"</font>,
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{castle: <font color="red">"Camelot"</font>})
<br>&nbsp;&nbsp;&nbsp;&nbsp;.then(<font color="purple">function</font> (<font color="blue">result</font>) {
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">var</font> records = [];
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">for</font> (<font color="purple">var</font> <font color="blue">i</font> = 0; <font color="blue">i</font> < <font color="blue">result</font>.records.length; <font color="blue">i</font>++) {
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="blue">records</font>.push(<font color="blue">result</font>.records[<font color="blue">i</font>]);
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">return</font> <font color="blue">records</font>;
<br>&nbsp;&nbsp;&nbsp;&nbsp;})
<br>&nbsp;&nbsp;&nbsp;&nbsp;.then(<font color="purple">function</font> (<font color="blue">records</font>) {
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">for</font>(<font color="purple">var</font> <font color="blue">i</font> = 0; <font color="blue">i</font> < <font color="blue">records</font>.length; <font color="blue">i</font> ++) {
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;console.log(<font color="blue">records</font>[<font color="blue">i</font>].get(<font color="red">"name"</font>) + <font color="red">" is a knight of Camelot"</font>);
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;session.close();
<br>
<br>&nbsp;&nbsp;&nbsp;&nbsp;});
</td></tr></table>

Python
<table><tr><td width="1200">
session = driver.session()
<br>result = session.run(<font color="red">"MATCH (knight:Person:Knight) WHERE knight.castle = {castle} "
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"RETURN knight.name AS name"</font>, {<font color="red">"castle"</font>: <font color="red">"Camelot"</font>})
<br>retained_result = <font color="blue">list</font>(result)
<br>session.close()
<br><font color="purple">for</font> record <font color="purple">in</font> retained_result:
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">print</font>(<font color="red">"%s is a knight of Camelot"</font> % record[<font color="red">"name"</font>])
</td></tr></table>

<font color="gray">예제 4.19. 새로운 쿼리에서 결과 사용하기</font>

c#
<table><tr><td width="1200">
<font color="purple">var</font> <font color="blue">result</font> = session.Run(<font color="red">"MATCH (knight:Person:Knight) WHERE knight.castle = <br>{castle} "</font> +
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="red">"RETURN id(knight) AS knight_id"</font>,
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">new</font> Dictionary<<font color="green">string, object</font>> {{<font color="red">"castle", "Camelot"</font>}});
<br>
<br><font color="purple">foreach</font> (<font color="purple">var</font> <font color="blue">record</font> <font color="purple">in</font> result)
<br>{
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;session.Run(<font color="red">"MATCH (knight) WHERE id(knight) = {id} "</font> +
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="red">"MATCH (king:Person) WHERE king.name = {king} "</font> +
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="red">"CREATE (knight)-[:DEFENDS]->(king)"</font>,
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">new</font> Dictionary<<font color="green">string, object</font>> {{<font color="red">"id"</font>, record["<font color="red">knight_id"</font>]}, {<font color="red">"king", "Arthur"</font>}});
<br>}
</td></tr></table>

Java
<table><tr><td width="1200">
StatementResult result = <font color="blue">null</font>;
<br><font color="purple">try</font> ( Transaction transaction = session.beginTransaction() )
<br>{
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;result = transaction.run(
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="red">"MATCH (knight:Person:Knight) WHERE knight.castle = {castle} RETURN id(knight) AS knight_id"</font>,
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Values.parameters( <font color="red">"castle", "Camelot"</font> ) );
<br>}
<br><font color="purple">for</font> ( Record record : result.list() )
<br>{
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">try</font> ( Transaction tx = session.beginTransaction() )
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;tx.run( <font color="red">"MATCH (knight) WHERE id(knight) = {id} "</font> +
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="red">"MATCH (king:Person) WHERE king.name = {king} "</font> +
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="red">"CREATE (knight)-[:DEFENDS]->(king)"</font>,
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Values.parameters( <font color="red">"id"</font>, record.get( <font color="red">"knight_id"</font> ), <font color="red">"king", "Arthur"</font> ) );
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;tx.success();
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
<br>}
</td></tr></table>

Javascript
<table><tr><td width="1200">
session
<br>&nbsp;&nbsp;&nbsp;&nbsp;.run(<font color="red">"MATCH (knight:Person:Knight) WHERE knight.castle = {castle} RETURN id(knight) AS knight_id"</font>,
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{castle: <font color="red">"Camelot"</font>})
<br>&nbsp;&nbsp;&nbsp;&nbsp;.subscribe({
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;onNext: <font color="purple">function</font>(<font color="blue">record</font>) {
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;session
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.run(<font color="red">"MATCH (knight) WHERE id(knight) = {id} MATCH (king:Person) WHERE king.name = {king} CREATE (knight)-[:DEFENDS]->(king)"</font>,
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{id: <font color="blue">record</font>.get(<font color="red">"knight_id"</font>), king: <font color="red">"Arthur"</font>});
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;onCompleted: <font color="purple">function</font>() {
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;session
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.run(<font color="red">"MATCH (:Knight)-[:DEFENDS]->() RETURN count(\*)"</font>)
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.then(<font color="purple">function</font> (<font color="blue">result</font>) {
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;console.log(<font color="red">"Count is "</font> + <font color="blue">result</font>.records[<font color="green">0</font>].get(<font color="green">0</font>).toInt());
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;session.close();
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;});
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;onError: <font color="purple">function</font>(<font color="blue">error</font>) {
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;console.log(<font color="blue">error</font>);
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
<br>&nbsp;&nbsp;&nbsp;&nbsp;});
</td></tr></table>

Python
<table><tr><td width="1200">
result = session.run(<font color="red">"MATCH (knight:Person:Knight) WHERE knight.castle = {castle} "
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"RETURN id(knight) AS knight_id"</font>, {<font color="red">"castle": "Camelot"</font>})
<br><font color="purple">for</font> record <font color="purple">in</font> result:
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;session.run(<font color="red">"MATCH (knight) WHERE id(knight) = {id} "
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"MATCH (king:Person) WHERE king.name = {king} "
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"CREATE (knight)-[:DEFENDS]->(king)"</font>, {<font color="red">"id"</font>: record[<font color="red">"knight_id"</font>], <font color="red">"king": "Arthur"</font>})
</td></tr></table>

### 4.5.3. 결과 요약
예를 들어 <font color="red">PROFILE</font> 또는 <font color="red">EXPLAIN</font>과 함께 Cypher 쿼리를 추가하여 얻어지는 쿼리 계획이 되는 것처럼 메타 데이터는 결과 스트림과 함께 검색 될 수 있습니다.


다음 예는 데이터베이스에서 쿼리를 실행하여 정확한 숫자와 함께 실행 계획을 작성할 것입니다. 그러면 이것은 쿼리 프로필을 출력합니다 (쿼리 프로파일링에 대한 더 자세한 정보는 [섹션 3.6.2, "쿼리 프로파일링"](https://neo4j.com/docs/developer-manual/current/cypher/query-tuning/how-do-i-profile-a-query/)을 참조하십시오.).

<font color="gray">예제 4.20. Profile 쿼리와 실행 계획 출력하기</font>

c#
<table><tr><td width="1200">
<font color="purple">var</font> <font color="blue">result</font> = session.Run(<font color="red">"PROFILE MATCH (p:Person {name: {name}}) RETURN id(p)"</font>,
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color=purple>new</font> Dictionary<<font color="green">string, object</font>> {{<font color="red">"name", "Arthur"</font>}});
<br><br>IResultSummary summary = result.Consume();
<br><br>Output.WriteLine(summary.StatementType.ToString());
<br>Output.WriteLine(summary.Profile.ToString());
</td></tr></table>

Java
<table><tr><td width="1200">
<font color="purple">try</font> ( Transaction tx = session.beginTransaction() )
<br>{
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;StatementResult result = tx.run( <font color="red">"PROFILE MATCH (p:Person { name: {name} }) RETURN id(p)"</font>,
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Values.parameters( <font color="red">"name", "Arthur"</font> ) );
<br><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ResultSummary summary = result.consume();
<br><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System.out.println( summary.statementType() );
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System.out.println( summary.profile() );
<br>}
</td></tr></table>

Javascript
<table><tr><td width="1200">
session
<br>&nbsp;&nbsp;&nbsp;&nbsp;.run(<font color="red">"PROFILE MATCH (p:Person {name: {name}}) RETURN id(p)"</font>, {name: <font color="red">"Arthur"</font>})
<br>&nbsp;&nbsp;&nbsp;&nbsp;.then(<font color="purple">function</font> (<font color="blue">result</font>) {
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;console.log(<font color="blue">result</font>.summary.profile);
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;session.close();
<br>&nbsp;&nbsp;&nbsp;&nbsp;});
</td></tr></table>

Python
<table><tr><td width="1200">
result = session.run(<font color="red">"PROFILE MATCH (p:Person {name: {name}}) "
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"RETURN id(p)"</font>, {<font color="red">"name": "Arthur"</font>})
<br>summary = result.consume()
<br><font color="purple">print</font>(summary.statement_type)
<br><font color="purple">print</font>(summary.profile)
</td></tr></table>

Neo4j는 또한 쿼리에 대한 알림을 제공할 수도 있습니다. - 예를 들어, 쿼리로 인해 실행 엔진이 카티션 프로덕트를 만들 때 경고 표시. 다음 예제는 실제로 데이터베이스에서 쿼리를 실행하지 않고 예상된 숫자가 있는 실행 계획을 작성할 것입니다.


<font color="gray">예제 4.21. Explain 쿼리와 알림 출력하기

c#
<table><tr><td width="1200">
<font color="purple">var</font> <font color="blue">summary</font> = session.Run(<font color="red">"EXPLAIN MATCH (king), (queen) RETURN king, queen"</font>).Consume();
<br><br><font color="purple">foreach</font> (<font color="purple">var</font> <font color="blue">notification</font> <font color="purple">in</font> summary.Notifications)
<br>{
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Output.WriteLine(notification.ToString());
<br>}
</td></tr></table>

Java
<table><tr><td width="1200">
<font color="purple">try</font> ( Transaction tx = session.beginTransaction() )
<br>{
    <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ResultSummary summary = tx.run( <font color="red">"EXPLAIN MATCH (king), (queen) RETURN king, queen"</font> ).consume();    
    <br><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">for</font> ( Notification notification : summary.notifications() )
    <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
    <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System.out.println( notification );
    <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
<br>}
</td></tr></table>


Javascript
<table><tr><td width="1200">
session
  <br>&nbsp;&nbsp;&nbsp;&nbsp;.run(<font color="red">"EXPLAIN MATCH (king), (queen) RETURN king, queen"</font>)
  <br>&nbsp;&nbsp;&nbsp;&nbsp;.then(<font color="purple">function</font> (<font color="blue">result</font>) {
  <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">var</font> <font color="blue">notifications</font> = <font color="blue">result</font>.summary.notifications, <font color="blue">i</font>;
  <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">for</font> (<font color="blue">i</font> = 0; <font color="blue">i</font> < <font color="blue">notifications</font>.length; <font color="blue">i</font>++) {
  <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;console.log(<font color="blue">notifications</font>[<font color="blue">i</font>].code);
  <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
  <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;session.close();
  <br>&nbsp;&nbsp;&nbsp;&nbsp;});
</td></tr></table>

Python
<table><tr><td width="1200">
result = session.run(<font color="red">"EXPLAIN MATCH (king), (queen) RETURN king, queen"</font>)
<br>summary = result.consume()
<br><font color="purple">for</font> notification <font color="purple">in</font> summary.notifications:
<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">print</font>(notification)
</td></tr></table>

드라이버는 추가적으로, 결과를 소비 가능하도록 만들기 위해 서버가 소비한 시간을 반환해주는 <font color="red">resultAvailableAfter</font>와 같은 쿼리 실행을 위해 타이밍을 노출합니다. 자세한 내용은 드라이버 API 문서를 참조하십시오.
