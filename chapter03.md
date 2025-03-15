# **3. Core Concepts in `asyncio`**  

In this chapter, we’ll explore the fundamental concepts of `asyncio`, including **coroutines**, **awaitables**, **tasks**, and how to run multiple asynchronous functions concurrently. Understanding these concepts is essential for writing efficient async programs.  

---

## **3.1 `async` and `await` Keywords**  

The `async` and `await` keywords are the foundation of Python’s asynchronous programming model.  

- **`async def`** defines a coroutine function.  
- **`await`** pauses execution of a coroutine until the awaited operation completes.  

### **Example: Basic `async` and `await` Usage**  

```python
import asyncio

async def greet():
    print("Hello!")
    await asyncio.sleep(1)  # Simulating an async operation
    print("Goodbye!")

asyncio.run(greet())
```

### **What Happens Here?**  
1. `greet()` is an **async function**, meaning it returns a coroutine.  
2. `await asyncio.sleep(1)` **pauses execution** of `greet()`, allowing the event loop to run other tasks.  
3. After 1 second, execution resumes, and `"Goodbye!"` is printed.  

**Key Rule:** A function containing `await` **must be async**, or Python will raise a `SyntaxError`.  

---

## **3.2 Coroutines and Awaitables**  

### **What is a Coroutine?**  
A **coroutine** is a special function that can be paused and resumed. In Python, coroutines are defined with `async def`.  

```python
async def my_coroutine():
    return "I'm a coroutine!"
```

Calling an `async` function doesn’t execute it immediately. Instead, it returns a **coroutine object**, which must be awaited or run inside an event loop:  

```python
coro = my_coroutine()
print(coro)  # <coroutine object my_coroutine at 0x...>

asyncio.run(coro)  # Executes the coroutine
```

### **What is an Awaitable?**  
An **awaitable** is anything that can be used with `await`. There are three types:  
1. **Coroutines** (defined with `async def`)  
2. **Tasks** (created with `asyncio.create_task()`)  
3. **Futures** (low-level async objects for handling results)  

---

## **3.3 Tasks vs. Futures**  

### **Tasks: Running Coroutines Concurrently**  
A **Task** is a wrapper around a coroutine that runs it **concurrently** within the event loop.  

#### **Example: Running Two Tasks in Parallel**  

```python
import asyncio

async def task(name, delay):
    print(f"{name} starting...")
    await asyncio.sleep(delay)
    print(f"{name} finished!")

async def main():
    task1 = asyncio.create_task(task("Task 1", 2))
    task2 = asyncio.create_task(task("Task 2", 1))

    await task1  # Wait for Task 1 to complete
    await task2  # Wait for Task 2 to complete

asyncio.run(main())
```

### **How It Works:**  
- `asyncio.create_task(task())` starts each coroutine **immediately**.  
- `await task1` and `await task2` ensure we wait for both to complete.  

**Expected Output:**  
```
Task 1 starting...
Task 2 starting...
Task 2 finished!
Task 1 finished!
```

Since `Task 2` has a shorter delay, it completes first, even though `Task 1` started earlier.  

### **Futures: Low-Level Awaitables**  
A **Future** is an object representing an operation that **hasn’t completed yet**. While `asyncio` tasks are built on futures, you usually don’t need to create futures manually.  

For example, `asyncio.sleep()` returns a future that completes after a delay:  

```python
future = asyncio.sleep(2)
print(future)  # <Future pending>
```

---

## **3.4 Running Multiple Coroutines Concurrently**  

### **Using `asyncio.gather()`**  
`asyncio.gather()` runs multiple coroutines **concurrently** and returns their results as a list.  

```python
import asyncio

async def task(name, delay):
    await asyncio.sleep(delay)
    return f"{name} completed"

async def main():
    results = await asyncio.gather(
        task("Task 1", 2),
        task("Task 2", 1),
        task("Task 3", 3)
    )
    print(results)

asyncio.run(main())
```

**Expected Output:**  
```
['Task 1 completed', 'Task 2 completed', 'Task 3 completed']
```

**Key Benefits:**  
- Runs tasks **in parallel** instead of waiting for one to finish before starting another.  
- Returns **results in the same order** as the input arguments.  

### **Using `asyncio.wait()` (Alternative to `gather()`)**  
`asyncio.wait()` also runs multiple coroutines concurrently but returns a **set of completed tasks** instead of results.  

```python
import asyncio

async def task(name, delay):
    await asyncio.sleep(delay)
    print(f"{name} finished!")

async def main():
    tasks = {task("Task 1", 2), task("Task 2", 1)}
    done, pending = await asyncio.wait(tasks)

asyncio.run(main())
```

Use `asyncio.wait()` when you need finer control over completed and pending tasks.  

---

## **3.5 Handling Exceptions in `asyncio` Tasks**  

Errors inside `asyncio` coroutines **don’t propagate** unless awaited.  

### **Example: Handling Exceptions in `gather()`**  

```python
import asyncio

async def risky_task():
    await asyncio.sleep(1)
    raise ValueError("Something went wrong!")

async def safe_task():
    await asyncio.sleep(1)
    return "All good!"

async def main():
    try:
        results = await asyncio.gather(risky_task(), safe_task(), return_exceptions=True)
        print(results)
    except Exception as e:
        print(f"Exception caught: {e}")

asyncio.run(main())
```

**Output (with `return_exceptions=True`)**:  
```
[ValueError('Something went wrong!'), 'All good!']
```

If `return_exceptions=False`, the exception would be raised immediately instead.  

---

## **3.6 Summary**  

✅ **`async` and `await`** enable non-blocking execution.  
✅ **Coroutines** are functions that can be paused and resumed.  
✅ **Awaitables** include coroutines, tasks, and futures.  
✅ **`asyncio.create_task()`** allows coroutines to run concurrently.  
✅ **`asyncio.gather()`** collects results from multiple coroutines.  
✅ **Handle exceptions properly** when working with multiple tasks.  

---

## **Next Steps**  
Now that we understand how `asyncio` works under the hood, we’ll explore **task management**, including **cancelling tasks, handling timeouts, and scheduling recurring tasks** in the next chapter.
