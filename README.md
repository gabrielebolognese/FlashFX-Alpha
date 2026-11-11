# FlashFX Alpha

The earliest public prototype of [FlashFX](https://flashfx.app), a browser-based motion design editor.

> This repository is kept for history. It was followed by [FlashFX_MVP](https://github.com/gabrielebolognese/FlashFX_MVP), and the current editor is [FlashFX-v2](https://github.com/gabrielebolognese/FlashFX-v2), live at [editor.flashfx.app](https://editor.flashfx.app).

## What is in it

- A design tool for shapes, text and images
- A first timeline
- Export
- Accounts and projects stored in Supabase
- An in-app tutorial

## Stack

React 18, TypeScript, Vite, Tailwind CSS and Supabase (auth, database and storage).

## Running it

```bash
npm install
npm run dev
```

The app needs a Supabase project. Apply the SQL in `supabase/migrations/` and provide the project URL and anon key as Vite environment variables.
