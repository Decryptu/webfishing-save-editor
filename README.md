# webfishing-save-editor

A save editor for [WEBFISHING](https://store.steampowered.com/app/3146520/WEBFISHING/), written in TypeScript with Svelte.

Icons (in `public/`) are original game content and exempt from the project license.

## Local Development

```bash
# Install dependencies
bun install

# Start the dev server (with hot reload)
bun run dev

# Build for production
bun run build
```

## Deployment

### Vercel

Import the repo on Vercel with these settings:

- **Framework Preset:** Vite
- **Build Command:** `npm run build`
- **Output Directory:** `dist`

No extra configuration needed — `base` defaults to `"/"`.

### GitHub Pages

Set the `BASE_PATH` environment variable to your repo subpath:

```bash
BASE_PATH=/webfishing-save-editor/ npm run build
```
