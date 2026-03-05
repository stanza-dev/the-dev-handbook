---
source_course: "rails-foundations"
source_lesson: "rails-foundations-controllers-basics"
---

# Understanding Controllers

## Introduction

Controllers are the coordinators of your Rails application. They receive HTTP requests from the router, interact with models to fetch or modify data, and decide what response to send back to the browser.

## Key Concepts

- **Controller**: A Ruby class that inherits from `ApplicationController` and contains action methods.
- **Action**: A public method in a controller that handles a specific HTTP request.
- **Instance variables** (`@var`): Variables prefixed with `@` that are automatically shared with the view template.
- **Implicit rendering**: Rails automatically renders the view template matching the controller and action name.

## Real World Context

Controllers enforce the separation of concerns in MVC. Keeping controllers thin and pushing business logic into models is a key Rails best practice. A well-structured controller reads like a simple script: find the data, do the thing, respond.

## Deep Dive

### A Complete CRUD Controller

```ruby
class ArticlesController < ApplicationController
  def index
    @articles = Article.all
  end

  def show
    @article = Article.find(params[:id])
  end

  def new
    @article = Article.new
  end

  def create
    @article = Article.new(article_params)
    if @article.save
      redirect_to @article, notice: "Article created!"
    else
      render :new, status: :unprocessable_entity
    end
  end

  def edit
    @article = Article.find(params[:id])
  end

  def update
    @article = Article.find(params[:id])
    if @article.update(article_params)
      redirect_to @article, notice: "Article updated!"
    else
      render :edit, status: :unprocessable_entity
    end
  end

  def destroy
    @article = Article.find(params[:id])
    @article.destroy
    redirect_to articles_path, notice: "Article deleted!"
  end

  private
  def article_params
    params.require(:article).permit(:title, :body, :published)
  end
end
```

### Rendering vs Redirecting

```ruby
render :new                    # Render a view template
render json: @article          # Render JSON
redirect_to @article           # Send a redirect response
redirect_back fallback_location: root_path
```

**Key difference**: `render` displays a view within the current request. `redirect_to` sends a new HTTP request.

## Common Pitfalls

- **Fat controllers**: Putting business logic in controllers instead of models makes code hard to test and reuse.
- **Double render errors**: Calling both `render` and `redirect_to` in the same action raises an error. Use `return` after early renders.
- **Forgetting the `@` prefix**: Local variables (without `@`) are not available in views.

## Best Practices

- Keep controllers thin: move business logic to models or service objects.
- Use `redirect_to` after successful mutations (POST/PATCH/DELETE) to follow the Post-Redirect-Get pattern.
- Always set the HTTP status code on error renders (`:unprocessable_entity` for validation errors).

## Summary

- Controllers inherit from `ApplicationController` and contain action methods.
- Instance variables (`@article`) are automatically shared with views.
- Rails implicitly renders the view matching the controller/action name.
- Use `redirect_to` after successful writes, `render` for displaying forms with errors.
- Keep controllers thin by delegating logic to models.

## Code Examples

**A controller with index and create actions demonstrating implicit rendering, redirect after success, and re-rendering the form on validation failure.**

```ruby
class ArticlesController < ApplicationController
  def index
    @articles = Article.all
    # Renders app/views/articles/index.html.erb automatically
  end

  def create
    @article = Article.new(article_params)
    if @article.save
      redirect_to @article, notice: "Article created!"
    else
      render :new, status: :unprocessable_entity
    end
  end

  private
  def article_params
    params.require(:article).permit(:title, :body)
  end
end
```


## Resources

- [Action Controller Overview](https://guides.rubyonrails.org/action_controller_overview.html) — Complete guide to Rails controllers

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*