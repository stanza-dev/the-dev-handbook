---
source_course: "rails-foundations"
source_lesson: "rails-foundations-controllers-basics"
---

# Understanding Controllers

Controllers are the middlemen of your Rails application. They receive HTTP requests, interact with models, and render responses.

## Controller Basics

Generate a controller:

```bash
bin/rails generate controller Articles index show new create
```

This creates `app/controllers/articles_controller.rb`:

```ruby
class ArticlesController < ApplicationController
  def index
    # Handle GET /articles
  end

  def show
    # Handle GET /articles/:id
  end

  def new
    # Handle GET /articles/new
  end

  def create
    # Handle POST /articles
  end
end
```

## Controller Actions

Each public method in a controller is an "action" that handles requests:

```ruby
class ArticlesController < ApplicationController
  # GET /articles
  def index
    @articles = Article.all
    # Automatically renders app/views/articles/index.html.erb
  end

  # GET /articles/:id
  def show
    @article = Article.find(params[:id])
    # Automatically renders app/views/articles/show.html.erb
  end

  # GET /articles/new
  def new
    @article = Article.new
    # Automatically renders app/views/articles/new.html.erb
  end

  # POST /articles
  def create
    @article = Article.new(article_params)

    if @article.save
      redirect_to @article, notice: "Article created!"
    else
      render :new, status: :unprocessable_entity
    end
  end

  # GET /articles/:id/edit
  def edit
    @article = Article.find(params[:id])
  end

  # PATCH/PUT /articles/:id
  def update
    @article = Article.find(params[:id])

    if @article.update(article_params)
      redirect_to @article, notice: "Article updated!"
    else
      render :edit, status: :unprocessable_entity
    end
  end

  # DELETE /articles/:id
  def destroy
    @article = Article.find(params[:id])
    @article.destroy
    redirect_to articles_path, notice: "Article deleted!"
  end
end
```

## Instance Variables

Variables prefixed with `@` are available in views:

```ruby
def show
  @article = Article.find(params[:id])  # Available in view
  article = Article.last                 # NOT available in view
end
```

## Rendering and Redirecting

### Implicit Rendering

Rails automatically renders `app/views/controller_name/action_name.html.erb`:

```ruby
def index
  @articles = Article.all
  # Automatically renders app/views/articles/index.html.erb
end
```

### Explicit Rendering

```ruby
render :new                    # Render app/views/articles/new.html.erb
render "products/show"         # Render different controller's view
render plain: "Hello"          # Render plain text
render json: @article          # Render JSON
render status: :not_found      # Render with status code
```

### Redirecting

```ruby
redirect_to articles_path      # Redirect to index
redirect_to @article           # Redirect to show (uses article_path(@article))
redirect_to root_path          # Redirect to homepage
redirect_back fallback_location: root_path  # Go back or fallback
```

**Key difference**: `render` displays a view, `redirect_to` sends a new HTTP request.

## Resources

- [Action Controller Overview](https://guides.rubyonrails.org/action_controller_overview.html) — Complete guide to Rails controllers

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*