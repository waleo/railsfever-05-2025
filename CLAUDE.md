# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Jekyll-based static website for Rails Fever, a Rails consultancy company. The site is configured for GitHub Pages deployment and showcases various Rails consulting services.

## Development Commands

### Local Development
```bash
bundle exec jekyll serve
```
Starts the Jekyll development server with live reload.

### Building for Production
```bash
bundle exec jekyll build
```
Builds the static site for production deployment.

### Installing Dependencies
```bash
bundle install
```
Installs Ruby gems defined in the Gemfile.

## Architecture and Structure

### Site Configuration
- **Framework**: Jekyll static site generator
- **Theme**: Minima theme as base
- **Deployment**: GitHub Pages
- **Ruby Version**: 3.3.1 (use `rbenv local 3.3.1` - 3.4.1 has commonmarker compatibility issues)

### Key Configuration Files
- `_config.yml`: Main Jekyll configuration with site metadata, plugins, and build settings
- `Gemfile`: Ruby dependencies including GitHub Pages gems
- Site metadata includes business info, contact details, and Calendly scheduling link

### Content Structure
- `index.html`: Homepage with hero section, services grid, about section, testimonials, and contact
- `services/`: Individual service pages for each consulting offering
  - Rails Rescue Kit
  - Rails Upgrade Express  
  - Rails Tech Audit
  - Rails Care Plan
  - Rails Support Desk
  - Fractional CTO Service
- `_layouts/default.html`: Main site template with header, navigation, and footer

### Styling Architecture
- `assets/css/main.scss`: Main stylesheet that imports all component styles
- `_sass/_base.scss`: Base styles for typography, header, footer, buttons, and responsive navigation
- `_sass/_home.scss`: Homepage-specific styles
- `_sass/services/`: Service page styles organized by service type
  - Each service has its own SCSS partial for styling isolation

### Key Features
- Responsive design with mobile hamburger navigation
- Google Analytics integration (G-DS91BPQ010)
- Automatic sitemap generation via jekyll-sitemap plugin
- RSS feed generation via jekyll-feed plugin
- Professional branding with custom fonts (Montserrat, Roboto)

### Service Page Linking
When creating content (blog posts, pages), always link to service pages using the correct URL format:

**Correct format**: `/services/service_name/` (with trailing slash, no .html extension)

**Available service pages**:
- Rails Rescue Kit: `/services/rails_rescue_kit/`
- Rails Upgrade Express: `/services/rails_upgrade_express/`
- Rails Tech Audit: `/services/rails_tech_audit/`
- Rails Care Plan: `/services/rails_care_plan/`
- Rails Support Desk: `/services/rails_support_desk/`
- Fractional CTO Service: `/services/fractional_cto/`

**Examples**:
- ✅ `[Rails Care Plan](/services/rails_care_plan/)`
- ❌ `[Rails Care Plan](/services/rails_care_plan.html)`
- ❌ `[Rails Care Plan](/services/rails-care-plan/)`

### Business Context
Rails Fever is a consultancy specializing in Rails application modernization, upgrades, and maintenance. The site serves as a marketing platform showcasing expertise in legacy Rails app rescue, upgrades, audits, ongoing support, and fractional CTO services.