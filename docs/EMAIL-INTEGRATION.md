# Email Integration

## Status

| Feature | Status |
|---------|--------|
| Outlook Add-in | ✅ Working |
| Deep links (`orbit://email?data=...`) | ✅ Working |
| Email tab display | ✅ Working |
| Draft Reply UI | ✅ Working |
| mailto: send | ✅ Working |
| AI generation | ❌ Needs AI configured in Settings |
| Project context for AI | ❌ UI only, not wired up |
| Extract Tasks | ❌ Not implemented |

## Outlook Add-in Setup (macOS)

The add-in is hosted on GitHub Pages. No local server needed.

### Install

1. Download [manifest.xml](../outlook-addin/manifest.xml)
2. Open Outlook → **Settings** (gear) → **View all Outlook settings**
3. Go to **Mail** → **Customize actions** → **Add-ins**
4. Click **Custom add-ins** → **Add from file**
5. Select the downloaded `manifest.xml`

### Use

1. Open any email in Outlook
2. Click **"Open in Orbit"** button in the ribbon
3. Confirm the "Open Orbit?" dialog
4. Email opens in Orbit as a tab

## How It Works

```
Outlook Add-in (GitHub Pages)
    ↓
Extracts email → base64 encodes → opens orbit://email?data={base64}
    ↓
Tauri deep-link plugin receives URL
    ↓
Frontend decodes → opens email tab (session only, not persisted)
    ↓
User clicks "Draft Reply" → AI generates draft → opens mailto:
```

## Add-in Hosting (GitHub Pages)

The add-in files are hosted on a separate public repo via git subtree:

- **Public repo**: https://github.com/v1lling/orbit-outlook-addin
- **GitHub Pages URL**: https://v1lling.github.io/orbit-outlook-addin/
- **Local source**: `outlook-addin/src/` (pushed via subtree)

### Deploy Changes

```bash
# From orbit root - pushes outlook-addin/src to public repo
npm run deploy --prefix outlook-addin
```

This runs: `git subtree split --prefix=outlook-addin/src` → push to `addin-origin` remote.

## Key Files

- `outlook-addin/src/commands.js` - Add-in logic
- `src/lib/email/deep-link.ts` - URL parser
- `src/components/email/email-viewer.tsx` - Email display
- `src/components/email/draft-reply-panel.tsx` - AI draft UI
