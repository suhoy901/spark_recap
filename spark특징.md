# 스파크 특징

## 스파크가 하둡보다 나은 점

- 여러가지 언어 지원
- 대화형 쉘 지원
- 다양한 함수 지원(Map Reduce 를 포함하여 집계함수나 MR, Streaming 등)
- in memory, cache, 카탈리스트 엔진 등을 통한 작업 속도 향상
- 여러 데이터 저장 서비스 지원(HBase, MongoDB, S3 등)

 

 

 

## 스파크의 특징

![img](https://k.kakaocdn.net/dn/daPs0W/btqCxQHpNdw/yNOUIogC8N5pww8moQTA31/img.png)

> 위의 그림을 참고.

- 하나의 노드 위에서 여러개의 executor 가 실행될 수 있지만, 하나의 executor 가 여러 노드에 걸쳐있을 순 없다.
- 하나의 데이터 파티션은 하나의 executor 위에서 처리되지만, 여러 executor 위에서 처리될 순 없다.
- 하나의 executor 가 갖는 core 수에 따라 파티션을 받고, 그 수 만큼 병렬 처리 할 수 있다.
- 한 번 생성된 executor 는 애플리케이션이 끝날 때 까지 남아있다.
- spark 에서 캐싱한 RDD 등은 executor 위에 남아있는다.
- 하나의 어플리케이션은 하나의 SparkSession 을 만든다. 즉, 하나의 어플리케이션 = 하나의 Spark Session
- 데이터가 **물리적으로** 여러 컴퓨터에 나뉘어져 있는 것이 파티션(청크) 

 

### 카탈리스트

- 실행 계획 수립이나 처리에 사용하는 자체 데이터 타입 정보를 갖고 있는 엔진.

- python 이나 R 등으로 구조적 API 를 사용해도, 카탈리스트가 내부적으로 스파크 데이터 타입으로 변환하여 명령을 효율적으로 처리.

 

### Dataframe 은 Row 타입으로 구성된 Dataset임.

- Row 타입은 자체 데이터 포맷을 사용하기 때문에, Garbage collector 나 객체 초기화 부하가 있는 JVM 데이터 타입 사용 안 함. 따라서 매우 효율적인 연산이 가능.

 

### API 실행 과정

1. 코드
2. 논리적 실행 계획 : 카탈리스트 옵티마이저가 사용자의 코드에서 최적화 할 수 있는 부분을 찾아 최적화
3. 물리적 실행 계획 : 논리적 실행 계혹이 **실제 클러스터에서 실행하는 방법**을 정의. 비용 모델을 비교하여 최적의 전략 선택.
   - 이 과정에서 사용자 쿼리를 RDD 트랜스포메이션으로 컴파일함.
4. 실행 : RDD 를 대상으로 코드 실행

 

### 브로드캐스트 변수

- 모든 task 에서 볼 수 있는(공유되는) 불변값. 직렬화/역직렬화 하지 않아서 큰 데이터 공유하는 데 효율적임.

 

### SparkSession

- 저수준 API, 기존 컨텍스트, 관련 설정 정보 등에 접근 가능



### SparkContext

- SparkSession 으로 접근 가능. 스파크 클러스터에 대한 연결을 나타냄. RDD 같은 저수준 API, 브로드캐스트 변수 등을 사용 가능.

 

### 실행 계획 : job > stage > task

- job : action 에 의해 생성. action 이 2개면 job 2개 생성됨. 여러 stages 로 구성됨
- **stage** : 넓은 transformation 에 의해 생성. 다수의 머신에서 동일한 연산을 수행하는 여러 tasks 의 그룹.
  - 하나의 stage 내에 tasks 들은 모두 같은 일을 하며, 병렬 처리 한다.
  - 따라서 stage 1 이 끝나고 난 뒤에야 stage 2 가 시작된다.

- task : 단일 익스큐터에서 실행할 데이터의 블록과 다수의 좁은 transformation 의 조합.
  - 파티션 개수에 따라 task 가 만들어짐. 만약 파티션이 5개면 5개의 task 가 만들어져서 5개가 병렬 실행됨
  - 병렬 처리되는 모든 task 는 **서로 다른 데이터**를 대상으로 **동일한 코드**를 실행한다.

 

### DAG

- DAG 에서 각 job 의 스테이지 그래프를 만들고, 각 task 가 실행될 위치를 결정한다.
- action 에 의해 실행되는 job 은 하나의 DAG 를 만든다.
  - 액션이 한 번 호출된 이후에는 더 이상 DAG 에 추가 안 됨.

 

### Spark이 빠른 이유

- spark 가 빠른 이유는, in memory 를 사용하기 때문도 있지만, 코드 최적화 ( 좁은 transformation 을 파이프라인으로 뭉쳐서, IO 없이 한 번에 처리 ) 가 잘 되기 때문도 있다.
- Spark 에서 **셔플이 발생하면, (사용자가 직접 캐싱하지 않아도 ) 셔플 결과가 디스크에 저장**되어진다.
  - 나중에 재사용 할 일이 생기면, 셔플 결과를 다시 가져와서 사용한다. 마치 캐싱 한 것 처럼!

 



### 잡스케쥴링

- 잡 스케줄링을 할 수 있다.
- 스파크 애플리케이션에서 별도의 스레드를 사용해 여러 job을 동시에 실행 가능하다 ( 스레드 안정성은 보장 
- 기본적으로 FIFO 큐를 사용하며, head 부분의 job이 전체 자원을 사용하지 않는다면, 남은 자원은 두번째 job 이 사용하여 병렬 처리 가능하다.
- FIFO 처럼 순서대로 처리할 수 있고, 또 round robin 방식을 사용할 수 있다. 이것을 fair scheduler 라고 한다.
- 긴 job이 오면 처리하다가 짧은 job 이 오면 잠깐 자원을 줘서 실행시키고 다시 긴 job 을 처리하는 방식.

 

### 정적 할당

- **하나의 클러스터**에서 **여러 스파크 애플리케이션을 실행**하려면, 실행될 애플리케이션을 위해, 클러스터가 갖고 있는 제한적인 **자원을 최대한 할당**해줌. 자원을 갖고 있는 애플리케이션이 끝나야 다음 애플리케이션이 실행됨.

 

### 동적 할당

- **하나의 클러스터**에서 **여러 스파크 애플리케이션을 실행**하려면, 워크로드에 따라 **애플리케이션이 점유하는 자원을 동적으로 조정**해야 함. 애플리케이션이 사용하지 않는 자원을 클러스터에 반환하고 필요할 때 다시 요청하는 방식. 필요할 때 받고 다 쓰면 반납. 
- 다수의 애플리케이션이 하나의 스파크 클러스터 자원을 공유하는 환경에서 유용함.

 



## executor 이해

- 스파크의 모든 executor는 JVM (자바 가상 머신)을 하나 실행한 후 그 위에서 실행된다.
- 스파크 드라이버에서 모든 애플리케이션의 상태가 보관되며, 안정적으로 실행 중인지 확인 가능하다.
- RDD 캐싱 : 물리적 데이터(bit 값 그 자체)를 캐시(executor 의 ram)에 저장
  - 구조적 API 캐싱 : 물리적 실행 계획을 캐시(executor 의 ram)에 저장

- 캐싱 레벨로 Memory_only 를 사용할 때, JVM(executor) 에 할당된 메모리보다 RDD 크기가 크면, 캐싱을 하지 않는다.
- RDD 는 executor 혹은 slave node 위에 저장된다. 애플리케이션이 동작하는 동안 로드된 RDD 는 executor 노드의 메모리 위에 있다.
- executor 는 캐싱과 실행을 위한 공간을 각각 갖고 있는 JVM 이다!
- 파티션이 중간에 유실되어도, RDD 재계산에 필요한 종속성 그래프에 대한 충분한 정보를 통해 빠른 복구가 가능하다.
- 넓은 transformation 이후에 파티션 개수가 따로 지정되어있지 않다면, spark.default.parallelism 으로 직접 파티션 개수 지정이 가능하다.
- 각 executor 에 얼마만큼의 cpu cores 를 할당하는 게 좋은가?
  - 물론 컴퓨터 환경마다 다르겠지만, 일단 책의 내용은 아래와 같다.
  - 각 노드가 갖고 있는 cpu cores 를 5로 나눈 몫만큼 각 노드에 executor 를 설정한다.
  - 즉, 노드마다 5개 cores를 갖는 executor 가 최대한 들어가게 만든다. (샌디 라이자가 말하길, executor 당 코어는 최대 다섯 개 정도가 좋다고 함)

 



![img](https://k.kakaocdn.net/dn/bgdhTz/btqCAPUS8kN/3IbHDK7CslmaFohKvrXMl1/img.png)executor 의 메모리 사용 영역



### Cluster Manger와 Executor

- cluster manager 가 할당한 memory 자원으로 executor 가 만들어지면, executor 는 위와 같이 ram 을 구별하여 사용한다.
  1. overhead : 어디에 쓰이는 건지는 아직 모르겠지만, 아무튼 필요함. 얼마만큼 필요한지 계산하는 방법은 아래 적어둠 
  2. M 영역 : executor 가 갖고 있는 전체 memory 에서 overhead 를 제외한 부분.실행(transformation)과 캐싱을 위한 공간. 기본 값은 60%이며, 늘리거나 줄일 수 있음 
  3. 실행 영역 : transformation 을 처리하기 위한 공간
  4. R : 캐싱된 파티션을 저장하기 위한 공간. 캐싱된 파티션이 비어있으면 이 공간을 실행 영역으로 사용할 수 있다고 함(위에 그림처럼)
     - (연산이 많지 않다면) R 을 벗어나 실행 영역으로 캐싱 파티션을 저장 가능하지만, 연산이 많아지게 되면, 연산에 실행 영역을 보장해줘야 하므로 가장 오래 전에 사용된 캐싱 파티션을 제거한다(LRU)

- 각 executor 가 실행할 때의 memory overhead : Max ( 요청한 executor 메모리의 n % , 기본 오버헤드 값(384mb) )여기서 n % 는 default가 10 % 인데, 내가 맘대로 수정 가능.



 yarn container 에서 executor 를 실행한다고 할 때, yarn 의 memory 가 executor 에서 요청한 메모리+overhead 보다 커야 한다.

 

- executor 의 memory M 공간 계산 하는 방법

  - M = spark.executor.memory * spark.memory.fraction
  - (M = 전체 executor memory * M에 할당된 비율)

  

- executor 의 memory R 공간 계산 하는 방법
  - R = spark.executor.memory * spark.memory.fraction * spark.memory.storageFraction
  - (R = 전체 executor memory * M에 할당된 비율 * R에 할당된 비율)