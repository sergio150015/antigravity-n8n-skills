# Advanced JavaScript Patterns for n8n Code Nodes

Detailed patterns, built-in functions, and best practices.

---

## Table of Contents

- [Common Patterns (Detailed)](#common-patterns-detailed)
  - [Multi-Source Data Aggregation](#1-multi-source-data-aggregation)
  - [Filtering with Regex](#2-filtering-with-regex)
  - [Data Transformation and Enrichment](#3-data-transformation--enrichment)
  - [Top N Filtering and Ranking](#4-top-n-filtering--ranking)
  - [Aggregation and Reporting](#5-aggregation--reporting)
- [Built-in Functions and Helpers](#built-in-functions--helpers)
  - [$helpers.httpRequest()](#helpershttprequest)
  - [DateTime (Luxon)](#datetime-luxon)
  - [$jmespath()](#jmespath)
- [Best Practices (Detailed)](#best-practices-detailed)
  - [Validate Input Data](#1-always-validate-input-data)
  - [Try-Catch Error Handling](#2-use-try-catch-for-error-handling)
  - [Prefer Array Methods](#3-prefer-array-methods-over-loops)
  - [Filter Early Process Late](#4-filter-early-process-late)
  - [Descriptive Variable Names](#5-use-descriptive-variable-names)
  - [Debug with console.log()](#6-debug-with-consolelog)

---

## Common Patterns (Detailed)

### 1. Multi-Source Data Aggregation
Combine data from multiple APIs, webhooks, or nodes

```javascript
const allItems = $input.all();
const results = [];

for (const item of allItems) {
  const sourceName = item.json.name || 'Unknown';
  // Parse source-specific structure
  if (sourceName === 'API1' && item.json.data) {
    results.push({
      json: {
        title: item.json.data.title,
        source: 'API1'
      }
    });
  }
}

return results;
```

### 2. Filtering with Regex
Extract patterns, mentions, or keywords from text

```javascript
const pattern = /\b([A-Z]{2,5})\b/g;
const matches = {};

for (const item of $input.all()) {
  const text = item.json.text;
  const found = text.match(pattern);

  if (found) {
    found.forEach(match => {
      matches[match] = (matches[match] || 0) + 1;
    });
  }
}

return [{json: {matches}}];
```

### 3. Data Transformation & Enrichment
Map fields, normalize formats, add computed fields

```javascript
const items = $input.all();

return items.map(item => {
  const data = item.json;
  const nameParts = data.name.split(' ');

  return {
    json: {
      first_name: nameParts[0],
      last_name: nameParts.slice(1).join(' '),
      email: data.email,
      created_at: new Date().toISOString()
    }
  };
});
```

### 4. Top N Filtering & Ranking
Sort and limit results

```javascript
const items = $input.all();

const topItems = items
  .sort((a, b) => (b.json.score || 0) - (a.json.score || 0))
  .slice(0, 10);

return topItems.map(item => ({json: item.json}));
```

### 5. Aggregation & Reporting
Sum, count, group data

```javascript
const items = $input.all();
const total = items.reduce((sum, item) => sum + (item.json.amount || 0), 0);

return [{
  json: {
    total,
    count: items.length,
    average: total / items.length,
    timestamp: new Date().toISOString()
  }
}];
```

See [COMMON_PATTERNS.md](COMMON_PATTERNS.md) for 10 production-tested patterns.

---

## Built-in Functions & Helpers

### $helpers.httpRequest()

Make HTTP requests from within code:

```javascript
const response = await $helpers.httpRequest({
  method: 'GET',
  url: 'https://api.example.com/data',
  headers: {
    'Authorization': 'Bearer token',
    'Content-Type': 'application/json'
  }
});

return [{json: {data: response}}];
```

### DateTime (Luxon)

Date and time operations:

```javascript
// Current time
const now = DateTime.now();

// Format dates
const formatted = now.toFormat('yyyy-MM-dd');
const iso = now.toISO();

// Date arithmetic
const tomorrow = now.plus({days: 1});
const lastWeek = now.minus({weeks: 1});

return [{
  json: {
    today: formatted,
    tomorrow: tomorrow.toFormat('yyyy-MM-dd')
  }
}];
```

### $jmespath()

Query JSON structures:

```javascript
const data = $input.first().json;

// Filter array
const adults = $jmespath(data, 'users[?age >= `18`]');

// Extract fields
const names = $jmespath(data, 'users[*].name');

return [{json: {adults, names}}];
```

See [BUILTIN_FUNCTIONS.md](BUILTIN_FUNCTIONS.md) for complete reference.

---

## Best Practices (Detailed)

### 1. Always Validate Input Data

```javascript
const items = $input.all();

// Check if data exists
if (!items || items.length === 0) {
  return [];
}

// Validate structure
if (!items[0].json) {
  return [{json: {error: 'Invalid input format'}}];
}

// Continue processing...
```

### 2. Use Try-Catch for Error Handling

```javascript
try {
  const response = await $helpers.httpRequest({
    url: 'https://api.example.com/data'
  });

  return [{json: {success: true, data: response}}];
} catch (error) {
  return [{
    json: {
      success: false,
      error: error.message
    }
  }];
}
```

### 3. Prefer Array Methods Over Loops

```javascript
// GOOD: Functional approach
const processed = $input.all()
  .filter(item => item.json.valid)
  .map(item => ({json: {id: item.json.id}}));

// SLOWER: Manual loop
const processed = [];
for (const item of $input.all()) {
  if (item.json.valid) {
    processed.push({json: {id: item.json.id}});
  }
}
```

### 4. Filter Early, Process Late

```javascript
// GOOD: Filter first to reduce processing
const processed = $input.all()
  .filter(item => item.json.status === 'active')  // Reduce dataset first
  .map(item => expensiveTransformation(item));  // Then transform

// WASTEFUL: Transform everything, then filter
const processed = $input.all()
  .map(item => expensiveTransformation(item))  // Wastes CPU
  .filter(item => item.json.status === 'active');
```

### 5. Use Descriptive Variable Names

```javascript
// GOOD: Clear intent
const activeUsers = $input.all().filter(item => item.json.active);
const totalRevenue = activeUsers.reduce((sum, user) => sum + user.json.revenue, 0);

// BAD: Unclear purpose
const a = $input.all().filter(item => item.json.active);
const t = a.reduce((s, u) => s + u.json.revenue, 0);
```

### 6. Debug with console.log()

```javascript
// Debug statements appear in browser console
const items = $input.all();
console.log(`Processing ${items.length} items`);

for (const item of items) {
  console.log('Item data:', item.json);
  // Process...
}

return result;
```
