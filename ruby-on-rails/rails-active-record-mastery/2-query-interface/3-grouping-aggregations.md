---
source_course: "rails-active-record-mastery"
source_lesson: "rails-active-record-mastery-grouping-aggregations"
---

# Grouping and Aggregations

Active Record provides powerful methods for grouping data and calculating aggregates like counts, sums, and averages.

## Basic Aggregations

```ruby
# Count
Article.count                    # Total articles
Article.where(published: true).count

# Sum
Order.sum(:total)                # Sum of all totals
Order.where(status: "completed").sum(:total)

# Average
Product.average(:price)          # Returns BigDecimal
Product.average(:rating).to_f    # Convert to float

# Minimum and Maximum
Product.minimum(:price)
Product.maximum(:price)
Order.maximum(:created_at)       # Most recent order

# Multiple at once
Product.pick(:min_price, :max_price)  # Returns array
```

## Grouping Data

```ruby
# Count by category
Article.group(:category_id).count
# => {1 => 10, 2 => 5, 3 => 8}

# Sum by status
Order.group(:status).sum(:total)
# => {"pending" => 1500, "completed" => 8500}

# Average by category
Product.group(:category_id).average(:price)

# Multiple groupings
Order.group(:status, :payment_method).count
# => {["completed", "credit_card"] => 50, ["completed", "paypal"] => 30}
```

## Grouping by Date

```ruby
# Group by day (PostgreSQL)
Order.group("DATE(created_at)").count

# Group by month
Order.group("DATE_TRUNC('month', created_at)").count

# Group by year
Article.group("EXTRACT(YEAR FROM created_at)").count

# For SQLite
Order.group("strftime('%Y-%m', created_at)").count
```

## Having Clause

Filter groups after aggregation:

```ruby
# Categories with more than 10 products
Product.group(:category_id)
       .having("COUNT(*) > ?", 10)
       .count

# Authors with average rating above 4
Article.group(:author_id)
       .having("AVG(rating) > ?", 4.0)
       .average(:rating)

# Orders totaling more than $1000
OrderItem.group(:order_id)
         .having("SUM(price * quantity) > ?", 1000)
         .sum("price * quantity")
```

## Select with Calculations

```ruby
# Custom calculated columns
Order.select("status, COUNT(*) as order_count, SUM(total) as total_amount")
     .group(:status)
     .map { |o| [o.status, o.order_count, o.total_amount] }

# With model objects
authors = Author.select(
  "authors.*,
   COUNT(articles.id) as articles_count,
   AVG(articles.rating) as avg_rating"
).joins(:articles)
 .group("authors.id")

authors.each do |author|
  puts "#{author.name}: #{author.articles_count} articles, #{author.avg_rating} avg"
end
```

## Distinct

```ruby
# Count distinct values
Article.distinct.count(:author_id)   # Number of unique authors

# Select distinct
Article.select(:category_id).distinct

# Distinct with pluck
Article.distinct.pluck(:category_id)
```

## Practical Examples

```ruby
# Dashboard statistics
stats = {
  total_orders: Order.count,
  revenue: Order.where(status: "completed").sum(:total),
  avg_order: Order.where(status: "completed").average(:total),
  top_categories: Product.joins(:order_items)
                         .group(:category_id)
                         .order("COUNT(*) DESC")
                         .limit(5)
                         .count
}

# Monthly sales report
monthly_sales = Order
  .where(created_at: 1.year.ago..)
  .where(status: "completed")
  .group("DATE_TRUNC('month', created_at)")
  .select("DATE_TRUNC('month', created_at) as month,
           COUNT(*) as orders,
           SUM(total) as revenue")
```

## Resources

- [Active Record Query Interface - Calculations](https://guides.rubyonrails.org/active_record_querying.html#calculations) — Guide to calculations and aggregations

---

> 📘 *This lesson is part of the [Active Record Mastery](https://stanza.dev/courses/rails-active-record-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*