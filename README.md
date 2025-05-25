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

---

### ➜ Asynchronous Programming

13. **Implement Promise.any and Promise.allSettled:**

```js
Promise.any = function(promises) {
  return new Promise((resolve, reject) => {
    let rejections = [];
    let count = 0;
    promises.forEach(p => Promise.resolve(p).then(resolve, err => {
      rejections.push(err);
      if (++count === promises.length) reject(rejections);
    }));
  });
};

Promise.allSettled = function(promises) {
  return Promise.all(promises.map(p =>
    Promise.resolve(p).then(
      value => ({ status: 'fulfilled', value }),
      reason => ({ status: 'rejected', reason })
    )));
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
async function runConcurrently(tasks) {
  return Promise.all(tasks.map(task => task()));
}
```

17. **Execute N async tasks in a race condition:**

```js
async function runRace(tasks) {
  return Promise.race(tasks.map(task => task()));
}
```

18. **Throttle promises to limit API request rates:**

```js
function throttlePromises(tasks, limit) {
  let i = 0;
  let active = 0;
  const results = [];

  return new Promise((resolve, reject) => {
    const next = () => {
      if (i === tasks.length && active === 0) return resolve(results);
      while (active < limit && i < tasks.length) {
        const idx = i++;
        active++;
        tasks[idx]().then(res => {
          results[idx] = res;
          active--;
          next();
        }, reject);
      }
    };
    next();
  });
}
```

19. **Cache identical API requests:**

```js
const cache = new Map();
function cachedRequest(url) {
  if (!cache.has(url)) {
    cache.set(url, fetch(url).then(res => res.json()));
  }
  return cache.get(url);
}
```

20. **Retry a promise-based function:**

```js
function retry(fn, times) {
  return new Promise((resolve, reject) => {
    function attempt() {
      fn().then(resolve).catch(err => {
        if (--times === 0) reject(err);
        else attempt();
      });
    }
    attempt();
  });
}
```

21. **Timeout for a promise:**

```js
function withTimeout(promise, ms) {
  return Promise.race([
    promise,
    new Promise((_, reject) => setTimeout(() => reject(new Error('Timeout')), ms))
  ]);
}
```

22. **Combine multiple promise-based functions (parallel):**

```js
function runParallel(promises) {
  return Promise.all(promises);
}
```

---

### ➜ Event Handling and Callbacks

23. **Class that subscribes and emits events:**

```js
class EventEmitter {
  constructor() {
    this.events = {};
  }
  on(event, cb) {
    (this.events[event] ||= []).push(cb);
  }
  emit(event, ...args) {
    (this.events[event] || []).forEach(cb => cb(...args));
  }
}
```

24. **Debounce function with cancel:**

```js
function debounce(fn, delay) {
  let timer;
  function debounced(...args) {
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(this, args), delay);
  }
  debounced.cancel = () => clearTimeout(timer);
  return debounced;
}
```

25. **Throttle function with cancel:**

```js
function throttle(fn, delay) {
  let last = 0, timer;
  function throttled(...args) {
    const now = Date.now();
    if (now - last >= delay) {
      last = now;
      fn.apply(this, args);
    } else {
      clearTimeout(timer);
      timer = setTimeout(() => {
        last = Date.now();
        fn.apply(this, args);
      }, delay - (now - last));
    }
  }
  throttled.cancel = () => clearTimeout(timer);
  return throttled;
}
```

26. **Own version of `call`:**

```js
Function.prototype.myCall = function(context, ...args) {
  context = context || globalThis;
  const fn = Symbol();
  context[fn] = this;
  const result = context[fn](...args);
  delete context[fn];
  return result;
};
```

27. **Polyfills for `call`, `apply`, and `bind`:**

```js
Function.prototype.myApply = function(context, args) {
  context = context || globalThis;
  const fn = Symbol();
  context[fn] = this;
  const result = context[fn](...(args || []));
  delete context[fn];
  return result;
};

Function.prototype.myBind = function(context, ...args) {
  const fn = this;
  return function(...newArgs) {
    return fn.apply(context, [...args, ...newArgs]);
  };
};
```

28. **Simple pub/sub pattern:**

```js
class PubSub {
  constructor() {
    this.subs = {};
  }
  subscribe(event, cb) {
    (this.subs[event] ||= []).push(cb);
  }
  publish(event, data) {
    (this.subs[event] || []).forEach(cb => cb(data));
  }
}
```

29. **Once-only event emitter:**

```js
class OnceEventEmitter {
  constructor() {
    this.events = {};
  }
  once(event, cb) {
    const onceWrapper = (...args) => {
      cb(...args);
      this.off(event, onceWrapper);
    };
    this.on(event, onceWrapper);
  }
  on(event, cb) {
    (this.events[event] ||= []).push(cb);
  }
  off(event, cb) {
    this.events[event] = (this.events[event] || []).filter(fn => fn !== cb);
  }
  emit(event, ...args) {
    (this.events[event] || []).forEach(cb => cb(...args));
  }
}
```

30. **Handle event delegation:**

```js
document.getElementById('parent').addEventListener('click', function(e) {
  if (e.target && e.target.matches('.child')) {
    console.log('Child clicked:', e.target);
  }
});
```
