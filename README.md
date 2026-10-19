# Bowtie risk analysis editor

A small proof-of-concept editor for **bowtie diagrams** (risk analysis). It is a single, self-contained
file, `bowtie.html`, built to run on locked-down machines: no install, no internet, no build step.

## Running it

Double-click `bowtie.html` (or drag it into Microsoft Edge). It runs from `file://`, uses only system
fonts and makes no network requests. The example bowtie (`examples/sample-bowtie.json` has the same
content) is loaded on start.

## Features

- **Automatic layout**: hazard on top, top event in the centre, threats stacked left, consequences stacked
  right, barriers placed in order along each line. No manual positioning.
- **Editing**: click an element to select it and edit it in the side panel. Add threats/consequences
  (toolbar) and barriers (select a threat, consequence or barrier first). Delete with inline confirmation.
  Reorder barriers with *Move left / right*. Escalation factors can be added to barriers (name + description).
- **Visual cues**: barrier colour = effectiveness (green good, amber fair, red poor, grey unknown); a tag shows
  the barrier type (HW/HU/PR/OT); a badge shows the number of escalation factors. Hazard = yellow hexagon,
  top event = dark red ellipse, threat = orange box, consequence = red-edged box with severity.
- **Zoom/pan**: mouse wheel, drag the background, +/− buttons, *Fit to screen*.
- **Undo/redo**: Ctrl+Z, Ctrl+Y (or Ctrl+Shift+Z); typing in a field is one undo step.
- **Open / Save / Save as**: File System Access API when available (Save writes back to the same file),
  otherwise `<input type=file>` and a `.json` download. Drag and drop a `.json` file onto the page.
  Loaded files are validated (including `schemaVersion`) with readable errors.
  Unsaved changes are shown in the toolbar and trigger a `beforeunload` warning.
- **Autosave/recovery**: a copy goes to `localStorage` on each change; on start you are offered a restore.
- **Export**: SVG, PNG (2x), and a print stylesheet (Ctrl+P / *Print / PDF*) that prints only the diagram.
- **New** loads the example bowtie.

Keyboard: Ctrl+S save, Ctrl+Z/Y undo/redo, Delete removes the selection, Esc deselects.

## Known limitations

- Layout is automatic only; row heights are fixed, long names are truncated in the boxes (full text is in the
  tooltip and side panel).
- Escalation factors are not drawn as boxes on the diagram, only counted on the barrier. Escalation-factor
  barriers exist in the data model and are preserved in the file, but have no editor UI yet.
- Only one diagram per file. Print uses the browser's page setup (A4 landscape suggested).
- When the File System Access API is blocked, *Save* downloads a new file each time instead of overwriting.
- localStorage autosave holds a single recovery copy only.
- No automated tests; verification was done by exercising the page in headless Chromium.

See `ARCHITECTURE.md` for details.
