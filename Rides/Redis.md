* **Redis的应用场景**
    * [1.库存超卖](#redis库存超卖场景)


## 1.库存超卖

### Q：为什么要使用redis去解决高并发下的库存超卖。

#### A：因为redis的一个特性：**高并发**。缓存是走内存的，内存天然就支撑高并发。 


### 场景分析
假定有一个商城购物的场景，之前客流量都很小，每秒并发无非是几十。突然有天，搞了一次火爆活动，一下子涌入大量的客源，最多会有1万人去抢购下单，这时候因为之前的设计，没有使用上缓存，这1万多请求，都是直接访问到数据库层面。 
如果说只是读数据，1万多请求靠加机器能勉强撑下来，但涉及到库存等先读后写的操作，肯定是不行的，尤其在分布式系统中，为了保证库存的准确性，一般会使用分布式锁，去对一些不允许出现差错的数据上锁，让他们串性执行保证数据的原子性。但这样会有一个问题，在高并非的情况下，会存在很严重的锁竞争问题，如果请求在等待的过程中竞争不到锁，就会出现获取锁超时的情况，如果没有后续的补偿逻辑，那么这次请求就相当于失败了。对用户的体验也不是很好，这样是我们不希望出现的。
所以我们为了减少对数据库的访问，就引入了缓存去做高并发的处理。


### 代码实现

    private static final String GOOD_STOCK = "good:stock:key_:%s";
     
    /**
     * 对redis中库存进行校验并扣减
     */
    private void checkGoodStock(GoodReq goodOrderReq) {
        // 库存key
        final String stockKey = Good.stockKey(goodOrderReq.getGoodId());
        String value = redisTemplate.opsForValue().get(stockKey);
        if (ObjectUtils.isEmpty(value)) {
            // 如果缓存中没有库存数据，则进行数据装载
            setGoodStockToRedis(goodOrderReq);
        }
        // 进行rides预减库存
        long remainingStock = redisTemplate.opsForValue().increment(stockKey, -1L);
        // 如果减1库存之后，结果小于0，则进行逻辑判断
        if (remainingStock < 0) {
            Integer redisStock = Integer.valueOf(redisTemplate.opsForValue().get(stockKey));
            if (ObjectUtils.isEmpty(redisStock)) {
                // 如果缓存中没有库存数据，则进行数据装载为0，存储
                redisTemplate.opsForValue().set(stockKey, String.valueOf(0), 5, TimeUnit.MINUTES);
            } 
            if (Long.parseLong(redisTemplate.opsForValue().get(stockKey)) <= 0L) {
                System.out.println("提示库存不足");
            }
        }
    }
    /**
     * 
     * 将库存放入缓存中
     *
     */
    private void setGoodStockToRedis(GoodReq goodOrderReq) {
        // 库存key
        final String stockKey = Good.stockKey(goodOrderReq.getGoodId());
        // 先去缓存中查询库存是否存在，如果不存在，则进行初始化
        if (ObjectUtils.isEmpty(redisTemplate.opsForValue().get(stockKey))) {
            lockHandler.tryLock("good:lock:key_:%s"+goodOrderReq.getGoodId(), () -> {
                // 先去缓存中查询库存是否存在，如果不存在，则进行初始化
                if (ObjectUtils.isEmpty(redisTemplate.opsForValue().get(stockKey))) {
                    Good good = findByGoodId(goodOrderReq.getGoodId());
                    redisTemplate.opsForValue().set(stockKey, good.getStock().toString(), 5, TimeUnit.MINUTES);
                }
            });
        }
    }

