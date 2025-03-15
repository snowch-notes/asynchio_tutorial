# **6. Synchronization Primitives in `asyncio`**  

In asynchronous programming, multiple tasks often share resources like files, network connections, or in-memory data structures. **Without proper synchronization, race conditions and data corruption can occur.**  

This chapter explores key **synchronization primitives** in `asyncio`:  
- **`asyncio.Lock`** – Prevents multiple tasks from accessing a resource simultaneously.  
- **`asyncio.Queue`** – Implements producer-consumer patterns.  
- **`asyncio.Event`, `asyncio.Condition`, and `asyncio.Semaphore`** – Manage signaling and concurrency limits.  

---

## **6.1 Using `asyncio.Lock` for Resource Protection**  

A **lock** ensures that only one task accesses a shared resource at a time.  

### **Example: Race Condition Without a Lock**  
In the following example, multiple tasks **increment a shared counter simultaneously**, causing incorrect results.

```python
import asyncio

counter = 0  # Shared resource

async def increment():
    global counter
    temp = counter
    await asyncio.sleep(0.1)  # Simulate processing time
    counter = temp + 1  # Race condition!

async def main():
    tasks = [increment() for _ in range(10)]
    await asyncio.gather(*tasks)
    print("Final counter value:", counter)  # Might not be 10!

asyncio.run(main())
```

💡 **Problem:** Due to concurrent access, `counter` may not be incremented correctly.  

---

### **Fixing with `asyncio.Lock`**  
We use **`async with lock:`** to ensure only one task modifies `counter` at a time.

```python
import asyncio

counter = 0  # Shared resource
lock = asyncio.Lock()

async def increment():
    global counter
    async with lock:  # Prevents race conditions
        temp = counter
        await asyncio.sleep(0.1)  # Simulate processing time
        counter = temp + 1

async def main():
    tasks = [increment() for _ in range(10)]
    await asyncio.gather(*tasks)
    print("Final counter value:", counter)  # Always 10!

asyncio.run(main())
```

✅ **Solution:** Using `asyncio.Lock`, the final value is always **10**.

---

## **6.2 `asyncio.Queue` for Producer-Consumer Patterns**  

A **queue** helps **producers** generate data and **consumers** process it efficiently.

### **Example: Producer-Consumer Using `asyncio.Queue`**  

```python
import asyncio

async def producer(queue):
    for i in range(5):
        await asyncio.sleep(1)  # Simulate work
        await queue.put(f"Task {i}")
        print(f"Produced: Task {i}")

async def consumer(queue):
    while True:
        task = await queue.get()
        print(f"Consumed: {task}")
        queue.task_done()  # Mark task as processed

async def main():
    queue = asyncio.Queue()
    producers = [asyncio.create_task(producer(queue))]
    consumers = [asyncio.create_task(consumer(queue)) for _ in range(2)]  # 2 consumers

    await asyncio.sleep(6)  # Allow tasks to be produced and consumed
    for c in consumers:
        c.cancel()  # Stop consumers after processing

asyncio.run(main())
```

✅ **Key Features:**  
- **Producers** add items to the queue.  
- **Consumers** process items.  
- **Queue ensures orderly processing** and prevents resource exhaustion.

---

## **6.3 `asyncio.Event`, `asyncio.Condition`, and `asyncio.Semaphore`**  

### **6.3.1 `asyncio.Event` – Task Synchronization**  
An **event** allows one task to signal another when a condition is met.

```python
import asyncio

event = asyncio.Event()

async def waiter():
    print("Waiting for event to be set...")
    await event.wait()  # Wait until event is set
    print("Event received!")

async def setter():
    await asyncio.sleep(2)
    print("Setting event!")
    event.set()  # Notify waiting task

async def main():
    await asyncio.gather(waiter(), setter())

asyncio.run(main())
```

✅ **Usage:** Useful for coordinating between tasks, such as waiting for a network connection.

---

### **6.3.2 `asyncio.Condition` – Advanced Synchronization**  
A **condition variable** allows multiple tasks to wait for a shared condition.

```python
import asyncio

condition = asyncio.Condition()
shared_resource = 0

async def waiter():
    async with condition:
        await condition.wait()  # Wait until condition is met
        print("Condition met! Resource:", shared_resource)

async def updater():
    global shared_resource
    await asyncio.sleep(2)
    shared_resource = 42
    async with condition:
        condition.notify_all()  # Wake up waiting tasks

async def main():
    await asyncio.gather(waiter(), updater())

asyncio.run(main())
```

✅ **Usage:** Useful when multiple consumers depend on a shared condition.

---

### **6.3.3 `asyncio.Semaphore` – Limiting Concurrency**  
A **semaphore** controls how many tasks can access a resource at the same time.

```python
import asyncio

semaphore = asyncio.Semaphore(2)  # Allow 2 tasks at a time

async def worker(id):
    async with semaphore:
        print(f"Task {id} is working...")
        await asyncio.sleep(2)
        print(f"Task {id} is done!")

async def main():
    tasks = [worker(i) for i in range(5)]
    await asyncio.gather(*tasks)

asyncio.run(main())
```

✅ **Usage:**  
- Prevents overloading databases or APIs by **limiting concurrent requests**.  
- Ensures that **only a fixed number of tasks** run at a time.  

---

## **6.4 Summary**  

| **Primitive**       | **Use Case** |
|---------------------|-------------|
| **`asyncio.Lock`** | Prevents race conditions on shared resources |
| **`asyncio.Queue`** | Implements producer-consumer patterns |
| **`asyncio.Event`** | Allows one task to signal another |
| **`asyncio.Condition`** | Allows multiple tasks to wait for a condition |
| **`asyncio.Semaphore`** | Limits concurrency (e.g., API requests) |

---

## **Next Steps**  
In the next chapter, we will explore **asynchronous networking with `asyncio`**, covering TCP/UDP servers and clients. 🚀
