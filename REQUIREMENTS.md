# Hacker News Comment Tree Visualizer - Requirements Document

## Project Overview

A single-file HTML tool (no frameworks like React) that fetches and visualizes Hacker News comment threads in a horizontal row-based tree layout. The tool allows users to navigate through nested comment hierarchies, search for content, and explore discussion patterns.

**Data Source:** [Algolia HN API](https://hn.algolia.com/api) - `https://hn.algolia.com/api/v1/items/{id}`
**Prior inspiration:** [Simon Willison's Hacker News thread export](https://tools.simonwillison.net/hacker-news-thread-export)

---

## Core Layout: Horizontal Row Navigation

The primary visualization is a **horizontal row-based tree** where:

1. **Each row represents a depth level** in the comment tree
2. **Siblings are displayed horizontally** within their row (scrollable if many)
3. **One comment per row is "selected"** (active) - its children populate the row below
4. **Clicking any sibling** selects it, rebuilding child rows beneath
5. **First child is auto-selected** by default at each level, creating a default "path" through the tree

### Visual States
- **Active (selected) comment:** Orange border (`#ff6600`) with subtle glow/shadow
- **Inactive siblings:** Slightly grayed out (~60% opacity) but still readable; brightens on hover (~85%)
- **Depth indicator:** Small label on each row (e.g., "Level 1", "Level 2")

---

## Comment Box Display

Each comment box should have:

### Header
- **Author name:** Links to HN profile (`https://news.ycombinator.com/user?id={author}`)
- **"link" text:** Links to original comment on HN (`https://news.ycombinator.com/item?id={id}`)
- **No timestamps**

### Body
- **Comment text:** HTML content rendered properly
- **Truncation:** Truncate at 300 characters with `[expand]`/`[collapse]` toggle
- **Fixed width:** ~320px per comment box

### Footer (for selected comment)
- **Subtree stats:** "↓ X replies" and "👤 Y users" (unique participants in subtree)
- **Leaf indicator:** Show "leaf" for comments with no children
- **Sibling navigation:** Prev/Next buttons with position indicator (e.g., "2/6")
  - Buttons gray out (disabled) at first/last sibling - no wrap-around

### Deleted Comments
- Display as grayed box with `[deleted]` author and `[comment deleted]` text
- **Do not count** toward any statistics (reply counts, user counts)

---

## Thread Header

When a thread is loaded, display:
- **Title:** Links to the story URL (or HN link if no URL)
- **Author:** Links to HN profile
- **"view on HN" link**
- **Stats:** Total comments and unique participants (e.g., "47 comments by 23 unique participants")

---

## Search Functionality

### Search Input
- Text input field with debounced search (300ms delay)

### Search Options (checkboxes)
- **Include authors:** Search author names too (default: OFF)
- **Case sensitive:** (default: OFF)

### Search Results
- **Match counter:** Display "X of Y" (e.g., "3 of 17")
- **Prev/Next buttons:** Navigate between matches; disable at first/last match
- **Yellow highlight:** `<mark>` tags with yellow background on matching text

### Search Navigation Behavior
When navigating to a search match:
1. Rebuild the selected path to include the matching comment
2. Expand/select ancestors so the match is visible
3. Apply yellow highlight to matches
4. Scroll the comment into view
5. Add visual focus indicator (e.g., yellow border glow)

---

## Sorting Option

Checkbox toggle: **"Sort by most discussed (participants)"**

- **ON (default):** Sort siblings by unique participant count in subtree (descending - busiest threads first)
- **OFF:** Use original API order (chronological)

When toggled, re-initialize the selected path and re-render.

---

## Keyboard Navigation

### Row Focus
- **Up/Down arrows:** Move focus between depth rows
- Focused row gets a visual highlight (10% darker background)
- Focused row scrolls into view if needed

### Selection Change
- **Left/Right arrows:** Change the selected comment within the focused row
- This traverses siblings at that depth level
- Stops at first/last sibling (no wrap)

### Text Expansion
- **Enter key:** Toggle expand/collapse of the selected comment's text (if truncated)

### Hint
Display keyboard hint in the toolbar: "↑↓ rows • ←→ selection • Enter expand"

### Input Focus
Keyboard navigation is disabled when focus is in a text input (search field, URL input).

---

## Layout

### No Horizontal Scrolling
- Rows use `flex-wrap: wrap` instead of horizontal scroll
- Comments wrap to next line if too many to fit
- Selection changes only via buttons or arrow keys (no scrolling required)

---

## Sticky Search Bar

The search/options toolbar should:
- Use `position: sticky` at `top: 0`
- When "stuck" (user has scrolled past it):
  - Add drop shadow for visual separation
  - Optionally expand to full viewport width

---

## Input Handling

Accept either:
- **Raw HN ID:** e.g., `39581733`
- **Full HN URL:** e.g., `https://news.ycombinator.com/item?id=39581733`

Extract ID using regex: `/id=(\d+)/` or validate as numeric string.

---

## Debug/Mock Mode

For development and testing when API is unavailable:

### Activation
- URL parameter: `?debug=mock`

### Behavior
- Display a **warning banner** at top: "🔧 Debug Mode: Using mock data. Any ID/URL will return the same test thread. [Disable]"
- For any fetch request, return a static mock payload instead of calling the API
- Add small simulated delay (e.g., 300ms) for realistic feel

### Mock Data Structure
Include in a **separate, clearly-marked `<script>` tag** at the end of the file for easy removal:
```html
<!-- MOCK DATA FOR DEBUG MODE - Remove this entire script tag in production -->
<script>
  const MOCK_HN_RESPONSE = { ... };
</script>
<!-- END MOCK DATA -->
```

Mock payload should include test scenarios:
- Multiple top-level comments (6+)
- Deep nesting (4-5 levels)
- At least one deleted comment with replies
- At least one very long comment (600+ chars) for truncation testing
- Back-and-forth conversations between same users
- Recognizable fake usernames for easy identification

---

## Styling & Theme

- **Color scheme:** HN orange (`#ff6600`) and beige (`#f6f6ef`) theme
- **Font:** Verdana or similar sans-serif, 13px base
- **Simple, clean aesthetic** - no heavy gradients or effects
- **Desktop-only** - no mobile responsiveness required

---

## Technical Constraints

- **Single HTML file** with inline CSS and JavaScript
- **No frameworks** (React, Vue, etc.) - vanilla JS only
- **CDN libraries allowed** if needed (e.g., for specific utilities)
- **No build step** - file should work when opened directly in browser

---

## Out of Scope (Explicitly Excluded)

- Timestamps on comments
- Deep linking / URL state persistence
- Mobile responsive design
- Performance optimization for very large threads (1000+ comments)
- Accessibility features (ARIA, screen reader support)
- Global collapse/expand all buttons
- Support for other platforms (Reddit, Twitter, etc.)
- Horizontal scrolling of comment rows

---

## API Reference

### Algolia HN Items Endpoint
```
GET https://hn.algolia.com/api/v1/items/{id}
```

### Response Structure (relevant fields)
```json
{
  "id": 12345678,
  "title": "Story title",
  "url": "https://example.com",
  "author": "username",
  "type": "story",
  "children": [
    {
      "id": 12345679,
      "author": "commenter",
      "text": "<p>Comment HTML content</p>",
      "children": [ ... ]
    },
    {
      "id": 12345680,
      "author": null,
      "text": null,
      "deleted": true,
      "children": [ ... ]
    }
  ]
}
```

- Deleted comments have `author: null` and/or `deleted: true`
- Comment text contains raw HTML (paragraph tags, links, etc.)
- Children array is nested recursively

---

## Summary Checklist

- [ ] Single HTML file, vanilla JS, no build step
- [ ] Fetch from Algolia API (with mock mode fallback)
- [ ] Horizontal row layout with sibling comments (flex-wrap, no scroll)
- [ ] Active comment has orange border; inactive siblings are grayed
- [ ] Auto-select first child to create default path through tree
- [ ] Click sibling to select it and rebuild child rows
- [ ] Subtree stats: reply count + unique user count
- [ ] Sibling navigation buttons (Prev/Next) with edge detection
- [ ] Keyboard navigation: ↑↓ for rows, ←→ for siblings, Enter to expand/collapse
- [ ] Row focus highlight (10% darker background)
- [ ] Full-text search with match counter and Prev/Next navigation
- [ ] Search options: include authors, case sensitivity
- [ ] Yellow highlight on search matches
- [ ] Sort toggle: most-discussed-first (default ON) vs. API order
- [ ] Sticky search bar
- [ ] Comment text truncation at 300 chars with expand toggle
- [ ] Author links to HN profile, comment links to HN
- [ ] Deleted comments displayed but excluded from stats
- [ ] Debug mock mode via URL parameter
- [ ] HN orange/beige theme

