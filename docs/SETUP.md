# LAI Monorepo Setup Guide

## Prerequisites

You need:
- **Node.js** 18+ (check: `node --version`)
- **pnpm** 9.0.0+ (check: `pnpm --version`)
- **Rust** 1.70+ (for Tauri desktop app - check: `rustc --version`)

### Install pnpm (if needed)

```bash
npm install -g pnpm@9
```

### Install Rust (if needed)

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env
```

## Quick Start

### 1. Clone or Navigate to Monorepo

```bash
cd /home/nixstation-remote/Projects/lai
```

### 2. Install Dependencies

```bash
pnpm install
```

This will:
- Install all dependencies for both packages
- Create symlinks for internal packages (`@lai/core`)
- Prepare development environment

**Expected output:** No errors, clean lockfile

### 3. Build Packages

```bash
# Build all packages
pnpm build

# Or build specific package
pnpm build:core      # Just core AI engine
pnpm build:lai       # Just desktop app
```

**Expected output:**
- `packages/core/dist/` created with compiled JavaScript
- `packages/lai/dist/` created with bundled assets

### 4. Verify Linking

Check that @lai/core is properly linked:

```bash
ls -la packages/lai/node_modules/@lai/core
```

Should show:
```
packages/lai/node_modules/@lai/core -> ../../../packages/core
```

## Development

### Start Development Mode

**Just @lai/core:**
```bash
pnpm dev:core
```
(Watches TypeScript files for changes)

**Just LAI desktop app:**
```bash
pnpm dev:lai
```
(Starts Vite dev server with hot-reload)

**Both in parallel:**
```bash
pnpm dev
```

### Testing

```bash
# Run all tests
pnpm test

# Watch mode (rerun on changes)
pnpm test:watch

# Coverage report
pnpm test:coverage
```

### Code Quality

```bash
# Lint all code
pnpm lint

# Format with Prettier
pnpm format

# Fix linting issues
pnpm lint:fix
```

## Configuration Files

### Root-Level

| File | Purpose |
|------|---------|
| `package.json` | Workspace scripts, root dependencies |
| `pnpm-workspace.yaml` | Declares packages in workspace |
| `tsconfig.base.json` | Shared TypeScript settings |
| `.gitignore` | Git exclusions |
| `.eslintrc.js` | ESLint rules |
| `.prettierrc.json` | Code formatting rules |

### Per-Package

Each package has its own:
- `package.json` - Dependencies and scripts
- `tsconfig.json` - Package-specific TypeScript config
- `jest.config.js` / `vitest.config.ts` - Test configuration

## Environment Variables

### For @lai/core

Create `packages/core/.env.local`:
```bash
# Optional: test mode
NODE_ENV=development

# Optional: Ollama local server
OLLAMA_BASE_URL=http://localhost:11434
```

### For LAI Desktop App

Create `packages/lai/.env.local`:
```bash
# Tauri dev options
VITE_DEV_SERVER_HOST=localhost
VITE_DEV_SERVER_PORT=5173
```

## Common Tasks

### Adding a New Dependency

To add to @lai/core:
```bash
cd packages/core
pnpm add some-package
```

To add to LAI:
```bash
cd packages/lai
pnpm add some-package
```

To add dev dependency:
```bash
pnpm add --save-dev some-package
```

### Cleaning Up

Remove all build artifacts and dependencies:
```bash
pnpm clean
```

This runs `pnpm -r clean` (removes dist/), then deletes `node_modules/` and `pnpm-lock.yaml`.

### Fresh Install

```bash
pnpm clean
pnpm install
pnpm build
```

## Troubleshooting

### "Cannot find module '@lai/core'"

**Problem:** TypeScript can't resolve @lai/core

**Solution:**
```bash
pnpm install
pnpm build:core
```

Then restart your IDE's TypeScript server.

### "pnpm is not recognized"

**Problem:** pnpm not installed globally

**Solution:**
```bash
npm install -g pnpm@9
```

### "@lai/core/dist/index.js not found"

**Problem:** Core package not built

**Solution:**
```bash
pnpm build:core
```

### "node_modules/@lai/core is not a symlink"

**Problem:** Package wasn't linked correctly

**Solution:**
```bash
rm -rf node_modules pnpm-lock.yaml
pnpm install
```

### Build succeeds but changes don't show up

**Problem:** Stale build artifacts

**Solution:**
```bash
pnpm clean
pnpm build
```

## Next Steps

1. **Review the codebase:** Read `packages/core/src/index.ts` and `packages/lai/src/App.tsx`
2. **Run tests:** `pnpm test` to ensure everything works
3. **Start developing:** `pnpm dev` to begin making changes
4. **Check docs:** See `ARCHITECTURE.md` for deep dive

## Getting Help

- Check `docs/TROUBLESHOOTING.md` for common issues
- Review `docs/ARCHITECTURE.md` for system design
- Look at test files (`__tests__/`) for usage examples

## Project Status

**Current Phase:** MVP Integration
- ✅ Monorepo set up
- ✅ Packages linked
- [ ] Core fully integrated in LAI
- [ ] Basic conversation UI working
- [ ] Multi-provider switching tested

See `tasks.md` for full project roadmap.
