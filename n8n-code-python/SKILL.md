---
name: n8n-code-python
description: Write Python code in n8n Code nodes. Use when writing Python in n8n, using _input/_json/_node syntax, working with standard library, or need to understand Python limitations in n8n Code nodes.
---

# Python Code Node (Beta)

Expert guidance for writing Python code in n8n Code nodes.

---

## Important: JavaScript First

**Recommendation**: Use **JavaScript for 95% of use cases**. Only use Python when:
- You need specific Python standard library functions
- You're significantly more comfortable with Python syntax
- You're doing data transformations better suited to Python

**Why JavaScript is preferred**: Full n8n helper functions, Luxon DateTime, no library limitations, better community support.

---

## Quick Start

```python
# Basic template for Python Code nodes
items = _input.all()

# Process data
processed = []
for item in items:
    processed.append({
        "json": {
            **item["json"],
            "processed": True,
            "timestamp": datetime.now().isoformat()
        }
    })

return processed
```

### Essential Rules

1. **Consider JavaScript first** - Use Python only when necessary
2. **Access data**: `_input.all()`, `_input.first()`, or `_input.item`
3. **CRITICAL**: Must return `[{"json": {...}}]` format
4. **CRITICAL**: Webhook data is under `_json["body"]` (not `_json` directly)
5. **CRITICAL LIMITATION**: **No external libraries** (no requests, pandas, numpy)
6. **Standard library only**: json, datetime, re, base64, hashlib, urllib.parse, math, random, statistics

---

## Mode Selection Guide

### Run Once for All Items (Recommended - Default)

**Use this mode for:** 95% of use cases

- Code executes **once** regardless of input count
- Data access: `_input.all()` or `_items` array (Native mode)
- Best for: Aggregation, filtering, batch processing, transformations

```python
all_items = _input.all()
total = sum(item["json"].get("amount", 0) for item in all_items)

return [{
    "json": {
        "total": total,
        "count": len(all_items),
        "average": total / len(all_items) if all_items else 0
    }
}]
```

### Run Once for Each Item

**Use this mode for:** Specialized cases only

- Code executes **separately** for each input item
- Data access: `_input.item` or `_item` (Native mode)
- Best for: Item-specific logic, independent operations, per-item validation

```python
item = _input.item

return [{
    "json": {
        **item["json"],
        "processed": True,
        "processed_at": datetime.now().isoformat()
    }
}]
```

---

## Python Modes: Beta vs Native

### Python (Beta) - Recommended
- **Use**: `_input`, `_json`, `_node` helper syntax
- **Helpers available**: `_now`, `_today`, `_jmespath()`

### Python (Native) (Beta)
- **Use**: `_items`, `_item` variables only
- **No helpers**: No `_input`, `_now`, etc.
- **Use when**: Need pure Python without n8n helpers

**Recommendation**: Use **Python (Beta)** for better n8n integration.

---

## Data Access Patterns

| Pattern | Mode | Use When |
|---------|------|----------|
| `_input.all()` | All Items | Processing arrays, batch operations, aggregations |
| `_input.first()` | All Items | Working with single objects, API responses |
| `_input.item` | Each Item | In "Run Once for Each Item" mode |
| `_node["Name"]["json"]` | Any | Need data from specific nodes in workflow |

**See**: [DATA_ACCESS.md](DATA_ACCESS.md) for comprehensive guide with examples

---

## Critical: Webhook Data Structure

**MOST COMMON MISTAKE**: Webhook data is nested under `["body"]`

```python
# WRONG - Will raise KeyError
name = _json["name"]

# CORRECT - Webhook data is under ["body"]
name = _json["body"]["name"]

# SAFER - Use .get() for safe access
webhook_data = _json.get("body", {})
name = webhook_data.get("name")
```

---

## Return Format Requirements

**CRITICAL RULE**: Always return list of dictionaries with `"json"` key

```python
# Single result
return [{"json": {"field1": value1}}]

# Multiple results
return [{"json": {"id": 1}}, {"json": {"id": 2}}]

# Empty result
return []
```

**Wrong formats** (will cause workflow failure):
- `return {"json": {...}}` - missing list wrapper
- `return [{"field": value}]` - missing json wrapper
- No return statement

**See**: [ERROR_PATTERNS.md](ERROR_PATTERNS.md) for detailed error solutions

---

## Critical Limitation: No External Libraries

**Cannot import external packages** - only standard library:

| Available | NOT Available |
|-----------|---------------|
| json, datetime, re | requests, pandas, numpy |
| base64, hashlib, urllib.parse | scipy, BeautifulSoup, lxml |
| math, random, statistics | Any pip package |

**Workarounds**: HTTP Request node (instead of requests), JavaScript `$helpers.httpRequest()`, HTML Extract node (instead of BeautifulSoup)

**See**: [STANDARD_LIBRARY.md](STANDARD_LIBRARY.md) for complete reference

---

## Error Prevention - Top 5 Mistakes

| # | Mistake | Fix |
|---|---------|-----|
| 1 | `import requests` | Use HTTP Request node or switch to JavaScript |
| 2 | No return statement | Always `return [{"json": {...}}]` |
| 3 | `return {"json": {...}}` | Wrap in list: `return [{"json": {...}}]` |
| 4 | `_json["user"]["name"]` | Use `.get()`: `_json.get("user", {}).get("name")` |
| 5 | `_json["email"]` (webhook) | Use `_json["body"]["email"]` |

**See**: [ERROR_PATTERNS.md](ERROR_PATTERNS.md) for comprehensive error guide

---

## Quick Reference Checklist

Before deploying Python Code nodes, verify:

- [ ] **Considered JavaScript first** - Using Python only when necessary
- [ ] **Return statement exists** - Must return list of dictionaries
- [ ] **Proper return format** - Each item: `{"json": {...}}`
- [ ] **Data access correct** - Using `_input.all()`, `_input.first()`, or `_input.item`
- [ ] **No external imports** - Only standard library
- [ ] **Safe dictionary access** - Using `.get()` to avoid KeyError
- [ ] **Webhook data** - Access via `["body"]` if from webhook
- [ ] **Mode selection** - "All Items" for most cases

---

## Additional Resources

### Reference Files
- [DATA_ACCESS.md](DATA_ACCESS.md) - Comprehensive Python data access patterns
- [COMMON_PATTERNS.md](COMMON_PATTERNS.md) - 10 Python patterns for n8n
- [ERROR_PATTERNS.md](ERROR_PATTERNS.md) - Top 5 errors and solutions
- [STANDARD_LIBRARY.md](STANDARD_LIBRARY.md) - Complete standard library reference
- [ADVANCED_PATTERNS.md](ADVANCED_PATTERNS.md) - Detailed patterns, stdlib quick-ref, best practices, Python vs JS comparison

### Integration with Other Skills
- **n8n Expression Syntax**: Expressions use `{{ }}` in other nodes; Code nodes use Python directly
- **n8n MCP Tools Expert**: Find Code node with `search_nodes({query: "code"})`
- **n8n Node Configuration**: Mode selection, language selection
- **n8n Code JavaScript**: When to use JavaScript instead
