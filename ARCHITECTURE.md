# Architecture Technique - Dashboard CRM "Fondateur"
## Stack Svelte 5 + SvelteKit

> **Document de référence** : Architecture technique du Dashboard CRM mono-utilisateur
> **Version** : 1.0
> **Date** : 2026-01-18
> **Stack** : SvelteKit + TypeScript + shadcn-svelte

---

## 1. Vue d'Ensemble

### Contexte Projet
Dashboard CRM Single Page Application pour fondateur solo permettant le suivi de deals commerciaux avec focus sur volume et valeur. Source de données initiale : CSV (migration SQL future prévue).

### Objectifs Techniques
- ✅ **Time-to-MVP** : Livraison Phase 1 (MVP) en 2-3 jours
- ✅ **Performance** : Bundle ultra-léger (<100kb), chargement <1s
- ✅ **Maintenabilité** : TypeScript strict, architecture modulaire
- ✅ **Scalabilité** : Code structuré pour migration SQL future
- ✅ **Developer Experience** : Réactivité native Svelte, hot reload instantané

### Phases de Développement (PRD)
1. **Phase 1 - MVP** : Upload CSV + KPIs macro + graphique répartition statuts
2. **Phase 2 - V1** : Pipeline pondéré + tableau complet + Top 5 deals + alertes
3. **Phase 3 - V2** : Filtres avancés + export CSV + variations KPIs

---

## 2. Stack Technique Complète

```
┌─────────────────────────────────────────────┐
│       DASHBOARD CRM "FONDATEUR"             │
│          (Stack Svelte 5)                   │
├─────────────────────────────────────────────┤
│ Runtime       : Node.js 20+                 │
│ Framework     : SvelteKit 2.x               │ ← Full-stack framework
│ Language      : TypeScript 5.6              │ ← Strict mode
│ Build Tool    : Vite 5                      │ ← Dev server + bundler
│ Package Mgr   : npm (ou pnpm)               │
├─────────────────────────────────────────────┤
│ UI Framework  : Svelte 5 (runes)            │ ← Réactivité native
│ Components    : shadcn-svelte               │ ← Composants pré-stylés
│ Styling       : Tailwind CSS 3.4            │ ← Utility-first
│ Icons         : lucide-svelte               │ ← 1k+ icons SVG
├─────────────────────────────────────────────┤
│ Data Table    : TanStack Table v8           │ ← Headless table (@tanstack/svelte-table)
│ Charts        : svelte-chartjs + Chart.js   │ ← Graphiques réactifs
│ CSV Parsing   : PapaParse 5.4               │ ← RFC 4180 compliant
│ Date Utils    : date-fns 4.1                │ ← Manipulation dates
├─────────────────────────────────────────────┤
│ State Mgmt    : Svelte 5 Runes              │ ← $state, $derived (pas de store)
│ Routing       : SvelteKit file-based        │ ← src/routes/+page.svelte
│ Type Safety   : TypeScript + svelte-check   │
├─────────────────────────────────────────────┤
│ Tests         : Vitest 2.x                  │ ← Unit tests
│ Tests E2E     : Playwright (Phase V2)       │ ← Tests end-to-end (optionnel)
│ Linting       : ESLint + Prettier           │
│ Git Hooks     : simple-git-hooks (opt.)     │
├─────────────────────────────────────────────┤
│ Deployment    : VPS auto-hébergé            │
│ Container     : Docker + Docker Compose     │
│ Reverse Proxy : nginx                       │
│ Process Mgr   : PM2 (ou Docker restart)     │
│ CI/CD         : GitHub Actions (opt.)       │
└─────────────────────────────────────────────┘
```

### Bundle Size Estimé
- **JavaScript** : ~75-90kb (gzipped)
- **CSS** : ~15-20kb (Tailwind purged + shadcn)
- **Total First Load** : <110kb ⚡

---

## 3. Architecture Projet SvelteKit

### Structure de Dossiers

