# Blog Images Setup Guide

## Image Requirements

### You'll need **2 sizes** for each blog post:

#### 1. **Preview/Featured Image** (Blog Index)
- **Dimensions**: 400×300px (4:3 aspect ratio)  
- **Usage**: Blog index thumbnails, social media cards
- **File size**: < 50KB optimized
- **Format**: WebP preferred, JPG fallback

#### 2. **Full-Size Featured Image** (Individual Posts) 
- **Dimensions**: 800×600px or 1200×600px
- **Usage**: Blog post headers, detailed viewing
- **File size**: < 150KB optimized
- **Format**: WebP preferred, JPG fallback

## Current Blog Posts Need Images

### 1. Rails 8 Upgrade Post
**Suggested images:**
- Rails 8 logo/branding
- Before/after version comparison  
- Code migration screenshots
- Terminal/console screenshots

**Search terms:** "rails upgrade", "ruby on rails 8", "software update", "programming"

### 2. Performance Optimization Post
**Suggested images:**
- Performance charts/graphs
- Speed indicators or dashboards
- Server monitoring screenshots  
- Database query optimization visuals

**Search terms:** "performance optimization", "web performance", "speed charts", "analytics dashboard"

### 3. Legacy Debugging Post
**Suggested images:**
- Detective/magnifying glass imagery
- Code debugging screenshots
- Console/terminal with debugging
- Vintage computer or legacy system imagery

**Search terms:** "debugging", "detective work", "legacy systems", "code investigation"

## Where to Find Images

### Free Stock Photos
1. **Unsplash.com** - High-quality, free images
   - Search: "programming", "code", "developer", "technology"
   - License: Free for commercial use

2. **Pexels.com** - Free stock photos
   - Good for tech/programming imagery
   - License: Free for commercial use

3. **Pixabay.com** - Free images and vectors
   - Great for tech illustrations  
   - License: Free for commercial use

### Technical Screenshots
- **Your own Rails applications** (best option)
- **Rails documentation examples**
- **Terminal/console outputs**
- **Code editor screenshots**

## Image Optimization Process

### Step 1: Download/Create Images
- Download at highest quality available
- Take screenshots at 2x resolution if possible

### Step 2: Resize Images  
- Create 400×300px version for previews
- Create 800×600px version for full-size
- Maintain aspect ratio, crop if necessary

### Step 3: Optimize File Size
**Online tools:**
- TinyPNG.com - PNG/JPG compression
- Squoosh.app - Google's image optimizer
- Kraken.io - Image optimization service

**Mac apps:**
- ImageOptim - Free image optimizer
- Retina Image2x - Batch processing

### Step 4: Convert to WebP
- Use Squoosh.app for WebP conversion
- Keep JPG versions as fallbacks
- WebP typically 25-35% smaller file size

## File Naming Convention

```
rails-8-upgrade-400x300.webp    (preview)
rails-8-upgrade-800x600.webp    (full-size)  
rails-8-upgrade-400x300.jpg     (preview fallback)
rails-8-upgrade-800x600.jpg     (full-size fallback)
```

## Implementation in Posts

### In Post Front Matter:
```yaml
featured_image: "/assets/images/blog/rails-8-upgrade-400x300.jpg"
```

### For Responsive Images in Content:
```liquid
{% include responsive-image.html src="blog/rails-8-upgrade-800x600" alt="Rails 8 upgrade process" %}
```

## Quick Start for Your Blog

1. **Download 3-4 images** for each blog post from Unsplash
2. **Resize to 400×300px** using any image editor  
3. **Optimize with TinyPNG.com**
4. **Save to** `/assets/images/blog/` directory
5. **Update post front matter** with image paths

## Example Image Search Queries

- "ruby on rails programming code"
- "web development performance metrics"  
- "software debugging investigation"
- "database optimization charts"
- "terminal console programming"
- "code review developer"

This will give your blog a professional, polished appearance with proper imagery that enhances the content rather than distracting from it.