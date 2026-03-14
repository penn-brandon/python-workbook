# Python Cheatsheet for Advanced Developers

## 🔹 Advanced Metaprogramming

### Metaclasses Deep Dive
```python
class SingletonMeta(type):
    _instances = {}
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            instance = super().__call__(*args, **kwargs)
            cls._instances[cls] = instance
        return cls._instances[cls]

class ThreadSafeSingleton(metaclass=SingletonMeta):
    pass

# Dynamic class creation
DynamicClass = type('DynamicClass', (BaseClass,), {'attr': 'value'})
Class Decorators & Registration
registry = {}

def register(cls):
    registry[cls.__name__] = cls
    return cls

@register
class PluginA: pass

@register
class PluginB: pass

# Auto-discovery pattern
def get_plugins():
    return [cls() for cls in registry.values()]
__slots__ for Memory Optimization
class Optimized:
    __slots__ = ['x', 'y']  # No __dict__, ~40% less memory
    
    def __init__(self, x, y):
        self.x = x
        self.y = y

# With inheritance
class Child(Optimized):
    __slots__ = ['z']  # Must declare all slots in hierarchy
🔹 Advanced Decorators
Decorator Factory with State
from functools import wraps
from collections import defaultdict

def retry(max_attempts=3, delay=1, exceptions=(Exception,)):
    stats = defaultdict(int)
    
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except exceptions as e:
                    stats[func.__name__] += 1
                    if attempt == max_attempts - 1:
                        raise
                    time.sleep(delay)
        return wrapper
    return decorator

# Usage
@retry(max_attempts=5, delay=0.5, exceptions=(ConnectionError,))
def unstable_api_call():
    pass
Property Decorators with Validation
class ValidatedProperty:
    def __init__(self, validator):
        self.validator = validator
        self.name = None
    
    def __set_name__(self, owner, name):
        self.name = f"_{name}"
    
    def __get__(self, obj, objtype=None):
        if obj is None: return self
        return getattr(obj, self.name)
    
    def __set__(self, obj, value):
        if not self.validator(value):
            raise ValueError(f"Invalid value: {value}")
        setattr(obj, self.name, value)

class Person:
    age = ValidatedProperty(lambda x: 0 <= x <= 150)
    email = ValidatedProperty(lambda x: '@' in x)
🔹 Async/Await Advanced Patterns
Async Context Managers
from contextlib import asynccontextmanager

@asynccontextmanager
async def async_database_connection(db_url):
    conn = await connect(db_url)
    try:
        yield conn
    finally:
        await conn.close()

async with async_database_connection(DB_URL) as conn:
    await conn.execute(query)
Asyncio Task Groups & Cancellation
import asyncio

async def task_group(tasks):
    """Run tasks with proper cancellation"""
    pending = set(asyncio.create_task(t) for t in tasks)
    try:
        return await asyncio.gather(*pending, return_exceptions=True)
    except asyncio.CancelledError:
        for task in pending:
            task.cancel()
        await asyncio.gather(*pending, return_exceptions=True)
        raise

# Semaphore for rate limiting
async def rate_limited_fetch(urls, limit=10):
    semaphore = asyncio.Semaphore(limit)
    
    async def fetch_with_limit(url):
        async with semaphore:
            return await fetch(url)
    
    return await asyncio.gather(*(fetch_with_limit(u) for u in urls))
Async Generators
async def async_generator(n):
    for i in range(n):
        await asyncio.sleep(0.1)
        yield i

async for item in async_generator(5):
    print(item)

# Async context manager with generator
async def async_resource_manager():
    resource = await acquire()
    try:
        yield resource
    finally:
        await release(resource)
🔹 Concurrency & Parallelism
Multiprocessing with Process Pools
from multiprocessing import Pool, cpu_count

def heavy_computation(x):
    return x ** 2

with Pool(cpu_count()) as pool:
    results = pool.map(heavy_computation, range(1000))

# Async with callbacks
pool.apply_async(heavy_computation, args=(5,), callback=lambda r: print(r))
Threading with Locks & RLocks
import threading

class ThreadSafeCounter:
    def __init__(self):
        self._value = 0
        self._lock = threading.RLock()  # Reentrant lock
    
    def increment(self):
        with self._lock:
            self._value += 1
            return self._value
    
    @property
    def value(self):
        with self._lock:
            return self._value

# Condition variables
class ProducerConsumer:
    def __init__(self):
        self.queue = []
        self.lock = threading.Lock()
        self.not_empty = threading.Condition(self.lock)
        self.not_full = threading.Condition(self.lock)
    
    def put(self, item):
        with self.not_full:
            while len(self.queue) >= MAX_SIZE:
                self.not_full.wait()
            self.queue.append(item)
            self.not_empty.notify()
    
    def get(self):
        with self.not_empty:
            while not self.queue:
                self.not_empty.wait()
            item = self.queue.pop(0)
            self.not_full.notify()
            return item
Concurrent.futures Advanced
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor, wait, FIRST_COMPLETED

# Executor with timeout
with ThreadPoolExecutor(max_workers=10) as executor:
    futures = [executor.submit(task, arg) for arg in args]
    done, not_done = wait(futures, timeout=30, return_when=FIRST_COMPLETED)

# Map with chunk size optimization
results = executor.map(process_item, items, chunksize=100)
🔹 Memory Management & Profiling
Reference Cycles & Garbage Collection
import gc
import sys

# Check reference cycles
gc.collect()
print(gc.garbage)  # Objects that couldn't be freed

# Disable GC temporarily for performance
gc.disable()
# ... critical section ...
gc.enable()

# Weak references for caches
import weakref

class Cache:
    def __init__(self):
        self._cache = weakref.WeakValueDictionary()
    
    def get(self, key):
        return self._cache.get(key)
    
    def set(self, key, value):
        self._cache[key] = value
Memory Profiling
import tracemalloc
import objgraph

# Trace allocations
tracemalloc.start()
# ... code ...
snapshot = tracemalloc.take_snapshot()
top_stats = snapshot.statistics('lineno')

# Find memory leaks
objgraph.show_growth()
objgraph.show_backrefs(objgraph.by_type('MyClass')[:5])

# Custom memory tracker
from pympler import asizeof
print(asizeof.asizeof(large_object))
Performance Optimization Techniques
# Local variable caching
def fast_function():
    local_append = list.append
    local_range = range
    result = []
    for i in local_range(1000000):
        local_append(result, i)
    return result

# NumPy vectorization (100x faster than loops)
import numpy as np
arr = np.array(range(1000000))
result = arr ** 2 + np.sin(arr)

# Cython for CPU-bound code
# %%cython
# def cython_func(double[:] arr):
#     cdef int i
#     cdef double total = 0
#     for i in range(len(arr)):
#         total += arr[i]
#     return total
🔹 Advanced Type Hints
Generics & Protocols
from typing import TypeVar, Generic, Protocol, runtime_checkable
from collections.abc import Iterator

T = TypeVar('T')
K = TypeVar('K')
V = TypeVar('V')

class Container(Generic[T]):
    def __init__(self) -> None:
        self._items: list[T] = []
    
    def add(self, item: T) -> None:
        self._items.append(item)
    
    def __iter__(self) -> Iterator[T]:
        return iter(self._items)

@runtime_checkable
class Drawable(Protocol):
    def draw(self, canvas: 'Canvas') -> None: ...
    @property
    def bounds(self) -> tuple[float, float, float, float]: ...

def render_all(objects: list[Drawable]) -> None:
    for obj in objects:
        obj.draw(canvas)
TypedDict & NewType
from typing import TypedDict, Required, NotRequired, NewType

class User(TypedDict, total=False):
    id: Required[int]
    name: Required[str]
    email: str  # Optional
    metadata: NotRequired[dict]

UserId = NewType('UserId', int)
def get_user(user_id: UserId) -> User:
    pass
Overload Signatures
from typing import overload

@overload
def process(value: int) -> str: ...
@overload
def process(value: str) -> int: ...
@overload
def process(value: None) -> None: ...

def process(value):
    if isinstance(value, int):
        return str(value)
    elif isinstance(value, str):
        return int(value)
    return value
🔹 Import System Internals
Custom Import Hooks
import sys
import importlib.util

class MyLoader(importlib.abc.Loader):
    def create_module(self, spec):
        return None
    
    def exec_module(self, module):
        code = compile(source, module.__file__, 'exec')
        exec(code, module.__dict__)

class MyFinder(importlib.abc.MetaPathFinder):
    def find_spec(self, fullname, path, target=None):
        if fullname.startswith('custom_'):
            spec = importlib.util.spec_from_loader(fullname, MyLoader())
            return spec
        return None

sys.meta_path.insert(0, MyFinder())
Package Entry Points
# pyproject.toml
[project.entry-points."myapp.plugins"]
plugin_a = "package.module:PluginA"
plugin_b = "package.module:PluginB"

# Runtime discovery
import importlib.metadata
entry_points = importlib.metadata.entry_points(group='myapp.plugins')
for ep in entry_points:
    plugin_class = ep.load()
🔹 Design Patterns
Dependency Injection
from abc import ABC, abstractmethod
from typing import Protocol

class Database(Protocol):
    def query(self, sql: str) -> list: ...

class Service:
    def __init__(self, db: Database):
        self.db = db
    
    def get_users(self):
        return self.db.query("SELECT * FROM users")

# Container
class DIContainer:
    def __init__(self):
        self._services = {}
    
    def register(self, interface, implementation):
        self._services[interface] = implementation
    
    def resolve(self, interface):
        return self._services[interface]()

container = DIContainer()
container.register(Database, PostgreSQLDB)
service = Service(container.resolve(Database))
Observer Pattern (Advanced)
from dataclasses import dataclass, field
from typing import Callable, Set

@dataclass
class Observable:
    _observers: Set[Callable] = field(default_factory=set)
    
    def subscribe(self, callback: Callable):
        self._observers.add(callback)
    
    def unsubscribe(self, callback: Callable):
        self._observers.discard(callback)
    
    def notify(self, *args, **kwargs):
        for observer in self._observers:
            try:
                observer(*args, **kwargs)
            except Exception as e:
                logger.error(f"Observer error: {e}")

# Event bus pattern
class EventBus:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance._events = {}
        return cls._instance
    
    def emit(self, event: str, data: dict):
        for handler in self._events.get(event, []):
            handler(data)
🔹 Testing Strategies
Advanced Mocking
from unittest.mock import AsyncMock, MagicMock, patch, call

# Async mocking
async_mock = AsyncMock(return_value={'status': 200})

# Side effects
mock.side_effect = [ValueError, TypeError, None]

# Assert call order
mock.assert_has_calls([call('arg1'), call('arg2')])

# Patch multiple
with patch.multiple('module', func1=MagicMock(), func2=MagicMock()):
    pass

# Mock context manager
mock.return_value.__enter__.return_value = mock_context
Property-Based Testing
from hypothesis import given, strategies as st

@given(st.integers(), st.integers())
def test_addition_commutative(a, b):
    assert a + b == b + a

@given(st.lists(st.integers(), min_size=1))
def test_max_not_less_than_any(nums):
    m = max(nums)
    assert all(m >= n for n in nums)
Integration Testing
import pytest
from httpx import AsyncClient

@pytest.fixture
async def client():
    async with AsyncClient(app=app, base_url="http://test") as ac:
        yield ac

@pytest.mark.asyncio
async def test_api_endpoint(client):
    response = await client.get("/api/users")
    assert response.status_code == 200
    assert response.json()["count"] > 0
🔹 Packaging & Distribution
Build Backend Configuration
# pyproject.toml
[build-system]
requires = ["hatchling", "hatch-vcs"]
build-backend = "hatchling.build"

[project]
name = "mypackage"
dynamic = ["version"]
dependencies = [
    "requests>=2.28.0,<3.0.0",
    "numpy>=1.21.0; python_version>='3.9'",
]

[project.scripts]
mycli = "mypackage.cli:main"

[tool.hatch.version]
source = "vcs"

[tool.hatch.build.targets.wheel]
packages = ["src/mypackage"]
Wheel Building
# Build wheel
python -m build --wheel

# Install from wheel
pip install dist/mypackage-0.1.0-py3-none-any.whl

# Publish to PyPI
twine upload dist/*
🔹 C Extensions & Performance
ctypes for C Interop
import ctypes

# Load C library
libc = ctypes.CDLL("libc.so.6")

# Define function signature
libc.printf.argtypes = [ctypes.c_char_p]
libc.printf.restype = ctypes.c_int

# Call C function
libc.printf(b"Hello from C!\n")

# Struct mapping
class Point(ctypes.Structure):
    _fields_ = [("x", ctypes.c_double), ("y", ctypes.c_double)]
Cython Compilation
# cython_file.pyx
# cython: language_level=3
# cython: boundscheck=False
# cython: wraparound=False

def fast_sum(double[:] arr):
    cdef int i
    cdef double total = 0.0
    for i in range(arr.shape[0]):
        total += arr[i]
    return total
🔹 Debugging & Diagnostics
Advanced pdb
import pdb

# Conditional breakpoint
if condition:
    pdb.set_trace()

# Post-mortem debugging
try:
    risky_operation()
except:
    import traceback
    traceback.print_exc()
    pdb.post_mortem()

# IPython debugger
from IPython.core.debugger import set_trace
set_trace()
Logging Best Practices
import logging
import logging.handlers

# Rotating file handler
handler = logging.handlers.RotatingFileHandler(
    'app.log', maxBytes=10*1024*1024, backupCount=5
)

# Structured logging
import structlog
structlog.configure(
    processors=[
        structlog.stdlib.filter_by_level,
        structlog.stdlib.add_logger_name,
        structlog.processors.JSONRenderer()
    ]
)

# Context-aware logging
logger = structlog.get_logger()
with logger.contextualize(user_id=user.id):
    logger.info("request_processed")
🔹 Common Pitfalls & Gotchas
# Mutable default argument (WRONG)
def append_to(item, lst=[]):
    lst.append(item)
    return lst

# Fixed version
def append_to(item, lst=None):
    if lst is None:
        lst = []
    lst.append(item)
    return lst

# Late binding in closures
def create_functions():
    return [lambda x: x * i for i in range(5)]  # All return 4*x

# Fixed
def create_functions():
    return [lambda x, i=i: x * i for i in range(5)]

# Float precision issues
from decimal import Decimal
result = Decimal('0.1') + Decimal('0.2')  # 0.3 exactly

# GIL awareness
# CPU-bound: use multiprocessing
# I/O-bound: use threading/asyncio
🔹 Quick One-Liners
# Deep copy without import
import copy
deep = copy.deepcopy(obj)

# Merge dictionaries (Python 3.9+)
merged = dict1 | dict2 | dict3

# Group by key
from itertools import groupby
grouped = {k: list(g) for k, g in groupby(sorted(items, key=key), key=key)}

# Top N items
import heapq
top_n = heapq.nlargest(10, items, key=score)

# Flatten nested list
flat = [x for sublist in nested for x in sublist]

# Chunk list
chunks = [lst[i:i+n] for i in range(0, len(lst), n)]

# Unique while preserving order
unique = list(dict.fromkeys(items))

# Partition by predicate
from itertools import tee
def partition(pred, iterable):
    t1, t2 = tee(iterable)
    return filterfalse(pred, t1), filter(pred, t2)