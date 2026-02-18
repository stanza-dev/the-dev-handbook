---
source_course: "rails-foundations"
source_lesson: "rails-foundations-controller-filters"
---

# Controller Filters and Callbacks

Filters are methods that run before, after, or around controller actions. They're perfect for authentication, logging, and shared setup code.

## Before Actions

Run code before an action executes:

```ruby
class ArticlesController < ApplicationController
  before_action :set_article, only: [:show, :edit, :update, :destroy]
  before_action :authenticate_user!, except: [:index, :show]

  def show
    # @article is already set!
  end

  def edit
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

## Filter Options

### only and except

```ruby
# Run only for specific actions
before_action :authenticate, only: [:create, :update, :destroy]

# Run for all actions except these
before_action :authenticate, except: [:index, :show]
```

### if and unless

```ruby
before_action :check_admin, if: :admin_area?
before_action :load_sidebar, unless: -> { request.xhr? }
```

## After Actions

Run code after an action (but before response is sent):

```ruby
class ApplicationController < ActionController::Base
  after_action :log_activity

  private

  def log_activity
    logger.info "User #{current_user&.id} accessed #{request.path}"
  end
end
```

## Around Actions

Wrap an action (rarely used):

```ruby
class ArticlesController < ApplicationController
  around_action :measure_time

  private

  def measure_time
    start = Time.current
    yield  # Execute the action
    duration = Time.current - start
    logger.info "Action took #{duration} seconds"
  end
end
```

## Skipping Filters

Skip inherited filters in child controllers:

```ruby
class ApplicationController < ActionController::Base
  before_action :authenticate!
end

class PublicController < ApplicationController
  skip_before_action :authenticate!
end
```

## Common Patterns

### Authentication

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

### Resource Loading

```ruby
class ArticlesController < ApplicationController
  before_action :set_article, only: [:show, :edit, :update, :destroy]

  private

  def set_article
    @article = Article.find(params[:id])
  rescue ActiveRecord::RecordNotFound
    redirect_to articles_path, alert: "Article not found"
  end
end
```

### Halting the Filter Chain

Calling `redirect_to` or `render` in a before_action stops the chain:

```ruby
before_action :require_admin

def require_admin
  unless current_user&.admin?
    redirect_to root_path, alert: "Access denied"
    # Action won't run, subsequent filters won't run
  end
end
```

## Resources

- [Action Controller Overview - Filters](https://guides.rubyonrails.org/action_controller_overview.html#filters) — Official guide to controller filters

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*