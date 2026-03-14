Thought about this
Python Cheatsheet for Intermediate Developers
# Python Cheatsheet for Intermediate Developers

## 🔹 Advanced Functions

### Decorators
```python
from functools import wraps
import time

def timer(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        print(f"{func.__name__} took {time.time() - start:.2f}s")
        return result
    return wrapper

@timer
def slow_function():
    time.sleep(1)

# Decorator with arguments
def repeat(times):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for _ in range(times):
                func(*args, **kwargs)
        return wrapper
    return decorator

@repeat(3)
def greet():
    print("Hello!")
Lambda & Higher-Order Functions
# Lambda with conditions
square = lambda x: x**2 if x > 0 else 0

# map, filter, reduce
from functools import reduce

numbers = [1, 2, 3, 4, 5]
squared = list(map(lambda x: x**2, numbers))
evens = list(filter(lambda x: x % 2 == 0, numbers))
product = reduce(lambda x, y: x * y, numbers)

# sorted with key
items = [('apple', 3), ('banana', 1), ('cherry', 2)]
sorted_items = sorted(items, key=lambda x: x[1])
🔹 Context Managers
Custom Context Manager
from contextlib import contextmanager

@contextmanager
def open_file(path, mode):
    f = open(path, mode)
    try:
        yield f
    finally:
        f.close()

# Using __enter__ and __exit__
class DatabaseConnection:
    def __init__(self, db_url):
        self.db_url = db_url
        self.conn = None
    
    def __enter__(self):
        self.conn = connect(self.db_url)
        return self.conn
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        if exc_type:
            self.conn.rollback()
        else:
            self.conn.commit()
        self.conn.close()
        return False  # Don't suppress exceptions
contextlib Utilities
from contextlib import nullcontext, suppress, redirect_stdout

# Suppress specific exceptions
with suppress(FileNotFoundError):
    os.remove('nonexistent.txt')

# Redirect output
with open('log.txt', 'w') as f:
    with redirect_stdout(f):
        print("This goes to file")

# No-op context manager
with nullcontext():
    # Conditional context
    pass
🔹 Generators & Iterators
Generator Functions
def fibonacci(n):
    a, b = 0, 1
    for _ in range(n):
        yield a
        a, b = b, a + b

# Generator expression
squares = (x**2 for x in range(10))

# Sending values to generator
def accumulator():
    total = 0
    while True:
        value = yield total
        if value is None:
            break
        total += value

acc = accumulator()
next(acc)  # Prime
print(acc.send(5))   # 5
print(acc.send(3))   # 8
itertools
from itertools import chain, cycle, islice, groupby, permutations, combinations

# Chain multiple iterables
list(chain([1, 2], [3, 4]))  # [1, 2, 3, 4]

# Infinite cycling
list(islice(cycle([1, 2, 3]), 5))  # [1, 2, 3, 1, 2]

# Group consecutive elements
data = ['a', 'a', 'b', 'b', 'b', 'c']
{key: list(group) for key, group in groupby(data)}

# Permutations and combinations
list(permutations([1, 2, 3], 2))  # [(1,2), (1,3), (2,1), (2,3), (3,1), (3,2)]
list(combinations([1, 2, 3], 2))  # [(1,2), (1,3), (2,3)]
🔹 Advanced Data Structures
collections Module
from collections import defaultdict, Counter, deque, namedtuple, OrderedDict

# Default dict
word_counts = defaultdict(int)
for word in words:
    word_counts[word] += 1

# Counter
Counter("abracadabra").most_common(3)  # [('a', 5), ('b', 2), ('r', 2)]

# Double-ended queue
dq = deque(maxlen=5)
dq.append(1)
dq.appendleft(0)

# Named tuple
Point = namedtuple('Point', ['x', 'y'])
p = Point(10, 20)
print(p.x, p.y)

# OrderedDict (preserves insertion order in Python 3.7+)
ordered = OrderedDict([('a', 1), ('b', 2)])
dataclasses
from dataclasses import dataclass, field
from typing import List

@dataclass(frozen=True)  # Immutable
class Person:
    name: str
    age: int
    hobbies: List[str] = field(default_factory=list)
    
    def __post_init__(self):
        if self.age < 0:
            raise ValueError("Age must be positive")
🔹 Async Programming
async/await Basics
import asyncio
import aiohttp

async def fetch(url):
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            return await response.text()

async def main():
    urls = ['http://example.com', 'http://example.org']
    tasks = [fetch(url) for url in urls]
    results = await asyncio.gather(*tasks)

asyncio.run(main())
Async Patterns
# Timeout
try:
    await asyncio.wait_for(fetch(url), timeout=5.0)
except asyncio.TimeoutError:
    print("Request timed out")

# Semaphore for rate limiting
semaphore = asyncio.Semaphore(10)

async def limited_fetch(url):
    async with semaphore:
        return await fetch(url)

# Background tasks
async def background_task():
    while True:
        await asyncio.sleep(1)
        print("Background running...")

async def main():
    task = asyncio.create_task(background_task())
    await asyncio.sleep(5)
    task.cancel()
🔹 Metaclasses
Basic Metaclass
class Meta(type):
    def __new__(mcs, name, bases, attrs):
        # Modify class before creation
        attrs['created_by'] = 'Meta'
        return super().__new__(mcs, name, bases, attrs)

class MyClass(metaclass=Meta):
    pass

print(MyClass.created_by)  # 'Meta'
Common Use Cases
# Singleton pattern via metaclass
class Singleton(type):
    _instances = {}
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Database(metaclass=Singleton):
    pass
🔹 Descriptors
class ValidatedAttribute:
    def __init__(self, validator):
        self.validator = validator
        self.data = {}
    
    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return self.data.get(id(obj))
    
    def __set__(self, obj, value):
        if not self.validator(value):
            raise ValueError(f"Invalid value: {value}")
        self.data[id(obj)] = value

class Person:
    age = ValidatedAttribute(lambda x: 0 <= x <= 150)
    
    def __init__(self, age):
        self.age = age
🔹 Performance Optimization
Profiling
import cProfile
import pstats

# Profile code
cProfile.run('my_function()')

# Save and analyze
profiler = cProfile.Profile()
profiler.enable()
my_function()
profiler.disable()
profiler.dump_stats('profile.prof')

# Line-by-line profiling
# pip install line_profiler
# @profile
# def my_func(): ...
Memory Management
import tracemalloc

tracemalloc.start()
# ... code ...
current, peak = tracemalloc.get_traced_memory()
print(f"Current: {current / 10**6:.2f} MB")
tracemalloc.stop()

# Weak references
import weakref
class Node:
    pass
node = Node()
weak_node = weakref.ref(node)
Common Optimizations
# Use local variables (faster lookup)
def fast_function():
    local_len = len
    for _ in range(1000000):
        local_len([1, 2, 3])

# List comprehension vs map
# List comprehension is generally faster
result = [x**2 for x in range(1000)]

# Use built-ins
sum(numbers)  # Faster than manual loop

# Generator for memory efficiency
def process_large_file():
    with open('large.txt') as f:
        for line in f:
            yield process(line)
🔹 Type Hints & Static Analysis
from typing import (
    List, Dict, Optional, Union, Callable, 
    TypeVar, Generic, Protocol, Literal
)
from dataclasses import dataclass

T = TypeVar('T')

# Generic class
class Stack(Generic[T]):
    def __init__(self) -> None:
        self._items: List[T] = []
    
    def push(self, item: T) -> None:
        self._items.append(item)
    
    def pop(self) -> T:
        return self._items.pop()

# Protocol (structural subtyping)
class Drawable(Protocol):
    def draw(self) -> None: ...

def render(obj: Drawable) -> None:
    obj.draw()

# Literal types
def set_mode(mode: Literal['read', 'write', 'append']) -> None:
    pass

# TypedDict
from typing import TypedDict

class User(TypedDict):
    name: str
    age: int
    active: bool = True
🔹 Testing
pytest Patterns
import pytest
from unittest.mock import Mock, patch, MagicMock

# Parametrized tests
@pytest.mark.parametrize('input,expected', [
    (2, 4), (3, 9), (4, 16)
])
def test_square(input, expected):
    assert input ** 2 == expected

# Fixtures
@pytest.fixture
def sample_data():
    return {'users': ['alice', 'bob']}

def test_users(sample_data):
    assert len(sample_data['users']) == 2

# Mocking
def test_api_call():
    with patch('module.requests.get') as mock_get:
        mock_get.return_value.status_code = 200
        result = module.fetch_data()
        mock_get.assert_called_once_with('http://api.example.com')

# Async tests
@pytest.mark.asyncio
async def test_async_function():
    result = await async_function()
    assert result is not None
Coverage
# Run with coverage
pytest --cov=module --cov-report=html

# Exclude from coverage
# pragma: no cover
def debug_helper():
    pass
🔹 Packaging & Distribution
pyproject.toml (Modern)
[build-system]
requires = ["setuptools>=45", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "mypackage"
version = "0.1.0"
description = "My package"
dependencies = [
    "requests>=2.28.0",
    "numpy>=1.21.0",
]

[project.optional-dependencies]
dev = ["pytest", "black", "mypy"]

[tool.black]
line-length = 88
target-version = ['py39']
Entry Points
# In pyproject.toml
[project.scripts]
my-cli = "mypackage.cli:main"

# In package/cli.py
def main():
    import argparse
    parser = argparse.ArgumentParser()
    parser.parse_args()
🔹 Design Patterns
Factory Pattern
from abc import ABC, abstractmethod

class PaymentProcessor(ABC):
    @abstractmethod
    def process(self, amount):
        pass

class CreditCardProcessor(PaymentProcessor):
    def process(self, amount):
        return f"Processing ${amount} via credit card"

class PayPalProcessor(PaymentProcessor):
    def process(self, amount):
        return f"Processing ${amount} via PayPal"

class PaymentFactory:
    @staticmethod
    def create_processor(method: str) -> PaymentProcessor:
        processors = {
            'credit': CreditCardProcessor,
            'paypal': PayPalProcessor,
        }
        return processors[method]()
Observer Pattern
class Subject:
    def __init__(self):
        self._observers = []
    
    def attach(self, observer):
        self._observers.append(observer)
    
    def notify(self, message):
        for observer in self._observers:
            observer.update(message)

class Observer:
    def update(self, message):
        print(f"Received: {message}")
🔹 Common Pitfalls
# Mutable default argument
def bad_append(item, list=[]):  # WRONG
    list.append(item)
    return list

def good_append(item, list=None):  # RIGHT
    if list is None:
        list = []
    list.append(item)
    return list

# Late binding closures
def create_multipliers():
    return [lambda x: x * i for i in range(5)]  # WRONG

def create_multipliers_fixed():
    return [lambda x, i=i: x * i for i in range(5)]  # RIGHT

# Copying nested structures
import copy
shallow = original.copy()  # Only copies first level
deep = copy.deepcopy(original)  # Recursively copies
🔹 Useful One-Liners
# Flatten list
flat = [item for sublist in nested for item in sublist]

# Dictionary merge (Python 3.9+)
merged = dict1 | dict2

# Get unique items
unique = list(dict.fromkeys(items))

# Chunk list
chunks = [items[i:i+n] for i in range(0, len(items), n)]

# Zip with index
for idx, val in enumerate(values):
    pass

# Check all/any
all(x > 0 for x in numbers)
any(x < 0 for x in numbers)
🔹 Debugging Tools
# pdb
import pdb; pdb.set_trace()

# Better debugging
from rich.traceback import install
install()

# Logging configuration
import logging
logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)
logger = logging.getLogger(__name__)

# Assertions with messages
assert condition, "Descriptive error message"
Save as python_intermediate.md for quick reference!


---