---
source_course: "rails-foundations"
source_lesson: "rails-foundations-strong-parameters"
---

# Strong Parameters and Security

## Introduction

Strong Parameters protect your application from mass assignment vulnerabilities by requiring you to explicitly whitelist which request parameters are allowed. This is a critical security feature built into Rails.

## Key Concepts

- **Mass assignment**: Passing a hash of attributes directly to `create` or `update`. Without protection, attackers can set fields they should not have access to.
- **`params.require`**: Ensures a top-level parameter key exists, raising an error if missing.
- **`params.permit`**: Whitelists specific attributes, silently ignoring any unpermitted ones.
- **`params.expect`**: A more explicit alternative that combines require and permit.

## Real World Context

Without Strong Parameters, a malicious user could add `user[admin]=true` to a form submission and grant themselves admin access. Strong Parameters ensure that only the attributes you explicitly allow can be set through user input.

## Deep Dive

### The Problem

```ruby
# DANGEROUS - Never do this!
def create
  User.create(params[:user])  # Attacker could set admin=true!
end
```

### The Solution

```ruby
class ArticlesController < ApplicationController
  def create
    @article = Article.new(article_params)
    # ...
  end

  private
  def article_params
    params.require(:article).permit(:title, :body, :published)
  end
end
```

### Permitting Different Types

```ruby
# Simple attributes
params.require(:user).permit(:name, :email)

# Arrays
params.require(:article).permit(tag_ids: [])

# Nested attributes
params.require(:user).permit(:name, address_attributes: [:street, :city])
```

### params.expect

Rails provides a more explicit syntax:

```ruby
def article_params
  params.expect(article: [:title, :body, :published])
end
```

This is functionally equivalent to `require().permit()` but more explicit about the expected structure.

## Common Pitfalls

- **Permitting too many attributes**: Only permit what the current action needs. An admin controller might permit different fields than a public one.
- **Forgetting nested params**: Arrays and nested hashes need special syntax (`tag_ids: []`, `address: [:street]`).
- **Not using strong params at all**: Passing raw `params` to `create` or `update` raises `ActiveModel::ForbiddenAttributesError`.

## Best Practices

- Define a private `_params` method for each resource (e.g., `article_params`, `user_params`).
- Be explicit about what is permitted; never use `permit!` which allows everything.
- Use different parameter methods for different roles if needed.

## Summary

- Strong Parameters prevent mass assignment attacks by whitelisting allowed attributes.
- Use `params.require(:key).permit(:attr1, :attr2)` to filter incoming data.
- Passing raw params to `create`/`update` raises `ForbiddenAttributesError`.
- Arrays need `field: []` syntax; nested hashes need `field: [:subfield]`.
- `params.expect` provides a more explicit alternative syntax.

## Code Examples

**Strong Parameters whitelist allowed attributes in a private method, preventing mass assignment vulnerabilities.**

```ruby
class ArticlesController < ApplicationController
  def create
    @article = Article.new(article_params)
    if @article.save
      redirect_to @article
    else
      render :new, status: :unprocessable_entity
    end
  end

  private
  def article_params
    params.require(:article).permit(:title, :body, :published, tag_ids: [])
  end
end
```


## Resources

- [Action Controller Overview - Strong Parameters](https://guides.rubyonrails.org/action_controller_overview.html#strong-parameters) — Official guide to Strong Parameters

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*