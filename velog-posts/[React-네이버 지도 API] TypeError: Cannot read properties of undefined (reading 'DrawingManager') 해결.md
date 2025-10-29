<h3 id="오류-내용">오류 내용</h3>
<blockquote>
</blockquote>
<pre><code class="language-TypeScript">    var drawingManager = new naver.maps.drawing.DrawingManager({
      map: mapInstance,
    });</code></pre>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/42c6e272-b10d-4bd1-9d7b-1793ac9776a9/image.png" /></p>
<p>자료가 많지 않아서 네이버에서 제공하는 예시를 보고 따라했는데 위와 같은 에러가 발생하였다
<a href="https://navermaps.github.io/maps.js.ncp/docs/naver.maps.drawing.DrawingManager.html">https://navermaps.github.io/maps.js.ncp/docs/naver.maps.drawing.DrawingManager.html</a></p>
<p>다시 말해 naver.maps.drawing 이후에 DrawingManager를 발견할 수 없는 문제였다.</p>
<h3 id="해결법">해결법</h3>
<blockquote>
<pre><code class="language-html"></code></pre>
</blockquote>

<p>```</p>
<p>서브 모듈 DrawingManager를 로드하지 않아 생긴 문제였다.
해당 줄을 index.html를 넣어서 해결할 수 있었다.</p>