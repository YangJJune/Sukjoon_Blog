<h3 id="epoll이란">Epoll이란?</h3>
<p>Epoll은 간략히 정의하면 다음과 같다</p>
<p>커널이 어떤 파일 디스크립터가 변경되었는지 알려주면, 프로세스는 커널과 프로세스 파일 디스크럽터의 변경을 이벤트처럼 알 수 있는 기능을 제공하는 것이다.</p>
<p>동작 과정 및 자세한 내용은 아래 링크에서 확인하면 상세히 설명이 되어있다.</p>
<p><a href="https://medium.com/@avocadi/what-is-epoll-9bbc74272f7c">https://medium.com/@avocadi/what-is-epoll-9bbc74272f7c</a></p>
<h3 id="문제상황-설명">문제상황 설명</h3>
<p>현재 우리 서버는 다음과 같이 설계되어있다.</p>
<blockquote>
<ol>
<li>socketService.py가 epoll 객체를 만든다.</li>
<li>epoll을 통해 소켓을 통해 데이터가 들어오거나, 새로운 연결이 일어난 경우를 감지한다.</li>
<li>프론트와 약속된 형식의 Json 데이터를 주고 받으며 실시간 위치 공유 서비스를 진행한다.</li>
</ol>
</blockquote>
<p>그런데 2번 과정에서 정상적이지 않은 접근이 발생함을 확인하였다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/63918178-c8e4-4b6e-8f3b-882b0ff982da/image.png" /></p>
<p>위는 이번 프로젝트는 아니고, 명확한 설명을 위해 다른 프로젝트에서 경험한 정상적이지 않은 접근이 찍힌 로그를 사진으로 가져왔다. </p>
<p>정상적이지 않은 접근이란 위와 같이 “<a href="http://example.com/%E2%80%9D">http://example.com/”</a> “<a href="http://example.com/doc/index.html%E2%80%9D">http://example.com/doc/index.html”</a> ““<a href="http://example.com/download/powershell/%E2%80%9D">http://example.com/download/powershell/”</a> 같이 서버 도메인을 향해 스크래핑처럼 임의의 요청을 보내는 접근을 말한다.</p>
<p>소켓 서버에도 마찬가지로 동일한 접근이 이루어졌는데, 문제는 이에 대한 처리가 부족하여 예상하지 못한 에러가 발생하여 서버가 터져버리는 현상이 지속되었다.</p>
<h3 id="http-통신-역시도-tcp-이다">HTTP 통신 역시도 TCP 이다</h3>
<p>근데 도대체 왜 단순한 HTTP 요청이 소켓을 통해 들어오나 의문이 들었다. 확인 결과, 브라우저를 통한 특정 주소 접속 역시 그 과정에서 소켓 통신을 한다는 사실을 알게 되었다.</p>
<blockquote>
<p>At the transport layer, TCP handles all handshaking and transmission details and presents an abstraction of the network connection to the application typically through a <a href="https://en.wikipedia.org/wiki/Network_socket">network socket</a> interface.
<br /><a href="https://en.wikipedia.org/wiki/Transmission_Control_Protocol">https://en.wikipedia.org/wiki/Transmission_Control_Protocol</a></p>
</blockquote>
<p>HTTP는 HTTP/3.0 (QUIC)을 사용하지 않는 이상 <strong>TCP를 사용</strong>한다.</p>
<p>그리고 <strong>TCP는 3 way handshake를 통해 소켓을 establish 하고 데이터를 통신</strong>한다. </p>
<p>(솔직히 필자는 이번 기회에 다시 알았다. 연결지향형이라는게 소켓을 열어두고 통신한다는 의미였다니..)</p>
<p>따라서 HTTP 요청을 통해 서버에 요청을 보내어도 소켓이 열리기 때문에 epoll은 이 역시도 이벤트로 감지하는 것이었다.</p>
<h3 id="올바른-해결방법">올바른 해결방법</h3>
<p>일단 필자는 발생하는 에러에 대하여 다음과 같이 해결을 하였다.
<strong>1. 발생할 수 있는 예외들을 예외처리 해주기</strong>
*<em>2. 특수한 경우들에 대하여 소켓 정리해주기 *</em>(cleanup_connection)</p>
<pre><code class="language-python">epoll = select.epoll()

    HOST = '0.0.0.0'
    PORT = 9000
        # 소켓 세팅
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    server_socket.bind((HOST, PORT))
    server_socket.listen(5)
    server_socket.setblocking(0)

        # epoll에 등록
    epoll.register(server_socket.fileno(), select.EPOLLIN)
    fd_to_socket[server_socket.fileno()] = server_socket

    print(f&quot;Server started on {HOST}:{PORT}&quot;)

    try:
        while True:
            events = epoll.poll(1)
            for fd, event in events:
                sock = fd_to_socket[fd]

                # 새로운 연결 수락
                if sock == server_socket:
                    handle_new_connection(server_socket, epoll)

                # 클라이언트로부터 데이터 수신
                elif event &amp; select.EPOLLIN:
                    try:
                        originalData = sock.recv(1024)
                        if originalData == b'':
                            print(f&quot;[DISCONNECT] fd={fd} 클라이언트가 연결을 종료했습니다.&quot;)
                            cleanup_connection(fd, epoll)
                            continue  # 다음 이벤트로 넘어가기
                        splitedData = originalData.split(b&quot;\n&quot;)

                        for data in splitedData:
                            if data:
                                try:
                                    message = json.loads(data.decode())
                                                                # 1번째 예외 : JsonDecode Error    
                                except json.JSONDecodeError:
                                    send_to_socket(fd, {&quot;status&quot;: &quot;error&quot;, &quot;msg&quot;: &quot;invalid json&quot;})
                                    continue
                                                                # 2번째 예외 : message가 적절하지 않은 타입인 경우 (dict가 아닌 경우)
                                if not isinstance(message, dict):
                                    send_to_socket(fd, {&quot;status&quot;: &quot;error&quot;, &quot;msg&quot;: &quot;invalid message format&quot;})
                                    continue

                                msg_type = message.get(&quot;type&quot;)
                                # ~~~~ 실질적인 처리 (보안상 생략)~~~~ 
                                # 3번째 예외 : 적절하지 않은 type
                                else:
                                    send_to_socket(fd, {&quot;status&quot;: &quot;error&quot;, &quot;msg&quot;: &quot;unknown message type&quot;})
                    # 자체적으로 발생하는 Connection 에러 (소켓 연결 자체의 문제)
                    except (ConnectionResetError, ConnectionAbortedError, BrokenPipeError):
                        # send_to_socket(fd, {&quot;status&quot;: &quot;error&quot;, &quot;msg&quot;: &quot;Network Error&quot;})
                        cleanup_connection(fd, epoll)
                                # epoll에서 에러 발생
                elif event &amp; (select.EPOLLHUP | select.EPOLLERR):
                    print(&quot;ELIF!!!&quot;)
                    cleanup_connection(fd, epoll)

                                .....
