<h2 id="개요">개요</h2>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/bd695502-18ed-48be-82f8-829688a2f064/image.png" />
최근 KBO:Note라는 프로젝트를 진행하며 혹시라도 많은 사용자가 몰릴 것에 대비해 단일 서버에 얼마나 많은 부하를 걸 수 있는 지, 어디서 병목이 걸리는 지를 확인하고 HikariCP / Tomcat 쓰레드풀을 조정하던 중 docker-proxy라는 프로세스가 200% 가량의 CPU 부하를 잡아먹고 있음을 확인했다.</p>
<p>평소에 다른 프로젝트에서 Docker를 오케스트레이션 할 때는 보지 못했던 프로세스라 무슨 일을 하지, 왜 생겼는지 문득 궁금해졌다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/ad7f26ec-bff0-444a-8c1d-dc7945c50ea5/image.png" />그렇게 시작한 이번 Docker-Proxy 공부는 내가 잘못 사용하고 있었기 때문에 생긴 일이었음을 직시하게 해줬다.</p>
<h2 id="docker-proxy란">docker-proxy란?</h2>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/2ddc4992-024c-4f74-8b86-a59dcf4891cd/image.png" /></p>
<p><code>docker</code>는 기본적으로 컨테이너를 격리하는 정책을 사용한다. 이는 네트워크에도 마찬가지이다.</p>
<p><code>docker-proxy</code>는 도커 데몬이 컨테이너를 띄울 때 호스트와 컨테이너 간의 포트 포워딩을 위해 띄우는 <strong>유저 공간(Userland)의 TCP/UDP 프록시 프로세스</strong>이다.</p>
<p>도커는 기본적으로 리눅스 커널의 <code>iptables</code>를 이용해 네트워크 트래픽을 처리한다. 그러나 간혹 <code>iptables</code> 만으로는 해결 불가한 경우가 발생한다.</p>
<blockquote>
<p>① 다른 Docker 네트워크에 연결된 컨테이너가 서비스에 접근하려고 할 때
② 로컬 프로세스가 루프백 인터페이스를 통해 서비스에 접근하려고 할 때
&nbsp;
[출처] <a href="https://blog.ipspace.net/kb/DockerSvc/40-userland-proxy/">https://blog.ipspace.net/kb/DockerSvc/40-userland-proxy/</a></p>
</blockquote>
<p>이때 필요한 것이 <code>docker-proxy</code>이다.</p>
<p>즉, <code>docker-proxy</code>는 모든 트래픽을 처리하는 메인 일꾼이 아니라, 환경을 타지 않고 언제나 포트 매핑이 동작하도록 보장해 주는 <strong>'보험'이자 '예외 처리용(Fallback)' 프로세스</strong>이다.</p>
<p>그리고 만약 당신이 개발자라면 여기서 아마 문제를 느꼈을 것이다.
'보험이 왜 작동을 하고 있던거지?'</p>
<p>일단 이 의문은 잠시 미뤄두고 조금 더 진행해보도록 하겠다.</p>
<h2 id="도커-네트워크-bridge-vs-host-모드">도커 네트워크: bridge vs host 모드</h2>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/922ffaad-49de-45cd-a645-0501fdd16bfc/image.png" />이 문제를 맞닥뜨렸을 때 가장 먼저 들었던 생각은 다음과 같았다.</p>
<blockquote>
<p>&quot;NAT 문제인가? <strong>그럼 네트워크 모드를 bridge에서 host로 바꿔버리면 되는거 아닌가?</strong>&quot;
&amp;nbsp
(※ 물론 이 밖에도 userland-proxy 속성을 비활성화해서 이를 예방하는 방법이 존재한다)</p>
</blockquote>
<blockquote>
<p>*<em>Bridge (기본값) *</em>|  호스트와 컨테이너 네트워크가 격리되어 있으며, 따라서 포트 바인딩(<code>p 8080:80</code>)을 통해 통신한다.
&amp;nbsp
*<em>Host *</em>  | 컨테이너가 호스트의 네트워크 환경을 그대로 공유한다. 따라서 NAT를 거치지 않아 성능이 가장 좋지만, 포트 충돌 위험이 있어 추가적인 관리가 필요하며 컨테이너의 격리라는 도커의 핵심 철학을 포기해야 한다.</p>
</blockquote>
<p>상술했듯 당장 Host로 전환하여 문제를 해결할 수는 있다.</p>
<p>그러나 이 상황이 너무 찜찜했다. 각 서버 컨테이너들은 격리성을 완전히 잃었다고 볼 수 있으니 보안적인 측면에서나 인프라 관리 측면에서나 불안한 점이 많았다.</p>
<p>심지어는 개발자끼리 배포 과정 중에 포트 충돌로 인해 문제가 생기기도 하였다.</p>
<p>나는 이 문제 해결법이 혹시 잘못된게 아닌가라는 생각을 하였고 원론적인 부분보다 더 깊게 파보기로 했다.</p>
<h2 id="docker-proxy가-문제되는-이유">Docker-Proxy가 문제되는 이유</h2>
<p>다시 돌아와서 왜 빠르고 효율적인 커널(iptables)을 놔두고, 트래픽이 <code>docker-proxy</code>로 흘러가 CPU를 200%나 태우게 된 건지 살펴보기로 했다.</p>
<p>원인은 <code>docker-proxy</code>가 커널이 아닌 <code>User Space</code>에서 동작하는 프로세스라는 점에 있다.</p>
<p>커널 공간에서 동작하는 iptables나 뒤에 나올 L2 브릿지를 통한 통신에 비해 <code>User Space</code>에서의 프로그램이 직접 라우팅을 제어하므로 컨텍스트 스위칭 비용이 굉장히 크다. 
따라서 당시 테스트 상황의 2,400명의 VU 트래픽이 이 프록시를 타면서 오버헤드가 걸렸으며, 이는 <strong>성능에 악영향을 끼칠 수가 있는 병목지점</strong>이라고 할 수 있다.</p>
<p>이쯤에서 userland-proxy를 다시 언급하자면, 위에서 이야기한 &quot;유저 공간&quot;을 사용하지 않고 커널 프로세스를 사용하겠다는, iptables를 통해서만 통신을 하겠다는 것이다.</p>
<blockquote>
<p>여담) <code>iptables</code> 자체는 User Space 프로그램이다. 
여기에 적용된 규칙이 커널 단의 <code>Netfilter</code> 프레임워크에서 동작하므로 사실상 커널 수준에서 처리한다고 이야기한다.</p>
</blockquote>
<p>따라서 daemon.json에 <code>userland-proxy : false</code>를 주면 docker-proxy 프로세스는 더 이상 올라오지 않는다.</p>
<p>여기까지의 해결방법을 정리해보면 다음과 같은 문제점을 갖고 있다.</p>
<p><strong>1. 네트워크 모드를 Host로 바꾼다</strong></p>
<ul>
<li><strong>포트 충돌</strong>
호스트의 포트를 직접 점유하므로, 동일한 서비스를 여러 개 띄우려면 포트를 직접 관리해야하는 공수가 생긴다. </li>
<li><strong>보안 격리 해제</strong>
컨테이너가 호스트의 네트워크 스택에 직접 접근하므로, 컨테이너가 탈취당할 경우 호스트 네트워크 전체가 위험에 노출된다. 도커의 철학 자체가 깨지게 된다.</li>
<li><strong>이식성 저하</strong>
특정 호스트의 포트 상황에 의존하게 되므로, 다른 서버로 옮길 때 꼬일 가능성이 높아진다.</li>
</ul>
<p><strong>2. userland-proxy를 false로 한다</strong></p>
<ul>
<li><strong>커널 의존성 심화</strong>
네트워크 처리에 문제가 생겼을 때 리눅스 커널 로그(dmesg)를 뒤져야 하며, 다른 쪽에서도 iptables를 사용하거나 kubernetes 환경인 경우 규칙 설정에 충돌이 발생할 가능성이 높아진다.</li>
<li><strong>안정성 약화</strong>
docker-proxy 는 어떻게든 패킷을 컨테이너에 무사히 보내기 위해 존재합니다. 이를 끈다는 것은 안정성의 약화를 야기할 수 있다.</li>
<li><strong>특수 환경에서의 이슈 (Hairpin NAT 등)</strong>
루프백 통신이나 IPv6 변환 등에서 커널 설정이 완벽하지 않으면 패킷 유실이 발생할 수 있다. 
(다만, 현대적인 프로덕션 배포 환경에서는 이 옵션을 false로 끄고 사용하는 것이 오히려 보편적인 성능 최적화 방법이기도 하다)</li>
</ul>
<h2 id="어떻게-해결하는게-맞을까">어떻게 해결하는게 맞을까?</h2>
<p>위에서 언급했듯 docker-proxy는 보험의 느낌이라고 볼 수 있다.
그러나 --network host 옵션은 <strong>격리성을 깨버린다.</strong>
iptables는 <strong>관리하기 복잡하고 변수가 많다.</strong></p>
<p>이 모든 걸 해결하는 것이 바로 <code>사용자 정의 도커 네트워크(user-defined bridge)</code>이다.</p>
<h3 id="사용자-정의-네트워크란">사용자 정의 네트워크란?</h3>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/6d3fdafe-62dd-498e-8797-16cad4eafcfd/image.png" /></p>
<p>도커를 설치하면 기본적으로 docker0라는 이름의 기본 브릿지(Default Bridge) 네트워크가 생성된다. 별도의 설정 없이 컨테이너를 띄우면 모두 이곳에 소속된다.</p>
<p>하지만 <code>사용자 정의 네트워크(user-defined bridge)</code>는 말 그대로 사용자가 목적에 맞게 직접 설계한 가상 네트워크 대역이다. <code>docker network create</code> 명령으로 생성하며, 특정 서비스군(예: API 서버와 DB)을 하나의 격리된 브릿지 대역으로 묶어줄 수 있다.</p>
<h3 id="왜-사용자-정의-네트워크를-써야-하는가">왜 사용자 정의 네트워크를 써야 하는가?</h3>
<p>기본 브릿지(<code>docker0</code>)와 비교했을 때 사용자 정의 네트워크를 써야 하는 이유는 명확하다.</p>
<p><strong>1. 내장 DNS를 통한 서비스 디스커버리</strong>
컨테이너의 IP가 바뀌어도 컨테이너 이름(예: <code>main-db</code>)으로 통신할 수 있다.
<em>(Default Bridge에서는 지원되지 않는다)</em></p>
<p><strong>2. 보안과 격리</strong>
같은 네트워크에 속한 컨테이너끼리만 자유롭게 통신하고, 외부나 다른 네트워크의 접근은 철저히 차단할 수 있다.</p>
<p><strong>3. 불필요한 중계(Proxy) 제거</strong>
가장 중요한 지점이다. 사용자 정의 네트워크 내에서는 서로를 '이웃'으로 인식하기 때문에, -p 옵션을 주지 않으면 <code>docker-proxy</code>의 간섭을 원천 차단할 수 있다.</p>
<p>사용자 정의 네트워크(<code>user-defined bridge</code>)에서 성능 이득이 발생하는 핵심은 <strong>호스트의 개입 최소화</strong>에 있다.
동일한 브릿지 네트워크에 속한 컨테이너 간 통신 시, 패킷은 다음과 같은 경로를 거친다.</p>
<blockquote>
<p><strong>1. L2 영역의 포워딩</strong> : 송신 컨테이너가 패킷을 쏘면, 커널의 <strong>가상 스위치(Linux Bridge)</strong>가 목적지 MAC 주소를 확인한다
<strong>2. 경로 스킵(Shortcut)</strong> : 패킷이 호스트의 L3(IP) 라우팅이나 무거운 iptables NAT 검문소를 거치지 않는다. 당연히 유저 공간(L7)의 <code>docker-proxy</code>로 데이터가 올라가는 일도 없다
<strong>3. 패킷 전달</strong> : 커널 내부에서 패킷이 바로 옆 컨테이너의 가상 랜선(veth) 포트로 전달된다</p>
</blockquote>
<p>여기서 오해하지 말아야 할 점은 패킷의 <strong>L3/L4 레이어 자체가 사라지는 것이 아니라는 점이다.</strong> 패킷은 여전히 표준 프로토콜을 따르지만, 호스트(OS)가 이 패킷을 처리하기 위해 상위 레이어(NAT, Proxy 등)까지 까보며 연산할 필요가 없어지는 것이다</p>
<p>결과적으로 위에서 언급한 여러 문제도 대부분 해결이 된다.</p>
<p>격리성을 유지할 수 있고, <code>docker-proxy</code>로 최후의 보루는 유지되며 속도마저도 더 빠르다는 것을 확인할 수 있었다.</p>
<h2 id="마치며">마치며</h2>
<p>사실 이번 내용은 본인의 미숙함보다도 귀찮음에서 왔던 엔지니어링에서 비롯되었다.</p>
<p><strong>&quot;뭐, 작동하잖아&quot;</strong>라는 잘못된 생각이 이런 화를 불러왔지만, 오히려 이 기회에 단순히 <strong>'도커 네트워크를 사용해야한다'</strong> 라는 원칙보다는 <strong>'왜 도커 네트워크를 사용해야하는가?'</strong>를 배울 수 있었다는 것이 전화위복이 되었다고 생각한다.</p>
<p>여담이지만 항상 주변 사람들에게 하는 말이 있는 필자의 신조가 있다.</p>
<blockquote>
<p>실패할 것이라 생각하고 하지 않는 것과 직접 해보고 실패하는 것에는 분명한 차이가 있다</p>
</blockquote>
<p>이번 일과 완벽하게 일치하지는 않지만 직접 해보고 실패한 경우 이러한 지식과 경험을 얻을 수 있다는 점에서 한번 가져와보았다.</p>