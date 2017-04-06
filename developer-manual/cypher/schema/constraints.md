### 3.5.2. 제약조건

###### Neo4j는 제약조건을 사용하여 데이터 무결성을 강화합니다. 제약조건은 노드 또는 관계에 적용될 수 있습니다. 고유한 노드 속성 제약조건뿐만 아니라 노드 및 관계 속성 존재 제약조건을 작성할 수 있습니다.

고유한 속성 제약조건을 사용하여 속성 값이 특정 레이블이 있는 모든 노드에 대해 고유한지를 확인할 수 있습니다. 고유 제한 조건은 모든 노드가 특성에 고유 한 값을 가져야 함을 의미하지는 않습니다. 속성이 없는 노드는 이 규칙의 적용을 받지 않습니다.

속성 존재 제약조건을 사용하여 속성 레이블이 있는 모든 노드 또는 특정 유형의 모든 관계에 대해 속성이 존재하는지 확인할 수 있습니다. 속성없이 새 노드 또는 관계를 만들려고 하는 모든 쿼리 또는 필수 속성을 제거하려고 하는 쿼리는 실패하게 됩니다.

속성 존재 제약조건은 Neo4j Enterprise Edition에서만 사용할 수 있습니다. 속성 존재 제약조건이 있는 데이터베이스는 Neo4j Community Edition을 사용하여 열 수 없습니다. 주어진 레이블에 대해 여러 개의 제약 조건을 가질 수 있으며 같은 속성에 고유성과 속성 존재 제약조건을 결합 할 수도 있습니다.

제약조건을 추가하는 것은 잠시 동안 걸릴 수있는 원자적 연산이므로 Neo4j가 제약 조건을 '켜기'전에 모든 기존 데이터를 검사해야 한다는 것을 기억하십시오.

속성에 고유 한 속성 제약조건을 추가하면 해당 속성에 인덱스가 추가되므로 별도로 이러한 인덱스를 추가 할 수 없습니다. Cypher는 다른 인덱스처럼 검색을 위해 해당 인덱스를 사용합니다. 고유 한 특성 제약조건을 제거하고나서도 속성에 대한 인덱스를 필요로 하는 경우, 인덱스를 작성해야합니다.

#### 3.5.2.1. 고유한 노드 속성 제약조건

**고유한 제약조건 생성**
데이터베이스에 특정 레이블과 속성 값이 있는 둘 이상의 노드가 절대로 포함되지 않도록 하는 제약조건을 생성하려면 IS UNIQUE 구문을 사용하십시오.

**Query.**
```Javascript
CREATE CONSTRAINT ON (book:Book) ASSERT book.isbn IS UNIQUE
```
**Result.**
```
+-------------------+
| No data returned. |
+-------------------+
Unique constraints added: 1
```

**고유 제약조건 제거**
###### DROP CONSTRAINT를 사용하여 데이터베이스에서 제약조건을 제거합니다.

**Query.**
```Javascript
DROP CONSTRAINT ON (book:Book) ASSERT book.isbn IS UNIQUE
```
**Result.**
```
+-------------------+
| No data returned. |
+-------------------+
Unique constraints removed: 1
```

---

**고유 한 속성 제약조건을 준수하는 노드 만들기**
데이터베이스에 isbn이 없는 Book 노드를 만듭니다.

**Query.**
```Javascript
CREATE (book:Book { isbn: '1449356265', title: 'Graph Databases' })
```
**Result.**
```
+-------------------+
| No data returned. |
+-------------------+
Nodes created: 1
Properties set: 2
Labels added: 1
```
---
**고유 한 속성 제약조건을 위반하는 노드 만들기**
이미 데이터베이스에서 사용되는 isbn을 사용하여 Book 노드를 만듭니다.

**Query.**
```Javascript
CREATE (book:Book { isbn: '1449356265', title: 'Graph Databases' })
```

이 경우 노드는 그래프에 생성되지 않습니다.

**Error message.**
```
Node 0 already exists with label Book and property "isbn"=[1449356265]
```
---

**충돌하는 노드로 인해 고유한 속성 제약조건을 만들지 못함**
같은 isbn을 가진 두 개의 노드가 있을 때 Book 레이블이 있는 노드의 isbn 속성에 고유한 속성 제약조건을 만듭니다.

**Query.**
```Javascript
CREATE CONSTRAINT ON (book:Book) ASSERT book.isbn IS UNIQUE
```

이 경우 제약조건이 기존 데이터에 위배되므로 생성 할 수 없습니다. 우리는 [3.5.1. "인덱스"]()를 대신 사용하거나 위배되는 노드를 제거한 다음에 제약 조건을 다시 적용 할 수 있습니다.

**Error message.**
```
Unable to create CONSTRAINT ON ( book:Book ) ASSERT book.isbn IS UNIQUE:
Multiple nodes with label `Book` have property `isbn` = '1449356265':
  node(0)
  node(1)
```

#### 3.5.2.2. 노드 속성 존재 제약조건

**노드 속성 존재 제약조건 생성**
특정 레이블이 있는 모든 노드가 특정 속성을 갖도록하는 제약조건을 생성하려면 ASSERT exists (variable.propertyName) 구문을 사용하십시오.

**Query.**
```Javascript
CREATE CONSTRAINT ON (book:Book) ASSERT exists(book.isbn)
```
**Result.**
```
+-------------------+
| No data returned. |
+-------------------+
Property existence constraints added: 1
```

