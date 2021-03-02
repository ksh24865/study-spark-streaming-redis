# study-spark-streaming-redis

# Apache spark


>
- 유사한 straming sw(storm, flink, samza 등)들과 실시간 분산처리 성능은 유사하고,
- 또한 데이터의 유실을 방지하는 exactly once도 유사하게 지원한다.
- Apache spark의 장점은 실시간 처리와 함께 다양한 plugin(Graphx, SQL, MLlib)을 제공하여 원하는 제품을 쉽게 확장할 수 있는데 그 장점이 있다. (Apache Flink도 일부 유사함)
- 또한 수많은 commiter & contrubuter들의 참여 및 기업에서의 적용/투자를 통하여 제품의 안정성 및 성능이 지속적으로 검증 및 개선된다는 장점도 있다
- 이는 Open source sw를 도입하고자 하는 기업의 관점에서는 가장 중요한 항목중에 하나이다.
>
## 1. Install
  * install
  ```
  $ cd ~/study-spark-streaming-redis/sw/
  $ wget http://d3kbcqa49mib13.cloudfront.net/spark-2.0.1-bin-hadoop2.7.tgz
  $ tar -xvf spark-2.0.1-bin-hadoop2.7.tgz
  $ cd spark-2.0.1-bin-hadoop2.7  
  ```
  * set spark configuration
    * spark environment
    ```
    # slave 설정
    $ cp conf/slaves.template conf/slaves
    localhost //현재  별도의 slave node가 없으므로 localhost를 slave node로 사용
    # spark master 설정
    # 현재 demo에서는 별도로 변경할 설정이 없다. (실제 적용시 다양한 설정 값 적용)
    $ cp conf/spark-env.sh.template conf/spark-env.sh
    ```
    * add spark path to system path
    ```
    vi ~/.bashrc
    export SPARK_HOME=~/demo-spark-analytics/sw/spark-2.0.1-bin-hadoop2.7
    export PATH=$PATH:$SPARK_HOME/bin
    ```
    * Set ssh connection without password
    ```
    $ ssh-keygen -t rsa  # 설정사항 모두 공백인 상태로 Enter
    $ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
    $ chmod og-wx ~/.ssh/authorized_keys 
    $ ssh user_id@localhost # 정상적으로 접속되는지 확인
    ```
## 2. Run
  * run spark master
  ```
  $ cd ~/study-spark-streaming-redis/sw/spark-2.0.1-bin-hadoop2.7
  $ sbin/start-all.sh
  ```
  * open spark master web-ui with web browser
  ```
  http://localhsot:8080
  ```

# redis
>
- In memory cache, NoSQL key-value data store.
>

## 1. Install (redis 3.0.7)
  * install redis
  ```
  $ cd ~/study-spark-streaming-redis/sw
  $ wget http://download.redis.io/releases/redis-3.0.7.tar.gz
  $ tar -xzf redis-3.0.7.tar.gz
  $ cd redis-3.0.7
  $ make
  ```
  * python에서 redis에 접속하기 위해서 redis 라이브러리를 설치
  ```
  $ sudo pip install redis
  $ sudo pip install numpy
  ```

## 2. run 
```
> src/redis-server
```


## 3. test
  * set & get example
  ```
  > cd ~/study-spark-streaming-redis/sw/redis-3.0.7
  > src/redis-cli
  redis> set kim seongho
  OK
  redis> get kim
  "seongho"
  ```
  * hashmap example
  ```
  redis> HSET testhash field1 "kim"
  (integer) 1
  redis> HSET testhash field2 "seongho"
  (integer) 1
  redis> HGETALL testhash
  1) "field1"
  2) "kim"
  3) "field2"
  4) "seongho"

  redis> HGET myhash field1
  "kim"
  ```

# elasticsearch & kibana
  * 기존 사용하던 Docker-Elasticsearch-Kibana 사용
# apache kafka
  * 기존 사용하던 docker-kafka-zookeeper 사용

