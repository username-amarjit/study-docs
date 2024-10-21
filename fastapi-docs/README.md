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


## Query Parameters

> When you declare other function parameters that are not part of the path parameters, they are automatically interpreted as "query" parameters


```python
fake_items_db = [{"item_name": "Foo"}, {"item_name": "Bar"}, {"item_name": "Baz"}]

@app.get("/items/")
async def read_item(skip: int = 0, limit: int = 10):
    #The query is the set of key-value pairs that go after the ? in a URL, separated by & characters.

    return fake_items_db[skip : skip + limit]
```

For example, in the URL:
the query parameters are:

- skip: with a value of 0
- limit: with a value of 10

`http://127.0.0.1:8000/items/?skip=0&limit=10`

As they are part of the URL, they are "naturally" strings.

**Defaults**

> As query parameters are not a fixed part of a path, they can be optional and can have default values.
In the example above they have default values of skip=0 and limit=10.

**Optional parameters**

> The same way, you can declare optional query parameters, by setting their default to None.

```python 
@app.get("/items/{item_id}")
async def read_item(item_id: str, q: str | None = None):
    if q:
        return {"item_id": item_id, "q": q}
    return {"item_id": item_id}
```

- Also notice that FastAPI is smart enough to notice that the path parameter item_id is a path parameter and q is not, so, it's a query parameter.

**Query parameter type conversion**

> You can also declare bool types, and they will be converted:

```python
@app.get("/items/{item_id}")
async def read_item(item_id: str, q: str | None = None, short: bool = False):
    item = {"item_id": item_id}
    if q:
        item.update({"q": q})
    if not short:
        item.update(
            {"description": "This is an amazing item that has a long description"}
        )
    return item
```

**Multiple path and query parameters**
> You can declare multiple path parameters and query parameters at the same time, FastAPI knows which is which.
And you don't have to declare them in any specific order.
They will be detected by name.

```python
@app.get("/users/{user_id}/items/{item_id}")
async def read_user_item(
    user_id: int, item_id: str, q: str | None = None, short: bool = False
):
    item = {"item_id": item_id, "owner_id": user_id}
    if q:
        item.update({"q": q})
    if not short:
        item.update(
            {"description": "This is an amazing item that has a long description"}
        )
    return item
```


**Required query parameters**
> When you declare a default value for non-path parameters (for now, we have only seen query parameters), then it is not required.
If you don't want to add a specific value but just make it optional, set the default as None.
But when you want to make a query parameter required, you can just not declare any default value.

```python
@app.get("/items/{item_id}")
async def read_user_item(item_id: str, needy: str):
    item = {"item_id": item_id, "needy": needy}
    return item
```

- Here the query parameter needy is a required query parameter of type str.
- You could also use Enums the same way as with Path Parameters.



##