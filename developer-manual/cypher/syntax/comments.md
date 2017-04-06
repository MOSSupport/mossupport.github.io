### 3.2.6. 주석 (Comments)

쿼리에 주석을 추가하려면 이중 슬래시를 사용하십시오. 예 :

```Javascript
MATCH (n) RETURN n //This is an end of line comment
```
```Javascript
MATCH (n) //This is a whole line comment RETURN n
```
```Javascript
MATCH (n) WHERE n.property = '//This is NOT a comment' RETURN n
```
