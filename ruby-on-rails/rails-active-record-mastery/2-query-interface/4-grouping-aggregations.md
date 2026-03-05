---
source_course: "rails-active-record-mastery"
source_lesson: "rails-active-record-mastery-grouping-aggregations"
---

# Grouping and Aggregations

## Introduction
Active Record provides powerful methods for grouping data and calculating aggregates like counts, sums, and averages. These map directly to SQL `GROUP BY`, `HAVING`, and aggregate functions, letting you compute statistics without loading records into Ruby.

## Key Concepts
- **Aggregate methods**: `count`, `sum`, `average`, `minimum`, `maximum` -- each translates to the corresponding SQL function.
- **`group`**: Groups records by one or more columns, returning a hash of results.
- **`having`**: Filters groups after aggregation (like `where` but for grouped results).
- **`pick`**: Returns a single row's values as an array, useful for fetching one aggregate result.

## Real World Context
Dashboards, analytics pages, and reporting features all rely on aggregations. Computing monthly revenue, counting orders by status, or finding the top-selling category are everyday tasks. Doing these in SQL via Active Record is orders of magnitude faster than loading all records into Ruby and computing in memory.

## Deep Dive

### Basic Aggregations

```ruby
Article.count                    # Total articles
Order.sum(:total)                # Sum of all totals
Product.average(:price)          # Returns BigDecimal
Product.minimum(:price)
Product.maximum(:price)
```

These execute a single SQL query each and return a scalar value.

### Grouping Data

```ruby
Article.group(:category_id).count
# => {1 => 10, 2 => 5, 3 => 8}

Order.group(:status).sum(:total)
# => {"pending" => 1500, "completed" => 8500}

# Multiple groupings
Order.group(:status, :payment_method).count
# => {["completed", "credit_card"] => 50, ["completed", "paypal"] => 30}
```

The result is a hash where keys are the grouped values and values are the aggregate results.

### Having Clause

Filter groups after aggregation:

```ruby
Product.group(:category_id)
       .having("COUNT(*) > ?", 10)
       .count

Article.group(:author_id)
       .having("AVG(rating) > ?", 4.0)
       .average(:rating)
```

`having` is to `group` what `where` is to the full result set. It filters after the aggregation.

### Select with Calculations

```ruby
authors = Author.select(
  "authors.*,
   COUNT(articles.id) as articles_count,
   AVG(articles.rating) as avg_rating"
).joins(:articles)
 .group("authors.id")

authors.each do |author|
  puts "#{author.name}: #{author.articles_count} articles"
end
```

Custom select with aggregate functions lets you compute multiple metrics in a single query.

### Distinct

```ruby
Article.distinct.count(:author_id)   # Number of unique authors
Article.distinct.pluck(:category_id) # Unique category IDs
```

## Common Pitfalls
1. **Grouping without aggregation** -- Calling `group(:status)` without an aggregate method returns raw grouped relations. Always pair `group` with `count`, `sum`, etc.
2. **Mixing group and non-group columns** -- In strict SQL mode, selecting columns not in the GROUP BY clause raises an error. Only select grouped or aggregated columns.
3. **Using Ruby for aggregation** -- `Order.all.map(&:total).sum` loads every record. Use `Order.sum(:total)` instead.

## Best Practices
1. **Do math in SQL** -- Aggregations in SQL are faster and use less memory than loading records into Ruby.
2. **Use `having` for group filters** -- Do not filter aggregated results in Ruby; push the filter to the database.
3. **Add database indexes on grouped columns** -- If you frequently group by `status` or `category_id`, index those columns.

## Summary
- Active Record provides `count`, `sum`, `average`, `minimum`, and `maximum` as direct SQL aggregate wrappers.
- `group` returns a hash of grouped results; pair it with an aggregate method.
- `having` filters groups after aggregation.
- Always compute aggregations in SQL, not Ruby, for performance.
- Use `distinct` to count or pluck unique values.

## Code Examples

**Grouping and aggregation examples -- counting by status, summing revenue by month, and filtering groups with HAVING**

```ruby
# Count orders by status
Order.group(:status).count
# => {"pending" => 15, "completed" => 85, "cancelled" => 5}

# Monthly revenue
Order.where(status: "completed")
     .group("DATE_TRUNC('month', created_at)")
     .sum(:total)

# Categories with more than 10 products
Product.group(:category_id)
       .having("COUNT(*) > ?", 10)
       .count
```


## Resources

- [Active Record Query Interface - Calculations](https://guides.rubyonrails.org/active_record_querying.html#calculations) — Guide to calculations and aggregations

---

> 📘 *This lesson is part of the [Active Record Mastery](https://stanza.dev/courses/rails-active-record-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*