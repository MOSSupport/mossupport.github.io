### 3.2.1. 값 (Values)

Cypher에서 처리하는 모든 값에는 고유 한 유형이 있습니다. 지원되는 값 유형은 다음과 같습니다.


- 숫자 값(Numeric values),
- 문자열 값(String values),
- 부울 값(Boolean values),
- 노드(Nodes),
- 관계(Relationships),
- 경로(Paths),
- 문자열을 다른 값으로 매핑,
- 다른 타입의 값 리스트.

대부분의 유형의 값은 리터럴 표현식을 사용하여 쿼리에서 생성 될 수 있습니다 ( [3.2.2, "표현식" 참조](./expressions.html)).  ```null```은 모든  유형의 값이므로 ```null``` 을 사용할 때는 특별한 주의가 필요합니다. ([3.2.9, “null 작업” 참조](./working-with-null.html)). 패턴 일치의 결과로 노드(nodes), 관계(relationships) 및 경로(paths)가 리턴됩니다.

레이블은 값이 아니지만 패턴 구문의 한 형태임을 유의하십시오.