# logstash
## 1. Install (redis 3.0.7)
  * install
  ```
  $ mkdir ~/study-spark-streaming-redis/sw
  $ cd ~/demo-spark-analytics/sw
  $ wget https://artifacts.elastic.co/downloads/logstash/logstash-7.6.1.tar.gz
  $ tar xvf logstash-7.6.1.tar.gz
  ```
  * set logstash path to $path
  ```
  $ vi ~/.bashrc
  export PATH=$PATH:~/study-spark-streaming-redis/sw/logstash-7.6.1/bin
  ```
## 2. test
  ```
  logstash -e 'input { stdin { } } output { stdout {} }'
  # 아래와 같은 메세지가  stdin 입력을 받을 준비가 됨.
  Settings: Default pipeline workers: 1
  Pipeline main started
  # 메세지 입력 후 엔터
  hi logstash
  # 아래와 같은 메세지가 출력되며 정상
  2021-03-0201:22:14.405+0000 0.0.0.0 hi logstash
  # Ctrl + D로 종료
  ```
  
# 실습

* 개요
  * logstash에서 kafka로 저장하고, 이를 spark에서 실시간 분산처리 후 ES에 저장
  * logstash ➡ kafka ➡ spark streaming ➡ ES <br/>
 　　　　　　　　　　　　redis⤴ <br/>
* 특징
  * logstash의 biz logic(filter)을 단순화하여 최대한 많은 양을 전송하는 용도로 활용한다.
  * kafka를 이용하여 대량의 데이터를 빠르고, 안전하게 저장 및 전달하는 Message queue로 활용한다.
  * Spark streaming은 kafka에서 받아온 데이터를 실시간 분산처리하여 대상 DB(ES)에 병렬로 저장한다.
  * redis는 spark streaming에서 customer/music id를 빠르게 join하기 위한 memory cache역할을 한다.
* Software 구성도
<img width="865" alt="stage2" src="https://user-images.githubusercontent.com/55729930/109618961-89eaea80-7b7b-11eb-9992-aa8348f3b464.png">

# kafka
* create kafka topic(realtime)
  * logstash에서 수집한 log 메세지를 kafka로 보낼 때, realtime topic을 지정한다.
  ```
  $ cd ~/study-spark-streaming-redis/sw/kafka_2.12-2.6.0
  $ bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic realtime
    # check created topic "realtime"
    # replication-factor : 메세지를 복제할 개수 (1은 원본만 유지)
    # partitions : 메세지를 몇개로 분산하여 저장할 것인지 결정 (갯수 만큼 병렬로 write/read 함)
  $ bin/kafka-topics.sh --list --zookeeper localhost:2181
  realtime
  ```
# redis
* import customer info to redis
  * run import_customer_info.py (read customer info and insert into redis)
  ```
  $ cd ~/study-spark-streaming-redis/00.stage2
  $ python import_customer_info.py
  ```
  * redis에 정상적으로 저장되었는지 확인
  ```
  $ ~/demo-spark-analytics/sw/redis-3.0.7/src/redis-cli
  127.0.0.1:6379> keys *
  # 사용자 id별로 key 생성
     1) "4042"
     2) "4763"
  ...
  ...
  4998) "1434"
  4999) "782"
  ```
  
  　
   
  ```
  127.0.0.1:6379> hgetall 2 
  #사용자 id 2번에 대한 정보를 조회
   1) "name"
   2) "Paula Peltier"
   3) "gender"
   4) "0"
   5) "age"
   6) "28"
   7) "zip"
   8) "66216"
   9) "Address"
  10) "10084 Easy Gate Bend"
  11) "SignDate"
  12) "01/13/2013"
  13) "Status"
  14) "1"
  15) "Level"
  16) "0"
  17) "Campaign"
  18) "4"
  19) "LinkedWithApps"
  20) "1"
  ```
    
# logstash
* Read logs and send to kafka
  * logstash configuration
    * input : tracks_live.csv ( data_generator로 생성하는 file )
    * filter : 적용하지 않음 ( logstash는 빠르게 수집, 실제 데이터에 대한 처라(filter, aggregation...)는 spark streaming에서 처리 )
    * output : logs에서 읽은 문자열을 그대로 kafka로 produce
  ```
  
  ```
  * run logstash
