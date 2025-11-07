# Contributing to LAI

## Development Philosophy

- **Code clarity over cleverness** - Easy to understand is more important than concise
- **Type safety** - Use TypeScript strictly, no `any` without justification
- **Modular design** - Single responsibility principle for all modules
- **Testing first** - Write tests alongside code, not after
- **Documentation** - Code comments explain *why*, not *what*

## Getting Started

1. **Set up the monorepo:** Follow `SETUP.md`
2. **Understand the architecture:** Read `ARCHITECTURE.md`
3. **Explore the codebase:** Start with `packages/core/src/index.ts`

## Code Structure

### @lai/core (packages/core/)

```
src/
├── client.ts          # Main AIClient class - START HERE
├── types.ts           # TypeScript interfaces
├── providers/         # AI provider implementations
│   ├── base.ts        # Provider interface (read this)
│   ├── anthropic.ts   # Claude implementation
│   ├── openai.ts      # GPT implementation
│   ├── gemini.ts      # Google implementation
│   └── ollama.ts      # Local models
├── storage/           # Database persistence
│   ├── conversations.ts
│   ├── messages.ts
│   ├── search.ts      # Full-text search (FTS5)
│   └── migrations.ts
├── context/           # Context extraction
│   ├── builder.ts     # Main context builder
│   ├── files.ts       # File context
│   └── git.ts         # Git history context
├── privacy/           # Privacy features
│   ├── controller.ts  # Privacy control logic
│   ├── encryption.ts  # Optional encryption
│   └── audit.ts       # Audit logging
├── streaming/         # Response streaming
│   ├── handler.ts     # Main streaming logic
│   └── parser.ts      # Provider-specific parsing
└── __tests__/         # Test files
```

### linux-ai-assistant (packages/lai/)

```
src/
├── App.tsx            # Main React component
├── components/        # Reusable React components
│   ├── ChatPanel.tsx
│   ├── Settings.tsx
│   └── ...
├── lib/               # Utilities and helpers
│   ├── tauriClient.ts # Tauri IPC bridge
│   └── ...
└── __tests__/         # Tests

src-tauri/
├── src/
│   ├── lib.rs         # Rust command handlers
│   ├── main.rs        # Tauri app entry
│   └── ...
└── Cargo.toml

cli/
├── src/               # CLI tool code
├── Cargo.toml
└── ...
```

## Making Changes

### Adding a New AI Provider

1. **Create provider implementation:**
   ```bash
   touch packages/core/src/providers/newprovider.ts
   ```

2. **Implement Provider interface:**
   ```typescript
   import { Provider, ProviderType } from './base';

   export class NewProvider implements Provider {
     type: ProviderType = 'newprovider';

     async complete(options) {
       // Implementation
     }

     async stream(options) {
       // Stream implementation
     }
   }
   ```

3. **Register in ProviderFactory:**
   ```typescript
   // packages/core/src/providers/index.ts
   import { NewProvider } from './newprovider';

   export function createProvider(type: ProviderType) {
     switch (type) {
       case 'newprovider':
         return new NewProvider(config);
       // ...
     }
   }
   ```

4. **Add tests:**
   ```bash
   touch packages/core/src/__tests__/providers/newprovider.test.ts
   ```

5. **Build and test:**
   ```bash
   pnpm build:core
   pnpm test:core
   ```

### Adding a New UI Component

1. **Create component:**
   ```bash
   touch packages/lai/src/components/MyComponent.tsx
   ```

2. **Write component + tests:**
   ```typescript
   // MyComponent.tsx
   export function MyComponent() {
     // ...
   }

   // MyComponent.test.tsx
   import { render } from '@testing-library/react';
   import { MyComponent } from './MyComponent';

   describe('MyComponent', () => {
     it('should render', () => {
       const { container } = render(<MyComponent />);
       expect(container).toBeInTheDocument();
     });
   });
   ```

3. **Test in dev mode:**
   ```bash
   pnpm dev:lai
   # Browser opens with HMR
   ```

### Updating Database Schema

1. **Create migration:**
   ```bash
   # Edit packages/core/src/storage/migrations.ts
   ```

2. **Add migration function:**
   ```typescript
   async function migrateV2(db: Database) {
     db.exec(`ALTER TABLE messages ADD COLUMN newField TEXT`);
   }
   ```

3. **Increment schema version in storage initialization**

4. **Test with fresh database:**
   ```bash
   rm test-conversations.db
   pnpm test:core
   ```

