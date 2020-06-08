**对各种数据结构的命令格式一般是：action key value**

*字符串类型操作*

增:set, 删: del, 查: get

```

127.0.0.1:6379> set hello world
OK
127.0.0.1:6379> get hello
"world"
127.0.0.1:6379> del hello
(integer) 1
127.0.0.1:6379> get hello
(nil)

```

*列表类型操作*

增:lpush、rpush, 删: lpop、rpop, 查: lrange、lindex

```

127.0.0.1:6379> rpush list-key item
(integer) 1
127.0.0.1:6379> rpush list-key item2
(integer) 2
127.0.0.1:6379> rpush list-key item
(integer) 3
127.0.0.1:6379> lrange list-key 0 -1
1) "item"
2) "item2"
3) "item"
127.0.0.1:6379> LINDEX list-key 1
"item2"
127.0.0.1:6379> lpop list-key
"item"
127.0.0.1:6379> lrange list-key 0 -1
1) "item2"
2) "item"

```

*集合类型操作*

增:sadd, 删: srem, 查: smembers、sismember

```

127.0.0.1:6379> sadd set-key item
(integer) 1
127.0.0.1:6379> sadd set-key item2
(integer) 1
127.0.0.1:6379> sadd set-key item3
(integer) 1
127.0.0.1:6379> sadd set-key item
(integer) 0
127.0.0.1:6379> smembers set-key
1) "item3"
2) "item2"
3) "item"
127.0.0.1:6379> SISMEMBER set-key item4
(integer) 0
127.0.0.1:6379> SISMEMBER set-key item
(integer) 1
127.0.0.1:6379> SREM set-key item2
(integer) 1
127.0.0.1:6379> srem set-key item2
(integer) 0
127.0.0.1:6379> SMEMBERS set-key
1) "item3"
2) "item"

```

*散列类型操作*

增:hset, 删: hdel, 查: hget、hgetall

```

127.0.0.1:6379> hset hash-key sub-key1 value1
(integer) 1
127.0.0.1:6379> hset hash-key sub-key2 value2
(integer) 1
127.0.0.1:6379> hset hash-key sub-key1 value1
(integer) 0
127.0.0.1:6379> HGETALL hash-key
1) "sub-key1"
2) "value1"
3) "sub-key2"
4) "value2"
127.0.0.1:6379> HDEL hash-key sub-key2
(integer) 1
127.0.0.1:6379> HDEL hash-key sub-key2
(integer) 0
127.0.0.1:6379> HGET hash-key sub-key1
"value1"
127.0.0.1:6379> HGETALL hash-key
1) "sub-key1"
2) "value1"

```

*有序集合类型操作*

增:zadd, 删: zrem, 查: zrange、zrangebyscore

```

127.0.0.1:6379> zadd zset-key 728 member1
(integer) 1
127.0.0.1:6379> zadd zset-key 982 member0
(integer) 1
127.0.0.1:6379> zadd zset-key 982 member0
(integer) 0
127.0.0.1:6379> zrange zset-key 0 -1
1) "member1"
2) "member0"
127.0.0.1:6379> zrange zset-key 0 -1 withscores
1) "member1"
2) "728"
3) "member0"
4) "982"
127.0.0.1:6379> ZRANGEBYSCORE zset-key 0 800 withscores
1) "member1"
2) "728"
127.0.0.1:6379> zrem zset-key member1
(integer) 1
127.0.0.1:6379> zrem zset-key member1
(integer) 0
127.0.0.1:6379> zrange zset-key 0 -1 withscores
1) "member0"
2) "982"

```

