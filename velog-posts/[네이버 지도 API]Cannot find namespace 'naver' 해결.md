<h2 id="문제">문제</h2>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/3190b97b-4e48-48a0-9083-699145f1e55d/image.png" /><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/22972a21-917e-4da0-8bf0-50a52dbcb69a/image.png" /></p>
<p>naver.maps 라는 namespace에 접근이 안 되어 관련하여 모조리 컴파일이 안 되는 상황이었다.
그 과정에서 @types/navermaps가 인식이 안 되고 있다는 점을 확인하였다</p>
<h2 id="해결">해결</h2>
<p>tsconfig.json 또는 tsconfig.app.json (나는 vite라 그런 듯하다)에서 &quot;types&quot;에 &quot;navermaps&quot;를 추가 해준다</p>
<h3 id="before">Before</h3>
<pre><code>  &quot;compilerOptions&quot;: {
    &quot;types&quot;: [&quot;node&quot;],
    &quot;target&quot;: &quot;ES2020&quot;,
    &quot;useDefineForClassFields&quot;: true,
    &quot;lib&quot;: [&quot;ES2020&quot;, &quot;DOM&quot;, &quot;DOM.Iterable&quot;],
    &quot;module&quot;: &quot;ESNext&quot;,
    &quot;skipLibCheck&quot;: true,
</code></pre><h3 id="after">After</h3>
<pre><code>  &quot;compilerOptions&quot;: {
    &quot;types&quot;: [&quot;node&quot;,&quot;navermaps&quot;],
    &quot;target&quot;: &quot;ES2020&quot;,
    &quot;useDefineForClassFields&quot;: true,
    &quot;lib&quot;: [&quot;ES2020&quot;, &quot;DOM&quot;, &quot;DOM.Iterable&quot;],
    &quot;module&quot;: &quot;ESNext&quot;,
    &quot;skipLibCheck&quot;: true,
</code></pre><p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/b643e48f-c4fa-4ee1-80c1-736029f9e5a9/image.png" />
<img alt="" src="https://velog.velcdn.com/images/ysj7191/post/8a846d15-2135-4608-a105-950a8a2a9222/image.png" /></p>
<p>올바르게 인식하여 편안해진 모습</p>