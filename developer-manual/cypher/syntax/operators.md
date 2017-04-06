### 3.2.5. 연산자 (Operators)

#### 3.2.5.1. 수학 연산자

수학 연산자는 `+`, `-`, `*`, `/`, `%`, `^` 입니다..

#### 3.2.5.2. 비교 연산자

비교 연산자는  `=`, `<>`, `<`, `>`, `<=`, `>=`, `IS NULL` 및 `IS NOT NULL` 입니다. 동작방식은 [3.2.5.7, “값의 동일성 및 비교”](#3257-값의-동일성-및-비교)를 참조하십시오.

연산자 `STARTS WITH`, `ENDS WITH` 그리고 `CONTAINS`을 사용하여 내용의 문자열 값을 검색할 수 있습니다.

#### 3.2.5.3. 부울 연산자

부울 연산자는  `AND`, `OR`, `XOR` 및 `NOT` 입니다..

#### 3.2.5.4. 문자열 연산자

문자열은 `+`연산자를 사용하여 연결 할 수 있습니다. 정규식 일치의 경우 `=~` 연산자가 사용됩니다.

#### 3.2.5.5. 리스트 연산자

리스트는 `+`연산자를 사용하여 연결 할 수 있습니다. 요소가 목록에 있는지 확인하려면 `IN` 연산자를 사용할 수 있습니다.

#### 3.2.5.6. 속성 연산자

Since version 2.0, the previously existing property operators ? and ! have been removed. This syntax is no longer supported. Missing properties are now returned as null. Please use (NOT(has(<ident>.prop)) OR <ident>.prop=<value>) if you really need the old behavior of the ? operator. — Also, the use of ? for optional relationships has been removed in favor of the newly introduced OPTIONAL MATCH clause.

#### 3.2.5.7. 값의 같음 및 비교

**같음**

Cypher는 `=` 및 `<>` 연산자를 사용하여 동등한 값 ( [3.2.1, "값" 참조](./values.html)) 비교를 지원합니다.

동일한 유형의 값은 동일한 형태의 값 (예 : `3 = 3` 및 ``"x" <> "xy"`` ) 인 경우에만 동일합니다.

맵은 같은 키에 같은 값이 정확하게 매핑하는 동일 시퀀스의 리스트(예. `[3, 4] = [1+2, 8/2]`) 일 경우 동일합니다.

다른 유형의 값은 다음 규칙에 따라 동일하게 간주됩니다.:

- 경로는 노드와 관계가 교대로 반복되는 리스트로 취급되며 노드와 관계의 동일한 순서를 포함하는 모든 리스트와 동일합니다.
- `=` 및 `<>` 연산자로 항상 `null` 에 대한 값을 테스트하면 항상 `null` 입니다. 여기에는 `null = null` 및 `null <> null` 이 포함되어 있습니다. `v`가 `null`인지 확실하게 확인하는 방법은 `v IS NULL` 또는 `v IS NOT NULL` 의 연산자를 이용하는 것 입니다.

값 유형의 다른 모든 조합은 서로 비교할 수 없습니다. 특히, 노드, 관계 및 리터럴 맵은 서로 비교할 수 없습니다.

비교할 수없는 값을 비교하는 것은 오류입니다.

#### 3.2.5.8. 값의 순서와 비교

비교연산자 `<=`, `<` (오름차순) 와 `>=`, `>` (내림차순)은 정렬 값을 비교하는데 사용됩니다. 다음은 비교가 수행되는 방법데 대한 세부 사항입니다.

- 숫자값을 순서대로 사용하여 수치 값을 비교합니다. (예. `3 < 4` 는 참).
- 특수 값 `java.lang.Double.NaN` 은 다른 모든 숫자보다 큰 것으로 간주됩니다.
- 문자열 값은 사전 식 순서 (예 : ``"x" < "xy"``)를 사용하여 순서를 비교합니다.
- 부울 값은 `false < true` 와 같은 순서로 비교됩니다.
- 비교하는 인수중 하나가 `null`일 때의 비교 (예. `null < 3` 은 `null`).
- 다른 유형의 값을 순서대로 비교하는 것은 오류입니다.

#### 3.2.5.9. 연결된 비교 연산

비교는 임의로 연결 될 수 있습니다, 예를 들어  `x < y <= z` 는 `x < y AND y <= z` 와 동일합니다.

공식적으로 `a, b, c, ..., y, z`가 표현식이고 `op1, op2, ..., opN` 이 비교연산자인 `a op1 b op2 c ... y opN` 는 `op1 b and b op2 c and ... y opN z` 와 동일합니다..

 `a op1 b op2 c` 에서  `a` 와 `c` 사이의 어떤 종류의 비교도 의미하지 않는다, 예를 들어 `x < y > z`는 (보기에 좋지는 않지만) 정상이다. .

예:
```Javascript
MATCH (n) WHERE 21 < n.age <= 30 RETURN n
```
다음과 같습니다.

```Javascript
MATCH (n) WHERE 21 < n.age AND n.age <= 30 RETURN n
```

따라서 나이가 21에서 30 사이인 모든 노드와 일치합니다.


이 구문은 모든 평등 및 불평등 비교뿐만 아니라 3보다 긴 체인까지 확장됩니다.


예:
```
a < b = c <= d <> e
```
다음과 같습니다.:
```
a < b AND b = c AND c <= d AND d <> e
```
다른 비교 연산자에 대해서는 [3.2.5.2 절. "비교 연산자"](#3252-비교-연산자)를 참조하십시오.
