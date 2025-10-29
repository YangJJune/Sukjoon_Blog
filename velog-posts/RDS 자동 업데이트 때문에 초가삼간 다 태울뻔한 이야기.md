<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/2e5bc80d-1c8e-4340-805a-081c98dc194f/image.png" /></p>
<h3 id="문제-발생">문제 발생!</h3>
<p>최근 효과적인 로그 감지를 위해 Sentry를 달았습니다.
그런데 1년 동안 로그를 찍어보며 진짜 처음보는 에러가 전방위적으로 잡히는 것을 보게 됐다.</p>
<blockquote>
<p>Could not open JPA EntityManager for transaction</p>
</blockquote>
<p>일단 대충 읽어보면 transaction 하려고 JPA EntityManager를 열었는데 안 열려서 화가 난거 같은데...
이제부터 오류의 원인을 찾아보자</p>
<h3 id="에러-추적">에러 추적</h3>
<p>한번 직접 docker에 들어가 로그를 찍어보았는데, timeout 에러가 발생했던 것을 알 수 있다.</p>
<blockquote>
<p>WARN)  o.h.e.j.s.SqlExceptionHelper.logExceptions [traceId=2a8f3ab3463fc61fd36a5def53d30e42 spanId=e6f0a70b159de663] - SQL Error: 0, SQLState: 08S01
ERROR) o.h.e.j.s.SqlExceptionHelper.logExceptions [traceId=2a8f3ab3463fc61fd36a5def53d30e42 spanId=e6f0a70b159de663] - HikariPool-1 - Connection is not available, request timed out after 30000ms.
ERROR) o.h.e.j.s.SqlExceptionHelper.logExceptions [traceId=2a8f3ab3463fc61fd36a5def53d30e42 spanId=e6f0a70b159de663] - Communications link failure
ERROR) U.g.c.c.a.t.TracingConfig$TracingAspect.traceController [traceId=2a8f3ab3463fc61fd36a5def53d30e42 spanId=4935d1378a429aba] - [ERROR] WaitingController.getMyWaitingList(..) : org.springframework.transaction.CannotCreateTransactionException: Could not open JPA EntityManager for transaction</p>
</blockquote>
<p>timeout 에러가 발생했다는 것은 당연히 연결되어야하는 노드에 문제가 생겼을 가능성이 높다.
여기서 연결하려고 했던 곳은 RDS에 올라가있는 DB 이므로 이를 확인해보았다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/18f840ed-b891-4cc6-8183-80aac43a4cf4/image.png" />
RDS는 다음과 같이 에러 상황 발생 시 로그를 제공한다.
(여담인데 같은 VPC에 속하지 않아서 접속시도한 경우도 로그가 잡히는 듯 하다)</p>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/e135f20c-131b-42bd-bea7-5e126bbf0ada/image.png" />
같은 시간에 업그레이드가 진행되어 백엔드 서버와 RDS가 연결이 끊겼음을 확인할 수 있었다.</p>
<h3 id="해결방법">해결방법</h3>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/4baade1d-c856-4bf7-a4ce-76baa9f96c16/image.png" />
RDS에서 수정을 통해 들어가면 다음과 같이 자동 업그레이드를 끌 수가 있다..
이번엔 실제 프로덕션 2일 전에 생긴 3분 간의 장애라 탈이 없었지만 아다리가 안 좋았다면 실서비스 날에 터졌을테니 굉장히 오싹오싹한 순간이었다.</p>