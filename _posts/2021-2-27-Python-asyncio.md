The new asyncio framework introduced in python3 is amazing. It allows us to run tasks concurrently in python similar to Golang and other languages that support
concurrency.

It still doesn't by-pass GIL but can allow you to do concurrent network operations which usually solves most of the problems.

To do this you need to create coroutines instead of functions which you can do by simply adding `async` before `def`ing a function like this:
```python
async def say_hello():
  print("Hello World!")
```

Simply executing this wont make it run async-ly. There are few more steps required like adding it to the loop and then awaiting it to complete.

If we try running this task it will simply return a coroutine instance
```
>>> async def say_hello():
...   print("Hello World!")
...
>>> say_hello()
<coroutine object say_hello at 0x1013d6840>
>>> 
```

To actually run this coroutine we need to pass the coroutine object that we get by executing `say_hello` to `asyncio.run`
```
>>> import asyncio
>>> asyncio.run(say_hello())
Hello World!
>>>
```

Ok now we know how to create and execute coroutines great, right? not so fast `asyncio.run` simply executes a coroutine it doesn't actually allow concurrency.
Let me explain, if we modify our `say_hello` function to `say_hello_after(n)` which waits for `n` secs before printing anything.

```python
import asyncio
import time


async def say_hello_after(n: int):
    await asyncio.sleep(n)
    print("Hello World!")


async def main():
    await say_hello_after(1)
    await say_hello_after(2)


print(f"started at: {time.strftime('%X')}")
asyncio.run(main())
print(f"finished at: {time.strftime('%X')}")
```

Ok we are introducing a few new things here first of all we are using `asyncio.sleep` instead of `time.sleep` to avoid CPU blocking(we have to use async functions
to do async programming). We are also adding an `await` next to it which actually sleep the current function until the thing it is waiting for is completed.

Enough explaining how do you think this program will behave. Will it take 2 secs to complete or 3 secs? Here is the output:

```
started at: 21:06:26
Hello World!
Hello World!
finished at: 21:06:29
```

The program is still taking 3 secs to complete, which is not good, no concurrency, this feels sequential.

We need to make one final adjustment we need to spawn the tasks in the event loop. which we can do by wrapping the coroutine calls in `asyncio.create_task` function
which actually spawns the task on the event loop and gives you a task object that can be awaited. Here is the final code:

```python
import asyncio
import time


async def say_hello_after(n: int):
    await asyncio.sleep(n)
    print("Hello World!")


async def main():
    t1 = asyncio.create_task(say_hello_after(1))
    t2 = asyncio.create_task(say_hello_after(2))

    await t1
    await t2


print(f"started at: {time.strftime('%X')}")
asyncio.run(main())
print(f"finished at: {time.strftime('%X')}")
```
Executing this we get:
```
started at: 21:11:55
Hello World!
Hello World!
finished at: 21:11:57
```

Here `asyncio.create_task` actually spawns the task on the event loop and the `await` calls in the `main()` function actually wait for it to complete.
We can actually skip awaiting for `t1` here since it will complete before `t2` as it only runs for 1 sec, but its always a good idea to wait for all the tasks 
you spawn.

More details on this can be found here: https://docs.python.org/3/library/asyncio.html

