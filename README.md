# Diffly

A lightweight, zero-dependency text comparison tool that runs entirely in your browser. No server, no network calls — just open the HTML file and start comparing.

## Features

- **Diff modes**: ASCII (strip non-ASCII) or full Unicode
- **Granularity**: Line, word, or character-level comparison
- **View modes**: Side-by-side or unified diff
- **Inline mirror overlay**: See diff highlights directly in the text input areas
- **File support**: Drag & drop or file picker (with 1 MB warning)
- **Diff navigation**: Jump between changes with prev/next buttons and counter
- **Collapsible blocks**: Auto-collapse identical runs of 4+ lines, with Collapse All / Expand All
- **Fullscreen mode**: Focus on the diff output
- **Auto-compare**: Debounced real-time comparison as you type
- **Keyboard shortcuts**: `Esc` exits fullscreen, `Alt+C` collapses all, `Alt+E` expands all
- **Responsive**: Works on mobile with stacked layout

## Usage

1. Open `diffly_app.html` in any modern browser
2. Paste text into panels A and B, or drag & drop files
3. Adjust mode, granularity, and view as needed
4. Click **Compare** or let auto-compare run as you type

## Requirements

- Any modern browser (Chrome, Firefox, Safari, Edge)
- No build step, no dependencies, no internet connection required

## License

See [LICENSE](LICENSE)