```
crm-dashboard/
├── src/
│   ├── lib/
│   │   ├── components/
│   │   │   ├── ui/                    # shadcn-svelte components
│   │   │   │   ├── button.svelte
│   │   │   │   ├── card.svelte
│   │   │   │   ├── dropdown-menu.svelte
│   │   │   │   ├── table.svelte
│   │   │   │   └── dialog.svelte
│   │   │   ├── KPICard.svelte         # Widget KPI personnalisé
│   │   │   ├── StatusChart.svelte     # Bar/Donut chart statuts
│   │   │   ├── ProspectsTable.svelte  # TanStack Table wrapper
│   │   │   ├── TopDealsPanel.svelte   # Top 5 deals
│   │   │   ├── FileUpload.svelte      # Drag & drop CSV
│   │   │   └── AlertsBadge.svelte     # Badge overdue tasks
│   │   ├── stores/
│   │   │   └── prospects.svelte.ts    # State global (runes)
│   │   ├── utils/
│   │   │   ├── csvParser.ts           # PapaParse wrapper
│   │   │   ├── kpiCalculations.ts     # Calculs KPIs
│   │   │   ├── probabilities.ts       # Config statuts (10%, 40%, 75%, 100%)
│   │   │   └── cn.ts                  # Tailwind class merger (shadcn)
│   │   └── types/
│   │       └── prospect.ts            # TypeScript interfaces
│   ├── routes/
│   │   ├── +page.svelte               # Dashboard principal (/)
│   │   ├── +layout.svelte             # Layout global (header, nav)
│   │   └── +page.ts                   # Load data (optionnel)
│   ├── app.html                       # HTML template
│   └── app.css                        # Global CSS (Tailwind imports)
├── static/
│   └── favicon.png
├── tests/
│   ├── unit/
│   │   ├── kpiCalculations.test.ts
│   │   └── csvParser.test.ts
│   └── e2e/                           # Playwright (Phase V2)
├── Dockerfile
├── docker-compose.yml
├── nginx.conf                         # Config reverse proxy
├── package.json
├── svelte.config.js                   # SvelteKit config
├── vite.config.ts                     # Vite config
├── tailwind.config.ts                 # Tailwind config
├── tsconfig.json                      # TypeScript config
└── .env.example
```

---

## 4. Configuration des Dépendances

### `package.json`

```json
{
  "name": "crm-dashboard",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite dev --host",
    "build": "vite build",
    "preview": "vite preview",
    "test": "vitest",
    "test:ui": "vitest --ui",
    "check": "svelte-kit sync && svelte-check --tsconfig ./tsconfig.json",
    "check:watch": "svelte-kit sync && svelte-check --tsconfig ./tsconfig.json --watch",
    "lint": "prettier --check . && eslint .",
    "format": "prettier --write ."
  },
  "dependencies": {
    "@tanstack/svelte-table": "^8.20.0",
    "svelte-chartjs": "^3.1.5",
    "chart.js": "^4.4.0",
    "papaparse": "^5.4.1",
    "date-fns": "^4.1.0",
    "lucide-svelte": "^0.462.0",
    "clsx": "^2.1.1",
    "tailwind-merge": "^2.5.0",
    "tailwind-variants": "^0.2.0"
  },
  "devDependencies": {
    "@sveltejs/adapter-node": "^5.2.0",
    "@sveltejs/kit": "^2.7.0",
    "@sveltejs/vite-plugin-svelte": "^4.0.0",
    "@types/papaparse": "^5.3.14",
    "svelte": "^5.0.0",
    "svelte-check": "^4.0.0",
    "typescript": "^5.6.0",
    "vite": "^5.4.0",
    "vitest": "^2.1.0",
    "@vitest/ui": "^2.1.0",
    "tailwindcss": "^3.4.0",
    "autoprefixer": "^10.4.0",
    "postcss": "^8.4.0",
    "eslint": "^9.0.0",
    "eslint-plugin-svelte": "^2.46.0",
    "prettier": "^3.3.0",
    "prettier-plugin-svelte": "^3.2.0",
    "prettier-plugin-tailwindcss": "^0.6.0"
  }
}
```

### Installation shadcn-svelte

```bash
# Après init du projet SvelteKit
npx shadcn-svelte@latest init

# Ajouter les composants nécessaires
npx shadcn-svelte@latest add button card table dropdown-menu dialog badge
```

---

## 5. Modèle de Données TypeScript

### `src/lib/types/prospect.ts`

