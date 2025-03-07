---
title: Redisson分布式锁
tags:
  - Java
  - Redis
categories:
  - 分布式
top_img: 'https://haowallpaper.com/link/common/file/previewFileImg/15918089447001472'
cover: 'https://cdn.jsdelivr.net/gh/KNeegcyao/picdemo/img/image-20241207152648265.png'
abbrlink: 41a6994d
date: 2024-12-05 22:18:59
---

# 概览

<img src="https://cdn.jsdelivr.net/gh/KNeegcyao/picdemo/img/image-20241205222651622.png" alt="image-20241205222651622" style="zoom: 50%;" />

Redisson不只是一个 Java Redis 客户端，它是一个以内存 Redis 服务器作为后端的处理 Java 对象(如 `java.util.List`, `java.util.Map`, `java.util.Set`, `java.util.concurrent.locks.Lock` 等)的一个框架。 

 Redisson提供了使用Redis的最简单和最便捷的方法。Redisson的宗旨是促进使用者对Redis的关注分离（Separation of Concern），从而让使用者能够将精力更集中地放在处理业务逻辑上,Redisson底层采用的是Netty框架。

[基于 `SETNX` 实现的分布式锁](https://kneegcyao.github.io/2024/12/03/%E9%BB%91%E9%A9%AC%E7%82%B9%E8%AF%84/#:~:text=%E5%9F%BA%E4%BA%8E%20setnx%20%E5%AE%9E%E7%8E%B0,%E5%AE%9E%E4%BE%8B%E4%BA%89%E6%8A%A2%E3%80%82"黑马点评中的应用解释")适用于简单场景，但在高可靠性、高并发的生产环境中，建议使用更完善的分布式锁实现。

Redisson实现了Redis文档中提到像分布式锁Lock这样的更高阶应用场景。事实上Redisson并没有不止步于此，在分布式锁的基础上还提供了联锁（MultiLock），读写锁（ReadWriteLock），公平锁（Fair Lock），红锁（RedLock），信号量（Semaphore），可过期性信号量（PermitExpirableSemaphore）和闭锁（CountDownLatch）这些实际当中对多线程高并发应用至关重要的基本部件。正是通过实现基于Redis的高阶应用方案，使Redisson成为构建分布式系统的重要工具。

# 配置

1.引入依赖

```xml
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson-spring-boot-starter</artifactId>
    <version>3.23.5</version>
</dependency>
```

2.配置属性

```java
@Configuration
public class RedissonConfig {
    @Bean
    public RedissonClient redissonClient(){
        //配置
        Config config = new Config();
        config.useSingleServer().setAddress("redis://xxx.xxx.xx.xxx:xxxx").setPassword("xxxx");
        return Redisson.create(config);
    }
}

```

3.使用测试

```java
@Resource
private RedissonClient redissonClient;

@Test
void testRedisson() throws InterruptedException {
    //获取可重入锁
    RLock lock = redissonClient.getLock("anyLock");
    //尝试获取锁，三个参数分别是：获取锁的最大等待时间(期间会重试)，锁的自动释放时间，时间单位
    boolean success = lock.tryLock(1,10, TimeUnit.SECONDS);
    //判断获取锁成功
    if (success) {
        try {
            System.out.println("执行业务");
        } finally {
            //释放锁
            lock.unlock();
        }
    }
}
```

# Redisson分布式原理

## 可重入原理

<img src="https://cdn.jsdelivr.net/gh/KNeegcyao/picdemo/img/6tltgdnq5kf3m_d531681722e140368fd43583a3d6226f.png" alt="img" style="zoom: 50%;" />

利用redis的hash结构存储锁，key值随意，field属性为线程标识，value为锁次数。当线程获取一次锁后，如果此时redis中没有这个锁，则创建并将锁次数置为1；接下来如果线程再次获取锁会进行一次判断。即对比线程标识是否是同一个线程的多次获取，如果是的话锁次数+1。同样的，如果是释放锁的话也需要对线程标识进行判断，然后让对应的锁次数-1，当锁的次数为0时，表示此时可以删除锁了。

<img src="https://cdn.jsdelivr.net/gh/KNeegcyao/picdemo/img/image-20241205233234744.png" alt="image-20241205233234744" style="zoom: 33%;" />

## 锁可重试和超时续约原理

![image-20241206235133484](https://cdn.jsdelivr.net/gh/KNeegcyao/picdemo/img/image-20241206235133484.png)

![image-20241206235233851](https://cdn.jsdelivr.net/gh/KNeegcyao/picdemo/img/image-20241206235233851.png)

- `leaseTime` 是锁的租约时间，即锁的有效期。在锁被成功获取后，若没有手动释放锁，则锁会在 `leaseTime` 时间后自动失效。
  - 如果 `leaseTime` 设置为 `-1`，表示锁的租约时间是无限期的，这种情况下需要使用 **WatchDog** 来自动续约。
  - 如果 `leaseTime` 是一个具体的时间值（例如 30 秒），锁会在这个时间后自动失效，无需续约。

- `ttl`表示锁当前的剩余有效期（Time-To-Live）。在尝试获取锁时，Redisson 会检查目标锁的 

  - **`ttl == null`**：目标锁不存在，可以尝试获取锁。

  - **`ttl != null`**：目标锁已经存在，需要判断剩余的等待时间是否超过指定阈值或是否可以订阅释放信号。

- `watchDog` 是 Redisson 内部的锁自动续约机制（看门狗）。当锁的 `leaseTime` 设置为 `-1` 时，`watchDog` 会定期自动延长锁的有效期，确保锁在业务逻辑执行过程中不会因超时而被意外释放。
  - 在锁成功获取后，`watchDog` 会启动一个后台线程，默认每隔 30 秒自动将锁的有效期续约为 30 秒。
  - 一旦锁被手动释放或业务逻辑执行完毕，`watchDog` 会停止工作。

如果我们自己设置的leaseTime则没有`开门狗`,没设置才会开启`开门狗`不停更新你的有效期。订阅别的新程释放锁的消息，可以重试。

释放锁成功时，发送释放锁信息 ，让其他获取锁失败的得到你释放锁成功的消息，再次争抢这把锁，然后取消``开门狗`避免无休止运行。

## 锁主从一致性原理

<img src="https://cdn.jsdelivr.net/gh/KNeegcyao/picdemo/img/image-20241207145452802.png" alt="image-20241207145452802" style="zoom:67%;" />

运用Redisson的联锁multiLock，`将多个 RLock 对象关联为一个联锁，每个 RLock 对象实例可以来自于不同的 Redisson 实例`

MultiLock 的底层实现基于 Redisson 的分布式锁实现。当我们对一个 MultiLock 进行加锁时，Redisson 会在 Redis 中为每个锁创建一个键，并将锁的状态设置为已加锁。当我们对一个 MultiLock 进行解锁时，Redisson 会将所有锁的状态设置为已解锁。

```java
import org.redisson.api.RLock;
import org.redisson.api.RMultiLock;
import org.redisson.api.RedissonClient;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class MultiLockService {

    @Autowired
    private RedissonClient redissonClient;

    public void performMultiLockOperation() {
        // 获取多个锁
        RLock lock1 = redissonClient.getLock("lock1");
        RLock lock2 = redissonClient.getLock("lock2");
        RLock lock3 = redissonClient.getLock("lock3");

        // 创建 MultiLock
        RMultiLock multiLock = redissonClient.getMultiLock(lock1, lock2, lock3);

        try {
            // 尝试加锁，最多等待 10 秒，锁定后 30 秒自动释放
            boolean isLocked = multiLock.tryLock(10, 30, java.util.concurrent.TimeUnit.SECONDS);
            if (isLocked) {
                try {
                    // 执行业务逻辑
                    System.out.println("所有锁已获取，正在执行操作...");
                } finally {
                    // 释放锁
                    multiLock.unlock();
                    System.out.println("所有锁已释放");
                }
            } else {
                System.out.println("无法获取所有锁");
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
            Thread.currentThread().interrupt();
        }
    }
}

```

#### **1. 加锁过程**

当调用 `multiLock.lock()` 或 `multiLock.tryLock()` 时：

1. **逐个获取锁**：
   - 遍历 `RLock` 列表，对每个锁执行加锁操作。
   - Redis 内部使用 `SETNX` 命令（设置值，如果不存在则创建）实现锁的占用。
2. **所有锁获取成功**：
   - 如果所有锁都成功获取，则整个 `RMultiLock` 加锁成功。
3. **部分锁获取失败**：
   - 如果任何一个锁获取失败，则释放已成功获取的锁（回滚），加锁失败。
4. **Lua 脚本**：
   - Redisson 使用 Lua 脚本保证多个锁的加锁操作具有原子性，防止网络延迟或进程意外终止导致锁状态不一致。

------

#### **2. 解锁过程**

当调用 `multiLock.unlock()` 时：

1. **依次释放锁**：
   - 遍历 `RLock` 列表，对每个锁执行解锁操作。
   - 解锁通过 Redis 的 `DEL` 或 Lua 脚本来完成，确保只释放由当前客户端持有的锁。
2. **解锁的幂等性**：
   - 即使某些锁已经超时或被释放，其他锁的解锁操作仍会继续，确保逻辑上的完整性。

------

#### **3. 超时机制**

1. **锁的持有时间**：
   - 每个 `RLock` 在加锁时可以设置一个过期时间（TTL），防止死锁。
   - 如果锁的持有线程在 TTL 内未能主动释放锁，Redis 会自动释放该锁。
2. **续约机制**：
   - Redisson 提供了 WatchDog 自动续约机制，确保在锁的持有线程存活的情况下，锁不会意外释放。
