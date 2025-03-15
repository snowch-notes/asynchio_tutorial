# **4. Managing Tasks in `asyncio`**  

Now that we understand how to create and run asynchronous tasks, this chapter explores how to **cancel tasks, handle timeouts, and schedule recurring tasks**.  

---

## **4.1 Cancelling Tasks**  

Sometimes, we need to cancel a running task before it completes. `asyncio` provides the `.cancel()` method to stop a task before it finishes.  

### **Example: Cancelling a Task**  

```python
import asyncio

async def long_task():
    try:
        print("Task started...")
        await asyncio.sleep(5)  # Simulating a long-running operation
        print("Task finished!")
    except asyncio.CancelledError:
        print("Task was cancelled!")

async def main():
    task = asyncio.create_task(long_task())

    await asyncio.sleep(2)  # Let the task run for 2 seconds
    task.cancel()  # Cancel the task

    try:
        await task  # Awaiting a cancelled task raises an exception
    except asyncio.CancelledError:
        print("Handled task cancellation.")

asyncio.run(main())
```

### **How It Works:**  
1. We create a long-running task with `asyncio.create_task(long_task())`.  
2. We let it run for **2 seconds** before calling `.cancel()`.  
3. Inside `long_task()`, the `CancelledError` exception is caught.  
4. If the task is cancelled, it doesn’t reach `"Task finished!"`.  

**Expected Output:**  
```
Task started...
Task was cancelled!
Handled task cancellation.
```

---

## **4.2 Handling Timeouts**  

We often need to set a **timeout** for tasks that might take too long. `asyncio.timeout()` (introduced in Python 3.11) makes this easy.  

### **Example: Using `asyncio.timeout()`** (Python 3.11+)  

```python
import asyncio

async def slow_operation():
    await asyncio.sleep(5)
    return "Completed"

async def main():
    try:
        async with asyncio.timeout(2):  # Set a 2-second timeout
            result = await slow_operation()
            print(result)
    except TimeoutError:
        print("Task timed out!")

asyncio.run(main())
```

**Expected Output:**  
```
Task timed out!
```

If using **Python <3.11**, use `asyncio.wait_for()`:  

```python
async def main():
    try:
        result = await asyncio.wait_for(slow_operation(), timeout=2)
        print(result)
    except asyncio.TimeoutError:
        print("Task timed out!")

asyncio.run(main())
```

### **How It Works:**  
- `asyncio.timeout(2)` cancels the task if it doesn’t complete within **2 seconds**.  
- `TimeoutError` is raised, which we handle in the `except` block.  

---

## **4.3 Waiting for Multiple Tasks with Timeouts**  

When working with multiple tasks, we may want to **cancel only the slowest ones** instead of failing everything.  

### **Example: Handling Partial Timeouts with `asyncio.wait()`**  

```python
import asyncio

async def task(name, delay):
    await asyncio.sleep(delay)
    print(f"{name} finished!")
    return name

async def main():
    tasks = {task("Task 1", 2), task("Task 2", 4), task("Task 3", 6)}

    done, pending = await asyncio.wait(tasks, timeout=3)

    print("Completed tasks:", [t.result() for t in done])
    for p in pending:
        p.cancel()  # Cancel remaining tasks
        print(f"Cancelled {p}")

asyncio.run(main())
```

### **How It Works:**  
- `asyncio.wait(tasks, timeout=3)` waits **up to 3 seconds**.  
- It returns two sets:  
  - **`done`** (completed tasks)  
  - **`pending`** (still running tasks)  
- We manually cancel the **pending tasks**.  

**Expected Output:**  
```
Task 1 finished!
Completed tasks: ['Task 1']
Cancelled <Task pending...>
Cancelled <Task pending...>
```

---

## **4.4 Scheduling Recurring Tasks**  

Some tasks should run **at regular intervals**, like polling an API or checking a sensor.  

### **Example: Running a Periodic Task**  

```python
import asyncio

async def periodic_task():
    while True:
        print("Task executed!")
        await asyncio.sleep(2)  # Run every 2 seconds

async def main():
    task = asyncio.create_task(periodic_task())

    await asyncio.sleep(7)  # Let it run for 7 seconds
    task.cancel()
    print("Periodic task stopped.")

asyncio.run(main())
```

**Expected Output:**  
```
Task executed!
Task executed!
Task executed!
Periodic task stopped.
```

### **Stopping a Periodic Task Gracefully**  

Instead of `.cancel()`, we can use an **event** to stop the task cleanly:  

```python
import asyncio

stop_event = asyncio.Event()

async def periodic_task():
    while not stop_event.is_set():
        print("Task executed!")
        await asyncio.sleep(2)

async def main():
    task = asyncio.create_task(periodic_task())

    await asyncio.sleep(7)
    stop_event.set()  # Signal the task to stop
    await task  # Ensure cleanup

asyncio.run(main())
```

**Why use an event?**  
- It **avoids `CancelledError` exceptions**.  
- The task can perform cleanup before stopping.  

---

## **4.5 Running Background Tasks**  

Sometimes, we want a background task to run independently while the main program continues.  

### **Example: Background Task with `asyncio.create_task()`**  

```python
import asyncio

async def background_task():
    while True:
        print("Background task running...")
        await asyncio.sleep(3)

async def main():
    asyncio.create_task(background_task())  # Starts the background task
    await asyncio.sleep(10)  # Main task runs independently
    print("Main task done.")

asyncio.run(main())
```

**Expected Output:**  
```
Background task running...
Background task running...
Background task running...
Main task done.
```

Since the background task runs indefinitely, it outlives the main function.  

---

## **4.6 Summary**  

✅ **Cancelling Tasks:** `.cancel()` stops a running task.  
✅ **Timeouts:** Use `asyncio.timeout()` (Python 3.11+) or `asyncio.wait_for()`.  
✅ **Waiting for Multiple Tasks:** `asyncio.wait()` lets us handle timeouts while keeping completed tasks.  
✅ **Recurring Tasks:** Use a loop with `await asyncio.sleep()` for periodic execution.  
✅ **Background Tasks:** `asyncio.create_task()` runs tasks in the background.  

---

## **Next Steps**  
In the next chapter, we’ll explore **asynchronous I/O operations**, including working with files, network requests, and databases using `asyncio`.