```typescript
/**
 * Interface principale pour un prospect/deal
 * Source : CSV avec colonnes enrichies et calculs
 */
export interface Prospect {
  // ===== Colonnes brutes CSV =====
  taskName: string;               // "Sophie Martin - TechStart"
  status: ProspectStatus;
  dateCreated: Date;
  dueDate: Date | null;
  startDate: Date | null;
  assignee: string;
  priority: Priority;
  tags: string[];                 // Parsed depuis "SaaS|B2B" → ["SaaS", "B2B"]
  taskContent: string;            // Notes contextuelles
  montantDeal: number;

  // ===== Colonnes enrichies (parsing) =====
  contactName: string;            // "Sophie Martin" (avant tiret)
  companyName: string;            // "TechStart" (après tiret)

  // ===== Calculs métier =====
  probability: number;            // Basé sur status : 0.1 | 0.4 | 0.75 | 1.0
  weightedValue: number;          // montantDeal * probability
  isOverdue: boolean;             // dueDate < today && status !== 'gagné - en cours'
}

/**
 * Statuts possibles d'un prospect
 */
export type ProspectStatus =
  | 'prospect'
  | 'qualifié'
  | 'négociation'
  | 'gagné - en cours'
  | 'à relancer';  // Ajouté (présent dans CSV demo)

/**
 * Niveaux de priorité
 */
export type Priority = 'low' | 'medium' | 'high';

/**
 * KPIs calculés globaux
 */
export interface KPIs {
  totalDeals: number;             // Nombre de deals actifs
  pipelineBrut: number;           // Somme montantDeal (tous statuts sauf perdus)
  pipelinePondere: number;        // Somme weightedValue (CA prévisionnel)
  panierMoyen: number;            // pipelineBrut / totalDeals
  overdueTasks: number;           // Nombre de deals avec isOverdue = true
}

/**
 * Configuration des probabilités par statut
 * (Centralisé pour modification facile)
 */
export const STATUS_PROBABILITIES: Record<ProspectStatus, number> = {
  'prospect': 0.10,
  'qualifié': 0.40,
  'négociation': 0.75,
  'gagné - en cours': 1.00,
  'à relancer': 0.05  // Probabilité faible
};

/**
 * Row CSV brut (avant transformation)
 */
export interface CSVRow {
  'Task Name': string;
  'Status': string;
  'Date Created': string;
  'Due Date': string;
  'Start Date': string;
  'Assignees': string;
  'Priority': string;
  'Tags': string;
  'Task Content': string;
  'Montant Deal': string;
}
```

---

## 6. State Management avec Svelte 5 Runes

### `src/lib/stores/prospects.svelte.ts`

```typescript
/**
 * State global de l'application avec Svelte 5 runes
 * Gestion des prospects, KPIs, filtres
 */
import { calculateKPIs } from '$lib/utils/kpiCalculations';
import { parseCSV } from '$lib/utils/csvParser';
import type { Prospect, KPIs, ProspectStatus, Priority } from '$lib/types/prospect';

// ===== State réactif (équivalent useState React) =====
let prospects = $state<Prospect[]>([]);
let isLoading = $state(false);
let error = $state<string | null>(null);

// ===== Filtres (Phase V2) =====
let statusFilter = $state<ProspectStatus | null>(null);
let priorityFilter = $state<Priority | null>(null);
let searchQuery = $state('');

// ===== KPIs calculés automatiquement (équivalent useMemo) =====
const kpis = $derived<KPIs>(calculateKPIs(prospects));

// ===== Prospects filtrés (Phase V2) =====
const filteredProspects = $derived<Prospect[]>(() => {
  let filtered = prospects;

  if (statusFilter) {
    filtered = filtered.filter(p => p.status === statusFilter);
  }

  if (priorityFilter) {
    filtered = filtered.filter(p => p.priority === priorityFilter);
  }

  if (searchQuery.trim()) {
    const query = searchQuery.toLowerCase();
    filtered = filtered.filter(p =>
      p.contactName.toLowerCase().includes(query) ||
      p.companyName.toLowerCase().includes(query) ||
      p.taskContent.toLowerCase().includes(query)
    );
  }

  return filtered;
}());

// ===== Top 5 deals (Phase V1) =====
const topDeals = $derived<Prospect[]>(
  [...prospects]
    .sort((a, b) => b.weightedValue - a.weightedValue)
    .slice(0, 5)
);

// ===== Actions =====

/**
 * Charge les prospects depuis un fichier CSV
 */
export async function loadFromCSV(file: File): Promise<void> {
  isLoading = true;
  error = null;

  try {
    const parsed = await parseCSV(file);
    prospects = parsed;
  } catch (err) {
    error = err instanceof Error ? err.message : 'Erreur parsing CSV';
    console.error('CSV parsing error:', err);
  } finally {
    isLoading = false;
  }
}

/**
 * Exporte les prospects en CSV (Phase V2)
 */
export function exportToCSV(): void {
  // TODO: Implémenter avec Papa.unparse()
}

/**
 * Reset filtres
 */
export function resetFilters(): void {
  statusFilter = null;
  priorityFilter = null;
  searchQuery = '';
}

// ===== Getters (accès depuis composants) =====
export function getProspects() { return prospects; }
export function getFilteredProspects() { return filteredProspects; }
export function getKPIs() { return kpis; }
export function getTopDeals() { return topDeals; }
export function getLoadingState() { return isLoading; }
export function getError() { return error; }
export function getSearchQuery() { return searchQuery; }
export function setSearchQuery(query: string) { searchQuery = query; }
export function setStatusFilter(status: ProspectStatus | null) { statusFilter = status; }
export function setPriorityFilter(priority: Priority | null) { priorityFilter = priority; }
```

