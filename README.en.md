[简体中文](./README.md) | [English](#) | [繁體中文](./README.zh-Hant.md) | [日本語](./README.ja.md) | [Русский](./README.ru.md)

# ReFlow Smart Designator

Intelligent schematic reference designator reassignment extension for JLCEDA / EasyEDA Pro

<img src="./images/ScreenShot.png" alt="ReFlow Smart Designator Screenshot" width="480" />

## Overview

**ReFlow Smart Designator** understands your circuit logic. It automatically identifies electrically connected components as functional modules (clusters) and assigns consecutive reference designators within each cluster, keeping your schematic organized and readable.

Traditional sequential numbering scatters related components across the designator range. ReFlow uses a Union-Find algorithm to analyze wire connectivity, ensuring that components belonging to the same functional block receive consecutive designators (R/C/U, etc.), dramatically improving schematic readability and maintainability.

## Features

### Three Numbering Modes

| Mode | Example | Description |
|------|---------|-------------|
| Continuous | R1, R2, R3... | Sequential across all pages |
| Per-page | R101, R201 | R101 = page 1 R01, R201 = page 2 R01 |
| Per-block | R101, R201 | R101 = block 1 R01, R201 = block 2 R01, by connected module |

### Scope

- **All Pages** — Assign starting from P1 in page order (default)
- **Current Page** — Reassign only the currently open page

### Starting Number

- **Auto** — Starts from 1 in "All Pages" mode; from max existing + 1 in "Current Page" mode
- **Custom** — Specify a starting number

### Existing Designator Handling (Current Page mode)

- **Skip Existing** — Automatically skip designators used on other pages (default)
- **Overwrite Existing** — Force assign; conflicting designators on other pages become unassigned
- **Clear All** — Assign without regard to other pages

### Lock Components

Enter or auto-fill via the "From Selection" button to preserve specific component designators. Others skip around them.

### More

- **Undo** — One-click restore to previous designators
- **i18n** — Supports Chinese / English interface
- **Live Log** — Shows assignment progress and designator change report

## Usage

1. Open a schematic page
2. Menu → **ReFlow Smart Designator** → **Re-annotate...**
3. Select numbering mode and processing options in the UI
4. Click "Start"

## License

[Apache License 2.0](https://choosealicense.com/licenses/apache-2.0/)
