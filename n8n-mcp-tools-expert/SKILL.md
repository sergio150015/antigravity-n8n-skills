---
name: n8n-mcp-tools-expert
description: Expert guide for using n8n-mcp MCP tools effectively. Use when searching for nodes, validating configurations, accessing templates, managing workflows, or using any n8n-mcp tool. Provides tool selection guidance, parameter formats, and common patterns.
---

# n8n MCP Tools Expert

Master guide for using n8n-mcp MCP server tools to build workflows.

---

## Tool Categories

n8n-mcp provides tools organized into categories:

1. **Node Discovery** -> [SEARCH_GUIDE.md](SEARCH_GUIDE.md)
2. **Configuration Validation** -> [VALIDATION_GUIDE.md](VALIDATION_GUIDE.md)
3. **Workflow Management** -> [WORKFLOW_GUIDE.md](WORKFLOW_GUIDE.md)
4. **Template Library** - Search and deploy 2,700+ real workflows
5. **Documentation & Guides** - Tool docs, AI agent guide, Code node guides

---

## Quick Reference

### Most Used Tools (by success rate)

| Tool | Use When | Speed |
|------|----------|-------|
| `search_nodes` | Finding nodes by keyword | <20ms |
| `get_node` | Understanding node operations (detail="standard") | <10ms |
| `validate_node` | Checking configurations (mode="full") | <100ms |
| `n8n_create_workflow` | Creating workflows | 100-500ms |
| `n8n_update_partial_workflow` | Editing workflows (MOST USED!) | 50-200ms |
| `validate_workflow` | Checking complete workflow | 100-500ms |
| `n8n_deploy_template` | Deploy template to n8n instance | 200-500ms |

---

## Tool Selection Guide

### Finding the Right Node

```
1. search_nodes({query: "keyword"})
2. get_node({nodeType: "nodes-base.name"})
3. [Optional] get_node({nodeType: "nodes-base.name", mode: "docs"})
```

**Common pattern**: search -> get_node (18s average)

### Validating Configuration

```
1. validate_node({nodeType, config: {}, mode: "minimal"}) - Check required fields
2. validate_node({nodeType, config, profile: "runtime"}) - Full validation
3. [Repeat] Fix errors, validate again
```

**Common pattern**: validate -> fix -> validate (23s thinking, 58s fixing per cycle)

### Managing Workflows

```
1. n8n_create_workflow({name, nodes, connections})
2. n8n_validate_workflow({id})
3. n8n_update_partial_workflow({id, operations: [...]})
4. n8n_validate_workflow({id}) again
5. n8n_update_partial_workflow({id, operations: [{type: "activateWorkflow"}]})
```

**Common pattern**: iterative updates (56s average between edits)

---

## Critical: nodeType Formats

**Two different formats** for different tools!

| Context | Format | Example |
|---------|--------|---------|
| Search/Validate tools | SHORT prefix | `nodes-base.slack` |
| Workflow tools | FULL prefix | `n8n-nodes-base.slack` |
| Langchain (search) | | `nodes-langchain.agent` |
| Langchain (workflow) | | `@n8n/n8n-nodes-langchain.agent` |

```javascript
// search_nodes returns BOTH formats
{
  "nodeType": "nodes-base.slack",          // For search/validate tools
  "workflowNodeType": "n8n-nodes-base.slack"  // For workflow tools
}
```

---

## Common Mistakes

### Mistake 1: Wrong nodeType Format

```javascript
// WRONG
get_node({nodeType: "slack"})  // Missing prefix
get_node({nodeType: "n8n-nodes-base.slack"})  // Wrong prefix for search/validate

// CORRECT
get_node({nodeType: "nodes-base.slack"})
```

### Mistake 2: Using detail="full" by Default

```javascript
// WRONG - Returns 3-8K tokens
get_node({nodeType: "nodes-base.slack", detail: "full"})

// CORRECT - Returns 1-2K tokens, covers 95% of needs
get_node({nodeType: "nodes-base.slack"})  // detail="standard" is default
```

**Better alternatives**: `get_node({mode: "docs"})`, `get_node({mode: "search_properties", propertyQuery: "auth"})`

### Mistake 3: Not Using Validation Profiles