---

## 7. Parsing CSV & Transformations

### `src/lib/utils/csvParser.ts`

```typescript
import Papa from 'papaparse';
import { parseISO, isValid } from 'date-fns';
import { STATUS_PROBABILITIES, type Prospect, type CSVRow, type ProspectStatus } from '$lib/types/prospect';

/**
 * Parse un fichier CSV et retourne les prospects enrichis
 */
export async function parseCSV(file: File): Promise<Prospect[]> {
  return new Promise((resolve, reject) => {
    Papa.parse<CSVRow>(file, {
      header: true,
      dynamicTyping: false, // On gère le typing manuellement
      skipEmptyLines: true,
      transformHeader: (header) => header.trim(),
      complete: (results) => {
        try {
          const prospects = results.data
            .map(transformRow)
            .filter((p): p is Prospect => p !== null);

          resolve(prospects);
        } catch (err) {
          reject(err);
        }
      },
      error: (error) => {
        reject(new Error(`Erreur parsing CSV: ${error.message}`));
      }
    });
  });
}

/**
 * Transforme une row CSV en Prospect enrichi
 */
function transformRow(row: CSVRow): Prospect | null {
  try {
    // Parsing identité (Task Name = "Nom Prénom - Entreprise")
    const [contactName, companyName] = parseTaskName(row['Task Name']);

    // Parsing dates
    const dateCreated = parseDate(row['Date Created']);
    const dueDate = parseDate(row['Due Date']);
    const startDate = parseDate(row['Start Date']);

    // Montant (conversion string → number)
    const montantDeal = parseFloat(row['Montant Deal']) || 0;

    // Status (normalisation)
    const status = normalizeStatus(row['Status']);

    // Priority
    const priority = (row['Priority']?.toLowerCase() || 'medium') as 'low' | 'medium' | 'high';

    // Tags (parsing "A|B|C" → ["A", "B", "C"])
    const tags = row['Tags']
      ? row['Tags'].split('|').map(t => t.trim()).filter(Boolean)
      : [];

    // Calculs
    const probability = STATUS_PROBABILITIES[status];
    const weightedValue = montantDeal * probability;
    const isOverdue = dueDate !== null && dueDate < new Date() && status !== 'gagné - en cours';

    return {
      taskName: row['Task Name'],
      status,
      dateCreated,
      dueDate,
      startDate,
      assignee: row['Assignees'] || '',
      priority,
      tags,
      taskContent: row['Task Content'] || '',
      montantDeal,
      contactName,
      companyName,
      probability,
      weightedValue,
      isOverdue
    };
  } catch (err) {
    console.warn('Erreur transformation row:', row, err);
    return null; // Skip invalid rows
  }
}

/**
 * Parse "Sophie Martin - TechStart" → ["Sophie Martin", "TechStart"]
 */
function parseTaskName(taskName: string): [string, string] {
  const parts = taskName.split('-').map(s => s.trim());
  return [parts[0] || '', parts[1] || ''];
}

/**
 * Parse date string → Date object (null si invalide)
 */
function parseDate(dateStr: string): Date | null {
  if (!dateStr || dateStr.trim() === '') return null;

  const parsed = parseISO(dateStr);
  return isValid(parsed) ? parsed : null;
}

/**
 * Normalise les variations de statuts
 */
function normalizeStatus(status: string): ProspectStatus {
  const normalized = status.toLowerCase().trim();

  if (normalized.includes('gagné') || normalized.includes('gagn')) {
    return 'gagné - en cours';
  }
  if (normalized.includes('qualif')) return 'qualifié';
  if (normalized.includes('négo')) return 'négociation';
  if (normalized.includes('relance')) return 'à relancer';

  return 'prospect'; // Défaut
}
```

