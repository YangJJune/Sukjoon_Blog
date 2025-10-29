<h2 id="소켓-통신이란">소켓 통신이란?</h2>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/c1b8e673-6580-47ab-8e4a-55bda6b467cb/image.png" />
나는 소켓통신을 아예 처음 보는 사람한테 도로를 빗대어 설명하고는 한다. HTTP 통신은 매번 도로를 깔아야한다. 차라리, 그냥 도로를 깔아놓고 통행하는 것이 나을 것이다. 그렇다, 우린 그걸 소켓 통신이라고 부른다.</p>
<p>장점은 아무래도 언제든 차를 통행시킬 수 있으니 실시간성이 보장되고 빠르다.
단점은 도로 유지비가 들어간다는 것이다.</p>
<p>&nbsp;</p>
<h2 id="소켓-통신의-예시">소켓 통신의 예시</h2>
<ul>
<li>게임 서버</li>
<li>채팅 애플리케이션</li>
<li>FTP</li>
<li>실시간 주식 거래 시스템
etc...</li>
</ul>
<p>&nbsp;</p>
<h2 id="tcpudp">TCP/UDP?</h2>
<p>본격적으로 소켓 프로그래밍을 하기 전, TCP UDP 개념에 대해 알아갈 필요성이 있다.</p>
<h3 id="tcptransmission-control-protocol">TCP(Transmission Control Protocol)</h3>
<p>연결 지향적인 프로토콜로, 데이터를 안정적이고 신뢰성 있게 전송하는 데 사용된다.
<img alt="" src="https://velog.velcdn.com/images/ysj7191/post/3de4c64f-4ece-4807-b7d2-0b1caeda4ae2/image.png" /></p>
<p>특징은 다음과 같다</p>
<ul>
<li>연결 지향적</li>
<li>신뢰성 있는 전송</li>
<li>흐름 제어</li>
<li>혼잡도 제어</li>
<li>손실/오류 발생 시 재전송</li>
</ul>
<h3 id="udpuser-datagram-protocol">UDP(User Datagram Protocol)</h3>
<p>비연결 지향적인 전송 계층 프로토콜로, 데이터를 빠르고 간단하게 전송하는 데 사용된다.
<img alt="" src="https://velog.velcdn.com/images/ysj7191/post/f7959e88-823e-4fb3-82b2-9e8510e0d37d/image.png" /></p>
<ul>
<li>비연결 지향적</li>
<li>신뢰성 없음</li>
<li>데이터 순서의 보장 없음</li>
<li>헤더가 간단함</li>
<li>속도와 효율성</li>
</ul>
<p>처음 보는 사람들은 무슨 말인지 잘 모를 수 있는데, 핵심은 어떠한 블로그에 매우 잘 정리가 되어있었다. 
<a href="https://velog.io/@hidaehyunlee/TCP-%EC%99%80-UDP-%EC%9D%98-%EC%B0%A8%EC%9D%B4">https://velog.io/@hidaehyunlee/TCP-%EC%99%80-UDP-%EC%9D%98-%EC%B0%A8%EC%9D%B4</a></p>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/ce58b927-1661-40c2-a8f6-309dfb79feca/image.png" /></p>
<p>이것이 TCP이고,</p>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/30b60a26-915b-4923-9930-31dd590b695e/image.png" /></p>
<p>이것이 UDP이다.</p>
<p>가장 큰 차이를 확실하게 보여주는 도표라 가져와보았다.
그럼 이제 TCP와 UDP도 알았으니 본격적으로 소켓 통신은 어떻게 이루어지는지, 구현과정까지 살펴보도록 하겠다</p>
<h2 id="소켓통신-과정">소켓통신 과정</h2>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/27157af4-13fa-4c28-ba39-345c8bc7a103/image.png" />
이 도표가 전체적인 그림을 가장 잘 나타내준다.</p>
<ol>
<li>서버와 클라이언트에서 소켓을 생성한다</li>
<li>서버에서 소켓을 특정 주소에 소켓을 bind 시킨다. ※여기서 주소는 IP와 port로 이루어져있다</li>
<li>소켓을 listen 상태로 한다</li>
<li>클라이언트에서 소켓에 connect 요청을 한다</li>
<li>서버가 listen 중이면 accept한다</li>
<li>상호 간에 데이터를 주고 받는다</li>
<li>연결이 종료되면 소켓을 닫는다(close)</li>
</ol>
<h2 id="소켓통신의-구현">소켓통신의 구현</h2>
<pre><code class="language-python">#server.py
import socket

# 소켓 생성
server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# 소켓 주소 세팅
server_address = ('localhost', 9999)
print('Start up on {} port {}'.format(*server_address))

# 소켓 정보 bind
server_socket.bind(server_address)

# 소켓 listen
server_socket.listen()
print('accept wait')

# 소켓 accept 대기
client_socket, client_address = server_socket.accept()
while True:
   try:
      # 클라이언트로부터 data recv
      data = client_socket.recv(1024)
      data = data.decode()
      print(&quot;received : &quot;+data)
      encodedData = bytes(&quot;ACK : &quot;+data,'utf-8')
      # 클라이언트에 data send
      client_socket.sendall(encodedData)

   except Exception as err:
      client_socket.close()
      print(err)
      break

</code></pre>
<pre><code class="language-python">#client.py
import socket

# 소켓 생성
client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# 서버 주소 지정
server_address = ('localhost', 9999)

# 서버 소켓에 접속
client_socket.connect(server_address)

while True:
   try:
      # 서버로 전송할 데이터 입력 받기
      data = input('')
      if data == 'quit':
         break
      # 데이터 인코딩
      data = data.encode()
      # 서버로 전송
      client_socket.send(data)

      # 서버로부터 데이터 받기
      encodedData = client_socket.recv(1024)
      message = encodedData.decode('utf-8')
      print(&quot;SERVER&gt; &quot;+message)

   except Exception as err: # 예외 발생 시
      client_socket.close()
      print(err)
      break

client_socket.close()</code></pre>
<h3 id="결과화면">결과화면</h3>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/db796ed3-6865-416a-8f55-15b1555985de/image.png" />
좌측이 server.py이고 우측이 client.py이다.</p>