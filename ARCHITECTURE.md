# Architecture

## Data model (`schemaVersion: 1`)

The JSON model is the single source of truth; the diagram is only a rendering of it. See
`examples/sample-bowtie.json`.

```
{ schemaVersion, meta, hazard, topEvent, threats[], consequences[] }
```

| Object | Fields |
|---|---|
| `meta` | `title`, `author`, `created`, `modified` (ISO 8601 strings; `modified` is set on save) |
| `hazard`, `topEvent` | `id`, `name`, `description` |
| `threat` | `id`, `name`, `description`, `barriers[]` (preventive, ordered from the threat towards the top event) |
| `consequence` | `id`, `name`, `description`, `severity` (`""`, `minor`, `moderate`, `major`, `catastrophic`), `barriers[]` (mitigative, ordered from the top event outwards) |
| `barrier` | `id`, `name`, `description`, `owner`, `type` (`hardware`/`human`/`procedural`/`other`), `effectiveness` (`good`/`fair`/`poor`/`unknown`), `escalationFactors[]` |
| `escalationFactor` | `id`, `name`, `description`, `barriers[]` (same shape as a barrier; recursive) |

IDs come from `crypto.randomUUID()` with a timestamp/random fallback. On load, `normalizeModel` fills defaults and
coerces unknown enum values, so a model written by the editor reloads unchanged.

## Code layout (`bowtie.html`, one inline classic `<script>`)

1. **STATE / MODEL**: `state`, id generation, factories, `validateModel`, `normalizeModel`, `findItem`.
2. **LAYOUT**: `layout(model) -> { nodes, links, bounds }` (pure, no DOM).
3. **RENDER**: `render(positioned, opts) -> <svg>` (builds SVG with presentation attributes so exports are standalone).
4. **HISTORY**: snapshot undo/redo; `mutate(fn, {coalesceKey})` is the only way edits change the model.
5. **EDITING**: select/add/move/delete operations on the model.
6. **PERSISTENCE**: open/save/save-as (with fallbacks), dirty tracking, autosave/restore, drag and drop.
7. **EXPORT**: SVG, PNG, print.
8. **UI WIRING**: canvas (zoom/pan), side panel, banners, keyboard, init.

## Layout and rendering

Rows are `max(#threats, #consequences, 1) * rowH` tall. Threats and consequences are vertically centred on the
centre of that block; the top event sits at the centre, the hazard `hazardZone` px above it. Each line is a
polyline: horizontal at the row height through aligned barrier columns (`barW + gap` apart), then a diagonal in
the final gap to the top event (mirrored on the right). The width of each side is sized to the line with most
barriers. Selection highlight is a thicker stroke. The viewport is a `<g>` with a `translate/scale` transform
(`state.view`); fit-to-screen computes it from `bounds`. Exports re-render with `standalone: true`, which adds a
white background and a `viewBox` covering the whole diagram.

## Persistence strategy and fallbacks

- Primary storage is the user's own `.json` file.
- Open: `showOpenFilePicker` -> falls back to a hidden `<input type=file>` (also used if the picker throws).
  Drag and drop reads `DataTransferItem.getAsFileSystemHandle()` when present to keep a writable handle.
- Save: writes to the stored handle; without a handle it behaves as Save as.
- Save as: `showSaveFilePicker` -> falls back to a Blob download via `<a download>`.
- Dirty flag = current serialized model differs from the serialized model at the last save/open.
- Autosave: one `localStorage` key (`bowtie.autosave.v1`) written on every change inside try/catch; offered for restore at start.

## `file://` constraints

The tool must work when opened from disk in Edge under group policy. On `file://`, ES modules, `fetch()` of
local files and file-loaded Workers are blocked (origin is opaque), and there is no network. Therefore: one file,
a classic inline script, no external resources, system fonts. The SVG namespace URI is built as a split string so
the source contains no URL literals. PNG export loads the SVG through a `data:` URL (no taint on the canvas).
`localStorage` and the File System Access API may be unavailable by policy and are feature-detected.

## Next steps / open questions

- Draw escalation factors (and their barriers) on the diagram and add editor UI for escalation-factor barriers.
- Barrier ↔ threat/consequence cross-references, barrier reuse across lines?
- Schema migrations when `schemaVersion` increases.
- Text measuring instead of character-count wrapping; per-row auto height.
- Multiple diagrams per file, Excel export, risk matrix (out of scope for the POC).
- Automated tests (e.g. unit tests for `layout`, `validateModel`) once a tooling environment is available.
- Open question: preferred print paper size and branding (logo, title block).
