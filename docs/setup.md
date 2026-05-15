# Setup and Usage

Monty can be called from Python, JavaScript/TypeScript, or Rust. This page walks through installation and the most common usage patterns.

## Python

### Install

Install with [uv](https://docs.astral.sh/uv/):

```bash
uv add pydantic-monty
```

Or with pip:

```bash
pip install pydantic-monty
```

### Basic example

```python
from typing import Any
import pydantic_monty

code = """
async def agent(prompt: str, messages: Messages):
    while True:
        print(f'messages so far: {messages}')
        output = await call_llm(prompt, messages)
        if isinstance(output, str):
            return output
        messages.extend(output)

await agent(prompt, [])
"""

type_definitions = """
from typing import Any
Messages = list[dict[str, Any]]
async def call_llm(prompt: str, messages: Messages) -> str | Messages:
    raise NotImplementedError()
prompt: str = ''
"""

m = pydantic_monty.Monty(
    code,
    inputs=['prompt'],
    script_name='agent.py',
    type_check=True,
    type_check_stubs=type_definitions,
)


Messages = list[dict[str, Any]]


async def call_llm(prompt: str, messages: Messages) -> str | Messages:
    if len(messages) < 2:
        return [{'role': 'system', 'content': 'example response'}]
    else:
        return f'example output, message count {len(messages)}'


async def main():
    output = await m.run_async(
        inputs={'prompt': 'testing'},
        external_functions={'call_llm': call_llm},
    )
    print(output)


if __name__ == '__main__':
    import asyncio
    asyncio.run(main())
```

### Iterative execution with `start()` and `resume()`

Use `start()` and `resume()` when you want to handle external function calls iteratively, taking control over every call:

```python
import pydantic_monty

code = """
data = fetch(url)
len(data)
"""

m = pydantic_monty.Monty(code, inputs=['url'])

# Start execution - pauses when fetch() is called
result = m.start(inputs={'url': 'https://example.com'})

# result is a FunctionSnapshot; result.function_name == 'fetch'
# Perform the actual fetch, then resume with the result
result = result.resume(return_value='hello world')
# result.output == 11
```

### Serialization with `dump()` and `load()`

Both `Monty` and snapshot types such as `FunctionSnapshot` can be serialized to bytes and restored later. This lets you cache parsed code or suspend execution across process boundaries:

```python
import pydantic_monty

# Serialize parsed code to avoid re-parsing
m = pydantic_monty.Monty('x + 1', inputs=['x'])
data = m.dump()

m2 = pydantic_monty.Monty.load(data)
print(m2.run(inputs={'x': 41}))  # 42

# Serialize execution state mid-flight
m = pydantic_monty.Monty('fetch(url)', inputs=['url'])
progress = m.start(inputs={'url': 'https://example.com'})
state = progress.dump()

progress2 = pydantic_monty.load_snapshot(state)
result = progress2.resume(return_value='response data')
```

## Rust

Use `MontyRun` to parse and execute code from Rust:

```rust
use monty::{MontyRun, MontyObject, NoLimitTracker, PrintWriter};

let code = r#"
def fib(n):
    if n <= 1:
        return n
    return fib(n - 1) + fib(n - 2)

fib(x)
"#;

let runner = MontyRun::new(code.to_owned(), "fib.py", vec!["x".to_owned()]).unwrap();
let result = runner.run(vec![MontyObject::Int(10)], NoLimitTracker, PrintWriter::Stdout).unwrap();
assert_eq!(result, MontyObject::Int(55));
```

### Rust serialization

`MontyRun` and `RunProgress` expose `dump()` and `load()` methods that mirror the Python API:

```rust
use monty::{MontyRun, MontyObject, NoLimitTracker, PrintWriter};

let runner = MontyRun::new("x + 1".to_owned(), "main.py", vec!["x".to_owned()]).unwrap();
let bytes = runner.dump().unwrap();

let runner2 = MontyRun::load(&bytes).unwrap();
let result = runner2.run(vec![MontyObject::Int(41)], NoLimitTracker, PrintWriter::Stdout).unwrap();
assert_eq!(result, MontyObject::Int(42));
```

## Community bindings

- **Go**: [gomonty](https://github.com/ewhauser/gomonty/) - community-maintained Go bindings for the Monty interpreter.
