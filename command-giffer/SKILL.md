---
name: command-giffer
description: >
  Create animated terminal demo GIFs for GitHub READMEs.
  Describe what the terminal should show in natural language and get a polished GIF.
  Use when user says "create demo gif", "terminal animation", "readme gif",
  "record terminal demo", "make a gif for my readme", or invokes /command-giffer.
license: MIT
allowed-tools: Bash, Read, Write, Glob, Grep, mcp__playwright__browser_run_code, mcp__playwright__browser_navigate, mcp__playwright__browser_snapshot, mcp__playwright__browser_close, mcp__playwright__browser_wait_for, mcp__playwright__browser_tabs, mcp__playwright__browser_install
argument-hint: "<description of terminal demo>" [--output path/to/output.gif] [--width 820] [--height 400] [--title "Window Title"]
metadata:
  author: Snir Balgaly
  version: "1.0.0"
  tags: gif, terminal, demo, readme, animation
---

# command-giffer

Generate animated terminal demo GIFs from natural language descriptions.

## Step 1: Parse Input

Parse the user's arguments to extract:

- **description**: The natural language description of what the terminal should show (required, the main argument text)
- **--output**: Output GIF path (default: `demo.gif` in the current working directory)
- **--width**: Terminal viewport width in pixels (default: `820`)
- **--height**: Terminal viewport height in pixels (default: `400`)
- **--title**: Terminal window title (auto-derived from the command if not specified)

If the description is empty or missing, ask the user what they want the terminal to show.

## Step 2: Design the Animation

This is the creative step. Interpret the user's natural language description and design a convincing terminal animation. Decide:

1. **The command** — what appears after the `$` prompt (e.g., `pytest tests/ -v`, `npm install`, `git status`)
2. **The output lines** — each line has text content, CSS color classes, and a delay before appearing
3. **The window title** — shown in the macOS-style title bar (defaults to the tool name)

### Color Conventions

Use these CSS classes to color output lines convincingly:

| Class | Color | Use for |
|-------|-------|---------|
| `green` | `#7ee787` | Success, PASSED, checkmarks, "done" messages |
| `red` | `#f85149` | Failures, FAILED, errors |
| `yellow` | `#d29922` | Warnings, deprecation notices |
| `cyan` | `#79c0ff` | Paths, URLs, links |
| `purple` | `#d2a8ff` | Bullet points, list markers, section names |
| `dim` | `#8b949e` | Metadata, timestamps, version info |
| `bold white` | `#e6edf3` | Headers, section dividers, summaries |
| `white` | `#e6edf3` | Normal text |

Combine classes: `bold green` for emphasized success, `bold red` for emphasized failure.

### Common Tool Patterns

**pytest**: Header line → platform info (dim) → collected N items → blank → test lines (green PASSED / red FAILED) → blank → summary line (bold green if all pass, bold red if failures).

**npm install**: "added N packages" → timing → optional audit line (dim).

**npm test / npm run**: Script echo (dim) → tool output → summary.

**git status**: "On branch main" → section headers (bold white) → file listings with color (green for new, red for modified, cyan for paths).

**git log**: Commit hashes (yellow) → author/date (dim) → messages (white).

**cargo / rustc**: Compiling lines (green) → Finished (bold green) or error (bold red).

**Generic**: Use sensible defaults — dim metadata, green for success, red for errors, bold white for headers.

### Timing Guidelines

- `TYPING_SPEED`: 50-70ms per character (looks natural)
- `INITIAL_DELAY`: 400-600ms (pause before typing starts)
- `POST_COMMAND_DELAY`: 400-800ms (simulates tool startup)
- Line delays: 60-100ms for fast sequential lines, 200-400ms for "processing" steps, 80-120ms for metadata
- `POST_ANIMATION_DELAY`: 1200-1800ms (hold final frame for readability)

Keep total animation under 8 seconds. If there are many output lines, reduce per-line delays.

## Step 3: Ensure Dependencies

