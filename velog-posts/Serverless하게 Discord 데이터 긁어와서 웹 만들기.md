<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/82e84b66-f090-4e84-abf3-d0ce0ec59fa0/image.png" /></p>
<p>오늘은 velog를 본따서 내가 소속되있는 단체인 &quot;블로그 안 쓰면 죽는 모임&quot;의 웹을 하나 만들어보고자 한다.</p>
<h3 id="왜-만드는가">왜 만드는가</h3>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/b25cb8e5-bd21-4b07-bd0e-bf05319f7985/image.png" />본인은 썸네일 만들기를 정말 좋아한다.
그러나 디스코드에는 썸네일이 너무 작게 보인다는 불만사항이 하나 있었다.</p>
<p>뭔가 만들어놓으면 조회수, 좋아요 관리 같은 기능들을 구현하여
앞으로 모임에서 쓸 일이 꽤 있을 것만 같았다.</p>
<p>다만, 단순히 기능의 구현을 하는 것은 늘 하던 노가다니까, 뭔가 색다른 시도들을 하고 싶었다.</p>
<h3 id="색다른-시도">색다른 시도</h3>
<ol>
<li>대부분 바이브 코딩으로 빠르게 구현해보기</li>
<li>Serverless하게 Discord 데이터 관리</li>
</ol>
<h3 id="왜-serverless하게">왜 Serverless하게?</h3>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/ce1d8628-b9f7-4f48-8171-7460dc159890/image.png" /></p>
<p>아무래도 가장 큰 것은 비용 문제다.
요즘 <strong>달러가 너무 비싼지라</strong> 달러 지출이 많은 개발자 특성상 부담이 되기 시작했다.</p>
<p>그러던 중, 뭔가 서버 없이 될 거 같다는 생각이 들었다. 그 방법은 다름이 아닌 
<img alt="" src="https://velog.velcdn.com/images/ysj7191/post/dc27acc0-52cd-4faa-bc50-b7a596cd24c4/image.png" /></p>
<p>우리에게 너무나도 친숙한 Github를 이용하는 것이다.
사실 본인이 최근 알아낸 사실이 있는데 Github는 크론잡을 구동할 수 있다.
즉, 타이머를 맞춰두고 언제 어떤 작업을 맡길 수 있는 것이다.</p>
<p>그리고 Github / Github  Actions는 아주 유명한 CI 툴이다.
DB를 구동하는 것은 아니라도, 위에서 돌린 크론잡의 결과를 커밋으로 저장하고, 이걸 배포하는 식으로 관리한다면 정말 팀원 10여명이서 갖고 놀기는 충분한 결과물이 <strong>관리비 없이</strong> 탄생하는 것이다.</p>
<h3 id="구현-아키텍처">구현 아키텍처</h3>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/f7fc6abc-08a2-4ea8-8edd-dcd060805d50/image.png" /></p>
<h3 id="동작-모습">동작 모습</h3>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/88737545-c803-487d-8365-2d83ae74fe5e/image.png" />이번 봇은 고수의 상징인 <strong>킹니갓사</strong>를 이용해서 우리 모임의 강력함을 과시해보았다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/16089fd3-31c3-44d2-84b9-a268ffd9a380/image.png" />매일 오전/오후 1시에 작동하는 
여담이지만 매번 배포가 이루어지는건 의미가 없으니까, 배포시에 변경 사항이 있는지 체크를 진행한다.
<code>git diff</code> 명령어를 활용한다.</p>
<pre><code class="language-shell">    # 변경사항이 있을 때만 커밋
    if git diff --staged --quiet; then
          echo &quot;No changes to commit&quot;
    else
          git commit -m &quot;chore: update forum-posts.json [skip ci]&quot;
          git push
          echo &quot;✅ Changes committed and pushed&quot;
    fi</code></pre>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/65af84f3-fc8b-448b-992a-7480007855af/image.png" />
사실 이건 좀 충격먹었는데, css에 전혀 관여를 하지 않고, 
Claude에게 velog 던져주고 이 디자인을 차용하라고 명령하니 그대로 추출해버리는 모습이다.</p>
<p>심지어 시키지도 않은 야간모드까지 구현해버린 건 충격이었다.
<img alt="" src="https://velog.velcdn.com/images/ysj7191/post/0fa753bb-e317-4114-ab5f-a69c1cb0ef22/image.png" />심지어 작동까지 한다 ㄷㄷㄷ</p>
<h3 id="앞으로-추가할-내용">앞으로 추가할 내용</h3>
<p>막상 그런데 서버리스로 구현하니까 환경세팅은 오래 걸리지 않았던 대신에, 자유롭지 못한 점들이 있었다.
좋아요 같은 기능을 구현할 수도 없었고, 기능의 추가를 진행할 때는 대부분 클라이언트 입장에서 진행할 수 밖에 없었다.</p>
<p>따라서 앞으로 다음과 같은 기능의 추가를 계획하고 있다.</p>
<blockquote>
<ol>
<li>트렌딩(인기순) / 최신 / 오래된 순 정렬 구현</li>
<li>검색 구현 (가능하면 검색엔진 느낌으로)</li>
<li>이미지 로딩, 번들 사이즈 최적화</li>
<li>버그 수정</li>
</ol>
</blockquote>
<h3 id="참조">참조</h3>
<p><a href="https://yangjjune.github.io/Blog-Or-Death/">https://yangjjune.github.io/Blog-Or-Death/</a> 블죽모 사이트 링크</p>
<p><a href="https://blog.dishost.kr/15">https://blog.dishost.kr/15</a> 디스코드 봇 생성 및 초대링크 만들기
<a href="https://github.com/YangJJune/Blog-Or-Death">https://github.com/YangJJune/Blog-Or-Death</a> 깃헙 링크</p>