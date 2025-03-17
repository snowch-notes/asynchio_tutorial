# **10. Advanced Topics in `asyncio`**  

This chapter explores advanced `asyncio` features, including:  

- **Running synchronous code inside async functions** using `asyncio.run_in_executor()`.  
- **Streaming data processing** with `aiokafka`.  
- **Writing custom event loops** for fine-grained control.  
- **Profiling and optimizing async code** for performance.  

---

## **10.1 `asyncio.run_in_executor()` – Mixing Sync & Async Code**  

Not all libraries support `asyncio`. If you need to call blocking code (e.g., a CPU-intensive task or a slow database query), use `asyncio.run_in_executor()` to run it in a thread or process.  

### **Example: Running Blocking Code in a Thread**  
```python
import asyncio
import time
from concurrent.futures import ThreadPoolExecutor

def blocking_task(n):
    time.sleep(2)
    return f"Processed {n}"

async def main():
    loop = asyncio.get_running_loop()
    with ThreadPoolExecutor() as executor:
        result = await loop.run_in_executor(executor, blocking_task, 42)
        print(result)

asyncio.run(main())
```

✅ **Key Takeaways:**  
- Runs CPU-bound tasks without blocking the event loop.  
- Uses `ThreadPoolExecutor` for I/O-bound tasks and `ProcessPoolExecutor` for CPU-bound tasks.  

---

## **10.2 Using `aiokafka` for Streaming Data Processing**  

Kafka is a popular real-time streaming platform. `aiokafka` provides an async Kafka client for Python.  

### **Installation**  
```bash
pip install aiokafka
```

### **Example: Async Kafka Consumer**  
```python
import asyncio
from aiokafka import AIOKafkaConsumer

async def consume():
    consumer = AIOKafkaConsumer(
        "my_topic", bootstrap_servers="localhost:9092", group_id="my_group"
    )
    await consumer.start()
    try:
        async for message in consumer:
            print(f"Received: {message.value.decode()}")
    finally:
        await consumer.stop()

asyncio.run(consume())
```

✅ **Key Takeaways:**  
- `aiokafka` enables non-blocking Kafka producers/consumers.  
- Works seamlessly with `asyncio`.  

---

## **10.3 Writing Custom Event Loops**  

While `asyncio` provides a default event loop, sometimes you need a custom loop for specialized behavior.  

### **Example: Custom Event Loop**  
```python
import asyncio

class CustomEventLoop(asyncio.SelectorEventLoop):
    def run_forever(self):
        print("Custom loop running...")
        super().run_forever()

async def main():
    print("Hello from custom loop")

loop = CustomEventLoop()
asyncio.set_event_loop(loop)
loop.run_until_complete(main())
```

✅ **Key Takeaways:**  
- Extending `SelectorEventLoop` allows custom behavior.  
- Useful for integrating with non-standard event-driven systems.  

---

## **10.4 Profiling and Performance Optimization**  

### **Finding Bottlenecks with `asyncio.TaskGroup` (Python 3.11+)**  
```python
import asyncio
import time

async def slow_task(n):
    await asyncio.sleep(n)
    return n

async def main():
    async with asyncio.TaskGroup() as tg:
        tasks = [tg.create_task(slow_task(i)) for i in range(1, 4)]
    
    results = [task.result() for task in tasks]
    print("Results:", results)

asyncio.run(main())
```
✅ **Key Takeaways:**  
- `TaskGroup` helps manage async tasks efficiently.  
- Ensures all tasks complete before proceeding.  

### **Using `cProfile` for Performance Analysis**  
```bash
python -m cProfile -s time my_async_script.py
```
✅ **Key Takeaways:**  
- Helps identify slow async operations.  
- Sort results by execution time for better analysis.  

---

## **10.5 Summary**  

| **Feature** | **Key Benefit** |
|------------|----------------|
| `asyncio.run_in_executor()` | Runs sync code inside async functions |
| `aiokafka` | Async Kafka streaming |
| Custom event loops | Fine-grained control over async behavior |
| `cProfile` | Performance optimization |

---

## **Next Steps**  
In the next chapter, we will conclude with best practices, additional resources, and future directions for `asyncio`. 🚀