Check that all required tools are available. Run these checks:

### ffmpeg

```bash
# Check ffmpeg availability
which ffmpeg 2>/dev/null || command -v ffmpeg 2>/dev/null || ffmpeg -version 2>/dev/null
```

If ffmpeg is not found, check common installation locations:
- macOS: `/opt/homebrew/bin/ffmpeg`, `/usr/local/bin/ffmpeg`
- Linux: `/usr/bin/ffmpeg`, `/snap/bin/ffmpeg`
- Windows: `C:/ProgramData/chocolatey/bin/ffmpeg.exe`, `C:/ffmpeg/bin/ffmpeg.exe`

If still not found, **ask the user before installing**:
> "ffmpeg is required but not found. Would you like me to install it? (brew install ffmpeg / choco install ffmpeg / sudo apt install ffmpeg)"

Do NOT install without user confirmation.

### Playwright MCP

Test that the Playwright MCP browser is available by calling `mcp__playwright__browser_snapshot`. If it returns an error about browser not being installed, call `mcp__playwright__browser_install` to install it.

### Python HTTP Server

```bash
python3 --version 2>/dev/null || python --version 2>/dev/null
```

Python is needed for `python -m http.server`. If not available, the skill cannot proceed — inform the user.

## Step 4: Generate HTML

Read the reference template to understand the structure:

```
Read file: <skill-dir>/references/template.html
```

Where `<skill-dir>` is the directory containing this SKILL.md file. Use Glob to locate it if needed:

```
Glob pattern: **/command-giffer/references/template.html
```

Create a temporary working directory:

```bash
TMPDIR=$(mktemp -d 2>/dev/null || mktemp -d -t 'command-giffer')
echo $TMPDIR
```

On Windows (Git Bash), if `mktemp` fails, use:
```bash
TMPDIR="$TEMP/command-giffer-$$"
mkdir -p "$TMPDIR"
echo "$TMPDIR"
```

Write a new HTML file to `$TMPDIR/animation.html` based on the template structure, but with:

1. **Your designed content** from Step 2 substituted into the configurable data section
2. **Exact viewport dimensions** set on `html, body`:
   ```css
   html, body {
     width: WIDTHpx;
     height: HEIGHTpx;
     overflow: hidden;
   }
   ```
3. **The window title** set in both the title-bar span and the `WINDOW_TITLE` variable
4. **The command and output lines** from your design in Step 2

Keep the animation engine code (`runAnimation`, `typeText`, `showOutputLine`, `sleep`, `createLine`) exactly as-is from the template. Only modify the configurable data section.

## Step 5: Serve and Record

### Start HTTP Server

Start a Python HTTP server on a free port, serving the temp directory:

```bash
# Find a free port using Python
PORT=$(python3 -c "import socket; s=socket.socket(); s.bind(('',0)); print(s.getsockname()[1]); s.close()" 2>/dev/null || python -c "import socket; s=socket.socket(); s.bind(('',0)); print(s.getsockname()[1]); s.close()")

cd "$TMPDIR" && python3 -m http.server $PORT --bind 127.0.0.1 &
# or on Windows / if python3 not available:
# cd "$TMPDIR" && python -m http.server $PORT --bind 127.0.0.1 &

# Wait for server to start
sleep 1
```

### Record with Playwright

**This is the critical step.** Use `mcp__playwright__browser_run_code` to create a new browser context with video recording enabled. The standard Playwright MCP tools cannot set `recordVideo` — only `browser_run_code` can do this.

Call `mcp__playwright__browser_run_code` with this code (adjust PORT, WIDTH, HEIGHT, and TMPDIR_PATH):

```javascript
async (page) => {
  const browser = page.context().browser();
  const context = await browser.newContext({
    viewport: { width: WIDTH, height: HEIGHT },
    deviceScaleFactor: 2,
    recordVideo: {
      dir: 'TMPDIR_PATH',
      size: { width: WIDTH * 2, height: HEIGHT * 2 }
    }
  });
  const recPage = await context.newPage();
  await recPage.goto('http://127.0.0.1:PORT/animation.html');
  await recPage.waitForFunction(() => document.title === 'ANIMATION_DONE', null, { timeout: 30000 });
  await recPage.waitForTimeout(500);
  await context.close();
  return 'Recording complete';
}
```

