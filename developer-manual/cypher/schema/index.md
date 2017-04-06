### 3.5.1. 인덱스

데이터베이스 인덱스는 데이터를 보다 효율적으로 검색하기 위한 목적으로 데이터베이스에 중복된 정보사본입니다. 이는 추가 저장 공간과 더 느린 쓰기를 필요로하므로 인덱스 할 항목과 인덱스하지 않을 항목을 결정하는 것이 중요하고 종종 사소하기도 한 작업입니다.

Cypher를 사용하면 기존의 레이블이 있는 모든 노드의 속성에 대한 인덱스를 생성할 수 있습니다. 일단 인덱스가 생성되면 그래프가 변경 될 때마다 데이터베이스에 의해 자동으로 관리되고 최신 상태로 유지됩니다. Neo4j는 일단 인덱스가 생성되고 온라인화되면 인덱스를 자동으로 선택하여 사용합니다.

#### 3.5.1.1. 인덱스 생성

레이블이 있는 모든 노드의 특성에 인덱스를 작성하려면 CREATE INDEX ON을 사용하십시오. 인덱스는 즉시 사용할 수는 없지만 백그라운드에 생성됩니다.

**쿼리(Query).**
```Javascript
CREATE INDEX ON :Person(name)
```
**결과.**
```
+-----------------------------------------------------------+
| No data returned, and nothing was changed. |
+-----------------------------------------------------------+
```
#### 3.5.1.2. 인덱스 제거

레이블과 속성 조합이 있는 모든 노드에서 인덱스를 삭제하려면 DROP INDEX 절을 사용하시기 바랍니다.

**쿼리(Query).**
```Javascript
DROP INDEX ON :Person(name)
```
**결과.**
```
+-------------------+
| No data returned. |
+-------------------+
Indexes removed: 1
```
#### 3.5.1.3. 일반적인 쿼리(Query)의 인덱스 사용

일반적으로 쿼리에 사용할 인덱스를 지정할 필요가 없으므로 Cypher는 그 인덱스를 알아낼 것입니다. 예를 들어, 아래의 쿼리는 Person (이름) 인덱스가 있는 경우 해당 인덱스를 사용합니다. Cypher가 특정 인덱스를 사용하게 하려면 힌트를 사용하여 시행하십시오. [3.6.4. "사용하기"](../query-tuning/using.html) 참조.

**쿼리(Query).**
```Javascript
MATCH (person:Person { name: 'Andres' })
RETURN person
```
**Query Plan.**
```
+-----------------+----------------+------+---------+-----------+---------------+
| Operator        | Estimated Rows | Rows | DB Hits | Variables | Other         |
+-----------------+----------------+------+---------+-----------+---------------+
| +ProduceResults |              1 |    1 |       0 | person    | person        |
| |               +----------------+------+---------+-----------+---------------+
| +NodeIndexSeek  |              1 |    1 |       2 | person    | :Person(name) |
+-----------------+----------------+------+---------+-----------+---------------+
Total database accesses: 2
```

#### 3.5.1.4. WHERE절의 항등비교(Equality)를 이용한 인덱스 사용

인덱스는 WHERE 절의 인덱스 된 속성에 대한 동등한 비교를 위해 자동으로 사용됩니다. Cypher가 특정 인덱스를 사용하게 하려면 힌트를 사용하여 시행하십시오. [3.6.4. "USING"](../query-tuning/using.html)를보십시오.

**쿼리(Query).**
```Javascript
MATCH (person:Person)
WHERE person.name = 'Andres'
RETURN person
```
**Query Plan.**
```
+-----------------+----------------+------+---------+-----------+---------------+
| Operator        | Estimated Rows | Rows | DB Hits | Variables | Other         |
+-----------------+----------------+------+---------+-----------+---------------+
| +ProduceResults |              1 |    1 |       0 | person    | person        |
| |               +----------------+------+---------+-----------+---------------+
| +NodeIndexSeek  |              1 |    1 |       2 | person    | :Person(name) |
+-----------------+----------------+------+---------+-----------+---------------+
Total database accesses: 2
```

