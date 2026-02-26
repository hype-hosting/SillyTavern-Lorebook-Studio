# Lorebook Studio - Organizational Suite Implementation Plan

## Overview

Replace the "Manual Links" section in the sidebar with a full **Studio Metadata** panel. Add Categories, Tags, Studio Notes, Status/Workflow, Pin/Bookmark, and Color Override. All metadata is stored in extension settings (not in ST's lorebook data). Node colors on the 3D graph are driven by category/status/pin instead of just enabled/disabled state.

Manual links stay functional but move into the right-click context menu (already has "Connect To...") and the toolbar Link button, removing the sidebar dropdown.

---

## Phase 1: Data Layer

### 1a. Extend settings (`src/utils/settings.ts`)

Add new interfaces and settings fields:

```typescript
interface CategoryDef {
  id: string;       // e.g. "cat-abc123"
  name: string;     // e.g. "Characters"
  color: string;    // hex e.g. "#e06c75"
}

type EntryStatus = 'draft' | 'in-progress' | 'review' | 'complete';

interface EntryMeta {
  categoryId: string | null;
  tags: string[];
  notes: string;
  status: EntryStatus | null;
  pinned: boolean;
  colorOverride: string | null;
}

interface StudioData {
  categories: CategoryDef[];
  entryMeta: Record<string, EntryMeta>;  // keyed by entry UID string
}
```

Add to `LorebookStudioSettings`:
```typescript
studioData: Record<string, StudioData>;  // keyed by book name
```

### 1b. Create studio data module (`src/data/studioData.ts`)

CRUD helpers for categories, entry metadata, tags:
- `getStudioData(bookName): StudioData`
- `getEntryMeta(bookName, uid): EntryMeta`
- `updateEntryMeta(bookName, uid, partial): void`
- `getCategories(bookName): CategoryDef[]`
- `addCategory(bookName, name, color): CategoryDef`
- `updateCategory(bookName, id, partial): void`
- `deleteCategory(bookName, id): void`
- `getAllTags(bookName): string[]` — collects unique tags across all entries for autocomplete
- `cleanupEntryMeta(bookName, uid): void` — called on entry deletion

---

## Phase 2: Sidebar Redesign

### 2a. Update drawer HTML (`src/templates/drawer.html`)

Replace the manual links section with a "Studio" section containing:

1. **Category** — `<select>` dropdown populated with user categories + "Manage..." option
2. **Tags** — Input field + tag pills container
3. **Status** — `<select>` with Draft/In Progress/Review/Complete/None
4. **Pin** — Checkbox toggle (star icon)
5. **Notes** — `<textarea>` for studio notes
6. **Color Override** — Small color input (hidden by default, expandable)

### 2b. Category Manager modal

A small floating panel (like the stats panel) for managing categories:
- List of existing categories with color swatch + name + edit/delete buttons
- "Add Category" row at bottom with name input + color picker + add button
- Editable inline (click name to rename, click swatch to recolor)

### 2c. Update sidebar logic (`src/ui/sidebar.ts`)

- Remove: `populateManualLinks()`, `populateLinkTargets()`, `addLink()`, manual link event handlers
- Add: `populateStudioSection()` — loads entry meta into the studio fields
- Add: `saveStudioMeta()` — reads studio fields and calls `updateEntryMeta()`
- Add: Tag input handler — Enter to add tag, click X to remove, autocomplete dropdown from `getAllTags()`
- Integrate studio saves into the existing `saveEntry()` flow
- Move manual link creation to context menu only (already exists there via "Connect To...")

---

## Phase 3: Node Styling

### 3a. Extend `GraphNode` interface (`src/graph/nodeStyles.ts`)

Add fields:
```typescript
categoryColor: string | null;
colorOverride: string | null;
status: EntryStatus | null;
pinned: boolean;
hasTags: boolean;
hasNotes: boolean;
```

### 3b. Update `buildNode()` in `graphManager.ts`

Look up entry meta from `getEntryMeta()` and populate the new GraphNode fields.

### 3c. Update color cascade in `createCardSprite()` and `createLabelSprite()`

New priority order:
1. Connect source → special border
2. Selected → yellow
3. Highlighted (search match) → purple
4. **Color override** → use as border color if set
5. **Category color** → use as border color if set
6. Disabled → gray (keep existing)
7. Default → neutral purple

Status indicator: Add a small text prefix or suffix to the label (e.g., a colored dot character).

Pinned indicator: Add a star character (★) prefix to the label.

Notes indicator: Add a small dot (●) after the name if `hasNotes`.

### 3d. Constant/Orphan styling

These become **secondary indicators** — still visually distinct but via a subtle text badge or border style rather than overriding the category color entirely.

---

## Phase 4: Toolbar Enhancements

### 4a. Add filter controls (`src/templates/drawer.html` + `src/ui/toolbar.ts`)

Add to the toolbar center section (alongside search):
- **Category filter** — `<select>` dropdown: "All Categories" + list of categories + "Uncategorized"
- **Status filter** — `<select>` dropdown: "All Statuses" + Draft/In Progress/Review/Complete
- **Pinned filter** — Button toggle (like Orphans/Disabled)

### 4b. Extend search (`src/ui/toolbar.ts`)

Add tags and studio notes to the searchable text:
```typescript
const searchableText = [
  entry.comment, entry.key.join(' '), entry.keysecondary.join(' '),
  entry.content,
  meta.tags.join(' '),  // NEW
  meta.notes,           // NEW
].join(' ').toLowerCase();
```

### 4c. Filter logic

Category and status filters work like Orphans/Disabled — they call `applySearchHighlight()` with matching IDs. When multiple filters are active, they intersect (AND logic).

---

## Phase 5: Stats Panel Enhancement

### 5a. Add new stat sections (`src/ui/statsPanel.ts`)

- **Category breakdown** — entries per category, with color swatches. Clickable.
- **Status breakdown** — entries per status. Clickable.
- **Tag cloud** — most-used tags with counts.
- **Pinned entries** — count + clickable list.
- **Entries with notes** — count.

---

## Phase 6: Cleanup & Integration

### 6a. Remove manual links from sidebar

- Remove the manual links HTML section from `drawer.html`
- Remove `populateManualLinks`, `populateLinkTargets`, `addLink` from `sidebar.ts`
- Keep the imports of `manualLinks.ts` in other files (connect mode, graph manager) — manual links still work through the toolbar Link button and context menu

### 6b. Entry deletion cleanup

In `entryCrud.ts` `deleteEntryById()`, add call to `cleanupEntryMeta()` alongside existing `removeLinksForEntry()`.

### 6c. Events

Add new events to `STUDIO_EVENTS`:
- `STUDIO_META_UPDATED` — emitted when entry meta changes (triggers graph refresh for color update)
- `CATEGORIES_CHANGED` — emitted when categories are added/removed/edited (triggers toolbar dropdown refresh + graph recolor)

### 6d. CSS (`src/style.css`)

- Tag pill styles (small rounded pills with X button)
- Category manager panel styles
- Category color swatch in sidebar dropdown
- Status badge colors
- Pin star styling
- Color picker input styling
- Updated toolbar filter dropdown styles

---

## File Changes Summary

| File | Action |
|------|--------|
| `src/utils/settings.ts` | Add interfaces, extend settings |
| `src/data/studioData.ts` | **NEW** — studio metadata CRUD |
| `src/templates/drawer.html` | Replace manual links with studio section, add category manager, add toolbar filters |
| `src/ui/sidebar.ts` | Replace manual links logic with studio metadata logic |
| `src/ui/categoryManager.ts` | **NEW** — category CRUD panel |
| `src/ui/toolbar.ts` | Add category/status/pin filters, extend search |
| `src/ui/statsPanel.ts` | Add category/status/tag stats |
| `src/graph/nodeStyles.ts` | Extend GraphNode, update color cascade |
| `src/graph/graphManager.ts` | Pass studio meta into buildNode |
| `src/features/entryCrud.ts` | Add meta cleanup on delete |
| `src/utils/events.ts` | Add new events |
| `src/style.css` | Tag pills, category manager, status badges, filter styles |
| `src/index.ts` | Init new default settings |

---

## Implementation Order

1. **Data layer first** — settings + studioData.ts (everything else depends on this)
2. **Node styling** — get category colors rendering on nodes (most visible impact)
3. **Sidebar redesign** — studio metadata panel replacing manual links
4. **Category manager** — the modal for creating/editing categories
5. **Toolbar filters** — category/status/pin filtering
6. **Stats panel** — enhanced analytics
7. **Search extension** — tags + notes in search
8. **Cleanup** — remove dead manual links sidebar code, final polish
