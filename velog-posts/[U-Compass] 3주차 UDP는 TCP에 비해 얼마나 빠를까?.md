<h2 id="실험-개요">실험 개요</h2>
<p><a href="https://steady-coding.tistory.com/516">TCP랑 UDP, UDP가 더 빠르다</a>
비연결성, 신뢰성 보장 X 등 왜 빠른지도 이해를 할 수 있으나 그동안 들었던 의문은 <strong>도대체 얼마나 빠르다는건데?</strong> 였다.</p>
<p>&nbsp;</p>
<h2 id="실험-내용">실험 내용</h2>
<p>UDP가 TCP 보다 얼마나 빠른 지 알아보자. 또한 이에 따른 defect는 없는지 알아보자.</p>
<p>&nbsp;</p>
<h2 id="환경-설정">환경 설정</h2>
<p>※ 최대한 동일한 RTT가 될 수 있어야한다. 이를 고려하자</p>
<p><a href="https://medium.com/@su_bak/%EC%99%95%EB%B3%B5-%EC%8B%9C%EA%B0%84-round-trip-time-rtt-%EC%9D%B4%EB%9E%80-96d78eea40b7">※ RTT : Round Trip Time, 인터넷 상에서 출발지에서 목적지로 가는 데 걸리는 시간</a></p>
<ul>
<li><p>많은 양의 데이터를 (data send 수)를 보내보자</p>
</li>
<li><p><del>커다란 용량의 데이터를 보내보자</del> (어차피 패킷을 쪼개고 쏘는 게 똑같아서 위의 실험과 차이가 없다고 판단, 특이점이 있다면 TCP는 패킷을 알아서 쪼개주고 UDP는 패킷을 직접 쪼개야함)</p>
<p>  → 이때 각각의 상황에서 속도(소요 시간)와 <a href="https://developer.android.com/topic/performance/power/battery-historian?hl=ko">배터리 성능을 고려하자</a></p>
</li>
</ul>
<p>&nbsp;</p>
<h2 id="실험-모델">실험 모델</h2>
<blockquote>
<p>1:1 통신의 경우, server와 client 간에 소켓을 열기만 하면 된다.</p>
</blockquote>
<p>&nbsp;</p>
<h3 id="특이사항---localhost에서는-왜-차이가-안-날까">특이사항 - localhost에서는 왜 차이가 안 날까?</h3>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/f767918a-af0e-427b-81b2-2358d03cbdde/image.png" /></p>
<blockquote>
<p>**    1. 앱에서 데이터 전송**
        - 웹 브라우저로 <a href="http://127.0.0.xn--1-w30f/">http://127.0.0.1로</a> 요청을 보내요.
        - 이 요청은 <code>TCP/IP</code> 스택을 통해 패킷으로 나가려고 하겠죠?</p>
</blockquote>
<blockquote>
<p>**    2. <code>TCP/IP</code> 스택에서 <code>IP주소</code>를 보고 <code>루프백 NI</code>로 갑니다.**
        - 목적지 주소 <code>127.0.0.1</code>을 확인했으니,이제 패킷을 외부로 보내지 않고 로컬로 다시 보냅니다.</p>
</blockquote>
<blockquote>
<p>**    3. 로컬로 패킷 전달**
        - 패킷은 다시 <code>TCP/IP</code> 스택 계층으로 전달되고, 올바른 <code>포트</code>로 전달됩니다.</p>
</blockquote>
<blockquote>
<p>**    4. 서버 앱 요청 수신**
        - 로컬 서버 앱은 이 요청을 수신하고, 적절한 응답을 생성하여 다시 루프백 인터페이스로 보냅니다.
        - 이 앱은 우리가 만든 <code>node.js</code> 앱이나 기타 앱들이 되겠죠?</p>
</blockquote>
<blockquote>
<p>**    5. 응답 패킷의 루프백 경로**
        - 응답 패킷도 동일한 과정을 거쳐 결국, 최종적으로 요청했던 앱인 로컬 브라우저로 전달됩니다.</p>
</blockquote>
<p>   <strong>정리하자면 localhost가 감지되는 순간 외부로 나가지 않고 내부에서 돌기 때문에 빠르다</strong>
   출처 : <a href="https://velog.io/@480/localhost-%EC%9D%98-%EB%8F%99%EC%9E%91-%EC%9B%90%EB%A6%AC">https://velog.io/@480/localhost-%EC%9D%98-%EB%8F%99%EC%9E%91-%EC%9B%90%EB%A6%AC</a></p>
