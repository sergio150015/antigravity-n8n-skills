# Advanced Python Patterns for n8n Code Nodes

Detailed patterns, standard library reference, and best practices.

---

## Table of Contents

- [Common Patterns (Detailed)](#common-patterns-detailed)
  - [Data Transformation](#1-data-transformation)
  - [Filtering and Aggregation](#2-filtering--aggregation)
  - [String Processing with Regex](#3-string-processing-with-regex)
  - [Data Validation](#4-data-validation)
  - [Statistical Analysis](#5-statistical-analysis)
- [Standard Library Reference](#standard-library-reference)
- [Best Practices (Detailed)](#best-practices-detailed)
  - [Use .get() for Dictionary Access](#1-always-use-get-for-dictionary-access)
  - [Handle None/Null Values](#2-handle-nonenull-values-explicitly)
  - [Use List Comprehensions](#3-use-list-comprehensions-for-filtering)
  - [Return Consistent Structure](#4-return-consistent-structure)
  - [Debug with print()](#5-debug-with-print-statements)
- [When to Use Python vs JavaScript](#when-to-use-python-vs-javascript)

---

## Common Patterns (Detailed)

### 1. Data Transformation
Transform all items with list comprehensions

```python
items = _input.all()

return [
    {
        "json": {
            "id": item["json"].get("id"),
            "name": item["json"].get("name", "Unknown").upper(),
            "processed": True
        }
    }
    for item in items
]
```

### 2. Filtering & Aggregation
Sum, filter, count with built-in functions

```python
items = _input.all()
total = sum(item["json"].get("amount", 0) for item in items)
valid_items = [item for item in items if item["json"].get("amount", 0) > 0]

return [{
    "json": {
        "total": total,
        "count": len(valid_items)
    }
}]
```

### 3. String Processing with Regex
Extract patterns from text

```python
import re

items = _input.all()
email_pattern = r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b'

all_emails = []
for item in items:
    text = item["json"].get("text", "")
    emails = re.findall(email_pattern, text)
    all_emails.extend(emails)

# Remove duplicates
unique_emails = list(set(all_emails))

return [{
    "json": {
        "emails": unique_emails,
        "count": len(unique_emails)
    }
}]
```

### 4. Data Validation
Validate and clean data

```python
items = _input.all()
validated = []

for item in items:
    data = item["json"]
    errors = []

    # Validate fields
    if not data.get("email"):
        errors.append("Email required")
    if not data.get("name"):
        errors.append("Name required")

    validated.append({
        "json": {
            **data,
            "valid": len(errors) == 0,
            "errors": errors if errors else None
        }
    })

return validated
```

### 5. Statistical Analysis
Calculate statistics with statistics module

```python
from statistics import mean, median, stdev

items = _input.all()
values = [item["json"].get("value", 0) for item in items if "value" in item["json"]]

if values:
    return [{
        "json": {
            "mean": mean(values),
            "median": median(values),
            "stdev": stdev(values) if len(values) > 1 else 0,
            "min": min(values),
            "max": max(values),
            "count": len(values)
        }
    }]
else:
    return [{"json": {"error": "No values found"}}]
```

See [COMMON_PATTERNS.md](COMMON_PATTERNS.md) for 10 detailed Python patterns.

---

## Standard Library Reference

### Most Useful Modules

```python
# JSON operations
import json
data = json.loads(json_string)
json_output = json.dumps({"key": "value"})

# Date/time
from datetime import datetime, timedelta
now = datetime.now()
tomorrow = now + timedelta(days=1)
formatted = now.strftime("%Y-%m-%d")

# Regular expressions
import re
matches = re.findall(r'\d+', text)
cleaned = re.sub(r'[^\w\s]', '', text)

# Base64 encoding
import base64
encoded = base64.b64encode(data).decode()
decoded = base64.b64decode(encoded)

# Hashing
import hashlib
hash_value = hashlib.sha256(text.encode()).hexdigest()

# URL parsing
import urllib.parse
params = urllib.parse.urlencode({"key": "value"})
parsed = urllib.parse.urlparse(url)

# Statistics
from statistics import mean, median, stdev
average = mean([1, 2, 3, 4, 5])
```

See [STANDARD_LIBRARY.md](STANDARD_LIBRARY.md) for complete reference.

---

## Best Practices (Detailed)

### 1. Always Use .get() for Dictionary Access

```python
# SAFE: Won't crash if field missing
value = item["json"].get("field", "default")

# RISKY: Crashes if field doesn't exist
value = item["json"]["field"]
```

### 2. Handle None/Null Values Explicitly

```python
# GOOD: Default to 0 if None
amount = item["json"].get("amount") or 0

# GOOD: Check for None explicitly
text = item["json"].get("text")
if text is None:
    text = ""
```

### 3. Use List Comprehensions for Filtering

```python
# PYTHONIC: List comprehension
valid = [item for item in items if item["json"].get("active")]

# VERBOSE: Manual loop
valid = []
for item in items:
    if item["json"].get("active"):
        valid.append(item)
```

### 4. Return Consistent Structure

```python
# CONSISTENT: Always list with "json" key
return [{"json": result}]  # Single result
return results  # Multiple results (already formatted)
return []  # No results
```

### 5. Debug with print() Statements

```python
# Debug statements appear in browser console (F12)
items = _input.all()
print(f"Processing {len(items)} items")
print(f"First item: {items[0] if items else 'None'}")
```

---

## When to Use Python vs JavaScript

### Use Python When:
- You need `statistics` module for statistical operations
- You're significantly more comfortable with Python syntax
- Your logic maps well to list comprehensions
- You need specific standard library functions

### Use JavaScript When:
- You need HTTP requests ($helpers.httpRequest())
- You need advanced date/time (DateTime/Luxon)
- You want better n8n integration
- **For 95% of use cases** (recommended)

### Consider Other Nodes When:
- Simple field mapping -> Use **Set** node
- Basic filtering -> Use **Filter** node
- Simple conditionals -> Use **IF** or **Switch** node
- HTTP requests only -> Use **HTTP Request** node
