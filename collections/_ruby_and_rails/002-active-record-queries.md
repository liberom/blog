---
layout: post
title: "ActiveRecord Queries: From Basics to Advanced"
author: "Liberom"
date: 2025-01-14
category: "Rails"
toc: true
excerpt: "Master ActiveRecord query methods, scopes, and performance optimization techniques."
---

# ActiveRecord Queries: From Basics to Advanced

ActiveRecord is Rails' ORM layer. Mastering queries is essential for efficient applications.

## Basic CRUD Operations

### Create
```ruby
# Single insert
user = User.create(name: 'John', email: 'john@example.com')
user = User.create!(name: 'John')  # Raises exception on failure

# Create without saving
user = User.new(name: 'John')
user.save
user.save!

# Create with block
user = User.create do |u|
  u.name = 'John'
  u.email = 'john@example.com'
end
```

### Read
```ruby
# Find by primary key
user = User.find(1)
user = User.find([1, 2, 3])  # Multiple records

# Find by attributes
user = User.find_by(email: 'john@example.com')
user = User.find_by!(email: 'john@example.com')  # Raises exception

# Get first/last
user = User.first
user = User.last
user = User.first(10)  # First 10
```

### Update
```ruby
# Update and save
user = User.find(1)
user.update(name: 'Jane', email: 'jane@example.com')
user.update!(name: 'Jane')  # Raises on validation error

# Update without callbacks
user.update_column(:name, 'Jane')
user.update_columns(name: 'Jane', email: 'jane@example.com')

# Update multiple records
User.where(status: 'inactive').update_all(status: 'active')
```

### Delete
```ruby
# Delete single record
user = User.find(1)
user.delete       # Executes DELETE without callbacks
user.destroy      # Executes callbacks first

# Delete multiple records
User.where(status: 'spam').delete_all    # No callbacks
User.where(status: 'spam').destroy_all   # With callbacks
User.ids.each { |id| User.find(id).destroy }  # Individual
```

## Query Methods

### Filtering
```ruby
# Where clause
User.where(status: 'active')                          # Column = value
User.where(name: ['John', 'Jane'])                    # Column IN [...]
User.where('age > ?', 18)                             # SQL inequality
User.where(age: 18..65)                               # Range
User.where('status = ?', 'active').where('age > 18') # Chaining (AND)
```

### Ordering & Limiting
```ruby
User.order(:name)                    # ASC by default
User.order(name: :asc, age: :desc)  # Multiple columns
User.order(created_at: :desc)       # Descending
User.limit(10)                       # TOP 10
User.offset(20)                      # Skip first 20
User.limit(10).offset(20)           # Pagination: page 3
User.order(:name).reverse_order     # Reverse order
```

### Aggregation
```ruby
User.count                           # Total count
User.where(status: 'active').count  # Conditional count
User.sum(:salary)                    # Sum column
User.average(:age)                   # Average
User.minimum(:age)                   # Min value
User.maximum(:age)                   # Max value
User.pluck(:email)                   # Array of values
User.pick(:name)                     # Single value
User.distinct.pluck(:status)         # Distinct values
```

### Existence Checks
```ruby
User.exists?                         # Any records?
User.where(age: 18).exists?         # Conditional existence
User.where(age: 18).any?            # Collection method
User.where(age: 18).none?           # None match?
User.where(age: 18).one?            # Exactly one?
```

## Scopes

Scopes are reusable query fragments.

```ruby
class User < ApplicationRecord
  # Simple scope
  scope :active, -> { where(status: 'active') }
  scope :older_than, ->(age) { where('age > ?', age) }
  scope :by_name, -> { order(:name) }

  # Complex scope
  scope :high_value, lambda {
    where('lifetime_value > ?', 10000)
      .includes(:orders)
      .order(lifetime_value: :desc)
  }

  # Class method (alternative)
  def self.active
    where(status: 'active')
  end
end

# Usage
User.active                          # Active users
User.older_than(30)                 # Users over 30
User.active.older_than(30)          # Chain scopes
User.active.older_than(30).by_name  # Multiple scopes
```

## Joins & Associations

```ruby
# Inner join
User.joins(:posts)                  # Users with posts
User.joins(:posts, :comments)       # Multiple tables

# Include (eager load)
User.includes(:posts)               # Prevents N+1
User.includes(posts: :comments)     # Nested

# Preload
User.preload(:posts)                # Similar to includes

# References (for where on joined table)
User.joins(:posts).where(posts: { published: true })
User.references(:posts)             # Tell Rails about joined table
```

## Where with Complex Conditions

```ruby
# AND conditions
User.where(status: 'active', role: 'admin')
User.where(status: 'active').where(role: 'admin')

# OR conditions
User.where('status = ? OR role = ?', 'admin', 'moderator')
User.or(User.where(status: 'admin')).where(role: 'moderator')

# NOT conditions
User.where.not(status: 'deleted')
User.where('status != ?', 'deleted')

# IN conditions
User.where(id: [1, 2, 3])
User.where(status: ['active', 'pending'])
```

## Advanced Techniques

### Subqueries
```ruby
# Users with orders
users_with_orders = User.where('id IN (?)', Order.select(:user_id))

# More efficient
users_with_orders = User.joins(:orders).distinct
```

### Batch Processing
```ruby
# Process large datasets efficiently
User.find_each(batch_size: 100) do |user|
  user.update_score!
end

User.find_in_batches(batch_size: 100) do |batch|
  batch.each { |user| user.process! }
end
```

### Raw SQL
```ruby
# Use SQL when needed
users = User.find_by_sql("SELECT * FROM users WHERE age > 18")
users = User.connection.execute("SELECT * FROM users")

# Sanitize input
User.find_by_sql(["SELECT * FROM users WHERE email = ?", params[:email]])
```

### Explain
```ruby
# Analyze query performance
User.where(status: 'active').explain
# Output: EXPLAIN ANALYZE from database
```

## Performance Tips

1. **Use includes, not joins** - Avoid N+1 queries
2. **Select only needed columns** - `User.select(:id, :name)`
3. **Use batch processing** - For large datasets
4. **Add database indices** - On frequently queried columns
5. **Avoid raw SQL** - Use ActiveRecord unless necessary
6. **Cache expensive queries** - Use Rails.cache
7. **Profile queries** - Use Rails Query Log or Rack Mini Profiler

## Common Mistakes

```ruby
# BAD - N+1 query
users = User.all
users.each { |u| puts u.posts.count }

# GOOD - Eager load
users = User.includes(:posts)
users.each { |u| puts u.posts.count }

# BAD - Loading all data
User.all.map(&:email)

# GOOD - Use pluck
User.pluck(:email)

# BAD - Inefficient
User.all.select { |u| u.age > 18 }

# GOOD - Filter in database
User.where('age > ?', 18)
```

## Conclusion

Master these query techniques:
- Use `where` and `select` for filtering
- Use `includes` for associations
- Use scopes for reusability
- Batch process large datasets
- Always check the SQL generated
