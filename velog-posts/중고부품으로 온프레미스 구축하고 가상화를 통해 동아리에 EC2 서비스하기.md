<h2 id="갑자기-동아리-ec2를-만들고-싶었던-계기">갑자기 동아리 EC2를 만들고 싶었던 계기</h2>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/36ee47d3-e129-43c8-9c67-3d050e26071e/image.png" /></p>
<p>작년에 컴퓨터를 케이스갈이를 하면서 업그레이드 하는 과정에 남는 부품이 생겼었다.
이게 1년 째 방치되어있다보니, 어디 좋은 곳에 쓸 수 없을까 생각을 하다 동아리에 복지 차원에서 서버를 열면 2~3개의 프로젝트는 굴릴 수 있을 거 같아서 온프레미스를 구축하기로 했다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/909e6814-d52b-4381-93f8-dd6ba32fbcb1/image.png" /></p>
<p>당연히 중고 하드디스크를 들고 있는 부원은 없어서 사비를 들여 중고거래로 HDD, PSU를 구매하고 데스크탑을 구축하기로 했다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/65c2a5d7-57b2-4383-ada1-64a772af9e4f/image.png" /></p>
<p>여담이지만 나사를 드럽게 못 박아서 친구의 도움을 조금 받아 4~50분 만에 완성할 수 있었다.
아무래도 미니타워 케이스다 보니까 귀여운 맛이 있는 편이다.</p>
<p>OS는 안정적이고 익숙한 Ubuntu 24.04 LTS 버전으로 설치하였다.</p>
<p>그럼 이제부터 이 컴퓨터로 가상화를 통해 각 프로젝트 별로 완전히 격리된 환경을 제공하려면 어떻게 해야할까?</p>
<h2 id="kvm이란">KVM이란?</h2>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/d7686be9-6979-4a85-8f25-eae499df29db/image.png" /></p>
<blockquote>
<p><strong>커널 기반 가상 머신(Kernal-based Virtual Machine)</strong>은 물리적 Linux 시스템에 설치하여 가상 머신을 생성할 수 있는 소프트웨어다. VMWare와는 다르게 오픈소스 기반이라는 점과  장점이 있다.
<strong>가상머신(Virtual Machine)</strong>은 물리적 시스템과 CPU 사이클, 네트워크 대역폭 및 메모리와 같은 리소스를 공유한다.</p>
</blockquote>
<h3 id="kvm-환경-구성하기">KVM 환경 구성하기</h3>
<pre><code>$ lscpu | grep Virtualization</code></pre><p>위는 가상화를 지원하는 CPU인지 체크하는 명령어다.
해당 명령어를 입력하였을 때 <strong>VT-x</strong> 또는 <strong>AMD-V</strong>로 표기되면 사용 가능하다.</p>
<pre><code>$ sudo apt update
$ sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients virtinst virt-manager bridge-utils</code></pre><p>위의 명령어는 우분투 기준으로, 다른 OS의 경우 달라질 수 있다.</p>
<pre><code>$ sudo adduser $USER kvm
$ sudo reboot</code></pre><p>위의 명령어로 현재 계정을 KVM 그룹에 추가할 수 있다.
이후에는 reboot을 통해 OS를 재시작함으로서 KVM을 적용한다.</p>
<h2 id="cockpit">Cockpit</h2>
<p>이제 KVM 환경은 구성이 되었다. 그러나 CLI 상에서 KVM을 다루는 것보다 웹 UI가 있으면 부원들을 상대로 설정하기 편할 것이라고 생각하였다.
찾아본 결과 Cockpit이라는 라이브러리가 있었다.</p>
<pre><code>sudo apt install cockpit
sudo systemctl enable --now cockpit.socket</code></pre><p>다음의 명령어로 cockpit을 설치하고 활성화할 수 있다.
이를 외부에서 접속이 가능하도록 하려면 9090포트의 포트포워딩을 진행하면 된다.</p>
<h3 id="cockpit이-제공하는-기능">Cockpit이 제공하는 기능</h3>
<ul>
<li>웹 브라우저로 VM 생성 및 관리</li>
<li>원격 콘솔 접속 (VNC 또는 터미널)</li>
<li>리눅스 소프트웨어의 업데이트</li>
<li>etc...</li>
</ul>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/a67e7e1f-a9dc-4450-9269-a099e6400f00/image.png" />9090포트에 접속하였을 때, 위의 페이지가 뜨면 성공이다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/027728d2-bb07-472d-ba72-f3890bb299c1/image.png" />다음과 같이 여러 요소를 AWS 만지듯 구성할 수 있다. 사용법은 자세히 다루지 않겠습니다.
(제가 이번에 가상머신의 분리에만 집중해서 차차 알아나가려고 합니다)</p>
<h2 id="동아리원과-공유를-해보자">동아리원과 공유를 해보자</h2>
<img src="https://velog.velcdn.com/images/ysj7191/post/277674f2-b482-4d12-bd93-e6e36279526a/image.png" width="100%" />
<img src="https://velog.velcdn.com/images/ysj7191/post/e993a0ed-e6ae-4d1a-ba69-d9d4f89e5c94/image.png" width="100%" />

