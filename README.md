# CodeCrafter-blog

A Jekyll-based tech blog for iOS, Swift, system design, and engineering notes.

## Prerequisites

- **Ruby** 2.6+ (Ruby 3.0+ recommended)
- **Bundler** gem (`gem install bundler`)

## Setup After Cloning

1. **Navigate to the blog directory**
   ```bash
   cd medium-blog
   ```

2. **Install dependencies**
   ```bash
   bundle install --path vendor/bundle
   ```

3. **Run locally**
   ```bash
   bundle exec jekyll serve
   ```

4. **Open in browser**
   ```
   http://127.0.0.1:4000/medium-blog/
   ```

## Adding New Posts

Create files in `medium-blog/_posts/` with format `YYYY-MM-DD-title.md`:

```yaml
---
layout: post
title: "Your Title"
date: YYYY-MM-DD
categories: [swift, ios]
---

Your content here...
```

## Troubleshooting

**Ruby version error:** Upgrade Ruby using `rbenv` or `rvm`

**Permission error:** Use `bundle install --path vendor/bundle`

## Deployment

Configured for GitHub Pages at: https://bholusingh.github.io/CodeCrafter-blog/medium-blog/
