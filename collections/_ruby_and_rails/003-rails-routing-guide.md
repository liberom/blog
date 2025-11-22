---
layout: post
title: "Rails Routing: RESTful Routes and Beyond"
author: "Liberom"
date: 2025-01-13
category: "Rails"
toc: true
excerpt: "Master Rails routing: RESTful conventions, nested routes, route constraints, and advanced routing patterns."
---

# Rails Routing: RESTful Routes and Beyond

Rails routing maps URLs to controller actions. Understanding routing is critical for building clean, maintainable APIs.

## Basic RESTful Routing

### Resource Declaration

```ruby
# config/routes.rb
Rails.application.routes.draw do
  resources :posts              # Generates 7 RESTful routes
  resources :comments           # Nested routes possible
  resources :users              # Standard REST endpoints
end
```

This creates 7 routes automatically:

| HTTP Method | URL | Controller#Action | Purpose |
|-------------|-----|------------------|---------|
| GET | /posts | posts#index | List all posts |
| GET | /posts/new | posts#new | Form for new post |
| POST | /posts | posts#create | Create post |
| GET | /posts/:id | posts#show | View single post |
| GET | /posts/:id/edit | posts#edit | Edit form |
| PATCH/PUT | /posts/:id | posts#update | Update post |
| DELETE | /posts/:id | posts#destroy | Delete post |

### Customizing Resources

```ruby
# Limit actions
resources :posts, only: [:index, :show]
resources :comments, except: [:destroy]

# Rename path but keep model name
resources :articles, path: 'blog_posts'    # /blog_posts instead of /articles

# Change controller
resources :posts, controller: 'articles'   # Maps to ArticlesController

# Add custom action
resources :posts do
  member do
    patch :publish              # PATCH /posts/:id/publish
    post :archive               # POST /posts/:id/archive
  end

  collection do
    get :trending               # GET /posts/trending
    get :search                 # GET /posts/search
  end
end
```

## Nested Routes

Use nested routes for hierarchical resources:

```ruby
# config/routes.rb
resources :users do
  resources :posts              # GET /users/:user_id/posts
end

resources :posts do
  resources :comments           # GET /posts/:post_id/comments
  resources :likes              # GET /posts/:post_id/likes
end
```

### Shallow Nesting (Best Practice)

Deep nesting becomes unwieldy. Use shallow nesting:

```ruby
# BAD - Too deep
resources :users do
  resources :posts do
    resources :comments         # /users/:user_id/posts/:post_id/comments
  end
end

# GOOD - Shallow nesting
resources :users do
  resources :posts, shallow: true do
    resources :comments
  end
end

# Results in:
# /users/:user_id/posts           (index, new, create)
# /posts/:id                       (show, edit, update, delete)
# /posts/:post_id/comments         (index, new, create)
# /comments/:id                    (show, edit, update, delete)
```

## Route Constraints

Limit routes based on parameters:

```ruby
# Constraint on parameter format
resources :posts, constraints: { id: /\d+/ }   # Only numeric IDs

# Custom constraint
class AdminConstraint
  def matches?(request)
    request.session[:user_id] && User.find(request.session[:user_id]).admin?
  end
end

constraints AdminConstraint.new do
  resources :admin_panel
end

# Domain-based routing
namespace :admin do
  root 'dashboard#index'
  resources :users
end

# Subdomain routing
constraints(subdomain: 'api') do
  namespace :api do
    resources :posts
  end
end
```

## Scoped Routes

Group related routes without creating a new resource:

```ruby
# Namespace - creates module and path prefix
namespace :admin do
  resources :users              # Admin::UsersController at /admin/users
  resources :posts              # Admin::PostsController at /admin/posts
end

# Scope - only path prefix, no module
scope :api do
  resources :posts              # PostsController at /api/posts
end

# Scope with module
scope :api, module: 'api' do
  resources :posts              # Api::PostsController at /api/posts
end
```

## Advanced Routing Patterns

