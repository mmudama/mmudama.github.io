
## Common Hugo Commands

### Development

```bash
# Start the Hugo development server with draft content
hugo server -D

# Start the Hugo development server (published content only)
hugo server

# Start server and automatically open in browser
hugo server --open
```

### Building the Site

```bash
# Build the site for production (output to public/)
hugo

# Build with clean destination directory
hugo --cleanDestinationDir

# Build including draft content
hugo -D
```

### Creating Content

```bash
# Create a new post
hugo new content/posts/2026/my-new-post.md

# Create a new post in a specific year folder
hugo new content/posts/$(date +%Y)/post-name.md

# Create a new tech page
hugo new content/tech/page-name.md
```

### Other Useful Commands

```bash
# Check Hugo version
hugo version

# List all content
hugo list all

# List draft content
hugo list drafts

# List future-dated content
hugo list future

# Display configuration
hugo config
```

## Project Structure

- `content/` - Markdown content files
  - `posts/` - Blog posts organized by year
  - `tech/` - Technical documentation pages
- `public/` - Generated static site (git ignored)
- `themes/hugo-coder/` - Hugo Coder theme
- `hugo.toml` - Site configuration

## Quick Start

1. Install Hugo (extended version recommended)
2. Clone this repository
3. Run `hugo server -D` to start development server
4. Visit http://localhost:1313
5. Edit content in `content/` directory
6. Build with `hugo` when ready to deploy