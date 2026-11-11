# FlashFX Alpha

FlashFX Alpha is the earliest prototype of FlashFX, a browser-based design editor intended to grow into a motion design tool. It is a single-page React application with a canvas for shapes, text, lines and images, a layers panel, a properties panel, PNG/JPEG/ZIP export, and project storage either in the browser (guest mode) or in Supabase (signed-in accounts). The page title in `index.html` is "FlashFX DEMO - UI Design Tool", which is an accurate summary of what this version does: the design side works, while the animation side is only a visual scaffold.

This repository is kept for history. It was followed by [FlashFX_MVP](https://github.com/gabrielebolognese/FlashFX_MVP), and the current editor is [FlashFX-v2](https://github.com/gabrielebolognese/FlashFX-v2).

## Features

### Canvas and elements

- Element types: rectangle, circle, text, button, chat bubble, chat frame, line, arrow, pen path, image and group.
- Per-element position, size, rotation, opacity, lock and visibility.
- Fill colour or linear/radial gradient with multiple colour stops, stroke, corner radius and drop shadow.
- Text controls: font family (a fixed list of six fonts), size, weight, alignment (horizontal and vertical), colour, letter spacing, word spacing, line height and decoration.
- Lines, arrows and pen paths: editable points, per-point corner radius, arrowheads (triangle, circle, bar, diamond) on either end, line cap and join, dash patterns and smoothing. The line renderer also understands trim start/end and closed pen paths, but the editor has no controls for them.
- Grouping and ungrouping, duplication, multi-selection and a right-click context menu.
- Undo and redo history.
- Zoom between 25% and 300%, a configurable grid, snap to grid and visual snap guides.
- Canvas background built from stacked gradient layers with colour stops, direction and blend modes.

### Images

- Import images from disk.
- Image search through the Google Custom Search API and image generation with DALL-E 3 (see the note on API keys below).
- Image adjustment controls in the properties panel: brightness, contrast, exposure, gamma, saturation, vibrance, temperature, tint, hue, lightness, grayscale, sepia, colour balance, levels, RGB channels, blur and sharpen. These sliders only store values on the element: the canvas draws the image unfiltered, and the WebGL filter processor in `src/utils/webglFilters.ts` is not imported anywhere, so the adjustments have no visible effect in this version.

### Panels

- Layers panel with tabs for layers, AI chat and presets.
- Properties panel with dedicated tabs for shapes, text, lines and images.
- Presets: save a selection as a reusable preset and insert it again later (stored in the Supabase `presets` table).
- Plain-text JSON editors for a single element and for the whole project; the project editor validates against a JSON schema with Ajv.
- Editor settings modal and a keyboard shortcuts reference modal.
- A seven-step in-app tutorial overlay.

### AI (experimental)

- An AI chat tab that turns a text prompt into canvas elements through a three-stage pipeline built on the OpenAI Assistants API: prompt validation, a high-level breakdown into shapes, and low-level generation of each shape. Chat history is kept in `localStorage`.
- The pipeline uses assistant IDs hardcoded in `src/services/OpenAIService.ts`, so it only works with the OpenAI account those assistants belong to.

### Timeline (scaffold only)

- A two-part timeline area with a general clip timeline and an animation timeline, a ruler, a draggable playhead and a resizable splitter.
- Keyframes are not implemented. The animation timeline shows "No keyframe features available yet" and its "Add Keyframe" control is disabled. Only the design layout mode is available.

### Export

- Export the whole canvas as PNG or JPEG.
- Export every shape as a separate PNG inside a ZIP archive.
- Export the current selection as a PNG, or as a ZIP when several elements are selected.

### Projects and accounts

- Guest mode is on by default: projects are stored in `localStorage` and no account is needed.
- Sign up with email, password and username; sign in with email or username; OAuth sign-in with Google (requires the provider to be enabled in the Supabase project). A GitHub sign-in function exists in `src/contexts/AuthContext.tsx` but is only used by the unused `LoginPage.tsx`.
- Cloud projects in Supabase with per-user storage tracking (default limit 100 MB), and a storage indicator. An avatar upload service (`src/services/AvatarService.ts`, 5 MB maximum) exists but no screen uses it.
- Save and load projects as `.ffxproj` files.
- Automatic preview backup every 60 seconds: for signed-in users a canvas preview is uploaded to the `project-previews` storage bucket; in guest mode it is kept in `localStorage`.

## Getting started

### Prerequisites

- Node.js 18 or 20+ (required by Vite 5) and npm.
- Optional: a Supabase project, needed only for accounts, cloud projects, presets and avatars.

### Download

```bash
git clone https://github.com/gabrielebolognese/FlashFX-Alpha.git
cd FlashFX-Alpha
```

### Install dependencies

```bash
npm install
```

### Configuration

Create a `.env` file in the project root (it is git-ignored). There is no `.env.example`; the variable names below come from the code.

| Variable | Used in | Purpose |
| --- | --- | --- |
| `VITE_SUPABASE_URL` | `src/lib/supabase.ts` | Supabase project URL |
| `VITE_SUPABASE_ANON_KEY` | `src/lib/supabase.ts` | Supabase anon key |
| `VITE_OPENAI_API_KEY` | `src/services/OpenAIService.ts` | OpenAI key for the AI chat pipeline |

Without the Supabase variables the app still starts, logs an error to the console and falls back to a placeholder client, so only guest mode works.

To set up the database, apply the SQL files in `supabase/migrations/` to your Supabase project in filename order. They create the `profiles`, `projects`, `presets` and `project_files` tables and the `project-previews` and `avatars` storage buckets.

Note on API keys: the Google image search modal, the DALL-E modal, `FlashFX_AI_Component.tsx` and `AIChatTab.tsx` contain credentials written directly in the source instead of environment variables, and `OpenAIService.ts` falls back to one when `VITE_OPENAI_API_KEY` is not set. To use those features you have to replace them with your own keys in `src/components/image/GoogleImageSearchModal.tsx`, `src/components/image/DalleGenerateModal.tsx`, `src/components/FlashFX_AI_Component.tsx`, `src/components/design-tool/AIChatTab.tsx` and `src/services/OpenAIService.ts`. All of these calls are made from the browser, so any key you supply is visible to users of the built app.

### Run in development

```bash
npm run dev
```

Vite prints the local URL to open in the browser.

## Build and deploy

```bash
npm run build
```

The production build is written to `dist/`. It is a static site and can be served by any static host; `npm run preview` serves the build locally. Routing is hash-based (`#/home`, `#/auth`, `#/editor`), so no server-side rewrite rules are needed. The repository contains no deployment configuration or CI workflow.

## Usage

1. Open the app. It starts on the home page (`#/home`) in guest mode.
2. Create a new project with a name and background colour, or load a `.ffxproj` file. Use `#/auth` to sign up or sign in if Supabase is configured.
3. Opening a project loads the editor (`#/editor?project=<id>`). Add elements from the top bar, arrange them on the canvas, and edit them in the properties panel on the right. The layers panel on the left also holds the AI and presets tabs.
4. Export from the top bar or with `Ctrl+E`.

### npm scripts

| Script | What it does |
| --- | --- |
| `npm run dev` | Starts the Vite development server |
| `npm run build` | Builds the production bundle into `dist/` |
| `npm run preview` | Serves the production build locally |
| `npm run lint` | Runs ESLint over the project |

### Keyboard shortcuts

These are the shortcuts implemented in `src/hooks/useKeyboardShortcuts.ts` and `src/hooks/useGlobalKeyboardShortcuts.ts`. The in-app shortcuts modal lists some different keys and some animation shortcuts that are not implemented.

| Keys | Action |
| --- | --- |
| `Q` / `W` / `E` | Create rectangle / circle / text |
| `R` / `T` / `Y` | Create button / chat bubble / chat frame |
| `U` | Create line |
| `I` | Open a file picker; the shortcut only adds a placeholder rectangle named after the chosen file (use the top bar to import a real image) |
| `G` | Toggle grid |
| `+` / `-` | Zoom in / out |
| `Ctrl+Z`, `Ctrl+Shift+Z` or `Ctrl+Y` | Undo, redo |
| `Ctrl+D` | Duplicate selection |
| `Ctrl+G`, `Ctrl+Shift+G` | Group, ungroup |
| `Ctrl+A` | Select all |
| `Ctrl+E` | Export |
| `Ctrl+;` | Toggle snapping |
| Arrow keys, `Shift` + arrow keys | Nudge selection by 1 px, 10 px |
| `Delete` / `Backspace` | Delete selection |
| `Esc` | Deselect |

## Project structure

| Path | Contents |
| --- | --- |
| `src/App.tsx` | Hash router: home, auth and editor pages |
| `src/pages/` | Home (project list), auth and login pages |
| `src/components/UIDesignTool.tsx` | Editor root: state, history, shortcuts, modals |
| `src/components/design-tool/` | Canvas, elements, lines, layers, AI chat, presets, JSON editors, grid, snapping |
| `src/components/layout/` | Top bar, properties panel tabs, background settings, design layout |
| `src/components/image/` | Image import, Google search, DALL-E and adjustment controls |
| `src/components/timeline/` | General and animation timeline scaffolds |
| `src/components/tutorial/` | Tutorial overlay and panel |
| `src/components/modals/`, `auth/`, `project/`, `storage/` | Project, settings, auth and storage UI |
| `src/export/` | Canvas, shape and ZIP exporters and the export dialog |
| `src/services/` | Supabase projects, presets, storage, avatars, auto backup, project files, OpenAI |
| `src/hooks/`, `src/contexts/` | Editor hooks, auth and tutorial contexts |
| `src/types/`, `src/utils/` | Type definitions, filters, JSON schema and validation, grouping helpers |
| `supabase/migrations/` | SQL migrations for tables, policies and storage buckets |

## Tech stack

- React 18 and TypeScript
- Vite 5
- Tailwind CSS 3 with PostCSS and Autoprefixer
- Supabase (`@supabase/supabase-js`) for auth, database and storage
- Ajv for project JSON validation (`monaco-editor` is listed in `package.json` but is not imported by the code)
- html-to-image, JSZip and file-saver for export
- lucide-react icons
- OpenAI Assistants and Images APIs, Google Custom Search API (called from the browser)
- ESLint with typescript-eslint

## Further documentation

Implementation notes written during development are kept in the repository root. They describe the intent at the time and may not match the final code.

- [EXECUTIVE_SUMMARY.md](EXECUTIVE_SUMMARY.md)
- [AI_IMPLEMENTATION_ANALYSIS.md](AI_IMPLEMENTATION_ANALYSIS.md)
- [AI_PIPELINE_STEP_BREAKDOWN.md](AI_PIPELINE_STEP_BREAKDOWN.md)
- [AUTHENTICATION_SYSTEM_DOCUMENTATION.md](AUTHENTICATION_SYSTEM_DOCUMENTATION.md)
- [AUTHENTICATION_FIX_IMPLEMENTATION.md](AUTHENTICATION_FIX_IMPLEMENTATION.md)
- [AUTH_DATABASE_ANALYSIS.md](AUTH_DATABASE_ANALYSIS.md)
- [AUTO_BACKUP_IMPLEMENTATION.md](AUTO_BACKUP_IMPLEMENTATION.md)
- [BACKGROUND_FEATURE_DOCUMENTATION.md](BACKGROUND_FEATURE_DOCUMENTATION.md)
- [DUAL_TIMELINE_LAYOUT_README.md](DUAL_TIMELINE_LAYOUT_README.md)
- [EXPORT_SYSTEM_DOCUMENTATION.md](EXPORT_SYSTEM_DOCUMENTATION.md)
- [INTEGRATION_GUIDE.md](INTEGRATION_GUIDE.md)
- [KEYBOARD_SHORTCUTS_IMPLEMENTATION.md](KEYBOARD_SHORTCUTS_IMPLEMENTATION.md)
- [LAYOUT_BAR_IMPLEMENTATION.md](LAYOUT_BAR_IMPLEMENTATION.md)
- [SNAPPING_VISUALIZATION_ENHANCEMENT.md](SNAPPING_VISUALIZATION_ENHANCEMENT.md)
- [TEST_VERIFICATION_GUIDE.md](TEST_VERIFICATION_GUIDE.md)
- [UI_MODIFICATIONS_SUMMARY.md](UI_MODIFICATIONS_SUMMARY.md)

## Status

Superseded prototype, kept for history and no longer developed. The design tools, export and project storage work; the timeline is a non-functional scaffold with no keyframes or animation playback, and the AI features depend on external accounts and keys. There are no automated tests. The image adjustment sliders are not applied to the rendered image. Some files are leftovers that nothing imports (`MainPage.tsx`, `Header.tsx`, `LayoutComposer.tsx`, `LoginPage.tsx`, `webglFilters.ts`). For the current editor see [FlashFX-v2](https://github.com/gabrielebolognese/FlashFX-v2).
