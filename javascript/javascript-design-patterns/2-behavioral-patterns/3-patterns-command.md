---
source_course: "javascript-design-patterns"
source_lesson: "javascript-design-patterns-command"
---

# The Command Pattern

## Introduction

The Command pattern encapsulates a request as an object, enabling undo/redo, command queuing, and logging.

## Deep Dive

### Undo/Redo System

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

## Summary

Command pattern encapsulates operations as objects for undo/redo, queuing, and logging. Essential for text editors and transaction systems.

## Resources

- [Refactoring Guru: Command](https://refactoring.guru/design-patterns/command) — Command pattern guide

---

> 📘 *This lesson is part of the [JavaScript Design Patterns](https://stanza.dev/courses/javascript-design-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*