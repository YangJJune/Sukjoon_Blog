<h3 id="serverless-1--1-소켓-통신과-보안">serverless 1 : 1 소켓 통신과 보안</h3>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/8deaaf79-d309-4955-a0fd-08a2559f898f/image.png" /></p>
<ul>
<li>#2 Server를 포함한 일대일 아키텍처는 저번 주에 다루었으나,  #1의 아키텍처를 구현하던 중 보안 문제와 직결하였다.</li>
</ul>
<blockquote>
<p><strong>접속한 클라이언트의 IP를 서버가 지속적으로 관리</strong></p>
</blockquote>
<ul>
<li>모바일 네트워크의 습성을 습득</li>
<li>IP가 계속 바뀔 수 밖에 없음, 이를 방지하고자 모바일 IP 체계 도입</li>
<li>Home Network, Foreign Network으로 나누어 진행</li>
<li>자세한 건 링크 참조</li>
</ul>
<p>당시 적었던 신청서 내용입니다</p>
<p>이는 모바일 IP의 변동성에 대한 고려만 있는 것을 알 수 있습니다. 그리고 한 가지 기본적인 사항을 간과하였는데, 바로 포트와 공인 IP입니다.</p>
<h3 id="nat-network-address-translation">NAT (Network Address Translation)</h3>
<ul>
<li>컴퓨터 네트워킹에서 쓰이는 용어로서, IP 패킷의 TCP/UDP 포트 숫자와 소스 및 목적지의 IP 주소 등을 재기록하면서 라우터를 통해 네트워크 트래픽을 주고 받는 기술</li>
</ul>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/707856b5-9704-404a-8c51-835c7aa0b088/image.png" /></p>
<p><strong>NAT를 왜 쓰는건가??</strong></p>
<ul>
<li><strong>공인 IP의 절약</strong><ul>
<li>하나의 공인 IP에 포트와 사설 IP를 대응시키는 것으로 결과적으로 필요 IP 개수를 줄일 수 있음</li>
</ul>
</li>
<li><strong>보안을 위함</strong><ul>
<li>내부망 / 외부망 사이에 방화벽을 둘 수 있어 외부 공격으로 부터 보호</li>
</ul>
</li>
</ul>
<h3 id="문제사항">문제사항</h3>
<ul>
<li>모바일 기기는 NAT를 본인이 관리하지 않는다</li>
<li><strong>데이터 네트워크일 때</strong><ul>
<li>기본적으로 모바일 기기는 인터넷과 연결되어있지 않다. 우리가 인터넷과 연결이 가능한 것은 직접 인터넷에서 데이터를 받아오는 것이 아닌 통신사가 대신 데이터를 정리하여 보내주는 것이다</li>
<li>모바일 기기의 NAT는 통신사에서 관리하기에, 통신사의 보안 수준을 해제하거나 포트를 임의로 여는 것은 불가하다</li>
</ul>
</li>
<li><strong>Wi-Fi 일 때</strong><ul>
<li>와이파이의 주인이 본인이라면, 포트포워딩을 통해 해결이 가능하다. 그러나 이 순간 모바일 통신으로서 동작하지 않는 것과 마찬가지다</li>
<li>와이파이의 주인이 타인인 경우, 권한이 없어 포트를 열 수 없고 그 어떤 보안정책이든 우회가 불가능하다</li>
</ul>
</li>
</ul>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/f780dfad-ccb0-49f8-9915-dcf9ab7517f1/image.png" />
신사의 데이터 네트워크 통신 방식은 들어가면 더 자세히 팔 수 있으나, 본 프로젝트에서 주된 내용은 아니기에 간략히 정리된 사진을 첨부한다.</p>
<h3 id="결론">결론</h3>
<p>안타깝게도 결론은 <strong>Serverless</strong>한 아키텍처를 만드는 것은 불가능했다는 결론이었다.
따라서 아키텍처는 고정되었다고 볼 수 있으며, 앞으로의 학습은 이를 제외한 부분에서 속도와 효율을 올릴 수 있는 방법에 대한 강구가 될 것이다.</p>
<h2 id="1---n-소켓-통신-및-api-통신">1 :  N 소켓 통신 및 API 통신</h2>
<h3 id="향후-환경에서-db를-써야할까"><strong>향후 환경에서 DB를 써야할까?</strong></h3>
<p><strong>DB I/O 시간이 긴 이유</strong></p>
<p><strong>1. 디스크 접근 속도 한계</strong>
HDD: 물리적으로 디스크를 회전시키고 헤드를 움직여 데이터를 읽어야 하기 때문에 느리다
SSD: 빠르지만 여전히 메모리(RAM)보다 느리다</p>
<p><strong>2. 랜덤 액세스와 순차 액세스</strong>
HDD의 경우 헤더가 움직이기 때문에 물리적인 시간 소요</p>
<p><strong>3. 네트워크 오버헤드</strong>
클라우드 환경에서는 네트워크를 통해 데이터를 주고받으므로 추가 지연이 발생한다</p>
<p><strong>4. 트랜잭션 및 동시성 제어</strong>
ACID 특성을 보장하기 위한 로그 기록, 락(lock) 처리, 동시성 제어가 필요해 추가적인 시간 소요.</p>
<p><strong>5. 캐싱이 안 된 경우</strong>
자주 사용하는 데이터는 캐싱(DB 버퍼 캐시, OS 캐시 등)되지만, 캐시에 없는 데이터는 디스크에서 직접 읽어야 하므로 속도가 느려짐.</p>
<h3 id="db를-쓰는-이유--데이터의-영속성">DB를 쓰는 이유 : 데이터의 영속성</h3>
<ul>
<li>RAM에 단순히 저장한 데이터들은 전원이 꺼지면 휘발된다.</li>
<li>그렇다고 데이터를 파일입출력으로 관리할 수도 없다</li>
</ul>
<p><strong>근데 안 쓰고하면 데이터가 휘발될 수 있지 않은가?</strong></p>
<ul>
<li>어차피 데이터는 클라이언트로부터 지속적으로 제공된다</li>
<li>서비스가 진행되는 동안엔 전원이 내려가지 않는다</li>
<li>만일 DB까지 갈 이유도 없다면 DB를 사용하는건 오버엔지니어링이 아닐까</li>
</ul>
<h3 id="그럼-결국-1n-통신은-이미-아키텍처가-결정난-셈이다">그럼 결국 1:N 통신은 이미 아키텍처가 결정난 셈이다.</h3>
<p>중요한 건 ThreadPool을 관리하고 일을 처리하는 방식이 될 것이다</p>
<h3 id="threadpool이-갑자기">ThreadPool이 갑자기?</h3>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/2877f48e-376c-4649-8609-6dc68a3afa1c/image.png" />
CS를 배우다 보면 concurrency라는 개념이 나온다. 동시성이라는 뜻으로, 작업을 언제나 하나씩만 진행을 하는 건 비효율적이니, 여러 작업을 동시에 수행할 수 있도록 프로그램을 짜는 것을 의미한다.
그래서 나온 개념이 <strong>Thread</strong>이다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/fa980133-0793-4864-98c8-4cef1a284577/image.png" /></p>
<p>Thread란 스케줄러가 독립적으로 관리할 수 있는 가장 작은 프로그래밍된 명령어 시퀀스로, 프로세스의 구성요소이다.
그리고 이러한 Thread가 여러 개 모여있는 Pool을 ThreadPool이라고 한다.</p>
<p>Thread Pool은 동시 실행을 위해 작업이 할당될 때까지 기다리는 여러 스레드를 유지하는 역할을 한다.
이는 모델의 성능을 높이고 수명이 짧은 작업에 대해 스레드가 생성되고 삭제되는 일로 인한 지연을 방지한다.</p>
<p>Thread Pool은 앞으로 소켓 관리를 할 때 가장 필수적인 하나의 그룹이 될 것이며, 많은 소켓을 관리하며 생기는 동기화 (Syncronization / Lock ) 문제를 해결해야할 것으로 보인다.</p>
<p>아래는 임의로 짠 1:N 아키텍처의 서버 코드이다.</p>
<h3 id="1n-아키텍처-서버-코드">1:N 아키텍처 서버 코드</h3>
<pre><code class="language-kotlin">package com.ysj.socket_test

