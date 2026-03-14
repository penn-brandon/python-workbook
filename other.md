# Python Survival Guide Cheatsheet - Other Cheatsheet
    What the Beginner/Medium/Advanced Sheets Missed: Ops, Security, Modern Tooling, and the "Gotchas" That Cost Money
    The other three cheatsheets teach you to write Python. 
    This one teaches you to ship it without burning down the server. 
    Keep this handy when the "it works on my machine" phase ends.

## Observability & Production Debugging

*Beginner/Medium/Advanced sheets covered `print` and `pdb`. Real life requires structured logs and tracing.*

### Structured Logging (JSON)

Stop printing strings. Parseable logs are non-negotiable for ELK/Splunk/Datadog.

```python
import logging
import json
from pythonjsonlogger import jsonlogger

logger = logging.getLogger(__name__)
handler = logging.StreamHandler()
formatter = jsonlogger.JsonFormatter()
handler.setFormatter(formatter)
logger.addHandler(handler)
logger.setLevel(logging.INFO)

# Usage
logger.info("User login failed", extra={"user_id": 123, "ip": "1.2.3.4", "reason": "invalid_token"}) 
    
Output: {"message": "User login failed", "user_id": 123, "ip": "1.2.3.4", "reason": "invalid_token"}
```

### Distributed Tracing (OpenTelemetry)
When microservices fail, print is useless. You need traces.

```python
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter

trace.set_tracer_provider(TracerProvider())
tracer = trace.get_tracer(__name__)

with tracer.start_as_current_span("process_order") as span:
    span.set_attribute("order_id", "12345")
    # ... logic ...
    span.record_exception(e)  # Auto-capture errors
```
## Security Hygiene (Beyond try/except)
The other sheets showed ValueError. They didn't show you how to stop getting hacked.

### Secrets Management
NEVER hardcode secrets. Even in .env files in git repos.
```python
from dotenv import load_dotenv
import os

load_dotenv()  # Loads .env into os.environ

API_KEY = os.getenv("API_KEY")
if not API_KEY:
    raise RuntimeError("API_KEY missing! Check env vars.")

# Better: Use a secret manager in prod (AWS Secrets Manager, HashiCorp Vault)
# from aws_secretsmanager_caching import SecretCache
```
### Input Sanitization & SQL Injection
The advanced sheet showed dataclasses, not SQL safety.

```python
# WRONG (String interpolation)
cursor.execute(f"SELECT * FROM users WHERE id = {user_id}")

# RIGHT (Parameterized queries)
cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))

# For HTML/JS: Always escape output
from html import escape
safe_input = escape(user_input)
```
### Dependency Scanning
```python
# Check for known CVEs in your requirements
pip install safety
safety check

# Or use pip-audit (faster, modern)
pip install pip-audit
pip-audit
```
## Modern Tooling & Workflow
The sheets mentioned pytest and black. They missed the ecosystem that makes them useful.

### Pre-commit Hooks
Run linters/formatters before you commit. Stop wasting time fixing style in PRs.
```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.1.6
    hooks:
      - id: ruff
        args: [--fix]
      - id: ruff-format
```
### Virtual Environments (The Right Way)
venv is fine, but uv or poetry is the future.
```yaml
# Using uv (Rust-based, instant)
uv venv
source .venv/bin/activate
uv pip install -r requirements.txt

# Using Poetry (Dependency resolution king)
poetry init
poetry add requests
poetry run python script.py
```
### Type Checking Rigor
Advanced sheet showed typing. It missed enforcing it.
```yaml
# mypy.ini
[mypy]
strict = true
warn_return_any = true
disallow_untyped_defs = true

# Run it
mypy src/
```
## Deployment & Containerization
Nowhere in the previous sheets did we talk about Dockerizing Python properly.

### Multi-Stage Dockerfile
Don't ship your dev dependencies to production.
```BASH
# Stage 1: Builder
FROM python:3.12-slim as builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# Stage 2: Runner
FROM python:3.12-slim
WORKDIR /app
COPY --from=builder /root/.local /root/.local
COPY . .
ENV PATH=/root/.local/bin:$PATH
CMD ["gunicorn", "app:app"]
```

### Health Checks
Docker/K8s need to know if your app is actually alive.

```python
from fastapi import FastAPI
app = FastAPI()

@app.get("/health")
def health_check():
    return {"status": "healthy", "db": check_db_connection()}
```
## The "Silent Killers" (Gotchas)
These are the bugs that don't throw exceptions but ruin performance.

### The GIL & CPU Bound Tasks
Advanced sheet mentioned multiprocessing. It missed the nuance.

* I/O Bound: asyncio or threading (GIL releases on I/O)
* CPU Bound: multiprocessing (GIL blocks threads)
* Hybrid: Use concurrent.futures with a mix of executors
* Memory Leaks in Caches
* Advanced sheet showed WeakValueDictionary. It missed LRU limits.
```python
from functools import lru_cache

# BAD: Unbounded cache grows forever
@lru_cache(maxsize=None)
def expensive_calc(x): ...

# GOOD: Limit size
@lru_cache(maxsize=128)
def expensive_calc(x): ...
```

### Circular Imports
The sheets showed imports. They didn't show how to fix circular deps.
```python
# module_a.py
from module_b import func_b  # ERROR if module_b imports module_a

# FIX: Move import inside function (lazy loading)
def func_a():
    from module_b import func_b
    return func_b()
```
### Quick "Ops" One-Liners
```python
# Measure execution time (decorator style)
import time

def timer(func):
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        res = func(*args, **kwargs)
        print(f"{func.__name__} took {time.perf_counter() - start:.4f}s")
        return res
    return wrapper

# Check disk space before writing
import shutil
free_space = shutil.disk_usage('/').free
if free_space < 1024**3:  # Less than 1GB
    raise OSError("Disk full!")

# Graceful shutdown signal handling
import signal, sys

def handle_sigterm(signum, frame):
    print("Shutting down gracefully...")
    cleanup_resources()
    sys.exit(0)

signal.signal(signal.SIGTERM, handle_sigterm)
```
## Recommended Stack for 2026
If you're building something new today, this is the baseline:

| Category | Recommendation | Notes |
|----------|----------|----------|
Runtime | Python 3.14.3+ | Faster startup, better typing |
Web Framework | FastAPI or Starlette | Async native, auto-docs |
ORM | SQLAlchemy 2.0 or Tortoise ORM | Async support |
Task Queue | Celery or ARQ | Redis/RabbitMQ vs Redis only |
Testing | pytest + pytest-asyncio + hypothesis | Property-based testing |
Linting | ruff |Replaces flake8, black, isort, pylint |
Packaging |	uv or hatch | Modern dependency management |