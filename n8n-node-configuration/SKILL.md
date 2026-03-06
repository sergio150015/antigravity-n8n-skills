---
name: n8n-node-configuration
description: Operation-aware node configuration guidance. Use when configuring nodes, understanding property dependencies, determining required fields, choosing between get_node detail levels, or learning common configuration patterns by node type.
---

# n8n Node Configuration

Expert guidance for operation-aware node configuration with property dependencies.

---

## Configuration Philosophy

**Progressive disclosure**: Start minimal, add complexity as needed

Configuration best practices:
- `get_node` with `detail: "standard"` is the most used discovery pattern
- 56 seconds average between configuration edits
- Covers 95% of use cases with 1-2K tokens response

**Key insight**: Most configurations need only standard detail, not full schema!

---

## Core Concepts

### 1. Operation-Aware Configuration

**Not all fields are always required** - it depends on operation!

**Example**: Slack node
```javascript
// For operation='post'
{
  "resource": "message",
  "operation": "post",
  "channel": "#general",  // Required for post
  "text": "Hello!"        // Required for post
}

// For operation='update'
{
  "resource": "message",
  "operation": "update",
  "messageId": "123",     // Required for update (different!)
  "text": "Updated!"      // Required for update
  // channel NOT required for update
}
```

**Key**: Resource + operation determine which fields are required!

### 2. Property Dependencies

**Fields appear/disappear based on other field values**

**Example**: HTTP Request node
```javascript
// When method='GET'
{
  "method": "GET",
  "url": "https://api.example.com"
  // sendBody not shown (GET doesn't have body)
}

// When method='POST'
{
  "method": "POST",
  "url": "https://api.example.com",
  "sendBody": true,       // Now visible!
  "body": {               // Required when sendBody=true
    "contentType": "json",
    "content": {...}
  }
}
```

**Mechanism**: displayOptions control field visibility

### 3. Progressive Discovery

**Use the right detail level**:

1. **get_node({detail: "standard"})** - DEFAULT
   - Quick overview (~1-2K tokens)
   - Required fields + common options
   - **Use first** - covers 95% of needs

2. **get_node({mode: "search_properties", propertyQuery: "..."})** (for finding specific fields)
   - Find properties by name
   - Use when looking for auth, body, headers, etc.

3. **get_node({detail: "full"})** (complete schema)
   - All properties (~3-8K tokens)
   - Use only when standard detail is insufficient

---

## Configuration Workflow

### Standard Process

```
1. Identify node type and operation
   |
2. Use get_node (standard detail is default)
   |
3. Configure required fields
   |
4. Validate configuration
   |
5. If field unclear -> get_node({mode: "search_properties"})
   |
6. Add optional fields as needed
   |
7. Validate again
   |
8. Deploy
```

### Example: Configuring HTTP Request

**Step 1**: Identify what you need
```javascript
// Goal: POST JSON to API
```

**Step 2**: Get node info
```javascript
const info = get_node({
  nodeType: "nodes-base.httpRequest"
});
// Returns: method, url, sendBody, body, authentication required/optional
```

**Step 3**: Minimal config
```javascript
{
  "method": "POST",
  "url": "https://api.example.com/create",
  "authentication": "none"
}
```

**Step 4**: Validate -> Fix -> Validate again (iterate until valid)
```javascript
validate_node({nodeType: "nodes-base.httpRequest", config, profile: "runtime"});
// Add sendBody, body, etc. as validation errors guide you
```

---

## get_node Detail Levels

### Standard Detail (DEFAULT - Use This!)

```javascript
get_node({
  nodeType: "nodes-base.slack"
});
// detail="standard" is the default
```

**Returns** (~1-2K tokens): Required fields, common options, operation list, metadata
**Use**: 95% of configuration needs

### Full Detail (Use Sparingly)

```javascript
get_node({
  nodeType: "nodes-base.slack",
  detail: "full"
});
```

**Returns** (~3-8K tokens): Complete schema, all properties, all nested options
**Warning**: Large response, use only when standard insufficient

### Search Properties Mode

```javascript
get_node({
  nodeType: "nodes-base.httpRequest",
  mode: "search_properties",
  propertyQuery: "auth"
});
```

**Use**: Find authentication, headers, body fields, etc.

### Decision Tree

