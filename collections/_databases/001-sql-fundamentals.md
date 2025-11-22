---
layout: post
title: "SQL Fundamentals: Queries, Joins, and Optimization"
author: "Liberom"
date: 2025-01-10
category: "Databases"
toc: true
excerpt: "Master SQL: CRUD operations, joins, subqueries, and query optimization techniques."
---

# SQL Fundamentals: Queries, Joins, and Optimization

SQL (Structured Query Language) is essential for working with relational databases. Understanding SQL deeply leads to better application performance.

## Basic CRUD Operations

### SELECT - Reading Data

```sql
-- Select all columns
SELECT * FROM users;

-- Select specific columns
SELECT id, name, email FROM users;

-- Aliases
SELECT id AS user_id, name AS full_name FROM users;

-- DISTINCT - Remove duplicates
SELECT DISTINCT country FROM users;

-- LIMIT - Pagination
SELECT * FROM users LIMIT 10;
SELECT * FROM users LIMIT 10 OFFSET 20;  -- Page 3 with 10 per page

-- Ordering
SELECT * FROM users ORDER BY created_at DESC;
SELECT * FROM users ORDER BY name ASC, age DESC;
```

### WHERE - Filtering

```sql
-- Simple conditions
SELECT * FROM users WHERE age > 18;
SELECT * FROM users WHERE status = 'active';

-- Multiple conditions
SELECT * FROM users WHERE age > 18 AND country = 'USA';
SELECT * FROM users WHERE status = 'active' OR status = 'pending';
SELECT * FROM users WHERE NOT status = 'deleted';

-- IN operator
SELECT * FROM users WHERE id IN (1, 2, 3, 4, 5);
SELECT * FROM users WHERE country IN ('USA', 'Canada', 'Mexico');

-- BETWEEN
SELECT * FROM users WHERE age BETWEEN 18 AND 65;
SELECT * FROM users WHERE created_at BETWEEN '2024-01-01' AND '2024-12-31';

-- LIKE - Pattern matching
SELECT * FROM users WHERE name LIKE 'John%';      -- Starts with John
SELECT * FROM users WHERE name LIKE '%Smith';     -- Ends with Smith
SELECT * FROM users WHERE name LIKE '%oh%';       -- Contains oh

-- IS NULL
SELECT * FROM users WHERE phone IS NULL;
SELECT * FROM users WHERE phone IS NOT NULL;
```

### INSERT - Creating Data

```sql
-- Insert single row
INSERT INTO users (name, email, age) VALUES ('John', 'john@example.com', 30);

-- Insert multiple rows
INSERT INTO users (name, email, age) VALUES
  ('Alice', 'alice@example.com', 25),
  ('Bob', 'bob@example.com', 35),
  ('Charlie', 'charlie@example.com', 28);

-- Insert with default values
INSERT INTO users (name, email) VALUES ('Jane', 'jane@example.com');
-- age gets default value

-- Insert from SELECT
INSERT INTO users_archive (name, email, age)
SELECT name, email, age FROM users WHERE created_at < '2020-01-01';
```

### UPDATE - Modifying Data

```sql
-- Update single row
UPDATE users SET email = 'newemail@example.com' WHERE id = 1;

-- Update multiple columns
UPDATE users SET status = 'inactive', updated_at = NOW() WHERE last_login < '2023-01-01';

-- Update with expressions
UPDATE products SET price = price * 1.1;  -- 10% price increase
UPDATE users SET age = age + 1 WHERE birthday_today = true;

-- Conditional update
UPDATE orders SET status = 'shipped' WHERE status = 'processing' AND created_at < NOW() - INTERVAL '1 day';
```

### DELETE - Removing Data

```sql
-- Delete single row
DELETE FROM users WHERE id = 1;

-- Delete multiple rows
DELETE FROM users WHERE status = 'deleted';

-- Delete all (be careful!)
DELETE FROM users;

-- Safe deletion pattern
DELETE FROM users WHERE id IN (SELECT id FROM users WHERE status = 'inactive' LIMIT 100);
```

## Joins - Combining Tables

### INNER JOIN - Common Records

```sql
-- Get users and their posts
SELECT users.name, posts.title, posts.created_at
FROM users
INNER JOIN posts ON users.id = posts.user_id;

-- Aliases
SELECT u.name, p.title, p.created_at
FROM users u
INNER JOIN posts p ON u.id = p.user_id;

-- Multiple joins
SELECT u.name, p.title, c.body
FROM users u
INNER JOIN posts p ON u.id = p.user_id
INNER JOIN comments c ON p.id = c.post_id;

-- Join with WHERE
SELECT u.name, p.title
FROM users u
INNER JOIN posts p ON u.id = p.user_id
WHERE p.published = true;
```

### LEFT JOIN - All Left Records

```sql
-- All users, even without posts
SELECT u.name, p.title
FROM users u
LEFT JOIN posts p ON u.id = p.user_id;

-- Count posts per user (including users with no posts)
SELECT u.name, COUNT(p.id) as post_count
FROM users u
LEFT JOIN posts p ON u.id = p.user_id
GROUP BY u.id, u.name;

-- Find users with no posts
SELECT u.name
FROM users u
LEFT JOIN posts p ON u.id = p.user_id
WHERE p.id IS NULL;
```

### RIGHT JOIN, FULL OUTER JOIN

```sql
-- RIGHT JOIN - All right table records
SELECT u.name, p.title
FROM users u
RIGHT JOIN posts p ON u.id = p.user_id;

-- FULL OUTER JOIN - All records from both tables
SELECT u.name, p.title
FROM users u
FULL OUTER JOIN posts p ON u.id = p.user_id;

-- CROSS JOIN - Cartesian product (use carefully!)
SELECT u.name, c.name
FROM users u
CROSS JOIN categories c;
```

