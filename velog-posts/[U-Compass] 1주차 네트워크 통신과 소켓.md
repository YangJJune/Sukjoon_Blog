<h2 id="서론">서론</h2>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/dcf95156-b6eb-46e7-98e9-0103a42aa2cb/image.png" /></p>
<p>이번에 드림학기제라는 교내 활동에 합격하여 U-Compass라는 서비스를 제작하고자 한다. 중간발표 전까지는 소켓 통신에 대하여 공부하고 연구한 뒤, 이를 바탕으로 최종 발표 때 U-Compass 어플을 제작하는 것이 목표이다.</p>
<h2 id="뭘-할-것인가-u-compass-란">뭘 할 것인가? U-Compass 란?</h2>
<p>위에서 U-Compass라는 프로젝트를 위해 공부 및 연구를 진행한다고 하였다. 따라서 U-Compass가 무엇인지 설명을 해야한다고 본다. U-Compass란 친구 간의 실시간 위치정보를 공유하는 서비스이다. 여기서 주목해야하는 점은 실시간이라는 점, 데이터를 공유한다는 특성, 그리고 모바일 앱이라는 것이다. 이는 필연적으로 어떠한 구조로 소켓 통신 또는 HTTP 통신을 하는 것이 가장 빠르고, 리소스를 효율적으로 사용하는지 등을 고려하여만 한다. 따라서 2달 간 이에 대한 연구를 진행할 것이다.</p>
<h2 id="네트워크란">네트워크란</h2>
<blockquote>
<p>네트워크는 둘 이상의 컴퓨터와 이들을 연결하는 링크의 조합입니다. 물리적 네트워크는 네트워크를 구성하는 하드웨어(어댑터, 케이블 및 전화선과 같은 장비)입니다. 소프트웨어 및 개념 모델이 논리적 네트워크를 형성합니다. 다른 유형의 네트워크 및 에뮬레이터는 서로 다른 기능을 제공합니다.</p>
</blockquote>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/1921ea48-c178-4cbe-9ca4-c4e24b6735e6/image.png" /></p>
<p>말이 처음보는 인원에겐 어려울 수 있는데 간단히 말해 두 컴퓨터를 연결하여 데이터를 전달하는 것을 네트워크라고 할 수 있다.</p>
<h2 id="osi-모델">OSI 모델?</h2>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/e77369fe-1df3-48b4-a93d-8dde113d7ac8/image.png" />
그런데 위에서 쉽게 말하긴 했지만 두 컴퓨터 사이를 연결하는 것은 절대 단순한 일이 아니다. 이런저런 처리가 필요하기에 이 처리에 대하여 7가지 계층으로 분류한 모델이 있다. 그것이 OSI 7계층 모델이다 (OSI Model)</p>
<p>각각의 layer에 대해 간단히 짚고 넘어가겠다</p>
<blockquote>
<p>Application Layer - 유저가 상호작용하는 부분들을 총괄하는 레이어
Presentation Layer - data가 사용가능한 format인지 보증 및 데이터 암호화를 담당하는 레이어
Session Layer - connection을 유지하는 레이어 (포트와 세션 관리)
Transport Layer - TCP와 UDP 같은 프로토콜을 사용하여 데이터를 전송하는 레이어
Network Layer - 어떤 physical path를 통해 data를 전달할 지 정하는 레이어
Data Link Layer - 네트워크 상에서 data의 format을 정의하는 레이어
Physical Layer - 가장 하단 계층인 만큼 raw bit 로 이루어진 stream을 보내는 레이어</p>
</blockquote>
<p>&nbsp;</p>
<h2 id="네트워크-통신이란">네트워크 통신이란?</h2>
<blockquote>
<p>컴퓨터나 장치들이 네트워크를 통해 서로 데이터를 주고받는 과정이다. 네트워크 통신은 정보나 데이터를 전송하기 위한 규칙과 방법을 정의한 다양한 통신 프로토콜을 사용하여 이루어지며,이러한 통신은 물리적인 연결(예: 케이블, Wi-Fi)을 기반으로 하며, 이를 통해 여러 시스템이 상호작용하고 협력할 수 있다. 우리는 여기서 HTTP 통신과 소켓 통신에 대해 다룬다</p>
</blockquote>
<p>&nbsp;</p>
<h2 id="http-통신이란">HTTP 통신이란?</h2>
<blockquote>
<p>웹에서 클라이언트와 서버 간에 데이터를 전송하기 위한 프로토콜이다. HTTP는 주로 웹 브라우저와 웹 서버 간의 통신에 사용되며, 클라이언트가 서버에 요청을 보내고 서버가 응답을 반환하는 방식으로 작동한다.</p>
</blockquote>
<ul>
<li>클라이언트가 요청을 보내면 서버가 응답을 반환한다.</li>
<li>GET, POST, PUT, DELETE 등 다양한 메서드를 사용하여 요청의 종류를 정의한다.</li>
<li>요청에 대한 응답으로 상태 코드를 반환하여 요청의 성공 여부를 나타낸다(예: 200 OK, 404 Not Found).</li>
<li>단발적이다. (유지되지 않는다)</li>
</ul>
<p>&nbsp;</p>
<h2 id="socket">Socket</h2>
<blockquote>
<p>소켓(Socket)은 네트워크 상에서 통신을 하기 위한 종단점(endpoint)이다. 소켓은 프로세스 간 통신을 위해 필요하며, 클라이언트와 서버가 데이터를 주고받는 데 사용된다. 소켓은 운영체제에 의해 관리되며, IP 주소와 포트 번호를 통해 식별된다.</p>
</blockquote>
<p>&nbsp;</p>
<h2 id="http-통신과-socket의-비교">HTTP 통신과 Socket의 비교</h2>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/ef8f6bfd-37fb-4b70-a849-77600bcf55b8/image.png" /></p>
<h2 id="장단점-비교">장단점 비교</h2>
<h3 id="http-통신">HTTP 통신</h3>
<p><strong>장점</strong></p>
<ol>
<li>웹 브라우저와 서버 간의 통신을 쉽게 설정할 수 있어 사용이 간편하다.</li>
<li>HTTP는 널리 사용되는 표준 프로토콜로, 다양한 플랫폼과 언어에서 지원된다.</li>
<li>요청의 결과를 명확하게 전달하는 상태 코드를 통해 오류 처리가 용이하다.</li>
</ol>
<p><strong>단점</strong></p>
<ol>
<li>각 요청이 독립적이기 때문에 상태를 유지하기 어렵고, 세션 관리가 복잡할 수 있다.</li>
<li>매 요청마다 연결을 설정하고 해제해야 하므로, 대량의 요청 처리 시 성능 저하가 발생할 수 있다.</li>
</ol>
<h3 id="socket-1">Socket</h3>
<p><strong>장점</strong></p>
<ol>
<li>클라이언트와 서버 간의 지속적인 연결을 통해 실시간 데이터 전송이 가능하다.</li>
<li>다양한 프로토콜(TCP, UDP)을 사용할 수 있어 특정 요구 사항에 맞게 최적화할 수 있다.</li>
<li>연결이 유지되므로 상태 정보를 쉽게 관리할 수 있다.</li>
</ol>
<p><strong>단점</strong></p>
<ol>
<li>소켓 프로그래밍은 상대적으로 복잡하여 구현이 어려울 수 있다.</li>
<li>기본적으로 보안 기능이 없으므로, 추가적인 보안 조치가 필요하다.</li>
</ol>