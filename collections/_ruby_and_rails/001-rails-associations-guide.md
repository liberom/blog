---
layout: post
title: "Rails Associations: A Complete Guide"
author: "Liberom"
date: 2025-01-15
category: "Rails"
toc: true
excerpt: "Master Rails associations: has_many, belongs_to, has_many_through, and polymorphic relationships with practical examples."
---

# Rails Associations: A Complete Guide

Rails associations define relationships between models. Understanding them is critical for building well-structured applications.

## One-to-Many (has_many / belongs_to)

The most common association. One model "has many" instances of another.

```ruby
# app/models/user.rb
class User < ApplicationRecord
  has_many :posts, dependent: :destroy
end

# app/models/post.rb
class Post < ApplicationRecord
  belongs_to :user
end
```

### Usage
```ruby
user = User.find(1)
user.posts                    # Get all posts
user.posts.create(title: 'New Post')  # Create new post
user.posts.count              # Count posts
post.user                     # Get associated user
```

### Key Options
- `dependent: :destroy` - Delete associated records when parent is deleted
- `dependent: :nullify` - Set foreign key to NULL (soft delete)
- `dependent: :delete_all` - Delete without running callbacks
- `through: :association` - For many-to-many relationships

## Many-to-Many (has_many through)

Models are related through a join table.

```ruby
# app/models/student.rb
class Student < ApplicationRecord
  has_many :enrollments, dependent: :destroy
  has_many :courses, through: :enrollments
end

# app/models/course.rb
class Course < ApplicationRecord
  has_many :enrollments, dependent: :destroy
  has_many :students, through: :enrollments
end

# app/models/enrollment.rb
class Enrollment < ApplicationRecord
  belongs_to :student
  belongs_to :course
end
```

### Database Migration
```ruby
create_table :enrollments do |t|
  t.references :student, foreign_key: true
  t.references :course, foreign_key: true
  t.timestamps
end
```

### Usage
```ruby
student.courses              # All courses for student
course.students             # All students in course
student.enrollments         # Join records (useful for extra data)
student.courses << course   # Add course to student
```

## One-to-One (has_one / belongs_to)

One model owns exactly one other model.

```ruby
# app/models/user.rb
class User < ApplicationRecord
  has_one :profile, dependent: :destroy
end

# app/models/profile.rb
class Profile < ApplicationRecord
  belongs_to :user
end
```

### Usage
```ruby
user.profile                # Get profile
user.profile.destroy        # Destroy profile
profile.user               # Get user
```

## Polymorphic Associations

One model belongs to multiple different models.

```ruby
# app/models/comment.rb
class Comment < ApplicationRecord
  belongs_to :commentable, polymorphic: true
end

# app/models/post.rb
class Post < ApplicationRecord
  has_many :comments, as: :commentable
end

# app/models/photo.rb
class Photo < ApplicationRecord
  has_many :comments, as: :commentable
end
```

### Database Migration
```ruby
create_table :comments do |t|
  t.text :body
  t.references :commentable, polymorphic: true
  t.timestamps
end
```

### Usage
```ruby
post.comments               # Comments on post
photo.comments             # Comments on photo
comment.commentable        # Returns either Post or Photo
comment.commentable_type   # String: "Post" or "Photo"
```

## Common Pitfalls & Solutions

### N+1 Query Problem
```ruby
# BAD - Multiple queries
users = User.all
users.each { |user| puts user.posts.count }  # One query per user!

# GOOD - Single query with joins
users = User.includes(:posts)
users.each { |user| puts user.posts.count }  # No additional queries
```

### Circular Dependencies
```ruby
# BAD
class User < ApplicationRecord
  has_many :posts
end

class Post < ApplicationRecord
  belongs_to :user
  has_many :users  # Circular!
end

# GOOD - Use different association name
class Post < ApplicationRecord
  belongs_to :author, class_name: 'User'
end
```

### Foreign Key Constraints
```ruby
# Always add indices and constraints
create_table :posts do |t|
  t.references :user, foreign_key: true, null: false
end

# Equivalent to:
create_table :posts do |t|
  t.integer :user_id, null: false
  t.index :user_id
  t.foreign_key :users
end
```

## Best Practices

1. **Always specify dependent option** - Prevent orphaned records
2. **Use includes/joins** - Eliminate N+1 queries
3. **Add uniqueness constraints** - Prevent duplicates
4. **Use class_name for clarity** - When association name differs from model name
5. **Test associations** - Easy to break, easy to test

## Advanced: Scopes with Associations

```ruby
class User < ApplicationRecord
  has_many :posts
  has_many :published_posts, -> { where(published: true) }, class_name: 'Post'
  has_many :recent_posts, -> { order(created_at: :desc).limit(5) }, class_name: 'Post'
end

# Usage
user.published_posts        # Only published
user.recent_posts          # Last 5, ordered by recency
```

## Conclusion

Rails associations are powerful once you understand them. Always remember:
- Use `includes` for performance
- Specify `dependent` behavior
- Keep associations simple and semantic
- Test edge cases thoroughly
