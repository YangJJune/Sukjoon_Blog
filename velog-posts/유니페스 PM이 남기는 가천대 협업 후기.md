<h2 id="서론">서론</h2>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/f9991cab-7835-4541-8cd9-6b4b79e1f5c6/image.png" />
2025년 9월 11일 부로 &quot;가천대 가을축제 무한전야&quot;를 위한 서비스가 성황리에 종료되었다.</p>
<p>이번엔 요구사항을 적극적으로 받고 완전 가천대에 맞춤형으로 서비스를 제공하였기에 개발자와 고객 사이에서 뛰어다닐 일이 많아서 쉽지 않았다. 그러나 총학생회 측의 배려와 적극적인 협조로 양측에서 원하는 결말로 무사히 마무리된거 같아 정말 다행이라고 생각한다.</p>
<p>오늘은 현장에서 직접 실서비스를 하지 않으면 겪을 수 없는 이야기를 담으며 후기를 작성하고자 한다. </p>
<h2 id="문제는-첫-날에-대부분-발생한다">문제는 첫 날에 대부분 발생한다</h2>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/5d57548f-27ae-460f-8a00-e6c85bc7173c/image.png" /></p>
<p>본인은 유니페스의 서비스를 진행할 때마다 최소한 첫 날은 현장에 가서 비상상황에 대비하여 대기한다. </p>
<p>서버에 문제가 있다면 또는 어플 사용에 어려움이 있다면 대부분 첫날일 것이라는 계산이 있었기 때문이다.
그리고 <strong>그동안은</strong> 정말 다행히도 대부분은 큰 문제 없이, QA에서 벗어난 사소한 오류들이 잡히고는 했다.
그동안은 말이지...</p>
<p>첫 문제는 주점 영업 직전, 오후 5시 경에 발생한다
&nbsp;</p>
<h2 id="양석준의-오관참장">양석준의 오관참장</h2>
<blockquote>
<p>오관참장(五關斬將) : 대충 삼국지에서 관우가 형님 만나겠다고 5개 관문을 다 때려부수고 간 썰</p>
</blockquote>
<h3 id="1-웨이팅하는데-일부-손님이-호출이-안-돼요">#1 웨이팅하는데 일부 손님이 호출이 안 돼요!</h3>
<p>갑자기 주점으로부터 웨이팅 인원을 호출하면 간헐적으로 오류가 발생한다고 민원이 들어왔다.
일단은 사전에 제시한 메뉴얼대로 전화번호를 통해 수동으로 손님들을 호출해달라고 했다.</p>
<p>제대로 웨이팅의 운영이 시작되는 오후 6시 전까지 이 문제를 해결해야했다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/d7f37857-634f-4e33-9c40-16407d733a60/image.png" />
안타깝게도 지금 백엔드 팀원들이 대부분 현직자라 퇴근 전까지 당장 처리가 불가능한 상황이었다.
따라서, 현장에서 내가 직접 보는 수 밖에 없었다.</p>
<p>Sentry 로그와 직접 백엔드 코드를 확인한 결과 Redis에서 유저의 FCMToken을 불러오는 구조인데 불러오지 못해서 null 값이 반환되고 있어 Exception을 던지고 있음을 확인하였다. 그리고 FCMToken은 Waiting 테이블에도 저장하고 있음을 확인하였다.</p>
<p>당장 중요한 건 웨이팅 호출이 간헐적으로 해당 사유로 안된다는 것이었기 때문에 Redis에 토큰이 없는 경우 WaitingRepository에서 가져오고, 그럼에도 없으면 Exception을 throw 하는 것으로 핫픽스를 진행했다.</p>
<p>핫픽스의 배포 과정에는 다행히 연락이 되는 팀원이 있어서 간단하게 확인하고 배포를 진행했다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/40f510b6-a194-48d5-b99a-28397e785dd1/image.png" /></p>
<p>다행히 핫픽스가 유효하여 문제 없이 웨이팅 로직이 굴러갔고, 모든 부스를 돌아다니며 웨이팅이 잘 이루어지고 있는지 확인한 후에야 안심할 수 있었다.</p>
<p>여담이지만 이번 포스트에서는 각각의 문제를 간단히만 서술하고, 버그 하나 당 포스트를 하나 작성하여 제대로 파고 어떻게 해결하고 방지하는 것이 맞을 지 공부하고 구상할 예정이다.</p>
<h3 id="2-운영자-웹이-너무-느려요">#2 운영자 웹이 너무 느려요</h3>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/2b198eea-3911-4589-87c3-752bd2ea5383/image.png" /></p>
<p>오후 7시 경, 본격적인 트래픽이 몰리기 시작한지 1시간 째였다. 갑자기 웹이 너무 느리다는 연락을 다수 받았다.
*<em>'이건 진짜 서버에 무슨 일이 생긴건가?' *</em>라는 불안한 생각과 함께 빠르게 서버를 확인하고 Sentry에 Exception이 다수 잡히고 있는 지 등을 확인하였다. </p>
<p>아무리 찾아봐도 서버에는 트래픽은 몰리고 있었으나 리소스적으로 큰 문제가 없었고, 이를 전달하고자 부스 관계자분들과 소통을 하는데 놀랍게도 전화가 되지 않는다는 것을 알았다.</p>
<p>수 년전에 SETEC 전시장에서 열리는 행사에 방문할 때 많은 인파가 몰리면 데이터도 전화도 잘 터지지 않는 다는 것을 경험한 바가 있었다. 중계기 대역폭이 터진 듯하다. </p>
<p>솔직히 이거만큼은 멘붕이 왔다. 그동안 한번도 서비스를 운영하며 겪지 못한 문제이며, 내가 직접적으로 해결할 수 있는 일은 없다는 걸 깨달았기 때문이었다.</p>
<p>어쩔 수 없이 총학과 부스 운영자 측에 네트워크/전파 장애가 발생했음을 안내드리고, 그나마 부스가 몰려있는 공간보다는 길가로 나오면 통신이 원활한 편이라는 것을 안내하여 다소 불편하더라도 어떻게든 1일차 서비스를 진행했다. 다행히 장애는 1시간 가량 지속된 후, 연예인 공연이 시작되니 휴대폰 통신량이 줄어서 그런지 다소 해소되었다.</p>
<p>1일차 주점이 종료되고 총학생회 측과 이동식 기지국의 설치를 이야기해보았으나, 아무래도 당장 그걸 설치하는 것은 불가할 것이라고 결론이 났다. 부스 운영자들만큼은 네트워크가 터지는 걸 방지하고자 무선 공유기를 4대 정도 설치하고 하나의 허브에 물린 다음, 100m 가량 가설하여 유선으로 빼는 것도 고려해보았으나 이 역시도 힘든 상황이었다. 
그러던 중 GA를 통해 BoothDetailView(부스 정보 페이지)를 가장 많이 호출하는 것을 알게 되었는데, 이미지 경량화를 진행하는 것이 효과적일 것이라고 생각하였다. 다만 avif나 webp같은 확장자가 각 네이티브 플랫폼(Android, iOS)에서 지원이 되는 라이브러리를 쓰는지 확실치 않아, 이미 사용하고 있던 jpeg로 압축하고 해상도를 낮추어 대부분 100kb 이내로 이미지를 구성하였다.</p>
<table>
<thead>
<tr>
<th></th>
<th></th>
</tr>
</thead>
<tbody><tr>
<td><img alt="이미지 1" src="https://velog.velcdn.com/images/ysj7191/post/960ddbe0-4def-4177-a044-e8e99d4f2369/image.png" width="400" /></td>
<td><img alt="이미지 2" src="https://velog.velcdn.com/images/ysj7191/post/b6edd1a7-14c0-42a3-9cce-d36669b99a60/image.png" width="480" /></td>
</tr>
</tbody></table>
<p>결과적으로 꽤 유의미한 결과가 나왔다. 
특히 2일차에는 여전히 장애가 발생하기는 했으나, 장애의 정도가 덜한 것을 확인하였기 때문이다. </p>
<p>지표로 보아도 이를 확인할 수 있는데, 트래픽이 더 몰렸으나 다운로드된 데이터 바이트 크기가 1/2 정도로 감소하였으니 그만큼의 대역폭의 여유가 생겼을 것이라고 보고 있다.</p>
<p>이 정도의 트래픽을 안 받아보니 이미지의 경량화가 이렇게 중요한지 이제서야 꺠달은 자신을 반성하는 바이다.</p>
<h3 id="3-무한-리프레시-토큰-재발급-현상">#3 무한 리프레시 토큰 재발급 현상</h3>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/8e99fc6a-59ec-498c-acb1-079a17356088/image.png" />
갑자기 10일 점심 즈음 (축제를 시작하기 전이다), 리프레시 토큰 관련 에러가 폭증했다. 이는 하나의 웹으로부터 오고 있음을 확인할 수 있었는데, 사실 이건 아직도 명확한 원인을 찾지 못했다.</p>
<p>해당 로직을 짠게 본인이 아니라 그런 것도 있지만 문제의 재현이 어려워서 어디서 이런 일이 생기는지는 명확히 알아낼 수 없었다. 그저 운영자 웹에서 문제가 있었을 거라 추정하는게 전부였다.</p>
<p>프론트의 문제이긴 하지만, 어쨌든 서비스 이용에 방해가 되지 않도록 문제를 해결해야했다.</p>
<p>코드를 뜯어보니 RefreshToken은 레디스 상에서 관리되고 있었고, 이를 계정을 특정할 수 있는 email값과 매칭하는 식으로 구성되어있었다. </p>
<p>redis-cli를 통해 확인해보니 특정 리프레시 토큰에 대한 조회가 지속적으로 이루어지고 있으나 무슨 연유로 삭제되어 nil을 반환하고 있기에 AccessToken을 발급하지 못하고 문제가 무한하게 반복되고 있음을 알 수 있었다.</p>
<p>따라서 일단 redis에 해당 RefreshToken과 일치하는 더미 계정을 하나 SET 해놓아 accessToken을 강제로 재발급 시켜서 해결하였다.</p>
<p><del>근데 여담이지만 저걸 호출하고 있는 게 내 노트북이라는 걸 알게된 건 30분 뒤의 일이었다..</del></p>
<h3 id="4-timeout-문제">#4 timeout 문제</h3>
<p>이건 2번과 연계되는 문제이다.
그리고 이걸 조금 더 빨리 알았으면 좋았을 걸이라는 후회가 지금도 든다. 왜냐하면 2일차 밤에 해당 문제가 있음을 깨달아서 3일차에 해결된 버전의 운영자 웹을 배포하였기 때문이다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/932a5c41-fd69-431b-aedf-7db0d02f16df/image.png" /></p>
<p>timeout이라는 것은 기본적으로 서버가 죽었음에도 계속 대기하는 현상을 방지하고자, 리소스를 절약하고자 사용한다.
아주 일반적인 상황에서는 10초는 충분한 시간이었으나, 2번 문제에서 이야기했듯 네트워크 상황이 정상이 아니었다.</p>
<p>(사실 초당 200Kbps는 우리 서비스를 이용하기에 충분한 속도인데, 다소 부정확하다고 평을 받는 fast.com에서 저 정도가 나온다는 건 심각한 상황이 맞다고 생각한다)</p>
<p>그래서 어떤 일이 생겼냐면, 운영자웹은 빠른 인터랙션이 중요한 건 아니라서 결국 그 결과만 얻을 수 있으면 된다. (&quot;A 손님을 호출한다&quot;, &quot;A 손님을 입장처리한다&quot;) 그냥 가만히 30초가 되었던 40초가 되었던 호출을 보내면 문제가 없던 상황인데 저 10초짜리 타임아웃이 발목을 잡고 있었던 것이다.</p>
<p>이걸 혹시 빨리 제보해주셨다면 바로 고칠 수 있었겠지만, 늦게서야 그 문제를 떠올려 직접 적용하는 바람에 하루가 소모된 점이 지금 생각해도 너무 아쉽다.</p>
<h3 id="5-s3야-어째서">#5 S3야 어째서...?</h3>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/c6fe7d19-6f54-4430-832f-b7c6bdfc6453/image.png" />
사실 계속 알아봐야되긴 하는데 도대체 어디서 780GB나 쓴 건지 아직도 모르겠다.
어제 파악한 문제라 계속 확인을 해야하고 최악의 경우 AWS에 빌넣을 해야하는 상황이다.</p>
<p>CloudFront 쓸 걸... 이라고 후회가 들 뿐이다. 여러분은 S3의 대역폭 비용이 생각보다 비싸니 감안해서 사용하기 바란다...</p>
<h2 id="실적">실적</h2>
<p>어쨌든 우여곡절은 있었으나 큰 문제는 없었고, 축제는 성황리에 마무리되었다. 
우리 팀도 많은 성과를 남길 수 있었다.</p>
<table>
<thead>
<tr>
<th>MAU 5400 달성, 3일간 DAU 합계 7000+ 달성</th>
<th>1일차 최대 동접을 기록한 순간</th>
</tr>
</thead>
<tbody><tr>
<td><img alt="이미지 1" src="https://velog.velcdn.com/images/ysj7191/post/9ec20b6f-8ace-48b1-a80b-f14478423f7a/image.png" width="480" /></td>
<td><img alt="이미지 2" src="https://velog.velcdn.com/images/ysj7191/post/4998af06-34ec-4e75-8009-844fa05faa01/image.png" width="480" /></td>
</tr>
</tbody></table>
<p>&nbsp;</p>
<table>
<thead>
<tr>
<th></th>
<th></th>
<th></th>
</tr>
</thead>
<tbody><tr>
<td><img alt="이미지 1" src="https://velog.velcdn.com/images/ysj7191/post/17b4d29b-e8ba-4242-b55a-0878462db836/image.png" /></td>
<td><img alt="이미지 2" src="https://velog.velcdn.com/images/ysj7191/post/f439dbd1-d5de-43d8-be8e-843144bd8a85/image.png" /></td>
<td><img alt="이미지 2" src="https://velog.velcdn.com/images/ysj7191/post/4bec3782-dc81-46bb-9270-63c94cdc20bc/image.png" /></td>
</tr>
</tbody></table>
<p>&nbsp;</p>
<table>
<thead>
<tr>
<th>앱스토어 라이프스타일 19위</th>
<th>플레이스토어 72위</th>
</tr>
</thead>
<tbody><tr>
<td><img alt="이미지 1" src="https://velog.velcdn.com/images/ysj7191/post/18f92a8c-1019-46e5-88bc-78a3e8d5a902/image.png" /></td>
<td><img alt="이미지 2" src="https://velog.velcdn.com/images/ysj7191/post/5fea1c69-ade3-4dca-aca2-3bffdcaedc05/image.png" /></td>
</tr>
<tr>
<td>&nbsp;</td>
<td></td>
</tr>
</tbody></table>
<h2 id="후기">후기</h2>
<h3 id="202402-유니페스-개발팀-결성">2024.02 유니페스 개발팀 결성</h3>
<p><img alt="" src="https://api.velog.io/rss/@ysj7191" /></p>
<div style="display: flex;">
    <img alt="이미지 1" src="https://velog.velcdn.com/images/ysj7191/post/08c07543-8c30-461a-b0b1-cffa84e26542/image.png" style="width: 30%;" />
    <img alt="이미지 2" src="https://velog.velcdn.com/images/ysj7191/post/b9bdec0b-37f9-490c-a327-45ac773e5e3e/image.png" style="width: 70%;" />