<p>&nbsp;</p>
<h2 id="실험-결과">실험 결과</h2>
<p>각각 동일한 환경에서 총 3회씩 실행하고 평균으로 표현 (큰 차이 없음 확인)</p>
<p>※ 나중에 하단에 기술하겠지만 UDP의 경우 Timeout을 500ms로 걸음</p>
<h3 id="localhost-→-localhost-상황"><strong>localhost → localhost 상황</strong></h3>
<blockquote>
<p><strong>100,000회</strong>
TCP 42778ms
UDP 41806ms
&nbsp;
<strong>1,000,000회</strong>
TCP 394727ms
UDP 406091ms</p>
</blockquote>
<p>속도가 굉장히 빠르기에 증가폭도 굉장히 linear하며, 오히려 TCP가 더 빠르게 측정되기도 한다. 
유의미한 차이는 없다고 보인다</p>
<p><strong>로컬 내부 네트워크 → 로컬 내부 네트워크 (무선)</strong></p>
<blockquote>
</blockquote>
<p><strong>100회</strong>
TCP 1522ms
UDP 1565ms
&nbsp;
<strong>1,000회</strong>
TCP 18205ms
UDP 9584ms
&nbsp;
<strong>10,000회</strong>
TCP 163663ms
UDP 21728ms
&nbsp;
<strong>100,000회</strong>
TCP 1801237ms
UDP   90765ms
<strong>⇒ 무려 20배 차이</strong></p>
<p><strong>모바일 → 서버 (실질 모델)</strong></p>
<blockquote>
<p><strong>100 회</strong>
TCP 24680ms
UDP 14618ms
&nbsp;
<strong>1,000 회</strong>
TCP 240775ms
UDP 158400ms
&nbsp;
<strong>10,000 회</strong>
TCP 2457118ms
UDP 754998ms</p>
</blockquote>
<p><strong>패킷 크기의 차이</strong></p>
<blockquote>
<p><strong>TCP 패킷</strong> 442B
<strong>UDP 패킷</strong> 321B</p>
</blockquote>
<h2 id="트러블슈팅">트러블슈팅</h2>
<h3 id="1-메소드-이해-부족으로-인한-유사-데드락-발생"><strong>1. 메소드 이해 부족으로 인한 유사 데드락 발생</strong></h3>
<p><strong>A. TCP의 경우</strong></p>
<ul>
<li>socket.accept()의 경우 await와 동일하게 connect()가 들어올 때 까지 기다린다</li>
<li>이를 실수로 무한루프 안에 두어 매 통신마다 기다리는 상황 생김</li>
</ul>
<p>※ 실제로 여러 소켓 통신을 열어야하는 경우 thread를 사용해야합니다</p>
<p><strong>B. UDP의 경우</strong></p>
<ul>
<li>손실이 발생할 경우, 제대로 보내지지 않는 경우 그대로 상대방 응답을 리슨하는 상황이 되어버림. </li>
<li>UDP의 단점인 손실 발생시 조치를 해줌(TIMEOUT 처리)</li>
</ul>
<h3 id="2-에뮬레이터는-로컬통신을-지원하지-않으며-자체-라우터의-사용-및-고유-ip가-존재한다"><strong>2. 에뮬레이터는 로컬통신을 지원하지 않으며 자체 라우터의 사용 및 고유 IP가 존재한다.</strong></h3>
<ul>
<li>인스턴스는 라우터에 의해 격리되며 같은 네트워크에서 서로를 감지할 수 없다.</li>
</ul>
<p>관련자료 : <a href="https://developer.android.com/studio/run/emulator-networking?hl=ko">https://developer.android.com/studio/run/emulator-networking?hl=ko</a>
&nbsp;</p>
<h3 id="3-broken-pipeline-error"><strong>3. Broken pipeline Error</strong></h3>
<ul>
<li><p>TCP 통신이 끝났는데, 또는 어떠한 사유로 소켓이 닫혔는데 통신할 경우 발생한다 볼 수 있음</p>
<p><strong>생길 수 있는 경우</strong></p>
<ol>
<li>클라이언트 또는 서버가 소켓을 닫았는데 데이터를 보낸 경우</li>
<li>서버의 프로세스에 과부하가 걸려 처리할 수 없는 상황에 데이터를 보낸 경우</li>
<li>etc.. <img alt="" src="https://velog.velcdn.com/images/ysj7191/post/1a339a38-5c5d-46b6-92e0-cf03f4b244cc/image.png" /></li>
</ol>
</li>
</ul>
<h3 id="4-포트포워딩-이슈">4. 포트포워딩 이슈</h3>
<ol>
<li>집 공유기 환경이 다소 복잡한데, 무선으로 연결된 모바일 디바이스와의 포트포워딩에 이슈가 있어 시간이 오래 걸렸음</li>
<li>총괄 허브, 방 내의 Wi-Fi 공유기 포트포워딩 완료하여 해결하였다</li>
</ol>
<p>유사한 상황에서 꼭 본인의 네트워크 환경을 잘 파악하고 해결해야 오래 막히지 않을 것이다.</p>
<h2 id="결론">결론</h2>
<blockquote>
<p>전송속도가 점점 느려질수록, RTT가 길어질 수록 TCP와 UDP 간의 속도 차이가 점점 벌어진다.
이 차이는 linear하지 않으며 exponential 하게 증가하는 것으로 보인다.</p>
</blockquote>
<h3 id="추가사항">추가사항</h3>
<ul>
<li>배터리 성능에 대한 점검법에 대한 확인이 필요합니다 (Android Studio Profiler</li>
</ul>