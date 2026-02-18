---
source_course: "rails-api-development"
source_lesson: "rails-api-development-api-authorization"
---

# API Authorization

Authentication verifies who you are. Authorization determines what you can do.

## Simple Role-Based Authorization

```ruby
# app/models/user.rb
class User < ApplicationRecord
  ROLES = %w[user editor admin].freeze

  validates :role, inclusion: { in: ROLES }

  def admin?
    role == "admin"
  end

  def editor?
    role.in?(%w[editor admin])
  end
end

# app/controllers/api/v1/base_controller.rb
module Api
  module V1
    class BaseController < ApplicationController
      before_action :authenticate_user!

      private

      def require_admin!
        unless current_user&.admin?
          render json: { error: "Forbidden" }, status: :forbidden
        end
      end

      def require_editor!
        unless current_user&.editor?
          render json: { error: "Forbidden" }, status: :forbidden
        end
      end
    end
  end
end

# Usage
class Api::V1::AdminController < Api::V1::BaseController
  before_action :require_admin!
end
```

## Resource-Based Authorization

```ruby
class Api::V1::ArticlesController < Api::V1::BaseController
  before_action :set_article, only: [:show, :update, :destroy]
  before_action :authorize_article!, only: [:update, :destroy]

  def update
    if @article.update(article_params)
      render json: @article
    else
      render json: { errors: @article.errors }, status: :unprocessable_entity
    end
  end

  private

  def set_article
    @article = Article.find(params[:id])
  end

  def authorize_article!
    unless can_modify?(@article)
      render json: { error: "Not authorized to modify this article" },
             status: :forbidden
    end
  end

  def can_modify?(article)
    current_user.admin? || article.author_id == current_user.id
  end
end
```

## Policy Objects

For complex authorization, use policy objects:

```ruby
# app/policies/article_policy.rb
class ArticlePolicy
  attr_reader :user, :article

  def initialize(user, article)
    @user = user
    @article = article
  end

  def show?
    article.published? || owned? || user&.admin?
  end

  def create?
    user.present?
  end

  def update?
    owned? || user&.admin?
  end

  def destroy?
    owned? || user&.admin?
  end

  def publish?
    user&.editor? && owned?
  end

  private

  def owned?
    article.author_id == user&.id
  end
end

# Usage in controller
class Api::V1::ArticlesController < Api::V1::BaseController
  def update
    @article = Article.find(params[:id])
    policy = ArticlePolicy.new(current_user, @article)

    unless policy.update?
      return render json: { error: "Forbidden" }, status: :forbidden
    end

    if @article.update(article_params)
      render json: @article
    else
      render json: { errors: @article.errors }, status: :unprocessable_entity
    end
  end
end
```

## Scoping Queries

```ruby
# app/policies/article_policy.rb
class ArticlePolicy
  class Scope
    def initialize(user, scope)
      @user = user
      @scope = scope
    end

    def resolve
      if user&.admin?
        scope.all
      elsif user
        scope.where("published = ? OR author_id = ?", true, user.id)
      else
        scope.where(published: true)
      end
    end

    private

    attr_reader :user, :scope
  end
end

# Usage
def index
  @articles = ArticlePolicy::Scope.new(current_user, Article).resolve
  render json: @articles
end
```

## Resources

- [Pundit GitHub](https://github.com/varvet/pundit) — Popular authorization library for Rails

---

> 📘 *This lesson is part of the [Building RESTful APIs with Rails](https://stanza.dev/courses/rails-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*