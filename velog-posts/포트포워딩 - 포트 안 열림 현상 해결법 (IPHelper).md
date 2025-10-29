<h3 id="서론">서론</h3>
<p>시작하기 앞서 데스크탑의 네트워크 구조는 다음과 같다</p>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/cbee428b-71ab-47ef-b78c-8ebba27894ad/image.png" /></p>
<p>기본적으로 통신사에서 설치한 허브, 방으로 타고 들어오는 LAN에 물린 무선 공유기, 그리고 그 아래에 2개의 말단 기기 (노트북, 데스크탑)이 달려있다</p>
<p>굳이 따지면 복잡한 구조인데, 메인 허브와 방에 있는 무선 공유기 둘 다 포트포워딩을 해야하는 구조이기 때문이다. 하지만 원리만 이해하면 어렵지 않다.</p>
<h3 id="포트포워딩-하기">포트포워딩 하기</h3>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/0fd524cd-d4d7-4eb2-8b4e-0acaa728fbc0/image.png" /></p>
<p>제품 별로 방법이 다르겠지만 NAT 설정을 찾아서 같은 설정을 해주면 된다. 보통 여기까지가 포트포워딩을 하는 법이다. 하지만 본인의 경우 노트북에서 서버를 기동했지만 포트가 열리지 않았다.</p>
<h3 id="방화벽-규칙-설정">방화벽 규칙 설정</h3>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/cf2912d7-f027-4f21-84a8-aa660bb2e5fe/image.png" /></p>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/56d1c69b-5324-440a-adb4-debb4e3a82a8/image.png" /></p>
<blockquote>
<p>Windows Defender 방화벽 &gt; 고급 설정 &gt; 인바운드 규칙</p>
</blockquote>
<p>이 곳에서 새 규칙을 설정하여 사용하는 port의 인바운드를 설정할 수 있다
여기까지는 검색해서 많이 찾아볼 수 있는 부분이었다.</p>
<p>하지만 그럼에도 열리지 않아서 위의 과정을 반복해보았지만 변하는 것은 없었다. 도대체 왜 그럴까 생각하고 netstat을 찍어보았는데..</p>
<h3 id="netstat으로-이미-사용하는-포트인지-확인">netstat으로 이미 사용하는 포트인지 확인</h3>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/ec531a20-ee1e-4fa9-bce5-39008c5588b9/image.png" /></p>
<blockquote>
<p>netsat -ano</p>
</blockquote>
<p>명령 프롬프트에 위의 명령어를 입력하여 port를 잘 listening하고 있는지 확인해보았다.</p>
<p>확인 결과, <strong>이미 3000번 포트를 사용하는 프로세스가 있었다.</strong>
우측에 표기된 pid를 확인하고 찾아본 결과 그것은 <strong>iphelper</strong>라는 프로그램이었다.</p>
<blockquote>
<p><strong>이미 다른 프로세스가 해당 포트를 bind하고 있었기에 생긴 문제였다</strong></p>
</blockquote>
<p>그렇다면 iphelper를 꺼버리면 해결이 되겠다 생각했지만, 그 전에 어떤 프로세스인지 확인을 할 필요가 있었다. 어디선가 사용하고 있는 프로세스 일 수도 있기 때문이다.</p>
<p>본인은 이미 프로젝트 진행 중에 3000번 포트를 사용하겠다고 고정한 상태였기에 프로세스를 끄는 판단을 하였지만, 실제로는 <strong>다른 포트를 사용하면</strong> 해결되는 문제다.</p>
<h3 id="iphelper란">iphelper란?</h3>
<blockquote>
<p>iphelper는 다음과 같은 일을 한다고 한다</p>
</blockquote>
<ul>
<li>IPv6 transition technologies</li>
<li>IPv6 connectivity</li>
<li>Teredo tunneling</li>
<li>6to4 tunneling</li>
<li>ISATAP tunneling</li>
<li>Network Layer Discovery</li>
</ul>
<p>참고한 사이트를 번역하자면, IPv4, IPv6 네트워크 간의 깔끔한 연결을 보장하고 유지함에 필요한 프로세스라고 한다.</p>
<p>가장 필요한 정보인 <strong>&quot;iphelp를 꺼도 되는가?에 대해서는 권장하지 않는다&quot;</strong>라고 기술되어있다.</p>
<p>그 이유는 네트워크 연결 관련 issue를 일으킬 수 있고, 네트워크 기능이 똑바로 작동하지 않을 수 있다하였다. 그러나 다른 곳을 찾아보니 <strong>&quot;IPv6을 사용할때에만 위의 사항이 적용된다&quot;</strong>라고 기술되어 있다. </p>
<blockquote>
<p>결론적으로 필자는 <strong>항상 사용하는 노트북이 아니었으며, IPv6을 사용하지 않으면서 현재 반드시 3000번 포트로 열어야하는 상황이었기에 iphelper를 꺼버렸고 마침내 포트를 열어 백엔드 서버와 통신할 수 있었다.</strong></p>
</blockquote>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/f63a72ec-7e70-4b8b-af74-381110422b93/image.png" /></p>
<p>참고하시는 분들도 자신의 상황에 맞추어 포트를 바꾸거나, iphelper 프로세 관련을 알아보는 것이 좋을 듯 하다</p>
<h4 id="참고-링크">참고 링크</h4>
<p><a href="https://malwaretips.com/blogs/service-host-ip-helper-process/#google_vignette">https://malwaretips.com/blogs/service-host-ip-helper-process/#google_vignette</a></p>
<p><a href="https://www.lifewire.com/what-is-iphlpsvc-in-windows-10-4781139#:~:text=Although%20the%20service%20is%20safe,or%20disable%20it%20from%20startup">https://www.lifewire.com/what-is-iphlpsvc-in-windows-10-4781139#:~:text=Although%20the%20service%20is%20safe,or%20disable%20it%20from%20startup</a>.</p>