### `src/lib/utils/kpiCalculations.ts`

```typescript
import type { Prospect, KPIs } from '$lib/types/prospect';

/**
 * Calcule les KPIs globaux depuis la liste de prospects
 */
export function calculateKPIs(prospects: Prospect[]): KPIs {
  // Filtre deals actifs (exclure "perdus" si statut existe plus tard)
  const activeDeals = prospects.filter(p => p.status !== 'perdu' as any); // Type guard

  const totalDeals = activeDeals.length;

  const pipelineBrut = activeDeals.reduce((sum, p) => sum + p.montantDeal, 0);

  const pipelinePondere = activeDeals.reduce((sum, p) => sum + p.weightedValue, 0);

  const panierMoyen = totalDeals > 0 ? pipelineBrut / totalDeals : 0;

  const overdueTasks = activeDeals.filter(p => p.isOverdue).length;

  return {
    totalDeals,
    pipelineBrut,
    pipelinePondere,
    panierMoyen,
    overdueTasks
  };
}

/**
 * Formate un montant en euros
 */
export function formatCurrency(amount: number): string {
  return new Intl.NumberFormat('fr-FR', {
    style: 'currency',
    currency: 'EUR',
    minimumFractionDigits: 0,
    maximumFractionDigits: 0
  }).format(amount);
}
```

---

## 8. Composants Clés

### `src/routes/+page.svelte` (Dashboard Principal)

```svelte
<script lang="ts">
  import { getProspects, getKPIs, getTopDeals, loadFromCSV, getLoadingState, getError } from '$lib/stores/prospects.svelte';
  import KPICard from '$lib/components/KPICard.svelte';
  import StatusChart from '$lib/components/StatusChart.svelte';
  import ProspectsTable from '$lib/components/ProspectsTable.svelte';
  import TopDealsPanel from '$lib/components/TopDealsPanel.svelte';
  import FileUpload from '$lib/components/FileUpload.svelte';
  import { formatCurrency } from '$lib/utils/kpiCalculations';

  const prospects = getProspects();
  const kpis = getKPIs();
  const topDeals = getTopDeals();
  const isLoading = getLoadingState();
  const error = getError();

  async function handleFileUpload(file: File) {
    await loadFromCSV(file);
  }
</script>

<div class="container mx-auto p-6 space-y-6">
  <header>
    <h1 class="text-3xl font-bold">Dashboard CRM Fondateur</h1>
    <p class="text-muted-foreground">Suivi commercial - Volume & Valeur</p>
  </header>

  <!-- Upload CSV -->
  <FileUpload on:upload={(e) => handleFileUpload(e.detail)} />

  {#if error}
    <div class="bg-destructive/10 text-destructive p-4 rounded">
      Erreur: {error}
    </div>
  {/if}

  {#if prospects.length > 0}
    <!-- Phase 1: KPIs Macro -->
    <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
      <KPICard
        title="Volume Pipeline"
        value={kpis.totalDeals}
        subtitle="deals actifs"
      />
      <KPICard
        title="CA Total Brut"
        value={formatCurrency(kpis.pipelineBrut)}
        subtitle="sans pondération"
      />
      <KPICard
        title="Panier Moyen"
        value={formatCurrency(kpis.panierMoyen)}
        subtitle="par deal"
      />
    </div>

    <!-- Phase 1: Graphique répartition -->
    <StatusChart {prospects} />

    <!-- Phase 2: Top 5 Deals -->
    <TopDealsPanel deals={topDeals} />

    <!-- Phase 2: Tableau complet -->
    <ProspectsTable {prospects} />
  {:else if !isLoading}
    <div class="text-center text-muted-foreground py-12">
      Aucun prospect chargé. Uploadez un fichier CSV pour commencer.
    </div>
  {/if}

  {#if isLoading}
    <div class="text-center py-12">Chargement...</div>
  {/if}
</div>
```

### `src/lib/components/StatusChart.svelte` (Phase 1)

