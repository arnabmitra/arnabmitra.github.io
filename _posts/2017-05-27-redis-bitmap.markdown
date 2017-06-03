---
layout: post
title:  "Analytics using Redis data structure:Bitmap."
categories: "redis,nosql,analytics"
date:   2017-05-27 20:52:29 -0700
categories: jekyll update
---
Chances are, if you have ever worked on a highly trafficked website, you may have a need
for real time metrics.Whether it be counting the number of times all users have clicked a button
or figuring out how many distinct actions an user has performed.

The basis of my post today is not original but is based on this article from way back in time
[fast-easy-realtime-metrics-using-redis-bitmaps](http://blog.getspool.com/2011/11/29/fast-easy-realtime-metrics-using-redis-bitmaps/)

## Theoretical scenario

Lets say you run an e-commerce website and you want to

- See how many users have clicked a certain button or performed a certain action.
Example actions, could be how many times has someone performed the
internal search action(clicked the search button) on the site.
How many times have customers clicked add to cart button in the last hour and such.

- Track if a customer has performed a certain set of actions,
 for e.g if they clicked on the
search button, gone to a certain page, gone to the home page etc.
- Track how many users have performed a particular action or a set of actions.


Redis bitmaps are super simple way to track user events.

Redis bitmaps are not a special datatype in REDIS but is a String datatype in redis which has its value set
by the SETBIT [fSETBIT](https://redis.io/commands/setbit) operation.
This value can be retrieved by the [GETBIT](https://redis.io/commands/getbit) operation.

Examples
```
redis> SETBIT mykey 7 1
(integer) 0
redis> GETBIT mykey 0
(integer) 0
redis> GETBIT mykey 7
(integer) 1
redis> GETBIT mykey 100
(integer) 0
redis>
```
Bitmaps are a collection of bits 1 representing an action that has been
taken by the customer and 0 representing no action, with the offset as
the customer id

To set the bit value for a customer using the very popular java client Jedis https://github.com/xetorthio/jedis
use the code below.

```
 private void setBit( final String key, final long offset, final boolean value ) {
        redisTemplate.execute((RedisCallback<Void>) connection -> {
          connection.setBit( ( ( RedisSerializer< String > )redisTemplate.getKeySerializer() ).serialize( key ), offset, value );
          return null;
        });
      }
```

To get a customer action i.e see if a customer has performed a given action, use the code below
```
 private boolean getBit( final String key, final long offset ) {
    return redisTemplate.execute(
        new RedisCallback< Boolean >() {
          @Override
          public Boolean doInRedis( RedisConnection connection ) throws DataAccessException {
            return connection.getBit( ( (RedisSerializer< String >)redisTemplate.getKeySerializer() ).serialize( key ), offset );
          }
        }
    );
  }
```

To get an associated bit with every customer action
```
return getBit(userKey,customerId);
```
To set an associated bit with every customer action
```
 setBit( userKey, customerId, true );
```

To get a bitcount i.e the number of customers who have performed an action
```
   return redisTemplate.execute(new RedisCallback<Long>() {
         @Override public Long doInRedis(RedisConnection connection) throws DataAccessException {
           return connection.bitCount(( ( RedisSerializer< String > )redisTemplate.getKeySerializer() ).serialize(userKey));
         }
       });
```
To get cardinality across different bitmaps we get the bitset for each user action(i.e to get the count of users who have performed all the actions),we
get the cardibality across each action and it super simple
```
  BitSet  initial=getBitSet(userKey[0]);
       for (String userKeyStr:userKey)
      {
         initial.and(getBitSet(userKeyStr));
      }
      return initial.cardinality();
```
The below code shows the full Redis repository code which implements the below methods
```
 /**
   * Sets particular action for a give customer id.
   * @param userKey
   * @param customerId
   */
  void setUserAction(String userKey, long customerId);

  /**
   * Figure out if a customer has actually performed an action.
   * @param userKey
   * @param customerId
   * @return
   */
  boolean isUserAction(String userKey, Long customerId);

  /**
   * Get the number of users who have performed a particular action.
   * @param userKey
   * @return
   */
  long getNumberOfUserClicksPerAction(String userKey);

  /**
   * Get the number of users who have performed a set of particular action.
   * @param userKey
   * @return
   */
  long getNumberOfUserClicksPerAction(String... userKey);

```

```
@Repository
   public class RedisRepositoryImpl implements RedisRepository {

     private final RedisTemplate<String,Boolean> redisTemplate;

     @Inject
     public RedisRepositoryImpl(RedisTemplate redisTemplate) {
       this.redisTemplate = redisTemplate;
     }


     @Override
     public boolean isUserAction(String userKey,Long customerId) {
       return getBit(userKey,customerId);
     }

     @Override
     public void setUserAction(String userKey, long customerId) {
       setBit( userKey, customerId, true );
     }

     private boolean getBit( final String key, final long offset ) {
       return redisTemplate.execute(
           new RedisCallback< Boolean >() {
             @Override
             public Boolean doInRedis( RedisConnection connection ) throws DataAccessException {
               return connection.getBit( ( (RedisSerializer< String >)redisTemplate.getKeySerializer() ).serialize( key ), offset );
             }
           }
       );
     }

     @Override public long getNumberOfUserClicksPerAction(String userKey) {
       return redisTemplate.execute(new RedisCallback<Long>() {
         @Override public Long doInRedis(RedisConnection connection) throws DataAccessException {
           return connection.bitCount(( ( RedisSerializer< String > )redisTemplate.getKeySerializer() ).serialize(userKey));
         }
       });
     }

     @Override public long getNumberOfUserClicksPerAction(String... userKey) {
       BitSet  initial=getBitSet(userKey[0]);
       for (String userKeyStr:userKey)
      {
         initial.and(getBitSet(userKeyStr));
      }
      return initial.cardinality();
     }

     private BitSet getBitSet(String userKeyStr) {
       return redisTemplate.execute(new RedisCallback<BitSet>() {
         @Override public BitSet doInRedis(RedisConnection connection) throws DataAccessException {
           return BitSet.valueOf(connection.get(( (RedisSerializer< String >)redisTemplate.getKeySerializer() ).serialize(userKeyStr)));
         }
       });
     }

     private void setBit( final String key, final long offset, final boolean value ) {
       redisTemplate.execute((RedisCallback<Void>) connection -> {
         connection.setBit( ( ( RedisSerializer< String > )redisTemplate.getKeySerializer() ).serialize( key ), offset, value );
         return null;
       });
     }
   }

```

The code for all the above can be found at:
https://github.com/arnabmitra/redis-bitmap
This is a simple Spring boot application starting up the webserver on port 8080.

This code run against a local docker cluster, to start up this local docker cluster download
this github repo
https://github.com/arnabmitra/docker-redis-cluster.git

and run
```
docker-compose -f docker-compose.yml build

 docker-compose -f docker-compose.yml up
```
This will start up a redis cluster with the following details
```
Using 3 masters:
redis-cluster_1  | 127.0.0.1:7000
redis-cluster_1  | 127.0.0.1:7001
redis-cluster_1  | 127.0.0.1:7002
redis-cluster_1  | Adding replica 127.0.0.1:7003 to 127.0.0.1:7000
redis-cluster_1  | Adding replica 127.0.0.1:7004 to 127.0.0.1:7001
redis-cluster_1  | Adding replica 127.0.0.1:7005 to 127.0.0.1:7002
redis-cluster_1  | M: eedadbfd24d9453cd96153221cd52c1be238af68 127.0.0.1:7000
redis-cluster_1  |    slots:0-5460 (5461 slots) master
redis-cluster_1  | M: 709b198594553bda7c65ab0f0f6b18a1953bb379 127.0.0.1:7001
redis-cluster_1  |    slots:5461-10922 (5462 slots) master
redis-cluster_1  | M: ecf12deb2fb1eddb9f858a02d1a358121ffb5b37 127.0.0.1:7002
redis-cluster_1  |    slots:10923-16383 (5461 slots) master
redis-cluster_1  | S: b1dcf2fd327ac0f34121580f2120239a9dbc5b63 127.0.0.1:7003
redis-cluster_1  |    replicates eedadbfd24d9453cd96153221cd52c1be238af68
redis-cluster_1  | S: addb068819513b1e9601e48559a733d26d1ece29 127.0.0.1:7004
redis-cluster_1  |    replicates 709b198594553bda7c65ab0f0f6b18a1953bb379
redis-cluster_1  | S: 43f0d69cd483a00b2a9fe62e5f549f9beeba9665 127.0.0.1:7005
redis-cluster_1  |    replicates ecf12deb2fb1eddb9f858a02d1a358121ffb5b37
```


Lets write a test, to see how all this ties in

```
  @Test public void setCustomerAction() {
    for (int i = 10; i < 50; i++) {
      customerService.setUserAction("user:clickProduct", i);
    }
    for (int i = 10; i < 25; i++) {
      customerService.setUserAction("user:clickSearch", i);
    }
    customerService.setUserAction("user:clickProduct", 20000);

  }

  @Test public void getCustomerAction() {
    for (int i = 10; i < 50; i++) {
      boolean getUserAction = customerService.isUserAction("user:clickProduct", Long.valueOf(i));
      assertThat(getUserAction).isEqualTo(true);
    }
    long numberOfUserActions = customerService.getNumberOfUserClicksAllAction("user:clickProduct", "user:clickSearch");
    assertThat(numberOfUserActions).isEqualTo(15);
    boolean getUserAction = customerService.isUserAction("user:clickProduct", 51110l);
    assertThat(getUserAction).isEqualTo(false);
  }
```

If you connect to a redis instance in the cluster using redis-cli
```
redis-cli -h 127.0.0.1 -p 7001
```
and watch the instance using the MONITOR command, you will see the following output.
```
1496100681.933094 [0 172.19.0.1:39616] "SETBIT" "\xac\xed\x00\x05t\x00\x11user:clickProduct" "10" "1"
1496100681.936218 [0 172.19.0.1:39616] "SETBIT" "\xac\xed\x00\x05t\x00\x11user:clickProduct" "11" "1"
1496100681.937057 [0 172.19.0.1:39616] "SETBIT" "\xac\xed\x00\x05t\x00\x11user:clickProduct" "12" "1"
1496100681.937744 [0 172.19.0.1:39616] "SETBIT" "\xac\xed\x00\x05t\x00\x11user:clickProduct" "13" "1"
1496100681.938371 [0 172.19.0.1:39616] "SETBIT" "\xac\xed\x00\x05t\x00\x11user:clickProduct" "14" "1"
```

```
1496100682.008025 [0 172.19.0.1:39616] "GETBIT" "\xac\xed\x00\x05t\x00\x11user:clickProduct" "10"
1496100682.057913 [0 172.19.0.1:39616] "GETBIT" "\xac\xed\x00\x05t\x00\x11user:clickProduct" "11"
1496100682.060131 [0 172.19.0.1:39616] "GETBIT" "\xac\xed\x00\x05t\x00\x11user:clickProduct" "12"
1496100682.061615 [0 172.19.0.1:39616] "GETBIT" "\xac\xed\x00\x05t\x00\x11user:clickProduct" "13"
1496100682.062810 [0 172.19.0.1:39616] "GETBIT" "\xac\xed\x00\x05t\x00\x11user:clickProduct" "14"
```
