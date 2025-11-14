<h2 id="문제상황">문제상황</h2>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/81d43b19-d3a3-4696-a9c2-42740119281e/image.png" /></p>
<p>이전 포스트에서 작성했던 다음과 같은 플로우를 구동하던 중, 3번에서 4번 과정이 제대로 트리거되지 않음을 확인할 수 있었다. </p>
<p>하나의 워크플로우로 합치는 방안도 있겠지만, 각각 workflow에 소모되는 시간도 있고 코드 난독화가 우려되어 기존에 2개의 워크플로우로 분리했던 점은 유지하고자 했다.</p>
<p>오늘은 이 문제를 해결해보는 과정을 서술해보려한다.</p>
<h2 id="해결과정">해결과정</h2>
<h3 id="시도-1---skip-ci-삭제">시도 1 - skip ci 삭제</h3>
<pre><code class="language-java">    run: |
      git config user.name &quot;github-actions[bot]&quot;
      git config user.email &quot;github-actions[bot]@users.noreply.github.com&quot;
      git add public/forum-posts.json
      # 변경사항이 있을 때만 커밋
      if git diff --staged --quiet; then
        echo &quot;No changes to commit&quot;
      else
        git commit -m &quot;chore: update forum-posts.json [skip ci]&quot;
        git push
        echo &quot;✅ Changes committed and pushed&quot;
      fi</code></pre>
<p>fetch하는 경우 다음과 같이 <code>skip ci</code> 구문이 포함이 되어 있었다.
우리는 fetch된 json파일에 변화가 있다면 웹페이지를 재배포를 해야하므로 해당 구문을 삭제하였다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/897de461-a69d-4703-9eaf-68c777f3df66/image.png" /></p>
<p>그러나 여전히 원하는 동작이 이루어지지 않았다.
다른 문제가 더 있는지 살펴볼 필요가 있을 것 같다.</p>
<h3 id="시도-2---github_token-변경">시도 2 - GITHUB_TOKEN 변경</h3>
<pre><code class="language-java">jobs:
  fetch-data:
    runs-on: ubuntu-latest
    permissions:
      contents: write # 파일 커밋을 위한 쓰기 권한

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          token: ${{ secrets.GITHUB_TOKEN }}</code></pre>
<p>다음과 같이 워크플로우 전반에 GITHUB_TOKEN을 사용하여 작업을 수행했다.</p>
<p>그러나 GITHUB_TOKEN을 토큰으로 사용하는 경우는 간편하지만, 다음과 같은 주의사항이 있음을 간과하였다.</p>
<blockquote>
<p>워크플로 실행에서 저장소를 사용하여 코드를 푸시하는 경우, 저장소에 이벤트 발생 시 실행되도록 구성된 워크플로가 포함되어 있더라도 새 워크플로는 실행되지 않습니다</p>
</blockquote>
<blockquote>
<p><code>GITHUB_TOKEN</code>를 사용하여 GitHub Actions 워크플로에서 푸시된 커밋은 GitHub Pages 빌드를 트리거하지 않습니다.</p>
</blockquote>
<p><strong>출처 |</strong> <a href="https://docs.github.com/en/actions/concepts/security/github_token">https://docs.github.com/en/actions/concepts/security/github_token</a></p>
<p>따라서 Github Token 대신 기존에 이용하던 <code>PAT 토큰</code>을 가져와서 사용하기로 하였다.</p>
<p>안타깝게도 이것이 문제를 해결해주지는 않았다.</p>
<h3 id="시도-3---pat-권한을-적절히-주었는가">시도 3 - PAT 권한을 적절히 주었는가?</h3>
<p>PAT를 생성할 때 아래 사진과 같이 권한을 설정할 수 있다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/6785469a-b77e-40a1-a591-bb19b2e3ade0/image.png" /></p>
<p>우리는 workflow에 적용하여 다른 workflow를 트리거하는 등의 작업을 해야하므로 그러한 권한이 필요하다.
따라서 새로운 PAT를 생성할 때 repo와 workflow 권한을 주었다.</p>
<h2 id="결과">결과<img alt="" src="https://velog.velcdn.com/images/ysj7191/post/14155dc2-7f09-47d5-abe8-30deefbf6cd2/image.png" /></h2>
<p>드디어 생각한 대로 잘 수행됨을 확인할 수 있었다.</p>