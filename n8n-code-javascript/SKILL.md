---
name: n8n-code-javascript
description: Write JavaScript code in n8n Code nodes. Use when writing JavaScript in n8n, using $input/$json/$node syntax, making HTTP requests with $helpers, working with dates using DateTime, troubleshooting Code node errors, or choosing between Code node modes.
---

# JavaScript Code Node

Expert guidance for writing JavaScript code in n8n Code nodes.

---

## Quick Start

```javascript
// Basic template for Code nodes
const items = $input.all();

// Process data
const processed = items.map(item => ({
  json: {
    ...item.json,
    processed: true,
    timestamp: new Date().toISOString()
  }
}));

return processed;
```

### Essential Rules

1. **Choose "Run Once for All Items" mode** (recommended for most use cases)
2. **Access data**: `$input.all()`, `$input.first()`, or `$input.item`
3. **CRITICAL**: Must return `[{json: {...}}]` format
4. **CRITICAL**: Webhook data is under `$json.body` (not `$json` directly)
5. **Built-ins available**: $helpers.httpRequest(), DateTime (Luxon), $jmespath()

---

## Mode Selection Guide

### Run Once for All Items (Recommended - Default)

**Use this mode for:** 95% of use cases

- Code executes **once** regardless of input count
- Data access: `$input.all()` or `items` array
- Best for: Aggregation, filtering, batch processing, transformations, API calls

```javascript
const allItems = $input.all();
const total = allItems.reduce((sum, item) => sum + (item.json.amount || 0), 0);

return [{
  json: {
    total,
    count: allItems.length,
    average: total / allItems.length
  }
}];
```

### Run Once for Each Item

**Use this mode for:** Specialized cases only

- Code executes **separately** for each input item
- Data access: `$input.item` or `$item`
- Best for: Item-specific logic, independent operations, per-item validation

```javascript
const item = $input.item;

return [{
  json: {
    ...item.json,
    processed: true,
    processedAt: new Date().toISOString()
  }
}];
```

**Decision Shortcut:**
- **Need to look at multiple items?** -> Use "All Items" mode
- **Each item completely independent?** -> Use "Each Item" mode
- **Not sure?** -> Use "All Items" mode

---

## Data Access Patterns

| Pattern | Mode | Use When |
|---------|------|----------|
| `$input.all()` | All Items | Processing arrays, batch operations, aggregations |
| `$input.first()` | All Items | Working with single objects, API responses |
| `$input.item` | Each Item | In "Run Once for Each Item" mode |
| `$node["Name"].json` | Any | Need data from specific nodes in workflow |

**See**: [DATA_ACCESS.md](DATA_ACCESS.md) for comprehensive guide with examples

---

## Critical: Webhook Data Structure

**MOST COMMON MISTAKE**: Webhook data is nested under `.body`

```javascript
// WRONG - Will return undefined
const name = $json.name;

// CORRECT - Webhook data is under .body
const name = $json.body.name;

// Or with $input
const webhookData = $input.first().json.body;
```

---

## Return Format Requirements

**CRITICAL RULE**: Always return array of objects with `json` property

```javascript
// Single result
return [{json: {field1: value1}}];

// Multiple results
return [{json: {id: 1}}, {json: {id: 2}}];

// Empty result
return [];
```

**Wrong formats** (will cause workflow failure):
- `return {json: {...}}` - missing array wrapper
- `return [{field: value}]` - missing json wrapper
- `return "processed"` - plain string
- No return statement

**See**: [ERROR_PATTERNS.md](ERROR_PATTERNS.md) for detailed error solutions

---

## Error Prevention - Top 5 Mistakes

| # | Mistake | Fix |
|---|---------|-----|
| 1 | No return statement | Always `return [{json: {...}}]` |
| 2 | `"{{ $json.field }}"` in code | Use `$input.first().json.field` |
| 3 | `return {json: {...}}` | Wrap in array: `return [{json: {...}}]` |
| 4 | `item.json.user.email` | Use `item.json?.user?.email` |
| 5 | `$json.email` (webhook) | Use `$json.body.email` |

**See**: [ERROR_PATTERNS.md](ERROR_PATTERNS.md) for comprehensive error guide

---

## When to Use Code Node

**Use Code node when:**
- Complex transformations requiring multiple steps
- Custom calculations or business logic
- Recursive operations, API response parsing
- Data aggregation across items

**Consider other nodes when:**
- Simple field mapping -> **Set** node
- Basic filtering -> **Filter** node
- Simple conditionals -> **IF** or **Switch** node
- HTTP requests only -> **HTTP Request** node

---

## Quick Reference Checklist

Before deploying Code nodes, verify:

- [ ] **Code is not empty** - Must have meaningful logic
- [ ] **Return statement exists** - Must return array of objects
- [ ] **Proper return format** - Each item: `{json: {...}}`
- [ ] **Data access correct** - Using `$input.all()`, `$input.first()`, or `$input.item`
- [ ] **No n8n expressions** - Use JavaScript template literals: `` `${value}` ``
- [ ] **Error handling** - Guard clauses for null/undefined inputs
- [ ] **Webhook data** - Access via `.body` if from webhook
- [ ] **Mode selection** - "All Items" for most cases

---

## Additional Resources

### Reference Files
- [DATA_ACCESS.md](DATA_ACCESS.md) - Comprehensive data access patterns
- [COMMON_PATTERNS.md](COMMON_PATTERNS.md) - 10 production-tested patterns
- [ERROR_PATTERNS.md](ERROR_PATTERNS.md) - Top 5 errors and solutions
- [BUILTIN_FUNCTIONS.md](BUILTIN_FUNCTIONS.md) - Complete built-in reference
- [ADVANCED_PATTERNS.md](ADVANCED_PATTERNS.md) - Detailed patterns, built-in functions ($helpers, DateTime, $jmespath), best practices

### Integration with Other Skills
- **n8n Expression Syntax**: Expressions use `{{ }}` in other nodes; Code nodes use JavaScript directly
- **n8n MCP Tools Expert**: Find Code node with `search_nodes({query: "code"})`
- **n8n Node Configuration**: Mode selection, language selection
- **n8n Validation Expert**: Validate Code node configuration
