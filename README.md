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

6. **Polyfill for `typeof`:**

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


### ➜ Asynchronous Programming

13. **Implement `Promise.any`:**

```js
Promise.any = function(promises) {
  return new Promise((resolve, reject) => {
    let rejections = [];
    let count = 0;
    promises.forEach((p, i) => {
      Promise.resolve(p).then(resolve).catch(err => {
        rejections[i] = err;
        count++;
        if (count === promises.length) {
          reject(new AggregateError(rejections, 'All promises were rejected'));
        }
      });
    });
  });
};
```

**`Promise.allSettled`:**

```js
Promise.allSettled = function(promises) {
  return Promise.all(promises.map(p =>
    Promise.resolve(p)
      .then(value => ({ status: 'fulfilled', value }))
      .catch(reason => ({ status: 'rejected', reason }))
  ));
};
```

15. **Run N async tasks in series:**

```js
async function runInSeries(tasks) {
  const results = [];
  for (const task of tasks) {
    results.push(await task());
  }
  return results;
}
```

16. **Execute N async tasks concurrently:**

```js
function runConcurrently(tasks) {
  return Promise.all(tasks.map(task => task()));
}
```

17. **Race N async tasks:**

```js
function raceTasks(tasks) {
  return Promise.race(tasks.map(task => task()));
}
```

18. **Throttle promises (limit API rate):**

```js
function throttle(tasks, limit) {
  let i = 0;
  const results = [];
  return new Promise(resolve => {
    const exec = () => {
      if (i >= tasks.length) return resolve(Promise.all(results));
      const p = tasks[i++]().finally(exec);
      results.push(p);
    };
    for (let j = 0; j < limit; j++) exec();
  });
}
```

19. **Cache identical API requests:**

```js
const cache = new Map();
function cachedFetch(url) {
  if (cache.has(url)) return cache.get(url);
  const promise = fetch(url).then(res => res.json());
  cache.set(url, promise);
  return promise;
}
```

20. **Retry a promise function:**

```js
function retry(fn, retries) {
  return fn().catch(err => {
    if (retries <= 0) throw err;
    return retry(fn, retries - 1);
  });
}
```

21. **Timeout a promise:**

```js
function timeout(promise, time) {
  return Promise.race([
    promise,
    new Promise((_, reject) => setTimeout(() => reject(new Error('Timeout')), time))
  ]);
}
```

22. **Combine and run promises in parallel:**

```js
function runParallel(...fns) {
  return Promise.all(fns.map(fn => fn()));
}
```

---

### ➜ Event Handling and Callbacks

23. **Event Emitter class:**

```js
class EventEmitter {
  constructor() {
    this.events = {};
  }
  on(event, fn) {
    (this.events[event] ||= []).push(fn);
  }
  emit(event, ...args) {
    (this.events[event] || []).forEach(fn => fn(...args));
  }
}
```

24. **Debounce with cancel:**

```js
function debounce(fn, delay) {
  let timer;
  function debounced(...args) {
    clearTimeout(timer);
    timer = setTimeout(() => fn(...args), delay);
  }
  debounced.cancel = () => clearTimeout(timer);
  return debounced;
}
```

25. **Throttle with cancel:**

```js
function throttle(fn, delay) {
  let last = 0;
  let timer;
  function throttled(...args) {
    const now = Date.now();
    if (now - last >= delay) {
      last = now;
      fn(...args);
    } else {
      clearTimeout(timer);
      timer = setTimeout(() => {
        last = Date.now();
        fn(...args);
      }, delay - (now - last));
    }
  }
  throttled.cancel = () => clearTimeout(timer);
  return throttled;
}
```

26. **Custom call method:**

```js
Function.prototype.myCall = function(context, ...args) {
  context = context || globalThis;
  const fn = Symbol('fn');
  context[fn] = this;
  const result = context[fn](...args);
  delete context[fn];
  return result;
};
```

27. **Polyfills for `call`, `apply`, `bind`:**

```js
Function.prototype.myApply = function(context, args = []) {
  context = context || globalThis;
  const fn = Symbol();
  context[fn] = this;
  const result = context[fn](...args);
  delete context[fn];
  return result;
};

Function.prototype.myBind = function(context, ...args) {
  const fn = this;
  return function(...rest) {
    return fn.apply(context, [...args, ...rest]);
  };
};
```

28. **Simple Pub/Sub pattern:**

```js
class PubSub {
  constructor() {
    this.subscribers = {};
  }
  subscribe(event, fn) {
    (this.subscribers[event] ||= []).push(fn);
  }
  publish(event, data) {
    (this.subscribers[event] || []).forEach(fn => fn(data));
  }
}
```

29. **Once-only custom event emitter:**

```js
class OnceEmitter {
  constructor() {
    this.events = {};
  }
  once(event, fn) {
    const wrapper = (...args) => {
      fn(...args);
      this.off(event, wrapper);
    };
    this.on(event, wrapper);
  }
  on(event, fn) {
    (this.events[event] ||= []).push(fn);
  }
  off(event, fn) {
    this.events[event] = (this.events[event] || []).filter(f => f !== fn);
  }
  emit(event, ...args) {
    (this.events[event] || []).forEach(fn => fn(...args));
  }
}
```

30. **Event delegation:**

```js
document.body.addEventListener('click', e => {
  if (e.target.matches('.btn')) {
    console.log('Button clicked:', e.target.textContent);
  }
});
```


