<h2 id="서론">서론</h2>
<p>이번에 유니페스 프로덕션 서버와 개발 서버를 통합하는 과정에서 기존 가비아 도메인을 버리고 개발서버 도메인으로 통합하는 과정에서 https를 붙여야하는 상황이 되었다. </p>
<p>그러나 단순히 HTTPS를 도메인에 붙이는 법이라고 검색하고 AI에게 부탁하기 보다는 그 필요성과 적용과정 암호화 방식 등을 이해하는 것이 의미 있을 거 같아 이번 포스트를 작성해보았다.</p>
<h2 id="http란-무엇인가">HTTP란 무엇인가?</h2>
<p>HTTP(HyperText Transfer Protocol)는 웹(WWW)상에서 정보를 주고받기 위한 가장 기본적인 프로토콜이다. 클라이언트(주로 웹 브라우저)가 서버에 특정 데이터를 요청(Request)하고, 서버는 해당 요청에 맞는 데이터를 응답(Response)하는 방식으로 동작한다.</p>
<p>이는 텍스트, 이미지, 영상 등 하이퍼미디어 문서를 전송하기 위해 고안되었으며, TCP/IP 위에서 작동한다. HTTP의 가장 큰 특징은 <strong>평문(Plain Text)</strong>으로 데이터를 전송한다는 점이다. 즉, 중간에 누군가 네트워크 통신을 가로챈다면(Sniffing) 전송되는 아이디, 비밀번호, 혹은 개인 정보 등을 그대로 확인할 수 있는 보안상의 취약점을 가진다.</p>
<pre><code># HTTP 요청 데이터를 뜯었을 때의 예시

HTTP/1.1 200 OK
Date: Wed, 30 Jan 2019 12:14:39 GMT
Server: Apache
Last-Modified: Mon, 28 Jan 2019 11:17:01 GMT
Accept-Ranges: bytes
Content-Length: 12
Vary: Accept-Encoding
Content-Type: text/plain

{
    &quot;name&quot; : &quot;홍길동&quot;,
    &quot;tel&quot; : 01012345678
}</code></pre><h2 id="https란-무엇인가">HTTPS란 무엇인가?</h2>
<p>HTTPS(HyperText Transfer Protocol Secure)는 HTTP의 보안이 강화된 버전이다. 'Secure'라는 이름에서 알 수 있듯이, 이는 데이터 전송의 보안성을 확보하기 위해 탄생했다.</p>
<p>HTTPS는 HTTP 통신 소켓을 TCP에서** SSL(Secure Socket Layer)** 또는 <strong>TLS(Transport Layer Security)</strong>라는 별도의 보안 계층으로 대체한다. 이를 통해 클라이언트와 서버 간의 모든 통신 내용은 암호화되어 전송된다.</p>
<pre><code># &quot;HTTPS&quot; 요청 데이터를 뜯었을 때의 예시

GET /hello.txt HTTP/1.1
User-Agent: curl/7.63.0 libcurl/7.63.0 OpenSSL/1.1.l zlib/1.2.11
Host: www.example.com
Accept-Language: en

