# JavaScript-Questions

## JavaScript Interview Questions & Answers

### ➜ JSON and Object Manipulation

1. **Convert a JS value to a JSON string:**

```js
JSON.stringify(value);
```

2. **Deep copy of a value handling circular references:**

```js
function deepCopy(obj, seen = new WeakMap()) {
  if (obj === null || typeof obj !== 'object') return obj;
  if (seen.has(obj)) return seen.get(obj);
  let copy = Array.isArray(obj) ? [] : {};
  seen.set(obj, copy);
  for (let key in obj) {
    copy[key] = deepCopy(obj[key], seen);
  }
  return copy;
}
```

3. **Convert JSON string to JS value:**

```js
JSON.parse(jsonString);
```

4. **Merge two objects deeply:**

```js
function deepMerge(obj1, obj2) {
  const result = { ...obj1 };
  for (const key in obj2) {
    if (obj2[key] instanceof Object && key in obj1) {
      result[key] = deepMerge(obj1[key], obj2[key]);
    } else {
      result[key] = obj2[key];
    }
  }
  return result;
}
```

5. **Deep freeze an object:**

```js
function deepFreeze(obj) {
  Object.getOwnPropertyNames(obj).forEach(prop => {
    if (typeof obj[prop] === 'object' && obj[prop] !== null) {
      deepFreeze(obj[prop]);
    }
  });
  return Object.freeze(obj);
}
```

6. **Polyfill for ********`typeof`********:**

```js
function typeOf(value) {
  return Object.prototype.toString.call(value).slice(8, -1).toLowerCase();
}
```

7. **Object to query string:**

```js
function toQueryString(obj) {
  return Object.keys(obj).map(key => `${encodeURIComponent(key)}=${encodeURIComponent(obj[key])}`).join('&');
}
```

8. **Query string to object:**

```js
function parseQueryString(qs) {
  return Object.fromEntries(new URLSearchParams(qs));
}
```

9. **Return longest word in a string:**

```js
function longestWord(str) {
  return str.split(/\s+/).reduce((a, b) => a.length > b.length ? a : b);
}
```

10. **Deep equality check:**

```js
function deepEqual(a, b) {
  if (a === b) return true;
  if (typeof a !== 'object' || typeof b !== 'object' || a == null || b == null) return false;
  let keysA = Object.keys(a), keysB = Object.keys(b);
  if (keysA.length !== keysB.length) return false;
  return keysA.every(key => deepEqual(a[key], b[key]));
}
```

11. **Flatten a deeply nested object:**

```js
function flattenObject(obj, prefix = '', res = {}) {
  for (let key in obj) {
    let prop = prefix.length ? `${prefix}.${key}` : key;
    if (typeof obj[key] === 'object' && obj[key] !== null) {
      flattenObject(obj[key], prop, res);
    } else {
      res[prop] = obj[key];
    }
  }
  return res;
}
```

12. **Rename keys based on mapping:**

```js
function renameKeys(obj, mapping) {
  return Object.fromEntries(Object.entries(obj).map(([key, val]) => [mapping[key] || key, val]));
}
```

