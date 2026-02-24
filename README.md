# Lorebook Studio

A visual node-graph lorebook editor for [SillyTavern](https://github.com/SillyTavern/SillyTavern). See your lorebook entries as interactive card nodes, visualize recursion links between them, and edit everything from a single graph workspace.

Built for botmakers who think visually.

## Features

### Visual Graph View
- Lorebook entries rendered as **card-style nodes** showing name, keywords, and status
- **Automatic recursion detection** -- scans entry content for keywords that trigger other entries and draws directed connection lines
- **Manual links** -- create your own custom connections between entries for organizational purposes
- **Color-coded nodes** -- blue (enabled), gray (disabled), yellow (selected), dashed red border (orphan/no connections), green border (constant)
- Curved bezier edges with labels showing the triggering keyword
- Auto-detected links shown as solid blue lines; manual links as dashed pink lines

### Full Editing Suite
- Click any node to open the **editing sidebar** with all World Info fields: name, keys, secondary keys, content, position, depth, order, probability, group, and all toggles
- **Create** new entries directly from the graph view
- **Duplicate** entries with one click
- **Delete** entries with confirmation
- **Manual link creation** via "Connect to..." dropdown in the sidebar

### Search and Filter
- Real-time **full-text search** across entry names, keys, and content
- Non-matching nodes fade out while matches stay highlighted
- Quick filter buttons: **Orphans** (entries with no connections), **Disabled** entries

### Statistics Dashboard
- Total entry count with enabled/disabled/constant breakdown
- Connection counts (auto-detected vs manual, primary vs secondary key links)
- **Most connected entries** (top 5) -- click to navigate
- **Orphan entries** list -- click to navigate
- **Health checks**: empty content warnings, missing keys, duplicate key detection

### Layout Options
- **Force-Directed** (CoSE) -- default, organic clustering
- **Grid** -- ordered rows and columns
- **Circle** -- entries arranged in a ring
- **Hierarchical** -- directed tree layout
- Zoom in/out and fit-to-screen controls
- **Node positions are saved** per lorebook and persist across sessions

### Seamless Integration
- Full-page **drawer UI** slides in from the right (~95% viewport width)
- One-click launch from SillyTavern's World Info panel
- Close with ESC key, backdrop click, or the close button
- Auto-refreshes when lorebook data changes in SillyTavern
- All settings saved through SillyTavern's extension settings system

## Installation

1. Open SillyTavern
2. Go to **Extensions** (puzzle piece icon)
3. Click **Install Extension**
4. Paste this URL:
   ```
   https://github.com/hype-hosting/SillyTavern-Lorebook-Utility
   ```
5. Click Install and reload SillyTavern

The extension comes pre-built -- no additional setup required.

## Usage

1. Open any lorebook in SillyTavern's **World Info** panel
2. Click the **Lorebook Studio** button that appears in the panel header
3. The graph view opens as a full-page drawer showing all your entries as nodes
4. **Select a lorebook** from the dropdown if not auto-detected
5. **Click a node** to open the editing sidebar
6. **Drag nodes** to rearrange -- positions are saved automatically
7. Use the **toolbar** to search, filter, switch layouts, or toggle link visibility
8. Click **Stats** to see connection analysis and health checks

## Building from Source

If you want to modify the extension or contribute:

```bash
# Clone the repo
git clone https://github.com/hype-hosting/SillyTavern-Lorebook-Utility.git
cd SillyTavern-Lorebook-Utility

# Install dependencies
npm install

# Build for production
npm run build

# Build for development (unminified, with source maps)
npm run build:dev

# Lint
npm run lint
```

The compiled output goes to `dist/index.js`.

### Tech Stack

- **TypeScript** + **Webpack** build system
- **Cytoscape.js** for graph rendering
- SillyTavern extension API for data access and persistence

## Project Structure

```
src/
  index.ts                 # Extension entry point
  style.css                # UI stylesheet (dark theme)
  templates/drawer.html    # Drawer HTML template
  data/
    lorebookData.ts        # SillyTavern World Info read/write
    recursionDetector.ts   # Key-in-content scanning algorithm
    manualLinks.ts         # Manual link storage and CRUD
  graph/
    graphManager.ts        # Cytoscape.js instance lifecycle
    nodeStyles.ts          # Card-style node definitions
    edgeStyles.ts          # Edge styling (auto vs manual)
    layouts.ts             # Layout algorithm configurations
  ui/
    drawer.ts              # Full-page drawer component
    toolbar.ts             # Search, filters, layout controls
    sidebar.ts             # Entry editing panel
    statsPanel.ts          # Statistics dashboard
  features/
    entryEditor.ts         # Entry validation utilities
    entryCrud.ts           # Create/duplicate/delete operations
    search.ts              # Search and filter logic
  utils/
    events.ts              # Internal event bus + ST events
    settings.ts            # Extension settings persistence
```

## Support

If you find Lorebook Studio useful, consider supporting development:

- [Support on Ko-fi](https://ko-fi.com/hype)
- [Join the Discord](https://discord.gg/therealhype)

## License

[GPL-3.0](LICENSE)
