<h2 id="서론">서론</h2>
<p>서비스가 성장하며 사용자 수가 급증하면 데이터베이스는 큰 병목 지점이 된다. </p>
<p>쿼리 튜닝이나 인덱스 최적화만으로는 한계에 부딪히는 시점이 오게 되는데, 이때 하드웨어 성능을 올리는 Scale-up 대신 Scale-out 방식인 <strong>'샤딩(Sharding)'</strong>을 고려할 수 있다.</p>
<p>마침 이를 최근 기술면접 스터디에서 짚어봤던지라 간략하게 정리해보고자 한다.</p>
<h3 id="db-샤딩이란">DB 샤딩이란?</h3>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/bbfcce48-e2fb-47c2-88bc-db486d3fc9ef/image.png" /></p>
<p>샤딩은 거대한 하나의 데이터베이스를 <strong>'샤드(Shard)'</strong>라고 부르는 작은 단위로 쪼개어 여러 서버에 분산 저장하는 기술이다.</p>
<h3 id="수직-확장scale-up-vs-수평-확장scale-out">수직 확장(Scale-up) vs 수평 확장(Scale-out)</h3>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/ca940468-62de-478a-bdee-8074ed0ebb2c/image.png" /></p>
<blockquote>
<p><strong>Scale-up</strong>
서버 사양(CPU, RAM)을 높이는 방식이기에 비용이 비싸고 물리적 한계가 존재할 수 밖에 없다</p>
</blockquote>
<blockquote>
<p><strong>Scale-out</strong> - 저렴한 서버를 여러 대 추가하는 방식으로 Scale-up의 단점을 극복할 수 있다</p>
</blockquote>
<h2 id="db-샤딩의-장점">DB 샤딩의 장점</h2>
<h3 id="1-무한한-확장성">1. 무한한 확장성</h3>
<p>이론적으로 샤드 서버를 계속 추가함으로써 저장 공간과 처리량을 무한히 확장할 수 있다.</p>
<h3 id="2-성능-및-부하-분산">2. 성능 및 부하 분산</h3>
<p>데이터가 분산되므로 <strong>개별 서버가 처리해야 할 데이터 양과 인덱스 크기가 줄어든다</strong>.
이는 쿼리 응답 속도 향상으로 직결된다.</p>
<h3 id="3-고가용성availability">3. 고가용성(Availability)</h3>
<p>특정 샤드 서버에 문제가 생겨도 <strong>해당 샤드의 데이터만 영향을 받는다</strong>. 
따라서 전체 서비스가 중단되는 Single Point of Failure(SPOF) 문제를 방지할 수 있다.</p>
<h2 id="실무적-문제점-및-해결-방안">실무적 문제점 및 해결 방안</h2>
<h3 id="1-데이터-재분산rebalancing-문제">1. 데이터 재분산(Rebalancing) 문제</h3>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/8af75eff-8ab2-4b59-b537-e46b027c32ca/image.png" /></p>
<p>특정 샤드에 자주 조회되는 데이터가 몰리는 <strong>'Hotspot' 현상</strong>이 발생하면 샤드를 추가하거나 데이터를 옮겨야 한다.
예를 들자면 스포츠 선수 전체를 저장하는 DB에 손흥민, 김민재, 류현진, 추신수가 하나의 샤드에 있다고 해보자. 해당 샤드에만 요청이 빗발칠 가능성이 높을 것이다. </p>
<p>이러한 부하를 해결하고자 데이터를 옮기려는 시도를 할 수 있는데, 일반적인 Modulo 연산($ID % N$)을 쓰고 있다면 서버 추가 시 거의 모든 데이터를 이동시켜야 하는 재앙이 발생한다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/ad2b427b-b48b-4743-b020-78895cca261f/image.png" />
여기서는 <strong>일관된 해싱(Consistent Hashing) 알고리즘</strong>을 사용하여 노드가 추가되거나 삭제될 때 이동해야 할 데이터 범위를 최소화할 수 있다.</p>
<h3 id="2-분산-조인join-및-트랜잭션의-한계">2. 분산 조인(Join) 및 트랜잭션의 한계</h3>
<p>서로 다른 서버에 저장된 데이터를 Join하는 것은 사실상 불가능하거나 매우 느리다.
또한 여러 샤드에 걸친 트랜잭션을 보장하는 것도 매우 어렵습니다.</p>
<p>해결 방안으로는 두 가지가 있다.
첫째, <strong>비정규화(Denormalization)</strong>를 통해 Join이 필요한 데이터를 미리 합쳐서 저장하는 방법.
둘째, 가급적 한 서비스 내의 관련 데이터는 동일한 샤드에 담기도록 샤드 키를 전략적으로 설계하는 방법.</p>
<h3 id="3-글로벌-유니크-id-보장">#3 글로벌 유니크 ID 보장</h3>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/3662ada4-2067-4ad4-93fe-acbf540f50e2/image.png" /></p>
<p>단일 DB라면 auto_increment를 쓰면 되지만, 여러 샤드에서는 ID 중복 문제가 발생할 수 있다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/3e100b9d-8af1-4ce9-a14e-67968cf1c858/image.png" /></p>
<p>이 경우 Snowflake ID(Twitter 방식)나 UUID, 또는 별도의 ID 생성 전용 DB(Ticket Server)를 운영하여 전역적으로 유일한 식별자를 생성하는 것을 시도해볼 수 있겠다.</p>