</code></pre>
<p>그러나 이상적이지는 않다. 그 이유는 해당 예외처리에 걸리지 않는 문제가 발생하는 경우 여전히 서버가 터진다는 문제가 있으며, HTTP 통신인지 구분이 불가하다는 점에 있다.</p>
<p>따라서 좀 더 빠르게 확실하게 구분할 수 있는 방법들을 연구하였다.</p>
<p><strong>1) 수신된 데이터가 HTTP 통신인지 먼저 확인하는 로직의 추가</strong></p>
<pre><code class="language-python"># HTTP 통신은 다음과 같은 메소드 이름으로 시작한다는 것의 응용
if originalData.startswith(b&quot;GET&quot;) or originalData.startswith(b&quot;POST&quot;) or originalData.startswith(b&quot;HEAD&quot;) or b&quot;HTTP/&quot; in originalData:
    print(f&quot;[HTTP DETECTED] fd={fd} HTTP 요청 무시&quot;)
    cleanup_connection(fd, epoll)
    continue
</code></pre>
<p><strong>2) 커스텀 프로토콜 핸드쉐이크의 추가</strong></p>
<ul>
<li>말이 생소할 수 있으나 3 way handshake로 안정적인 연결을 하듯, 처음에 암구호를 대어 올바른 소켓 통신이 이루어짐을 검증하는 것이다.</li>
<li>이는 처음에 대는 암구호가 탈취당할 경우 마찬가지로 문제가 발생하기 때문에 여기서부터는 보안의 영역으로 넘어가게 된다.</li>
<li>ex) 소켓 통신 과정에서 클라이언트가 가장 처음에 서버로 “U-Compass”를 보내고 시작하기</li>
</ul>
<h3 id="여담">여담</h3>
<p>어이 없는 짓을 한번 시도했어서 포스트에 같이 기재합니다. 마음껏 웃고 가십시오.</p>
<pre><code class="language-python">except (ConnectionResetError, ConnectionAbortedError, BrokenPipeError):
    send_to_socket(fd, {&quot;status&quot;: &quot;error&quot;, &quot;msg&quot;: &quot;Network Error&quot;})
    cleanup_connection(fd, epoll)</code></pre>
<p>다음의 에러는 소켓 연결이 끊겼을 때 발생하는 에러들이다.
근데 대놓고 send_to_socket(fd, message) 메소드를 통해 에러 메세지를 보내고 있다...
그러다보니 분명히 try-except로 에러를 예외처리했음에도 자꾸 서버가 터져버리는 현상이 반복되어 의문을 가지고 있었는데, <strong>당연히 닫힌 소켓에다가 메세지를 보내려고 하니까 에러가 발생할 수 밖에....</strong></p>
<p>이걸 동료와 같이  트러블슈팅을 하다가 그의 말에 착안하여 그제서야 주석처리하여 고쳤고, 서버가 깔끔하게 돌아갔다는 후일담을 남긴다.</p>