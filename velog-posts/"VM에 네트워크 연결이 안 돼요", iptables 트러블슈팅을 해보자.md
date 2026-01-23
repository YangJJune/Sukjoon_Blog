<h2 id="0서론">0.서론</h2>
<p>KVM 가상화 환경을 구축하다 호스트 서버(Host)까지는 패킷이 정상적으로 도달하는데, 내부의 가상머신(VM)으로는 통신이 되지 않는 상황을 마주하게 되었다</p>
<p>문제를 해결하기 위해 <code>iptables</code>를 사용했지만, 패킷 흐름과 체인에 대한 이해가 부족해 여러 시행착오를 겪었다</p>
<p>따라서 이번엔 <code>iptables</code>와 트러블슈팅 과정을 정리하고자 한다</p>
<hr />
<h2 id="1-iptables란">1. iptables란?</h2>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/4717c815-ccc3-434c-bfb9-e2334879a092/image.png" /></p>
<p><code>iptables</code>는 관리자가 패킷 처리 규칙을 설정하는 유저 스페이스(User Space) 도구다.</p>
<ul>
<li><p><strong>작동 계층</strong>
주로 <strong>L3 / L4 수준에서 동작</strong>하며, IP 주소와 포트 번호를 기준으로 패킷을 제어한다</p>
</li>
<li><p><strong>L7이 아닌 이유..?</strong>
HTTP 헤더나 URL 경로 등을 분석하는 L7 영역은 Nginx와 같은 리버스 프록시가 담당하며, iptables는 <strong>이 단계 이전의 패킷 흐름을 관리</strong>한다 (데이터를 비교할 때도 바이트 수준에서 다룬다)</p>
</li>
</ul>
<h3 id="②-netfilter-커널-공간의-실행-엔진">② Netfilter: 커널 공간의 실행 엔진</h3>
<p>의외로 iptables는 유저 스페이스에서 동작하는 프로세스이다
실제로 <strong>리눅스 커널 내부에서 패킷을 낚아채고 규칙에 따라 처리하는 실체는 Netfilter라는 프레임워크</strong>이다</p>
<p>iptables로 규칙을 입력하면, 커널 내의 Netfilter가 패킷의 이동 경로 중 특정 지점(Hook)에서 해당 규칙을 실행하는 방식이다</p>
<hr />
<h2 id="2-패킷의-이동-경로와-체인chain">2. 패킷의 이동 경로와 체인(Chain)</h2>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/09b057c6-bb54-4aa9-bb11-b6a88b47ffaf/image.png" /></p>
<p>패킷이 서버에 들어와서 나갈 때까지의 여정은 크게 세 가지 경로로 분류된다</p>
<h3 id="1-local-input">1. Local Input</h3>
<p>외부에서 호스트 본체(Nginx 등)로 향하는 패킷</p>
<ul>
<li>경로: PREROUTING -&gt; INPUT -&gt; 호스트 애플리케이션</li>
</ul>
<p>&nbsp;</p>
<h3 id="2-forwarding">2. Forwarding</h3>
<p>외부에서 호스트를 거쳐 가상머신(VM)으로 향하는 패킷</p>
<ul>
<li>경로: PREROUTING -&gt; FORWARD -&gt; 가상머신(VM)</li>
</ul>
<p>&nbsp;</p>
<h3 id="3-local-output">3. Local Output</h3>
<p>호스트나 VM에서 외부로 나가는 패킷</p>
<ul>
<li>경로: 애플리케이션 -&gt; OUTPUT -&gt; POSTROUTING</li>
</ul>
<p>&nbsp;</p>
<hr />
<h2 id="3-실전-트러블슈팅-사례">3. 실전 트러블슈팅 사례</h2>
<h3 id="1-규칙-세팅을-마쳤으나-제대로-통신이-되지-않는다">#1 규칙 세팅을 마쳤으나 제대로 통신이 되지 않는다</h3>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/92be326c-f957-419b-bd01-715f080722f3/image.png" /></p>
<p>외부 클라이언트가 VM에 접속은 되지만 응답을 받지 못한다면, <strong>응답 패킷의 출발지 주소 불일치 문제</strong>를 의심해야 한다</p>
<blockquote>
<ul>
<li>VM의 사설 IP(192.168.x.x)를 가진 패킷이 그대로 인터넷으로 나가면, 클라이언트는 </li>
<li><em>자신이 요청한 호스트 IP가 아니라는 이유로 패킷을 폐기*</em>한다</li>
</ul>
</blockquote>
<ul>
<li>또한 인터넷 라우터(ISP) 단계에서도 사설 IP 패킷은 <strong>보안 규정상 차단될 확률이 높다</strong></li>
</ul>
<p>따라서 패킷이 호스트를 떠나기 직전(POSTROUTING)에 출발지 IP를 호스트의 공인 IP로 변환해주는 <code>MASQUERADE</code> 설정이 필요합니다.</p>
<pre><code class="language-shell">sudo iptables -t nat -A POSTROUTING -j MASQUERADE</code></pre>
<p>&nbsp;
&nbsp;</p>
<h3 id="2-prerouting로-인한-nginx-설정-무시-문제">#2 PREROUTING로 인한 Nginx 설정 무시 문제</h3>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/3e71d7a8-1d11-4c97-a098-21035189d896/image.png" /></p>
<p>분명히 규칙을 추가했는데도 Nginx가 응답하지 않거나, 추가한 규칙이 작동하지 않는다면 두 가지 관점에서 우선순위를 확인해야 한다.</p>
<p>** 1) PREROUTING 문제**</p>
<table>
<thead>
<tr>
<th align="center">올바르게 Nginx에 전달되는 경우</th>
<th align="center">PREROUTING으로 빠져나간 경우</th>
</tr>
</thead>
<tbody><tr>
<td align="center"><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/dd7e7092-c795-44f5-9a31-31399c1c6aba/image.png" /></td>
<td align="center"><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/80f3c526-81c7-4158-8552-ebf98853ad8a/image.png" /></td>
</tr>
</tbody></table>
<p>80번 포트 Nginx 설정을 마쳤는데, 연결이 되지 않았다
심지어는 <strong>access.log / error.log에도 잡히지 않았다</strong></p>
<p>이는 <strong>PREROUTING 규칙을 잘못 설정했을 가능성</strong>을 봐야한다.
PREROUTING 체인은 패킷이 호스트의 로컬 프로세스(INPUT)에 도달하기 전인 최상단에서 작동한다. </p>
<p>여기서 포트 포워딩(DNAT)을 설정하면 <strong>패킷은 Nginx를 만나기도 전에 VM으로 목적지가 바뀌어 FORWARD 경로로 빠진다.</strong></p>
<blockquote>
<p>Nginx는 신호를 기다리고 있는데 신호를 주지 않고 있는 상황이라는 의미이다</p>
</blockquote>
<p>*<em>2) 규칙 우선순위로 인해 다른 규칙을 덮어쓴 경우 *</em></p>
<p>그런데 규칙을 제대로 추가하였는데도 해당 포트를 여전히 이용할 수 없는 경우가 있다</p>
<p>Nginx에도 여전히 로그가 잡히지 않는다</p>
<p>이때는 <strong>다른 규칙이 본인이 설정한 규칙보다 상단</strong>에 있어 원치 않는 동작을 하고 있는 경우를 생각할 수 있다</p>
<p>다음과 같은 명령어로 확인할 수 있다.</p>
<pre><code class="language-shell"># 필터(Filter) 테이블의 규칙과 줄 번호 확인
sudo iptables -L -n -v --line-numbers