import java.net.InetAddress
import java.net.ServerSocket
import java.util.concurrent.Executors

class TCPServer_N {
    fun openServer() {
        // 예외를 처리하기 위해 runCatching 사용
        runCatching {
            val server = ServerSocket(3000, 50, InetAddress.getByName(&quot;0.0.0.0&quot;))
//            val threadPool = ThreadPoolExecutor(10, 200, 60L, TimeUnit.SECONDS, LinkedBlockingQueue&lt;Runnable&gt;())
            val threadPool = Executors.newFixedThreadPool(10)
            for(i in 0..9) {
                threadPool.submit {
                    while (true) {
                        println(&quot;서버 열기&quot;)

                        val client = server.accept()
                        var num = -1000
                        val list = ArrayList&lt;packetData&gt;()
                        var cnt = 0
                        while (true) {

                            val request = ByteArray(1024)
                            val inputStream = client.getInputStream()
                            val outputStream = client.getOutputStream()

                            inputStream.read(request)
                            if (request.decodeToString().contains(&quot;END&quot;)) {
                                client.close()
                                break
                            }

                            list.add(
                                packetData(
                                    cnt,
                                    System.currentTimeMillis(),
                                    String(request, Charsets.UTF_8).slice(0..4)
                                )
                            )


                            if ((1..2).random() == 1) {
                                num += 1
                            } else {
                                num -= 1
                            }
                            val content = num.toString()

                            outputStream.write(content.toByteArray())
                            outputStream.flush()
                            cnt++
                        }
                    }
                }
            }

        }.onFailure {
            // 예외가 발생한 경우 스택 트레이스 출력
            it.printStackTrace()
        }
    }
}</code></pre>
<p>&nbsp;</p>
<pre><code class="language-kotlin">val threadPool = ThreadPoolExecutor(10, 200, 60L, TimeUnit.SECONDS, LinkedBlockingQueue&lt;Runnable&gt;())</code></pre>
<p>주석처리 된 해당 부분은 ThreadPool을 언제나 100개를 가지고 있으면 리소스 낭비가 있을 수 있기에 효율적으로 ThreadPool을 관리하고자 ThreadPoolExecutor를 뜯어보는 과정에서 짠 코드이나, 바로 사용할 수 있는 정도는 아니라 다음 주차에 다뤄보고자 한다.</p>
<p>참고자료
<a href="https://ohksj77.tistory.com/m/252">https://ohksj77.tistory.com/m/252</a>
<a href="https://velog.io/@ddangle/%EC%88%9C%EC%B0%A8Sequential-IO%EC%99%80-%EB%9E%9C%EB%8D%A4Random-IO">https://velog.io/@ddangle/%EC%88%9C%EC%B0%A8Sequential-IO%EC%99%80-%EB%9E%9C%EB%8D%A4Random-IO</a>
<a href="https://velog.io/@yarogono/%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%B2%A0%EC%9D%B4%EC%8A%A4%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94-%EC%9D%B4%EC%9C%A0">https://velog.io/@yarogono/%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%B2%A0%EC%9D%B4%EC%8A%A4%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94-%EC%9D%B4%EC%9C%A0</a></p>