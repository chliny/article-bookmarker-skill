---
name: article-bookmarker
description: Collect and organize articles by fetching URLs, extracting content, generating AI summaries, auto-tagging topics, and maintaining a searchable bookmark collection with tag-based indexing. Use when the user wants to save articles from web links or text content for future reference and organization.
---

# Article Bookmarker Skill

## Quick Start

When the user provides a URL or article text to bookmark:

1. Use `web_fetch` to get the article content
2. Generate a concise summary using the current model
3. Auto-generate relevant tags based on content analysis
4. Create a markdown file with URL, content, summary, and tags
5. Save to the bookmark directory (configured via `$ARTICLE_BOOKMARK_DIR` environment variable) with descriptive filename
6. Update the tag index file

For deletion requests: find the article, confirm details with user, then remove and update index.

## Workflow

### Adding Articles

```
1. Receive URL or text content
2. Extract/save content (web_fetch for URLs)
3. Generate summary (model-based)
4. Auto-tag (keyword/topic analysis)
5. Create bookmark file (markdown format)
6. Update tag index
```

### Deleting Articles

```
1. Identify target article (by filename, topic, or content)
2. Display article details for confirmation
3. Get user confirmation
4. Delete bookmark file
5. Update tag index
```

## File Structure

Bookmarks are stored as individual markdown files in the directory specified by the `$ARTICLE_BOOKMARK_DIR` environment variable:

```
bookmarks/
├── article-title-slug.md (individual articles)
├── TAG_INDEX.md (tag to article mapping)
└── README.md (directory overview)
```

### Bookmark File Format

Each bookmark file contains:

```markdown
# Article Title

**Source:** URL  
**Bookmarked:** YYYY-MM-DD HH:MM GMT+8  
**Tags:** tag1, tag2, tag3

## Summary

AI-generated concise summary of the article...

## Content

Full extracted article content...

## Original URL

[Link](URL)
```

## Tag Management

### Auto-Tagging Logic

Generate tags by analyzing:
- Article domain/topic keywords
- Technical terms and concepts
- Content categories (tutorial, news, research, etc.)
- Named entities and proper nouns

Maintain consistent tag vocabulary to avoid duplicates (e.g., use "AI" not "artificial-intelligence").

### Tag Index Format

TAG_INDEX.md maintains bidirectional mapping:

```markdown
# Article Tag Index

## Tags

- **AI**: [article1](article1.md), [article2](article2.md)
- **Skills**: [skill-creation](skill-creation.md), [evaluation](evaluating-skill-output-quality.md)
- **Research**: [...]

## Articles by Tag Count

- 3 tags: [article1](article1.md)
- 2 tags: [article2](article2.md), [article3](article3.md)
- 1 tag: [...]
```

## Implementation Details

### Content Extraction

- Use `web_fetch` with `extractMode: "markdown"` for web articles
- Handle truncation gracefully (respect `maxChars` limits)
- Preserve original formatting where possible
- **GitHub Repository URLs**: When the URL is a GitHub repository (e.g., `https://github.com/user/repo`), prioritize fetching the README content from the repository's main page or from `README.md`, `readme.md`, or `README.rst` files in the root directory

### Proxy Configuration and Retry

When fetching article content from URLs fails:

1. **First Attempt**: Try fetching without proxy
2. **On Failure**: Load proxy configuration from environment variables:
   - `HTTP_PROXY` or `http_proxy`: HTTP proxy URL
   - `HTTPS_PROXY` or `https_proxy`: HTTPS proxy URL
   - `NO_PROXY` or `no_proxy`: Comma-separated list of hosts to bypass
3. **Retry**: Re-attempt fetching with proxy configuration
4. **Final Failure**: Notify user if both attempts fail

Example environment variables:
```bash
export HTTP_PROXY="http://proxy.example.com:8080"
export HTTPS_PROXY="http://proxy.example.com:8080"
export NO_PROXY="localhost,127.0.0.1,.example.com"
```

### Summary Generation

Generate 2-3 paragraph summaries that capture:
- Main thesis or argument
- Key insights or findings  
- Practical implications or applications

Keep summaries informative but concise (typically 150-300 words).

### File Naming

Create SEO-friendly filenames:
- Convert title to lowercase
- Replace spaces and special chars with hyphens
- Limit length to ~50 characters
- Ensure uniqueness by appending numbers if needed

### Safety Checks

- Validate URLs before fetching
- Confirm deletions with users (show path and key details)
- Maintain backup of index before modifications
- Handle concurrent access gracefully