```svelte
<script lang="ts">
  import { Bar } from 'svelte-chartjs';
  import { Chart, BarElement, CategoryScale, LinearScale, Tooltip, Legend } from 'chart.js';
  import type { Prospect } from '$lib/types/prospect';
  import { Card } from '$lib/components/ui/card';

  Chart.register(BarElement, CategoryScale, LinearScale, Tooltip, Legend);

  export let prospects: Prospect[];

  // Calcul répartition par statut
  const statusCounts = $derived(() => {
    const counts = new Map<string, number>();
    prospects.forEach(p => {
      counts.set(p.status, (counts.get(p.status) || 0) + 1);
    });
    return counts;
  });

  const chartData = $derived({
    labels: Array.from(statusCounts().keys()),
    datasets: [{
      label: 'Nombre de deals',
      data: Array.from(statusCounts().values()),
      backgroundColor: [
        'rgba(59, 130, 246, 0.8)',  // Blue
        'rgba(34, 197, 94, 0.8)',   // Green
        'rgba(251, 146, 60, 0.8)',  // Orange
        'rgba(168, 85, 247, 0.8)'   // Purple
      ]
    }]
  });

  const options = {
    responsive: true,
    maintainAspectRatio: false,
    scales: {
      y: { beginAtZero: true, ticks: { stepSize: 1 } }
    }
  };
</script>

<Card class="p-6">
  <h2 class="text-xl font-semibold mb-4">Répartition des Deals par Statut</h2>
  <div class="h-64">
    <Bar data={chartData} {options} />
  </div>
</Card>
```

---

## 9. Déploiement VPS (Auto-Hébergé)

### `Dockerfile`

```dockerfile
FROM node:20-alpine AS builder

WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm ci

# Copy source
COPY . .

# Build SvelteKit app
RUN npm run build

# Production image
FROM node:20-alpine

WORKDIR /app

# Copy built app
COPY --from=builder /app/build ./build
COPY --from=builder /app/package*.json ./

# Install production deps only
RUN npm ci --omit=dev

ENV NODE_ENV=production
ENV PORT=3000

EXPOSE 3000

CMD ["node", "build"]
```

### `docker-compose.yml`

```yaml
version: '3.8'

services:
  crm-dashboard:
    build: .
    container_name: crm-dashboard
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - ORIGIN=https://crm.votredomaine.com  # Remplacer
    networks:
      - crm-network

  nginx:
    image: nginx:alpine
    container_name: crm-nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./ssl:/etc/nginx/ssl:ro  # Certificats SSL
    depends_on:
      - crm-dashboard
    networks:
      - crm-network

networks:
  crm-network:
    driver: bridge
```

### `nginx.conf`

```nginx
events {
    worker_connections 1024;
}

http {
    upstream sveltekit {
        server crm-dashboard:3000;
    }

    server {
        listen 80;
        server_name crm.votredomaine.com;  # Remplacer

        # Redirect HTTP → HTTPS
        return 301 https://$host$request_uri;
    }

    server {
        listen 443 ssl http2;
        server_name crm.votredomaine.com;

        ssl_certificate /etc/nginx/ssl/fullchain.pem;
        ssl_certificate_key /etc/nginx/ssl/privkey.pem;

        # Security headers
        add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
        add_header X-Frame-Options "SAMEORIGIN" always;
        add_header X-Content-Type-Options "nosniff" always;

        location / {
            proxy_pass http://sveltekit;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_cache_bypass $http_upgrade;
        }
    }
}
```

### Instructions de Déploiement VPS

```bash
# Sur le VPS (Ubuntu/Debian)

# 1. Installer Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# 2. Cloner le repo
git clone <votre-repo> /opt/crm-dashboard
cd /opt/crm-dashboard

# 3. Configurer SSL (Let's Encrypt)
sudo apt install certbot
sudo certbot certonly --standalone -d crm.votredomaine.com
sudo cp /etc/letsencrypt/live/crm.votredomaine.com/*.pem ./ssl/

# 4. Build & Run
docker-compose up -d --build

# 5. Vérifier logs
docker-compose logs -f crm-dashboard

# 6. Auto-renouvellement SSL (cron)
sudo crontab -e
# Ajouter: 0 3 * * * certbot renew --quiet && docker-compose restart nginx
```

---

## 10. Tests avec Vitest

### `vitest.config.ts`

```typescript
import { defineConfig } from 'vitest/config';
import { svelte } from '@sveltejs/vite-plugin-svelte';

export default defineConfig({
  plugins: [svelte({ hot: !process.env.VITEST })],
  test: {
    globals: true,
    environment: 'jsdom'
  }
});
```

### `tests/unit/kpiCalculations.test.ts`

