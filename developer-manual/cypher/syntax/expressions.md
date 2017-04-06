### 3.2.2. 표현식 (Expressions)

#### 3.2.2.1. 일반적인 표현식

Cypher의 표현식은 다음과 같습니다:

숫자 (정수와 더블) 리터럴: `13`, `-40000`, `3.14`, `6.022E23`.
16진수 리터럴 (0x로 시작): `0x13zf`, `0xFC3A9`, `-0x66eff`.
8진수 리터럴 (0으로 시작): `01372`, `02127`, `-05671`.
문자열 리터럴: `'Hello'`, `"World"`.
부울 리터럴: `true`, `false`, `TRUE`, `FALSE`.
변수: `n`, `x`, `rel`, `myFancyVariable`, <code>\`A name with weird stuff in it[]!\`</code>.
속성: `n.prop`, `x.prop`, `rel.thisProperty`, `myFancyVariable.` <code>\`(weird property name)\`</code>.
동적 속성: `n["prop"]`, `rel[n.city + n.zip]`, `map[coll[0]]`.
파라미터: `$param`,`$0`
표현식 리스트: `['a', 'b']`, `[1, 2, 3]`, `['a', 2, n.property, $param]`, `[ ]`.
함수호출: `length(p)`, `nodes(p)`.
집계함수: `avg(x.prop)`, `count(*)`.
패스패턴: `(a)-→()←-(b)`.
연산자 응용: `1 + 2` 및 `3 < 4`.
술어 표현식은 true 또는 false를 리턴하는 표현식입니다.: `a.prop = 'Hello'`, `length(p) > 10`, `exists(a.name)`.
정규 표현식: `a.name =~ 'Tob.*'`
대소문자를 구분하는 문자 표현식: `a.surname STARTS WITH 'Sven'`, `a.surname ENDS WITH 'son'`, `a.surname CONTAINS 'son'`
`CASE` 표현식.

#### 3.2.2.2. 문자열 리터럴에 대한 참고사항

문자열 리터럴은 아래와 같은 이스케이프 시퀀스를 포함 할 수 있습니다.

| 이스케이프 시퀀스             | 캐릭터                          |
|--------------------|------------------|
| `\t`                                 | Tab                             |
| `\b`                                | Backspace                  |
| `\n`                                | Newline                      |
| `\r`                                 | Carriage return           |
| `\f`                                 | Form feed                  |
| `\'`                                  | Single quote              |
| `\"`                                 | Double quote             |
| `\\`                                 | Backslash                   |
| `\uxxxx`                         | Unicode UTF-16 (`\u`로 시작하는 4 자리 16진수) |
| `\Uxxxxxxxx`                  | Unicode UTF-32 (`\U`로 시작하는 8 자리 16진수) |

#### 3.2.2.3. Case 표현식

Cypher는 다른 언어의 if/else와 비슷한 일반 조건식인 `CASE` 표현식을 제공합니다. `CASE`에는 단순한 CASE와 일반 CASE 두가지 형식이 있습니다. .

##### 단순한 CASE

표현식이 계산되고 일치 항목이 발견 될 때까지 순서대로 `WHEN` 절과 비교됩니다. 일치하는 항목이 없으면 `ELSE` 절의 표현식이 사용되며 `ELSE`의 경우가 없을 때는 `null` 이 사용됩니다.

**문법**:

```Javascript
CASE test
WHEN value THEN result
[WHEN ...]
[ELSE default]
END
```

**인수**:

|Name                   | 	Description                                           |
|--------------|--------------------------------|
| `test`                   |             유용한 표현식                               |
| `value`                 | 결과가 `test` 표현식과 비교 될 표현식입니다. |
| `result`                |값 표현식이 `test` 표현식과 일치하는 경우 사용되는 결과 표현식입니다.|
| `default`             | 일치하는 항목이 없으면 사용할 표현식입니다. |

**쿼리**
```Javascript
MATCH (n)
RETURN
CASE n.eyes
WHEN 'blue'
THEN 1
WHEN 'brown'
THEN 2
ELSE 3 END AS result
```
**결과값.**
```
+--------+
| result |
+--------+
| 2      |
| 1      |
| 3      |
| 2      |
| 1      |
+--------+
5 rows
```
#### 일반적인 CASE

술어는 실제 값이 발견 될 때까지 순서대로 평가되고 결과 값이 사용됩니다. 일치하는 항목이 없으면 `ELSE` 절의 표현식이 사용되며 `ELSE`의 경우가 없을 때는 `null` 이 사용됩니다.

**문법**:

```Javascript
CASE
WHEN predicate THEN result
[WHEN ...]
[ELSE default]
END
```
**인수**:

|Name                   | 	Description                                           |
|--------------|--------------------------------|
| `predicate`          | 유효한 대안을 찾기 위해 테스트 된 술어. |
| `result`                | 술어가 일치하는 경우 사용되는 결과 표현식입니다. |
| `default`             | 일치하는 항목이 없으면 사용할 표현식입니다. |


**쿼리**
```Javascript
MATCH (n)
RETURN
CASE
WHEN n.eyes = 'blue'
THEN 1
WHEN n.age < 40
THEN 2
ELSE 3 END AS result
```
**결과값.**
```
+--------+
| result |
+--------+
| 2      |
| 1      |
| 3      |
| 3      |
| 1      |
+--------+
5 rows
```
