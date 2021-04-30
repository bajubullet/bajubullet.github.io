This is a document describing sliding window implementation i recently did in python using redis.

## The Why?

We were facing issues with some of our customers spawning a lot of tasks in a very short duration of time. Since these customer knew they have spawned a lot of tasks they where ok waiting for them to complete. But other customers who have spawned only a few tasks where left waiting for the tasks from first customer to complete. This was happening because all of our tasks go through a single queue.

So to fix this issue we decided to use the priority queue implementation in rabbitmq to reduce the priority of the tasks from the customers spawning more tasks. This way we can allow tasks from other customers to execute before all of the tasks from high load customers are executed. We decided to use the **Sliding Window Algorithm** to generate the priority of the tasks for each customer.

## Sliding Window Algorithm

Sliding window algorthim tries to keep track of tasks in the last hour by tacking timestamp as the tasks are added to the queue. It periodically filters the tasks older than an hour and we can calculates tasks spawned in the last hour just by counting the timestamps present in the set.

## The implementation

We will be using python and redis to implement the above algorithm. Lets see the code for same:
```python
def sliding_window(key: str, size: int = 3600) -> int:
    redis_obj = redis.StrictRedis(get_sys_config("RedisServer"))
    now = time.time()
    sw_key = f"sw-{key}"
    pipe = redis_obj.pipeline()
    pipe.zremrangebyscore(sw_key, 0, now - size)
    pipe.zrange(sw_key, 0, -1)
    pipe.zadd(sw_key, {now: now})
    pipe.expire(sw_key, size)
    _, values, _, _ = pipe.execute()
    return len(values)
```

Here the function `sliding_window` does the following things:
- filters out entries that are older than 1 hr(3600 secs)
- add an entry to a sorted set in redis each time its called
- returns the existing count of entries in the sorted set, which is the number of tasks spawned in the last hour.

Lets breakdown each line in the function above to see what its actually doing.

```python
redis_obj = redis.StrictRedis(get_sys_config("RedisServer"))
```
This line simply creates a redis object to be used for performing upcoming actions.

```python
now = time.time()
```
Gets the current unix timestamp e.g. 1619795184.2651541

```python
sw_key = f"sw-{key}"
```
Creates a key to be used in redis. We will be creating a separate key for each of our customers to track the number of tasks they have added to the queue.

```python
pipe = redis_obj.pipeline()
```
This basically starts a transaction block in redis. Commands in a pipeline execute atomically and are buffered in a single request that can be used to dramatically increase the performance of groups of commands by reducing the number of back-and-forth TCP packets between the client and server. Read more about it [here](https://github.com/andymccurdy/redis-py/#pipelines).

```python
pipe.zremrangebyscore(sw_key, 0, now - size)
```
This removes all the entries older than the given interval which in our case is `size = 3600` seconds.

```python
pipe.zrange(sw_key, 0, -1)
```
This is used to get a list of all timestamps that happened during the window.

```python
pipe.zadd(sw_key, {now: now})
```
This adds a new entry in the sorted set for the task we are going to add.

```python
pipe.expire(sw_key, size)
```
This resets the expiry date of the sorted set to 3600 seconds in future.

```python
_, values, _, _ = pipe.execute()
```
This executes the whole pipeline and stores the output for `pipe.zrange(sw_key, 0, -1)` in values.

```python
return len(values)
```
This line just returns the count values that we added in last hour.

## Closing thoughts

This way we can make sure we only keep track of values in the last hour, discard older values and have the exact count of tasks added in last hour.
The only limitation I see with this implementation is if multiple calls are happening in same nanosecond for which we need a very performant system.
