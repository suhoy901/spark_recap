# Databricks
Azure Databricks Tuto
- https://pages.databricks.com/rs/094-YMS-629/images/01-Delta%20Lake%20Workshop%20-%20Delta%20Lake%20Primer.html

### Change Data Captures
#### 1. CDC Project1
> MySQL -> Debezium -> Kafka -> Spark Streaming -> PostgreSQL <br>
- https://medium.com/@suchit.g/spark-streaming-with-kafka-connect-debezium-connector-ab9163808667
- Git : https://github.com/suchitgupta01/spark-streaming-with-debezium

#### 2. MySQL CDC with Apache Kafka and Debezium
- https://blog.clairvoyantsoft.com/mysql-cdc-with-apache-kafka-and-debezium-3d45c00762e4


## JDBC
### numPartitions, lowerBound, UpperBound
- partitionColumn는 파티션을 결정하는 데 사용되는 컬럼
- lowerBound 및 upperBound은 가져올 값의 범위를 결정. 완전한 데이터 세트는 아래의 쿼리에 해당하는 행을 사용함
~~~sql
SELECT * FROM table WHERE partitionColumn BETWEEN lowerBound AND upperBound
numPartitions는 만들 파티션 수를 결정합니다. lowerBound와 upperBound 사이의 범위는 각각 numPartitions로 나누고 보폭은 다음과 같습니다.
~~~
- upperBound / numPartitions - lowerBound / numPartitions
- 예를 들면
    - lowerBound : 0
    - upperBound : 1000
    - numPartitions : 10
- Stride는 100이되고 파티션은 아래 쿼리에 해당

~~~sql
SELECT * FROM table WHERE partitionColumn BETWEEN 0 AND 100
SELECT * FROM table WHERE partitionColumn BETWEEN 100 AND 200
...
SELECT * FROM table WHERE partitionColumn BETWEEN 900 AND 1000
~~~

### Fetch
- JDBC 드라이버에는 `fetchSize` 라는 원격 JDBC 데이터베이스에서 **한 번에 인출되는 행 수를 제어하는 매개 변수**가 있음. 
- 이 값을 너무 **낮게 설정** 하면 전체 결과 집합을 인출하기 위해 **Spark와 외부 데이터베이스 간에 왕복 요청 수가 많기** 때문에 작업 시간이 지연 될 수 있음
- 이 값이 너무 **높으면 OOMs 위험**이 있습니다. 
- 최적의 값은 결과 스키마, 결과의 문자열 크기 등에 따라 달라 지므로 작업에 따라 달라 지지만, 기본값에서 조금만 증가시키는데도 성능이 크게 향상 될 수 있음
- Oracle의 기본값 fetchSize 은 10임. 100이 훨씬 더 큰 성능 향상을 제공하고, 2000와 같이 더 높은 값으로 이동하면 추가 개선 사항이 제공됨
