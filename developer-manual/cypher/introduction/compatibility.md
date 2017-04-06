### 3.1.5. 적합성 (Compatibility)

Cypher는 정적 언어가 아닙니다. 새 버전에서는 새로운 기능이 도입되고 때로는 오래된 기능이 삭제되는 경우가 있습니다. 필요한 경우 이전 버전의 언어에 계속 액세스 할 수 있습니다. 쿼리에서 사용할 버전을 선택하는 방법에는 두 가지가 있습니다.

1.  `cypher.default_language_version` 설정 매개 변수로 모든 쿼리에 대한 버전 설정하여 데이터베이스를 구성하고 사용할 버전을 입력 할 수 있습니다 ( 3.1.5.2 절. "지원 언어 버전" 참조). 아래의 다음 항목에서 설명한대로 쿼리가 명시 적으로 구성되지 않았다면 모든 Cypher 쿼리는이 버전을 사용합니다.

2. 쿼리 단위로 쿼리에 버전 설정 다른 방법은 쿼리 단위의 쿼리입니다. 단순히 `CYPHER 2.3` 을 시작 부분에두면 Neo4j 2.3에 포함 된 Cypher 버전으로 특정 쿼리가 실행됩니다.

다음은 `START` 절을 사용하여 레거시 인덱스에 액세스하는 예제입니다.
```Javascript
CYPHER 2.3
START n=node:nodes(name = "A")
RETURN n
```

#### 3.1.5.1. START를 통해 ID로 엔티티에 액세스하기

2.2 이전의 Cypher 버전에서는 `START` 절을 사용하여 특정 노드 또는 관계에 액세스 할 수도 있었습니다. 이 경우 다음과 같은 구문을 사용할 수 있습니다.
```Javascript
CYPHER 1.9
START n=node(42)
RETURN n
```
<table width=1200><tr><td><i class="fa icon-alert-circle" title="important"></i><img src="./img/note.gif" width="60"></td><td width=90%>ID로 노드를 찾기 위해 `START` 절을 사용하는 것은 Cypher 2.0부터 사용되지 않으며 이제는 Cypher 2.2 이상에서 완전히 비활성화됩니다. 대신 시작점에 MATCH 절을 사용해야합니다. 이에 대한 올바른 구문에 대한 자세한 내용은 <a href="https://neo4j.com/docs/developer-manual/3.1/cypher/clauses/match/">3.3.1 절, "MATCH"</a> 를 참조하십시오. START 절은 레거시 색인에 액세스 할 때만 사용해야합니다.
</td></tr><table>

#### 3.1.5.2. 지원되는 언어 버전

Neo4j 3.1은 다음 버전의 Cypher 언어를 지원합니다.

- Neo4j Cypher 3.1sl
- Neo4j Cypher 3.0
- Neo4j Cypher 2.3

<table width=1200><tr><td><img src="./img/note.gif" width="60"><i class="fa icon-alert-circle" title="important"></i></td><td width=90%>Neo4j의 각 릴리스는 제한된 수의 이전 Cypher 언어 버전을 지원합니다. Neo4j의 새 버전으로 업그레이드 할 때 필요한 Cypher 언어 버전을 지원하는지 확인하십시오. 그렇지 않은 경우 최신 Cypher 언어 버전으로 작업하려면 쿼리를 수정해야 할 수 있습니다.
</td></tr><table>
