# **8. Real-World Examples of `asyncio`**  

Now that we've covered the fundamentals of `asyncio`, let's explore practical applications by implementing:  

- **An asynchronous web scraper** for fetching multiple web pages concurrently.  
- **A WebSocket-based chat server** using the `websockets` module.  
- **An async API** using FastAPI to handle concurrent requests efficiently.  

---

## **8.1 Building an Asynchronous Web Scraper**  

Web scraping involves making multiple HTTP requests, which can be **I/O-bound**. Using `asyncio` with `aiohttp` allows us to fetch multiple pages concurrently instead of sequentially.  

### **Installation**  
First, install `aiohttp` if you haven't already:  
```bash
pip install aiohttp
```

### **Async Web Scraper with `aiohttp`**  
```python
import asyncio
import aiohttp

async def fetch_url(session, url):
    async with session.get(url) as response:
        content = await response.text()
        print(f"Fetched {url} with {len(content)} characters")
        return content

async def main():
    urls = [
        "https://example.com",
        "https://www.python.org",
        "https://docs.python.org/3/library/asyncio.html"
    ]
    
    async with aiohttp.ClientSession() as session:
        tasks = [fetch_url(session, url) for url in urls]
        results = await asyncio.gather(*tasks)  # Fetch all pages concurrently

asyncio.run(main())
```

✅ **Key Takeaways:**  
- Uses **`aiohttp.ClientSession()`** for async HTTP requests.  
- Fetches multiple URLs concurrently with **`asyncio.gather()`**.  

---

## **8.2 Implementing a Chat Server with WebSockets**  

WebSockets provide a **real-time, bidirectional** communication channel between a client and server. We'll use the `websockets` module to implement an async chat server.  

### **Installation**  
```bash
pip install websockets
```

### **Asynchronous WebSocket Server**  
```python
import asyncio
import websockets

clients = set()

async def handler(websocket):
    clients.add(websocket)
    try:
        async for message in websocket:
            print(f"Received: {message}")
            # Broadcast message to all connected clients
            await asyncio.gather(*(client.send(message) for client in clients))
    finally:
        clients.remove(websocket)

async def main():
    async with websockets.serve(handler, "localhost", 8765):
        await asyncio.Future()  # Keeps the server running

asyncio.run(main())
```

### **WebSocket Client Example**  
```python
import asyncio
import websockets

async def client():
    async with websockets.connect("ws://localhost:8765") as websocket:
        while True:
            msg = input("Enter message: ")
            await websocket.send(msg)
            response = await websocket.recv()
            print(f"Server response: {response}")

asyncio.run(client())
```

✅ **Key Takeaways:**  
- The **server broadcasts messages** to all connected clients.  
- Each client **sends and receives messages asynchronously**.  

---

## **8.3 Creating a Simple Async API with FastAPI**  

FastAPI is a high-performance framework for building APIs with **async support**.  

### **Installation**  
```bash
pip install fastapi uvicorn
```

### **Basic Async API with FastAPI**  
```python
from fastapi import FastAPI
import asyncio

app = FastAPI()

@app.get("/")
async def home():
    await asyncio.sleep(1)  # Simulate async operation
    return {"message": "Hello, FastAPI with asyncio!"}

@app.get("/items/{item_id}")
async def read_item(item_id: int):
    return {"item_id": item_id, "status": "available"}

# Run the server with: uvicorn filename:app --reload
```

✅ **Key Takeaways:**  
- **`async def` endpoints** allow efficient request handling.  
- **FastAPI + Uvicorn** enables high-performance async APIs.  

---

## **8.4 Summary**  

| **Example** | **Technology Used** | **Key Benefit** |
|------------|----------------|----------------|
| **Web Scraper** | `aiohttp` | Fetch multiple pages concurrently |
| **Chat Server** | `websockets` | Real-time bidirectional communication |
| **Async API** | `FastAPI` | Efficient, scalable web API |

---

## **Next Steps**  
In the next chapter, we will explore **advanced asyncio patterns** like **cancellation, backpressure handling, and integrating with external systems**. 🚀
