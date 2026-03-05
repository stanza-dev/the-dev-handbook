---
source_course: "rails-foundations"
source_lesson: "rails-foundations-controller-filters"
---

# Controller Filters and Callbacks

## Introduction

Filters (also called callbacks) are methods that run before, after, or around controller actions. They eliminate code duplication by extracting common setup, authentication checks, and logging into reusable hooks.

## Key Concepts

- **`before_action`**: Runs a method before the action executes. Used for authentication, loading resources, and setting up data.
- **`after_action`**: Runs after the action completes. Used for logging and modifying responses.
- **`only` / `except`**: Options that limit which actions a filter applies to.
- **`skip_before_action`**: Allows child controllers to opt out of inherited filters.

## Real World Context

Every Rails application uses `before_action` extensively. The most common patterns are authentication checks (`before_action :authenticate_user!`) and DRY resource loading (`before_action :set_article, only: [:show, :edit, :update, :destroy]`).

## Deep Dive

### Before Actions

```ruby
class ArticlesController < ApplicationController
  before_action :set_article, only: [:show, :edit, :update, :destroy]
  before_action :authenticate_user!, except: [:index, :show]

  def show
    # @article is already set!
  end

  private

  def set_article
    @article = Article.find(params[:id])
  end

  def authenticate_user!
    redirect_to login_path unless current_user
  end
end
```

### Skipping Filters

```ruby
class ApplicationController < ActionController::Base
  before_action :authenticate!
end

class PublicController < ApplicationController
  skip_before_action :authenticate!
end
```

### Halting the Chain

Calling `redirect_to` or `render` in a before_action stops the chain:

```ruby
before_action :require_admin

def require_admin
  unless current_user&.admin?
    redirect_to root_path, alert: "Access denied"
    # Action won't execute
  end
end
```

### Authentication Pattern

```ruby
class ApplicationController < ActionController::Base
  before_action :authenticate!

  private
  def authenticate!
    redirect_to login_path, alert: "Please log in" unless current_user
  end

  def current_user
    @current_user ||= User.find_by(id: session[:user_id])
  end
  helper_method :current_user
end
```

## Common Pitfalls

- **Overly broad filters**: Applying authentication to all actions when only some need it. Use `only:` or `except:` to be specific.
- **Not handling RecordNotFound**: A `set_article` before_action should rescue `ActiveRecord::RecordNotFound` to show a friendly error.
- **Forgetting `helper_method`**: Methods like `current_user` need `helper_method :current_user` to be accessible in views.

## Best Practices

- Use `before_action :set_resource` to DRY up resource loading across show, edit, update, destroy.
- Put authentication in `ApplicationController` and skip it where needed.
- Always use `only:` or `except:` to make filter scope explicit.

## Summary

- `before_action` runs code before actions for authentication, authorization, and resource loading.
- Use `only:` and `except:` to limit which actions a filter applies to.
- `skip_before_action` lets child controllers opt out of inherited filters.
- Calling `redirect_to` or `render` in a filter halts the action chain.
- Common patterns: authentication checks, DRY resource loading, authorization.

## Code Examples

**before_action callbacks DRY up resource loading and enforce authentication, running before the action method executes.**

```ruby
class ArticlesController < ApplicationController
  before_action :set_article, only: [:show, :edit, :update, :destroy]
  before_action :authenticate_user!, except: [:index, :show]

  def show
    # @article is already loaded by set_article
  end

  private
  def set_article
    @article = Article.find(params[:id])
  end
end
```


## Resources

- [Action Controller Overview - Filters](https://guides.rubyonrails.org/action_controller_overview.html#filters) — Official guide to controller filters

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*