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
