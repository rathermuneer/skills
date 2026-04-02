# Drupal Image Styles Plugin

A Claude Code plugin that extracts image styles from frontend design prototypes and generates Drupal image style configuration YAML — ready to import.

## What It Does

1. Crawls a design/prototype URL (Figma, Netlify, Vercel, etc.)
2. Extracts image dimensions, aspect ratios, responsive variants
3. Deduplicates and presents a consolidated table for approval
4. Checks for conflicts with existing project image styles
5. Generates Drupal config YAML files in your config sync directory
6. Runs config import/export to finalize

## Installation

### From Marketplace

If your team has the Axelerant marketplace configured:

```
/plugin install drupal-image-styles@axelerant-plugins
```

### Manual (Development)

```bash
claude --plugin-dir /path/to/claude-drupal-image-styles
```

## Usage

```
/drupal-image-styles:image-style-extractor https://your-design-url.com
```

The skill will:
- Auto-detect your Drupal project setup (drush prefix, docroot, config sync dir)
- Crawl all pages on the design URL
- Present extracted styles for your approval before generating anything
- Write config YAML to your sync directory
- Run the config import/export cycle

## Project Compatibility

Works with any Drupal 10/11 project. Auto-detects:
- `drupal-claude.yml` configuration (drupal-sdlc plugin)
- DDEV environments
- Standard Composer-based Drupal projects
- Custom docroot paths (`web/`, `app/`, `docroot/`)

## Requirements

- Claude Code CLI
- A Drupal project with config sync enabled
- WebFetch tool access (for crawling design URLs)

## License

MIT
