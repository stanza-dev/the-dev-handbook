---
source_course: "javascript-async-patterns"
source_lesson: "javascript-async-patterns-worker-patterns"
---

# Worker Patterns

## Introduction

Beyond basic workers, patterns like worker pools, promisified workers, and Comlink make workers easier to use. These patterns are essential for production worker usage.

## Deep Dive

### Promisified Worker Communication

```javascript
// worker-utils.js
function createWorker(workerFn) {
  let id = 0;
  const pending = new Map();
  
  const blob = new Blob(
    [`onmessage = async (e) => {
      const { id, args } = e.data;
      try {
        const result = await (${workerFn.toString()})(...args);
        postMessage({ id, result });
      } catch (error) {
        postMessage({ id, error: error.message });
      }
    }`],
    { type: 'application/javascript' }
  );
  
  const worker = new Worker(URL.createObjectURL(blob));
  
  worker.onmessage = (e) => {
    const { id, result, error } = e.data;
    const { resolve, reject } = pending.get(id);
    pending.delete(id);
    error ? reject(new Error(error)) : resolve(result);
  };
  
  return (...args) => new Promise((resolve, reject) => {
    const currentId = id++;
    pending.set(currentId, { resolve, reject });
    worker.postMessage({ id: currentId, args });
  });
}

// Usage
const heavyCalc = createWorker((n) => {
  let result = 0;
  for (let i = 0; i < n; i++) result += i;
  return result;
});

const result = await heavyCalc(1000000);  // Runs in worker!
```

### Worker Pool

```javascript
class WorkerPool {
  constructor(workerScript, size = navigator.hardwareConcurrency) {
    this.workers = Array.from({ length: size }, () => ({
      worker: new Worker(workerScript),
      busy: false
    }));
    this.queue = [];
  }
  
  exec(data) {
    return new Promise((resolve, reject) => {
      const task = { data, resolve, reject };
      const available = this.workers.find(w => !w.busy);
      
      if (available) {
        this.runTask(available, task);
      } else {
        this.queue.push(task);
      }
    });
  }
  
  runTask(workerObj, task) {
    workerObj.busy = true;
    workerObj.worker.onmessage = (e) => {
      task.resolve(e.data);
      workerObj.busy = false;
      if (this.queue.length) {
        this.runTask(workerObj, this.queue.shift());
      }
    };
    workerObj.worker.postMessage(task.data);
  }
  
  terminate() {
    this.workers.forEach(w => w.worker.terminate());
  }
}

const pool = new WorkerPool('processor.js', 4);
const results = await Promise.all(
  items.map(item => pool.exec(item))
);
```

### Using Comlink (Recommended Library)

```javascript
// worker.js
import * as Comlink from 'comlink';

const api = {
  async processData(data) {
    return heavyComputation(data);
  },
  counter: 0,
  increment() {
    return ++this.counter;
  }
};

Comlink.expose(api);

// main.js
import * as Comlink from 'comlink';

const worker = new Worker('worker.js', { type: 'module' });
const api = Comlink.wrap(worker);

// Use like a normal async API!
const result = await api.processData([1, 2, 3]);
await api.increment();
```

## Summary

Promisify worker communication for cleaner code. Use worker pools for parallel batch processing. Consider Comlink for RPC-style worker communication. Always pool workers for repeated tasks.

## Code Examples

**Promisified Worker Communication**

```javascript
// worker-utils.js
function createWorker(workerFn) {
  let id = 0;
  const pending = new Map();
  
  const blob = new Blob(
    [`onmessage = async (e) => {
      const { id, args } = e.data;
      try {
        const result = await (${workerFn.toString()})(...args);
        postMessage({ id, result });
      } catch (error) {
        postMessage({ id, error: error.message });
      }
    }`],
    { type: 'application/javascript' }
  );
```

**Worker Pool**

```javascript
class WorkerPool {
  constructor(workerScript, size = navigator.hardwareConcurrency) {
    this.workers = Array.from({ length: size }, () => ({
      worker: new Worker(workerScript),
      busy: false
    }));
    this.queue = [];
  }
  
  exec(data) {
    return new Promise((resolve, reject) => {
      const task = { data, resolve, reject };
      const available = this.workers.find(w => !w.busy);
      
      if (available) {
        this.runTask(available, task);
      } else {
        this.queue.push(task);
```


## Resources

- [Comlink](https://github.com/GoogleChromeLabs/comlink) — Comlink library for worker RPC

---

> 📘 *This lesson is part of the [Async JavaScript: Promises & Patterns](https://stanza.dev/courses/javascript-async-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*