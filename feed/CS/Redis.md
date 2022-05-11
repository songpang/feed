# Redis

> TODO
> Redis를 어떻게 설명할 것인가?  
> Redis를 data store로 쓰는 경우 리소스가 너무 비싸지 않은가? 그럼에도 불구하고 쓰는 서비스가 있는가? 있다면 왜쓰는가?  
> Redis와 Memcached를 비교하고 어떤 이유때문에 Redis가 점유율 경쟁에서 이겼는가?  
> Redis를 썼을 경우, 쓰지 않았을 경우가 있다면 어떤 .강력하고. .명확한. 이점때문에 사용하는가?  
> 이렇게 좋은 Redis를 사용하지 말아야 하는 경우가 있을까?  
> 강연 내용 마저 정리  



### Single Threaded
1. Packet이 processInputBuffer에서 command로 완성되면 처리
2. processCommand가 완료되기 전까지는 다음 command가 실행될 수 없음
3. 이 부분으로 인해서 Redis 전체의 Atomic이 보장되고 있다.

> Single Threaded의 의미?
> 동시에 여러개의 명령을 처리할 수 없다
> -> 그래서 수행 시간이 긴 명령을 수행하면 문제가 발생한다
> Redis의 경우 단순한 get/set의 경우 초당 10만 TPS 이상이 가능하다.
> -> 메모리 사이즈, CPU 속도에 영향을 받는다

### Redis의 대표적인 O(N) 명령들
1. 명확한 명령들
KEYS, FlushAll, FlushDB
2. 실수하기 쉬운 경우
많은 데이터가 있는 Collection 지우기
많은 데이터가 있는 Collection 가져오기.

### O(N) 대표적인 실수 사례

#### KEYS

- Key의 개수가 많은데 모니터링을 위해서 keys를 사용하는 경우
-> 몇천 단위일 경우에는 사용해도 무방하다.
-> 1분마다 keys를 사용해서 모니터링을 한다면 서비스가 1분마다 튈 수 있다.

* Key는 Scan으로 변경 가능
-> 하지만 Scan은 딱 그 시점의 스냅샷은 아닐 수도 있다는 Issue가 있을 수 있다
-> 그렇다면 이 문제는 어떻게 해결해야 할까..?

#### 잘못된 자료구조

- CASE - 예전 Spring Security의 RedisTokenStore 이슈
1. Access Token의 저장을 list에 하고 여기서 검색하고 삭제하는 기능이 있었다.
2. 서비스가 성장하면서 Collection에 많은 개수의 데이터가 들어가면서 로그인 장애 발생
3. 현재는 패치  [참고 Link](https://github.com/spring-projects/spring-security-oauth/commit/60f39ce82f380299cb1894baa02d65606f8f1365)

* 검색이 중요할 경우 O(N)의 List가 아니라 set, sorted set, hash 등으로 변경함으로써 속도 개선이 가능하다.

#### 잘못된 접근

- CASE
1. 랭킹이 유사한 유저들을 매칭해주는 기능이 있다.
2. 전체 유저의 랭킹을 가져와서 처리를 하기 위해서 sorted set의 전체구간을 가져왔었다.

* 전체 데이터가 아닌 일부 구간의 데이터를 나눠서 가져오는 것으로 해결할 수 있다.

#### 잘못된 데이터

- CASE
1. Hashes 하나의 칼럼에 18MB 정도의 데이터가 들어갔다.
2. 해당 데이터를 가져 갈 때마다 Redis에서 다른 명령을 처리하지 못한다.
3. 어드민에서 이미지를 base64로 인코딩해서 그대로 본문에 embed해서 발생한 이슈

* monitor 명령을 통해서 자주 호출하는 key를 찾은 다음 해당 key의 사이즈를 확인해서 발견하여 문제를 해결했다. 많은 서비스에서 하나의 Collection에 너무 큰 데이터를 집어넣는 실수를 많이 하기 때문에 대처가 필요하다.
* O(N)관련 모니터링 방법
info all 명령으로 commandstats 파트에서 특정 명령들의 call 수와 usec_per_call이 증가하는 지를 확인한다.


### Memory

> Redis는 메모리를 주 저장소로 사용한다
> -> 메모리보다 많은 데이터를 저장할 수 없다는 의미.

* Redis에서 메모리가 부족하면?
-> mexmemory와 memory-policy 설정
-> 메모리 사용량 > max_memroy 라면 memory-policy 설정으로 동작
-> 이게 Eviction.
* Redis의 max-memory는 실제 사용하는 메모리가 아니라 자신이 할당을 얼마나 하는지 계산을 해서 저장해둔 값이기 때문에 실제로 메모리 사용량이 훨씬 많을 수도 있다. (메모리 파편화)


