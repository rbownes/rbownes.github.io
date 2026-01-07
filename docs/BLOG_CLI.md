# Blog CLI Documentation

A command-line tool for managing your Jekyll blog with draft workflow, Obsidian-style tag linking, and automated publishing.

## Table of Contents

- [Installation](#installation)
- [Quick Start](#quick-start)
- [Commands Reference](#commands-reference)
- [Configuration](#configuration)
- [Tag Syntax](#tag-syntax)
- [Workflows](#workflows)
- [Troubleshooting](#troubleshooting)
- [FAQ](#faq)

## Installation

The blog CLI tool is already installed in your repository under `script/blog`.

### Prerequisites

Required:
- bash (>= 4.0)
- vim (or another text editor)
- git
- grep, sed, date (standard Unix utilities)

Optional:
- jq (recommended for tag indexing features)

### Verify Installation

```bash
script/blog version
script/blog help
```

## Quick Start

### 1. Create a New Draft

```bash
script/blog new "My First Blog Post"
```

This will:
- Create a file `_drafts/2026-01-06-my-first-blog-post.md`
- Populate it with Jekyll front matter
- Open it in vim for editing

### 2. Add Tags to Your Post

While writing your post, use Obsidian-style `[[tag]]` syntax:

```markdown
I'm learning about [[machine-learning]] and [[python]].
This relates to [[artificial-intelligence]] concepts.
```

### 3. Process Tags

After writing, process tags to create tag index and cross-references:

```bash
script/blog process
```

This will:
- Extract all `[[tag]]` references
- Create tag files in `_drafts/tags/`
- Build tag index in `.blog/tag-index.json`
- Generate backlinks in tag files
- **Add "Related Tags" section to your post**

### 4. Publish When Ready

Move your draft to `_posts/` for publication:

```bash
script/blog publish 2026-01-06-my-first-blog-post.md
```

This will:
- Validate front matter
- Update publish date
- Move file to `_posts/`
- **Publish tag files to `_posts/tags/`**
- Update tag index
- (Optional) Commit and push to git

### 5. Deploy

Push to GitHub to trigger the deployment pipeline:

```bash
git push origin main
```

Your post will be live at `https://rbownes.github.io/` within 30-40 seconds!

## Commands Reference

### `blog new <title>`

Create a new draft post with the specified title.

**Usage:**
```bash
script/blog new "My Post Title"
script/blog new Getting Started with AI
```

**Arguments:**
- `<title>` - Post title (required). Can be quoted or unquoted.

**Behavior:**
- Generates filename: `YYYY-MM-DD-<sanitized-title>.md`
- Creates file in `_drafts/`
- Adds Jekyll front matter with title, date, layout
- Opens in editor (vim by default)

**Filename Sanitization:**
- Converts to lowercase
- Replaces spaces with hyphens
- Removes special characters
- Example: "AI & Machine Learning!" → "ai-machine-learning"

---

### `blog process [file]`

Process tags in draft post(s).

**Usage:**
```bash
# Process all drafts
script/blog process

# Process specific file
script/blog process 2026-01-06-my-post.md
script/blog process _drafts/2026-01-06-my-post.md
```

**Arguments:**
- `[file]` - Optional. Specific file to process. If omitted, processes all drafts.

**Behavior:**
- Scans for `[[tag]]` syntax
- Creates missing tag files in `_drafts/tags/`
- Updates tag index (`.blog/tag-index.json`)
- Generates backlinks in tag files
- Preserves original `[[tag]]` syntax (non-invasive)

---

### `blog publish <file>`

Publish a draft to `_posts/`.

**Usage:**
```bash
script/blog publish 2026-01-06-my-post.md
script/blog publish _drafts/2026-01-06-my-post.md
```

**Arguments:**
- `<file>` - Draft filename (required). Can be with or without `_drafts/` prefix.

**Behavior:**
- Validates front matter (checks required fields)
- Updates `date:` field to current timestamp
- Moves file from `_drafts/` to `_posts/`
- Updates tag index with new file path
- Optionally commits and pushes to git (if `BLOG_AUTO_COMMIT=true`)

**Validation:**
- Checks for required front matter fields: `layout`, `title`, `date`
- Ensures file exists before publishing
- Prompts before overwriting existing files

---

### `blog list [--drafts|--tags]`

List drafts or tags.

**Usage:**
```bash
# List all drafts (default)
script/blog list
script/blog list --drafts

# List all tags
script/blog list --tags
```

**Output (--drafts):**
```
DRAFTS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
2026-01-06  My First Post              [3 tags]
2026-01-05  Testing Blog CLI           [1 tag]
2026-01-04  Machine Learning Basics    [5 tags]
```

**Output (--tags):**
```
TAG INDEX
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
machine-learning (3 references)
ai (5 references)
python (2 references)
```

---

### `blog status`

Show comprehensive blog status.

**Usage:**
```bash
script/blog status
```

**Output:**
```
BLOG STATUS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Drafts:          3
Published posts: 12
Total tags:      8

RECENT DRAFTS:
2026-01-06  My First Post
2026-01-05  Testing Blog CLI
```

---

### `blog help`

Display help information.

**Usage:**
```bash
script/blog help
```

---

### `blog version`

Display version information.

**Usage:**
```bash
script/blog version
```

## Configuration

Edit `.blog/config` to customize behavior.

### Configuration Options

```bash
# Editor to use (default: vim)
BLOG_EDITOR=vim

# Drafts directory (default: _drafts)
BLOG_DRAFTS_DIR=_drafts

# Published posts directory (default: _posts)
BLOG_POSTS_DIR=_posts

# Tag index location (default: .blog/tag-index.json)
BLOG_TAG_INDEX=.blog/tag-index.json

# Default post layout (default: post)
BLOG_DEFAULT_LAYOUT=post

# Auto-commit on publish (default: false)
BLOG_AUTO_COMMIT=false

# Tag files directory (default: _drafts/tags)
BLOG_TAG_DIR=_drafts/tags

# Auto-publish tag files when publishing posts (default: true)
BLOG_AUTO_PUBLISH_TAGS=true

# Add "Related Tags" section to posts during process (default: true)
BLOG_ADD_RELATED_TAGS=true

# Related tags section heading (default: "## Related Tags")
BLOG_RELATED_TAGS_HEADING="## Related Tags"
```

### Using a Different Editor

To use a different editor (e.g., nano, emacs):

```bash
# In .blog/config
BLOG_EDITOR=nano
```

Or set environment variable:

```bash
export BLOG_EDITOR=nano
script/blog new "My Post"
```

### Enable Auto-Commit

To automatically commit and push when publishing:

```bash
# In .blog/config
BLOG_AUTO_COMMIT=true
```

When enabled:
- `git add` is run automatically
- Commit message is auto-generated
- You'll be prompted before pushing

## Tag Syntax

### Creating Tags

Use double square brackets to create tags:

```markdown
I'm writing about [[machine-learning]] and [[python]].
```

### Tag Naming

- Use lowercase and hyphens for multi-word tags
- Examples: `[[ai]]`, `[[deep-learning]]`, `[[neural-networks]]`
- Avoid special characters

### How Tags Work

1. **Write** - Add `[[tag]]` references in your posts
2. **Process** - Run `blog process` to extract tags
3. **Index** - Tags are registered in `.blog/tag-index.json`
4. **Files** - Tag files created in `_drafts/tags/YYYY-MM-DD-tag.md`
5. **Backlinks** - Tag files list all posts that reference them
6. **Related Tags** - Posts automatically get a "Related Tags" section with links to tag pages
7. **Publish** - Tag files are published to `_posts/tags/` when you publish a post
8. **Bidirectional** - Full bidirectional navigation between posts and tags!

### Tag Files

Automatically generated tag files look like:

```markdown
---
layout: post
title: "Tag: machine-learning"
date: 2026-01-06 10:00:00 -0500
categories: [tags]
tags: [meta]
---

# Tag: machine-learning

This is an automatically generated tag index.

## Posts referencing this tag:

- [My First Post](../2026-01-06-my-first-post.md)
- [ML Basics](../2026-01-05-ml-basics.md)
```

### Related Tags in Posts

When you run `blog process`, a "Related Tags" section is automatically added to your posts:

```markdown
---

## Related Tags

- [machine-learning](tags/2026-01-07-machine-learning.md)
- [python](tags/2026-01-07-python.md)
- [artificial-intelligence](tags/2026-01-07-artificial-intelligence.md)
```

**Key features:**
- **Automatic** - Added during `blog process`
- **Idempotent** - Running process multiple times updates the section, doesn't duplicate
- **Non-invasive** - Original `[[tag]]` syntax is preserved in your content
- **Portable** - Relative paths work from both `_drafts/` and `_posts/`
- **Clickable** - When Jekyll renders the page, these become clickable links

**To disable:** Set `BLOG_ADD_RELATED_TAGS=false` in `.blog/config`

### Tag Index Structure

`.blog/tag-index.json` stores tag metadata:

```json
{
  "version": "1.0",
  "tags": {
    "machine-learning": {
      "file": "_drafts/tags/2026-01-06-machine-learning.md",
      "referenced_by": [
        "_drafts/2026-01-06-my-post.md",
        "_posts/2026-01-05-another-post.md"
      ]
    }
  },
  "last_updated": "2026-01-06T23:45:00Z"
}
```

## Workflows

### Standard Blog Post Workflow

```bash
# 1. Create draft
script/blog new "Understanding Neural Networks"

# 2. Write content with tags
# Add [[deep-learning]], [[ai]], [[neural-networks]]

# 3. Process tags
script/blog process

# 4. Review tag files (optional)
script/blog list --tags

# 5. Publish
script/blog publish 2026-01-06-understanding-neural-networks.md

# 6. Commit and push
git add _posts/2026-01-06-understanding-neural-networks.md
git commit -m "Publish: Understanding Neural Networks"
git push origin main

# 7. Verify deployment
# Visit https://rbownes.github.io/
```

### Batch Tag Processing

Process all drafts at once:

```bash
# After writing multiple drafts with tags
script/blog list --drafts

# Process all at once
script/blog process

# Review tags
script/blog list --tags

# Publish individually
script/blog publish 2026-01-06-post-1.md
script/blog publish 2026-01-05-post-2.md
```

### Editing Existing Drafts

```bash
# Open draft manually
vim _drafts/2026-01-06-my-post.md

# Or use your editor
code _drafts/2026-01-06-my-post.md

# Process after editing
script/blog process
```

## Troubleshooting

### "Required dependency not found"

**Problem:** Missing required tools

**Solution:**
```bash
# Check dependencies
which bash vim grep sed date git

# Install missing tools (macOS)
brew install vim git

# Install jq (optional but recommended)
brew install jq
```

---

### "No front matter found"

**Problem:** Post is missing front matter or has invalid format

**Solution:**

Ensure your post starts with:
```yaml
---
layout: post
title: "Your Title"
date: 2026-01-06 10:00:00 -0500
categories: []
tags: []
---
```

---

### "Draft not found"

**Problem:** File doesn't exist or wrong path

**Solution:**
```bash
# List drafts to see available files
script/blog list --drafts

# Use correct filename (with or without _drafts/ prefix)
script/blog publish 2026-01-06-my-post.md
```

---

### "jq not found" Warning

**Problem:** jq not installed (optional dependency)

**Solution:**
```bash
# Install jq (macOS)
brew install jq

# Or continue without jq (basic features still work)
```

---

### Tags Not Being Processed

**Problem:** `blog process` doesn't find tags

**Checklist:**
- Ensure tags use `[[double-brackets]]` syntax
- Check that file is in `_drafts/` directory
- Run with specific file: `script/blog process <file>`
- Verify tags aren't inside code blocks

---

### Permission Denied

**Problem:** Can't execute blog script

**Solution:**
```bash
# Make script executable
chmod +x script/blog
```

## FAQ

### How do I change the editor?

Edit `.blog/config` and set `BLOG_EDITOR=nano` (or your preferred editor).

### Can I track drafts in git?

Yes! By default, `_drafts/` is not ignored. You can commit drafts to track work in progress:

```bash
git add _drafts/
git commit -m "WIP: Draft posts"
```

To ignore drafts, uncomment the line in `.gitignore`:
```
# _drafts/  → _drafts/
```

### What happens to tags when I publish?

When you publish a draft:
1. The tag index is updated with the new file path (`_drafts/` → `_posts/`)
2. Tag files maintain their backlinks
3. You may want to run `blog process` again to update tag file links

### Can I rename tags?

Currently, there's no automatic tag rename command. To rename a tag:

1. Manually find and replace `[[old-tag]]` with `[[new-tag]]` in all posts
2. Delete the old tag file from `_drafts/tags/`
3. Run `script/blog process` to recreate the tag index

### Can I use this with a different static site generator?

The tool is designed for Jekyll but could be adapted. You'd need to:
- Modify front matter template in `blog-lib`
- Update directory structure in config
- Adjust publishing workflow

### Where is the tag index stored?

The tag index is stored in `.blog/tag-index.json`. This file is:
- Automatically generated
- Ignored by git (see `.gitignore`)
- Rebuilt when you run `blog process`

### Can I use spaces in tags?

Yes! Tags can contain spaces:
```markdown
[[machine learning]]  ✓
[[deep-learning]]     ✓
[[AI]]                ✓
```

The filename will sanitize to: `machine-learning.md`, `deep-learning.md`, `ai.md`

### What if I delete a tag file?

Run `script/blog process` to recreate it automatically.

### Can I customize the front matter template?

Yes! Edit the `generate_front_matter()` function in `script/blog-lib`.

### How do I preview my drafts locally?

Use Jekyll's draft preview mode:

```bash
# Start local server with drafts
bundle exec jekyll serve --drafts

# View at http://localhost:4000
```

Or use the existing script:

```bash
script/server  # (may need --drafts flag added)
```

## Advanced Usage

### Custom Commit Messages

If `BLOG_AUTO_COMMIT=false`, you can craft custom commit messages:

```bash
git add _posts/2026-01-06-my-post.md
git commit -m "Add post about machine learning

This post covers:
- Neural network basics
- Training techniques
- Real-world applications"
git push origin main
```

### Bulk Operations

Process multiple specific files:

```bash
for file in _drafts/2026-01-06-*.md; do
  script/blog process "$file"
done
```

### Integration with Scripts

Use blog CLI in shell scripts:

```bash
#!/bin/bash
# Daily blog reminder

drafts=$(script/blog list --drafts | tail -n +3 | wc -l)
echo "You have $drafts draft posts"

if [ "$drafts" -gt 5 ]; then
  echo "Time to publish something!"
fi
```

## Support

For issues or feature requests:
1. Check this documentation
2. Review the [troubleshooting section](#troubleshooting)
3. Open an issue on GitHub: `https://github.com/rbownes/rbownes.github.io/issues`

## Version History

- **v1.0.0** (2026-01-06) - Initial release
  - Draft creation and management
  - Tag extraction and indexing
  - Publishing workflow with git integration
  - Status and listing commands
