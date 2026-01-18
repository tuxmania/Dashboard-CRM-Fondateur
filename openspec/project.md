# Project Context

## Purpose
Dashboard CRM "Fondateur" - Single Page Application for solo founders to track commercial deals with focus on two axes:
- **Volume**: Number of active deals in the pipeline
- **Value**: Deal amounts and weighted revenue forecasting

The dashboard provides instant business visibility ("météo du business") and operational steering tools to maximize deal closure rates.

### Development Phases
1. **Phase 1 (MVP)**: CSV upload + macro KPIs + status distribution chart (2-3 days)
2. **Phase 2 (V1)**: Weighted pipeline + complete data table + Top 5 deals + alerts (3-4 days)
3. **Phase 3 (V2)**: Advanced filters + CSV export + KPI variations (2-3 days)

## Tech Stack

### Core Framework
- **Runtime**: Node.js 20+
- **Framework**: SvelteKit 2.x (full-stack framework)
- **Language**: TypeScript 5.6 (strict mode)
- **Build Tool**: Vite 5
- **Package Manager**: npm (or pnpm)

### Frontend
- **UI Framework**: Svelte 5 (runes - native reactivity)
- **Components**: shadcn-svelte (pre-styled components)
- **Styling**: Tailwind CSS 3.4 (utility-first)
- **Icons**: lucide-svelte (1k+ SVG icons)

### Data & Visualization
- **Data Table**: TanStack Table v8 (@tanstack/svelte-table - headless table)
- **Charts**: svelte-chartjs + Chart.js (reactive charts)
- **CSV Parsing**: PapaParse 5.4 (RFC 4180 compliant)
- **Date Utils**: date-fns 4.1

### State & Routing
- **State Management**: Svelte 5 Runes ($state, $derived - no stores)
- **Routing**: SvelteKit file-based routing (src/routes/+page.svelte)
- **Type Safety**: TypeScript + svelte-check

### Testing & Quality
- **Unit Tests**: Vitest 2.x
- **E2E Tests**: Playwright (Phase V2, optional)
- **Linting**: ESLint + Prettier
- **Git Hooks**: simple-git-hooks (optional)

### Deployment
- **Platform**: VPS auto-hébergé
- **Container**: Docker + Docker Compose
- **Reverse Proxy**: nginx
- **Process Manager**: PM2 (or Docker restart)
- **CI/CD**: GitHub Actions (optional)

### Bundle Size Target
- **JavaScript**: ~75-90kb (gzipped)
- **CSS**: ~15-20kb (Tailwind purged + shadcn)
- **Total First Load**: <110kb ⚡

## Project Conventions

### Code Style
- **TypeScript**: Strict mode enabled, no implicit any
- **Formatting**: Prettier with prettier-plugin-svelte and prettier-plugin-tailwindcss
- **Naming Conventions**:
  - Components: PascalCase (e.g., `KPICard.svelte`, `ProspectsTable.svelte`)
  - Types/Interfaces: PascalCase (e.g., `Prospect`, `KPIs`)
  - Functions: camelCase (e.g., `calculateKPIs`, `parseCSV`)
  - Files: camelCase for utils (e.g., `csvParser.ts`, `kpiCalculations.ts`)
- **File Extensions**:
  - `.svelte` for components
  - `.svelte.ts` for reactive stores using runes
  - `.ts` for utilities and types
- **Imports**: Use `$lib` alias for cleaner imports from src/lib

### Architecture Patterns

#### Component Structure
```
src/lib/components/
├── ui/              # shadcn-svelte base components
├── KPICard.svelte   # Custom business components
├── StatusChart.svelte
├── ProspectsTable.svelte
└── ...
```

#### State Management (Svelte 5 Runes)
- Use `$state` for reactive variables (replaces `let` in stores)
- Use `$derived` for computed values (replaces `$:` and `derived`)
- Centralize app state in `src/lib/stores/prospects.svelte.ts`
- Export getter/setter functions for controlled access
- No writable/readable stores - use runes exclusively

#### Data Flow
1. **CSV Upload** → `FileUpload.svelte`
2. **Parsing** → `csvParser.ts` (PapaParse + transformations)
3. **State Update** → `prospects.svelte.ts` (central store)
4. **KPI Calculation** → `kpiCalculations.ts` (pure functions)
5. **UI Display** → Components consume via getters

