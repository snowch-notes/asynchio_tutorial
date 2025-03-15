# **2. Getting Started with `asyncio`**  

In this chapter, we'll set up `asyncio`, run our first asynchronous program, and understand how the **event loop** works.  

---

## **2.1 Installing and Setting Up `asyncio`**  

`asyncio` is included in Python's standard library (Python 3.4+), so no installation is required. To check your Python version, run:  

```sh
python --version
```  

For the latest features, ensure you're using **Python 3.7+** (since `asyncio.run()` was introduced in Python 3.7).  

If you need to install or upgrade Python:  

- **Ubuntu/Linux**:  
  ```sh
  sudo apt update && sudo apt install python3
  ```
- **macOS** (via Homebrew):  
  ```sh
  brew install python
  ```
- **Windows**: Download from [python.org](https://www.python.org/downloads/) and follow the installation instructions.  

Once Python is set up, you’re ready to write your first `asyncio` program.  

---

## **2.2 Running Your First `asyncio` Program**  

Let’s write a simple asynchronous function and execute it using `asyncio.run()`.  

```python
import asyncio

async def greet():
    print("Hello!")
    await asyncio.sleep(1)  # Simulates an asynchronous delay
    print("Goodbye!")

asyncio.run(greet())
```  

### **How It Works:**  
1. The `greet()` function is defined as **async**, meaning it returns a coroutine.  
2. Inside `greet()`, we use `await asyncio.sleep(1)`, which **pauses execution** for 1 second without blocking the entire program.  
3. `asyncio.run(greet())` starts the **event loop**, executes `greet()`, and then closes the loop.  

**Expected Output:**  
```
Hello!
(1 second delay)
Goodbye!
```  

#### **What Happens if We Remove `await`?**  
If we remove `await` inside `greet()`, Python will complain:  
```
RuntimeWarning: coroutine 'greet' was never awaited
```
This is because `async` functions return coroutines, and **they must be awaited or scheduled as tasks**.  

---

## **2.3 Understanding the Event Loop**  

The **event loop** is the heart of `asyncio`. It manages coroutines and schedules them for execution.  

### **How the Event Loop Works:**  
1. A coroutine (async function) is submitted to the event loop.  
2. If a coroutine encounters an `await` statement, execution is paused, and control is given back to the event loop.  
3. The event loop runs other tasks while waiting for the awaited coroutine to complete.  
4. Once complete, execution resumes from where it was paused.  

### **Visualizing the Event Loop:**  
```
1. Task A starts
2. Task A hits "await", pauses
3. Task B starts while Task A is paused
4. Task B completes
5. Task A resumes after "await"
6. Task A completes
```

Let’s see an example with **multiple coroutines** running on the event loop:  

```python
import asyncio

async def task(name, delay):
    print(f"{name} starting...")
    await asyncio.sleep(delay)
    print(f"{name} finished!")

async def main():
    await asyncio.gather(
        task("Task 1", 2),
        task("Task 2", 1),
        task("Task 3", 3)
    )

asyncio.run(main())
```

**Expected Output:**  
```
Task 1 starting...
Task 2 starting...
Task 3 starting...
Task 2 finished!
Task 1 finished!
Task 3 finished!
```

### **Key Observations:**  
- All tasks **start immediately**.  
- `Task 2` finishes first because it has the shortest delay.  
- The event loop efficiently switches between tasks while waiting for `await` operations to complete.  

---

## **2.4 Alternative Ways to Run the Event Loop**  

`asyncio.run()` is the recommended way to run `asyncio` programs in Python 3.7+. However, there are other ways to manually manage the event loop.  

### **Method 1: Using `asyncio.run()` (Recommended)**  
```python
asyncio.run(main())  
```
- **Automatically creates and closes the event loop**.  
- Simple and ideal for most use cases.  

### **Method 2: Using `asyncio.get_event_loop()` (Manual Management)**  
For advanced users, manually running the event loop gives more control:  

```python
loop = asyncio.get_event_loop()
loop.run_until_complete(main())
loop.close()
```
- Useful when integrating `asyncio` with **other event loops** (e.g., GUI frameworks, networking libraries).  
- **Requires explicit loop management** (not recommended for beginners).  

### **Method 3: Running Coroutines Inside an Existing Event Loop**  
In **Jupyter Notebooks**, `asyncio.run()` doesn’t work because Jupyter itself manages an event loop. Instead, use:  

```python
import asyncio

async def example():
    print("Hello from Jupyter!")
    await asyncio.sleep(1)
    print("Goodbye!")

await example()
```

---

## **2.5 Summary**  

✅ `asyncio` allows Python programs to run non-blocking tasks concurrently.  
✅ The **event loop** schedules and runs coroutines.  
✅ Use `asyncio.run()` to execute an `async` function.  
✅ `await` is required to pause and resume coroutines efficiently.  
✅ `asyncio.gather()` runs multiple coroutines concurrently.  

---

## **Next Steps**  
Now that you have a working understanding of the event loop and how to run asynchronous functions, we’ll dive deeper into **coroutines, tasks, and awaitables** in the next chapter.