#### 3.5.1.5. WHERE절의 비항등비교(Inequality)를 이용한 인덱스 사용

또한 인덱스는 WHERE 절의 인덱스 된 속성의 비항등 (범위) 비교에 자동으로 사용됩니다. Cypher가 특정 인덱스를 사용하게 하려면 힌트를 사용하여 시행하십시오. [3.6.4. "USING"](../query-tuning/using.html) 참고.

**쿼리(Query).**
```Javascript
MATCH (person:Person)
WHERE person.name > 'B'
RETURN person
```
**Query Plan.**
```
+-----------------------+----------------+------+---------+-----------+---------------------------------+
| Operator              | Estimated Rows | Rows | DB Hits | Variables | Other                           |
+-----------------------+----------------+------+---------+-----------+---------------------------------+
| +ProduceResults       |             10 |    1 |       0 | person    | person                          |
| |                     +----------------+------+---------+-----------+---------------------------------+
| +NodeIndexSeekByRange |             10 |    1 |       2 | person    | :Person(name) > {  AUTOSTRING0} |
+-----------------------+----------------+------+---------+-----------+---------------------------------+
Total database accesses: 2
```

#### 3.5.1.6. IN을 이용한 인덱스 사용

다음 쿼리(Query)의 person.name에 대한 IN은 인덱스가 있을 경우, Person (이름) 인덱스를 사용합니다. Cypher가 특정 인덱스를 사용하게 하려면 힌트를 사용하여야 합니다. [3.6.4. "USING"](../query-tuning/using.html) 참조.

**쿼리(Query).**
```Javascript
MATCH (person:Person)
WHERE person.name IN ['Andres', 'Mark']
RETURN person
```
**Query Plan.**
```
+-----------------+----------------+------+---------+-----------+---------------+
| Operator        | Estimated Rows | Rows | DB Hits | Variables | Other         |
+-----------------+----------------+------+---------+-----------+---------------+
| +ProduceResults |             24 |    2 |       0 | person    | person        |
| |               +----------------+------+---------+-----------+---------------+
| +NodeIndexSeek  |             24 |    2 |       4 | person    | :Person(name) |
+-----------------+----------------+------+---------+-----------+---------------+
Total database accesses: 4
```

#### 3.5.1.7. STARTS WITH을 이용한 인덱스 사용

다음 쿼리(Query)에서 person.name에 대한 STARTS WITH는 Person (이름) 인덱스가 있으면 이를 사용합니다.

**쿼리(Query).**
```Javascript
MATCH (person:Person)
WHERE person.name STARTS WITH 'And'
RETURN person
```
**Query Plan.**
```
+-----------------------+----------------+------+---------+-----------+-------------------------------------------+
| Operator              | Estimated Rows | Rows | DB Hits | Variables | Other                                     |
+-----------------------+----------------+------+---------+-----------+-------------------------------------------+
| +ProduceResults       |             26 |    1 |       0 | person    | person                                    |
| |                     +----------------+------+---------+-----------+-------------------------------------------+
| +NodeIndexSeekByRange |             26 |    1 |       2 | person    | :Person(name STARTS WITH {  AUTOSTRING0}) |
+-----------------------+----------------+------+---------+-----------+-------------------------------------------+
Total database accesses: 2
```

#### 3.5.1.8. 인덱스를 사용한 속성의 존재 확인

다음 쿼리(Query)의 has (p.name)는 Person (이름) 인덱스가 있는 경우, 이를 사용합니다.

**쿼리(Query).**
```Javascript
MATCH (p:Person)
WHERE exists(p.name)
RETURN p
```
**Query Plan.**
```
+-----------------+----------------+------+---------+-----------+---------------+
| Operator        | Estimated Rows | Rows | DB Hits | Variables | Other         |
+-----------------+----------------+------+---------+-----------+---------------+
| +ProduceResults |              2 |    2 |       0 | p         | p             |
| |               +----------------+------+---------+-----------+---------------+
| +NodeIndexScan  |              2 |    2 |       3 | p         | :Person(name) |
+-----------------+----------------+------+---------+-----------+---------------+
Total database accesses: 3
```
