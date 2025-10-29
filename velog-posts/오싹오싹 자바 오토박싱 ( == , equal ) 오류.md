<h3 id="서론">서론</h3>
<p>이런 말이 있다.</p>
<blockquote>
<p>“왜 되는지는 모르겠지만 돌아가니까 됐잖아?”</p>
</blockquote>
<p>정확히 저 말은 아니어도 본인이 코딩을 한다면 위와 같은 이야기를 한 번 쯤 해봤을 것이다.</p>
<p>아마 재앙은 그렇게 시작되었을지도 모른다. 이 이야기는 1년 전에 짠 코드가 문제를 일으키는 이야기기 떄문이다.</p>
<h3 id="상황-발생">상황 발생</h3>
<p>고려대에 유니페스 서비스를 진행하고 있을 때였다. 한창 부스 운영자들이 우리 서비스에 회원가입을 하던 시기였는데 갑자기 CS 메일이 빗발쳤다. 부스와 관련한 다수의 기능에서 오류가 발생하고 있었다.</p>
<p>심각한 상황이었기에 본인이 백엔드 개발자는 아니지만 로그를 뜯어보고 그 원인을 확인해보았다.</p>
<p>모든 오류들이 가리키는 건 부스의 주인인가 검증하는 로직이었다.</p>
<pre><code class="language-java">private Booth verifyAuth(Long memberId, Long boothId){
    Booth findBooth = boothRepository.findByBoothId(boothId).orElseThrow(BoothNotFoundException::new);
    if(findBooth.getMember().getId() != memberId) throw new NotAuthorizedException();
    return findBooth;
}</code></pre>
<h3 id="왜-그런가">왜 그런가?</h3>
<p>사실 고수들은 저 코드를 보자마자 그 원인을 짐작하였거나, 문제점을 찾았을 것이다.</p>
<p>if의 조건 부분이다.</p>
<p>기본적으로 == 그리고 != 는 <strong>객체의 주소를 비교한다.</strong>
즉, <strong>두 Long 객체의 주소를 비교한다는 것</strong>이다.</p>
<p>그렇다면 문제는 왜 1년 간 문제 없이 잘 지냈던 것인지가 되겠다. 답은 가입한 유저 수에 있었다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/e17ac0da-19cb-4c3d-a3b1-b03ad3d4b8f6/image.png" /></p>
<p>보다시피 현재 130명 가량의 유저가 가입한 상태였다.</p>
<p>그리고 JAVA는 성능을 위해 <strong>-128 ~ 127을 캐싱</strong>한다. 즉, == 을 사용하고 있었어도, 128번째 유저까지는 실제 value를 비교하고 있었을 것이다.</p>
<p>그 결과 유저가 128명을 넘은 이번 축제에 갑자기 1년 묵은 버그가 터지고 만 것이다. 
<img alt="" src="https://velog.velcdn.com/images/ysj7191/post/1fe3b679-dc0b-4df0-af9e-6de853215a8e/image.png" /></p>
<p>관련한 자세한 포스트는 워낙 많아 당시 참고했던 링크를 첨부하는 바이다.
<a href="https://velog.io/@wujin/Java-Wrapper-%ED%81%B4%EB%9E%98%EC%8A%A4">https://velog.io/@wujin/Java-Wrapper-클래스</a></p>
<h3 id="해결">해결</h3>
<p>해결법은 쉽다. 
equals 메소드를 이용하여 값을 비교하면 끝.</p>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/1fa78877-4c8a-443c-8586-a5002d95c5b1/image.png" /></p>
<p>개념 자체는 기본적이지만 굉장히 특이하고 위험한 경험을 운좋게 하였기에 포스트 하는 바이다.</p>