---
name: image-style-extractor
description: Extract image styles from a design URL and generate Drupal image style config YAML. Use when given a frontend design/prototype URL and asked to extract image styles, dimensions, or aspect ratios for Drupal.
argument-hint: [design-url]
allowed-tools: WebFetch, Read, Write, Edit, Bash, Glob, Grep
---

# Image Style Extractor

You are a Drupal image style architect. Given a design URL, you will crawl the design pages, extract every image style specification, and generate ready-to-import Drupal config YAML.

## Project Context Detection

Detect the project's Drupal configuration automatically:

- **Drush prefix:** Check for `drupal-claude.yml` first. If it exists, read `drush_prefix` from it. Otherwise default to `drush` (or `ddev drush` if `.ddev/` directory exists).
- **Drupal version:** Check `drupal-claude.yml` for `drupal_version`. Otherwise read from `composer.json` (`drupal/core` version constraint).
- **Config sync directory:** Check `settings.php` for `$settings['config_sync_directory']`. Common locations: `sites/default/files/sync/`, `config/sync/`, `../config/sync/`.
- **Docroot:** Detect from `composer.json` `extra.drupal-scaffold.locations.web-root` or check for `web/` or `app/` directories.

## Input

The user provides a design URL: **$ARGUMENTS**

If no URL is provided, ask the user for the design/prototype URL before proceeding.

## Workflow

### Step 1: Discover Pages

Fetch the provided URL and extract all internal links/routes. Build a list of unique pages to crawl.

### Step 2: Extract Image Styles from Every Page

For each discovered page (and the homepage), fetch and extract:

- **Image dimensions** (width x height in px)
- **Aspect ratios** (e.g., 16:9, 3:4, 1:1)
- **CSS object-fit values** (cover, contain, etc.)
- **Border radius / shape** (circle, rounded, pill, etc.)
- **Responsive variants** (mobile vs desktop dimensions)
- **Context** where each image appears (hero, card, thumbnail, logo, icon, avatar, etc.)

### Step 3: Deduplicate and Consolidate

Merge identical styles found across pages. Create a consolidated table:

| Style Name | Aspect Ratio | Dimensions | Effect | Usage Context |
|-----------|-------------|-----------|--------|--------------|
| ... | ... | ... | Scale & Crop / Scale / Crop | Where it's used |

Present this table to the user and ask: **"Does this look correct? Should I add, remove, or adjust any styles before generating config?"**

**STOP here and wait for user confirmation.**

### Step 4: Check Existing Image Styles

After user approval, check what image styles already exist in the project:

```bash
$DRUSH_PREFIX config:list | grep image.style
```

Also check the config sync directory for existing image style files:

```bash
find $CONFIG_SYNC_DIR -name "image.style.*.yml" 2>/dev/null | sort
```

Report any overlaps to the user before proceeding.

### Step 5: Generate Drupal Image Style Config YAML

For each approved style, generate a config YAML file following this structure:

```yaml
langcode: en
status: true
dependencies: {}
name: MACHINE_NAME
label: 'Human Label'
effects:
  EFFECT_UUID:
    uuid: EFFECT_UUID
    id: EFFECT_ID
    weight: 1
    data:
      width: WIDTH
      height: HEIGHT
      anchor: center-center
      upscale: false
```

#### Effect Type Reference

| Desired Effect | Drupal Effect ID | Required Data |
|---------------|-----------------|---------------|
| Scale & Crop | `image_scale_and_crop` | width, height, anchor |
| Scale (maintain ratio) | `image_scale` | width OR height, upscale |
| Crop | `image_crop` | width, height, anchor |
| Resize (exact) | `image_resize` | width, height |

#### Naming Conventions

- Machine name: lowercase, underscores, descriptive (e.g., `card_16_9`, `hero_banner`, `avatar_square`)
- Label: Human-readable with dimensions (e.g., "Card 16:9 (768x432)")

### Step 6: Write Config Files

Write each image style config to the project's config sync directory:

```
$CONFIG_SYNC_DIR/image.style.MACHINE_NAME.yml
```

**IMPORTANT RULES:**
- NEVER write UUID values — leave them out; `config:import` + `config:export` will generate them
- Do NOT include `uuid:` in the top-level config or in effect entries
- Use a placeholder for effect UUIDs — they'll be regenerated on import
- Check existing styles for naming conventions (e.g., WebP conversion) and follow them

Write the files WITHOUT uuid fields:

```yaml
langcode: en
status: true
dependencies: {}
name: MACHINE_NAME
label: 'Human Label'
effects: {}
```

Then instruct the user to run the config import/export cycle to get proper UUIDs and effect entries.

### Step 7: Import and Export Config

Run the config management cycle:

```bash
$DRUSH_PREFIX config:import -y
$DRUSH_PREFIX config:export -y
$DRUSH_PREFIX config:status
```

Config status MUST show "No differences". If it does, the generated styles are ready.

### Step 8: Summary

Present a final summary:

1. Total image styles created
2. Table of all styles with machine names, labels, dimensions, and effects
3. Remind user to verify in admin UI: `/admin/config/media/image-styles`

## Important Rules

- NEVER hand-write UUIDs — always let config:export generate them
- ALWAYS run config:import then config:export after writing config files
- Prefer Scale & Crop for fixed-ratio images (cards, heroes)
- Prefer Scale for logos and icons where aspect ratio varies
- Include responsive variants as separate styles when mobile dimensions differ significantly
- Follow existing project naming conventions if image styles already exist
- Match existing effects chain (e.g., if project uses WebP conversion, include it)
