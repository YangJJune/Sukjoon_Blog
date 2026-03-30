<h2 id="서론">서론</h2>
<p>최근 기술 면접 스터디에서 <code>JPA</code>와 <code>Hibernate</code>를 주제로 모의면접을 진행하였었다
항상 JPA를 쉽게 DB를 다루기 위해서만 사용했었는데 이번 기회에 좀 더 깊게 알아보고 공부한 내용들을 남겨두고자 이번 포스트를 작성했다.</p>
<h3 id="orm-이란">ORM 이란?</h3>
<p>ORM은 이름 그대로 <strong>'객체(Object)'와 '관계형 데이터베이스(Relational Database)'를 매핑(Mapping)</strong> 해주는 기술이다.
우리가 프로그래밍할 때는 '객체 지향' 언어(Java)를 사용하고, 데이터를 저장할 때는 '관계형 데이터베이스'(MySQL, Oracle 등)를 사용한다. 이 둘은 데이터를 다루는 패러다임이 애초에 다르기 때문에 누군가 중간에서 객체를 SQL로, SQL을 객체로 변환해주는 작업이 필요한데, ORM 프레임워크가 바로 그 역할을 수행한다.</p>
<h3 id="jpa-란">JPA 란?</h3>
<p>JPA는 Java Persistence API 자바 진영의 ORM 기술에 대한 <strong>표준 명세(API)</strong> 이다.
중요한 점은 JPA 자체가 특정 기능을 수행하는 라이브러리나 프로그램이 아니라는 것이다. JPA는 인터페이스의 모음이며, &quot;자바에서 ORM 기술을 사용하려면 이러한 인터페이스들을 구현해서 만들어라&quot;라고 정해둔 <strong>규칙(가이드라인)</strong> 이라고 이해하면 된다.</p>
<h3 id="jpa를-왜-사용할까">JPA를 왜 사용할까?</h3>
<p>JPA를 사용하는 가장 큰 이유는 기존 SQL 중심 개발에서 발생하는 <strong>'패러다임의 불일치'</strong> 문제를 해결하기 위해서이다.</p>
<p>객체 지향 프로그래밍은 추상화, 캡슐화, 상속, 다형성 등을 제공하지만, 관계형 데이터베이스는 오직 데이터 중심으로 구조화되어 있어 이러한 개념들을 지원하지 않는다. 예를 들어, Java의 '상속' 구조를 DB 테이블에 그대로 저장하고 불러오는 것은 매우 번거로운 일이다.</p>
<p>기존의 MyBatis를 활용한 방식을 생각해보자.</p>
<pre><code>&lt;!-- 기존의 MyBatis를 활용한 쿼리 --&gt;
&lt;!-- animal이라는 테이블에 데이터를 저장하는 쿼리 --&gt;
&lt;insert id=&quot;insert&quot; parameterType=&quot;Animal&quot;&gt;
    insert into animal (animal_name, sex) values (&quot;고양이&quot;, &quot;수컷&quot;)
