---
name: n8n-expression-syntax
description: Validate n8n expression syntax and fix common errors. Use when writing n8n expressions, using {{}} syntax, accessing $json/$node variables, troubleshooting expression errors, or working with webhook data in workflows.
---

# n8n Expression Syntax

Expert guide for writing correct n8n expressions in workflows.

---

## Expression Format

All dynamic content in n8n uses **double curly braces**:

```
{{expression}}
```

**Examples**:
```
{{$json.email}}
{{$json.body.name}}
{{$node["HTTP Request"].json.data}}
$json.email  (no braces - treated as literal text)
{$json.email}  (single braces - invalid)
```

---

## Core Variables

### $json - Current Node Output

```javascript
{{$json.fieldName}}
{{$json['field with spaces']}}
{{$json.nested.property}}
{{$json.items[0].name}}
```

### $node - Reference Other Nodes

```javascript
{{$node["Node Name"].json.fieldName}}
{{$node["HTTP Request"].json.data}}
{{$node["Webhook"].json.body.email}}
```

**Important**: Node names **must** be in quotes, are **case-sensitive**, must match exact name.

### $now - Current Timestamp

```javascript
{{$now}}
{{$now.toFormat('yyyy-MM-dd')}}
{{$now.plus({days: 7})}}
```

### $env - Environment Variables

```javascript
{{$env.API_KEY}}
```

---

## CRITICAL: Webhook Data Structure

**Most Common Mistake**: Webhook data is **NOT** at the root!

```javascript
// Webhook Node Output Structure
{
  "headers": {...},
  "params": {...},
  "query": {...},
  "body": {           // USER DATA IS HERE!
    "name": "John",
    "email": "john@example.com"
  }
}
```

```javascript
// WRONG:
{{$json.name}}

// CORRECT:
{{$json.body.name}}
{{$json.body.email}}
```

---

## Common Patterns

### Access Nested Fields

```javascript
{{$json.user.email}}
{{$json.data[0].name}}
{{$json['field name']}}
```

### Reference Other Nodes

```javascript
{{$node["Set"].json.value}}
{{$node["HTTP Request"].json.data}}
{{$node["Webhook"].json.body.email}}
```

### Combine Variables

```javascript
// In text
Hello {{$json.body.name}}!

// In URLs
https://api.example.com/users/{{$json.body.user_id}}

// In object properties
{
  "name": "={{$json.body.name}}",
  "email": "={{$json.body.email}}"
}
```

---

## When NOT to Use Expressions

### Code Nodes

Code nodes use **direct JavaScript access**, NOT expressions!

```javascript
// WRONG in Code node
const email = '={{$json.email}}';

// CORRECT in Code node
const email = $json.email;
const email = $input.item.json.email;
```

### Webhook Paths
Static paths only: `path: "user-webhook"`

### Credential Fields
Use n8n credential system, not expressions.

---

## Validation Rules

| Rule | Wrong | Correct |
|------|-------|---------|
| Always use `{{}}` | `$json.field` | `{{$json.field}}` |
| Quotes for spaces | `{{$json.field name}}` | `{{$json['field name']}}` |
| Exact node names | `{{$node["http request"]}}` | `{{$node["HTTP Request"]}}` |
| No nested `{{}}` | `{{{$json.field}}}` | `{{$json.field}}` |

---

## Quick Fixes Table

| Mistake | Fix |
|---------|-----|
| `$json.field` | `{{$json.field}}` |
| `{{$json.field name}}` | `{{$json['field name']}}` |
| `{{$node.HTTP Request}}` | `{{$node["HTTP Request"]}}` |
| `{{{$json.field}}}` | `{{$json.field}}` |
| `{{$json.name}}` (webhook) | `{{$json.body.name}}` |
| `'={{$json.email}}'` (Code node) | `$json.email` |

---

## Working Examples

### Example 1: Webhook to Slack

**Webhook receives**: `{"body": {"name": "John", "email": "john@example.com"}}`

**In Slack node text field**:
```
New form submission!
Name: {{$json.body.name}}
Email: {{$json.body.email}}
```

### Example 2: HTTP Request to Email

```
Product: {{$node["HTTP Request"].json.data.items[0].name}}
Price: ${{$node["HTTP Request"].json.data.items[0].price}}
```

### Example 3: Format Timestamp

```javascript
{{$now.toFormat('yyyy-MM-dd')}}       // 2025-10-20
{{$now.toFormat('HH:mm:ss')}}         // 14:30:45
{{$now.toFormat('yyyy-MM-dd HH:mm')}} // 2025-10-20 14:30
```

---

## Best Practices

### Do
- Always use {{ }} for dynamic content
- Use bracket notation for field names with spaces
- Reference webhook data from `.body`
- Use $node for data from other nodes
- Test expressions in expression editor

### Don't
- Don't use expressions in Code nodes
- Don't forget quotes around node names with spaces
- Don't double-wrap with extra {{ }}
- Don't assume webhook data is at root (it's under .body!)
- Don't use expressions in webhook paths or credentials

---

## Detailed References

- **[COMMON_MISTAKES.md](COMMON_MISTAKES.md)** - Complete error catalog with fixes
- **[EXAMPLES.md](EXAMPLES.md)** - Real workflow examples
- **[ADVANCED_EXPRESSIONS.md](ADVANCED_EXPRESSIONS.md)** - Data types (arrays, objects, strings, numbers), advanced patterns (conditionals, dates, string manipulation), debugging, expression helpers

---

## Summary

**Essential Rules**:
1. Wrap expressions in {{ }}
2. Webhook data is under `.body`
3. No {{ }} in Code nodes
4. Quote node names with spaces
5. Node names are case-sensitive

**Related Skills**:
- **n8n MCP Tools Expert**: Validate expressions using MCP tools
- **n8n Workflow Patterns**: See expressions in real workflow examples
- **n8n Node Configuration**: Understand when expressions are needed
