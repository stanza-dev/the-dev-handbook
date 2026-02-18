---
source_course: "rails-foundations"
source_lesson: "rails-foundations-strong-parameters"
---

# Strong Parameters and Security

Strong Parameters protect your application from mass assignment vulnerabilities by explicitly whitelisting which parameters are allowed.

## The Problem: Mass Assignment

Imagine a User model with an `admin` attribute:

```ruby
# DANGEROUS - Never do this!
def create
  User.create(params[:user])
end
```

A malicious user could send `user[admin]=true` and make themselves an admin!

## The Solution: Strong Parameters

Rails requires you to explicitly permit parameters:

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

## Understanding params

The `params` hash contains all request parameters:

```ruby
# GET /articles?page=2&sort=title
params[:page]  # => "2"
params[:sort]  # => "title"

# POST /articles with form data
params[:article][:title]  # => "My Article"
params[:article][:body]   # => "Content..."
```

## Strong Parameters Methods

### require

Ensures a parameter exists:

```ruby
params.require(:article)
# Raises ActionController::ParameterMissing if :article is missing
```

### permit

Whitelists allowed attributes:

```ruby
params.require(:article).permit(:title, :body)
# Returns only title and body, ignores everything else
```

### Permitting Different Types

```ruby
# Simple attributes
params.require(:user).permit(:name, :email)

# Arrays
params.require(:article).permit(tag_ids: [])

# Nested attributes
params.require(:user).permit(:name, address_attributes: [:street, :city])

# Hash with any keys
params.require(:settings).permit(preferences: {})
```

## Complete Example

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

  def update
    @article = Article.find(params[:id])

    if @article.update(article_params)
      redirect_to @article
    else
      render :edit, status: :unprocessable_entity
    end
  end

  private

  def article_params
    params.require(:article).permit(:title, :body, :published, :category_id, tag_ids: [])
  end
end
```

## Rails 8: params.expect

Rails 8 introduces a more explicit syntax:

```ruby
def article_params
  params.expect(article: [:title, :body, :published])
end
```

This is functionally equivalent to `require().permit()` but more explicit about the structure.

## Resources

- [Action Controller Overview - Strong Parameters](https://guides.rubyonrails.org/action_controller_overview.html#strong-parameters) — Official guide to Strong Parameters

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*