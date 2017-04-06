## 4.3. 구성 및 연결
###### *<table width="1200"><td width=1200>Neo4j와 연결을 위해 드라이버를 구성하고 사용하십시오. 드라이버는 데이터베이스 작업에 대한 하나 이상의 트랜잭션 맥락인 세션을 제공합니다. </td></table>*


드라이버 개체 인스턴스는 어플리케이션 전체 및 여러 스레드에서 전역으로 공유 될 수 있습니다. 여기에는 작업을 수행하기 위해 빌릴 수 있는 클러스터의 다양한 멤버들에 대한 연결 풀이 포함되어 있습니다.

드라이버를 닫으면 현재 사용 중인지 여부에 관계없이 해당 드라이버가 생성 한 모든 연결이 끊어집니다.

### 4.3.1. 생성
드라이버 패키지에는 그래프 데이터베이스 객체가 포함되어 있습니다. 이 객체는 드라이버 인스턴스를 제공합니다. 데이터베이스 객체에서 드라이버 인스턴스를 요청할 때 URI가 제공됩니다. URI는 Neo4j 서버의 프로토콜, 호스트 및 포트를 선언합니다.

볼트 URI에는 다이렉트 URI와 라우팅 URI의 두 가지 종류가 있습니다. 다이렉트 URI는 단일 인스턴스에 연결하는 데 사용됩니다. 어플리케이션이 라우팅 및 로드 밸런싱에 의존하는 경우 이들은 반드시 별도로 제공되어야 합니다. 드라이버는 연결될 수 있는 클러스터를 알지 못하며 단일 서버에 대한 직접 연결만 인식합니다.이 모드는 독립 실행형 인스턴스로 작업하거나 사용자 정의 로드 밸런싱을 배포 할 때 적합합니다(예 : Neo4j 고 가용성 클러스터 및 HAProxy 사용).

라우팅 URI는 클러스터 인식 드라이버를 만드는 데 사용됩니다. 라우팅 드라이버는 클러스터에 연결하고 라우팅 및 로드 밸런싱을 처리합니다. 드라이버에 시드 URI를 제공 한 후 드라이버는 연결되어 클러스터 토폴로지 개요를 요청합니다. 그 후 드라이버는 특정 작업 항목을 서비스하는 데 가장 적합한 클러스터 멤버에 대해 세션을 생성 할 수 있습니다. 이 모드는 Neo4j Causal Cluster 작업에 적합합니다.

#### 4.3.1.1. 볼트 URI 포맷
볼트 URI는 <font color=red>scheme://host:port</font>와 같은 표준 URI 패턴을 따릅니다. 예를들어, <font color=red>bolt://203.0.113.1:1234</font>는 <font color=red>203.0.113.1</font> 주소로 포트 <font color=red>1234</font>를 통해 볼트 프로토콜을 사용하여 접속합니다. 호스트명은 IP 주소 대신에 사용될 수 있습니다: <font color=red>bolt://server:1234</font>. The default Bolt port is <font color=red>7687</font>.

라우팅 URI는 스키마 <font color=red>bolt+routing://server:1234</font>를 사용합니다.

<font color="gray">예제 4.3. 다이렉트 URI를 사용하여 드라이버 만들기</font>
C#
| <font color="purple">var</font> <font color="blue">driver</font> = GraphDatabase.Driver(<font color="red">"bolt://localhost:7687"</font>, AuthTokens.Basic(<font color="red">"neo4j"</font>, <font color="red">"neo4j"</font>)); |
|---|

라우팅 URI는 Neo4j Causal Cluster를 사용하기 위한 클러스터 인식 라우팅 드라이버를 만드는 데 사용됩니다

<font color="gray">예제 4.4. 라우팅 URI를 사용하여 드라이버 만들기</font>
| Driver driver = GraphDatabase.driver( <font color="red">"bolt+routing://localhost"</font>, AuthTokens.basic(<font color="red">"neo4j"</font>, <font color="red">"neo4j"</font>) ); |
|----------------|

생성시 오직 하나의 URI만 드라이버에 전달됩니다. 만일 해당 URI에서 응답 할 수있는 서버가 없으면 <font color="red">ServiceUnavailableException</font>이 발생합니다. 따라서 여러 시드 URI를 통해 루프 안에서 드라이버를 만드는 것이 유용 할 수 있습니다.

