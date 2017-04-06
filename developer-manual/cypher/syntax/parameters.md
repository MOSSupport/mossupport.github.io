### 3.2.4. 매개변수 (Parameters)

Cypher는 매개변수를  사용하는 쿼리를 지원합니다. 이는 개발자가 쿼리를 작성하기 위해 문자열 작성에 의존하지 않아도 된다는 것을 의미합니다. 또한 매개 변수를 사용하면 Cypher에서 실행 계획을 훨씬 쉽게 캐싱 할 수 있으므로 쿼리 실행 시간이 빨라집니다.

매개 변수는 다음에 사용될 수 있습니다.

- 리터럴 및 표현식
- 노드 및 관계 id
- 레거시 인덱스 전용: 인덱스 값 및 쿼리

매개 변수는 쿼리 계획으로 컴파일되는 쿼리 구조의 일부이므로 다음 구문에 사용할 수 없습니다.

- 속성키;  `MATCH (n) WHERE n.$param = 'something'` 는 틀린 구문입니다.
- 관계유형
- 레이블

매개 변수는 문자와 숫자 및 이들의 조합으로 구성 될 수 있지만 숫자나 통화 기호로 시작할 수 없습니다.

Neo4j REST API를 통한 매개 변수 사용에 대한 자세한 내용은 [5.1. "트랜잭션 Cypher HTTP 엔드포인트"](http://neo4j.com/docs/developer-manual/current/http-api/#http-api-transactional)를 참조하십시오.

아래에서는 매개 변수 사용에 대한 포괄적인 목록을 제공합니다. 이 예제에서 매개 변수는 JSON으로 제공됩니다. 제출 방법은 드라이버에 따라 다릅니다.

<table width=1200><tr><td><i class="fa icon-alert-circle" title="important"></i><img src="./img/note.gif" width="70"></td><td width=90%>이전 구문 <code>{param}</code> 은 더 이상 사용되지 않으며 이후 릴리스에서 모두 제거되므로 새로운 <code>$param</code> 구문을 사용되는 것이 좋습니다.
</td></tr><table>

#### 3.2.4.1. 문자열 리터럴

**매개변수**.
```
{
  "name" : "Johan"
}
```
**쿼리**.
```Javascript
MATCH (n:Person)
WHERE n.name = $name
RETURN n
```

이 구문에서도 매개 변수를 사용할 수 있습니다.:

**매개변수**.
```
{
  "name" : "Johan"
}
```

**쿼리**.
```Javascript
MATCH (n:Person { name: $name })
RETURN n
```

#### 3.2.4.2. 정규식

**매개변수**.
```
{
  "regex" : ".*h.*"
}
```

**쿼리**.
```Javascript
MATCH (n:Person)
WHERE n.name =~ $regex
RETURN n.name
```

#### 3.2.4.3. 대소문자를 구분하는 문자열 패턴 일치

**매개변수**.
```
{
  "name" : "Michael"
}
```

**쿼리**.
```Javascript
MATCH (n:Person)
WHERE n.name STARTS WITH $name
RETURN n.name
```

#### 3.2.4.4. 속성을 사용하여 노드 만들기

**매개변수**.
```
{
  "props" : {
    "name" : "Andres",
    "position" : "Developer"
  }
}
```

**쿼리**.
```Javascript
CREATE ($props)
```

#### 3.2.4.5. 속성을 사용하여 여러 노드 만들기

**매개변수**.
```
{
  "props" : [ {
    "awesome" : true,
    "name" : "Andres",
    "position" : "Developer"
  }, {
    "children" : 3,
    "name" : "Michael",
    "position" : "Developer"
  } ]
}
```

**쿼리**.
```Javascript
UNWIND $props AS properties
CREATE (n:Person)
SET n = properties
RETURN n
```

#### 3.2.4.6. 노드의 모든 속성 설정하기

Note that this will replace all the current properties.
현재의 모든 속성을 대체 할 것 입니다.


**매개변수**.
```
{
  "props" : {
    "name" : "Andres",
    "position" : "Developer"
  }
}
```

**쿼리**.
```Javascript
MATCH (n:Person)
WHERE n.name='Michaela'
SET n = $props
```

#### 3.2.4.7. SKIP 과 LIMIT

**매개변수**.
```
{
  "s" : 1,
  "l" : 1
}
```

**쿼리**.
```Javascript
MATCH (n:Person)
RETURN n.name
SKIP $s
LIMIT $l
```

#### 3.2.4.8. 노드 id

**매개변수**.
```
{
  "id" : 0
}
```

**쿼리**.
```Javascript
MATCH (n)
WHERE id(n)= $id
RETURN n.name
```

#### 3.2.4.9. 여러노드 id

**매개변수**.
```
{
  "ids" : [ 0, 1, 2 ]
}
```

**쿼리**.
```Javascript
MATCH (n)
WHERE id(n) IN $ids
RETURN n.name
```

#### 3.2.4.10. 프로시저 호출

**매개변수**.
```
{
  "indexname" : ":Person(name)"
}
```

**쿼리**.
```
CALL db.resampleIndex($indexname)
```

#### 3.2.4.11. 인덱스 값 (레거시 인덱스)

**매개변수**.
```
{
  "value" : "Michaela"
}
```

**쿼리**.
```Javascript
START n=node:people(name = $value)
RETURN n
```

#### 3.2.4.12. 인덱스쿼리 (레거시 인덱스)

**매개변수**.
```
{
  "query" : "name:Andreas"
}
```

**쿼리**.
```Javascript
START n=node:people($query)
RETURN n
```
