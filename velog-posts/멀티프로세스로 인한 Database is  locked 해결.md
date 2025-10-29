<h3 id="문제상황-설명">문제상황 설명</h3>
<p>소켓을 통해 실시간 위치 정보를 공유하는 서비스를 제작하던 중, 2개의 프로세스를 통해 하나의 DB 파일에 접근하다보니 다음과 같은 오류가 발생했다.</p>
<blockquote>
<p>Database is locked</p>
</blockquote>
<p>오늘은 이를 해결하기 위한 지식들과 해결방법에 대해 정리해보았다.</p>
<h3 id="db-lock이란">DB Lock이란?</h3>
<p>DB 락이란 DB의 일관성과 무결성을 유지하기 위해 트랜잭션의 순차적 진행을 보장할 수 있는 직렬화 장치이다. 사전적의미 그대로 DB를 Lock 한다는 것이다. </p>
<p>그럼 이걸 이번에 사용하는 SQLite3를 토대로 자세히 파보도록 하자.</p>
<h2 id="sqlite3에서의-락과-동시성-문제">SQLite3에서의 락과 동시성 문제</h2>
<blockquote>
<p>SQLite3 에서 잠금 및 동시성 제어는 pager 모듈 에서 처리하며, 이는 SQLite를 &quot;ACID&quot;하게 관리한다.</p>
</blockquote>
<p><strong>ACID</strong>가 뭘까? ACID를 알아보기 위해서는 Transaction에 대해 먼저 알아야한다.</p>
<h3 id="transaction">Transaction</h3>
<p><strong>여러 작업을 하나로 묶은 단위</strong>이다. 중요한 건 하나의 묶음이 전부가 다 이루어지거나 이루어지지 않는다는 점이다. 즉, 하나의 Transaction에서 일부 작업만 성공하는 경우는 없다는 뜻이다.</p>
<p>ACID는 각각 풀이해보면 다음과 같다.</p>
<h3 id="atomicity">Atomicity</h3>
<p>원자성 : 모든 작업이 반영되거나 롤백된다</p>
<h3 id="consistency">Consistency</h3>
<p>일관성 : 데이터는 미리 정의된 규칙에서만 수정이 가능하다</p>
<h3 id="isolation">Isolation</h3>
<p>고립성 : 데이터베이스가 상태의 불일치 없이 여러 트랜잭션이 동시에 발생할 수 있도록 보장한다</p>
<h3 id="durability"><strong>Durability</strong></h3>
<p>지속성 : 성공적으로 수행된 트랜잭션 커밋을 로그에 추가한다 (장애의 경우 복구 가능)</p>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/8980073f-dfa9-47c8-992b-311b225d984d/image.png" /></p>
<p>이 시점에서 지금 문제가 되는 부분을 살펴보면, SQLite3는 Isolation을 위해 Lock을 진행하도록 설계했음을 알 수 있다.</p>
<p>SQLite3는 다음과 같은 유형으로 Lock을 관리한다고 한다.</p>
<table>
<thead>
<tr>
<th>UNLOCKED</th>
<th>기본 상태. 락이 걸려있지 않은 상태로, 어떤 프로세스든 자유롭게 락을 걸 수 있다.</th>
</tr>
</thead>
<tbody><tr>
<td>SHARED</td>
<td>DB를 읽을 수 있으나 쓸(write) 수 없다. 여러 프로세스가 동시에 SHARED 락을 획득할 수 있으나 이 상태에서는 쓰기가 아예 불가능하다.</td>
</tr>
<tr>
<td>RESERVED</td>
<td>프로세스가 쓰기를 계획하고 있는 상태이다. 따라서 한번에 RESERVED 락은 하나만 걸려있을 수 있다. 다만, SHARED 락과 공존할 수 있는 상태이다.</td>
</tr>
<tr>
<td>PENDING</td>
<td>프로세스가 쓰기 작업을 목전에 두고 있는 상태. 모든 SHARED 락이 해제되기를 기다리고, 완료되면 EXCLUSIVE 락을 획득한다. 새로운 SHARED 락은 허용되지 않으나, 기존 SHARED 락은 허용된다.</td>
</tr>
<tr>
<td>EXCLUSIVE</td>
<td>DB에 쓰기 작업을 위해 필요한 락이다. 하나의 파일에 단 하나의 EXCLUSIVE 락만 허용된다. SQLite는 동시성을 최대화하기 위해 EXCLUSIVE 락을 최소한의 시간으로 관리한다고 한다.</td>
</tr>
</tbody></table>
<h3 id="올바른-해결방법">올바른 해결방법</h3>
<p>우선 공식 사이트 FAQ에 내용이 있어 가져와봤다.</p>
<p><strong>(5) Can multiple applications or multiple instances of the same application access a single database file at the same time?</strong></p>
<blockquote>
<p>Multiple processes can have the same database open at the same time. Multiple processes can be doing a SELECT at the same time. But only one process can be making changes to the database at any moment in time, however.</p>
<p>SQLite uses reader/writer locks to control access to the database. (Under Win95/98/ME which lacks support for reader/writer locks, a probabilistic simulation is used instead.) But use caution: this locking mechanism might not work correctly if the database file is kept on an NFS filesystem. This is because fcntl() file locking is broken on many NFS implementations. You should avoid putting SQLite database files on NFS if multiple processes might try to access the file at the same time. On Windows, Microsoft's documentation says that locking may not work under FAT filesystems if you are not running the Share.exe daemon. People who have a lot of experience with Windows tell me that file locking of network files is very buggy and is not dependable. If what they say is true, sharing an SQLite database between two or more Windows machines might cause unexpected problems.</p>
<p>We are aware of no other <em>embedded</em> SQL database engine that supports as much concurrency as SQLite. SQLite allows multiple processes to have the database file open at once, and for multiple processes to read the database at once. When any process wants to write, it must lock the entire database file for the duration of its update. But that normally only takes a few milliseconds. Other processes just wait on the writer to finish then continue about their business. Other embedded SQL database engines typically only allow a single process to connect to the database at once.</p>
<p>However, client/server database engines (such as PostgreSQL, MySQL, or Oracle) usually support a higher level of concurrency and allow multiple processes to be writing to the same database at the same time. This is possible in a client/server database because there is always a single well-controlled server process available to coordinate access. If your application has a need for a lot of concurrency, then you should consider using a client/server database. But experience suggests that most applications need much less concurrency than their designers imagine.</p>
<p>When SQLite tries to access a file that is locked by another process, the default behavior is to return SQLITE_BUSY. You can adjust this behavior from C code using the <a href="https://www.sqlite.org/c3ref/busy_handler.html">sqlite3_busy_handler()</a> or <a href="https://www.sqlite.org/c3ref/busy_timeout.html">sqlite3_busy_timeout()</a> API functions.</p>
</blockquote>
<p>위의 내용 중 해당되는 내용을 확인해보면, <strong>&quot;여러 프로세스가 동시에 같은 DB를 열 수 있게 하였으며, 동시에 읽기는 가능하나 쓰기는 불가능하다.&quot;</strong> 기재되어있다.</p>
<p>즉, 멀티 프로세스가 하나의 DB를 쓸 수 있도록은 했다는 취지로 되어있으나, 이 분야에서는 PostgreSQL이나 MySQL, Oracle 등을 사용하는 것이 더 좋다고 기재되어있다.</p>
<p>다만 아무래도 여러 사이트에서 “멀티 프로세스를 쓰는데 SQLite를 왜 함?”이라는 늬앙스의 글을 많이 보았으며, 현재 시간적인 제약이나 확실한 해결 등을 이유로 멀티 프로세스가 아닌 <strong>단일 프로세스가 DB를 조회</strong>하는 것으로 해결하고자 한다.</p>
<h3 id="해결-방안">해결 방안</h3>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/d9cb582b-ef23-4b00-bbf9-6b9409910035/image.png" /></p>
<p>고안한 방법은 2가지인데, 사실상 하나로 볼 수도 있다. 바로 데이터베이스에 접근하는 프로세스를 하나로 만들어 락 문제가 일어날 위험을 제거하는 것이다.</p>
<p>새롭게 그러한 역할을 하는 코드를 짜서 IPC(Inter Process Communication) 통신을 진행할 수도 있고, 기존 백엔드 서버를 통해 우회하는 방법이 있다.</p>
<p>가장 깔끔한 것은 DB Controller 역할을 하는 프로세스를 하나 더 파는 것이라 생각하지만, 시간 관계상 <strong>&quot;백엔드 서버에 엔드포인트를 하나 파서 localhost를 통해 접근한다&quot;</strong>라는 방법으로 작업하였다.</p>
<h3 id="적용-결과">적용 결과</h3>
<p><img alt="" src="https://velog.velcdn.com/images/ysj7191/post/72477c67-f9cf-45a5-8ed6-daa4417c3d02/image.png" /></p>
<p>해당 문제 없이 잘 작동하는 것을 확인할 수 있었다.</p>
<h3 id="참조">참조</h3>
<p><a href="https://www.sqlite.org/lockingv3.html">https://www.sqlite.org/lockingv3.html</a></p>