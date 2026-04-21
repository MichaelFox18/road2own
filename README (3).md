# HomePath — Home Ownership Planning Application

> A financial planning tool that helps users map a realistic path to home ownership based on their current financial situation and goals.

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Core User Flow](#core-user-flow)
3. [The Three Outputs](#the-three-outputs)
4. [Tech Stack](#tech-stack)
5. [Project Structure](#project-structure)
6. [User Input Fields](#user-input-fields)
7. [Financial Calculation Specifications](#financial-calculation-specifications)
8. [Loan Types](#loan-types)
9. [Design System](#design-system)
10. [Data Persistence Strategy](#data-persistence-strategy)
11. [API Integrations](#api-integrations)
12. [Environment Variables](#environment-variables)
13. [Setup & Development](#setup--development)
14. [Development Roadmap](#development-roadmap)
15. [Compliance & Disclaimers](#compliance--disclaimers)
16. [Deployment](#deployment)

---

## Project Overview

**HomePath** is a full-stack web application that helps aspiring homeowners understand whether their home ownership goals are achievable within their desired timeline. Users input their **ideal home price** and **ideal purchase timeline**, provide their financial details, and receive three actionable outputs that either confirm they're on track or reveal the gap between their current trajectory and their goal.

### Goals

- **Primary**: Provide accurate, transparent financial analysis for first-time and aspiring homebuyers.
- **Secondary**: Educate users on the home buying process through clear visualizations and loan type explanations.
- **Tertiary**: Serve as a foundation that will scale from a local MVP to a full production app with authentication, live rate data, and email notifications.

### Non-Goals (MVP)

- User authentication (must be architected for, not implemented)
- Actual mortgage brokerage or lending services
- Personalized AI-generated advice (placeholder only)
- Non-US markets

---

## Core User Flow

1. **Landing Page** → Value proposition + "Get Started" CTA
2. **Multi-Step Wizard** (single-page with step indicator):
   - Step 1: Goals (ideal home price, ideal timeline in months/years)
   - Step 2: Loan Preference (toggleable loan types with descriptions)
   - Step 3: Income & Employment
   - Step 4: Savings & Assets
   - Step 5: Debts & Monthly Obligations
   - Step 6: Credit & Location
   - Step 7: Preferences (down payment amount/percent, include closing costs toggle)
3. **Results Dashboard** → Three outputs with rich visualizations
4. **Save Progress** (MVP: local storage; Future: Supabase account)

---

## The Three Outputs

### Output 1: "On-Track" Affordability
**What it shows**: The home price the user can afford within their ideal timeline **with no changes** to their current financial behavior.

**Includes**:
- Recommended home price (with clear "you can afford this today" framing)
- Full PITI breakdown (Principal, Interest, Taxes, Insurance)
- PMI if applicable
- Monthly payment visualization (pie/donut chart)
- Comparison against user's ideal price

### Output 2: "Gameplan" to Reach Ideal Price in Ideal Timeline
**What it shows**: The specific financial adjustments required to hit their ideal home price by their ideal date.

**MVP Logic (Pure Math)**:
- Additional monthly savings needed
- Credit score improvement needed (in points)
- Debt payoff targets (if DTI is blocking)
- Income increase needed (if other levers are exhausted)

**Visualization**:
- Side-by-side "Current Path" vs "Gameplan Path" comparison
- Savings trajectory chart (line graph over time)
- Progress bars for each adjustment lever

> **Note**: The gameplan is deterministic math in the MVP. Architect this as a modular service so it can later be enhanced with AI-generated narrative advice via the Claude API.

### Output 3: Timeline to Ideal Home Price (No Changes)
**What it shows**: Realistically, how long will it take to afford their ideal home if nothing changes?

**Includes**:
- Target date (could be "X years, Y months" or "not achievable within 30 years")
- Handle impossibility gracefully — if their current savings rate + income growth assumption won't reach the goal, state this clearly with a helpful tone
- Timeline comparison chart (ideal vs projected)

### Supporting Visualizations (throughout Results)

- **Affordability breakdown** — donut chart of PITI components
- **Savings trajectory** — line chart showing down payment accumulation over time
- **Timeline comparison** — horizontal bar chart (ideal vs on-track vs no-change)
- **DTI gauge** — visual of current DTI vs lender limits
- **Loan comparison table** — side-by-side of loan types if user wants to explore

---

## Tech Stack

| Layer | Technology | Rationale |
|-------|------------|-----------|
| **Framework** | Next.js 14+ (App Router) | Full-stack in one codebase, API routes, excellent Netlify support, SSR-ready |
| **Language** | TypeScript (strict mode) | Type safety critical for financial math |
| **Styling** | Tailwind CSS | Utility-first, rapid iteration |
| **UI Components** | shadcn/ui | Customizable, warm aesthetic, accessible by default |
| **Icons** | Lucide React | Clean, consistent iconography |
| **Charts** | Recharts | Responsive, React-native, good default aesthetics |
| **Forms** | React Hook Form | Performant, minimal re-renders |
| **Validation** | Zod | Schema-based, integrates with React Hook Form and API routes |
| **State (Wizard)** | Zustand | Lightweight, persists form state across steps |
| **Database** | Supabase (PostgreSQL) | User-selected, includes auth-ready infrastructure |
| **ORM/Client** | Supabase JS Client | Direct integration, minimal overhead |
| **Deployment** | Netlify | User-selected |
| **Testing** | Vitest + React Testing Library | Fast, modern testing |

### Why Not Other Options?

- **Redux** → Overkill for this scope; Zustand covers our needs
- **Prisma** → Unnecessary layer when Supabase client is sufficient for MVP
- **Material UI** → Too opinionated; shadcn/ui gives more control over warm aesthetic
- **Chart.js** → Recharts has better React integration and declarative API

---

## Project Structure

```
homepath/
├── .env.local.example
├── .gitignore
├── README.md
├── next.config.js
├── package.json
├── tailwind.config.ts
├── tsconfig.json
├── components.json                    # shadcn config
│
├── public/
│   ├── favicon.ico
│   └── images/
│
├── src/
│   ├── app/
│   │   ├── layout.tsx                 # Root layout with providers
│   │   ├── page.tsx                   # Landing page
│   │   ├── globals.css
│   │   ├── wizard/
│   │   │   └── page.tsx               # Multi-step form
│   │   ├── results/
│   │   │   └── page.tsx               # Results dashboard
│   │   └── api/
│   │       ├── calculate/
│   │       │   └── route.ts           # Main calculation endpoint
│   │       ├── rates/
│   │       │   └── route.ts           # Mortgage rate endpoint (with placeholder)
│   │       └── scenarios/
│   │           └── route.ts           # Save/load scenarios (future auth-gated)
│   │
│   ├── components/
│   │   ├── ui/                        # shadcn components
│   │   ├── wizard/
│   │   │   ├── WizardShell.tsx
│   │   │   ├── StepIndicator.tsx
│   │   │   ├── steps/
│   │   │   │   ├── GoalsStep.tsx
│   │   │   │   ├── LoanTypeStep.tsx
│   │   │   │   ├── IncomeStep.tsx
│   │   │   │   ├── SavingsStep.tsx
│   │   │   │   ├── DebtsStep.tsx
│   │   │   │   ├── CreditLocationStep.tsx
│   │   │   │   └── PreferencesStep.tsx
│   │   │   └── WizardNavigation.tsx
│   │   ├── results/
│   │   │   ├── ResultsDashboard.tsx
│   │   │   ├── OnTrackCard.tsx        # Output 1
│   │   │   ├── GameplanCard.tsx       # Output 2
│   │   │   ├── TimelineCard.tsx       # Output 3
│   │   │   ├── PITIBreakdown.tsx
│   │   │   ├── SavingsTrajectoryChart.tsx
│   │   │   ├── TimelineComparisonChart.tsx
│   │   │   ├── DTIGauge.tsx
│   │   │   └── LoanComparisonTable.tsx
│   │   ├── shared/
│   │   │   ├── CurrencyInput.tsx
│   │   │   ├── PercentInput.tsx
│   │   │   ├── Tooltip.tsx
│   │   │   ├── InfoPopover.tsx
│   │   │   └── Disclaimer.tsx
│   │   └── layout/
│   │       ├── Header.tsx
│   │       └── Footer.tsx
│   │
│   ├── lib/
│   │   ├── calculations/
│   │   │   ├── mortgage.ts            # PITI, amortization
│   │   │   ├── dti.ts                 # DTI ratio calculations
│   │   │   ├── affordability.ts       # Output 1 logic
│   │   │   ├── gameplan.ts            # Output 2 logic
│   │   │   ├── timeline.ts            # Output 3 logic
│   │   │   ├── closingCosts.ts
│   │   │   ├── pmi.ts
│   │   │   ├── propertyTax.ts
│   │   │   └── insurance.ts
│   │   ├── constants/
│   │   │   ├── loanTypes.ts           # Loan type data + descriptions
│   │   │   ├── stateTaxRates.ts       # Avg property tax by state
│   │   │   ├── dtiLimits.ts
│   │   │   └── defaults.ts            # Default interest rates, etc.
│   │   ├── schemas/
│   │   │   ├── userInput.ts           # Zod schemas
│   │   │   └── results.ts
│   │   ├── supabase/
│   │   │   ├── client.ts              # Browser client
│   │   │   ├── server.ts              # Server client
│   │   │   └── types.ts               # Generated DB types
│   │   ├── api/
│   │   │   ├── rates.ts               # Rate API abstraction (swap providers)
│   │   │   └── index.ts
│   │   └── utils/
│   │       ├── format.ts              # Currency, percent formatters
│   │       └── cn.ts
│   │
│   ├── store/
│   │   └── wizardStore.ts             # Zustand store
│   │
│   └── types/
│       └── index.ts
│
└── tests/
    ├── calculations/
    │   ├── mortgage.test.ts
    │   ├── dti.test.ts
    │   ├── affordability.test.ts
    │   ├── gameplan.test.ts
    │   └── timeline.test.ts
    └── integration/
```

---

## User Input Fields

Keep the form friendly and conversational. **Group inputs logically** and use tooltips (`InfoPopover`) to explain any financial jargon.

### Step 1: Goals
| Field | Type | Notes |
|-------|------|-------|
| Ideal home price | Currency | Required |
| Ideal timeline | Months or Years selector | Required, min 1 month |

### Step 2: Loan Preference
| Field | Type | Notes |
|-------|------|-------|
| Preferred loan type | Radio group with descriptions | See [Loan Types](#loan-types) |
| "Not sure yet" option | Checkbox | If selected, run calcs against Conventional 30-year |

### Step 3: Income & Employment
| Field | Type | Notes |
|-------|------|-------|
| Annual gross income (primary) | Currency | Required |
| Co-borrower annual income | Currency | Optional |
| Employment type | Select (W-2, Self-employed, 1099, Other) | Affects loan qualification logic |
| Years at current job | Number | Optional, used for mortgage readiness |

### Step 4: Savings & Assets
| Field | Type | Notes |
|-------|------|-------|
| Current savings for down payment | Currency | Required |
| Monthly amount saved going forward | Currency | Required — key for trajectory |
| Expected annual income growth | Percent | Default 3%, editable |

### Step 5: Debts & Monthly Obligations
| Field | Type | Notes |
|-------|------|-------|
| Monthly debt payments (total) | Currency | Car, student loans, min credit card payments, etc. |
| Current monthly rent | Currency | Optional, for context |

### Step 6: Credit & Location
| Field | Type | Notes |
|-------|------|-------|
| Credit score | Number OR range selector | Ranges: <580, 580–619, 620–659, 660–699, 700–739, 740–799, 800+ |
| State | Select (US states) | Drives property tax + insurance estimates |
| ZIP code | Text | Optional, for future precision |

### Step 7: Preferences
| Field | Type | Notes |
|-------|------|-------|
| Down payment input mode | Toggle (Dollar / Percent) | User choice |
| Down payment value | Currency or Percent | Based on toggle |
| Include closing costs in calculation | Checkbox | **Default: ON**, user can opt out |
| Loan term | Radio (15-yr, 20-yr, 30-yr) | Only shown for fixed-rate options |

---

## Financial Calculation Specifications

> **Critical**: All financial math must be unit-tested. Create test fixtures in `tests/calculations/` with known input/output pairs.

### Interest Rate (MVP Placeholder)

```typescript
// src/lib/constants/defaults.ts
export const DEFAULT_RATES = {
  conventional30: 6.85,
  conventional15: 6.15,
  fha30: 6.50,
  va30: 6.25,
  usda30: 6.60,
} as const;
```

Structure the rate fetch as an **abstracted service** so plugging in the FRED or Mortgage News Daily API later only requires swapping the implementation:

```typescript
// src/lib/api/rates.ts
export interface RateProvider {
  getRate(loanType: LoanType, termYears: number): Promise<number>;
}

export class StaticRateProvider implements RateProvider { /* MVP */ }
export class FREDRateProvider implements RateProvider { /* Future */ }
```

### Monthly Mortgage Payment (P&I)

Standard amortization formula:

```
M = P × [r(1+r)^n] / [(1+r)^n – 1]

Where:
M = monthly payment
P = principal (home price – down payment)
r = monthly interest rate (annual rate / 12 / 100)
n = number of payments (years × 12)
```

### PITI — Full Monthly Payment

- **P + I**: From amortization formula above
- **Property Tax**: Use state-level average (stored in `stateTaxRates.ts`) as % of home value ÷ 12
  - Sourceable later from ATTOM or county assessor APIs
- **Homeowner's Insurance**: Estimate at 0.35% of home value annually ÷ 12 (MVP constant)
- **PMI** (when down payment < 20% on conventional):
  - Typically 0.5%–1.5% of loan amount annually
  - Use 0.8% as MVP default; scale by credit tier

### DTI (Debt-to-Income) Limits

Use **actual lender standards**:

| Ratio | Conventional | FHA | VA | USDA |
|-------|-------------|-----|-----|------|
| Front-end DTI max | 28% | 31% | N/A (residual income) | 29% |
| Back-end DTI max | 36% (up to 43–45% with compensating factors) | 43% (up to 50% with compensating factors) | 41% (flexible) | 41% |

- **Front-end DTI** = (Projected PITI) / Monthly gross income
- **Back-end DTI** = (Projected PITI + Other monthly debts) / Monthly gross income

Store these in `src/lib/constants/dtiLimits.ts` and reference them in all affordability calculations.

### Closing Costs

- Estimate at **3% of home price** for MVP
- User can opt out via checkbox — if opted out, exclude from required cash-to-close
- Must add to the savings target when calculating affordability

### Output 1 — On-Track Affordability (Algorithm)

```
Given: timeline (months), current savings, monthly savings, income, debts, credit
1. Project total cash available at target date:
   projectedCash = currentSavings + (monthlySavings × timelineMonths)
2. Determine max home price based on:
   a. Down payment constraint: projectedCash / (downPaymentPct + closingCostsPct)
   b. DTI constraint: solve for max PITI that keeps back-end DTI ≤ limit
3. Take the MIN of (a) and (b) — that's the on-track affordable price
4. Return with full PITI breakdown
```

### Output 2 — Gameplan (Algorithm, MVP "Pure Math")

```
Given: ideal home price, ideal timeline, current financials
1. Calculate what the ideal home would require:
   - Cash needed = (idealPrice × downPaymentPct) + closingCosts
   - Max allowed PITI = idealIncome × backEndDTIlimit – currentDebts
2. Identify gaps:
   - Savings gap: requiredCash – projectedCash
   - DTI gap: requiredPITI – currentMaxAllowedPITI
   - Credit gap: if credit < tier needed for best rate on chosen loan
3. Convert gaps into actionable targets:
   - Additional monthly savings = savingsGap / timelineMonths
   - Debt payoff target = amount needed to lower monthly debts enough
   - Income delta needed (if savings/debt levers insufficient)
   - Credit improvement target (tier thresholds: 620, 660, 700, 740)
4. Return structured adjustments object with each lever
```

### Output 3 — Timeline with No Changes (Algorithm)

```
Given: ideal home price, current financials, no changes assumed
1. Simulate month-by-month savings growth (include income growth %)
2. At each month, check if user can afford ideal home:
   - Has enough for down payment + closing costs?
   - Meets DTI ratio at current income?
3. First month where BOTH constraints clear = achievable date
4. If no month within 360 months (30 years) clears → return "not achievable without changes"
5. Handle gracefully — never leave user feeling hopeless
```

---

## Loan Types

Store in `src/lib/constants/loanTypes.ts`. Each must include user-facing description.

### 1. Conventional Fixed-Rate (30-Year)
- **Description**: "The most common mortgage. A fixed interest rate for 30 years means your principal and interest payment never changes. Great for buyers who plan to stay long-term and want predictable payments."
- **Min down payment**: 3% (with PMI until 20% equity)
- **Min credit**: 620 typically
- **PMI required**: Yes, if < 20% down

### 2. Conventional Fixed-Rate (15-Year)
- **Description**: "Same as the 30-year but paid off in half the time. Monthly payments are higher, but you build equity faster and pay significantly less interest overall."
- **Min down payment**: 3%
- **Min credit**: 620

### 3. FHA Loan
- **Description**: "Backed by the Federal Housing Administration, FHA loans are designed for buyers with lower credit scores or smaller down payments. More flexible qualification but requires mortgage insurance for the life of the loan (in most cases)."
- **Min down payment**: 3.5% (with 580+ credit), 10% (with 500–579 credit)
- **Min credit**: 500
- **Mortgage insurance**: MIP required (upfront + annual)

### 4. VA Loan
- **Description**: "Available to eligible active-duty military, veterans, and surviving spouses. No down payment required, no PMI, and competitive rates. One of the best deals in mortgage lending if you qualify."
- **Min down payment**: 0%
- **Min credit**: No official minimum (lenders typically require 620)
- **Eligibility check required**

### 5. USDA Loan
- **Description**: "For homes in eligible rural and some suburban areas. No down payment required and competitive rates for low-to-moderate income buyers. Income and property location restrictions apply."
- **Min down payment**: 0%
- **Min credit**: 640 (typical)
- **Geographic & income restrictions**

> **Note**: Build the loan type system as an extensible enum so ARM, Jumbo, and other types can be added later.

---

## Design System

### Aesthetic
**Warm, friendly, trustworthy** — think Rocket Money or Mint, not Chase or Bank of America. Users are often stressed about finances; the UI should feel supportive, not intimidating.

### Color Palette

```css
/* Suggested palette — adjust as desired */
--background: 36 40% 98%;          /* Warm off-white */
--foreground: 20 14% 15%;          /* Warm charcoal */
--primary: 160 45% 35%;            /* Forest green — growth, trust */
--primary-foreground: 36 40% 98%;
--secondary: 28 60% 55%;           /* Warm terracotta — accent */
--accent: 200 60% 50%;             /* Calm blue — info states */
--success: 145 50% 45%;
--warning: 38 85% 55%;
--destructive: 0 65% 50%;
--muted: 36 20% 92%;
--border: 36 15% 85%;
```

### Typography

- **Headings**: Inter or Geist Sans (friendly geometric sans-serif)
- **Body**: Same family, lighter weight
- **Numbers/Currency**: Tabular figures variant for clean alignment

### Component Guidelines

- Generous whitespace — financial apps feel cramped; don't be.
- Rounded corners (8–12px) throughout — warmer than sharp edges.
- Soft shadows, not harsh ones.
- Micro-animations on progression through the wizard.
- **Every financial term has a tooltip** — DTI, PITI, PMI, MIP, etc.
- Empty states should be encouraging, not blank.
- Error states should never blame the user.

### Responsive

- Mobile-first (many users will plan on their phones)
- Wizard steps should be vertically scrollable, not cramped
- Results dashboard should stack gracefully on mobile

### Accessibility

- All form inputs labeled (not just placeholder)
- Color contrast minimum WCAG AA
- Keyboard navigation for entire wizard
- ARIA labels on charts with text fallbacks

---

## Data Persistence Strategy

### MVP (No Auth)
- Zustand store persists wizard state to `localStorage`
- User can return and pick up where they left off
- Results can be re-computed from stored inputs

### Future (With Auth)
Architect the data layer to easily swap `localStorage` for Supabase now. The wizard store should go through a **repository interface**:

```typescript
// src/lib/supabase/scenarioRepository.ts
export interface ScenarioRepository {
  save(scenario: UserScenario): Promise<string>;
  load(id: string): Promise<UserScenario | null>;
  list(): Promise<UserScenario[]>;
}

export class LocalStorageScenarioRepo implements ScenarioRepository { /* MVP */ }
export class SupabaseScenarioRepo implements ScenarioRepository { /* Future */ }
```

### Future Supabase Schema (Create Migration Files Now, Don't Run Yet)

```sql
-- profiles table (links to auth.users)
create table profiles (
  id uuid references auth.users primary key,
  email text,
  created_at timestamptz default now(),
  notifications_enabled boolean default false
);

-- scenarios table
create table scenarios (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references profiles(id),
  name text,
  inputs jsonb not null,          -- all wizard inputs
  outputs jsonb not null,         -- cached calculation results
  created_at timestamptz default now(),
  updated_at timestamptz default now()
);

-- progress_snapshots table (future — for email progress updates)
create table progress_snapshots (
  id uuid primary key default gen_random_uuid(),
  scenario_id uuid references scenarios(id),
  snapshot_data jsonb not null,
  created_at timestamptz default now()
);

-- RLS policies ensuring users only see their own data
alter table profiles enable row level security;
alter table scenarios enable row level security;
alter table progress_snapshots enable row level security;
```

Place these migrations in `supabase/migrations/` for later application.

---

## API Integrations

### Current (MVP)

All external APIs are **abstracted behind interfaces** so real implementations can be dropped in later.

| Service | Status | Purpose |
|---------|--------|---------|
| Static rate constants | Active | Mortgage rate defaults |

### Future (Architected, Not Active)

| Service | Priority | Purpose | Notes |
|---------|----------|---------|-------|
| **FRED API** | High (free) | Historical mortgage rates (30-yr fixed, 15-yr fixed) | Requires free API key |
| **Mortgage News Daily** | High | Current daily rates | Scrapeable or has feeds |
| **Census Bureau API** | Medium (free) | Median home prices by region | Free, no key needed |
| **ATTOM Data** | Low (paid) | Property tax precision | Paid tier |
| **Plaid** | Medium | Bank account linking for income/savings verification | Free sandbox |
| **Resend / Postmark** | Medium | Email progress notifications | Free tiers available |
| **Claude API** | Low (future) | AI-powered gameplan narrative | Ties into Gameplan Output 2 |

### Rate Endpoint Example

```typescript
// src/app/api/rates/route.ts
import { getRateProvider } from '@/lib/api/rates';

export async function GET(request: Request) {
  const provider = getRateProvider();  // Returns static or live based on env
  const rate = await provider.getRate('conventional', 30);
  return Response.json({ rate, source: provider.name, fetchedAt: new Date() });
}
```

---

## Environment Variables

Create `.env.local.example` with:

```bash
# App
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Supabase (future — leave blank for MVP)
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=

# Rate Providers (future)
FRED_API_KEY=
MORTGAGE_NEWS_DAILY_API_KEY=

# Feature Flags
NEXT_PUBLIC_ENABLE_LIVE_RATES=false
NEXT_PUBLIC_ENABLE_AUTH=false
NEXT_PUBLIC_ENABLE_EMAIL_NOTIFICATIONS=false

# Analytics (future)
NEXT_PUBLIC_POSTHOG_KEY=
```

---

## Setup & Development

### Prerequisites

- Node.js 20+
- npm or pnpm (pnpm recommended)
- Git

### Installation

```bash
# Clone the repository
git clone <repo-url>
cd homepath

# Install dependencies
pnpm install

# Set up environment
cp .env.local.example .env.local

# Run development server
pnpm dev

# Open http://localhost:3000
```

### Development Scripts

```bash
pnpm dev          # Start dev server
pnpm build        # Production build
pnpm start        # Run production build locally
pnpm lint         # Lint
pnpm typecheck    # TypeScript check
pnpm test         # Run tests
pnpm test:watch   # Watch mode
```

### Code Quality Expectations

- All PRs pass typecheck, lint, and tests
- All financial calculation functions have unit tests
- New loan types require tests and UI descriptions
- No `any` types in financial calculation code

---

## Development Roadmap

### Phase 1 — MVP (Current)
- [ ] Project scaffolding (Next.js + TypeScript + Tailwind + shadcn/ui)
- [ ] Landing page
- [ ] 7-step wizard with Zustand state
- [ ] All financial calculations (tested)
- [ ] Results dashboard with all three outputs + visualizations
- [ ] LocalStorage persistence
- [ ] Rate provider abstraction (static)
- [ ] Full disclaimer copy
- [ ] Netlify deployment

### Phase 2 — Live Data
- [ ] FRED API integration
- [ ] Mortgage News Daily integration
- [ ] Census Bureau API for regional price data
- [ ] State-level property tax precision

### Phase 3 — Auth & Persistence
- [ ] Supabase Auth integration
- [ ] Scenario save/load (multiple scenarios per user)
- [ ] Share scenario via link
- [ ] User profile page

### Phase 4 — Notifications
- [ ] Email service integration (Resend)
- [ ] Monthly progress update emails
- [ ] Scenario revision reminders

### Phase 5 — Intelligent Advice
- [ ] Claude API integration for Output 2 narrative enhancement
- [ ] Personalized tips based on user's specific gaps
- [ ] "Ask a question" chat about their scenario

### Phase 6 — Advanced
- [ ] Plaid integration for verified income/savings
- [ ] Pre-approval readiness scoring
- [ ] Real estate listing integration (Zillow/Redfin partnerships)
- [ ] Multi-borrower scenarios
- [ ] Investment property mode

---

## Compliance & Disclaimers

> **Legal note**: This app provides educational estimates only. It is not mortgage advice, lending, or brokerage. Consult licensed professionals.

### Required Disclaimer Placements

1. **Footer (every page)**: Small persistent disclaimer
2. **Results page (prominent)**: Clear notice that results are estimates
3. **Specific outputs**: Contextual disclaimers for interest rates ("rates are estimates and change daily"), tax estimates, insurance estimates

### Disclaimer Copy (use verbatim or adjust with legal review)

**Short (footer)**:
> HomePath provides educational estimates only and is not a licensed mortgage broker, lender, or financial advisor. Consult qualified professionals before making home buying decisions.

**Long (results page)**:
> The calculations shown are estimates based on the information you provided and general market assumptions. Actual mortgage rates, property taxes, insurance premiums, and loan qualifications vary by lender, location, and individual circumstances. HomePath is not a licensed mortgage broker, lender, or financial advisor. This tool is for educational planning purposes only. Before making any home purchase or mortgage decisions, consult with a qualified mortgage professional, financial advisor, and/or real estate attorney.

**Rate-specific tooltip**:
> Interest rates shown are estimates and may not reflect current market conditions. Your actual rate will depend on credit score, loan type, down payment, and current market conditions at the time of lock.

### Data Privacy (Future — With Auth)
- Clear privacy policy before collecting PII
- Never store full credit scores; store tiers instead when possible
- Do not store SSN, bank account numbers, or equivalent
- Comply with applicable state privacy laws (CCPA, etc.)

---

## Deployment

### Netlify (Target)

1. Connect GitHub repo to Netlify
2. Build command: `pnpm build`
3. Publish directory: `.next`
4. Install Netlify's Next.js plugin (auto-detected)
5. Set environment variables in Netlify dashboard
6. Deploy

### Environment-Specific Config

- **Development**: Static rates, localStorage, no auth
- **Staging** (future): Live rates, Supabase staging project, feature flags for auth
- **Production** (future): Live rates, Supabase production, full auth, email notifications

### Pre-Deployment Checklist

- [ ] All environment variables set
- [ ] Build passes locally
- [ ] All tests pass
- [ ] Disclaimers displayed correctly
- [ ] Mobile responsiveness verified
- [ ] Accessibility audit passed
- [ ] Analytics configured (if applicable)

---

## Contributing / AI Agent Guidance

When extending this codebase:

1. **Financial logic changes require tests.** Never modify a calculation without updating or adding tests.
2. **Preserve abstraction layers.** Do not hardcode API calls inside components — route through `src/lib/api/`.
3. **Follow the wizard step pattern** when adding new input fields. Each step is self-contained.
4. **Update the loan types file** when adding new loan products — UI descriptions are as important as the math.
5. **Respect the design system.** Warm, friendly, supportive — never sterile or intimidating.
6. **Every user-facing financial term needs a tooltip.** No exceptions.
7. **Gracefully handle the "not achievable" case** in Output 3. Never discourage.

---

## License

TBD — pending project owner decision.

---

**Built to help people own homes, one honest number at a time.**
