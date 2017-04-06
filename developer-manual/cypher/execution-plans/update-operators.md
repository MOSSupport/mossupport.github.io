### 3.7.5. 업데이트 연산자

이 연산자는 그래프를 업데이트하는 쿼리에 사용됩니다.

#### 3.7.5.1. 제약조건 연산

(레이블, 속성) 쌍에 대한 제약 조건을 만듭니다. 다음 쿼리는 ```Country``` 레이블이 있는 노드의 ```name``` 속성에 대한 고유 제한 조건을 생성합니다.

**Query.**
```Javascript
CREATE CONSTRAINT ON (c:Country) ASSERT c.name IS UNIQUE
```
**Query Plan.**
```
+-------------------------+
| Operator                |
+-------------------------+
| +CreateUniqueConstraint |
+-------------------------+

Total database accesses: ?
```
#### 3.7.5.2. Empty Result

```EmptyResult``` 연산자에 들어오는 모든 것을 열심히 로드하고 버립니다.

**Query.**
```Javascript
CREATE (:Person)
```
**Query Plan.**
```
+-----------------+----------------+------+---------+-----------+
| Operator        | Estimated Rows | Rows | DB Hits | Variables |
+-----------------+----------------+------+---------+-----------+
| +ProduceResults |              1 |    0 |       0 |           |
| |               +----------------+------+---------+-----------+
| +EmptyResult    |                |    0 |       0 |           |
| |               +----------------+------+---------+-----------+
| +CreateNode     |              1 |    1 |       2 | anon[8]   |
+-----------------+----------------+------+---------+-----------+

Total database accesses: 2
```
#### 3.7.5.3. Update Graph

Applies updates to the graph.

**Query.**

```Javascript
CYPHER planner=rule
CREATE (:Person { name: 'Alistair' })
```
**Query Plan.**
```
+--------------+------+---------+-----------+------------+
| Operator     | Rows | DB Hits | Variables | Other      |
+--------------+------+---------+-----------+------------+
| +EmptyResult |    0 |       0 |           |            |
| |            +------+---------+-----------+------------+
| +UpdateGraph |    1 |       4 | anon[7]   | CreateNode |
+--------------+------+---------+-----------+------------+

Total database accesses: 4
```
#### 3.7.5.4. 병합

시작과 끝 노드가 이미 발견되면 ```Merge Into```를 사용하여 모든 연결 관계를 찾거나 두 노드 사이에 새로운 관계를 만듭니다.

**Query.**

```Javascript
CYPHER planner=rule
MATCH (p:Person { name: 'me' }),(f:Person { name: 'Andres' })
MERGE (p)-[:FRIENDS_WITH]->(f)
```
**Query Plan.**
```
+--------------+------+---------+------------------+--------------------------------+
| Operator     | Rows | DB Hits | Variables        | Other                          |
+--------------+------+---------+------------------+--------------------------------+
| +EmptyResult |    0 |       0 |                  |                                |
| |            +------+---------+------------------+--------------------------------+
| +Merge(Into) |    1 |       5 | anon[68] -- f, p | (p)-[:FRIENDS_WITH]->(f)       |
| |            +------+---------+------------------+--------------------------------+
| +SchemaIndex |    1 |       2 | f -- p           | {  AUTOSTRING1}; :Person(name) |
| |            +------+---------+------------------+--------------------------------+
| +SchemaIndex |    1 |       2 | p                | {  AUTOSTRING0}; :Person(name) |
+--------------+------+---------+------------------+--------------------------------+

Total database accesses: 9
```