t8Fw6T8UV81pQfyhDkhebbz7+oiwldr1j2gHBB3L3RFxP3HGwYCSIvyzS3MpmmSe4iaWKCOHQ==</code></pre><h2 id="ssltls란-무엇인가">SSL/TLS란 무엇인가?</h2>
<p><code>SSL(Secure Sockets Layer)</code>과 <code>TLS(Transport Layer Security)</code>는 네트워크를 통해 작동하는 두 컴퓨터 간의 통신을 암호화하기 위해 설계된 암호화 프로토콜이다. 사실상 SSL은 초창기 버전이며, 현재는 그 후속 버전이자 더욱 강력한 보안을 제공하는 TLS가 표준으로 사용된다. 하지만 관용적으로 두 용어를 혼용하거나 'SSL/TLS'로 통칭하기도 한다.</p>
<p>따라서 TLS는 공개 키 암호화라는 기술을 사용합니다.
<img alt="" src="https://velog.velcdn.com/images/ysj7191/post/a2459183-1342-4738-81ca-114434273dab/image.png" /></p>
<h3 id="공개-키-암호화">공개 키 암호화</h3>
<p>한 쌍의 '공개 키'와 '개인 키'로 작동한다. </p>
<ol>
<li><code>서버</code>는 자신의 공개 키를 'SSL/TLS 인증서'를 통해 <code>클라이언트</code>에게 공유한다.</li>
<li><code>클라이언트</code>가 <code>서버</code>와 보안 연결을 시작하면, <code>클라이언트</code>는 <code>서버</code>의 공개 키를 이용해 실제 통신에 사용할 '대칭 키(세션 키)' 를 암호화하여 <code>서버</code>에 전달한다. </li>
<li><code>서버</code>는 오직 자신만이 가진 '개인 키'를 사용해야만 이 암호화된 메시지를 해독하여 세션 키를 얻을 수 있다.</li>
<li>이렇게 양측이 안전하게 동일한 세션 키를 공유한 이후의 모든 통신은, 더 빠르고 효율적인 이 대칭 키(세션 키)를 사용하여 암호화된다.
&nbsp;</li>
</ol>
<h3 id="https-프로토콜의-핵심-역할">HTTPS 프로토콜의 핵심 역할</h3>
<blockquote>
<p><strong>암호화(Encryption)</strong> | 
제3자가 데이터를 가로채더라도 내용을 해독할 수 없도록 데이터를 암호화한다.</p>
</blockquote>
<blockquote>
<p><strong>인증(Authentication)</strong> | 
클라이언트가 접속하려는 서버가 정말 신뢰할 수 있는 서버가 맞는지 'SSL/TLS 인증서'를 통해 검증한다.</p>
</blockquote>
<blockquote>
<p><strong>무결성(Integrity)</strong> | 
데이터가 전송 도중에 위변조되지 않았음을 보장한다.</p>
</blockquote>
<h2 id="https의-장점-및-http와의-차이">HTTPS의 장점 및 HTTP와의 차이</h2>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/d1253a04-318b-45ac-ac25-cd86a833d221/image.png" /></p>
<p>HTTPS와 HTTP의 가장 근본적인 차이는 '보안성' 에 있다. HTTP가 데이터를 그대로 전송하는 반면, HTTPS는 SSL/TLS 계층을 통해 데이터를 암호화하여 전송한다.</p>
<p><strong>HTTPS의 주요 장점은 다음과 같다.</strong></p>
<blockquote>
<p><strong>데이터 보안</strong> | 
위에서 언급했듯이, 모든 통신 내용이 암호화된다. 만약 사용자의 로그인 정보나 결제 정보가 HTTP를 통해 전송된다면 이는 심각한 보안 사고로 이어질 수 있으나, HTTPS는 이를 방지한다.</p>
</blockquote>
<blockquote>
<p><strong>서버 신뢰도 확보</strong> | 
사용자는 브라우저의 자물쇠 아이콘을 통해 자신이 접속한 사이트가 SSL/TLS 인증서로 검증된, 신뢰할 수 있는 사이트임을 확인할 수 있다. 이는 피싱 사이트 등을 판별하는 데 도움을 준다.</p>
</blockquote>
<blockquote>
<p><strong>검색 엔진 최적화(SEO)</strong> | 
Google과 같은 주요 검색 엔진은 보안을 중시하여, HTTPS를 사용하는 사이트에 대해 검색 순위(SEO) 가산점을 부여한다.</p>
</blockquote>
<h2 id="여담---cors와-https">여담 - CORS와 https?</h2>
<p>종종 CORS(Cross-Origin Resource Sharing) 문제와 HTTPS를 연관 지어 생각하는 경우가 있다. 
본인 역시 개발 중에 CORS 문제를 처음으로 해결하던 시절 https에 대해 공부하던 시절이 있었던 거 같다.</p>
<p>CORS는 한 출처(Origin)에서 실행 중인 웹 애플리케이션이 다른 출처의 자원에 접근할 수 있도록 허용하는 <strong>보안 정책(Policy)</strong>이며, HTTPS는 통신 자체를 암호화하는 <strong>보안 프로토콜(Protocol)</strong>이다.</p>
<p>하지만 이 둘은 개발 환경에서 밀접하게 만나는 지점이 있다.</p>
<p>가장 흔히 마주치는 문제로, '혼합 콘텐츠(Mixed Content)' 오류가 있다. 
본인도 만약 사이트가 HTTPS 프로토콜을 통해 안전하게 로드되었는데, 이 페이지 내부에서 호출하는 API, 스크립트(JavaScript), CSS, 이미지 리소스 등이 HTTP 프로토콜을 사용한다면, 브라우저는 이를 보안 위협으로 간주한다.</p>
<p>또한 Origin에 있어서도 프로토콜이 서로 다르기 때문에 (ex. <a href="http://google.com">http://google.com</a> | <a href="https://google.com">https://google.com</a>) 서로 다른 origin으로 구분하여 CORS 문제가 생길 수 있다는 점을 곁들여서 적어둔다.</p>