# study-spark-streaming-redis

# Apache spark


>
- 유사한 straming sw(storm, flink, samza 등)들과 실시간 분산처리 성능은 유사하고,
- 또한 데이터의 유실을 방지하는 exactly once도 유사하게 지원한다.
- Apache spark의 장점은 실시간 처리와 함께 다양한 plugin(Graphx, SQL, MLlib)을 제공하여 원하는 제품을 쉽게 확장할 수 있는데 그 장점이 있다. (Apache Flink도 일부 유사함)
- 또한 수많은 commiter & contrubuter들의 참여 및 기업에서의 적용/투자를 통하여 제품의 안정성 및 성능이 지속적으로 검증 및 개선된다는 장점도 있다
- 이는 Open source sw를 도입하고자 하는 기업의 관점에서는 가장 중요한 항목중에 하나이다.
>

# redis
>
- In memory cache, NoSQL key-value data store.
>

## 1. Install (redis 3.0.7)
```
> cd ~/demo-spark-analytics/sw
> wget http://download.redis.io/releases/redis-3.0.7.tar.gz
> tar -xzf redis-3.0.7.tar.gz
> cd redis-3.0.7
> make
```

## 2. run 
```
> src/redis-server
```


## 3. test
```
> cd ~/demo-spark-analytics/sw/redis-3.0.7
> src/redis-cli
redis> set kim seongho
OK
redis> get kim
"seongho"
```

## etc
- hashmap example
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
  * 기존 사용하던 
# lo

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
