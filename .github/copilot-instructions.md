# Copilot Instructions for Soc Ops

## Project Overview

**Soc Ops** is an interactive **Social Bingo game** for in-person mixers. The core mechanic: find people matching icebreaker questions on a 5×5 grid to mark squares and win with 5 in a row (horizontal, vertical, or diagonal).

**Tech Stack:**
- React 19 + TypeScript
- Vite (build) + Vitest (testing)
- Tailwind CSS v4
- GitHub Pages deployment (auto-deploy from `main`)

**Key Constraint:** The center square is always a FREE SPACE (index 12), automatically marked.

---

## Architecture

### Component Hierarchy

```
App (central orchestrator)
├── StartScreen (initial game state)
├── GameScreen (active game mode)
│   └── BingoBoard
│       └── BingoSquare[] (individual squares)
└── BingoModal (win celebration overlay)
```

**Data Flow:**
- `useBingoGame` hook manages all game state (board, marks, win detection) + localStorage persistence
- Components are stateless; all mutations go through hook callbacks
- `bingoLogic.ts` contains pure utility functions (board generation, win checking)

### State Management Patterns

**Game State Values:**
- `'start'` → StartScreen shown
- `'playing'` → GameScreen shown
- `'bingo'` → BingoModal shown (overlays GameScreen)

**Board Persistence:**
- Stored in localStorage under key `bingo-game-state` with version validation
- Protects against version mismatches via `validateStoredData()` type guard
- On invalid data, silently resets to generate new board

**Winning Detection:**
- `checkBingo()` checks all 12 lines (5 rows, 5 cols, 2 diagonals)
- `getWinningSquareIds()` extracts square IDs in winning line
- Modal automatically shows on bingo; dismiss resets game

---

## Developer Workflows

### Build & Serve
```bash
npm run dev        # Vite dev server (auto-reload)
npm run build      # TypeScript + Vite production build
npm run test       # Vitest (single run, not watch)
npm run lint       # ESLint (configured for React + TypeScript)
```

### Environment
- **Node.js 22+** required
- Vite auto-detects `VITE_REPO_NAME` env var for GitHub Pages base path (set in CI/CD)
- `.devcontainer/` available for reproducible dev environment

### Testing
- **Test Framework:** Vitest (config in `vite.config.ts`)
- **Setup:** `src/test/setup.ts` initializes jest-dom matchers + React cleanup
- **Test File Convention:** `*.test.ts` (e.g., `bingoLogic.test.ts`)
- **Coverage Gap:** Only `bingoLogic.ts` tested; components lack unit tests

---

## Project-Specific Conventions

### File Organization
```
src/
  types/          # Domain types (BingoSquareData, GameState, BingoLine)
  utils/          # Pure functions (generateBoard, toggleSquare, checkBingo)
  data/           # Static content (questions array, FREE_SPACE constant)
  hooks/          # Custom React hooks (useBingoGame)
  components/     # UI components (GameScreen, BingoBoard, etc.)
  test/           # Shared test setup
```

### Type Safety
- All game domain types defined in `types/index.ts`
- `BingoSquareData` includes: `id`, `text`, `isMarked`, `isFreeSpace`
- `BingoLine` captures winning configuration: `type`, `index`, `squares[]`
- Use strict null checks; avoid optional chaining except in UI fallbacks

### Styling
- **Tailwind v4** via `@tailwindcss/vite` plugin
- **No custom CSS file** — all styling via utility classes
- Grid layout for board: `grid grid-cols-5 gap-1 aspect-square`
- Follow design patterns in `components/` for spacing/colors

### Questions Data
- Located in `src/data/questions.ts` (24 icebreaker prompts)
- Free space hardcoded as `"FREE SPACE"`
- Board generation **shuffles questions** using Fisher-Yates, ensuring randomness

---

## Critical Integration Points

### Board Lifecycle
1. **Generation:** `generateBoard()` picks 24 shuffled questions, injects FREE SPACE at center (index 12)
2. **Mutation:** `toggleSquare()` immutably flips `isMarked` for non-free squares
3. **Persistence:** Board serialized to localStorage after each click
4. **Win Check:** `checkBingo()` validates all lines on each state change

### localStorage Serialization
- Stored object: `{ version, gameState, board, winningLine }`
- Validator ensures data shape + version match before hydration
- Mismatch = discarded; fresh board generated on next load

### Component Communication
- **Props-only:** No Context API, no global state
- Parent passes `board` + `onSquareClick` to children
- Modal controlled via `showBingoModal` boolean + `dismissModal()` callback

---

## Testing Strategy

### Existing Tests
- `bingoLogic.test.ts` covers 80+ test cases for pure utility functions
- Tests use Vitest globals: `describe`, `it`, `expect`
- Includes randomization testing via `vi.spyOn(Math.random, ...)`

### Writing New Tests
- Unit test new utility functions in `src/utils/`
- Component tests should use `@testing-library/react` (see setup.ts)
- Mock localStorage in tests: `vi.spyOn(window.localStorage, 'getItem')`
- Always call `cleanup()` in teardown (auto-handled by setup.ts)

---

## Common Task Patterns

### Adding New Questions
1. Edit `src/data/questions.ts` (must be 24+ items)
2. No code changes needed; board generation auto-shuffles

### Modifying Game Rules
- **Win condition:** Edit `checkBingo()` logic in `bingoLogic.ts`
- **Board size:** Update `BOARD_SIZE` + `CENTER_INDEX` constants
- **Free space behavior:** Only modify FREE_SPACE related logic (center index 12)

### Styling Updates
- Edit Tailwind classes directly in component files
- No CSS imports needed; `@tailwindcss/vite` processes utilities
- Reference existing color/spacing patterns in GameScreen + BingoBoard

### Bug Fixes
- Start with `bingoLogic.test.ts` to understand expected behavior
- Add test for bug scenario before fixing
- Verify fix doesn't break persistence (test localStorage round-trip)

---

## Deployment & CI/CD

- **Auto-deploy:** Push to `main` triggers GitHub Actions
- **Environment:** `VITE_REPO_NAME` passed to build (sets base path for non-root repos)
- **Output:** Built to `dist/`, deployed via GitHub Pages
- Build command: `npm run build` (runs `tsc -b && vite build`)

---

## Key Files Reference

| File | Purpose |
|------|---------|
| `src/hooks/useBingoGame.ts` | Game state + actions; localStorage persistence |
| `src/utils/bingoLogic.ts` | Board generation, square toggling, win detection |
| `src/types/index.ts` | All domain type definitions |
| `src/data/questions.ts` | Icebreaker questions pool |
| `src/components/GameScreen.tsx` | Main game layout |
| `src/components/BingoBoard.tsx` | 5×5 grid renderer |
| `vite.config.ts` | Vite + Vitest setup |