```javascript
// WRONG - Uses default profile
validate_node({nodeType, config})

// CORRECT - Explicit profile
validate_node({nodeType, config, profile: "runtime"})
```

### Mistake 4: Not Using Smart Parameters

```javascript
// Old way (manual sourceIndex)
{type: "addConnection", source: "IF", target: "Handler", sourceIndex: 0}

// New way (semantic)
{type: "addConnection", source: "IF", target: "True Handler", branch: "true"}
{type: "addConnection", source: "Switch", target: "Handler A", case: 0}
```

### Mistake 5: Not Using intent Parameter

```javascript
// WRONG
n8n_update_partial_workflow({id: "abc", operations: [...]})

// CORRECT - Better AI responses
n8n_update_partial_workflow({id: "abc", intent: "Add error handling", operations: [...]})
```

---

## Tool Usage Patterns

### Pattern 1: Node Discovery (Most Common)

```javascript
// Step 1: Search
const results = await search_nodes({query: "slack", mode: "OR", limit: 20});

// Step 2: Get details (~18s later)
const details = await get_node({
  nodeType: "nodes-base.slack",
  includeExamples: true
});
```

### Pattern 2: Validation Loop

```javascript
// Step 1: Validate
const result = await validate_node({
  nodeType: "nodes-base.slack",
  config: {resource: "channel", operation: "create"},
  profile: "runtime"
});

// Step 2: Check errors -> Fix -> Validate again (repeat)
```

### Pattern 3: Workflow Editing

```javascript
// Iterative workflow building (NOT one-shot!)
await n8n_update_partial_workflow({
  id: "workflow-id",
  intent: "Add webhook trigger",
  operations: [{type: "addNode", node: {...}}]
});

// ~56s later... next edit
// Ready? Activate!
await n8n_update_partial_workflow({
  id: "workflow-id",
  operations: [{type: "activateWorkflow"}]
});
```

---

## Best Practices

### Do
- Use `get_node({detail: "standard"})` for most use cases
- Specify validation profile explicitly (`profile: "runtime"`)
- Use smart parameters (`branch`, `case`) for clarity
- Include `intent` parameter in workflow updates
- Follow search -> get_node -> validate workflow
- Iterate workflows (avg 56s between edits)
- Use `n8n_deploy_template` for quick starts

### Don't
- Use `detail: "full"` unless necessary (wastes tokens)
- Forget nodeType prefix (`nodes-base.*`)
- Skip validation profiles
- Try to build workflows in one shot (iterate!)
- Use full prefix (`n8n-nodes-base.*`) with search/validate tools

---

## Detailed Guides

- **[SEARCH_GUIDE.md](SEARCH_GUIDE.md)** - Node discovery: search_nodes, get_node modes and detail levels
- **[VALIDATION_GUIDE.md](VALIDATION_GUIDE.md)** - Validation profiles, validate_node, validate_workflow, auto-sanitization
- **[WORKFLOW_GUIDE.md](WORKFLOW_GUIDE.md)** - n8n_create_workflow, n8n_update_partial_workflow (17 ops!), smart params, AI connections, activation, deploy_template, versions
- **[TOOLS_REFERENCE.md](TOOLS_REFERENCE.md)** - Template usage, self-help tools, unified tool reference (get_node/validate_node), performance data, tool availability

---

## Summary

**Most Important**:
1. Use **get_node** with `detail: "standard"` (default) - covers 95% of use cases
2. nodeType formats differ: `nodes-base.*` (search/validate) vs `n8n-nodes-base.*` (workflows)
3. Specify **validation profiles** (`runtime` recommended)
4. Use **smart parameters** (`branch="true"`, `case=0`)
5. Include **intent parameter** in workflow updates
6. **Auto-sanitization** runs on ALL nodes during updates
7. Workflows are built **iteratively** (56s avg between edits)

**Related Skills**:
- n8n Expression Syntax - Write expressions in workflow fields
- n8n Workflow Patterns - Architectural patterns from templates
- n8n Validation Expert - Interpret validation errors
- n8n Node Configuration - Operation-specific requirements
- n8n Code JavaScript - Write JavaScript in Code nodes
- n8n Code Python - Write Python in Code nodes