---
**노드 속성 존재 제약조건 삭제**
DROP CONSTRAINT를 사용하여 데이터베이스에서 제약조건을 제거합니다.

**Query.**
```Javascript
DROP CONSTRAINT ON (book:Book) ASSERT exists(book.isbn)
```
**Result.**
```
+-------------------+
| No data returned. |
+-------------------+
Property existence constraints removed: 1
```

---
**속성 존재 제약조건을 준수하는 노드 생성하기**
기존 isbn 속성을 사용하여 Book 노드를 만듭니다.

**Query.**
```Javascript
CREATE (book:Book { isbn: '1449356265', title: 'Graph Databases' })
```
**Result.**
```
+-------------------+
| No data returned. |
+-------------------+
Nodes created: 1
Properties set: 2
Labels added: 1
```

---
**속성 존재 제약조건을 위반하는 노드 생성하기**
Book (isbn)에 대한 속성 존재 제약조건이 주어지면 isbn 속성없이 Book 노드를 만들려고합니다.

**Query.**
```Javascript
CREATE (book:Book { title: 'Graph Databases' })
```
이 경우 노드는 그래프에 생성되지 않습니다.

**Error message.**
```
Node 1 with label "Book" must have the property "isbn" due to a constraint
```

---
**존재 제약된 노드 속성 제거**
Book (isbn)에 대한 속성 존재 제약조건이 주어지면 기존 노드 Book에서 isbn 속성을 제거하려고 합니다.

**Query.**
```Javascript
MATCH (book:Book { title: 'Graph Databases' })
REMOVE book.isbn
```
이 경우 속성은 제거되지 않습니다.

**Error message.**
```
Node 0 with label "Book" must have the property "isbn" due to a constraint
```

---
**기존 노드로 인해 노드 속성 존재 제약조건을 생성하지 못했습니다.**
isbn이없는 노드가 이미 있을 경우 Book 레이블이 있는 노드의 isbn 등록 정보에 대한 제약조건을 생성하십시오.

**Query.**
```Javascript
CREATE CONSTRAINT ON (book:Book) ASSERT exists(book.isbn)
```
이 경우 제약조건은 기존 데이터에 위배되므로 생성 할 수 없습니다. 위반 노드를 제거한 다음 제약조건을 다시 적용하도록 선택할 수 있습니다.

**Error message.**
```
Unable to create CONSTRAINT ON ( book:Book ) ASSERT exists(book.isbn):
Node(0) with label `Book` has no value for property `isbn`
```
#### 3.5.2.3. 관계 속성 존재 제약조건

**관계 속성 존재 제약조건 생성**
특정 유형의 모든 관계에 특정 속성 있는지 확인하는 제약조건을 생성하려면 ASSERT exists (variable.propertyName) 구문을 사용하십시오.

**Query.**
```Javascript
CREATE CONSTRAINT ON ()-[like:LIKED]-() ASSERT exists(like.day)
```
**Result.**
```
+-------------------+
| No data returned. |
+-------------------+
Property existence constraints added: 1
```

---
**관계 속성 존재 제약조건 삭제**
데이터베이스에서 제약조건을 제거하려면 DROP CONSTRAINT를 사용하십시오.

**Query.**
```Javascript
DROP CONSTRAINT ON ()-[like:LIKED]-() ASSERT exists(like.day)
```
**Result.**
```
+-------------------+
| No data returned. |
+-------------------+
Property existence constraints removed: 1
```

---
**속성 존재 제약조건을 준수하는 관계 만들기**
기존의 day 속성과 LIKED 관계를 만듭니다.

**Query.**
```Javascript
CREATE (user:User)-[like:LIKED { day: 'yesterday' }]->(book:Book)
```
**Result.**
```
+-------------------+
| No data returned. |
+-------------------+
Nodes created: 2
Relationships created: 1
Properties set: 1
Labels added: 2
```

---
**속성 존재 제한조건을 없애는 관계를 만듭니다.**
:LIKED(day) 속성 존재 제약조건이 주어진 경우, 하루 속성없이 LIKED 관계를 만들려고 합니다.

**Query.**
```Javascript
CREATE (user:User)-[like:LIKED]->(book:Book)
```
이 경우 관계는 그래프에 생성되지 않습니다.

**Error message.**
```
Relationship 1 with type "LIKED" must have the property "day" due to a constraint
```

---
**제약이 있는 존재 관계 속성 제거하기**
LIKED 유형 (:LIKED(day))과 같은 기존 존재 관계에서 day 속성을 제거하려고 하였습니다.

**Query.**
```Javascript
MATCH (user:User)-[like:LIKED]->(book:Book)
REMOVE like.day
```
이 경우 속성은 제거되지 않습니다.

**Error message.**
```
Relationship 0 with type "LIKED" must have the property "day" due to a constraint
```

---
**기존 관계로 인해 관계 속성 존재 제약의 생성실패**
day라는 속성없이 관계가 이미 존재하는 경우, LIKED 유형과의 관계에서 속성 day에 대한 제약조건을 만듭니다.

**Query.**
```Javascript
CREATE CONSTRAINT ON ()-[like:LIKED]-() ASSERT exists(like.day)
```
이 경우 제약조건은 기존 데이터에 위배되므로 생성 할 수 없습니다. 위반 관계를 제거한 다음 제약조건을 다시 적용 할 수도 있습니다.

**Error message.**
```
Unable to create CONSTRAINT ON ()-[ liked:LIKED ]-() ASSERT exists(liked.day):
Relationship(0) with type `LIKED` has no value for property `day`
```
