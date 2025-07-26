---
layout: post
title: "Rails Performance Optimization: Beyond the Basics"
subtitle: "Advanced techniques to make your Rails application lightning fast"
date: 2024-11-28 09:30:00 -0500
author: "Wale Olaleye"
categories: [Rails, Performance]
tags: [optimization, performance, scaling, caching]
featured_image: "/assets/images/blog/placeholder-400x300.svg"
---

Performance optimization in Rails goes far beyond adding `includes` to avoid N+1 queries. After working with dozens of Rails applications, I've discovered that the most impactful optimizations often lie in the details that many developers overlook.

## The Performance Mindset

Before diving into specific techniques, it's crucial to adopt a performance-first mindset:

> Performance is not a feature you add at the end—it's a design principle you embrace from the beginning.

## Database Optimization Strategies

### 1. Index Strategy Beyond the Obvious

Most developers know to index foreign keys, but strategic indexing goes much deeper:

```ruby
# Don't just index individual columns
add_index :orders, :status

# Create composite indexes for common query patterns
add_index :orders, [:user_id, :status, :created_at]
add_index :orders, [:status, :created_at], where: "status IN ('pending', 'processing')"

# Partial indexes for common filters
add_index :users, :email, where: "active = true"
```

### 2. Query Optimization Techniques

```ruby
# Instead of this expensive operation
User.joins(:orders).where(orders: { status: 'completed' }).count

# Use exists? for better performance
User.where(id: Order.select(:user_id).where(status: 'completed')).count

# Or use a single query with window functions
User.joins(<<-SQL)
  LEFT JOIN (
    SELECT user_id, COUNT(*) as order_count 
    FROM orders 
    WHERE status = 'completed' 
    GROUP BY user_id
  ) completed_orders ON users.id = completed_orders.user_id
SQL
```

### 3. Connection Pool Optimization

```ruby
# config/database.yml
production:
  adapter: postgresql
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 25 } %>
  checkout_timeout: 5
  reaping_frequency: 10
  dead_connection_timeout: 5
  # Enable prepared statements for better performance
  prepared_statements: true
```

## Memory Management

### 1. Object Allocation Reduction

```ruby
# Memory-hungry approach
def format_prices(products)
  products.map { |product| "$#{product.price.to_f}" }
end

# Memory-efficient approach
def format_prices(products)
  products.pluck(:price).map { |price| "$#{price}" }
end

# Even better with database formatting
def format_prices(products)
  products.select("CONCAT('$', price) as formatted_price").pluck(:formatted_price)
end
```

### 2. Garbage Collection Tuning

```ruby
# config/puma.rb
# Tune GC for your workload
before_fork do
  # Compact memory before forking
  3.times { GC.start }
  GC.compact if GC.respond_to?(:compact)
end

# Set environment variables
ENV['RUBY_GC_HEAP_INIT_SLOTS'] = '800000'
ENV['RUBY_GC_HEAP_FREE_SLOTS'] = '600000'
ENV['RUBY_GC_HEAP_GROWTH_FACTOR'] = '1.25'
```

## Advanced Caching Strategies

### 1. Multi-Level Caching

```ruby
class ProductService
  def featured_products
    # Level 1: Memory cache (fastest)
    @featured_products ||= begin
      # Level 2: Redis cache (fast)
      Rails.cache.fetch("featured_products", expires_in: 1.hour) do
        # Level 3: Database with optimized query (slower)
        Product.featured.includes(:category, :images).limit(10)
      end
    end
  end
end
```

### 2. Smart Cache Invalidation

```ruby
class Product < ApplicationRecord
  after_update :invalidate_caches
  after_destroy :invalidate_caches
  
  private
  
  def invalidate_caches
    # Targeted cache invalidation
    Rails.cache.delete("product_#{id}")
    Rails.cache.delete("featured_products") if featured_changed?
    Rails.cache.delete_matched("category_#{category_id}_*")
    
    # Use cache versioning for atomic updates
    Rails.cache.increment("products_version")
  end
end
```

### 3. Database-Level Caching

```ruby
# Use materialized views for complex aggregations
class CreateProductStatsView < ActiveRecord::Migration[7.0]
  def up
    execute <<-SQL
      CREATE MATERIALIZED VIEW product_stats AS
      SELECT 
        products.id,
        products.name,
        AVG(reviews.rating) as avg_rating,
        COUNT(reviews.id) as review_count,
        COUNT(orders.id) as order_count
      FROM products
      LEFT JOIN reviews ON products.id = reviews.product_id
      LEFT JOIN order_items ON products.id = order_items.product_id
      LEFT JOIN orders ON order_items.order_id = orders.id
      GROUP BY products.id, products.name;
      
      CREATE UNIQUE INDEX ON product_stats (id);
    SQL
  end
end
```

## Asset and Frontend Optimization

