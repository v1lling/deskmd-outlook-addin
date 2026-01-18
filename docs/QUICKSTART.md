# Orbit Quick Start Guide

## Prerequisites

1. **Node.js** 18+
2. **Rust** (for Tauri) - [Install](https://www.rust-lang.org/tools/install)
3. **pnpm** (recommended) or npm

## Step 1: Initialize Next.js

```bash
cd /Users/sascha/Development/orbit

# Create Next.js app in current directory
npx create-next-app@latest . --typescript --tailwind --eslint --app --src-dir --import-alias "@/*"
```

When prompted:
- TypeScript? **Yes**
- ESLint? **Yes**
- Tailwind CSS? **Yes**
- `src/` directory? **Yes**
- App Router? **Yes**
- Import alias? **Yes** → `@/*`

## Step 2: Install Dependencies

```bash
# Core
npm install zustand @tanstack/react-query gray-matter

# UI utilities
npm install clsx tailwind-merge lucide-react date-fns

# Drag and drop (for kanban)
npm install @dnd-kit/core @dnd-kit/sortable @dnd-kit/utilities

# Dev
npm install -D @types/node
```

## Step 3: Set up shadcn/ui

```bash
npx shadcn@latest init
```

When prompted:
- Style: **Default**
- Base color: **Slate**
- CSS variables: **Yes**

Add initial components:
```bash
npx shadcn@latest add button card dialog dropdown-menu input label separator tabs scroll-area
```

## Step 4: Add Tauri

```bash
# Install Tauri CLI
npm install -D @tauri-apps/cli @tauri-apps/api

# Initialize Tauri
npx tauri init
```

When prompted:
- App name: **Orbit**
- Window title: **Orbit**
- Web assets path: **../out**
- Dev server URL: **http://localhost:3000**
- Dev command: **npm run dev**
- Build command: **npm run build**

## Step 5: Configure Next.js for Tauri

Update `next.config.js`:
```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  output: 'export',
  images: {
    unoptimized: true,
  },
}

module.exports = nextConfig
```

Update `package.json` scripts:
```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "tauri": "tauri",
    "tauri:dev": "tauri dev",
    "tauri:build": "tauri build"
  }
}
```

## Step 6: Create Directory Structure

```bash
# Components
mkdir -p src/components/{ui,layout,kanban,tasks,projects,notes,context}

# Libraries
mkdir -p src/lib/orbit

# State
mkdir -p src/stores

# Types
mkdir -p src/types
```

## Step 7: Verify Setup

```bash
# Run web dev server
npm run dev
# → Should open http://localhost:3000

# Run desktop app (in another terminal)
npm run tauri:dev
# → Should open Orbit desktop window
```

## Project Structure After Setup

```
orbit/
├── src/
│   ├── app/                    # Next.js pages
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   └── globals.css
│   ├── components/
│   │   ├── ui/                 # shadcn components
│   │   ├── layout/             # Sidebar, header
│   │   ├── kanban/             # Board, column, card
│   │   ├── tasks/              # Task forms
│   │   ├── projects/           # Project components
│   │   ├── notes/              # Note editor
│   │   └── context/            # File browser
│   ├── lib/
│   │   └── orbit/              # File operations
│   ├── stores/                 # Zustand stores
│   └── types/                  # TypeScript types
├── src-tauri/                  # Tauri (Rust)
├── docs/                       # Documentation
├── public/
├── package.json
├── tsconfig.json
├── tailwind.config.ts
├── next.config.js
└── CLAUDE.md
```

## Data Directory

Orbit stores data in `~/Orbit/`:

```
~/Orbit/
├── config.json
└── areas/
    └── {your-area}/
        ├── area.md
        └── projects/
            └── {your-project}/
                ├── project.md
                ├── tasks/
                ├── notes/
                └── context/
```

The app will create this structure on first run.

## Next Steps

1. Read [PHASES.md](./PHASES.md) for implementation roadmap
2. Start with Phase 1: Foundation
3. Build the layout and area switching first

## Useful Commands

```bash
# Development
npm run dev              # Web only
npm run tauri:dev        # Desktop app

# Add shadcn component
npx shadcn@latest add [component]

# Type check
npx tsc --noEmit

# Build
npm run build            # Web build
npm run tauri:build      # Desktop build (creates .app/.exe)
```

## Troubleshooting

### Tauri not starting
```bash
# Check Rust is installed
rustc --version

# Reinstall Tauri CLI
npm install -D @tauri-apps/cli
```

### Next.js build errors
- Ensure all pages use `'use client'` for client components
- No `getServerSideProps` - use API routes instead
- Check `output: 'export'` is set in next.config.js

### File system access
- Tauri sandboxes file access by default
- For development, full access is enabled
- Production builds need proper capability configuration