&lt;/insert&gt;</code></pre><p>이렇듯 기존에는 객체를 DB에 넣기 위해 CRUD 쿼리를 개발자가 하나하나 완벽하게 매핑하여 작성해주어야 했다. 만약 객체의 필드가 하나라도 수정되면 모든 관련 쿼리를 뜯어고쳐야 했다.</p>
<p>하지만 JPA를 활용한다면 어떻게 될까?</p>
<pre><code>// JPA를 활용할 때
em.persist(animal);</code></pre><p>이 한 줄의 코드로 끝난다.
JPA가 객체를 분석하여 내부적으로 적절한 <code>INSERT</code> SQL을 생성하고 DB에 전달해 준다. Java의 상속, 다형성 같은 패러다임과 DB 구조 사이의 불일치를 JPA가 중간에서 해소해 주는 것이다. 다시 말해, 개발자는 SQL 중심 개발에서 벗어나 본연의 업무인 <strong>'객체 중심 개발'</strong> 에 집중할 수 있게 된다.</p>
<h3 id="jpa를-사용했을-때의-장점">JPA를 사용했을 때의 장점</h3>
<p>이러한 특징들을 바탕으로 JPA를 도입했을 때 얻을 수 있는 장점은 구체적으로 다음과 같다.</p>
<p><strong>1. 압도적인 생산성</strong>
JPA가 제공하는 API를 사용하면 객체를 DB에 저장하고 조회할 때 지루하고 반복적인 CRUD 쿼리를 직접 작성할 필요가 없다. <code>persist()</code>, <code>find()</code> 등 제공되는 메서드를 호출하기만 하면 되므로, 비즈니스 로직 개발에 훨씬 더 많은 시간을 투자할 수 있다.</p>
<p><strong>2. 유지보수의 편리함</strong>
MyBatis 같은 SQL Mapper를 사용할 때는 객체의 필드(컬럼) 하나가 추가되거나 변경되면, 연관된 모든 DTO와 SQL 쿼리문을 직접 찾아서 수정해야 했다. 반면 JPA는 엔티티(Entity) 클래스의 필드만 수정하면, 관련된 SQL은 JPA가 알아서 동적으로 생성해주므로 유지보수가 매우 수월해진다.</p>
<p><strong>3. 성능 최적화 기능 제공</strong>
JPA는 애플리케이션과 데이터베이스 사이에 존재하는 계층이다. 이 중간 계층의 이점을 활용하여 1차 캐시, 쓰기 지연(Transactional Write-Behind), 지연 로딩(Lazy Loading) 등 다양한 성능 최적화 기법을 자동으로 제공한다. (이는 잘 이해하고 사용할 경우 시스템 성능을 비약적으로 끌어올릴 수 있다.)</p>
<p><strong>4. 벤더 독립성 (특정 DB에 종속되지 않음)</strong>
관계형 데이터베이스는 벤더(MySQL, Oracle, PostgreSQL 등)마다 미묘하게 다른 SQL 문법(방언, Dialect)을 사용한다. JPA는 설정 파일에서 사용할 데이터베이스의 종류만 지정해주면, 해당 DB에 맞는 쿼리를 알아서 번역해 준다. 따라서 훗날 데이터베이스를 변경해야 하는 상황이 오더라도 애플리케이션 코드를 거의 수정하지 않고 대처할 수 있다.</p>
<h3 id="지연-로딩lazy-loading의-원리">지연 로딩(Lazy Loading)의 원리</h3>
<p>JPA에서 성능 최적화를 논할 때 빠질 수 없는 것이 지연 로딩이다. 엔티티 매니저가 엔티티를 로드할 때, JPA는 연관된 객체까지 모두 가져오는 대신 <strong>프록시 객체(HibernateProxy)</strong> 를 실제 엔티티 대신 반환한다.</p>
<ul>
<li><strong>껍데기 프록시 객체 반환:</strong> 프록시 객체는 실제 엔티티와 동일한 인터페이스를 상속받은 껍데기이다.</li>
<li><strong>실제 데이터 로드 지연:</strong> 프록시 객체는 연관된 엔터티에 대한 참조를 가지고 있지만, 사용되기 전까지는 그 엔터티의 데이터를 로드하지 않는다.</li>
<li><strong>메서드 호출 시 로딩:</strong> 프록시 객체의 실제 데이터가 필요한 시점(ex. <code>getter</code> 호출)에 데이터베이스에 쿼리를 날려 실제 데이터를 가져오고 프록시를 초기화한다.</li>
<li><strong>초기화 이후:</strong> 한 번 프록시 객체가 초기화되면, 이후에는 DB 조회 없이 실제 객체를 참조하여 값을 반환한다.</li>
</ul>
<p><strong>지연 로딩을 사용해야 하는 이유</strong>
연관된 데이터가 필요 없는 상황에서도 항상 즉시 로딩(Eager Loading)을 사용하면 엄청난 성능 저하가 발생한다. 또한 즉시 로딩은 개발자가 예상하지 못한 거대한 JOIN SQL을 발생시킨다. 따라서 실무에서는 모든 연관 관계(ManyToOne, OneToOne 포함)에 지연 로딩(Lazy Loading) 전략을 사용하는 것이 기본 원칙이다.</p>
<h3 id="dirty-checking-변경-감지">Dirty Checking (변경 감지)</h3>
<p>JPA는 엔티티의 변경 사항을 자동으로 감지하여 데이터베이스에 반영한다. 이를 더티 체킹이라 부른다.
원리는 간단하다. 영속성 컨텍스트는 엔티티가 최초로 조회될 때의 상태를 <strong>스냅샷(Snapshot)</strong> 으로 저장해 둔다. 트랜잭션이 커밋되어 <strong>플러시(Flush)</strong>가 발생할 때, 현재 엔티티의 상태와 스냅샷을 비교하여 변경된 부분이 있다면 자동으로 UPDATE 쿼리를 생성하여 쓰기 지연 SQL 저장소에 등록한다. </p>
<p>즉, 개발자는 데이터를 수정하고 <code>save()</code>나 <code>update()</code> 메서드를 명시적으로 호출할 필요가 없다.</p>