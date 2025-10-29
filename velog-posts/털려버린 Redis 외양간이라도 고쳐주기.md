<img src="https://velog.velcdn.com/images/ysj7191/post/4d339200-45d1-4dc0-ad5f-429ef6fbe7d8/image.png" style="float: left;" width="480" />
<p style="clear: both;"></p>

<h2 id="상황발생">상황발생</h2>
<p>프로덕션 서버 세팅이 덜 되었을 때, 개발 서버를 프로덕션 서버처럼 쓰던 시기가 있었다.
그런데 갑자기 팀원들로부터 서버가 동작하지 않는다는 보고를 받았다.</p>
<p>어찌된 일일까? 지금부터 그걸 회고해보도록 하자. </p>
<h2 id="상태-확인">상태 확인</h2>
<p>문제가 생기면 어디서 문제가 생겼는지 추적하는 것이 우선이라고 생각한다.
우선 서버가 구동되는 컨테이너에서 무슨 문제가 생겼는지 파악해보았고, <code>io.lettuce.core.RedisConnectionException</code> 이 발생했음을 확인하였다.</p>
<p>따라서 Redis 컨테이너를 확인해보았는데 놀라운 일이 일어나고 있었다.</p>
<pre><code>1:S 19 Aug 2025 08:28:20.373 * Before turning into a replica, using my own master parameters to synthesize a cached master: I may be able to synchronize with the new master with just a partial transfer.
1:S 19 Aug 2025 08:28:20.373 * Connecting to MASTER [공격자 IP 주소]:24798
1:S 19 Aug 2025 08:28:20.374 * MASTER &lt;-&gt; REPLICA sync started
1:S 19 Aug 2025 08:28:20.374 * REPLICAOF [공격자 IP 주소]:24798 enabled (user request from 'id=45 addr=[공격자 IP 주소]:60296 laddr=172.19.0.2:6379 fd=9 name= age=3 idle=0 flags=N db=0 sub=0 psub=0 ssub=0 multi=-1 qbuf=50 qbuf-free=20424 argv-mem=27 multi-mem=0 rbs=1024 rbp=635 obl=0 oll=0 omem=0 tot-mem=22451 events=r cmd=slaveof user=default redir=-1 resp=3 lib-name= lib-ver=')
1:S 19 Aug 2025 08:28:20.583 # Error condition on socket for SYNC: Connection refused
1:S 19 Aug 2025 08:28:21.235 * Connecting to MASTER [공격자 IP 주소]:24798
1:S 19 Aug 2025 08:28:21.235 * MASTER &lt;-&gt; REPLICA sync started
.
.
.
1:S 20 Aug 2025 04:16:12.166 # Error condition on socket for SYNC: Connection refused
1:S 20 Aug 2025 04:16:12.963 * Connecting to MASTER [공격자 IP 주소]:24798
1:S 20 Aug 2025 04:16:12.963 * MASTER &lt;-&gt; REPLICA sync started
1:S 20 Aug 2025 04:16:13.168 # Error condition on socket for SYNC: Connection refused</code></pre><p>저 상태로 20시간 가량이 지난 상황에서 발견하고 말았다.
정말 수상한 위치에 있는 해외 IP로부터 REPLICAOF 명령어가 실행된 이후 무수히 많은 Connection이 Refuse되고 있었던 것을 확인하였다.</p>
<p>아무래도 어딘가 보안이 뚫려있었던 거 같다.
그럼 도대체 뭘 하려고 한걸까? Master? Slave? Replica가 뭘까?</p>
<h2 id="master-slave-그리고-replica">Master, Slave 그리고 Replica</h2>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/4c6de1f9-132c-4c8c-af60-fba41a5a7f6c/image.png" />
단어만 정리하자면 각각 주인, 노예, 복제라는 뜻이다.
주인 노예를 그림으로 그리면 다음과 같은데, Redis도 이와 같다.</p>
<h3 id="master-slave-패턴">Master-Slave 패턴</h3>
<p>Master-Slave 패턴에서 슬레이브는 마스터를 읽고 쓰며, 슬레이브의 데이터만 읽는다.
일반적으로 Slave는 2개 이상으로 구성된다. 그리고 Slave는 Master의 데이터를 실시간으로 복제한다.
(마스터가 죽은 경우 다른 하나의 슬레이브가 마스터 역할으로 전환하는 기능이 있다)</p>
<p>정리하자면, <strong>모든 쓰기 작업</strong>은 Master에서 처리되며, Slave에서는 일부 읽기 요청을 처리하는 구조이다.</p>
<h3 id="replicaof">ReplicaOf</h3>
<p>명령어가 입력된 서버를 지정한 서버의 복제 노드로 만드는 명령어이다.
즉, 해당 서버를 Slave 노드로 만들겠다는 뜻이다.</p>
<h2 id="상황정리">상황정리</h2>
<p>위와 같은 개념 속에서 로그를 다시 분석해보았다.</p>
<blockquote>
<ol>
<li>뭐야, 얘네 Redis가 열려있네?</li>
<li>Redis에 접속해서 공격해야지</li>
<li>Replicaof 시전해서 Redis를 slave 노드로 전환 (외부 공격자 데이터)</li>
<li>그러나 AWS에서 24798 아웃바운드 포트를 막아두어서 실패함 (Connection Refused)</li>
<li>명령어는 실행되어있으니까 현재 Redis는 slave 상태가 되어서 읽기 전용이 되었다. Write 불가한 상황</li>
<li>docker stop 후 start 하니 기본 config로 돌아가서 잘 동작함</li>
</ol>
</blockquote>
<img src="https://media1.tenor.com/m/UyeUDZucVzUAAAAd/shocked-surprised.gif" />