</div>

<hr />
<h3 id="202405-건국대-시범운영">2024.05 건국대 시범운영</h3>
<div style="display: flex;">
    <img alt="이미지 1" src="https://velog.velcdn.com/images/ysj7191/post/ab3bdd76-1520-4d2b-9253-a6ad86a82dbe/image.png" style="width: 50%;" />
    <img alt="이미지 2" src="https://velog.velcdn.com/images/ysj7191/post/3c87e698-adfb-4cf3-8f8b-8905c8eec28a/image.png" style="width: 50%;" />
</div>

<hr />
<h3 id="202410-최초의-공식-협업-한국교통대-대동제-youngone">2024.10 최초의 공식 협업, 한국교통대 대동제 Young:One</h3>
<div style="display: flex;">
    <img alt="이미지 1" src="https://velog.velcdn.com/images/ysj7191/post/a64278f9-7f00-4521-ad3c-12692c46f086/image.png" style="width: 50%;" />
    <img alt="이미지 2" src="https://velog.velcdn.com/images/ysj7191/post/b73091c4-4784-42d9-abf9-dcdfcf015ea4/image.png" style="width: 50%;" />
</div>

<hr />
<h3 id="202505-sky-학교와-협업-고려대학교-석탑대동제">2025.05 SKY 학교와 협업, 고려대학교 석탑대동제</h3>
<div style="display: flex;">
    <img alt="이미지 1" src="https://velog.velcdn.com/images/ysj7191/post/96cbcc22-cf37-4b24-a000-23d96346b698/image.png" style="width: 50%;" />
    <img alt="이미지 2" src="https://velog.velcdn.com/images/ysj7191/post/8b65ea3b-f7d9-4e15-9b7a-ed644dcbb69c/image.png" style="width: 50%;" />
</div>

<hr />
<h3 id="202509-최대의-실적-기록-가천대학교-무한전야">2025.09 최대의 실적 기록, 가천대학교 무한전야</h3>
<div style="display: flex;">
    <img alt="이미지 1" src="https://velog.velcdn.com/images/ysj7191/post/e6494323-12b7-460a-9fd5-45a588718160/image.png" style="width: 50%;" />
    <img alt="이미지 2" src="https://velog.velcdn.com/images/ysj7191/post/dc9f874b-78d1-4fe6-8f83-a998ce76b542/image.png" style="width: 50%;" />
</div>

<p>한번 정리해봤는데 정말 오랜 길을 달려왔다는 것을 느낀다. 
특히 에브리타임 공개 모집 시절을 생각하면 정말 아찔해진다.
자화자찬이지만 여기까지 도전해준 나에게 고맙고, 이런 저를 따라와주시고 도와주신 모든 팀원분들께 감사하다는 생각이 가장 크게 드는 거 같다.</p>