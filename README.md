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
