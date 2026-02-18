---
source_course: "ruby"
source_lesson: "ruby-blocks-yield"
---

# The Block

Blocks are chunks of code you can pass to methods. They're enclosed in `do...end` or `{...}`.

```ruby
[1, 2, 3].each do |n|
  puts n
end

# Single-line style
[1, 2, 3].each { |n| puts n }
```

## Writing Methods That Accept Blocks

Use `yield` to execute the block passed to your method:

```ruby
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
```

## Passing Arguments to Blocks

```ruby
def repeat(times)
  times.times do |i|
    yield i
  end
end

repeat(3) { |n| puts "Iteration #{n}" }
```

## Checking if a Block Was Given

```ruby
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