<p>동아리원들의 좋은 반응과 함께 결과적으로 <strong>2개의 프로젝트</strong>가 한 집 살림을 하게 되었다.
사실 이 경우 KVM을 굳이 써야하나, 쿠버네티스를 사용하면 어떨까 라는 생각도 들지만 KVM이라는 것을 한번 구축해보고 싶었기에 문제가 없다면 괜찮다고 생각합니다.</p>
<p>여담이지만, AI한테 저 모집 공고를 던져서 이미지로 만들어달라고 하니...
<img src="https://velog.velcdn.com/images/ysj7191/post/de7b8ca0-782a-4010-b2ac-0f330d296191/image.png" width="500px" /><img src="https://velog.velcdn.com/images/ysj7191/post/b4647186-af90-43c4-b78f-4e46e4dd41df/image.png" width="500px" /><div>정말 너무 무서운 것이다.</div></p>
<h2 id="기술적인-부채">기술적인 부채?</h2>
<p>여담이지만 PC에 한 가지 잠재적인 문제점이 있다. 단 하나의 1TB HDD로 구성이 되어있다는 것이다.
하나의 PC가 하나의 디스크를 쓰는 것은 자연스러운 현상이다. 그러나 여러 주체가 같은 디스크를 쓰기 시작하면 물리적으로 무서운 일이 벌어진다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/64ce4b75-122a-4556-a9ed-1543757f740e/image.png" />
HDD는 위와 같이 둥그런 플래터를 헤더가 직접 방문하여 읽고 쓰는 형태이다. 따라서 분명히 <strong>물리적인 수명이 존재한다</strong>.
하나의 PC라면 정상적으로 최적화가 되어, 최대한 적은 섹터를 방문하려고 할 것이다.</p>
<p>그러나 여러 가상머신이 하나의 HDD에서 돌아가게 되면, 아마 그런 최적화는 정상적으로 동작하지 않을 것이다.
따라서 물리적으로 헤더가 왔다갔다 움직여야하는 거리가 늘어나며, 속도가 느려짐과 동시에 수명이 줄어들 가능성이 높을 것이다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/27dc7126-568b-4a82-81db-e5559a477e39/image.png" />
이를 해결하기 위해 소프트웨어적으로 해결할 수 있는 방안을 고안하거나, RAID 10을 적용시킬까 생각하다, 단순히 물리적으로 하드디스크를 분리하여 사용하여 해결하기로 하였다.</p>
<p>또한 이번에 SSH를 진행하며 방화벽과 가상화 환경에서 네트워크에 대해 다룬 내용들이 있는데, 이를 정리해서 다음에 다루어보고자 한다.</p>
<h3 id="참조">참조</h3>
<p><a href="https://aws.amazon.com/ko/what-is/kvm/">https://aws.amazon.com/ko/what-is/kvm/</a>
<a href="https://42morrow.tistory.com/entry/%EC%9A%B0%EB%B6%84%ED%88%AC-2404%EC%97%90-KVM-%EA%B0%80%EC%83%81%ED%99%94-%ED%99%98%EA%B2%BD-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0">https://42morrow.tistory.com/entry/%EC%9A%B0%EB%B6%84%ED%88%AC-2404%EC%97%90-KVM-%EA%B0%80%EC%83%81%ED%99%94-%ED%99%98%EA%B2%BD-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0</a>
<a href="https://somaz.tistory.com/441">https://somaz.tistory.com/441</a></p>