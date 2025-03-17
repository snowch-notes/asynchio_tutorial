# **9. JupyterBook Integration**  

JupyterBook is a powerful framework for creating interactive, documentation-rich books. In this chapter, we will integrate our `asyncio` tutorial into JupyterBook, covering:  

- **Setting up JupyterBook**  
- **Writing and running asyncio code in Jupyter Notebooks**  
- **Adding interactive demos**  
- **Publishing the book online**  

---

## **9.1 Setting Up JupyterBook**  

### **Installation**  
Ensure you have Python 3.8+ and install JupyterBook:  

```bash
pip install jupyter-book
```

### **Creating a JupyterBook Project**  
Run the following command to create a new book structure:  

```bash
jupyter-book create my-asyncio-book
cd my-asyncio-book
```

This generates a directory with:  
- **`_config.yml`** – Book configuration  
- **`_toc.yml`** – Table of contents  
- **`content/`** – Folder for notebooks and markdown files  

### **Building the Book**  
To preview the book locally:  

```bash
jupyter-book build my-asyncio-book
```

To open it:  

```bash
cd my-asyncio-book/_build/html && python -m http.server
```

---

## **9.2 Writing Asyncio Code in Jupyter Notebooks**  

Since `asyncio.run()` does not work inside Jupyter Notebooks, we use `nest_asyncio` for interactive execution.  

### **Installing `nest_asyncio`**  
```bash
pip install nest_asyncio
```

### **Running Asyncio Code in Jupyter**  
```python
import asyncio
import nest_asyncio

nest_asyncio.apply()  # Allows running asyncio in Jupyter

async def hello():
    await asyncio.sleep(1)
    return "Hello, Asyncio in Jupyter!"

await hello()
```

✅ **Key Takeaways:**  
- `nest_asyncio.apply()` prevents event loop conflicts.  
- `await` allows interactive execution without `asyncio.run()`.  

---

## **9.3 Adding Interactive Demos**  

Jupyter widgets (`ipywidgets`) enable interactive `asyncio` demos.  

### **Installing `ipywidgets`**  
```bash
pip install ipywidgets
```

### **Example: Async Counter with Widgets**  
```python
import asyncio
import ipywidgets as widgets
from IPython.display import display

button = widgets.Button(description="Start Counting")
output = widgets.Output()

async def async_counter():
    for i in range(5):
        await asyncio.sleep(1)
        with output:
            print(f"Count: {i+1}")

def on_button_click(b):
    asyncio.create_task(async_counter())

button.on_click(on_button_click)
display(button, output)
```

✅ **Key Takeaways:**  
- `ipywidgets` enables user interaction.  
- `asyncio.create_task()` allows background tasks.  

---

## **9.4 Publishing the Book**  

### **Building the Final HTML**  
```bash
jupyter-book build my-asyncio-book
```

### **Deploying with GitHub Pages**  
1. Push the book to GitHub.  
2. Enable GitHub Pages in repository settings.  
3. Use:  
   ```bash
   ghp-import -n -p -f _build/html
   ```
4. Access the book at `https://your-username.github.io/my-asyncio-book/`.  

✅ **Key Takeaways:**  
- **JupyterBook + GitHub Pages** enables free hosting.  
- **Jupyter widgets** make demos interactive.  

---

## **Next Steps**  
In the next chapter, we will explore **advanced asyncio patterns**, including cancellation, backpressure handling, and integration with databases. 🚀
