<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/576f8009-6f40-4d3d-be7c-3e6ad6ae8705/image.png" /></p>
<h2 id="서론">서론</h2>
<p>Mixed-Content 문제가 발생하여 서버를 HTTPS로 배포할 필요성이 있었다. 
그러나 ISP가 80번 포트를 막아두었는지 HTTP 접근을 허용할 수 없는 환경을 마주하게 되었다. 마찬가지로 443, 8080 역시 막힌 것을 확인하였다. </p>
<p>이때 80번 포트 확인(HTTP-01) 대신 <strong>DNS-01 Challenge</strong> 방식을 사용하면 문제없이 SSL 인증서를 발급받을 수 있다.</p>
<h2 id="1-dns-01-challenge란">1. DNS-01 Challenge란?</h2>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/bea322a6-9875-42ae-83c9-df2f0a46d393/image.png" />
DNS-01 Challenge는 2016년 1월에 Let's Encrypt가 소개한 기능으로 웹 서버를 통해 파일을 확인하는 대신, 도메인의 DNS 설정에 특정 <strong>TXT 레코드</strong>를 추가하여 소유권을 증명하는 방식이다.</p>
<p>마치 회원가입 시에 전화번호를 통해 인증번호를 받거나, 통장으로 1원 송금을 통해 인증하는 방식과 비슷하다.</p>
<h2 id="2-dns-01-challenge의-장점">2. DNS-01 Challenge의 장점</h2>
<ul>
<li><strong>포트 개방 불필요:</strong> 80/443 포트가 막혀 있어도 인증 가능하다.</li>
<li><strong>와일드카드 인증서 발급:</strong> <code>*.example.com</code> 형태의 인증서는 오직 이 방식으로만 발급됩니다.</li>
<li><strong>폐쇄망 서버 적용:</strong> 외부에서 접속할 수 없는 내부 서버에도 공인 SSL 적용이 가능하다.</li>
</ul>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/bd963e9c-0522-4aaf-be32-8877d5fe6ed9/image.png" />
다만 DNS 설정을 추가적으로 해야한다는 단점이 있긴하다.</p>
<h2 id="3-적용하는-법">3. 적용하는 법</h2>
<p>특정 DNS 플러그인을 사용하지 않고 가장 범용적으로 사용할 수 있는 수동 인증 방법을 소개하고자 한다.</p>
<pre><code>sudo certbot certonly --manual --preferred-challenges dns -d yourdomain.com</code></pre><p>위의 명령어를 입력하면 약관 동의 후 중간에 _acme-challenge ~~ 로 TXT를 등록하라는 메시지가 뜬다.</p>
<p>이를 가비아 같은 도메인 관리 사이트에서 TXT 레코드를 등록한 뒤에 잠시 기다렸다가 엔터를 누르면 인증이 완료된다.</p>
<h3 id="왜-기다려야-하는가">왜 기다려야 하는가?</h3>
<p><code>DNS Propagation</code> 때문이다.
<img alt="" src="https://velog.velcdn.com/images/ysj7191/post/2d4d125a-2c4d-4652-a735-3e57009698d7/image.png" /></p>
<p>DNS Propagation이란, DNS 서버가 인터넷을 통해 DNS 레코드에 변경 내용을 전파하는 데 걸리는 시간을 의미한다.</p>
<p>즉, 우리가 저장을 누르고 적용을 누른다고 바로 A-&gt;B로 바뀌는 것이 아니다.</p>
<p><strong>&quot;얘들아 쟤 B로 바뀌었데!&quot;</strong> 라면서 점점 전파된다는 것이므로 바로 인증 확인을 받으려다가 DNS Propagation이 아직 덜 이루어져서 인증에 실패할 수도 있다는 것을 의미한다.</p>
<p>아래의 사이트를 이용할 수 있다.
<a href="https://www.whatsmydns.net/">https://www.whatsmydns.net/</a></p>
<h3 id="nginx-설정">Nginx 설정</h3>
<p>원래라면 먼저 80포트를 열고 Nginx로 프록시하는 것이 일반적이겠지만, 우리는 다른 방식으로 인증을 받았기 때문에 결과적으로 생성된 인증키를 이제서야 사용하면 https를 이용할 수 있다.</p>
<p>/etc/nginx/sites-available/default 에 해당 파일을 적용하였다.
(그 외에도 설정에 따라 다른 파일을 생성한 뒤에 작업해도 된다!)</p>
<pre><code class="language-shell">server {
    listen 443 ssl;
    server_name yourdomain.com;

    # SSL 인증서 경로
    ssl_certificate /etc/letsencrypt/live/[yourdomain.com/fullchain.pem](https://yourdomain.com/fullchain.pem);
    ssl_certificate_key /etc/letsencrypt/live/[yourdomain.com/privkey.pem](https://yourdomain.com/privkey.pem);

    # 보안 권장 설정 (선택 사항)
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;

    location / {
        # 프론트 또는 백엔드 연결 (예: ASP.NET, Ruby on Rails 등)
        proxy_pass [http://127.0.0.1:5000](http://127.0.0.1:5000);
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

# (선택) 80포트가 열려있을 경우 HTTPS로 리다이렉트
server {
    listen 80;
    server_name yourdomain.com;
    return 301 https://$host$request_uri;
}</code></pre>
<p>NGINX config를 수정하였기 때문에 restart 또는 reload 해준다.</p>
<pre><code class="language-bash"># Nginx 설정 문법 검사
sudo nginx -t

# Nginx 재시작 (reload해도 무방하다)
sudo systemctl restart nginx</code></pre>
<h2 id="4-최종-확인">4. 최종 확인</h2>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/e8aa7b23-df58-42fd-bafa-f86123b81289/image.png" /></p>
<p>최종적으로 완료되었음을 확인하였다.</p>