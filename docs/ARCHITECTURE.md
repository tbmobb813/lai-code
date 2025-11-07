# LAI Monorepo Architecture

## Overview

The LAI monorepo is a **pnpm workspace** containing multiple interconnected packages for building privacy-first, multi-provider AI applications.

## Packages

### 1. `@lai/core` (packages/core/)

**Purpose:** Core AI engine with multi-provider support and data persistence.

**Key Components:**
- **Providers** - Abstract layer for different AI services (OpenAI, Anthropic, Gemini, Ollama)
- **Storage** - SQLite-based conversation/message persistence with full-text search (FTS5)
- **Context Builder** - Extracts context from files, git history, and workspace
- **Privacy Controls** - Audit logging, optional encryption, data handling rules
- **Streaming** - Provider-agnostic response streaming with buffering

**Responsibilities:**
- ✅ Switch between AI providers seamlessly
- ✅ Persist conversations and search history
- ✅ Build rich context for better prompts
- ✅ Handle streaming responses efficiently
- ✅ Respect user privacy (local-first where possible)

**Dependencies:**
- `better-sqlite3` - Local database
- `node-fetch` - HTTP requests to API providers
- Standard TypeScript ecosystem

### 2. `linux-ai-assistant` (packages/lai/)

**Purpose:** Native Linux desktop application built with Tauri + React.

**Key Components:**
- **Tauri Backend** (Rust) - System integration, file watching, notifications
- **React Frontend** - Conversation UI, settings, provider switching
- **CLI Tool** - Command-line interface with terminal piping support

**Responsibilities:**
- ✅ System tray integration
- ✅ Global keyboard shortcuts
- ✅ File watching and quick-access
- ✅ Conversation UI and management
- ✅ Provider selection and configuration
- ✅ CLI tool for terminal integration

**Dependencies:**
- `@lai/core` - Core AI engine (workspace link)
- `@tauri-apps/*` - Desktop framework
- `react`, `zustand` - UI and state management
- Standard web dev stack (vite, vitest, playwright)

## Monorepo Structure

```
lai/
├── .git                      # Single unified git history
├── packages/
│   ├── core/                # @lai/core
│   │   ├── src/
│   │   │   ├── client.ts           # Main AIClient orchestrator
│   │   │   ├── types.ts            # TypeScript interfaces
│   │   │   ├── providers/          # AI provider implementations
│   │   │   ├── storage/            # Database persistence
│   │   │   ├── context/            # Context extraction
│   │   │   ├── privacy/            # Privacy & security
│   │   │   ├── streaming/          # Response streaming
│   │   │   └── utils/              # Utilities
│   │   ├── dist/            # Built output (tsconfig → tsc)
│   │   ├── package.json
│   │   ├── tsconfig.json
│   │   └── jest.config.js
│   └── lai/                 # linux-ai-assistant
│       ├── src/             # React components
│       ├── src-tauri/       # Rust backend
│       ├── cli/             # CLI tool
│       ├── playwright-e2e/  # E2E tests
│       ├── package.json
│       ├── vite.config.ts
│       └── tauri.conf.json
├── docs/                    # Monorepo documentation
├── package.json             # Root workspace config
├── pnpm-workspace.yaml      # Workspace declaration
├── tsconfig.base.json       # Shared TypeScript config
└── README.md                # Quick start guide
```

## Package Management with pnpm

### Workspace Setup

```yaml
# pnpm-workspace.yaml
packages:
  - 'packages/core'
  - 'packages/lai'

onlyBuiltDependencies:
  - better-sqlite3  # Built from source, not cached
```

### Dependency Resolution

- **Internal dependencies** use `workspace:*` protocol in package.json
  ```json
  {
    "dependencies": {
      "@lai/core": "workspace:*"
    }
  }
  ```
- pnpm automatically creates symlinks to local packages
- Changes in `@lai/core` are immediately reflected in `linux-ai-assistant`
- No need for `npm link` or `npm install` after changes

### Running Commands