<p>정말 <strong>흠좀무</strong>한 일이 일어났다.</p>
<p>이를 막기 위해서는 외부로부터 Redis를 보호할 필요가 있다.</p>
<h2 id="redis-보안을-강화해보자">Redis 보안을 강화해보자</h2>
<h3 id="1-redis-격리">1. Redis 격리</h3>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/493f8ec1-8686-4f51-8d4d-0751be4893de/image.png" /></p>
<p>일반적으로 VPC 환경에서 ElastiCache를 사용하면 외부로부터는 직접 접근이 불가능하며 <code>터널링</code>을 통해서만 접근이 가능하다
터널링을 통해 접속하면 아무래도 보안 측면에서 내부 저장소를 공개하지 않기 때문에 안전하고, 관리하기 편할 것이다.</p>
<p>이를 위해 <code>Redis 컨테이너</code>를 개발 시에 접속하기 편하게 열어두었던 포트를 닫고, 내부에서만 접근 가능하게 변경하였다.
물론 <code>Spring 컨테이너</code>의 환경변수의 <code>REDIS_HOST</code>를 줄 때 localhost 또는 도커 네트워크 설정을 통해 올바른 값을 지정해야한다.</p>
<p>※ 당연하지만 반대로 말하면 연결된 인스턴스가 털리면 위험하므로 SSH authentication에 더욱 <strong>신경써야한다</strong>.</p>
<h3 id="2-bind--비밀번호-설정">2. bind / 비밀번호 설정</h3>
<p>Redis를 만약 도커 이미지로 바로 pull해서 사용하게 되면 <code>redis.conf</code>의 설정값, <code>protected-mode</code>가 no로 꺼져있을 것이다.
이를 yes로 바꾸게 되면 password와 bind된 ip를 지정할 수 있다.</p>
<ol>
<li><p><code>protected-mode</code> (보호 모드)
protected-mode는 Redis 서버를 외부로부터 보호하는 가장 기본적인 장치다. 
인증되지 않은 외부 접속을 자동으로 차단하여 보안을 강화한다.</p>
</li>
<li><p><code>bind</code> (접속 IP 제한)
bind 설정은 Redis가 어떤 IP 주소로부터의 연결 요청을 수락할지 지정합니다. 이는 Redis 서버가 노출되는 네트워크 범위를 제한하여 보안을 강화하는 데 필수적입니다.</p>
</li>
<li><p><code>requirepass</code> (비밀번호 설정)
Redis에 접속하기 위한 비밀번호를 입력하게 한다.</p>
</li>
</ol>
<h3 id="3-tls">3. TLS</h3>
<p>Redis에 TLS 연결을 할 수 있다. TLS란 Transport Layer Security라는 뜻으로,  인터넷에서 웹 브라우저와 서버 간 통신 등 데이터를 안전하게 주고받기 위해 사용하는 암호화 프로토콜이다.</p>
<p>이 설정을 키면 통신 암호화, 신원 확인, 데이터 무결성 등의 장점이 생긴다.</p>
<p><em>※ 엄밀히 말하자면 redis 접근 시 --tls 모드를 켜기만 하면 되기에 이번 일과는 무방하다고 볼 수 있으나 보안과는 연결되는 부분이라 같이 기재하였다.</em></p>
<h2 id="결론">결론</h2>
<p>개발 편의를 위해 포트를 마구 열어두고, 보호 모드 등을 끄는 것은 정말 위험한 일이라는 것을 다시금 확인할 수 있었다.
개발서버라서 정말 다행인 일이었고, 덕분에 AWS 보안그룹을 한번 정리하는 계기가 되었다.</p>