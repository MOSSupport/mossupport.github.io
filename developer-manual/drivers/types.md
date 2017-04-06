## 4.6. 유형
###### *<table width=1200><td width=1200>드라이버는 해당 언어의 유형 시스템과 Neo4j에서 사용되는 유형 시스템 사이를 해석합니다.</td></table>*

Neo4j는 데이터 처리 및 저장을 위한 데이터 유형 시스템을 갖추고 있습니다. 드라이버는 어플리케이션 언어 유형과 Neo4j 유형 시스템 간에 변환을 해줍니다. 파라미터를 전달하고 결과를 처리하려면 Neo4j 유형 시스템의 기본 사항을 알고 드라이버에서 Neo4j 유형이 매핑되는 방식을 이해하는 것이 중요합니다.

다음 Neo4j 유형은 파라미터와 결과 모두에 유효합니다:

- Boolean
- Integer
- Float
- String
- List
- Map

맵과 목록은 데이터베이스에 의해 처리되어 명령문에서 사용될 수 있습니다. 목록(엘리먼트가 하나의 원시 Neo4j 유형을 공유하는)은 노드 및 관계에 대한 속성으로 저장될 수 있습니다. 엘리먼트가 혼합 된 유형을 지닌 맵 및 목록은 속성으로 저장될 수 없습니다.

파라미터와 결과 모두에 유효한 유형 이외에, 다음 Neo4j 유형은 오직 결과에만 유효합니다:

- Node
- Relationship
- Path

Neo4j 드라이버는 다음과 같이 Neo4j 유형을 자국어 유형에 매핑합니다.

<font color="gray">예제 4.22. Neo4j 유형을 자국어 유형에 맵핑하기</font>

C#
<table><tr><td width="1200">
<table><tr><td><b>Neo4j</b></td><td><b>.NET</b></td></tr>
<tr><td><font color="red">Null</font></td>
<td><font color="red">null</font></td></tr>
<tr><td><font color="red">Boolean</font></td>
<td><font color="red">System.Boolean</font></td>
<tr><td><font color="red">Integer</font></td>
<td><font color="red">System.Int64</font></td>
<tr><td><font color="red">Float</font></td>
<td><font color="red">System.Double</font></td>
<tr><td><font color="red">String</font></td>
<td><font color="red">System.String</font></td>
<tr><td><font color="red">List</font></td>
<td><font color="red">System.Collections.Generic.IList<<font>T</font>></font></td>
<tr><td><font color="red">Map</font></td>
<td><font color="red">System.Collections.IDictionary<<font>TKey, TValue</font>></font></td>
<tr><td><font color="red">Node</font></td>
<td><font color="red">Neo4j.Driver.V1.INode</font></td>
<tr><td><font color="red">Relationship</font></td>
<td><font color="red">Neo4j.Driver.V1.IRelationship</font></td>
<tr><td><font color="red">Path</font></td>
<td><font color="red">Neo4j.Driver.V1.IPath</font></td></tr></table>
</td></tr></table>

Java
<table><tr><td width="1200">
<table><tr><td><b>Neo4j</b></td><td><b>Java</b></td></tr>
<tr><td><font color="red">Null</font></td>
<td><font color="red">null</font></td></tr>
<tr><td><font color="red">Boolean</font></td>
<td><font color="red">java.lang.Boolean</font></td>
<tr><td><font color="red">Integer</font></td>
<td><font color="red">java.lang.Long</font></td>
<tr><td><font color="red">Float</font></td>
<td><font color="red">java.lang.Double</font></td>
<tr><td><font color="red">String</font></td>
<td><font color="red">java.lang.String</font></td>
<tr><td><font color="red">List</font></td>
<td><font color="red">java.util.List<<font>T</font>></font></td>
<tr><td><font color="red">Map</font></td>
<td><font color="red">java.util.Map<<font>K, V</font>></font></td>
<tr><td><font color="red">Node</font></td>
<td><font color="red">org.neo4j.driver.v1.types.Node</font></td>
<tr><td><font color="red">Relationship</font></td>
<td><font color="red">org.neo4j.driver.v1.types.Relationship</font></td>
<tr><td><font color="red">Path</font></td>
<td><font color="red">org.neo4j.driver.v1.types.Path</font></td></tr></table>
</td></tr></table>

javascript
<table><tr><td width="1200">
<table><tr><td><b>Neo4j</b></td><td><b>JavaScript</b></td></tr>
<tr><td><font color="red">Null</font></td>
<td><font color="red">null</font></td></tr>
<tr><td><font color="red">Boolean</font></td>
<td><font color="red">Boolean</font></td>
<tr><td><font color="red">Integer</font></td>
<td><font color="red">Integer (*)</font></td>
<tr><td><font color="red">Float</font></td>
<td><font color="red">Number</font></td>
<tr><td><font color="red">String</font></td>
<td><font color="red">String</font></td>
<tr><td><font color="red">List</font></td>
<td><font color="red">Array</font></td>
<tr><td><font color="red">Map</font></td>
<td><font color="red">Object</font></td>
<tr><td><font color="red">Node</font></td>
<td><font color="red">Node</font></td>
<tr><td><font color="red">Relationship</font></td>
<td><font color="red">Relationship</font></td>
<tr><td><font color="red">Path</font></td>
<td><font color="red">Path</font></td></tr></table>

(*) JavaScript는 기본적으로 64 비트 부호없는 정수를 지원하지 않으므로, 드라이버는 <font color="red">Integer</font> 형식을 제공하여 그러한 값을 결과 데이터에 정확하게 나타낼 수 있게 합니다.
</td></tr></table>

python
<table><tr><td width="1200">
<table><tr><td><b>Neo4j</b></td><td><b>Python</b></td></tr>
<tr><td><font color="red">Null</font></td>
<td><font color="red">None</font></td></tr>
<tr><td><font color="red">Boolean</font></td>
<td><font color="red">bool</font></td>
<tr><td><font color="red">Integer</font></td>
<td><font color="red">int/long (†)</font></td>
<tr><td><font color="red">Float</font></td>
<td><font color="red">float</font></td>
<tr><td><font color="red">String</font></td>
<td><font color="red">str/unicode (†)</font></td>
<tr><td><font color="red">List</font></td>
<td><font color="red">list</font></td>
<tr><td><font color="red">Map</font></td>
<td><font color="red">dict</font></td>
<tr><td><font color="red">Node</font></td>
<td><font color="red">Node</font></td>
<tr><td><font color="red">Relationship</font></td>
<td><font color="red">Relationship</font></td>
<tr><td><font color="red">Path</font></td>
<td><font color="red">Path</font></td></tr></table>

<b>(†) Python 버전</b>
Python 2에서 Neo4j <font color="red">Integer</font>는 <font color="red">int</font>에 매핑되고 Neo4j <font color="red">String</font>은 <font color="red">str</font>에 매핑됩니다.
Python 3에서 Neo4j <font color="red">Integer</font>는 <font color="red">long</font>으로 매핑되고 Neo4j <font color="red">String</font>은 <font color="red">unicode</font>로 매핑됩니다.
</td></tr></table>
