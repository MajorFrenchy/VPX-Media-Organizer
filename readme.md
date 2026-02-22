# 🎰 VPX Organizer

> **Bulk media file renamer and mover for Visual Pinball X tables.**  
> Converts between VPinFE and Batocera media structures in one click — no installation, no dependencies, runs entirely in your browser.

---

## What it does

VPX Organizer solves the tedious problem of reorganising hundreds of VPX table media files when switching between frontends. Instead of renaming files one by one, point it at your root tables folder and it renames and moves everything automatically.

It supports **two directions**:

| Mode | Direction | What happens |
|------|-----------|--------------|
| **VPinFE → Batocera** | `medias/` subfolder → table root | Files are renamed to Batocera conventions and moved up out of `medias/` |
| **Batocera → VPinFE** | Table root → `medias/` subfolder | Files are reverse-renamed to VPinFE conventions and moved into `medias/` (created if missing) |

---

## File Mapping

### VPinFE → Batocera

| Source (`medias/`) | Destination (table root) |
|--------------------|--------------------------|
| `bg.mp4` | `backglass.mp4` |
| `rules.png` / `flyer.jpg` | `backglass.png` / `backglass.jpg` |
| `fulldmd.mp4` | `dmd.mp4` |
| `fulldmd.png` / `fulldmd.jpg` | `dmd.png` / `dmd.jpg` |
| `wheel.apng` | `wheel.png` |
| `loading.mp3` | `audiolaunch.mp3` |
| `table.mp4` | `table.mp4` *(moved, not renamed)* |
| `audio.mp3` | `audio.mp3` *(moved, not renamed)* |

### Batocera → VPinFE *(exact reverse)*

| Source (table root) | Destination (`medias/`) |
|---------------------|-------------------------|
| `backglass.mp4` | `bg.mp4` |
| `backglass.png` / `backglass.jpg` | `rules.png` / `rules.jpg` |
| `dmd.mp4` | `fulldmd.mp4` |
| `dmd.png` / `dmd.jpg` | `fulldmd.png` / `fulldmd.jpg` |
| `wheel.png` | `wheel.apng` |
| `audiolaunch.mp3` | `loading.mp3` |
| `table.mp4` | `table.mp4` *(moved, not renamed)* |
| `audio.mp3` | `audio.mp3` *(moved, not renamed)* |

---

## Expected Folder Structure

### VPinFE (input for V→B, output for B→V)

```
Tables/
└── Attack from Mars/
    ├── Attack from Mars.vpx
    └── medias/
        ├── table.mp4
        ├── bg.mp4
        ├── rules.png
        ├── fulldmd.mp4
        ├── wheel.apng
        ├── loading.mp3
        └── audio.mp3
```

### Batocera (output for V→B, input for B→V)

```
Tables/
└── Attack from Mars/
    ├── Attack from Mars.vpx
    ├── table.mp4
    ├── backglass.mp4
    ├── backglass.png
    ├── dmd.mp4
    ├── wheel.png
    ├── audiolaunch.mp3
    └── audio.mp3
```

---

## Usage

### Requirements

- **Chrome 86+** or **Edge 86+** — the tool uses the [File System Access API](https://developer.mozilla.org/en-US/docs/Web/API/File_System_API) to read and write files directly on disk. Firefox and Safari do not support this API.
- No server, no install, no Node.js. Just open the HTML file.

### Steps

1. **Download** `vpx_organizer.html` from this repo
2. **Open** it in Chrome or Edge (double-click, or drag into browser)
3. **Select a mode** — VPinFE → Batocera or Batocera → VPinFE
4. **Click "Select Folder"** and choose your root tables folder (the folder that *contains* all your individual table subfolders)
5. **Click "Scan"** — the tool walks every subfolder, finds matching media files, and builds a full rename plan
6. **Review** the plan — click any table in the list to see exactly what will be renamed and moved
7. **Click "Run All"** — all renames and moves execute immediately, directly on disk
8. **Save Log** — optionally download a timestamped `.log` of every operation

> ⚠️ **The browser will ask for permission** to access your folder the first time. You must click *Allow* for the tool to work.

---

## UI Overview

```
┌─────────────────────────────────────────────────────────────┐
│  VPX ORGANIZER                                   [ status ] │
├─────────────────────────────────────────────────────────────┤
│  [ VPinFE → Batocera ]  [ Batocera → VPinFE ]              │
├─────────────────────────────────────────────────────────────┤
│  📁 Root folder: /path/to/Tables    [Select] [Scan]         │
├──────────┬──────────┬──────────┬───────────────────────────-┤
│  Tables  │  Files   │  Ops     │  Unmapped                  │
├──────────┴──────────┴──────────┴────────────────────────────┤
│  Table List          │  Rename Plan for selected table       │
│  ─────────────────   │  ──────────────────────────────────── │
│  ● Attack from Mars  │  bg.mp4        →  backglass.mp4       │
│  ● Medieval Madness  │  fulldmd.mp4   →  dmd.mp4             │
│  ● Theatre of Magic  │  wheel.apng    →  wheel.png           │
│                      │  audio.mp3     →  audio.mp3           │
│  [⚡ Run All] [↓ Log] │                                      │
├──────────────────────┴───────────────────────────────────────┤
│  Operation Log                          [████████░░░] 80%   │
│  [13:04:21] ✓ RENAME  bg.mp4  →  backglass.mp4              │
└──────────────────────────────────────────────────────────────┘
```

### Status indicators

| Dot colour | Meaning |
|-----------|---------|
| 🟡 Yellow | Has renames pending |
| 🟢 Green | All files already correctly named |
| 🔴 Red | Has unmapped files (no rule matched) |
| Faded green | Processing complete |

---

## How it works

The tool uses the browser's **File System Access API** (`showDirectoryPicker`) which grants genuine read/write access to a real folder on your disk — not a simulated virtual filesystem. When you click *Run All*:

1. Each media file is **read into memory** as an `ArrayBuffer`
2. The file is **written** to its destination path with the new name
3. The **original is deleted** from the source location

This is a true move-and-rename, equivalent to what a native desktop app would do.

---

## Limitations

- **Chrome / Edge only** — Firefox and Safari lack `showDirectoryPicker` support
- **Large files** — very large video files (>500 MB each) are loaded into browser memory. On most systems this is fine; on machines with limited RAM you may want to process tables in smaller batches
- The `medias/` folder itself is **not deleted** after V→B conversion — it will be left empty. You can remove it manually if desired
- `.vpx` files and hidden system files (`.DS_Store`, `Thumbs.db`, `desktop.ini`) are automatically skipped

---

## Contributing

Pull requests welcome. Some ideas for future improvements:

- [ ] Dry-run / preview mode that shows the plan without executing
- [ ] Custom rule editor (add your own filename mappings)
- [ ] Support for additional frontends (PinballY, PinUp Popper)
- [ ] Batch undo / restore from log

---

## License

MIT — do whatever you want with it.

