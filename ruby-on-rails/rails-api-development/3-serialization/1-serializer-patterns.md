---
source_course: "rails-api-development"
source_lesson: "rails-api-development-serializer-patterns"
---

# Serializer Patterns

Serializers give you control over exactly what data is included in your JSON responses. Let's explore different approaches.

## Plain Ruby Serializers

The simplest approach - a class that formats your data:

```ruby
# app/serializers/article_serializer.rb
class ArticleSerializer
  def initialize(article)
    @article = article
  end

  def as_json
    {
      id: @article.id,
      title: @article.title,
      body: @article.body,
      excerpt: excerpt,
      published: @article.published?,
      created_at: @article.created_at.iso8601,
      author: author_data,
      tags: tags_data
    }
  end

  private

  def excerpt
    @article.body.truncate(150)
  end

  def author_data
    return nil unless @article.author

    {
      id: @article.author.id,
      name: @article.author.name,
      avatar_url: @article.author.avatar_url
    }
  end

  def tags_data
    @article.tags.map { |t| { id: t.id, name: t.name } }
  end
end

# Usage in controller
def show
  @article = Article.find(params[:id])
  render json: ArticleSerializer.new(@article).as_json
end
```

## Collection Serializer

```ruby
# app/serializers/article_serializer.rb
class ArticleSerializer
  def initialize(article_or_articles, options = {})
    @options = options
    if article_or_articles.respond_to?(:each)
      @articles = article_or_articles
      @collection = true
    else
      @article = article_or_articles
      @collection = false
    end
  end

  def as_json
    if @collection
      { data: @articles.map { |a| serialize_one(a) } }
    else
      { data: serialize_one(@article) }
    end
  end

  private

  def serialize_one(article)
    data = {
      id: article.id,
      type: "article",
      attributes: {
        title: article.title,
        published: article.published?
      }
    }

    # Include full body only for single article
    data[:attributes][:body] = article.body unless @collection

    # Include associations if requested
    if @options[:include_author]
      data[:author] = serialize_author(article.author)
    end

    data
  end

  def serialize_author(author)
    return nil unless author
    { id: author.id, name: author.name }
  end
end

# Usage
render json: ArticleSerializer.new(@articles).as_json
render json: ArticleSerializer.new(@article, include_author: true).as_json
```

## Using Alba Gem

Alba is a fast, flexible serialization library:

```ruby
# Gemfile
gem 'alba'

# app/serializers/article_resource.rb
class ArticleResource
  include Alba::Resource

  attributes :id, :title, :created_at

  attribute :excerpt do |article|
    article.body.truncate(150)
  end

  attribute :published do |article|
    article.published?
  end

  one :author, resource: AuthorResource
  many :tags, resource: TagResource
end

class AuthorResource
  include Alba::Resource
  attributes :id, :name, :email
end

# Usage
render json: ArticleResource.new(@article).serialize
render json: ArticleResource.new(@articles).serialize
```

## Conditional Attributes

```ruby
class ArticleSerializer
  def initialize(article, current_user: nil)
    @article = article
    @current_user = current_user
  end

  def as_json
    data = public_attributes
    data.merge!(private_attributes) if can_see_private?
    data.merge!(admin_attributes) if admin?
    data
  end

  private

  def public_attributes
    { id: @article.id, title: @article.title }
  end

  def private_attributes
    { body: @article.body, draft_notes: @article.draft_notes }
  end

  def admin_attributes
    { internal_id: @article.internal_id, metrics: @article.metrics }
  end

  def can_see_private?
    @current_user&.id == @article.author_id
  end

  def admin?
    @current_user&.admin?
  end
end
```

## Resources

- [Alba Gem](https://github.com/okuramasafumi/alba) — Fast and flexible serialization library

---

> 📘 *This lesson is part of the [Building RESTful APIs with Rails](https://stanza.dev/courses/rails-api-development) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*