```typescript
import { describe, it, expect } from 'vitest';
import { calculateKPIs } from '$lib/utils/kpiCalculations';
import type { Prospect } from '$lib/types/prospect';

describe('calculateKPIs', () => {
  it('calcule correctement le pipeline brut', () => {
    const prospects: Prospect[] = [
      { montantDeal: 1000, weightedValue: 100, isOverdue: false } as Prospect,
      { montantDeal: 2000, weightedValue: 800, isOverdue: false } as Prospect
    ];

    const kpis = calculateKPIs(prospects);

    expect(kpis.pipelineBrut).toBe(3000);
    expect(kpis.totalDeals).toBe(2);
    expect(kpis.panierMoyen).toBe(1500);
  });

  it('compte les tâches overdue', () => {
    const prospects: Prospect[] = [
      { isOverdue: true } as Prospect,
      { isOverdue: false } as Prospect,
      { isOverdue: true } as Prospect
    ];

    const kpis = calculateKPIs(prospects);
    expect(kpis.overdueTasks).toBe(2);
  });
});
```

---

## 11. Configuration SvelteKit

### `svelte.config.js`

```javascript
import adapter from '@sveltejs/adapter-node';
import { vitePreprocess } from '@sveltejs/vite-plugin-svelte';

/** @type {import('@sveltejs/kit').Config} */
const config = {
  preprocess: vitePreprocess(),

  kit: {
    adapter: adapter({
      out: 'build',
      precompress: true,
      envPrefix: ''
    }),
    alias: {
      '$lib': './src/lib',
      '$lib/*': './src/lib/*'
    }
  }
};

export default config;
```

### `tailwind.config.ts`

```typescript
import type { Config } from 'tailwindcss';

export default {
  darkMode: ['class'],
  content: ['./src/**/*.{html,js,svelte,ts}'],
  theme: {
    extend: {
      colors: {
        border: 'hsl(var(--border))',
        background: 'hsl(var(--background))',
        foreground: 'hsl(var(--foreground))',
        // shadcn-svelte colors
      }
    }
  },
  plugins: []
} satisfies Config;
```

---

## 12. Migration SQL Future (Préparation)

### Architecture Évolutive

**Phase actuelle (V1-V2)** : Fichier CSV → Parsing côté client → State Svelte

**Phase future (V3)** : Base de données SQL → API REST → SvelteKit endpoints

### Changements Minimaux Requis

**1. Ajouter un backend SvelteKit** :

```typescript
// src/routes/api/prospects/+server.ts
import type { RequestHandler } from './$types';
import { json } from '@sveltejs/kit';
// import { db } from '$lib/server/database'; // Prisma, Drizzle, ou autre

export const GET: RequestHandler = async () => {
  // const prospects = await db.prospect.findMany();
  const prospects = []; // Placeholder
  return json(prospects);
};

export const POST: RequestHandler = async ({ request }) => {
  const data = await request.json();
  // await db.prospect.create({ data });
  return json({ success: true });
};
```

**2. Modifier le store** :

```typescript
// src/lib/stores/prospects.svelte.ts (V3 - SQL)
export async function loadFromAPI(): Promise<void> {
  isLoading = true;
  try {
    const res = await fetch('/api/prospects');
    prospects = await res.json();
  } catch (err) {
    error = 'Erreur chargement API';
  } finally {
    isLoading = false;
  }
}
```

**3. Composants inchangés** : Les composants consomment toujours `getProspects()`, fonctionnement identique !

---

## 13. Checklist de Production

### Avant Déploiement

- [ ] Tests unitaires passent (`npm run test`)
- [ ] Build successful (`npm run build`)
- [ ] Variables d'environnement configurées (`.env.production`)
- [ ] Certificats SSL installés sur VPS
- [ ] nginx configuré avec domaine correct
- [ ] Firewall VPS configuré (ports 80, 443, 22 seulement)
- [ ] Backup automatique configuré (docker volumes)

### Sécurité

- [ ] Pas de secrets dans le code (utiliser `.env`)
- [ ] CSP headers configurés (nginx)
- [ ] Rate limiting API (si migration SQL)
- [ ] HTTPS forcé (redirect 80→443)
- [ ] Logs d'accès nginx configurés

### Performance

- [ ] Bundle size < 120kb (vérifier `npm run build`)
- [ ] Images optimisées (si ajout futur)
- [ ] Compression gzip/brotli activée (nginx)
- [ ] Lighthouse score > 90 (Performance, Accessibility, Best Practices)

---

