# **5. Asynchronous I/O with `asyncio`**  

In this chapter, we'll explore **asynchronous input and output (I/O) operations** using `asyncio`, including:  
- **Reading and writing files asynchronously**  
- **Making HTTP requests with `aiohttp`**  
- **Working with databases asynchronously**  

---

## **5.1 Why Use Asynchronous I/O?**  

I/O-bound tasks (e.g., reading files, network requests, database queries) **block execution in synchronous code**. With `asyncio`, we can perform these operations without blocking the event loop, improving efficiency.

### **Synchronous vs. Asynchronous File I/O**  

#### **Synchronous File Read (Blocking)**  

```python
with open("example.txt", "r") as f:
    content = f.read()  # Blocks execution
    print(content)
```

#### **Asynchronous File Read (Non-Blocking)**  

```python
import aiofiles
import asyncio

async def read_file():
    async with aiofiles.open("example.txt", "r") as f:
        content = await f.read()  # Non-blocking
        print(content)

asyncio.run(read_file())
```

✅ **Key Benefit:** Other tasks can run while waiting for the file operation to complete.

---

## **5.2 Asynchronous File Handling with `aiofiles`**  

The `aiofiles` library provides an **async-friendly interface** for file operations.  

### **Example: Writing to a File Asynchronously**  

```python
import aiofiles
import asyncio

async def write_file():
    async with aiofiles.open("output.txt", "w") as f:
        await f.write("Hello, asyncio!\n")

asyncio.run(write_file())
```

### **Example: Reading a File Line by Line**  

```python
async def read_lines():
    async with aiofiles.open("example.txt", "r") as f:
        async for line in f:
            print(line.strip())

asyncio.run(read_lines())
```

---

## **5.3 Making Asynchronous HTTP Requests with `aiohttp`**  

For network I/O, `aiohttp` provides an asynchronous alternative to `requests`.

### **Example: Fetching a Web Page Asynchronously**  

```python
import aiohttp
import asyncio

async def fetch(url):
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            content = await response.text()
            print(content[:500])  # Print first 500 characters

asyncio.run(fetch("https://www.example.com"))
```

### **How It Works:**  
- `ClientSession()` creates a session (efficient for multiple requests).  
- `session.get(url)` sends an **asynchronous** request.  
- `await response.text()` reads the response **without blocking** the event loop.  

---

## **5.4 Making Multiple HTTP Requests Concurrently**  

Instead of fetching pages **one at a time**, we can **send multiple requests concurrently**.

### **Example: Fetching Multiple URLs**  

```python
import aiohttp
import asyncio

async def fetch(session, url):
    async with session.get(url) as response:
        return await response.text()

async def main():
    urls = ["https://example.com", "https://httpbin.org/get", "https://jsonplaceholder.typicode.com/todos/1"]
    
    async with aiohttp.ClientSession() as session:
        tasks = [fetch(session, url) for url in urls]
        responses = await asyncio.gather(*tasks)  # Fetch all URLs concurrently
    
    for response in responses:
        print(response[:200])  # Print first 200 characters of each response

asyncio.run(main())
```

✅ **`asyncio.gather(*tasks)` runs all tasks in parallel**, significantly speeding up execution.

---

## **5.5 Asynchronous Database Queries**  

Most databases support **async drivers** for efficient querying.  

### **Example: Async PostgreSQL Queries with `asyncpg`**  

```python
import asyncpg
import asyncio

async def fetch_data():
    conn = await asyncpg.connect("postgresql://user:password@localhost/dbname")
    rows = await conn.fetch("SELECT * FROM users")
    await conn.close()
    
    for row in rows:
        print(row)

asyncio.run(fetch_data())
```

✅ **Key Benefits:**  
- `asyncpg.connect()` establishes an async database connection.  
- `conn.fetch()` retrieves data **without blocking** the event loop.  

---

## **5.6 Summary**  

✅ **File I/O:** Use `aiofiles` for non-blocking file operations.  
✅ **HTTP Requests:** Use `aiohttp` to fetch data asynchronously.  
✅ **Concurrent Requests:** Use `asyncio.gather()` to run multiple requests in parallel.  
✅ **Database Queries:** Use async database clients like `asyncpg` for PostgreSQL.  

---

## **Next Steps**  

In the next chapter, we’ll dive into **synchronization primitives** (locks, events, and semaphores) in `asyncio` to manage shared resources safely.
