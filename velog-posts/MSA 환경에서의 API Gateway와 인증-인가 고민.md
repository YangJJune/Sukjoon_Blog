<h2 id="서론">서론</h2>
<p>사이드 프로젝트를 진행하며 아키텍처를 설계하다 보면, 편리함과 확장성 사이에서 항상 선택의 기로에 서게 된다. </p>
<p>이번 <strong>'KBO:Note'</strong> 프로젝트에서는 단순히 기능을 구현하는 것을 넘어, 팀원들의 기술적 니즈와 서비스의 확장 가능성을 고려해 <strong>MSA(Microservices Architecture)</strong>를 도입하기로 결정했다.</p>
<p>프로젝트의 전체적인 흐름을 설계하고 구성 요소를 배치하는 과정에서 마주한 고민과, 이를 해결하기 위해 구성한 API Gateway 및 인증/인가 전략을 정리하고자 한다.</p>
<h2 id="왜-msa인가">왜 MSA인가?</h2>
<p>이번 사이드 프로젝트는 야구 데이터를 가공하여 사용자에게 피드 형태로 제공하는 서비스다. MSA를 선택한 이유는 명확했다.</p>
<blockquote>
<p><strong>1. 개발 스택의 자율성</strong>
각 팀원이 경험하고 싶은 기술 스택이 조금씩 달랐다. MSA를 통해 도메인별로 독립된 환경을 제공함으로써 각자가 최적의 스택을 선택할 수 있도록 했다.</p>
</blockquote>
<blockquote>
<p><strong>2. 리소스의 효율적 관리</strong>
데이터 크롤링, AI 판독, 피드 제공 등 도메인별로 필요한 컴퓨팅 리소스가 제각각일 것이라 예측했다. 
특정 서비스에 트래픽이 몰릴 경우 해당 서비스만 <strong>유연하게 확장</strong>하기 위함이다.</p>
</blockquote>
<p>결과적으로 <strong>개발자는 본인이 담당한 마이크로서비스만을 독립적으로 개발하고 배포하면 되는 환경</strong>을 구축하고자 했다. </p>
<p>하지만 다음과 같은 문제를 직면하게 되었다.</p>
<h2 id="파편화된-서비스-어떻게-관리할-것인가">파편화된 서비스, 어떻게 관리할 것인가?</h2>
<p>서비스를 분리하고 나니 크게 두 가지 핵심적인 문제가 발생했다.</p>
<h3 id="1-서비스-엔드포인트의-독립-문제">1. 서비스 엔드포인트의 독립 문제</h3>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/4bb64dc1-2e6a-41b9-ab6f-073d51bafece/image.png" /></p>
<p>클라이언트(프론트엔드) 입장에서 각기 다른 도메인(피드 서버, 시즌 성적 서버 등)의 주소를 일일이 알고 호출하는 것은 <strong>매우 비효율적이다</strong> 서비스가 늘어날수록 클라이언트의 복잡도는 선형적으로 증가하며, CORS 설정이나 보안 정책을 모든 서버에 개별적으로 적용해야 하는 번거로움이 있었다.</p>
<h3 id="2-인증인가-문제">2. 인증/인가 문제</h3>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/042b4b6f-be4b-4920-ae5f-d4aa1a206d1b/image.png" /></p>
<p>각 마이크로서비스마다 인증 로직을 중복으로 구현하는 것은 <strong>'바퀴를 재발명'</strong>하는 일이다. </p>
<p>사용자 정보를 확인하고 권한을 검증하는 로직을 한곳에서 처리하고, 
하위 서비스들은 비즈니스 로직에만 집중할 수 있는 구조가 필요했다.</p>
<h2 id="해결방안-api-gateway와-auth-서버의-구현">해결방안 API Gateway와 Auth 서버의 구현</h2>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/31168b87-722d-4f5f-9ee9-96db81e0212c/image.png" /></p>
<p>문제를 해결하기 위해 아키텍처 중앙에 <strong>API Gateway</strong>를 배치하기로 했다.</p>
<h3 id="api-gateway란">API Gateway란?</h3>
<p>여기서 Gateway 그 자체에 대한 개념도 짚고 넘어가면 좋을 거 같은데</p>
<p><strong>Gateway</strong>는 서로 다른 네트워크 혹은 시스템 사이를 연결해주는 ‘중간 지점’이다.
외부에서 들어온 요청이 내부로 바로 전달되지 않고, 반드시 이 관문을 통과하도록 강제함으로써 흐름을 통제하고, 경로를 결정하는 역할을 수행한다. </p>
<p><strong>API Gateway</strong>는 간단히 말해 <strong>'모든 클라이언트 요청의 단일 진입점' 역할을 하는 서버</strong>다. 
수 많은 마이크로서비스가 얽혀있는 MSA 환경에서 클라이언트가 각각의 서비스와 직접 통신하는 대신, <strong>Gateway</strong>라는 '거대한 문' 하나만을 거치게 하는 것이다.</p>
<p>주요 역할은 다음과 같다</p>
<blockquote>
</blockquote>
<p><strong>1. 라우팅(Routing)</strong>
요청 경로에 따라 적절한 서비스 서버로 전달한다.
&nbsp;
<strong>2. 인증 및 인가(Auth)</strong>
요청이 내부망으로 진입하기 전, 유효한 사용자인지 검증한다.
&nbsp;
<strong>3. 부하 분산(Load Balancing)</strong>
트래픽을 여러 인스턴스로 분산시켜 안정성을 높인다.
&nbsp;
<strong>4. 로깅 및 모니터링</strong>
중앙에서 모든 트래픽을 관찰하므로 로그 수집이 용이하다.</p>
<p>우리 프로젝트에서는 특히 <strong>'인증의 단일화'</strong>와 <strong>'엔드포인트 통합'</strong>을 위해 이 Gateway의 역할이 매우 중요했다.</p>
<h2 id="spring-cloud-gateway-vs-custom-proxy--오버-엔지니어링에-대한-고민">Spring Cloud Gateway vs Custom Proxy? : 오버 엔지니어링에 대한 고민</h2>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/4d0c01d5-60ec-400f-8512-e8c656bf4d76/image.png" /></p>
<p>가장 먼저 고민한 것은 '이미 잘 만들어진 솔루션을 쓸 것인가, 직접 만들 것인가'였다.
처음에는 당연하게도 Java 진영의 표준과도 같은 <strong>Spring Cloud Gateway(SCG)</strong>를 고려했다.</p>
<p>하지만 설계를 구체화할수록 한 가지 의문이 생겼다.</p>
<p>&quot;우리 프로젝트에 이만큼의 기능이 정말 다 필요한가?&quot;</p>
<h3 id="오버-엔지니어링over-engineering의-경계">오버 엔지니어링(Over-engineering)의 경계</h3>
<p><code>Spring Cloud Gateway</code>는 강력하다. 
Circuit Breaker, Rate Limiting, Service Discovery 연동 등 대규모 트래픽을 처리하기 위한 온갖 기능을 제공한다.</p>
<p>하지만 <strong>SCG의 성능적 이점(비동기 논블로킹)</strong>을 누리기 위해서는 프로젝트 전체에 <strong>WebFlux 의존성</strong>을 추가해야 하며,
이는 곧 개발 난이도의 상승과 학습 곡선을 의미한다.</p>
<p>현재 우리 프로젝트 상황을 고려했을 때 다음과 같은 결론에 도달했다.</p>
<blockquote>
<ol>
<li>우리 서비스는 현재 초당 수만 건의 요청이 발생하는 단계가 아니다.</li>
<li>서킷 브레이커가 없더라도, 단순한 구조를 유지하며 빠른 트러블슈팅이 가능한 것이 더 유리하다 판단했다.</li>
</ol>
</blockquote>
<p>결국, 지금 우리에겐 SCG가 오버 엔지니어링이라 판단했고, 구조가 단순하며 커스텀이 용이한 가벼운 <strong>Custom Proxy를 구축</strong>하기로 결정했다.</p>
<p>&nbsp;
&nbsp;
&nbsp;</p>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/1f2bba0b-8f5d-42ae-9423-08ee4a4932da/image.png" /></p>
<h2 id="근데-이제-auth-서버를-곁들인">&quot;근데 이제 Auth 서버를 곁들인&quot;</h2>
<p>인증 문제를 해결하기 위해 API Gateway가 OAuth 서버의 역할을 겸하거나, 긴밀하게 통신하며 진입점에서 모든 권한 검증을 마치는 구조를 택했다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/60990d7a-07dd-46b0-bf26-94da876a9120/image.png" /></p>
<p>구상도에서 볼 수 있듯이, 모든 클라이언트의 요청은 API Gateway + OAuth 서버를 거치게 된다.</p>
<blockquote>
<p><strong>1. 인증 처리</strong> : Redis를 활용해 JWT 토큰 정보를 관리하며, API Gateway에서 유효한 사용자인지 먼저 판단한다.
<strong>2. 라우팅</strong> : 인증이 완료된 요청은 뒤편의 각각의 서버 등으로 분기된다. (이때 토큰은 넘어가지 않고 유저 정보만 넘긴다)
<strong>3. 보안 강화</strong> : 외부에서는 내부 마이크로서비스의 주소를 직접 알 수 없으므로 보안성이 향상된다.</p>
</blockquote>
<p>결과적으로 내부 서버들은 '누가 요청했는가'에 대한 고민을 Gateway에 맡기고, 
오로지 데이터 제공이라는 본연의 역할에만 충실할 수 있는 구조를 설계할 수 있었다.</p>
<h2 id="마치며">마치며</h2>
<p><strong>MSA는 시스템을 유연하게 만들지만</strong>, 그만큼 <strong>관리 포인트가 늘어나는</strong> 트레이드 오프가 확실한 아키텍처다. </p>
<p>이번 설계를 통해 복잡한 통신 구조를 API Gateway라는 단일 창구로 묶음으로써, 
협업 효율과 시스템 안정성이라는 두 마리 토끼를 잡고자 했던 경험을 적으며 이렇게 마무리한다.</p>