## 14. Roadmap d'Implémentation

### Phase 1 : Setup & MVP (2-3 jours)

**Jour 1 : Setup projet**
1. `npm create svelte@latest crm-dashboard` (SvelteKit)
2. Install deps : `npm i @tanstack/svelte-table svelte-chartjs chart.js papaparse date-fns lucide-svelte`
3. Setup shadcn-svelte : `npx shadcn-svelte@latest init`
4. Config Tailwind + TypeScript strict

**Jour 2-3 : MVP fonctionnel**
5. Créer types `prospect.ts`
6. Implémenter `csvParser.ts` + `kpiCalculations.ts`
7. Créer store `prospects.svelte.ts` (runes)
8. Composants : `FileUpload`, `KPICard` (x3), `StatusChart`
9. Page `+page.svelte` assemblage

**Livrable** : Upload CSV → Affichage 3 KPIs + graphique

---

### Phase 2 : V1 (3-4 jours)

**Jour 1 : KPI pondéré + TanStack Table**
1. Ajouter probabilités dans `probabilities.ts`
2. Calculer `weightedValue` dans parser
3. Ajouter KPI "CA Prévisionnel" (pipeline pondéré)
4. Setup TanStack Table dans `ProspectsTable.svelte`

**Jour 2 : Top 5 + Alertes**
5. Composant `TopDealsPanel` (tri par weightedValue)
6. Composant `AlertsBadge` (overdue count)
7. Intégrer recherche globale (input + filter state)

**Jour 3 : Drill-down**
8. Modal détails deal (shadcn Dialog)
9. Afficher tous les champs (tags, notes, dates)

**Livrable** : Dashboard complet opérationnel

---

### Phase 3 : V2 (2-3 jours)

**Jour 1 : Filtres**
1. Dropdowns Status/Priority (shadcn DropdownMenu)
2. State filtres dans store
3. Computed `filteredProspects`

**Jour 2 : Export + Polissage**
4. Bouton export CSV (`Papa.unparse()`)
5. Tests unitaires (kpiCalculations, csvParser)
6. Responsive mobile (Tailwind breakpoints)

**Jour 3 : Déploiement**
7. Dockerfile + docker-compose
8. Config nginx + SSL
9. Deploy sur VPS
10. Tests production

**Livrable** : App production-ready

---

## 15. Points d'Attention

### Limitations Connues (V1-V2)

1. **Pas de persistance** : Données perdues au refresh (résolu par CSV re-upload ou localStorage temporaire)
2. **CSV only** : Pas de CRUD manuel avant migration SQL
3. **Single-user** : Pas d'authentification ni multi-utilisateur
4. **Calculs client-side** : OK pour <1000 deals, sinon migrer backend

### Évolutions Futures Possibles

- **Phase V3** : Migration SQL (PostgreSQL + Prisma)
- **Phase V4** : Authentification (Auth.js)
- **Phase V5** : Multi-utilisateur + permissions
- **Phase V6** : API webhooks (intégration CRM externe)
- **Phase V7** : PWA (offline mode avec service worker)

---

## 16. Ressources & Documentation

### Documentation Officielle
- [SvelteKit Docs](https://kit.svelte.dev/docs)
- [Svelte 5 Tutorial](https://svelte.dev/tutorial/svelte/welcome-to-svelte)
- [shadcn-svelte](https://www.shadcn-svelte.com/)
- [TanStack Table Svelte](https://tanstack.com/table/latest/docs/framework/svelte/svelte-table)
- [svelte-chartjs](https://github.com/SauravKanchan/svelte-chartjs)

### Tutoriels Recommandés
- [SvelteKit Full-Stack App](https://joyofcode.xyz/sveltekit-for-beginners)
- [Svelte 5 Runes Explained](https://svelte.dev/blog/runes)
- [Deploying SvelteKit to VPS](https://joyofcode.xyz/sveltekit-deployment)

### Communauté
- [Svelte Discord](https://svelte.dev/chat)
- [Reddit r/sveltejs](https://reddit.com/r/sveltejs)
- Stack Overflow tags : `sveltekit`, `svelte`, `svelte-5`

---

## 17. Contact & Support

Pour questions techniques sur cette architecture :
- Référence PRD : `PRD.md`
- CSV démo : `crm_prospects_demo.csv`
- Ce document : `ARCHITECTURE.md`

**Date de création** : 2026-01-18
**Version** : 1.0
**Statut** : Ready for implementation ✅
