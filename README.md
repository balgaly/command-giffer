# command-giffer

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](./LICENSE)
[![Claude Code](https://img.shields.io/badge/Built%20with-Claude%20Code-blueviolet)](https://docs.anthropic.com/en/docs/claude-code)

Create animated terminal demo GIFs for GitHub READMEs with a single command. Describe what the terminal should show in natural language and get a polished GIF.

![Demo](./demo.gif)

## Installation

```bash
# Clone into your Claude Code skills directory
git clone https://github.com/balgaly/command-giffer.git ~/.claude/skills/command-giffer
```

Or add to a project's `.claude/skills/` directory for project-scoped use.

## Usage

```bash
# Basic usage — describe what you want
/command-giffer "Show pytest running 5 tests, all passing"

# Specify output path
/command-giffer "Show npm install adding 3 packages" --output assets/install-demo.gif

# Custom dimensions
/command-giffer "Show git status with staged and unstaged files" --width 900 --height 500

# Custom window title
/command-giffer "Show cargo build compiling 4 crates" --title "cargo"
```

## Options

| Option | Default | Description |
|--------|---------|-------------|
| `--output` | `demo.gif` | Output file path for the GIF |
| `--width` | `820` | Terminal viewport width in pixels |
| `--height` | `400` | Terminal viewport height in pixels |
| `--title` | Auto-derived | macOS-style window title bar text |

## How It Works

1. **Design** — Claude interprets your natural language description and designs a convincing terminal animation with realistic output, colors, and timing
2. **Generate HTML** — Creates a self-contained HTML file with a terminal UI, CSS animations, and a JavaScript animation engine
3. **Record** — Launches a headless browser with Playwright at 2x resolution and records the animation as video
4. **Convert** — Uses ffmpeg's two-pass palette method to create an optimized, high-quality GIF
5. **Clean up** — Removes all temporary files, leaving only the final GIF

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- [Playwright MCP](https://github.com/anthropics/mcp-playwright) server configured
- [ffmpeg](https://ffmpeg.org/) installed and in PATH
- Python 3 (for the local HTTP server)

## Playwright MCP Setup

Add the Playwright MCP server to your Claude Code config (`.mcp.json` in your project root or `~/.claude/mcp.json` globally):

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@anthropic-ai/mcp-playwright@latest"]
    }
  }
}
```

## Examples

### pytest with passing tests

```bash
/command-giffer "Show pytest running 3 tests, all passing with 0.42s total time"
```

### npm install

```bash
/command-giffer "Show npm install adding 127 packages in 4 seconds with no vulnerabilities"
```

### git status with changes

```bash
/command-giffer "Show git status on main branch with 2 staged files and 1 untracked file"
```

### Failed build

```bash
/command-giffer "Show cargo build failing with a type mismatch error on line 42 of main.rs"
```

## Troubleshooting

**"ffmpeg not found"**
Install ffmpeg: `brew install ffmpeg` (macOS), `choco install ffmpeg` (Windows), or `sudo apt install ffmpeg` (Linux).

**"Browser not installed"**
The skill will offer to install the Playwright browser automatically. If that fails, run `npx playwright install chromium`.

**GIF has scrollbar artifacts**
The HTML template uses `overflow: hidden` on html/body to prevent this. If you see scrollbars, try increasing `--height`.

**GIF is too large (>5MB)**
The skill automatically retries with reduced colors and frame rate. You can also reduce `--width`/`--height` or simplify the animation description.

**Recording hangs**
The animation signals completion via `document.title = 'ANIMATION_DONE'`. If it never fires, there may be a JavaScript error in the generated HTML. Check the browser console.

## License

[MIT](./LICENSE) - Snir Balgaly