### ➜ Functional Programming

31. **Memoize a function taking a single argument:**

```js
function memoize(fn) {
  const cache = new Map();
  return function(x) {
    if (!cache.has(x)) cache.set(x, fn(x));
    return cache.get(x);
  };
}
```

32. **Pipe function to chain N functions:**

```js
function pipe(...fns) {
  return input => fns.reduce((acc, fn) => fn(acc), input);
}
```

33. **Curried function with placeholders:**

```js
function curry(fn, placeholder = '_') {
  return function curried(...args) {
    return fn.length <= args.length && !args.includes(placeholder)
      ? fn(...args)
      : (...next) => curried(...args.map(a => a === placeholder && next.length ? next.shift() : a).concat(next));
  };
}
```

34. **Polyfill for `Object.assign`:**

```js
Object.myAssign = function(target, ...sources) {
  if (target == null) throw new TypeError('Cannot convert undefined or null to object');
  const to = Object(target);
  for (const src of sources) {
    if (src != null) {
      for (const key in src) {
        if (Object.prototype.hasOwnProperty.call(src, key)) {
          to[key] = src[key];
        }
      }
    }
  }
  return to;
};
```

35. **Polyfill for Lodash's `memoize`:**

```js
function lodashMemoize(fn) {
  const cache = new Map();
  return function(...args) {
    const key = JSON.stringify(args);
    if (!cache.has(key)) cache.set(key, fn(...args));
    return cache.get(key);
  };
}
```

36. **Factorial using recursion:**

```js
function factorial(n) {
  return n <= 1 ? 1 : n * factorial(n - 1);
}
```

37. **Generate Fibonacci numbers up to N:**

```js
function fibonacci(n) {
  const seq = [0, 1];
  for (let i = 2; i < n; i++) {
    seq.push(seq[i - 1] + seq[i - 2]);
  }
  return seq;
}
```

38. **Custom `_chunk` function like Lodash:**

```js
function chunk(arr, size) {
  const result = [];
  for (let i = 0; i < arr.length; i += size) {
    result.push(arr.slice(i, i + size));
  }
  return result;
}
```

39. **Compose function (right to left):**

```js
function compose(...fns) {
  return input => fns.reduceRight((acc, fn) => fn(acc), input);
}
```

40. **Lazy evaluation of chained functions:**

```js
function lazy(...fns) {
  return input => () => fns.reduce((acc, fn) => fn(acc), input);
}
```

41. **Flatten array using reduce:**

```js
function flatten(arr) {
  return arr.reduce((acc, val) => acc.concat(Array.isArray(val) ? flatten(val) : val), []);
}
```

---

### ➜ Array and String Manipulation

42. **Check deep equality of two values:**

```js
function isDeepEqual(a, b) {
  if (a === b) return true;
  if (typeof a !== typeof b) return false;
  if (typeof a !== 'object' || a === null || b === null) return false;
  if (Array.isArray(a) !== Array.isArray(b)) return false;
  const keysA = Object.keys(a), keysB = Object.keys(b);
  if (keysA.length !== keysB.length) return false;
  return keysA.every(k => isDeepEqual(a[k], b[k]));
}
```

43. **Recursively flatten an array:**

```js
function flattenRecursive(arr) {
  return arr.reduce((acc, val) => acc.concat(Array.isArray(val) ? flattenRecursive(val) : val), []);
}
```

44. **Negative indexing with Proxy:**

```js
function negativeIndexArray(arr) {
  return new Proxy(arr, {
    get(target, prop) {
      if (+prop < 0) prop = target.length + +prop;
      return target[prop];
    }
  });
}
```

45. **Custom Lodash `_.get`:**

```js
function get(obj, path, defaultVal = undefined) {
  return path.split('.').reduce((acc, key) => acc?.[key], obj) ?? defaultVal;
}
```

46. **Find intersection of two arrays:**

```js
function intersection(arr1, arr2) {
  const set = new Set(arr2);
  return arr1.filter(item => set.has(item));
}
```

47. **Remove duplicates from array:**

```js
function removeDuplicates(arr) {
  return [...new Set(arr)];
}
```

48. **Sort array of objects by property:**

```js
function sortBy(arr, prop) {
  return arr.sort((a, b) => a[prop] > b[prop] ? 1 : -1);
}
```

49. **Group array of objects by property:**

```js
function groupBy(arr, prop) {
  return arr.reduce((acc, obj) => {
    const key = obj[prop];
    (acc[key] ||= []).push(obj);
    return acc;
  }, {});
}
```

50. **Rotate array left by k positions:**

```js
function rotateLeft(arr, k) {
  return arr.slice(k).concat(arr.slice(0, k));
}
```


### ➜ DOM Manipulation and Events

51. **Add an event listener to multiple elements:**

```js
const buttons = document.querySelectorAll('button');
buttons.forEach(btn => btn.addEventListener('click', () => alert('Clicked!')));
```

52. **Remove all child nodes from a DOM element:**

```js
function removeAllChildren(parent) {
  while (parent.firstChild) {
    parent.removeChild(parent.firstChild);
  }
}
```

53. **Toggle a class on an element when clicked:**

```js
document.getElementById('myElement').addEventListener('click', function() {
  this.classList.toggle('active');
});
```

54. **Clone a DOM element and its children:**

```js
const original = document.getElementById('elementToClone');
const clone = original.cloneNode(true);
document.body.appendChild(clone);
```