### Singleton Routes

When only one instance exists:

```ruby
resource :profile                # No index, no ID in path
resource :settings               # GET /profile, PATCH /profile
resource :dashboard              # No collection

# Routes generated:
# GET /profile         → profiles#show
# GET /profile/edit    → profiles#edit
# PATCH /profile       → profiles#update
# DELETE /profile      → profiles#destroy
```

### Glob Routes (Catch-All)

```ruby
# Match any remaining segments
get '/docs/*page', to: 'docs#show'

# Matches:
# /docs/getting-started → params[:page] = "getting-started"
# /docs/api/v1          → params[:page] = "api/v1"
```

### Redirect Routes

```ruby
# Permanent redirect (301)
get '/old-path', to: redirect('/new-path')

# With parameters
get '/users/:id', to: redirect('/profiles/%{id}')

# Conditional redirect with block
get '/legacy/*path', to: redirect { |params, request|
  "/api/v2/#{params[:path]}"
}
```

## Root Routes & Named Routes

```ruby
# Root route
root 'pages#home'               # GET / → pages#home

# Named routes
get 'about', to: 'pages#about', as: 'about'
post 'contact', to: 'pages#contact', as: 'create_contact'

# In views:
<%= link_to 'About', about_path %>
<%= link_to 'Home', root_path %>
```

## HTTP Verb Routing

```ruby
# Explicit verb matching
get 'search', to: 'search#new'
post 'search', to: 'search#create'

# Multiple verbs
match 'search', to: 'search#index', via: [:get, :post]

# Match all verbs
match 'catch-all', to: 'pages#error', via: :all
```

## Generating URLs

```ruby
# In controllers/views
link_to 'Show', post_path(@post)
link_to 'Edit', edit_post_path(@post)
link_to 'Delete', post_path(@post), method: :delete

# With nested routes
user_posts_path(@user)                    # /users/:user_id/posts
user_post_path(@user, @post)              # /users/:user_id/posts/:id
edit_user_post_path(@user, @post)         # /users/:user_id/posts/:id/edit

# In redirects
redirect_to post_path(@post)
redirect_to user_posts_path(@user)
```

## Common Routing Mistakes

```ruby
# BAD - Wrong order (more specific should come first)
resources :posts
get '/posts/trending'           # Never matched! Resources catch it first

# GOOD - Specific routes first
get '/posts/trending', to: 'posts#trending'
resources :posts

# BAD - Overcomplicated nesting
resources :companies do
  resources :departments do
    resources :teams do
      resources :members
    end
  end
end

# GOOD - Shallow after 2 levels
resources :companies do
  resources :departments, shallow: true do
    resources :teams, shallow: true do
      resources :members
    end
  end
end

# BAD - Forgot to generate routes after changes
# (common mistake: routes cached in development)

# GOOD - Clear route structure
namespace :api do
  namespace :v1 do
    resources :posts
    resources :comments
  end
end
```

## Testing Routes

```ruby
# config/routes.rb is a file, test it directly

# In spec/routing/posts_routing_spec.rb
describe 'posts routing' do
  it 'routes to posts#index' do
    expect(get: '/posts').to route_to('posts#index')
  end

  it 'routes to posts#show' do
    expect(get: '/posts/1').to route_to('posts#show', id: '1')
  end

  it 'routes to posts#create' do
    expect(post: '/posts').to route_to('posts#create')
  end
end
```

## Route Debugging

```ruby
# View all routes
rails routes

# Filter routes
rails routes -c posts          # Only posts controller
rails routes -g post           # Matching pattern

# In Rails console
app.routes.url_helpers.posts_path
app.routes.url_helpers.post_path(1)
```

## Conclusion

Rails routing principles:
- Use RESTful conventions by default
- Keep nesting shallow (max 2 levels deep)
- Use namespaces for logical grouping
- Prefer named routes over hardcoded paths
- Test critical routing logic
- Use `rails routes` to audit your routes

