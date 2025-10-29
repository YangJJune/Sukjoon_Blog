<h2 id="서론">서론<img alt="" src="https://velog.velcdn.com/images/ysj7191/post/b5a6c994-2d01-4ddc-8097-1bd00cb35ff0/image.png" /></h2>
<p>유니페스 서비스가 끝나고 AWS 빌링을 확인하던 어느 날, 제 눈을 의심했습니다. <strong>S3 데이터 전송 요금으로 90$</strong>, 우리 돈으로 <strong>약 12만 원</strong>이 찍혀있었습니다. </p>
<p>원인을 파악해 보니 3일간의 축제 기간 동안 <strong>무려 780GB에 달하는 데이터가 S3에서 직접 전송</strong>된 것이었습니다.</p>
<p>이는 유저들이 앱에서 이미지를 볼 때마다 S3 버킷에서 원본 이미지를 그대로 가져다 쓴 결과였습니다. </p>
<p>다행히 AWS에 빌넣한 결과 이번 요금은 철회받을 수 있었습니다.
해당 문제를 이번 기회에 제대로 된 아키텍처를 구축해 요금 폭탄을 원천 차단하기로 했습니다.</p>
<h2 id="cloudfront">CloudFront</h2>
<p>이 문제를 해결하기 위해 가장 먼저 도입한 것은 Amazon CloudFront입니다.</p>
<h3 id="cloudfront-왜-써야-할까">CloudFront, 왜 써야 할까?</h3>
<p>Amazon CloudFront는 html, css, 이미지 파일 같은 웹 콘텐츠를 사용자에게 더 빠르고 효율적으로 배포하는 <code>CDN(콘텐츠 전송 네트워크)</code> 서비스입니다.</p>
<p>핵심은 <code>엣지 로케이션</code> 이라는 전 세계에 퍼져있는 데이터 센터 네트워크입니다. 사용자가 이미지를 요청하면, S3가 있는 먼 곳까지 가지 않고 사용자와 가장 가까운 엣지 로케이션으로 요청이 전달됩니다.</p>
<blockquote>
<p><strong>만약 엣지 로케이션에 이미 이미지 사본(캐시)이 있다면?</strong> 
-&gt; 즉시 사용자에게 전송합니다. (빠름)
&nbsp;
*<em>만약 없다면? *</em>
-&gt; 엣지 로케이션이 원본 서버(Origin), 즉 우리 S3 버킷에서 이미지를 가져와 사용자에게 전달하고, 자신에게 캐시로 저장해 둡니다.</p>
</blockquote>
<p>특히 CloudFront는 AWS의 <strong>백본 네트워크</strong>를 이용하기 때문에, 일반 인터넷망을 통해 S3까지 여러 네트워크를 거쳐가는 것보다 훨씬 빠르고 안정적입니다. </p>
<p>무엇보다 중요한 것은 S3에서 CloudFront 엣지 로케이션으로 데이터를 전송하는 비용은 무료라는 점입니다. </p>
<p>사용자는 CloudFront를 통해 훨씬 저렴한 비용으로 데이터를 전송받게 되므로, <strong>S3 데이터 전송 요금이 줄어듭니다.</strong></p>
<h3 id="cloudfront-세팅">CloudFront 세팅</h3>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/9ce8076b-a707-4233-8436-0b08fd2f7421/image.png" />1. Amazon CloudFront로 접근하여, CloudFront 배포 생성 클릭
<img alt="" src="https://velog.velcdn.com/images/ysj7191/post/97648e1d-d4e3-475e-b67d-38636073e0e6/image.png" />2. CloudFront 이름 지정 및 Single website of app 선택
<img alt="" src="https://velog.velcdn.com/images/ysj7191/post/9a109b77-903f-46b0-8fea-0c0e5f9cd236/image.png" />3. Origin Type을 Amazon S3로 선택 후 Browse S3를 통해 지정
<img alt="" src="https://velog.velcdn.com/images/ysj7191/post/9b25dbe7-16b4-4338-a747-7a2c18e851de/image.png" />4. WAF를 사용하려면 보안 보호 활성화 / 그렇지 않다면 비활성화</p>
<blockquote>
<p>WAF(Web Application Firewall)는 웹 애플리케이션 또는 API에 대한 악의적인 요청을 필터링합니다. 또한 트래픽 발생 지점에 대한 가시성을 높이고 Layer 7 배포된 서비스 거부(DDoS) 공격을 완화하여 애플리케이션 가용성을 확보하고 규정 준수 지시를 더욱 효과적으로 적용할 수 있습니다.
&nbsp;
※ 이번에 필자는 비용 문제로 WAF를 사용하지 않고 아키텍처를 구성합니다.</p>
</blockquote>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/0ae1855e-6d49-4212-8db4-52d2c0b26a45/image.png" />그렇게 생성까지 마무리하면 CloudFront 배포 도메인이 주어지는데, 이걸 이용하면 CloudFront의 세팅은 완료된다.</p>
<h2 id="기존-이미지-최적화">기존 이미지 최적화</h2>
<p>새로운 시스템을 도입했으니, 기존에 S3에 쌓여있던 이미지들도 정리해야 했습니다. 일종의 대청소를 진행한 셈입니다.</p>
<h3 id="db-주소-일괄-변경">DB 주소 일괄 변경</h3>
<blockquote>
<p>이미지 주소를 담고 있는 DB의 모든 레코드를 기존 S3 URL에서 새로 생성한 CloudFront 도메인으로 변경했습니다. 간단한 <strong>UPDATE SQL문</strong>으로 처리했습니다.</p>
</blockquote>
<h3 id="이미지-경량화-스크립트-작성">이미지 경량화 스크립트 작성</h3>
<blockquote>
<p>S3 버킷의 모든 이미지 파일을 내려받아 <strong>jpeg 형식으로 압축</strong>하고,
가로 사이즈가 600px를 초과하는 이미지는 <strong>600px로 리사이징</strong>하여 다시 S3에 업로드하는 코드를 작성했습니다.</p>
</blockquote>
<h3 id="lambda를-이용해서-이미지-경량-자동화">Lambda를 이용해서 이미지 경량 자동화</h3>
<p>앞으로 업로드되는 모든 이미지가 자동으로 경량화되도록 자동화 시스템을 구축하기로 했고, 여기서 AWS Lambda가 등장합니다.
AWS Lambda는 특정 이벤트가 발생했을 때만 코드를 실행하는 <strong>서버리스 컴퓨팅 서비스</strong>입니다.
<img alt="" src="https://velog.velcdn.com/images/ysj7191/post/659575c2-e5a7-4fd0-83b7-248de7ef044e/image.png" />**0. 위에서 작성한 이미지 경량화 코드를 압축해서 S3에 업로드한다.</p>
<ol>
<li>Lambda 함수를 생성한다.**
<img alt="" src="https://velog.velcdn.com/images/ysj7191/post/902a1354-039d-4cf4-98e1-01e3200c5733/image.png" /><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/5d92a7a5-c601-46fa-b51b-4b3ea068f5d0/image.png" /><strong>2. S3 버킷에 '모든 객체 생성 이벤트'를 트리거로 설정한다.</strong></li>
</ol>
<p>AWS Lambda는 인스턴스에 비해 실행된 시간만큼만 비용을 지불하므로 저렴하다는 장점이 있기에 이렇게 세팅을 마무리하였는데, 작업을 진행하던 중 <strong>2가지 문제를 마주했습니다</strong></p>
<h3 id="문제-발생-1---s3-버킷-재귀-호출">문제 발생 1 - S3 버킷 재귀 호출</h3>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/24c50496-c7af-4d60-bb71-d5d55825271b/image.png" />Lambda 함수를 설정하면서 마주한 재귀 호출 문구를 통해 문득 이 아키텍처에는 문제가 있음을 떠올렸습니다. 해당 경우는 다음과 같습니다.</p>
<blockquote>
<ol>
<li>사용자가 원본 이미지를 S3 버킷에 업로드한다.</li>
<li>Lambda 트리거가 발동해 이미지를 최적화한다.</li>
<li>최적화된 이미지를 같은 S3 버킷에 다시 업로드한다.</li>
<li><strong>새로운 파일이 업로드되었으므로 Lambda 트리거가 또 발동한다.</strong>
&nbsp;
(2 ~ 4 과정이 무한 반복될 수 있다)</li>
</ol>
</blockquote>
<h3 id="문제-발생-2---그럼-file-url은-어떻게-관리하지">문제 발생 2 - 그럼 File URL은 어떻게 관리하지?</h3>
<p>Spring 백엔드 서버는 S3에 파일을 업로드할 때, 자기가 지정한 파일명(1EA123-123456-aabe12-1902982.png 등)만 알고 있습니다. 하지만 Lambda에 의해 최적화된 최종 파일은 확장자가 .jpg로 바뀔 수도 있고, 
앞으로 발생할 오류에 따라 다른 확장자를 가질 수도 있습니다.</p>
<p>백엔드 입장에서는 &quot;일단 업로드는 했으니, Lambda가 잘 처리해서 DB에 .jpg로 저장하면 되겠지?&quot; 라고 낙관적으로 생각할 수 있습니다. 하지만 만약 Lambda 함수가 어떤 이유로 실패한다면? DB에는 존재하지만 실제로는 없는 이미지 주소가 저장되어 사용자에 불편을 야기하게 될 것입니다.</p>
<p>두 문제에 대한 해결 과정은 다음 포스트에서 자세히 다뤄보도록 하겠습니다.</p>