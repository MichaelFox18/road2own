# CLAUDE.md

> Operational guide for Claude Code working on HomePath. Read this before making changes. For full project specification, see `README.md`.

---

## Project Summary

HomePath is a Next.js 14+ full-stack web app that helps users plan a path to home ownership. Users enter financial data and goals; the app returns three outputs: (1) what they can afford on their current trajectory, (2) a gameplan to reach their ideal home, (3) the realistic timeline if they change nothing.

**Critical constraint**: MVP runs locally with no auth, but **every architectural decision must preserve the path to production features** (Supabase auth, live rate APIs, email notifications, AI-enhanced advice).

---

## Commands

```bash
pnpm dev            # Start dev server on localhost:3000
pnpm build          # Production build
pnpm start          # Serve production build
pnpm lint           # ESLint
pnpm typecheck      # tsc --noEmit
pnpm test           # Run all tests once
pnpm test:watch     # Tests in watch mode
pnpm test <path>    # Run a specific test file
```

**Before committing anything**: run `pnpm typecheck && pnpm lint && pnpm test`. All three must pass.

---

## Code Style Rules

### TypeScript

- **Strict mode is on.** Do not disable it.
- **No `any` types.** If the type is truly unknown, use `unknown` and narrow.
- **No non-null assertions (`!`) in calculation code.** Validate and throw explicitly.
- Use `type` for unions/primitives, `interface` for object shapes that might extend.
- All exported functions need explicit return types.

### React / Next.js

- Default to **Server Components**. Add `"use client"` only when you need state, effects, or browser APIs.
- Use the **App Router** (`src/app/`), never the Pages Router.
- Forms always use **React Hook Form + Zod**. No manual `useState` form handling.
- No inline styles. Use Tailwind classes.
- Extract components when a file exceeds ~250 lines or a component exceeds ~80 lines.

### Imports

- Use path aliases: `@/components/...`, `@/lib/...`, `@/types`.
- Group imports: (1) external, (2) internal `@/`, (3) relative, with blank lines between.

### Naming

- Components: `PascalCase.tsx`
- Hooks: `useCamelCase.ts`
- Utilities: `camelCase.ts`
- Constants files: `camelCase.ts`, exported values `SCREAMING_SNAKE_CASE`
- Test files: mirror the source file with `.test.ts`

---

## Non-Negotiable Rules

### 1. Financial Math Must Be Tested

**Every function in `src/lib/calculations/` requires unit tests.** No exceptions. Use known-answer test cases derived from authoritative sources (CFPB calculators, standard amortization tables).

When you write a new calculation:
1. Write the test first with a worked example.
2. Implement the function.
3. Verify the test passes.
4. Add edge cases: zero inputs, maximum values, impossible scenarios.

### 2. Never Hardcode API Endpoints in Components

All external data flows through `src/lib/api/`. Components consume repositories and providers via their interfaces. This is what makes the static-to-live-API swap trivial later.

**Wrong**:
```tsx
const rate = await fetch('https://api.fred.stlouisfed.org/...');
```

**Right**:
```tsx
const provider = getRateProvider();
const rate = await provider.getRate('conventional', 30);
```

### 3. Persistence Goes Through the Repository Interface

Do not call `localStorage` directly from components. Route through `ScenarioRepository` so the Supabase swap is painless.

**Wrong**:
```tsx
localStorage.setItem('scenario', JSON.stringify(data));
```

**Right**:
```tsx
const repo = getScenarioRepository();
await repo.save(scenario);
```

### 4. Validate All User Input with Zod

Every form step has a Zod schema in `src/lib/schemas/userInput.ts`. API routes re-validate on the server. Never trust that form validation ran.

### 5. Every Financial Term Gets a Tooltip

DTI, PITI, PMI, MIP, APR, front-end ratio, back-end ratio, amortization — any jargon the user sees needs an `InfoPopover` explaining it in plain language. This is a core UX principle, not optional polish.

### 6. Handle the "Not Achievable" Case Gracefully

Output 3 (timeline with no changes) may reveal that a user cannot reach their goal within 30 years. This must be communicated with honesty **and** empathy. Never use discouraging language. Pivot to Output 2 (the gameplan) as the constructive next step.

### 7. Disclaimers Are Required, Not Optional

The footer disclaimer appears on every page. The results page carries the full-length disclaimer prominently. Rate displays carry the rate-specific disclaimer. See README for exact copy.

---

## File Organization Rules

- **New calculation** → `src/lib/calculations/<name>.ts` + `tests/calculations/<name>.test.ts`
- **New loan type** → Add to `src/lib/constants/loanTypes.ts` with full user-facing description. Update tests.
- **New wizard step** → `src/components/wizard/steps/<Name>Step.tsx` + schema in `src/lib/schemas/userInput.ts` + register in `WizardShell`
- **New chart** → `src/components/results/<Name>Chart.tsx`, use Recharts
- **New API route** → `src/app/api/<name>/route.ts`, validate input with Zod

Do not invent new top-level directories without strong justification.

---

## Common Pitfalls to Avoid

