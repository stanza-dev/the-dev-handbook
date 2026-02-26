---
source_course: "javascript-design-patterns"
source_lesson: "javascript-design-patterns-command"
---

# The Command Pattern

## Introduction

The Command pattern encapsulates a request as an object, enabling undo/redo, command queuing, and logging.

## Key Concepts

**Command Object**: Encapsulates an action and the data needed to perform or reverse it.

**Invoker**: The object that triggers the command (e.g., a button or scheduler).

**Receiver**: The object the command acts upon (e.g., a document or canvas).

**Command History**: A stack of executed commands enabling undo/redo.

## Real World Context

Text editors (Ctrl+Z), drawing apps, transaction systems, and task queues all rely on the Command pattern. ES2025's `Promise.withResolvers()` is useful for deferred command execution, returning `{ promise, resolve, reject }` so you can settle a promise outside its constructor:

```javascript
function createDeferredCommand(action) {
  const { promise, resolve, reject } = Promise.withResolvers();
  return {
    execute() {
      try { resolve(action()); }
      catch (e) { reject(e); }
    },
    result: promise
  };
}
const cmd = createDeferredCommand(() => 'done');
cmd.execute();
await cmd.result; // 'done'
```

## Deep Dive

### Undo/Redo System

This implementation tracks a command stack with a position pointer. Executing a new command truncates any redo history, keeping the stack clean:

```javascript
class CommandHistory {
  #history = [];
  #position = -1;
  
  execute(command) {
    this.#history = this.#history.slice(0, this.#position + 1);
    command.execute();
    this.#history.push(command);
    this.#position++;
  }
  
  undo() {
    if (this.#position >= 0) {
      this.#history[this.#position].undo();
      this.#position--;
    }
  }
  
  redo() {
    if (this.#position < this.#history.length - 1) {
      this.#position++;
      this.#history[this.#position].execute();
    }
  }
}

class AddTextCommand {
  constructor(editor, text) {
    this.editor = editor;
    this.text = text;
  }
  
  execute() { this.editor.content += this.text; }
  undo() { this.editor.content = this.editor.content.slice(0, -this.text.length); }
}

const editor = { content: '' };
const history = new CommandHistory();
history.execute(new AddTextCommand(editor, 'Hello'));
history.undo();
history.redo();
```

Each command object encapsulates both the forward action (`execute`) and its reverse (`undo`). The `AddTextCommand` uses string slicing to reverse the insertion, keeping the undo logic self-contained.

## Common Pitfalls

1. **Forgetting to clone state for undo** — If `undo()` relies on the current state instead of a snapshot, reversal will be incorrect.
2. **Unbounded history** — Storing every command forever leaks memory. Set a max history size.
3. **Non-reversible commands** — Not every action has a clean inverse. Design commands with reversibility in mind from the start.

## Best Practices

1. **Keep commands small and focused** — One action per command object simplifies undo logic.
2. **Serialize commands for persistence** — Storing command history as JSON enables replay and audit logs.
3. **Use a command queue for batching** — Group related commands into macro-commands for atomic undo.

## Summary

Command pattern encapsulates operations as objects for undo/redo, queuing, and logging. Essential for text editors and transaction systems.

## Code Examples

**Command history with undo/redo — each command object must implement execute() and undo() methods**

```javascript
class CommandHistory {
  #history = [];
  #position = -1;

  execute(command) {
    this.#history = this.#history.slice(0, this.#position + 1);
    command.execute();
    this.#history.push(command);
    this.#position++;
  }

  undo() {
    if (this.#position >= 0) {
      this.#history[this.#position].undo();
      this.#position--;
    }
  }

  redo() {
    if (this.#position < this.#history.length - 1) {
      this.#position++;
      this.#history[this.#position].execute();
    }
  }
}
```


## Resources

- [Refactoring Guru: Command](https://refactoring.guru/design-patterns/command) — Command pattern guide

---

> 📘 *This lesson is part of the [JavaScript Design Patterns](https://stanza.dev/courses/javascript-design-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*