```
Starting new node config?
  YES -> get_node (standard)
    Standard has what you need?
      YES -> Configure with it
      NO  -> Looking for specific field?
        YES -> search_properties mode
        NO  -> get_node({detail: "full"})
```

---

## Property Dependencies Deep Dive

### displayOptions Mechanism

**Fields have visibility rules**:

```javascript
{
  "name": "body",
  "displayOptions": {
    "show": {
      "sendBody": [true],
      "method": ["POST", "PUT", "PATCH"]
    }
  }
}
```

**Translation**: "body" field shows when sendBody = true AND method = POST, PUT, or PATCH

### Common Dependency Patterns

| Pattern | Example | Mechanism |
|---------|---------|-----------|
| **Boolean Toggle** | HTTP Request: sendBody -> body appears | sendBody=true toggles body visibility |
| **Operation Switch** | Slack: post needs channel, update needs messageId | resource+operation change required fields |
| **Type Selection** | IF node: string/boolean change available operators | type determines operator options |

### Finding Property Dependencies

```javascript
// Search for a specific property
get_node({
  nodeType: "nodes-base.httpRequest",
  mode: "search_properties",
  propertyQuery: "body"
});

// Or use full detail for complete schema
get_node({
  nodeType: "nodes-base.httpRequest",
  detail: "full"
});
```

**Use this when**: Validation fails and you don't understand why field is missing/required

---

## Common Node Patterns

### Pattern 1: Resource/Operation Nodes

**Examples**: Slack, Google Sheets, Airtable

```javascript
{
  "resource": "<entity>",      // What type of thing
  "operation": "<action>",     // What to do with it
  // ... operation-specific fields
}
```

### Pattern 2: HTTP-Based Nodes

**Examples**: HTTP Request, Webhook

**Dependencies**: POST/PUT/PATCH -> sendBody available; sendBody=true -> body required; authentication != "none" -> credentials required

### Pattern 3: Database Nodes

**Examples**: Postgres, MySQL, MongoDB

**Dependencies**: executeQuery -> query required; insert -> table + values required; update -> table + values + where required

### Pattern 4: Conditional Logic Nodes

**Examples**: IF, Switch, Merge

**Dependencies**: Binary operators (equals, contains) -> value1 + value2; Unary operators (isEmpty) -> value1 only + singleValue: true

---

## Best Practices

### Do

1. **Start with get_node (standard detail)** - ~1-2K tokens, covers 95% of needs
2. **Validate iteratively** - Configure -> Validate -> Fix -> Repeat (avg 2-3 iterations)
3. **Use search_properties mode when stuck** - understand what controls field visibility
4. **Respect operation context** - different operations = different requirements
5. **Trust auto-sanitization** - operator structure fixed automatically

### Don't

1. **Jump to detail="full" immediately** - try standard first (full = 3-8K tokens)
2. **Configure blindly** - always validate before deploying
3. **Copy configs without understanding** - validate after copying
4. **Manually fix auto-sanitization issues** - let the system handle it

---

## Detailed References

For comprehensive guides on specific topics:

- **[DEPENDENCIES.md](DEPENDENCIES.md)** - Deep dive into property dependencies and displayOptions
- **[OPERATION_PATTERNS.md](OPERATION_PATTERNS.md)** - Common configuration patterns by node type
- **[ADVANCED_EXAMPLES.md](ADVANCED_EXAMPLES.md)** - Operation-specific examples (Slack, HTTP, IF), conditional requirements, and anti-patterns

---

## Summary

**Configuration Strategy**:
1. Start with `get_node` (standard detail is default)
2. Configure required fields for operation
3. Validate configuration
4. Search properties if stuck
5. Iterate until valid (avg 2-3 cycles)
6. Deploy with confidence

**Key Principles**:
- **Operation-aware**: Different operations = different requirements
- **Progressive disclosure**: Start minimal, add as needed
- **Dependency-aware**: Understand field visibility rules
- **Validation-driven**: Let validation guide configuration

**Related Skills**:
- **n8n MCP Tools Expert** - How to use discovery tools correctly
- **n8n Validation Expert** - Interpret validation errors
- **n8n Expression Syntax** - Configure expression fields
- **n8n Workflow Patterns** - Apply patterns with proper configuration
