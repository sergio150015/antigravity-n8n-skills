---
name: n8n-validation-expert
description: Interpret validation errors and guide fixing them. Use when encountering validation errors, validation warnings, false positives, operator structure issues, or need help understanding validation results. Also use when asking about validation profiles, error types, or the validation loop process.
---

# n8n Validation Expert

Expert guide for interpreting and fixing n8n validation errors.

---

## Validation Philosophy

**Validate early, validate often**

Validation is typically iterative:
- Expect validation feedback loops
- Usually 2-3 validate -> fix cycles
- Average: 23s thinking about errors, 58s fixing them

**Key insight**: Validation is an iterative process, not one-shot!

---

## Error Severity Levels

### 1. Errors (Must Fix)
**Blocks workflow execution** - Must be resolved before activation

**Types**: `missing_required`, `invalid_value`, `type_mismatch`, `invalid_reference`, `invalid_expression`

### 2. Warnings (Should Fix)
**Doesn't block execution** - Workflow can be activated but may have issues

**Types**: `best_practice`, `deprecated`, `performance`

### 3. Suggestions (Optional)
**Nice to have** - `optimization`, `alternative`

---

## The Validation Loop

### Pattern from Telemetry
**7,841 occurrences** of this pattern:

```
1. Configure node
   |
2. validate_node (23 seconds thinking about errors)
   |
3. Read error messages carefully
   |
4. Fix errors
   |
5. validate_node again (58 seconds fixing)
   |
6. Repeat until valid (usually 2-3 iterations)
```

**This is normal!** Don't be discouraged by multiple iterations.

---

## Validation Profiles

Choose the right profile for your stage:

| Profile | Use When | Validates |
|---------|----------|-----------|
| `minimal` | Quick checks during editing | Only required fields, basic structure |
| `runtime` | **Pre-deployment (RECOMMENDED)** | Required fields, value types, allowed values, basic dependencies |
| `ai-friendly` | AI-generated configurations | Same as runtime, reduces false positives |
| `strict` | Production, critical workflows | Everything + best practices + security |

---

## Common Error Types

### 1. missing_required
A required field is not provided. **Fix**: Use `get_node` to see required fields, add the missing field.

```javascript
// Error: "Channel name is required"
// Fix:
config.channel = "#general";
```

### 2. invalid_value
Value doesn't match allowed options. **Fix**: Check error message for allowed values.

```javascript
// Error: "Operation must be one of: post, update, delete"
// Fix:
config.operation = "post";  // Use valid operation
```

### 3. type_mismatch
Wrong data type for field. **Fix**: Convert value to correct type.

```javascript
// Error: "Expected number, got string"
// Fix:
config.limit = 100;  // Number, not "100"
```

### 4. invalid_expression
Expression syntax error. **Fix**: Use n8n Expression Syntax skill.

```javascript
// Error: 'Invalid expression: $json.name'
// Fix:
config.text = "={{$json.name}}";  // Add {{}}
```

### 5. invalid_reference
Referenced node doesn't exist. **Fix**: Check node name spelling (case-sensitive).

```javascript
// Error: "Node 'HTTP Requets' does not exist"
// Fix:
config.expression = "={{$node['HTTP Request'].json.data}}";
```

---

## Auto-Sanitization (Summary)

**Automatically fixes operator structure issues** on ANY workflow update.

| What | Fix Applied |
|------|-------------|
| Binary operators (equals, contains, etc.) | Removes `singleValue` property |
| Unary operators (isEmpty, isNotEmpty, true, false) | Adds `singleValue: true` |
| IF/Switch metadata | Adds `conditions.options` for v2.2+/v3.2+ |

**Cannot fix**: broken connections, branch count mismatches, paradoxical corrupt states.

**See**: [ADVANCED_VALIDATION.md](ADVANCED_VALIDATION.md) for full auto-sanitization details, false positives guide, validation result structure, workflow validation, and recovery strategies

---

## Best Practices

### Do

- Validate after every significant change
- Read error messages completely
- Fix errors iteratively (one at a time)
- Use `runtime` profile for pre-deployment
- Check `valid` field before assuming success
- Trust auto-sanitization for operator issues
- Use `get_node` when unclear about requirements

### Don't

- Skip validation before activation
- Try to fix all errors at once
- Ignore error messages
- Use `strict` profile during development (too noisy)
- Assume validation passed (always check result)
- Manually fix auto-sanitization issues
- Deploy with unresolved errors

---

## Detailed Guides

- **[ERROR_CATALOG.md](ERROR_CATALOG.md)** - Complete list of error types with examples
- **[FALSE_POSITIVES.md](FALSE_POSITIVES.md)** - When warnings are acceptable
- **[ADVANCED_VALIDATION.md](ADVANCED_VALIDATION.md)** - Auto-sanitization, false positives, result structure, workflow validation, recovery strategies

---

## Summary

**Key Points**:
1. **Validation is iterative** (avg 2-3 cycles, 23s + 58s)
2. **Errors must be fixed**, warnings are optional
3. **Auto-sanitization** fixes operator structures automatically
4. **Use runtime profile** for balanced validation
5. **False positives exist** - learn to recognize them
6. **Read error messages** - they contain fix guidance

**Related Skills**:
- n8n MCP Tools Expert - Use validation tools correctly
- n8n Expression Syntax - Fix expression errors
- n8n Node Configuration - Understand required fields
