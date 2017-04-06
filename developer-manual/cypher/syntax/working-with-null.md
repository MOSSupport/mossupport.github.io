### 3.2.9. null 작업 (Working with null)

#### 3.2.9.1. Cypher 에서 'null' 소개

Cypher에서 누락되거나 정의되지 않은 값을 나타내는 데 `null` 이 사용됩니다.  개념적으로, `null` 은 '누락 된 값을 알 수 없음'을 의미하며 다른 값과는 다소 다르게 처리됩니다. For example getting a property from a node that does not have said property produces null. `null` 을 입력으로 사용하는 대부분의 표현식은 `null` 을 생성합니다. 여기에는 `WHERE` 절의 술어로 사용되는 부울 표현식이 포함됩니다. 이 경우 `true`가 아닌 모든 것은 false로 해석됩니다.

`null` 는 `null` 와 동일하지 않습니다. 두 값을 모르는 것이 그것들이 같은 값이라는 것을 의미하지는 않습니다. 따라서 `null` = `null` 표현식은 `null`이 되고 `true`는 아닙니다.


#### 3.2.9.2. null 논리연산

논리연산자(`AND`, `OR`, `XOR`, `IN`, `NOT`)는 `null`은 3의 값인 'unknown'입니다. 다음은 `AND` , `OR` 및 `XOR` 에 대한 진리표입니다.

|     a    |     b    |   a `AND` b  |   a `AND` b  |   a `XOR` b  |
|----------|----------|--------------|--------------|--------------|
| `false`  | `false`  |   `false`    |   `false`    |   `false`    |
| `false`  | `null`   |   `false`    |   `null`     |   `null`     |
| `false`  | `true`   |   `false`    |   `true`     |   `true`     |
| `true`   | `false`  |   `false`    |   `true`     |   `true`     |
| `true`   | `null`   |   `null`     |   `true`     |   `null`     |
| `true`   | `true`   |   `true`     |   `true`     |   `false`    |
| `null`   | `false`  |   `false`    |   `null`     |   `null`     |
| `null`   | `null`   |   `null`     |   `null`     |   `null`     |
| `null`   | `true`   |   `null`     |   `true`     |   `null`     |

#### 3.2.9.3. IN 연산자와 null

`IN` 연산자도 비슷한 논리를 따릅니다. 사이퍼 (Cypher)가 목록에 무엇인가 존재한다는 것을 안다면 그 결과는 true 것이다. null 를 포함하거나 일치하는 요소를 가지지 않는 리스트는 `null`을 리턴합니다. 그렇지 않으면 결과가 false가 됩니다. 다음은 예제가있는 표입니다.

| 표    현                 |  결   과 |
|--------------------------|----------|
| 2 IN [1, 2, 3]           | `true`   |
| 2 IN [1, `null`, 3]      | `null`   |
| 2 IN [1, 2, null]        | `true`   |
| 2 IN [1]                 | `false`  |
| 2 IN []                  | `false`  |
| `null` IN [1, 2, 3]      | `null`   |
| `null` IN [1, null, 3]   | `null`   |
| `null` IN []             | `false`  |

`all` , `any` , `none` 및 `single` 하면 비슷한 규칙이 적용됩니다. 결과를 정확하게 계산할 수 있으면 `true` 또는 `false` 가 반환됩니다. 그렇지 않으면 null 이 생성됩니다.


#### 3.2.9.4. null 을 리턴하는 표현식

- 목록에서 누락 된 요소 가져 오기 : `[][0]`, `head([])`
- 노드 또는 관계에없는 속성에 액세스하려고합니다: `n.missingProperty`
- 양쪽이 `null` 일 때 : `1 < null`
- `null`을 포함하는 산술식 : `1 + null`
- 인수가 `null` 함수 호출 : `sin(null)`
