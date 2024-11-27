# Custom Templating

If you're not into Jinja templating, the `hx()` and `page()` decorators give you all the flexibility you need: you can integrate any HTML rendering or templating engine into `fasthx` simply by implementing the `HTMLRenderer` protocol. Similarly to the Jinja case, `hx()` only triggers HTML rendering for HTMX requests, while `page()` unconditionally renders HTML. See the example code below:

```python
from typing import Annotated, Any

from fastapi import Depends, FastAPI, Request
from fasthx import hx, page

# Create the app.
app = FastAPI()

# Create a dependecy to see that its return value is available in the render function.
async def get_random_number() -> int:
    return 4  # Chosen by fair dice roll.

DependsRandomNumber = Annotated[int, Depends(get_random_number)]

# Create the render methods: they must always have these three arguments.
# If you're using static type checkers, the type hint of `result` must match
# the return type annotation of the route on which this render method is used.
async def render_index(result: list[dict[str, str]], *, context: dict[str, Any], request: Request) -> str:
    return "<h1>Hello FastHX</h1>"

async def render_user_list(result: list[dict[str, str]], *, context: dict[str, Any], request: Request) -> str:
    # The value of the `DependsRandomNumber` dependency is accessible with the same name as in the route.
    random_number = context["random_number"]
    lucky_number = f"<h1>{random_number}</h1>"
    users = "".join(("<ul>", *(f"<li>{u['name']}</li>" for u in result), "</ul>"))
    return f"{lucky_number}\n{users}"

@app.get("/")
@page(render_index)
async def index() -> None:
    ...

@app.get("/htmx-or-data")
@hx(render_user_list)
async def htmx_or_data(random_number: DependsRandomNumber) -> list[dict[str, str]]:
    return [{"name": "Joe"}]

@app.get("/htmx-only")
@hx(render_user_list, no_data=True)
async def htmx_only(random_number: DependsRandomNumber) -> list[dict[str, str]]:
    return [{"name": "Joe"}]
```