### 1. Asset Pipeline Optimization

```ruby
# config/environments/production.rb
config.assets.compile = false
config.assets.compress = true
config.assets.css_compressor = :sass
config.assets.js_compressor = :terser

# Enable gzip compression
config.middleware.use Rack::Deflater

# Set far-future expires headers
config.public_file_server.headers = {
  'Cache-Control' => 'public, max-age=31536000'
}
```

### 2. Image Optimization

```ruby
class ImageUploader < CarrierWave::Uploader::Base
  include CarrierWave::MiniMagick
  
  # Create optimized versions
  version :thumb do
    process resize_to_fill: [300, 300]
    process optimize: [{ quality: 85 }]
  end
  
  version :medium do
    process resize_to_limit: [800, 600]
    process optimize: [{ quality: 90 }]
  end
  
  # Use WebP format for modern browsers
  version :webp do
    process convert: 'webp'
    process optimize: [{ quality: 85 }]
    
    def full_filename(for_file)
      super.chomp(File.extname(super)) + '.webp'
    end
  end
end
```

## Background Job Optimization

### 1. Job Performance Patterns

```ruby
class DataImportJob < ApplicationJob
  def perform(file_path)
    # Process in batches to avoid memory issues
    CSV.foreach(file_path, headers: true).each_slice(1000) do |batch|
      # Use bulk operations
      users_data = batch.map { |row| transform_row(row) }
      User.upsert_all(users_data, unique_by: :email)
      
      # Yield control to prevent blocking
      sleep(0.01) if batch.size == 1000
    end
  end
  
  private
  
  def transform_row(row)
    {
      email: row['email'],
      name: row['name'],
      created_at: Time.current,
      updated_at: Time.current
    }
  end
end
```

### 2. Queue Management

```ruby
# config/application.rb
config.active_job.queue_adapter = :sidekiq

# Prioritize critical jobs
class CriticalJob < ApplicationJob
  queue_as :critical, priority: 10
end

class RegularJob < ApplicationJob
  queue_as :default, priority: 5
end
```

## Monitoring and Profiling

### 1. Custom Performance Monitoring

```ruby
# config/initializers/performance_monitoring.rb
class PerformanceMonitor
  def self.track(operation_name)
    start_time = Time.current
    memory_before = `ps -o rss= -p #{Process.pid}`.to_i
    
    result = yield
    
    duration = Time.current - start_time
    memory_after = `ps -o rss= -p #{Process.pid}`.to_i
    memory_used = memory_after - memory_before
    
    Rails.logger.info({
      operation: operation_name,
      duration: duration,
      memory_used: memory_used,
      timestamp: Time.current
    }.to_json)
    
    result
  end
end

# Usage
def expensive_operation
  PerformanceMonitor.track("user_data_export") do
    User.includes(:orders, :preferences).find_each do |user|
      # Process user data
    end
  end
end
```

### 2. Query Performance Tracking

```ruby
# config/initializers/query_monitor.rb
ActiveSupport::Notifications.subscribe("sql.active_record") do |event|
  if event.duration > 100 # Log queries slower than 100ms
    Rails.logger.warn({
      type: "slow_query",
      duration: event.duration,
      sql: event.payload[:sql],
      backtrace: caller.first(5)
    }.to_json)
  end
end
```

## Production Optimization Checklist

### Server Configuration

- [ ] Enable gzip compression
- [ ] Configure proper caching headers
- [ ] Use a CDN for static assets
- [ ] Enable HTTP/2
- [ ] Implement proper load balancing

### Database Tuning

- [ ] Optimize connection pool size
- [ ] Enable query plan caching
- [ ] Configure appropriate work_mem
- [ ] Set up read replicas for read-heavy operations
- [ ] Monitor and optimize slow queries

### Application Monitoring

- [ ] Set up APM (Application Performance Monitoring)
- [ ] Monitor memory usage and garbage collection
- [ ] Track key business metrics
- [ ] Set up alerts for performance degradation
- [ ] Regular performance audits

## Real-World Performance Gains

From my experience optimizing Rails applications:

- **Database indexing**: 70% reduction in query time
- **Caching strategy**: 85% reduction in response time
- **Asset optimization**: 60% faster page loads
- **Memory management**: 40% reduction in server costs

## Conclusion

Performance optimization in Rails is an ongoing process that requires attention to detail and systematic measurement. The techniques covered here can dramatically improve your application's performance, but remember:

1. **Always measure before optimizing**
2. **Focus on the biggest bottlenecks first**
3. **Test thoroughly after each optimization**
4. **Monitor continuously in production**

Performance optimization is not just about making your app faster—it's about creating a better experience for your users and reducing operational costs.

---

*Need help optimizing your Rails application? [Schedule a consultation]({{ site.schedule_meeting_link }}) with Rails Fever to identify and fix your performance bottlenecks.*