**Important notes:**
- **Use `page.context().browser()`** to access the existing browser instance — `require('playwright')` is not available in the MCP execution context
- `deviceScaleFactor: 2` records at 2x resolution for crisp text
- `recordVideo.size` should be `width * 2` and `height * 2` to match the 2x scale
- Use forward slashes in `TMPDIR_PATH` even on Windows (for JS string)
- Escape backslashes if on Windows: replace `\` with `/` in the path
- Wait for `ANIMATION_DONE` title which the animation engine sets when done
- The extra `waitForTimeout(500)` ensures the last frame is captured
- `context.close()` finalizes the video file — do NOT call `browser.close()` as the browser is shared with the MCP server

After recording, find the video file:

```bash
# On Windows, Git Bash /tmp may differ from the Windows C:/tmp path
# Try both the variable and find to be safe
WEBM_FILE=$(ls "$TMPDIR"/*.webm 2>/dev/null | head -1)
if [ -z "$WEBM_FILE" ]; then
  WEBM_FILE=$(find "$(cygpath -w "$TMPDIR" 2>/dev/null || echo "$TMPDIR")" -name "*.webm" 2>/dev/null | head -1)
fi
echo "Video: $WEBM_FILE"
```

The video file will be in the temp directory with a random name like `a1b2c3d4.webm`.

## Step 6: Convert to GIF

Use a two-pass ffmpeg palette method for high-quality GIF output:

```bash
WEBM_FILE=$(ls "$TMPDIR"/*.webm 2>/dev/null | head -1)
if [ -z "$WEBM_FILE" ]; then
  WEBM_FILE=$(find "$(cygpath -w "$TMPDIR" 2>/dev/null || echo "$TMPDIR")" -name "*.webm" 2>/dev/null | head -1)
fi
OUTPUT_PATH="the --output path"
PALETTE="$TMPDIR/palette.png"

# Pass 1: Generate optimized palette
ffmpeg -i "$WEBM_FILE" -vf "fps=15,scale=WIDTH:-1:flags=lanczos,palettegen=max_colors=128:stats_mode=diff" -y "$PALETTE"

# Pass 2: Create GIF using the palette
ffmpeg -i "$WEBM_FILE" -i "$PALETTE" -lavfi "fps=15,scale=WIDTH:-1:flags=lanczos[x];[x][1:v]paletteuse=dither=floyd_steinberg:diff_mode=rectangle" -y "$OUTPUT_PATH"
```

Where `WIDTH` is the `--width` value from Step 1 (not the 2x recording size — ffmpeg scales down for the final output).

If the GIF is larger than 5MB, retry with `max_colors=64` and `fps=12`.

## Step 7: Clean Up

```bash
# Kill the HTTP server
kill %1 2>/dev/null || true

# If kill %1 doesn't work (e.g., different shell session), find and kill by port:
# lsof -ti:$PORT 2>/dev/null | xargs kill 2>/dev/null || true  # macOS/Linux
# On Windows Git Bash, the backgrounded process is usually cleaned up by kill %1

# Remove temp directory
rm -rf "$TMPDIR"
```

Verify the output GIF exists and is non-empty:

```bash
ls -la "$OUTPUT_PATH"
```

## Step 8: Report

Print a summary to the user:

```
GIF created successfully!

  Output: ./demo.gif
  Size:   245 KB
  Dimensions: 820 x 400

Add to your README:

  ![Demo](./demo.gif)
```

Include the actual file size (formatted in KB or MB) and the actual dimensions.

If the GIF file size exceeds 10MB, warn the user that GitHub has display issues with large GIFs and suggest reducing dimensions or line count.