- **Floating-point money math.** Use integers (cents) for intermediate calculations where precision matters, format to currency at display time. Or use a library like `decimal.js` if precision becomes a concern.
- **Assuming 20% down payment.** The user can input any down payment. Calculations must handle 0% (VA/USDA), 3% (conventional), 3.5% (FHA), or anything else.
- **Forgetting PMI thresholds.** PMI applies on conventional loans with <20% down. FHA has MIP (different rules). VA has funding fee instead. Each loan type has distinct insurance logic.
- **Ignoring DTI limits by loan type.** Conventional, FHA, VA, USDA all have different DTI caps. Use the `dtiLimits.ts` constants, don't hardcode.
- **Conflating front-end and back-end DTI.** They are different ratios with different caps. Front-end = housing only; back-end = housing + all debts.
- **Treating closing costs as fixed.** They scale with home price (~3% default). And the user can opt out of including them.
- **Recalculating on every render.** Use `useMemo` for expensive calculations. The results dashboard should not recompute on every mouse move.
- **Skipping ARIA labels on charts.** Recharts is not accessible by default. Add text alternatives.

---

## Testing Guidance

### What to Test

- **Always**: All `src/lib/calculations/` functions. All Zod schemas. All repository implementations.
- **Usually**: Custom hooks with non-trivial logic. API route handlers.
- **Sometimes**: Complex presentational components with conditional rendering logic.
- **Rarely**: Pure layout components, shadcn wrappers.

### Test Structure

```typescript
describe('calculateMonthlyPayment', () => {
  it('computes standard 30-year fixed correctly', () => {
    // $400,000 loan at 6.85% for 30 years = $2,621.00/month
    expect(calculateMonthlyPayment(400_000, 6.85, 30)).toBeCloseTo(2621.00, 2);
  });

  it('handles 0% interest edge case', () => {
    expect(calculateMonthlyPayment(360_000, 0, 30)).toBe(1000);
  });

  it('throws on negative principal', () => {
    expect(() => calculateMonthlyPayment(-1, 6, 30)).toThrow();
  });
});
```

### Known-Answer Sources

Validate test fixtures against CFPB's mortgage calculator, Bankrate's mortgage calculator, or a published amortization schedule. Document the source in a comment.

---

## When to Ask vs. When to Decide

**Decide on your own**:
- File names, variable names, minor component structure
- Which Tailwind classes to use for spacing/layout
- Whether to extract a helper function
- Test case structure

**Ask before acting**:
- Adding a new dependency not in the README tech stack
- Changing a financial calculation formula
- Altering the three-output contract
- Changing loan type definitions or DTI limits
- Adding a new top-level route or API endpoint
- Skipping a disclaimer anywhere

---

## Git Conventions

### Branch Names
`feat/short-description`, `fix/short-description`, `chore/short-description`, `test/short-description`

### Commit Messages
Imperative mood, present tense, lowercase type prefix:

```
feat: add FHA loan type with MIP calculations
fix: correct back-end DTI formula to include new PITI
test: add edge cases for zero-interest amortization
chore: bump Tailwind to v4
docs: update README with Plaid integration notes
```

One logical change per commit. Do not mix refactors with feature work.

---

## Performance Expectations

- Results dashboard should render in under 500ms after inputs submit.
- Wizard step transitions should feel instant (no loading states for step-to-step navigation).
- Charts should not cause layout shift on load.
- No client bundle bloat: audit with `pnpm build` output before adding heavy libraries.

---

## Accessibility Checklist (Run Before Marking Features Done)

- [ ] All form inputs have associated `<label>` elements
- [ ] Color contrast meets WCAG AA
- [ ] Full keyboard navigation works (wizard + dashboard)
- [ ] Screen reader announces step changes in wizard
- [ ] Charts have text alternatives or data tables
- [ ] Focus states are visible and not stripped by Tailwind resets

---

## Progressive Enhancement Philosophy

Build each feature so it **works today in MVP mode** and **enhances cleanly when dependencies are added**:

| Feature | MVP Mode | Enhanced Mode |
|---------|----------|---------------|
| Mortgage rates | Static constants | FRED / MND API |
| Persistence | localStorage | Supabase |
| Gameplan advice | Pure math | Claude API narrative |
| Scenario sharing | Local only | Shareable URLs via auth |
| Progress updates | Manual return | Email notifications |

Never let MVP mode block the enhanced mode, and never let enhanced mode break MVP mode.

---

## What "Done" Looks Like

A feature is done when:

1. TypeScript compiles with no errors
2. ESLint passes with no warnings
3. All tests pass, including new ones for the feature
4. Manual smoke test in the browser succeeds on both desktop and mobile widths
5. Disclaimers, tooltips, and accessibility are in place
6. No `console.log`, `TODO`, or commented-out code left behind
7. README is updated if the feature changes public behavior

---

## If You Get Stuck

1. Re-read the relevant section of `README.md` — the spec is detailed on purpose.
2. Check if a similar pattern exists elsewhere in the codebase and mirror it.
3. Write the test first — it often clarifies the intent.
4. If a decision isn't covered by README or CLAUDE.md, pause and ask.

---

**Build carefully. This app handles decisions that matter to people.**