## Testing

### Running Tests

```bash
# All tests
pnpm test

# Watch mode
pnpm test:watch

# Specific package
pnpm test:core
pnpm test:lai

# With coverage
pnpm test:coverage
```

### Writing Tests

**For @lai/core (Jest):**
```typescript
// src/__tests__/providers/openai.test.ts
describe('OpenAIProvider', () => {
  let provider: OpenAIProvider;

  beforeEach(() => {
    provider = new OpenAIProvider({
      apiKey: 'test-key',
      model: 'gpt-4',
    });
  });

  it('should complete prompts', async () => {
    const response = await provider.complete({
      messages: [{ role: 'user', content: 'Hello' }],
    });
    expect(response).toHaveProperty('text');
  });
});
```

**For LAI (Vitest + React Testing Library):**
```typescript
// src/__tests__/components/ChatPanel.test.tsx
import { render, screen } from '@testing-library/react';
import { ChatPanel } from '@/components/ChatPanel';

describe('ChatPanel', () => {
  it('should render chat input', () => {
    render(<ChatPanel />);
    expect(screen.getByPlaceholderText(/type a message/i)).toBeInTheDocument();
  });
});
```

## Code Style

### TypeScript

- **Use `const` by default**, `let` when needed
- **No `any` type** (use `unknown` and narrow types)
- **Explicit return types** on functions
- **Interfaces over types** for object shapes
- **Enums for fixed sets** of values

✅ Good:
```typescript
interface Message {
  role: 'user' | 'assistant';
  content: string;
}

async function sendMessage(msg: Message): Promise<string> {
  return msg.content;
}
```

❌ Bad:
```typescript
function sendMessage(msg: any) {
  return msg.content;
}
```

### Formatting

Prettier is configured automatically. Just run:
```bash
pnpm format
```

Or in your IDE, set "format on save" to use Prettier.

### Comments

Write comments that explain **why**, not **what**:

✅ Good:
```typescript
// We cache results to avoid re-querying the database
// for the same conversation within 5 seconds
const cachedResult = cache.get(conversationId);
```

❌ Bad:
```typescript
// Get cached result
const cachedResult = cache.get(conversationId);
```

## Debugging

### Debug @lai/core

1. **Run tests with logging:**
   ```bash
   NODE_DEBUG=* pnpm test:core
   ```

2. **Use TypeScript breakpoints in IDE** (e.g., VSCode)

3. **Check database:**
   ```bash
   sqlite3 test-conversations.db
   ```

### Debug LAI

1. **Tauri dev mode shows Rust + JS errors:**
   ```bash
   pnpm dev:lai
   ```

2. **Open DevTools:** Right-click → Inspect

3. **Check Rust logs:**
   ```bash
   RUST_LOG=debug pnpm dev:lai
   ```

## Pull Request Process

1. **Create feature branch:**
   ```bash
   git checkout -b feat/my-feature
   ```

2. **Make changes and commit:**
   ```bash
   git add .
   git commit -m "feat: add new feature"
   ```

3. **Run tests locally:**
   ```bash
   pnpm test
   pnpm lint
   ```

4. **Push and open PR:**
   ```bash
   git push origin feat/my-feature
   ```

### Commit Message Format

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
feat: add new feature
fix: resolve bug
docs: update documentation
refactor: improve code structure
test: add test coverage
chore: update dependencies
```

## Common Issues

### "Cannot find module '@lai/core'"
```bash
pnpm install
pnpm build:core
```

### "Database is locked"
Multiple tests accessing same DB:
```bash
# Use separate test databases per package
# Or mock database in tests
```

### "TypeScript compilation fails"
```bash
pnpm clean
pnpm install
pnpm build
```

## Performance Guidelines

- **@lai/core:** Keep initialization fast (< 100ms)
- **UI:** Aim for 60fps animations, < 16ms frame time
- **Database queries:** Add indexes for hot queries
- **Streaming:** Buffer responses to avoid excessive updates

## Security Considerations

- **Never commit API keys** (use `.env.local`)
- **Validate all user input** before sending to AI providers
- **Sanitize markdown** before rendering in UI
- **Encrypt sensitive conversations** when requested

## Need Help?

- **Architecture questions:** See `ARCHITECTURE.md`
- **Setup issues:** See `SETUP.md`
- **Code examples:** Check `__tests__/` directories
- **Type errors:** Use `pnpm typecheck` to see all issues

---

Happy coding! 🚀
