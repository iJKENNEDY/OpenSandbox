# OpenSandbox Kotlin Redis Pool Store

Redis-backed `PoolStateStore` implementation for distributed `SandboxPool` coordination.

## Installation

Use this module only when multiple SDK processes need to share one sandbox pool through Redis.

```kotlin
dependencies {
    implementation("com.alibaba.opensandbox:sandbox:{latest_version}")
    implementation("com.alibaba.opensandbox:sandbox-pool-redis:{latest_version}")
}
```

## Usage

Create and configure the Jedis client yourself, then pass it to `RedisPoolStateStore`.
The store does not create, configure, or close the Redis client.

```java
import com.alibaba.opensandbox.sandbox.pool.SandboxPool;
import com.alibaba.opensandbox.sandbox.domain.pool.PoolCreationSpec;
import com.alibaba.opensandbox.sandbox.infrastructure.pool.RedisPoolStateStore;
import redis.clients.jedis.JedisPooled;

JedisPooled redis = new JedisPooled("redis://user:password@redis.example.com:6379/0");

RedisPoolStateStore store = new RedisPoolStateStore(
    redis,
    "opensandbox:pool:prod"
);

SandboxPool pool = SandboxPool.builder()
    .poolName("demo-pool")
    .ownerId("worker-1")
    .maxIdle(10)
    .stateStore(store)
    .connectionConfig(config)
    .creationSpec(
        PoolCreationSpec.builder()
            .image("ubuntu:22.04")
            .build()
    )
    .build();

try {
    pool.start();
    // acquire and use sandboxes
} finally {
    pool.shutdown(true);
    redis.close();
}
```

## Notes

- `RedisPoolStateStore` supports standalone Redis or Redis-compatible proxy endpoints.
- Redis Cluster and Redis Sentinel clients are not supported by this store.
- All nodes in the same logical pool must use the same `keyPrefix` and `poolName`.
- Each process must use a unique `ownerId`.
- Configure Redis connection details, TLS, ACL, timeout, pooling, and monitoring through Jedis.
- Redis outages are surfaced as `PoolStateStoreUnavailableException`; the pool does not silently bypass shared state.
