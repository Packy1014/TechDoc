# Cache Issue

1. __Cache penetration__
+ Cache penetration is a scenario where the data to be searched doesn't exist at DB and the returned empty result set is not cached as well and hence every search for the key will hit the DB eventually. If a hacker tries to initiate some attack by launching lots of searches with such key, the underlying DB layer will be hit too often and may eventually be breakdown
In such cases, there are a few mitigation plans:
> * If there is no data for the key in DB, just return an empty result and cache it for a short period of time(Don't set a long expiration time)
> * Using Bloom filter. Bloom filter is similar to hbase set which can be used to check whether a key exists in the data set. If the key exists, go to the cache layer or DB layer, if it doesn't exists in the data set, then just return.
> * If the searched key has high repeat rate, then can adopt the first solution. Otherwise if the searched key has low repeat rate and the searched keys are too many, can adopt the second solution to filter most of them first.

2. __Cache breakdown__
+ Cache breakdown is a scenario where the cached data expires and at the same time there are lots of search on the expired data which suddenly cause the searches to hit DB directly and increase the load to the DB layer dramatically.
> * This would happen in high concurrency environment. Normally in this case, there needs to be a lock on the searched key so that other threads need to wait when some thread is trying to search the key and update the cache. After the cache is updated and lock is released, other threads will be able to read the newly cached data.
> * Another feasible method is to asynchronously update the cached data through a worker thread so that the hot data will never expire.

3. __Cache avalanche__
+ Cache avalanche is a scenario where lots of cached data expire at the same time or the cache service is down and all of a sudden all searches of these data will hit DB and cause high load to the DB layer and impact the performance. To mitigate the problem, some methods can be adopted:
> * Using clusters to ensure that some cache server instance is in service at any point of time. If Redis is used, can have redis clusters.
> * Some other approaches like hystrix circuit breaker and rate limit can be configured so that the underlying system can still serve traffic and avoid high load.
> * Can adjust the expiration time for different keys so that they will not expire at the same time.
+ _Example:_ ⽬前电商⾸⻚以及热点数据都会去做缓存 ，⼀般缓存都是定时任务去刷新，或者是查不到之后去更新的，定时任务刷新就有⼀个问题。如果所有⾸⻚的Key失效时间都是12⼩时，中午12点刷新的，我零点有个秒杀活动⼤量⽤户涌⼊，假设当时每秒6000个请求，本来缓存在可以扛住每秒5000个请求，但是缓存当时所有的Key都失效了。此时1秒6000个请求全部落数据库，数据库必然扛不住。此时，如果没⽤什么特别的⽅案来处理这个故障，DBA启数据库，但是数据库⽴⻢⼜被新的流量给打死了。同⼀时间⼤⾯积失效，那⼀瞬间Redis跟没有⼀样，那这个数量级别的请求直接打到数据库⼏乎是灾难性的，你想想如果打挂的是⼀个⽤户服务的库，那其他依赖他的库所有的接⼝⼏乎都会报错，如果没做熔断等策略基本上就是瞬间挂⼀⽚的节奏，你怎么重启⽤户都会把你打挂。
> * 处理缓存雪崩简单，在批量往Redis存数据的时候，把每个Key的失效时间都加个随机值就好了，这样可以保证数据不会在同⼀时间⼤⾯积失效。或者设置热点数据永远不过期，有更新操作就更新缓存就好了。
```java
setRedis（Key，value，time + Math.random() * 10000）；
```