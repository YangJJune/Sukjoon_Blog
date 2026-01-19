<p>리액트 생태계에 입문한지 오래되었지만, 아직도 모르는 게 태산이다.
오늘은 그동안 모르고 넘어갔던 부분들을 한번 짚어보고자 한다.</p>
<hr />
<h2 id="1-ts-vs-tsx-확장자의-한-끗-차이">1. TS vs TSX: 확장자의 한 끗 차이</h2>
<p>처음에 리액트를 시작할 때, styled-components의 style를 분리해서 작성하였는데 이를 tsx로 작성하였다가 코드리뷰에서 ts로 작성해야한다는 피드백을 받았다. 그 정확한 차이는  <strong>&quot;JSX 문법을 포함하느냐&quot;</strong>에 있다.</p>
<blockquote>
<p><strong><code>.ts</code></strong> : 순수한 <strong>로직</strong> 파일이다. 함수, 인터페이스, API 호출 정의 등 UI 요소(HTML 태그 형태)가 없는 경우에 사용합니다.</p>
</blockquote>
<blockquote>
<p>  <strong><code>.tsx</code></strong> : JSX 문법을 포함한 TypeScript 파일이다. TypeScript 문법에 JSX(HTML 형태의 코드)를 함께 사용할 때 반드시 이 확장자를 써야 한다.</p>
</blockquote>
<hr />
<h2 id="2-jsx란-무엇인가">2. JSX란 무엇인가?</h2>
<p>그렇다면 자연스럽게 JSX가 무엇인지 궁금해진다. JSX란 자바스크립트 안에서 UI 구조를 직관적으로 설계할 수 있게 돕는 확장 문법이다. 갑자기 왜 확장이 필요했는지는 아래 문서를 보면 알 수 있다.</p>
<h3 id="순수-js로-짠-경우">순수 JS로 짠 경우</h3>
<pre><code class="language-javascript">// React.createElement(태그명, 속성, 자식요소)
const element = React.createElement(
  'div',
  { className: 'container' },
  React.createElement(
    'h1',
    { style: { color: 'blue' } },
    '안녕하세요!'
  ),
  React.createElement(
    'p',
    null,
    'JSX 없이 리액트를 코딩하면 이런 모습입니다.'
  ),
  React.createElement(
    'button',
    { onClick: () =&gt; alert('클릭됨!') },
    '확인'
  )
);</code></pre>
<h3 id="jsx-문법">JSX 문법</h3>
<pre><code class="language-javascript">const element = (
  &lt;div className=&quot;container&quot;&gt;
    &lt;h1 style={{ color: 'blue' }}&gt;안녕하세요!&lt;/h1&gt;
    &lt;p&gt;JSX를 사용하면 코드가 훨씬 직관적입니다.&lt;/p&gt;
    &lt;button onClick={() =&gt; alert('클릭됨!')}&gt;
      확인
    &lt;/button&gt;
  &lt;/div&gt;
);</code></pre>
<p>JSX 문법이 훨씬 직관적임을 알 수 있다.
추상화가 중요한 요즘엔 이런 부분이 매우 중요하다고 생각한다.</p>
<p>결론적으로 해당 JSX 문법을 사용하는 경우 .tsx 그렇지 않은 경우 .ts 확장자를 사용하면 되겠다.</p>
<h2 id="2-jsx와-babel의-관계">2. JSX와 Babel의 관계</h2>
<p>우리는 리액트에서 HTML처럼 생긴 코드를 작성하지만, <strong>사실 브라우저는 이를 이해하지 못 한다</strong>
이때 필요한 것이 <strong>Babel</strong>이다.</p>
<p><strong><code>Babel</code></strong>은 최신 자바스크립트나 JSX를 모든 브라우저가 해석할 수 있는 <strong>일반 자바스크립트(ES5 이하)</strong>로 변환해 주는 컴파일러/트랜스파일러이다.</p>
<p>※ 더 알아본 결과 무조건 ES5로 변환되는 것은 아니고 정확히는 <strong>&quot;타겟 브라우저가 이해할 수 있는 문법&quot;</strong>으로 변환하는 것이라고 한다.</p>
<h3 id="babel의-동작방식">Babel의 동작방식</h3>
<p><strong>1. Parsing</strong>
바벨은 소스 코드를 분석하여 AST(Abstract Syntax Tree)로 변환한다.
AST란, 소스코드의 추상 구문 구조의 트리로, 어떤 식으로 변환되는지는 하단에서 확인할 수 있다</p>
<blockquote>
<p><a href="https://astexplorer.net/">https://astexplorer.net/</a></p>
</blockquote>
<p><strong>2. Transformation</strong>
Parsing 단계에서 생성된 AST를 브라우저가 지원하는 오래된 문법으로 AST를 변경한다.</p>
<blockquote>
<p>ex) <code>const</code>를 <code>var</code> 형태로 변경, <code>화살표 함수</code>를 <code>일반 함수</code>로 변경 등</p>
</blockquote>
<p><strong>3. Generating</strong>
수정된 AST를 다시 실제 자바스크립트 코드로 출력한다.</p>
<hr />
<h2 id="3-번들bundle과-최적화의-기술">3. 번들(Bundle)과 최적화의 기술</h2>
<p>현대의 웹 앱은 수많은 파일로 나뉜다. 
이를 효율적으로 배포하기 위해 '번들링'이 필요하다.</p>
<h3 id="번들bundle">번들(Bundle)</h3>
<blockquote>
<p>흩어져 있는 수십 개의 JS, CSS 파일을 하나로 묶은 결과물이다.
(Webpack, Vite 등이 이 일을 수행)</p>
</blockquote>
<h3 id="번들-최적화란"><strong>번들 최적화란?</strong></h3>
<p>파일 크기를 줄여 로딩 속도를 높이는 과정으로, 다음과 같은 방법들이 주로 사용된다</p>
<blockquote>
<p><strong>Tree Shaking :</strong> 나무를 흔들어 죽은 잎을 떨어뜨리듯, 사용하지 않는 코드를 제거한다
<strong>Code Splitting :</strong> 필요한 코드만 먼저 불러오도록 파일을 나눈다
<strong>Minification :</strong> 공백과 줄바꿈을 없애 용량을 최소화한다</p>
</blockquote>
<hr />
<h2 id="4-이-모든-것은-어떻게-ssr과-연결될까">4. 이 모든 것은 어떻게 SSR과 연결될까?</h2>
<p>번들링과 최적화 개념을 이해했다면, 왜 <strong>SSR(Server-Side Rendering)</strong>이 강력한 도구인지 더 이해하기 쉽다.</p>
<table>
<thead>
<tr>
<th align="left">특징</th>
<th align="left">SSR (Server-Side Rendering)의 이점</th>
</tr>
</thead>
<tbody><tr>
<td align="left"><strong>초기 로딩 (FCP)</strong></td>
<td align="left">브라우저가 거대한 번들 파일을 다 받기 전에, 서버에서 미리 만든 HTML을 보여주어 사용자 이탈을 막는다.</td>
</tr>
<tr>
<td align="left"><strong>Hydration</strong></td>
<td align="left">서버에서 받은 정적 HTML에 JS 번들이 '결합'되면서 상호작용이 가능하다.</td>
</tr>
<tr>
<td align="left"><strong>SEO 성능</strong></td>
<td align="left">완성된 HTML을 제공하므로 검색 엔진 로봇이 페이지 내용을 즉시 파악할 수 있다.</td>
</tr>
</tbody></table>
<hr />
<blockquote>
<p>※ <strong>Hydration</strong>이란, 서버에서 렌더링된 정적 HTML에 클라이언트 자바스크립트를 연결해 상호작용 가능한 상태로 만드는 과정이다.</p>
</blockquote>