```bash
# Install all dependencies
pnpm install

# Build all packages
pnpm build

# Build specific package
pnpm build:core
pnpm build:lai

# Run tests across all packages
pnpm test

# Dev mode for specific package
pnpm dev:core
pnpm dev:lai

# Run command in all packages
pnpm -r test
```

## Build & Compilation Pipeline

### @lai/core Build

```
src/
  ↓ (TypeScript)
tsc (tsconfig.json)
  ↓
dist/
  ├── index.js       # Main exports
  ├── index.d.ts     # TypeScript definitions
  └── ...            # Built modules
```

**Entry point:** `dist/index.js`
**Types:** `dist/index.d.ts`

### linux-ai-assistant Build

```
src/ (React + TypeScript)
  ↓ (Vite)
vite build
  ↓
dist/
  ├── index.html
  ├── assets/        # JS, CSS chunks
  └── ...

src-tauri/ (Rust)
  ↓ (Cargo + Tauri CLI)
tauri build
  ↓
src-tauri/target/release/
  └── linux-ai-assistant (binary)
```

## Development Workflow

### Making Changes to @lai/core

1. Edit files in `packages/core/src/`
2. Run `pnpm build:core` (or `pnpm build` for all)
3. Changes are immediately available to `linux-ai-assistant`
4. No relink needed—pnpm symlink handles it

### Making Changes to LAI

1. Edit files in `packages/lai/src/` or `packages/lai/src-tauri/`
2. Run `pnpm dev:lai` for dev mode (HMR with vite)
3. Or `pnpm build:lai` for production build

### Testing

```bash
# Test everything
pnpm test

# Test in watch mode
pnpm test:watch

# Coverage report
pnpm test:coverage
```

## TypeScript Configuration

### Root tsconfig.base.json

Shared settings for all packages:
- `target`: ES2020
- `module`: ESNext
- `strict`: true (no implicit any)
- `lib`: DOM + ES2020

### Path Aliases

```json
{
  "paths": {
    "@lai/core": ["packages/core/src"],
    "@lai/core/*": ["packages/core/src/*"]
  }
}
```

Allows importing: `import { AIClient } from '@lai/core'`

## Privacy & Security

- **Local-first by design:** Core supports local Ollama models
- **Multi-provider:** Supports cloud (OpenAI, Anthropic, Gemini) and local (Ollama)
- **Encryption optional:** Privacy controls for sensitive conversations
- **Audit logging:** Track what data is sent where

## Scaling & Future Considerations

### Adding UDP (United Dev Platform)

```
packages/
├── core/
├── lai/
└── udp/          # Web/mobile version
    ├── web/      # React web app
    └── mobile/   # React Native app
```

Both `lai` and `udp` can share `@lai/core`:
```json
{
  "dependencies": {
    "@lai/core": "workspace:*"
  }
}
```

### Database Evolution

Current: `better-sqlite3` (sync, single-device)
Future: Add async DB layer (for web/mobile)

### Open-Sourcing @lai/core

The monorepo structure makes it easy to:
1. Publish `@lai/core` to npm independently
2. Keep proprietary UI in `linux-ai-assistant`
3. Allow others to build on top of core

## Performance Considerations

- **Workspace linking:** Zero overhead (symlinks)
- **SQLite:** Fast local queries, FTS5 for search
- **Streaming:** Reduces perceived latency
- **Rust backend:** Efficient system integration

## Troubleshooting

**Q: Changes in @lai/core aren't reflected in LAI**
- Run `pnpm install` again
- Check symlink: `ls -la packages/lai/node_modules/@lai/core`

**Q: Build fails with dependency errors**
- Delete `pnpm-lock.yaml` and `node_modules/`
- Run `pnpm install` again

**Q: Type errors after changing @lai/core**
- Run `pnpm build:core`
- Restart TypeScript server in IDE

## References

- [pnpm Workspaces](https://pnpm.io/workspaces)
- [TypeScript Project References](https://www.typescriptlang.org/docs/handbook/project-references.html)
- [Monorepo Best Practices](https://monorepo.tools/)
