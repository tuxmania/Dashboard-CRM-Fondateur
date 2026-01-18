# Change: Implémenter le MVP du Dashboard CRM (Phase 1)

## Why

Le fondateur a besoin d'une vue macro immédiate de son pipeline commercial pour piloter deux axes critiques : le **volume** de deals actifs et la **valeur** totale du pipeline. Actuellement, aucune application n'existe - c'est le point de départ du projet.

Le MVP permet de valider rapidement l'approche technique (SvelteKit + CSV client-side) avant d'investir dans des features avancées (V1, V2) et une migration SQL future.

## What Changes

Cette change introduit l'infrastructure de base du Dashboard CRM et les fonctionnalités du MVP ("La météo du business") :

**Nouvelle infrastructure :**
- Projet SvelteKit 2.x initialisé avec TypeScript strict
- Configuration Tailwind CSS + shadcn-svelte
- Structure de dossiers selon ARCHITECTURE.md (src/lib/types, utils, components, stores)
- Configuration Docker + nginx pour déploiement VPS

**Nouvelles capabilities :**
- **csv-data-ingestion** : Upload et parsing de fichiers CSV avec transformation des données (parsing Task Name, typage dates/montants, normalisation statuts)
- **kpi-calculation** : Calcul des 3 KPIs macro (Volume pipeline, Pipeline brut, Panier moyen)
- **dashboard-visualization** : Affichage des KPIs dans des cards + graphique de répartition des deals par statut (Bar Chart)

**Contraintes respectées :**
- Pas de base de données (CSV uniquement)
- Pas de persistance entre sessions (rechargement = re-upload)
- Bundle size <110kb
- Code structuré pour migration SQL future (séparation data layer / business logic / UI)

## Impact

**Affected specs:**
- `csv-data-ingestion` (ADDED - nouveau)
- `kpi-calculation` (ADDED - nouveau)
- `dashboard-visualization` (ADDED - nouveau)

**Affected code:**
- Création complète du projet SvelteKit (src/, static/, package.json, configs)
- Fichiers clés :
  - `src/lib/types/prospect.ts` : Interfaces TypeScript
  - `src/lib/utils/csvParser.ts` : Parsing CSV avec PapaParse
  - `src/lib/utils/kpiCalculations.ts` : Calculs KPIs
  - `src/lib/utils/probabilities.ts` : Configuration probabilités par statut
  - `src/lib/stores/prospects.svelte.ts` : State global (Svelte 5 runes)
  - `src/lib/components/FileUpload.svelte` : Widget upload CSV
  - `src/lib/components/KPICard.svelte` : Card KPI réutilisable
  - `src/lib/components/StatusChart.svelte` : Graphique répartition
  - `src/routes/+page.svelte` : Page dashboard principale

**Dependencies:**
- Aucune dépendance externe (API, DB) pour le MVP
- Dépendances npm : @tanstack/svelte-table, svelte-chartjs, papaparse, date-fns, shadcn-svelte, lucide-svelte

**Migration path:**
- Le code est structuré pour faciliter la migration SQL future (Phase V3) :
  - Business logic dans utils/ (pure functions)
  - State centralisé dans stores/ (facile à remplacer par fetch API)
  - Types TypeScript réutilisables comme schema DB

**Out of scope (délibérément exclu du MVP) :**
- Tableau complet des prospects (Phase 2)
- Pipeline pondéré / weighted pipeline (Phase 2)
- Top 5 deals (Phase 2)
- Alertes overdue (Phase 2)
- Filtres avancés (Phase 3)
- Export CSV (Phase 3)
