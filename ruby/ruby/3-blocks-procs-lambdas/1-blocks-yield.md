---
source_course: "ruby"
source_lesson: "ruby-blocks-yield"
---

# Blocks & Yield

## Introduction

Blocks are one of Ruby's most distinctive features. They let you pass a chunk of code to a method, which can then execute that code with `yield`. Blocks power iteration methods like `each` and `map`, configuration DSLs, and resource management patterns. Understanding blocks is essential because they appear in virtually every Ruby program.

## Key Concepts

- **Block**: A chunk of code enclosed in `do...end` or `{...}` that you pass to a method. Blocks are not objects themselves, but can be captured as Procs.
- **`yield`**: A keyword that executes the block passed to the current method. It can pass arguments to the block and receive a return value.
- **`block_given?`**: A method that returns `true` if a block was passed to the current method, allowing you to write methods with optional block behavior.

## Real World Context

In a Rails application, blocks are everywhere. Database transactions wrap operations in a block (`ActiveRecord::Base.transaction { ... }`), file handling ensures cleanup (`File.open('log.txt') { |f| f.read }`), and testing frameworks use blocks to define test cases (`it 'works' { expect(result).to eq(42) }`). Every time you use `each`, `map`, or `select`, you are passing a block.

## Deep Dive

Blocks are chunks of code you can pass to methods. They are enclosed in `do...end` or `{...}`.

Here is the basic syntax for passing a block to a method:

```ruby
[1, 2, 3].each do |n|
  puts n
end

# Single-line style
[1, 2, 3].each { |n| puts n }
```

By convention, `do...end` is used for multi-line blocks and `{...}` for single-line blocks.

### Writing Methods That Accept Blocks

Use `yield` to execute the block passed to your method:

```ruby
def wrap
  puts \"Before\"
  yield           # Execute the block
  puts \"After\"
end

wrap { puts \"Inside\" }
# Output:
# Before
# Inside
# After
```

The `yield` keyword transfers control to the block, runs it, and then returns control to the method. This pattern is the foundation of Ruby's iterator design.

### Passing Arguments to Blocks

You can pass values from your method into the block via `yield`:

```ruby
def repeat(times)
  times.times do |i|
    yield i
  end
end

repeat(3) { |n| puts \"Iteration #{n}\" }
```

The value passed to `yield` becomes the block parameter (the variable between the pipes).

### The `it` Keyword (Ruby 3.4+)

Ruby 3.4 introduced `it` as a default block parameter for single-argument blocks, providing a concise alternative to numbered parameters (`_1`) or explicit names:

```ruby
# Traditional
[1, 2, 3].map { |n| n * 2 }   # => [2, 4, 6]

# With numbered parameter (Ruby 2.7+)
[1, 2, 3].map { _1 * 2 }       # => [2, 4, 6]

# With it (Ruby 3.4+)
[1, 2, 3].map { it * 2 }       # => [2, 4, 6]
```

The `it` keyword reads naturally and is ideal for simple one-argument blocks. It can only be used in blocks with no explicit parameters defined — you cannot mix `it` with named or numbered parameters.

### Checking if a Block Was Given

Use `block_given?` to make block usage optional in your methods:

```ruby
def maybe_yield
  if block_given?
    yield
  else
    puts \"No block provided\"
  end
end
```

This pattern lets you write methods that behave differently depending on whether the caller provided a block, which is common in Ruby's standard library.

## Common Pitfalls

1. **Calling `yield` without a block** — If you call `yield` and no block was given, Ruby raises a `LocalJumpError`. Always guard with `block_given?` if the block is optional.
2. **Confusing `do...end` and `{...}` precedence** — `method arg { ... }` binds the block to `arg`, while `method arg do ... end` binds it to `method`. This matters when passing blocks to methods with arguments.

## Best Practices

1. **Use `{...}` for single-line blocks and `do...end` for multi-line** — This convention is nearly universal in the Ruby community and makes code immediately scannable.
2. **Guard with `block_given?` for optional blocks** — If your method should work both with and without a block, always check `block_given?` before calling `yield`.

## Summary

- Blocks are chunks of code passed to methods, enclosed in `do...end` or `{...}`.
- The `yield` keyword executes the block from within the method, optionally passing arguments.
- Use `block_given?` to make block usage optional and avoid `LocalJumpError`.

## Code Examples

**Shows how to write methods that accept blocks using yield and block_given?**

```ruby
# Writing a method that accepts a block
def wrap
  puts "Before"
  yield           # Execute the block
  puts "After"
end

wrap { puts "Inside" }
# Output:
# Before
# Inside
# After

# Checking if a block was given
def maybe_yield
  if block_given?
    yield
  else
    puts "No block provided"
  end
end
```


## Resources

- [Proc Class](https://docs.ruby-lang.org/en/4.0/Proc.html) — Procs and Lambdas documentation

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*