# **7. Structuring and Debugging Async Code**  

Writing clean and efficient asynchronous code is crucial for maintaining readability, performance, and ease of debugging. In this chapter, we'll cover:  

- **Best practices** for writing clean `asyncio` code.  
- **Debugging techniques**, including inspecting running tasks.  
- **Exploring alternative async libraries** like `trio` and `anyio`.  

---

## **7.1 Best Practices for Writing Clean Async Code**  

### ✅ **1. Use `async` and `await` Properly**  
Avoid mixing **blocking** and **non-blocking** calls.  

❌ **Bad Example (Blocking I/O in Async Code)**  
```python
import asyncio
import time  # Blocking call!

async def task():
    print("Starting task...")
    time.sleep(2)  # BAD! This blocks the entire event loop
    print("Task done!")

async def main():
    await asyncio.gather(task(), task())

asyncio.run(main())  # This will freeze for 2s before executing the second task!
```

✅ **Good Example (Using `await asyncio.sleep()`)**  
```python
import asyncio

async def task():
    print("Starting task...")
    await asyncio.sleep(2)  # Non-blocking
    print("Task done!")

async def main():
    await asyncio.gather(task(), task())  # Both tasks run in parallel

asyncio.run(main())  # Executes efficiently
```

---

### ✅ **2. Use `asyncio.create_task()` for Fire-and-Forget Tasks**  
If a task should run in the background without blocking, use `asyncio.create_task()`.  

```python
import asyncio

async def background_task():
    while True:
        print("Background task running...")
        await asyncio.sleep(1)

async def main():
    asyncio.create_task(background_task())  # Runs independently
    await asyncio.sleep(5)  # Main program continues

asyncio.run(main())  # Runs for 5s, background task continues
```

---

### ✅ **3. Use `asyncio.Queue` Instead of Global Variables**  
Avoid sharing global variables between async tasks. Instead, use `asyncio.Queue`.  

```python
import asyncio

async def producer(queue):
    for i in range(5):
        await queue.put(f"Task {i}")
        print(f"Produced: Task {i}")
        await asyncio.sleep(1)

async def consumer(queue):
    while True:
        item = await queue.get()
        print(f"Consumed: {item}")
        queue.task_done()

async def main():
    queue = asyncio.Queue()
    asyncio.create_task(consumer(queue))
    await producer(queue)

asyncio.run(main())
```

---

### ✅ **4. Use Timeouts to Prevent Stalling**  
Set timeouts for slow operations to avoid indefinitely hanging code.  

```python
import asyncio

async def slow_task():
    await asyncio.sleep(10)  # Simulating a slow operation
    return "Task completed"

async def main():
    try:
        result = await asyncio.wait_for(slow_task(), timeout=5)  # Timeout after 5s
        print(result)
    except asyncio.TimeoutError:
        print("Task timed out!")

asyncio.run(main())  # Prints "Task timed out!"
```

---

## **7.2 Debugging with `asyncio.all_tasks()` and `asyncio.get_running_loop()`**  

### **Inspecting Running Async Tasks**  
Use `asyncio.all_tasks()` to find and debug hanging tasks.

```python
import asyncio

async def slow_task():
    await asyncio.sleep(10)

async def main():
    task1 = asyncio.create_task(slow_task())
    task2 = asyncio.create_task(slow_task())

    await asyncio.sleep(2)  # Wait before checking active tasks
    tasks = asyncio.all_tasks()
    print(f"Active tasks: {len(tasks)}")
    for task in tasks:
        print(f"Task: {task}")

asyncio.run(main())
```

✅ **Use Case:** Find stuck or unfinished tasks in large async programs.

---

### **Getting the Current Event Loop**  
Use `asyncio.get_running_loop()` to inspect the loop and its state.

```python
import asyncio

async def check_event_loop():
    loop = asyncio.get_running_loop()
    print(f"Current event loop: {loop}")

asyncio.run(check_event_loop())
```

✅ **Use Case:** Ensures async code is running in the correct event loop.

---

## **7.3 Using `trio` and `anyio` as Alternatives**  

### **Why Consider Alternatives?**  
While `asyncio` is built into Python, **`trio`** and **`anyio`** offer:  
- A **simpler, safer** API.  
- **Better structured concurrency** (automatic cleanup of background tasks).  
- **More predictable error handling**.  

---

### **Using `trio` for Asynchronous Tasks**  
Trio provides structured concurrency for managing async tasks efficiently.

```python
import trio

async def task(name):
    print(f"{name} started")
    await trio.sleep(2)
    print(f"{name} finished")

async def main():
    async with trio.open_nursery() as nursery:
        nursery.start_soon(task, "Task 1")
        nursery.start_soon(task, "Task 2")

trio.run(main)
```

✅ **Key Features:**  
- Uses **nurseries** for automatic cleanup of tasks.  
- More predictable than `asyncio.create_task()`.  

---

### **Using `anyio` for Async Compatibility**  
`anyio` allows switching between `asyncio` and `trio` seamlessly.

```python
import anyio

async def task():
    print("Task running")
    await anyio.sleep(1)
    print("Task done")

async def main():
    async with anyio.create_task_group() as tg:
        tg.start_soon(task)
        tg.start_soon(task)

anyio.run(main)
```

✅ **Key Features:**  
- Unified API for both `asyncio` and `trio`.  
- Works across different async frameworks.  

---

## **7.4 Summary**  

| **Topic** | **Key Takeaways** |
|-----------|------------------|
| **Best Practices** | Avoid blocking I/O, use `create_task()`, queues, and timeouts |
| **Debugging** | Use `asyncio.all_tasks()` and `asyncio.get_running_loop()` |
| **Trio** | Safer structured concurrency with nurseries |
| **Anyio** | Unified async API for both `asyncio` and `trio` |

---

## **Next Steps**  
In the next chapter, we will explore **real-world applications** of `asyncio`, such as building web scrapers and network clients. 🚀
