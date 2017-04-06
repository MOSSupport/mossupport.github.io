### 3.6.3. 기본 쿼리 튜닝 예제
먼저 프로파일링 쿼리를 수행하는 데 도움이 되는 기본 예제부터 살펴 보겠습니다. 다음 예제에서는 동영상 데이터 세트를 사용합니다.

데이터 가져오기부터 시작하겠습니다.

```Javascript
LOAD CSV WITH HEADERS FROM 'https://neo4j.com/docs/developer-manual/3.1/csv/query-tuning/movies.csv' AS line
MERGE (m:Movie { title: line.title })
ON CREATE SET m.released = toInt(line.released), m.tagline = line.tagline
```
```Javascript
LOAD CSV WITH HEADERS FROM 'https://neo4j.com/docs/developer-manual/3.1/csv/query-tuning/actors.csv' AS line
MATCH (m:Movie { title: line.title })
MERGE (p:Person { name: line.name })
ON CREATE SET p.born = toInt(line.born)
MERGE (p)-[:ACTED_IN { roles:split(line.roles, ';')}]->(m)
```
```Javascript
LOAD CSV WITH HEADERS FROM 'https://neo4j.com/docs/developer-manual/3.1/csv/query-tuning/directors.csv' AS line
MATCH (m:Movie { title: line.title })
MERGE (p:Person { name: line.name })
ON CREATE SET p.born = toInt(line.born)
MERGE (p)-[:DIRECTED]->(m)
```
'Tom 행크스'를 찾기 위해 쿼리를 작성한다고 가정해보겠습니다. 이렇게 하는 단순한 방법은 다음과 같습니다.

```Javascript
MATCH (p { name: 'Tom Hanks' })
RETURN p
```
이 쿼리는 'Tom Hanks'노드를 찾지만 데이터베이스의 노드 수가 증가함에 따라 느려집니다. 그 이유를 찾기 위해 쿼리를 프로파일링 할 수 있습니다.

[3.6.2 "쿼리 프로파일링"](./how-do-i-profile-a-query.html)에서 쿼리 프로파일링 옵션에 대해 자세히 배울 수 있습니다. 하지만 이 경우에는 쿼리에 PROFILE 접두어를 붙입니다.

```Javascript
PROFILE
MATCH (p { name: 'Tom Hanks' })
RETURN p
```
```
Compiler CYPHER 3.1

Planner COST

Runtime INTERPRETED

+-----------------+----------------+------+---------+-----------+---------------------------+
| Operator        | Estimated Rows | Rows | DB Hits | Variables | Other                     |
+-----------------+----------------+------+---------+-----------+---------------------------+
| +ProduceResults |             16 |    1 |       0 | p         | p                         |
| |               +----------------+------+---------+-----------+---------------------------+
| +Filter         |             16 |    1 |     163 | p         | p.name == {  AUTOSTRING0} |
| |               +----------------+------+---------+-----------+---------------------------+
| +AllNodesScan   |            163 |  163 |     164 | p         |                           |
+-----------------+----------------+------+---------+-----------+---------------------------+
Total database accesses: 327
```
실행 계획을 읽을 때 먼저 염두에 두어야 할 점은 아래에서 위로 읽어야한다는 것입니다.

마지막 행에서부터 시작하여 우리가 알아 차린 첫 번째 사실은 데이터베이스에 'Tom Hanks'라는 이름의 속성이있는 노드가 하나 뿐이므로 행 열의 값이 높다는 것입니다. Operator 열을 살펴보면 AllNodesScan이 사용되었음을 알 수 있습니다. 즉, 쿼리 플래너가 데이터베이스의 모든 노드를 통해 검색했음을 의미합니다.

이전 행으로 이동하면 AllNodesScan이 통과 한 각 노드에서 name 속성을 검사하는 Filter 연산자가 나타납니다.

이것은 사람이 아니기 때문에 우리가 찾고 있는 것이 아닌 많은 노드를 보고 있다는 점에서 'Tom 행크스'를 찾는 비효율적인 방법처럼 보입니다.

이 문제에 대한 해결책은 노드를 찾을 때마다 쿼리 플래너가 검색 공간을 좁히기 좋은 레이블을 지정해야한다는 것입니다. 이 쿼리에서는 Person 레이블을 추가해야 합니다.
```Javascript
MATCH (p:Person { name: 'Tom Hanks' })
RETURN p
```
이 쿼리는 첫 번째 쿼리보다 빠르지만 데이터베이스의 사용자 수가 증가함에 따라 쿼리가 다시 느려집니다.

다시 쿼리 결과를 분석하여 그 이유를 알 수 있습니다.
```Javascript
PROFILE
MATCH (p:Person { name: 'Tom Hanks' })
RETURN p
```
```
Compiler CYPHER 3.1

Planner COST

Runtime INTERPRETED

+------------------+----------------+------+---------+-----------+---------------------------+
| Operator         | Estimated Rows | Rows | DB Hits | Variables | Other                     |
+------------------+----------------+------+---------+-----------+---------------------------+
| +ProduceResults  |             13 |    1 |       0 | p         | p                         |
| |                +----------------+------+---------+-----------+---------------------------+
| +Filter          |             13 |    1 |     125 | p         | p.name == {  AUTOSTRING0} |
| |                +----------------+------+---------+-----------+---------------------------+
| +NodeByLabelScan |            125 |  125 |     126 | p         | :Person                   |
+------------------+----------------+------+---------+-----------+---------------------------+

Total database accesses: 251
```
이번에는 마지막 행의 행 값이 줄어들었기 때문에 우리가 전에 시작했던 노드 중 일부를 스캐닝하지 않아도 됩니다. NodeByLabelScan 연산자는 데이터베이스의 모든 Person 노드에 대한 선형 스캔을 먼저 수행하여 이를 수행했음을 나타냅니다.

일단 우리가 다시 한 번 Filter 연산자를 사용하여 모든 노드를 스캔하고 각각의 name 속성을 비교합니다.

이것은 어떤 경우에는 용인 될 수 있습니다만, 이름으로 사람들을 자주 찾는다면 우리는 Person 레이블의 name 특성에 대한 색인을 작성하여 더 나은 성능을 볼 수 있을 것입니다.

```Javascript
CREATE INDEX ON :Person(name)
```
이제 쿼리를 다시 실행하면 더 빨리 실행됩니다.

```Javascript
MATCH (p:Person { name: 'Tom Hanks' })
RETURN p
```
그 이유를 보기 위해 쿼리를 프로파일링 해 봅시다.
```Javascript
PROFILE
MATCH (p:Person { name: 'Tom Hanks' })
RETURN p
```
```
Compiler CYPHER 3.1

Planner COST

Runtime INTERPRETED

+-----------------+----------------+------+---------+-----------+---------------+
| Operator        | Estimated Rows | Rows | DB Hits | Variables | Other         |
+-----------------+----------------+------+---------+-----------+---------------+
| +ProduceResults |              1 |    1 |       0 | p         | p             |
| |               +----------------+------+---------+-----------+---------------+
| +NodeIndexSeek  |              1 |    1 |       2 | p         | :Person(name) |
+-----------------+----------------+------+---------+-----------+---------------+

Total database accesses: 2
```
우리의 실행 계획은 단일행으로 내려가고 적절한 인덱스를 찾기 위해 스키마 인덱스 탐색 ([3.5.1. "인덱스"](../schema/index.html) 참조)을 수행하는 Node Index Seek 연산자를 사용합니다.
