# kit

Zero-dependency Python TUI toolkit. True-color ANSI rendering, logging, progress bars, spinners, graphs, live dashboards, syntax highlighting, markdown, markup language, prompts, file trees, themes — all from scratch, no `rich`, no `curses`, no `pygments`.

```
pip install kit-tui          # (once published)
python -m kit.demo           # live preview of everything
```

Requires Python 3.10+, a terminal with true-color support (any modern terminal on Windows/macOS/Linux).

---

## Table of Contents

1. [Color & Gradient](#1-color--gradient)
2. [Style & Layout](#2-style--layout)
3. [LogKit — structured logger](#3-logkit--structured-logger)
4. [Spinners](#4-spinners)
5. [Progress Bars](#5-progress-bars)
6. [Graphs](#6-graphs)
7. [Markup Language](#7-markup-language)
8. [Syntax Highlighting](#8-syntax-highlighting)
9. [Live & Dashboard](#9-live--dashboard)
10. [Themes](#10-themes)
11. [Screen Control](#11-screen-control)
12. [Prompt](#12-prompt)
13. [Logging Bridge](#13-logging-bridge)
14. [Text Object](#14-text-object)
15. [File Tree](#15-file-tree)
16. [Markdown Renderer](#16-markdown-renderer)
17. [Loading Screen](#17-loading-screen)

---

## 1. Color & Gradient

```python
from kit import Color, Gradient
```

### `Color` — ANSI escape helpers

```python
# 16-color constants
Color.RED          # "\033[31m"
Color.BRIGHT_WHITE # "\033[97m"
Color.BG_BLUE      # "\033[44m"
Color.RESET        # "\033[0m"

# True-color (24-bit)
Color.rgb(88, 101, 242)      # → foreground escape
Color.bg_rgb(30, 30, 40)     # → background escape
Color.from_hex("#5865F2")    # hex string → rgb escape

# 256-color
Color.color256(196)
Color.bg_color256(52)

# Gradient text
Color.gradient_text("hello", (255, 80, 0), (255, 220, 0))
# multi-stop gradient — stops are (r,g,b) tuples
Color.multi_gradient("loading...", (88,101,242), (0,200,255), (0,255,150))

# Full HSV rainbow rotation
Color.rainbow("spectral")

# Utility
Color.blend((255,0,0), (0,0,255), 0.5)   # lerp → (127, 0, 127)
Color.pulse((88,101,242), t)             # sinusoidal brightness, t ∈ [0,1]
Color.dim_rgb((200,200,200), factor=0.4) # darken tuple → rgb escape
Color.strip("text with \033[31mcodes\033[0m")  # strip all ANSI

# Convenience wrappers
Color.bold("text")
Color.italic("text")
Color.underline("text")
Color.strike("text")
Color.dim_text("text")
```

### `Gradient` — named stop lists

Every preset is a `list[tuple[int,int,int]]` passed to `Color.multi_gradient`.

| Name      | Visual |
|-----------|--------|
| `FIRE`    | red → orange → yellow |
| `OCEAN`   | deep navy → cyan |
| `NEON`    | pink → violet → cyan |
| `MATRIX`  | dark green → bright green |
| `GOLD`    | amber → bright gold → amber |
| `BLOOD`   | dark red → bright red |
| `CYBER`   | teal → blue → violet |
| `DISCORD` | discord blurple loop |
| `CANDY`   | pink → yellow → light blue |
| `TOXIC`   | lime → yellow-green |
| `VOID`    | deep purple → violet |
| `LAVA`    | red → orange → yellow |
| `SUNSET`  | orange → crimson → indigo |
| `ARCTIC`  | ice blue → white-blue |
| `RUST`    | dark brown → orange |
| `ACID`    | lime → bright green |
| `GHOST`   | cool grey loop |
| `INFRA`   | deep blue → purple → red → yellow |
| `ROYAL`   | purple → lavender loop |

```python
# Build from hex strings
stops = Gradient.from_hex("#ff0080", "#8000ff", "#00c8ff")

# Expand stop list to n evenly-spaced (r,g,b) tuples
pts = Gradient.interpolate(Gradient.FIRE, 256)
```

---

## 2. Style & Layout

```python
from kit import Style, BorderStyle, render_panel, render_table, render_two_col, render_header, render_badge, strip_ansi, term_width
```

### `Style` — ANSI text attributes

```python
Style.BOLD, Style.DIM, Style.ITALIC, Style.UNDERLINE, Style.BLINK, Style.STRIKE, Style.RESET
```

### `BorderStyle` — box-drawing presets

`SINGLE`, `DOUBLE`, `ROUNDED`, `BOLD`, `DASHED`, `ASCII`

### Layout functions

```python
# Bordered panel — returns string
render_panel(
    lines,                      # list[str]
    title=None,                 # header text
    border=BorderStyle.ROUNDED,
    border_color=Color.rgb(88,101,242),
    title_color=Color.BRIGHT_WHITE,
    padding=1,
    width=None,                 # auto from terminal width
)

# Table — returns string
render_table(
    headers,                    # list[str]
    rows,                       # list[list[str]]
    border=BorderStyle.SINGLE,
    header_color=...,
    border_color=...,
    col_colors=None,            # list[str] per column
    zebra=False,                # alternate row shading
    align=None,                 # list["left"|"right"|"center"]
)

# Two-column layout — returns string
render_two_col(left_lines, right_lines, gap=4)

# Centered header bar — returns string
render_header(text, gradient=None, width=None)

# Small badge pill — returns string
render_badge(text, color=None)

# Strip ANSI escapes from a string
strip_ansi(s)

# Terminal width (falls back to 80)
term_width()
```

---

## 3. LogKit — structured logger

```python
from kit import LogKit, Level

log = LogKit(
    name="app",
    min_level=Level.DEBUG,
    show_time=True,
    show_name=True,
    name_gradient=Gradient.DISCORD,   # gradient for the name badge
    time_color=Color.BRIGHT_BLACK,
    file=sys.stdout,
    thread_safe=False,
    history=False,                    # keep in-memory ring buffer
    history_maxlen=500,
)
```

### Log levels

```python
log.debug("probe sent")
log.info("handshake complete")
log.success("payload injected")
log.warning("retry 3/5")
log.error("connection refused")
log.critical("watchdog timeout")

# Custom color override
log.info("custom", color=Color.rgb(200, 100, 255))
```

`Level` constants: `DEBUG=0`, `INFO=1`, `SUCCESS=2`, `WARNING=3`, `ERROR=4`, `CRITICAL=5`

### Display methods

```python
log.banner("SYSTEM COMPROMISED", gradient=Gradient.BLOOD, border=BorderStyle.DOUBLE)

log.rule("section header", gradient=Gradient.NEON)      # full-width divider
log.rule()                                               # plain divider
log.divider()                                            # blank line

log.section("phase 2", gradient=Gradient.CYBER)         # double-line section marker

log.panel(["line 1", "line 2"], title="status", border=BorderStyle.ROUNDED)

log.table(
    ["host", "status", "ip"],
    [["web01", "up", "10.0.0.1"], ["db02", "down", "10.0.0.2"]],
    zebra=True,
)

log.kv([("pid", "1234"), ("arch", "x64")], sep="  →  ")
log.keyval("target", "192.168.1.100")

log.columns(["item1","item2","item3","item4"], ncols=2, headers=["col A","col B"])

log.callout("Heap base leaked at 0x7fff00001000", style="note")
# styles: "note", "tip", "warn", "danger", "info"

log.badge("STABLE", color=Color.rgb(50,220,100))
```

### Data inspection

```python
log.tree({"a": {"b": [1, 2, 3]}, "c": "val"})

log.json({"status": "ok", "code": 200, "data": None})

log.hex_dump(bytes(range(64)), width=16, highlight=[0x00, 0xff])

log.diff(old_str, new_str, label_a="before", label_b="after")

log.inspect(obj, show_private=False, show_methods=False)

log.traceback()                # current exception
log.traceback(exc)             # specific exception object
log.multiline(["line1", "line2"], level=Level.INFO)
```

### Progress & metrics

```python
# Inline progress bar (overwrites current line)
for i in range(total + 1):
    log.progress_bar(i, total, label="downloading", show_eta=True, start_time=t0)

# Inline counter
log.counter("packets", current, total, bar=True, bar_width=20)

# Status line (overwritten by next log call)
log.status("waiting for beacon...")
log.status_clear()

# Sparkline
log.sparkline([10, 25, 18, 40, 35, 55], label="rtt", gradient=Gradient.NEON)

# Rate and elapsed
log.rate("req", count=1024, elapsed=2.5, unit="/s")
log.elapsed(start_time, label="total time")
```

### New methods (via module integration)

```python
log.syntax(code, lang="python", line_numbers=True, title="exploit.py")
log.markdown(md_text)
log.markup("[fire]armed[/fire] and [bold green]ready[/bold green]")
log.file_tree("./src", max_depth=3, show_size=True)
log.theme("dracula")       # switch active kit theme
```

### History

```python
log = LogKit("app", history=True)
log.info("boot")
records = log._hist.all()   # [{"level": 1, "msg": "boot", "ts": "..."}]
log._hist.clear()
```

---

## 4. Spinners

```python
from kit import Spinner, SpinnerGroup, SPINNER_FRAMES
```

### `Spinner`

```python
s = Spinner("connecting to C2", style="dots")  # or style="line", "arc", "bounce", etc.
s.start()
# ... work ...
s.stop("connected")      # final message replaces spinner

# Context manager
with Spinner("encrypting payload"):
    encrypt(data)

# Style names
list(SPINNER_FRAMES.keys())
# "dots", "line", "arc", "bounce", "pulse", "bar", "square", "clock", ...
```

### `SpinnerGroup`

```python
sg = SpinnerGroup([
    "stage 1 — recon",
    "stage 2 — exploit",
    "stage 3 — persist",
])
sg.start()
time.sleep(1)
sg.done(0)        # mark stage 1 done (checkmark)
time.sleep(1)
sg.done(1)
sg.stop()
```

---

## 5. Progress Bars

```python
from kit import ProgressBar, MultiBar, ThreadedBar, RateBar, BAR_STYLES
```

### `ProgressBar`

```python
b = ProgressBar(
    total=100,
    width=40,
    style="block",          # key from BAR_STYLES
    fill_gradient=Gradient.FIRE,
    prefix="download ",
    show_count=True,
    show_pct=True,
)

for i in range(101):
    b.update(i)             # rewrites current line
    time.sleep(0.02)
print()                     # newline after completion

# Iterable wrapper
for item in b(my_list):
    process(item)

# Available styles
list(BAR_STYLES.keys())
# "block", "shade", "slim", "dot", "pipe", "equal", ...
```

### `MultiBar`

Renders multiple bars simultaneously, clearing and redrawing on each call.

```python
mb = MultiBar([
    ("loader",  40, Gradient.DISCORD),   # (label, total, gradient_stops)
    ("encoder", 60, Gradient.MATRIX),
    ("packer",  30, Gradient.FIRE),
], width=35)

vals = [0, 0, 0]
for i in range(60):
    vals[0] = min(i + 1, 40)
    vals[1] = i + 1
    vals[2] = min(i + 1, 30)
    mb.render_all(vals)
    time.sleep(0.03)
print()
```

### `ThreadedBar`

Auto-rendering bar that runs in a background thread.

```python
tb = ThreadedBar(total=100, interval=0.05, prefix="upload ", fill_gradient=Gradient.OCEAN)
tb.start()
for chunk in chunks:
    upload(chunk)
    tb.inc()
tb.stop()
```

### `RateBar`

Tracks throughput (items/sec) alongside progress.

```python
rb = RateBar(total=1000, prefix="packets ")
rb.start()
for i in range(1001):
    rb.update(i)
    time.sleep(0.01)
print()
```

---

## 6. Graphs

```python
from kit import (
    bar_chart, sparkline, heatmap_row, column_chart,
    line_graph, scatter, gauge, timeline,
    stacked_bar_chart, pie_chart,
)
```

All graph functions return strings — `print()` them or embed in panels.

```python
# Horizontal bar chart
print(bar_chart(
    [92, 78, 45, 61],
    labels=["RCE", "LPE", "SQLi", "XSS"],
    title="severity",
    gradient=Gradient.BLOOD,
    width=35,
))

# Inline sparkline (returns string)
s = sparkline([10, 25, 18, 40, 35, 55], gradient=Gradient.NEON)

# Single heatmap row
print(heatmap_row([0.1, 0.5, 0.9, 0.3], labels=["M","T","W","T"]))

# Vertical column chart
print(column_chart([30, 80, 50, 90], labels=["Q1","Q2","Q3","Q4"], height=10))

# Multi-series line graph
# colors must be (r,g,b) tuples, NOT escape strings
print(line_graph(
    [[10,25,18,40], [5,15,30,20]],
    labels=["in", "out"],
    colors=[(88,101,242), (237,66,69)],
    height=8, width=50,
))

# Scatter plot
pts = [(x, y), ...]
print(scatter(pts, width=50, height=16, gradient=Gradient.NEON))

# Radial gauge
print(gauge(0.73, label="CPU", gradient=Gradient.LAVA))

# Horizontal timeline
events = [("boot", 0), ("exploit", 3), ("shell", 7), ("exfil", 12)]
print(timeline(events, width=60))

# Stacked bar chart
print(stacked_bar_chart(
    series=[[30,50,20], [40,35,25]],
    labels=["host1", "host2"],
    series_labels=["user", "kernel", "idle"],
    colors=[(88,101,242),(59,165,93),(250,166,26)],
    title="CPU breakdown",
    width=40,
))

# Pie chart (unicode block rendering)
print(pie_chart(
    data=[40, 30, 20, 10],
    labels=["RCE", "LPE", "DoS", "Info"],
    title="vuln classes",
    show_pct=True,
))
```

---

## 7. Markup Language

```python
from kit import markup_render, markup_strip, mprint, mformat
```

A lightweight inline markup language parsed without external deps.

### Tags

```
[red]text[/red]        [green] [blue] [yellow] [cyan] [magenta] [white] [dim]
[bold]text[/bold]      [italic] [underline] [strike] [blink]
[rgb(88,101,242)]text[/rgb]
[#5865f2]text[/#5865f2]
[bg_rgb(30,30,40)]text[/bg_rgb]

# Combined
[bold red]critical alert[/bold red]

# Named gradients  (no closing tag, applies to full argument)
[fire]armed[/fire]    [neon] [matrix] [ocean] [blood] [cyber] [discord] [candy]
[rainbow]full spectrum[/rainbow]

# Hyperlink
[link=https://example.com]click here[/link]
```

```python
# Render to ANSI string
s = markup_render("[bold red]ALERT[/bold red]: [neon]payload staged[/neon]")

# Strip all tags (plain text)
plain = markup_strip("[bold]text[/bold]")  # → "text"

# Print directly
mprint("[fire]system compromised[/fire]")

# Format with variables
mprint(mformat("[bold]{host}[/bold] responded in {ms}ms", host="10.0.0.1", ms=42))
```

---

## 8. Syntax Highlighting

```python
from kit import highlight, highlight_auto, detect_lang
```

Regex-based tokenizer, zero deps.

### Supported languages

`python`, `json`, `c`, `cpp`, `js`, `javascript`, `sh`, `bash`, `shell`,
`yaml`, `sql`, `html`, `css`, `rust`, `go`

```python
# Highlight with known language
code = "def exploit(buf): return buf[:8] + p64(0xdeadbeef)"
print(highlight(code, lang="python", line_numbers=True))

# Auto-detect from filename
print(highlight_auto(code, filename="exploit.py", line_numbers=True))

# Detect language from content/filename
lang = detect_lang(code, filename="main.rs")   # → "rust"
```

---

## 9. Live & Dashboard

```python
from kit import Live, Dashboard, LogPane
```

### `Live`

Continuously redraws a block of content in-place.

```python
import time
data = {"count": 0}

def render():
    return f"packets: {data['count']}"

with Live(render, refresh_rate=10, transient=False):
    for i in range(100):
        data["count"] = i
        time.sleep(0.05)
```

- `content` — string or `callable() → str`
- `refresh_rate` — redraws per second
- `transient=True` — clears the block on exit

### `Dashboard`

Fixed-layout grid of named cells.

```python
dash = Dashboard()

dash.row("header", height=3)
dash.cols(["left", "right"], heights=[20, 20])    # side-by-side columns
dash.row("footer", height=1)

dash["header"] = render_panel(["STATUS BOARD"], border=BorderStyle.DOUBLE)
dash["left"]   = "cell content left"
dash["right"]  = "cell content right"
dash["footer"] = "ready"

with Live(dash.render, refresh_rate=4):
    for t in range(50):
        dash["footer"] = f"tick {t}"
        time.sleep(0.1)
```

### `LogPane`

Scrolling fixed-height log buffer, thread-safe.

```python
pane = LogPane(height=10)

pane.push("[OK]  boot complete")
pane.push("[ERR] write failed")

# Integrate with Live
with Live(pane.render, refresh_rate=5):
    for event in event_stream():
        pane.push(format_event(event))
        time.sleep(0.05)
```

---

## 10. Themes

```python
from kit import theme_apply, theme_color, theme_names, theme_info
from kit import DISCORD, HACKER, DRACULA, NORD, MONOKAI, SOLARIZED, CYBERPUNK, BLOOD
```

### Available themes

`discord`, `hacker`, `dracula`, `nord`, `monokai`, `solarized`, `cyberpunk`, `blood`

### Usage

```python
theme_apply("dracula")

# Semantic color keys
# "primary", "secondary", "success", "warning", "error", "info", "dim", "bright"
# "bg" → (r,g,b) tuple
# "gradient" → list of (r,g,b) stops

c = theme_color("error")          # ANSI escape string
bg = theme_color("bg")            # (r,g,b) tuple
grad = theme_color("gradient")    # gradient stop list

# List all theme names
names = theme_names()

# Info dict for current/named theme
info = theme_info("monokai")
info = theme_info()               # active theme

# Or access dicts directly
DRACULA["primary"]   # ANSI escape
DRACULA["gradient"]  # stop list
```

---

## 11. Screen Control

```python
from kit import Screen, ScreenWriter
```

### `Screen` — classmethods

```python
Screen.move(row, col)          # move cursor
Screen.clear()                 # clear entire screen
Screen.hide_cursor()
Screen.show_cursor()
Screen.set_title("my app")     # terminal window title
Screen.bell()

w, h = Screen.size()           # → (columns, rows)

Screen.draw_box(row, col, width, height, char="─", border=BorderStyle.SINGLE)
Screen.fill_rect(row, col, width, height, char=" ", color=None)
Screen.hyperlink(text, url)    # OSC 8 hyperlink
Screen.notify(title, body)     # desktop notification (OSC 99)
```

### Context managers

```python
# Full alternate screen (hides scrollback, restores on exit)
with Screen.fullscreen:
    ...

# Hide cursor for a block
with Screen.cursor_hidden:
    ...

# Save cursor, move to (row,col), restore on exit
with Screen.at(5, 10):
    print("drawn at row 5 col 10")
```

### `ScreenWriter`

```python
sw = ScreenWriter()
sw.at(3, 5, f"{Color.rgb(88,101,242)}hello{Color.RESET}")
sw.at(4, 5, "world")
sw.flush()
```

---

## 12. Prompt

```python
from kit import Prompt
```

All methods read from stdin and render styled prompts.

```python
# Text input
name = Prompt.ask("target hostname", default="localhost", hint="e.g. 10.0.0.1")

# Boolean
confirmed = Prompt.confirm("continue?", default=True)  # → bool

# Numbered menu
choice = Prompt.choose(
    "select payload",
    choices=["reverse shell", "bind shell", "meterpreter"],
    default="reverse shell",
)

# Multi-select menu (returns list)
selected = Prompt.choose("select modules", choices=[...], multi=True)

# Masked input (password)
key = Prompt.secret("encryption key", confirm=True)

# Integer with range validation
port = Prompt.integer("port", min_val=1, max_val=65535, default=4444)

# Float
timeout = Prompt.float_("timeout seconds", default=5.0)

# Path (validates existence)
path = Prompt.path("config file", must_exist=True)

# Multi-line text
lines = Prompt.multi_choice("add hosts (one per line)", ...)
```

---

## 13. Logging Bridge

```python
from kit import LogKitHandler, setup_logging, suppress_loggers
```

Routes Python's stdlib `logging` module through a `LogKit` instance.

```python
import logging
from kit import LogKit, setup_logging, suppress_loggers

log = LogKit("app", show_time=True)

# Attach to root logger — all logging.* calls go through log
handler = setup_logging(log, level=logging.DEBUG, capture_warnings=True)

# Attach to specific loggers only
setup_logging(log, loggers=["httpx", "asyncio"])

# Silence noisy loggers
suppress_loggers("urllib3", "PIL")

# Manual handler
handler = LogKitHandler(log, show_name=True)
logging.getLogger().addHandler(handler)
```

---

## 14. Text Object

```python
from kit import Text
```

Composable styled text with layout methods.

```python
t = Text()
t.append("status: ", "bold")
t.append("ONLINE",   "rgb(50,220,100)")
t.append_markup("[fire]armed[/fire]")

print(t.render())      # ANSI string
print(t.plain)         # stripped text
print(t.width)         # visual width (no escape codes)

# Truncate / wrap / justify
t.truncate(40, overflow="…")
lines = t.wrap(60)                    # → list[Text]
t.justify("center", 80)

# Substring highlight
t.highlight_substr("ONLINE", "bold yellow")

# Class methods
t = Text.assemble(
    ("label: ", "dim"),
    ("value",   "bold green"),
)
t = Text.from_markup("[bold red]ALERT[/bold red]")
t = Text.gradient("spectral", (255,0,128), (0,200,255))
t = Text.rule(title="section", width=60, char="─")
```

---

## 15. File Tree

```python
from kit import render_file_tree, print_file_tree
```

60+ file extension color mappings, special file detection, symlink display.

```python
# Returns string
tree_str = render_file_tree(
    path="./src",
    max_depth=4,
    show_hidden=False,
    show_size=True,
    dir_only=False,
    sort_dirs_first=True,
    summary=True,       # "N files, M dirs" footer
)

# Print directly
print_file_tree("./src", max_depth=3, show_size=True)
```

Color-coded by extension: `.py` (blue), `.rs` (orange), `.c`/`.cpp` (cyan), `.sh` (green), `.json`/`.yaml` (yellow), `.md` (white), `.env` (red), images (magenta), binaries (bright red), and more. Special dirs `.git`, `node_modules`, `__pycache__` have distinct colors.

---

## 16. Markdown Renderer

```python
from kit import markdown_render, md_print
```

Two-pass GFM renderer, zero deps.

### Supported syntax

- Headings `#`–`######` — h1 with gradient bar, h2 with underline, h3+ prefixed
- **Bold** `**text**`, *italic* `*text*`, ~~strikethrough~~ `~~text~~`
- Inline `code`
- Links `[text](url)`, auto-links `<url>`
- Blockquotes `> text`
- Unordered lists `- / * / +` (3 levels of nesting with distinct bullets)
- Ordered lists `1. 2. 3.`
- GFM tables with alignment (`left`, `center`, `right`)
- Fenced code blocks ` ``` lang ` — syntax highlighted via `kit.syntax`
- Horizontal rules `---` / `***`

```python
md = """
# Exploitation Report

## Summary

Target responded to **CVE-2024-XXXX** with a *9.8 CVSS* score.

### Affected endpoints

- `/api/upload` — RCE via path traversal
- `/auth/token` — SQL injection

```python
payload = b"\\x00" * 8 + p64(win_addr)
sock.send(payload)
```

| Component | Status | Severity |
|-----------|--------|----------|
| web       | pwned  | critical |
| db        | clean  | none     |
"""

md_print(md)
s = markdown_render(md)   # → ANSI string
```

---

## 17. Loading Screen

```python
from kit import loading_screen, G_BOOT, G_READY, G_PULSE
```

Full-screen animated boot sequence.

```python
loading_screen(
    title="IMPLANT LOADER v2.0",
    steps=[
        "initializing crypto",
        "probing target",
        "establishing tunnel",
        "loading modules",
    ],
    gradient=G_BOOT,        # G_BOOT, G_READY, G_PULSE, or custom stop list
    step_delay=0.8,
    final_msg="ready",
)
```

---

## Quick-start example

```python
from kit import LogKit, Level, Color, Gradient, Spinner, ProgressBar, bar_chart, mprint, theme_apply

theme_apply("dracula")

log = LogKit("demo", show_time=True)
log.banner("KIT DEMO", gradient=Gradient.NEON)
log.info("starting up")
log.success("connected to 10.0.0.1:4444")
log.warning("retry 2/5 — timeout")
log.error("module load failed: access denied")

log.kv([
    ("pid",   "31337"),
    ("arch",  "x64"),
    ("priv",  "SYSTEM"),
    ("os",    "Windows 11 22H2"),
])

with Spinner("staging payload", style="dots"):
    import time; time.sleep(1.5)

b = ProgressBar(total=100, width=35, fill_gradient=Gradient.FIRE, prefix="upload ")
for i in range(101):
    b.update(i)
    time.sleep(0.01)
print()

print(bar_chart([92,78,45,61], labels=["RCE","LPE","SQLi","XSS"], gradient=Gradient.BLOOD))

mprint("[fire]armed[/fire] and [bold green]ready[/bold green]")
```

---

## Project layout

```
kit/
├── __init__.py      exports
├── colors.py        Color, Gradient
├── styles.py        Style, BorderStyle, layout helpers
├── logger.py        LogKit, Level
├── spinners.py      Spinner, SpinnerGroup
├── bars.py          ProgressBar, MultiBar, ThreadedBar, RateBar
├── graphs.py        bar_chart, line_graph, scatter, pie_chart, ...
├── markup.py        tag markup language
├── syntax.py        regex-based syntax highlighter
├── live.py          Live, Dashboard, LogPane
├── themes.py        8 named themes
├── screen.py        Screen, ScreenWriter
├── prompt.py        Prompt.*
├── loghandler.py    stdlib logging bridge
├── text.py          Text object
├── filetree.py      render_file_tree
├── markdown.py      GFM markdown renderer
├── loadscreen.py    animated boot screen
└── demo.py          runnable feature showcase
```

---

## License

MIT