#### Type Safety
- Define all data models in `src/lib/types/prospect.ts`
- Use discriminated unions for status types
- Centralize configuration (e.g., `STATUS_PROBABILITIES` object)
- Validate CSV data during parsing, skip invalid rows

### Testing Strategy

#### Unit Tests (Vitest)
- Test all utility functions (kpiCalculations, csvParser)
- Test data transformations and business logic
- Use `vitest --ui` for interactive testing
- Coverage target: >80% for utils

#### E2E Tests (Playwright - Phase V2)
- Test critical user flows (upload CSV → view dashboard)
- Test interactions (sorting table, filtering)
- Run before production deployment

#### Manual Testing
- Test with sample CSV (`crm_prospects_demo.csv`)
- Verify responsive design on mobile/tablet
- Cross-browser testing (Chrome, Firefox, Safari)

### Git Workflow
- **Main Branch**: `main` (production-ready code)
- **Feature Branches**: `feature/phase-1-mvp`, `feature/top-deals-panel`
- **Commit Messages**: Conventional commits format
  - `feat: add CSV upload component`
  - `fix: handle null dates in parser`
  - `refactor: extract KPI calculations to utility`
- **PR Process**: Optional for solo dev, recommended for major changes

## Domain Context

### CRM Sales Pipeline Terminology
- **Prospect**: Early-stage lead (10% probability)
- **Qualifié**: Qualified lead (40% probability)
- **Négociation**: Active negotiation (75% probability)
- **Gagné - en cours**: Won deal in progress (100% probability)
- **À relancer**: Follow-up needed (5% probability)

### Key Metrics (KPIs)
1. **Volume du Pipeline**: Total number of active deals (excluding lost)
2. **Pipeline Brut**: Sum of all deal amounts without weighting
3. **Pipeline Pondéré (Weighted)**: Sum of (deal amount × status probability) = forecasted revenue
4. **Panier Moyen**: Average deal size (pipeline brut / total deals)
5. **Overdue Tasks**: Deals past due date (excluding won deals)

### Business Rules
- **Probability Weighting**: Each status has a fixed probability of closure
- **Deal Parsing**: Task Name format = "Contact Name - Company Name"
- **Active Deals**: All deals except status "perdu" (if implemented)
- **Overdue Logic**: `dueDate < today AND status !== 'gagné - en cours'`

### Data Source (Current)
- CSV file with columns: Task Name, Status, Date Created, Due Date, Start Date, Assignee, Priority, Tags, Task Content, Montant Deal
- Tags format: pipe-separated (e.g., "SaaS|B2B|Enterprise")
- Dates: ISO 8601 format preferred

## Important Constraints

### Technical Constraints
1. **No Database (V1-V2)**: Data sourced from CSV uploads, no persistence between sessions
2. **Client-Side Processing**: All calculations happen in browser (scalable up to ~1000 deals)
3. **Single User**: No authentication or multi-user support in initial phases
4. **No CRUD Operations**: Cannot add/edit deals manually (CSV is source of truth)

### Architectural Constraints
1. **Migration Path to SQL**: Code must be structured to facilitate future database integration
   - Separate data layer (`stores/prospects.svelte.ts`)
   - Use TypeScript interfaces that match future DB schema
   - Keep business logic pure (functions in `utils/`)
2. **Adapter**: Use `@sveltejs/adapter-node` for VPS deployment
3. **SvelteKit SSR**: Can be disabled if pure SPA needed (set `ssr: false` in config)

### Performance Constraints
- Bundle size must stay <120kb total
- First contentful paint <1s
- Lighthouse score >90 (Performance, Accessibility, Best Practices)

### Deployment Constraints
- Self-hosted on VPS (no Vercel/Netlify)
- Docker containerization required
- HTTPS mandatory (nginx + Let's Encrypt)

## External Dependencies

### Current (V1-V2)
**None** - Fully self-contained application with no external API calls

### Future (V3+)
- **Database**: PostgreSQL + Prisma ORM (planned migration)
- **Authentication**: Auth.js / NextAuth (for multi-user support)
- **External CRM Integration**: Potential webhooks to sync with external CRMs (e.g., HubSpot, Pipedrive)

### Development Dependencies
- All listed in `package.json`
- shadcn-svelte components added via CLI: `npx shadcn-svelte@latest add <component>`

### Infrastructure Dependencies (Production)
- Docker & Docker Compose (container orchestration)
- nginx (reverse proxy, SSL termination)
- Let's Encrypt (SSL certificates)
- PM2 or Docker restart policies (process management)
