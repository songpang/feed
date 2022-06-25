
_WIP_
    
> _들어가기 전에 먼저_

## NoSQL DB

+ Not Only, Non Relational SQL

![EDED3FB0-CDB3-4A3B-8EBA-157B6BFEF15F](https://user-images.githubusercontent.com/39265399/175777781-0231576a-e0d3-411d-9e9c-05c3b1f21c02.png)

  
  SQL과 다른 특성을 가지고있는 NoSQL DataBase.  
  일반적인 SQL DB는 **ACID**의 특성을 따르는 반면 NoSQL는 **BASE** 특성을 따른다.  
  
  
  | 항목 | BASE | ACID |
|:---|:---:|:---:|
| `적용대상` | NoSQL | RDBMS |
| `범위` | 시스템 전체 대상 | 개별 트랜잭션 적용 |
| `중점사항` | 약한 일관성 | 강한 일관성 |
| `중점사항` | 성능과 가용성 | 무결성, 일관성 |
| `관리주체` | 주로 개발자 | DBMS 트랜잭션 |
| `데이터처리` | 유사 응답 허용 | 처리 순서 보장 |
| `변겅성` | 변경 어려움 | 변경 용이 |
| `디자인` | 쿼리 디자인 중요 | 테이블 디자인 중요 |
| `CAP이론` | C+P, A+P 만족 | C+A 만족 |
| `적용사례` | Big Table | Orable RAC |  
  
  
  NoSQL DB같은 경우는 인덱스를 거는 것이 RDBMS에 비해서 완전하게 비효율적이다. 또한 Insertion Time에는 정확한 Read를 보장하지 않는다.  
  여러 서버에 미러링된 한 파티션에 Data를 Insert했을 때 다른 서버에 Read요청이 들어온다면 일관성이 보장되지 않는다는 뜻.  
  (하지만 수 millisecond 안에 처리되기 때문에 사실 상 일반적인 경우에 큰 문제가 되지는 않는다.)  
  
  그만큼 Insertion에 최적화된 DB가 NoSQL DB. 이런 특성에 의해서 심지어 READ(간단한 조회보다는 여러 조건이 들어간)는 RDBMS에 비해서 느릴 수 있다.
  
  일반적으로 SQL DB는 Schema에 대해서 굉장히 Strict한 특성을 가지고 있는 반면 NoSQL DB는 Schema-Free한 특성을 지니고 있다.  
  또한 NoSQL DB로 DB를 설계할 경우 Scale-out에 굉장히 유리하다.    
  
  => RDBMS가 Scale-out이 불가능하다는 이야기는 아니다.  
     샤딩과 같은 기법을 통해서 RDBMS도 충분히 Scale-out이 가능하다. NoSQL의 유일특성이라고 생각하면 오산!
  
  자, 그럼 Schema가 어떻다는 이야기일까?
  
### Strict Schema
  
  일반적인 RDBMS는 테이블을 만들어놓고 데이터를 INSERT할 경우에 구조에 맞게 맞춰야한다.  
  Schema를 정의해놓고 엉뚱한 데이터를 집어넣었는데 DB에 데이터가 적재될 수 있을까?  
  강한 스키마로 되어있으면 스키마에 대한 의존때문에 데이터를 마구잡이로 집어 넣을 수 없다!  
  
  + Schema에 Strict하며 인덱스가 걸려있다면 탐색 속도는 흔히 알 수 있듯이 logN의 속도로 발견된다.  
 
  
### Schema - Free

  NoSQL의 강력한 특징은 Schema에 유연하다는 것이다. 심지어 Schema-less한 특성이 NoSQL의 특징이라고 말할 수도 있다.  
  이 부분은 NoSQL의 비정형성(unstructured) 데이터의 특징과 일맥상통 한다.  
  문제는 스키마가 없기 때문에 언제든 필드를 추가/수정할 수 있지만 잘못된 데이터가 들어가더라도 어디서도 알려주지 않는다.  
  정합성이 떨어질 수밖에 없고 Transaction을 사용하기 힘들기 때문에 금융, 결제와 같은 곳에 사용하기는 부적합하다.


_Cassandra DB를 파악하기 전에 NoSQL에 대해서 먼저 정리를 해 보았는데, 이러한 특징을 기억하고 Cassandra의 구조, 특성을 정리해보자._

  
# DB : Cassandra

- 아파치 카산드라(Apache Cassandra)는 자유 오픈 소스 분산형 노에스큐엘(NoSQL) 데이터베이스 관리 시스템(DBMS)의 하나로, 단일 장애 점 없이 고성능을 제공하면서 수많은 서버 간의 대용량의 데이터를 관리하기 위해 설계되었다.  
- 카산드라는 여러 데이터센터에 걸쳐 클러스터를 지원하며 마스터리스(masterless) 비동기 레플리케이션을 통해 모든 클라이언트에 대한 낮은 레이턴시 운영을 허용하며, 성능 면에서 높은 가치를 보인다.  
- Amazon의 Dynamo 분산 스토리지 및 복제 기술과 Google의 Bigtable 데이터 및 스토리지 엔진 모델이 결합된 모델로 처음에 단계적 이벤트 기반 아키텍처 (SEDA)를 사용하여 Facebook에서 설계되었다.  

### Cassandra의 데이터모델

- Key space
- Table
- Row
- column name : column value.  
  => SET, LIST, MAP 도 칼럼에 저장 가능. 
  
> (참고).  
> Cassandra : Key-space > Table > Row > Column name : Column value.  
> Mongodb : db > collection > document > key:value.  
> RDBMS : DB > Table > row > column.  
> elasticsearch : index > type> document > key : value.  