<font color="gray">예제 4.5. 여러 라우팅 URI를 사용하여 드라이버 만들기</font>
<table><tr><td>
<font color="purple">public</font> Driver <font color="blue">acquireDriver</font>(List<<font color="green">String</font>> uris, AuthToken authToken, Config config)<br>
{<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">for</font> (<font color="green">String</font> uri : uris)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">try</font> {<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp</font>;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">return</font> GraphDatabase.driver(uri, authToken, config);<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">catch</font> (ServiceUnavailableException ex)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="brown">// This URI failed, so loop around again if we have another.</font><br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="purple">throw new</font> ServiceUnavailableException(<font color="red">"No valid database URI found"</font>);<br>
}</td></tr>
</table>

일부 구성의 경우 IP 주소 대신 볼트 URI 내에서 호스트 이름을 사용하는 것도 고려해 볼 가치가 있습니다. 이것은 대체 서버로 리다이렉션 해야하는 경우 어플리케이션 구성 대신 DNS 수준에서 조정할 수 있는 추가 주소 계층을 허용합니다.

### 4.3.2. 구성
구성 설정이 드라이버 생성자에 제공됩니다. 이 섹션의 나머지 부분에 나오는 예제에서 이것의 사용법을 보여줍니다.

#### 4.3.2.1. 암호화와 인증
모든 Neo4j 드라이버는 SSL/TLS를 사용하여 Neo4j 드라이버와 Neo4j 인스턴스 간의 트래픽을 암호화합니다. Neo4j는 연결이 로컬 루프백 인터페이스에 대한 경우를 제외하고 기본적으로 암호화 된 연결만 허용합니다. 로컬 연결은 개발 환경에서 테스트하기 위한 경우가 일반적입니다. 테스트 시나리오가 진행되면서 빈번하게 여러 데이터베이스 서버가 차례로 시작되고 종료됩니다. 자동 생성 된 인증서 및 Trust On First Use 신뢰 전략을 사용하는 것은 문제가 될 수 있습니다. 첫 번째 서버에서 생성 된 인증서는 볼트 URI와 연결됩니다. 만일 시작된 다음 서버가 동일한 볼트 URI로 연결되어 지면, 인증서가 일치되지 않고 연결이 실패할 것입니다.

이를 줄이기 위해 기본 구성은 ENCRYPTION_NON_LOCAL 정책을 사용합니다. 즉, 원격 호스트와의 통신은 기본적으로 암호화되지만 로컬 연결은 암호화되지 않습니다.

이것은 모든 연결에 대해 암호화를 요구하도록 수정될 수 있습니다. 자세한 내용은 [Neo4j 운영 매뉴얼 → 볼트 커넥터 구성](https://neo4j.com/docs/operations-manual/3.1/configuration/connectors/)을 참조하십시오.

Neo4j에 저장된 인증 증명서와 데이터를 안전하게 유지하기 위해 암호화를 사용하는 것을 강력히 추천합니다.
<table bordercolor="white" border=0>
<tr><td><img src="./img/warning.gif" width="70"></img></td><td>기술 제한으로 인해 .NET 드라이버에는 기본적으로 활성화 된 암호화가 없습니다. JavaScript 드라이버는 웹 브라우저에서 실행될 때 기본적으로 활성화 된 암호화가 없습니다.</td></tr></table>

##### 암호화
TLS 암호화를 켜거나 끄도록 드라이버를 구성 할 수 있습니다. 암호화는 신뢰할 수 있는 내부 네트워크에서만 해제해야 합니다. 대부분의 드라이버는 기본적으로 암호화를 사용하도록 설정되어 있지만 명시적으로 암호화를 구성하는 것이 좋습니다. 이것은 다른 개발자가 코드를 읽을 때 명확한 암호화가 사용되도록 하여 기본값을 사용하여 야기되는 실수의 위험을 최소화합니다

<font color="gray">예제 4.6. TLS 암호화를 사용하도록 드라이버 구성하기</font>
C#

##### 신뢰
암호화 된 연결을 설정할 때 원격 피어가 우리가 연결할 것으로 예상되는 사람인지 확인해야 합니다. 이를 확인하지 않으면 공격자는 "man-in-the-middle" 공격을 쉽게 수행 할 수 있으며, 드라이버를 속여 Neo4j 인스턴스 대신 그녀와 암호화 된 연결을 설정할 수 있습니다. 이 때문에 Neo4j 드라이버는 신뢰 구축없이 암호화 된 연결을 설정하는 방법을 제공하지 않습니다.

두 가지 신뢰 전략을 사용할 수 있습니다:

- Trust on first use.
- Trust signed certificate.

*Trust on first use*는 드라이버가 안전하고 의도적인 호스트에 대한 첫 번째 연결을 신뢰한다는 것을 의미합니다. 후속 연결에서 드라이버는 첫 번째 연결에서와 동일한 호스트와 통신하는지 확인합니다. 이 인증 전략이 유효하도록 하려면 서버에 대한 첫 번째 연결을 신뢰할 수 있는 네트워크 환경에서 설정해야 합니다.

이 전략은 ssh 커맨드 라인 툴에서 사용하는 기본 전략과 같습니다. 이것은 아무런 구성없이 좋은 수준의 보안을 제공합니다. 이런 이유로, 이를 지원하는 모든 플랫폼에서 이것이 기본 전략입니다.

기술 제한으로 인해 이 전략은 .NET 플랫폼이나 웹 브라우저에서 JavaScript 드라이버를 사용하는 경우에는 사용할 수 없습니다.

<font color="gray">예제 4.7. 첫번째 사용을 신뢰하도록 드라이버 구성하기</font>
C#
<table width=1200><font color="brown"><tr><td width=1200>// Not supported in this driver</td></tr></font>
</table>

*Trust signed certificate*는 드라이버가 그것이 신뢰할 수 있는 인증서를 제공하는 경우에만  Neo4j 서버에 연결할 것이라는 것을 의미합니다. 신뢰할 수 있는 인증서는 드라이버에 의해 신뢰되도록 명시적으로 구성된 인증서이거나 일반적으로 드라이버가 신뢰하는 인증서로 서명된 인증서입니다. 모든 드라이버는 이 신뢰 전략을 지원합니다.

드라이버가 사용할 신뢰할 수 있는 인증서를 설치하는 방법은 드라이버마다 다릅니다. Java 드라이버는 신뢰할 수 있는 인증서의 파일 경로를 직접 받아들여 런타임에 자동으로 인증서를 로딩합니다. .NET 및 Python 드라이버는 먼저 인증서를 운영 체제의 신뢰할 수 있는 인증서 저장소에 설치해야 합니다.

<font color="gray">예제 4.8. 서명 된 인증서를 신뢰하도록 드라이버 구성하기</font>
C#
<table><tr><td width=1200> var driver = GraphDatabase.Driver("bolt://localhost", AuthTokens.Basic("neo4j", "neo4j"),<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Config.Builder.WithEncryptionLevel(EncryptionLevel.Encrypted)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.WithTrustStrategy(TrustStrategy.TrustSystemCaSignedCertificates).ToConfig());
        </td></tr>
        </table>

#### 사용자 인증
서버는 인증이 사용 불가능한 경우를 제외하고는, 사용자가 데이터베이스에 연결하기 위한 인증 증명서를 드라이버에게 제공하도록 요구합니다.

<font color="gray">예제 4.9. 인증이 비활성화 된 서버에 드라이버 연결하기</font>
C#

드라이버와 서버 간의 통신이 암호화되지 않으면 인증 증명서가 일반 텍스트로 전송됩니다. 인증을 사용할 때는 암호화를 사용하는 것이 좋습니다.
<table><tr><td><img src="./img/note.gif" width="70"></td>
<td with=1200>Neo4j에 처음 연결할 때, 사용자는 반드시 기본 암호를 변경해야 합니다. 신규 Neo4j 서버 설치시, 기본 암호로 Neo4j 서버에 연결하려고 하면 자격 증명서가 만료되었고 암호를 업데이트 해야한다는 알림이 제공됩니다. 브라우저는 만료 된 암호를 변경하는 데 사용될 수 있습니다.</td></tr></table>
