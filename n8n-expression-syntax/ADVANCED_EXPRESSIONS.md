# Advanced Expression Patterns

Data types, advanced patterns, debugging, and expression helpers.

---

## Table of Contents

- [Data Type Handling](#data-type-handling)
  - [Arrays](#arrays)
  - [Objects](#objects)
  - [Strings](#strings)
  - [Numbers](#numbers)
- [Advanced Patterns](#advanced-patterns)
  - [Conditional Content](#conditional-content)
  - [Date Manipulation](#date-manipulation)
  - [String Manipulation](#string-manipulation)
- [Debugging Expressions](#debugging-expressions)
  - [Test in Expression Editor](#test-in-expression-editor)
  - [Common Error Messages](#common-error-messages)
- [Expression Helpers](#expression-helpers)
  - [String Methods](#string-methods)
  - [Array Methods](#array-methods)
  - [DateTime (Luxon)](#datetime-luxon)
  - [Number Methods](#number-methods)

---

## Data Type Handling

### Arrays

```javascript
// First item
{{$json.users[0].email}}

// Array length
{{$json.users.length}}

// Last item
{{$json.users[$json.users.length - 1].name}}
```

### Objects

```javascript
// Dot notation (no spaces)
{{$json.user.email}}

// Bracket notation (with spaces or dynamic)
{{$json['user data'].email}}
```

### Strings

```javascript
// Concatenation (automatic)
Hello {{$json.name}}!

// String methods
{{$json.email.toLowerCase()}}
{{$json.name.toUpperCase()}}
```

### Numbers

```javascript
// Direct use
{{$json.price}}

// Math operations
{{$json.price * 1.1}}  // Add 10%
{{$json.quantity + 5}}
```

---

## Advanced Patterns

### Conditional Content

```javascript
// Ternary operator
{{$json.status === 'active' ? 'Active User' : 'Inactive User'}}

// Default values
{{$json.email || 'no-email@example.com'}}
```

### Date Manipulation

```javascript
// Add days
{{$now.plus({days: 7}).toFormat('yyyy-MM-dd')}}

// Subtract hours
{{$now.minus({hours: 24}).toISO()}}

// Set specific date
{{DateTime.fromISO('2025-12-25').toFormat('MMMM dd, yyyy')}}
```

### String Manipulation

```javascript
// Substring
{{$json.email.substring(0, 5)}}

// Replace
{{$json.message.replace('old', 'new')}}

// Split and join
{{$json.tags.split(',').join(', ')}}
```

---

## Debugging Expressions

### Test in Expression Editor

1. Click field with expression
2. Open expression editor (click "fx" icon)
3. See live preview of result
4. Check for errors highlighted in red

### Common Error Messages

**"Cannot read property 'X' of undefined"**
- Parent object doesn't exist
- Check your data path

**"X is not a function"**
- Trying to call method on non-function
- Check variable type

**Expression shows as literal text**
- Missing {{ }}
- Add curly braces

---

## Expression Helpers

### String Methods
- `.toLowerCase()`, `.toUpperCase()`
- `.trim()`, `.replace()`, `.substring()`
- `.split()`, `.includes()`

### Array Methods
- `.length`, `.map()`, `.filter()`
- `.find()`, `.join()`, `.slice()`

### DateTime (Luxon)
- `.toFormat()`, `.toISO()`, `.toLocal()`
- `.plus()`, `.minus()`, `.set()`

### Number Methods
- `.toFixed()`, `.toString()`
- Math operations: `+`, `-`, `*`, `/`, `%`
