The simplest FastAPI file could look like this:

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
async def root():
    return {"message": "Hello World"}
```


Run the live server: `fastapi dev main.py`

Interactive API docs at : `http://127.0.0.1:8000/docs.`


## Path Parameters

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_item(item_id):
    return {"item_id": item_id}
```

#### Path parameters with types

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_item(item_id: int):
    return {"item_id": item_id}
```

- In this case, item_id is declared to be an int.
- You can use the same type declarations with str, float, bool and many other complex data types.

**Order matters**
- When creating path operations, you can find situations where you have a fixed path.

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/users/me")
async def read_user_me():
    return {"user_id": "the current user"}


@app.get("/users/{user_id}")
async def read_user(user_id: str):
    return {"user_id": user_id}
```
- here `/users/me` path will be evaluated first and then `/users/{user_id}` will be evaluated.
- you cannot redefine a path operation.
- The first one will always be used since the path matches first.


**Predefined value**
> If you have a path operation that receives a path parameter, but you want the possible valid path parameter values to be predefined, you can use a standard Python Enum.

- Import Enum and create a sub-class that inherits from str and from Enum.

-  By inheriting from str the API docs will be able to know that the values must be of type string and will be able to render correctly.

- Then create class attributes with fixed values, which will be the available valid values

- Then create a path parameter with a type annotation using the enum class you created (ModelName):

```python
from enum import Enum

from fastapi import FastAPI


class ModelName(str, Enum):
    alexnet = "alexnet"
    resnet = "resnet"
    lenet = "lenet"


app = FastAPI()


@app.get("/models/{model_name}")
async def get_model(model_name: ModelName):
    if model_name is ModelName.alexnet:  
        # The value of the path parameter will be an enumeration member. You can compare it with the enumeration member in your created enum ModelName:
        return {"model_name": model_name, "message": "Deep Learning FTW!"}

    if model_name.value == "lenet":
        # You can get the actual value (a str in this case) using model_name.value, or in general, your_enum_member.value
        return {"model_name": ModelName.lenet.value, "message": "LeCNN all the images"}
        # You could also access the value "lenet" with ModelName.lenet.value.

    return {"model_name": model_name, "message": "Have some residuals"}
```

**Path parameters containing paths**

> OpenAPI doesn't support a way to declare a path parameter to contain a path inside, as that could lead to scenarios that are difficult to test and define.
Nevertheless, you can still do it in FastAPI.

`/files/{file_path:path}`
- In this case, the name of the parameter is `file_path`, and the last part, `:path`, tells it that the parameter should match any path.

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/files/{file_path:path}")
async def read_file(file_path: str):
    return {"file_path": file_path}
```