## Aggregation and Grouping

### Aggregate Functions

```sql
-- Count records
SELECT COUNT(*) FROM users;
SELECT COUNT(*) FROM users WHERE status = 'active';

-- Sum, Average, Min, Max
SELECT SUM(salary) FROM employees;
SELECT AVG(age) FROM users;
SELECT MIN(created_at) FROM posts;
SELECT MAX(price) FROM products;

-- Multiple aggregations
SELECT
  COUNT(*) as total_users,
  AVG(age) as average_age,
  MIN(age) as youngest,
  MAX(age) as oldest
FROM users;
```

### GROUP BY and HAVING

```sql
-- Count posts per user
SELECT user_id, COUNT(*) as post_count
FROM posts
GROUP BY user_id;

-- With column names
SELECT u.name, COUNT(p.id) as post_count
FROM users u
LEFT JOIN posts p ON u.id = p.user_id
GROUP BY u.id, u.name;

-- HAVING - Filter aggregations
SELECT user_id, COUNT(*) as post_count
FROM posts
GROUP BY user_id
HAVING COUNT(*) > 5;

-- Complex aggregation
SELECT
  country,
  COUNT(*) as user_count,
  AVG(age) as average_age
FROM users
WHERE status = 'active'
GROUP BY country
HAVING COUNT(*) > 10
ORDER BY user_count DESC;
```

## Subqueries and Common Table Expressions

### Subqueries

```sql
-- Subquery in WHERE
SELECT * FROM users WHERE id IN (
  SELECT user_id FROM posts WHERE published = true
);

-- Subquery with alias
SELECT u.name, user_posts.post_count
FROM users u
JOIN (
  SELECT user_id, COUNT(*) as post_count
  FROM posts
  GROUP BY user_id
) user_posts ON u.id = user_posts.user_id;

-- Correlated subquery
SELECT u.name, (
  SELECT COUNT(*) FROM posts WHERE user_id = u.id
) as post_count
FROM users u;
```

### Common Table Expressions (CTE)

```sql
-- WITH clause for cleaner queries
WITH active_users AS (
  SELECT id, name FROM users WHERE status = 'active'
)
SELECT au.name, COUNT(p.id) as post_count
FROM active_users au
LEFT JOIN posts p ON au.id = p.user_id
GROUP BY au.id, au.name;

-- Multiple CTEs
WITH user_posts AS (
  SELECT user_id, COUNT(*) as post_count
  FROM posts
  GROUP BY user_id
),
active_users AS (
  SELECT id, name FROM users WHERE status = 'active'
)
SELECT au.name, COALESCE(up.post_count, 0) as posts
FROM active_users au
LEFT JOIN user_posts up ON au.id = up.user_id;
```

## Query Optimization

### Indexing

```sql
-- Create index on frequently searched columns
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_posts_user_id ON posts(user_id);

-- Composite index for common queries
CREATE INDEX idx_posts_user_published ON posts(user_id, published);

-- View existing indexes
SELECT * FROM pg_indexes WHERE tablename = 'users';

-- Drop unused indexes
DROP INDEX idx_users_email;
```

### EXPLAIN ANALYZE - Understanding Query Performance

```sql
-- See query execution plan
EXPLAIN ANALYZE
SELECT u.name, COUNT(p.id) as post_count
FROM users u
LEFT JOIN posts p ON u.id = p.user_id
GROUP BY u.id, u.name;

-- Look for:
-- - Sequential Scans (should use indexes if possible)
-- - High cost values
-- - N+1 query patterns
```

### Common Performance Mistakes

```sql
-- BAD - N+1 Query Pattern
-- Get users
SELECT * FROM users;
-- Then for each user, get posts
SELECT * FROM posts WHERE user_id = ?;  -- Repeated N times

-- GOOD - Single join
SELECT u.id, u.name, COUNT(p.id) as post_count
FROM users u
LEFT JOIN posts p ON u.id = p.user_id
GROUP BY u.id, u.name;

-- BAD - LIKE with leading wildcard (won't use index)
SELECT * FROM users WHERE name LIKE '%john%';

-- GOOD - LIKE with leading character (uses index)
SELECT * FROM users WHERE name LIKE 'john%';

-- BAD - Type mismatch (forces conversion)
SELECT * FROM users WHERE id = '123';  -- id is integer

-- GOOD - Correct type
SELECT * FROM users WHERE id = 123;

-- BAD - Unnecessary functions (disables index)
SELECT * FROM users WHERE YEAR(created_at) = 2024;

-- GOOD - Range query (uses index)
SELECT * FROM users WHERE created_at >= '2024-01-01' AND created_at < '2025-01-01';
```

## Window Functions

```sql
-- Row number
SELECT
  id,
  name,
  salary,
  ROW_NUMBER() OVER (ORDER BY salary DESC) as rank
FROM employees;

-- Partition (GROUP BY alternative)
SELECT
  id,
  name,
  department,
  salary,
  AVG(salary) OVER (PARTITION BY department) as dept_avg_salary
FROM employees;

-- Lag/Lead
SELECT
  id,
  amount,
  LAG(amount) OVER (ORDER BY created_at) as previous_amount,
  LEAD(amount) OVER (ORDER BY created_at) as next_amount
FROM transactions;

-- Running total
SELECT
  id,
  amount,
  SUM(amount) OVER (ORDER BY created_at) as running_total
FROM transactions;
```

## Conclusion

SQL optimization principles:
- Use EXPLAIN ANALYZE to understand queries
- Create indices on frequently searched columns
- Join tables instead of running multiple queries
- Use GROUP BY and HAVING for aggregations
- Avoid leading wildcards in LIKE patterns
- Always use WHERE clauses to limit result sets
- Test queries with realistic data volumes

