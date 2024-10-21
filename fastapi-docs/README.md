The simplest FastAPI file could look like this:

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
async def root():
    return {"message": "Hello World"}
```


Run the live server:
``` fastapi dev main.py```