# NAT 테이블(포트 포워딩, 마스커레이드 등)의 규칙 확인
sudo iptables -t nat -L -n -v --line-numbers</code></pre>
<blockquote>
<p>iptables 규칙은 위에서부터 아래로 순차적으로 적용된다
&nbsp;
만약 -A (Append)를 사용하여 규칙을 리스트 맨 끝에 추가했는데, 그보다 윗줄에 REJECT나 DROP 규칙이 있다면 내 규칙은 영원히 실행되지 않는다 
&nbsp;
특히 Libvirt(KVM)나 Docker는 자동으로 복잡한 제한 규칙을 상단에 배치하는 경우가 많습니다.ㅗ</p>
</blockquote>
<p>따라서 중요한 허용 규칙은 반드시 체인의 맨 위(1번)로 삽입해야 한다</p>
<pre><code class="language-shell">sudo iptables -I FORWARD 1 -p tcp -d [VM_IP] --dport [원하는 포트] -j ACCEPT</code></pre>
<p>&nbsp;
&nbsp;</p>
<h3 id="3-규칙의-영구-저장-persistence">#3. 규칙의 영구 저장 (Persistence)</h3>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/3f876080-498d-4c2e-90d8-f46c7f53e67b/image.png" />
제미나이와 함께 작업을 하면서 지속적으로 날아가니까 꼭 저장을 하라는 언급이 있었다</p>
<p>그러나 이를 무시하고 일단 구동하는게 우선이라고 생각한 나는 무시하고 진행을 했고...</p>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/13db8f55-bf8d-458f-922a-2a171aabd78e/image.png" /></p>
<p>iptables 명령어는 메모리에서 동작하므로 <strong>서버 재부팅 시 모든 설정이 초기화</strong>된다
Ubuntu 환경에서는 다음과 같이 설정을 영구화할 수 있다</p>
<pre><code class="language-shell">sudo apt install iptables-persistent
sudo netfilter-persistent save</code></pre>
<p>&nbsp;
&nbsp;</p>
<hr />
<h2 id="4-윈도우에서는-안-해도-되던데">4. 윈도우에서는 안 해도 되던데?</h2>
<p>윈도우(Hyper-V, WSL2) 환경에서는 WinNAT 서비스가 이러한 NAT 설정을 백그라운드에서 자동으로 관리한다
따라서 사용자가 직접 주소 변환 규칙을 세우지 않아도 되는 편리함이 있다</p>
<p>반면, 리눅스(KVM) 환경은 시스템 관리자가 iptables를 통해 패킷의 흐름을 직접 설계하고 제어할 수 있는 유연성을 제공한다
이는 시스템 구조를 밑바닥부터 이해하고 최적화할 때 강력한 장점이 된다</p>
<hr />
<h2 id="5-마치며">5. 마치며</h2>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/dea45f64-4304-4001-9bfb-8cb4ae818a98/image.png" /></p>
<p>리눅스 iptables의 동작과정을 전체적으로 이해하면 그다지 어렵지 않다
사실 우리는 이걸 iptime 공유기와 윈도우 방화벽을 통해 자주 봐왔기 때문이다 </p>
<p>가상환경에서의 네트워크 구축의 핵심은 <strong>패킷이 어느 시점에, 어떤 테이블을 거쳐 주소가 변환되는지 정확히 추적하는 것</strong>이다
특히 외부 요청이 들어올 때의 DNAT와 응답이 나갈 때의 MASQUERADE가 한 쌍으로 동작해야 완전한 통신이 이루어진다는 점을 유의해야 하며, 연결에 문제가 있을 때는 언제나 문제를 잘라서 ping을 통해 각각의 노드끼리 통신이 되는지, 안 되는지로 장애를 격리해야한다</p>
<hr />
<h3 id="reference">Reference</h3>
<p><a href="https://ww2.coastal.edu/mmurphy2/oer/iptables/introduction/">https://ww2.coastal.edu/mmurphy2/oer/iptables/introduction/</a>
<a href="https://en.wikipedia.org/wiki/Iptables">https://en.wikipedia.org/wiki/Iptables</a></p>