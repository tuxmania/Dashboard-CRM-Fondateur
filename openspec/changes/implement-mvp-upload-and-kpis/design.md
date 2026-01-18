# Design - MVP Dashboard CRM (Phase 1)

## Context

Premier développement du Dashboard CRM "Fondateur" - projet greenfield sans code existant. Le MVP doit démontrer la viabilité technique de l'approche SvelteKit + CSV client-side avant d'investir dans les features avancées (Phase 2/3) et la migration SQL future (Phase V3+).

**Contraintes clés :**
- Pas de base de données (CSV uniquement)
- Single-user (pas d'auth)
- Bundle size <110kb
- Code structuré pour migration SQL future
- Déploiement VPS auto-hébergé (Docker + nginx)

**Stakeholders :**
- Fondateur solo (utilisateur final)
- Dev (moi) : Livraison rapide MVP en 2-3 jours

## Goals / Non-Goals

### Goals
1. **Validation technique** : Prouver que SvelteKit 5 + CSV parsing client-side peut gérer ~50-200 deals avec performance <1s
2. **Feedback utilisateur rapide** : Dashboard fonctionnel en 2-3 jours pour valider l'utilité des KPIs macro
3. **Foundation solide** : Architecture propre permettant d'ajouter Phase 2 (tableau complet, top deals) sans refactoring majeur
4. **Migration SQL-ready** : Séparation claire data layer / business logic / UI pour faciliter la migration future vers PostgreSQL

### Non-Goals
- ❌ Persistance des données (pas de localStorage, pas de DB)
- ❌ Multi-utilisateur ou authentification
- ❌ CRUD manuel de prospects (CSV = source de vérité)
- ❌ Optimisation prématurée (filtres complexes, virtualisation, etc.)

## Decisions

### Decision 1: Svelte 5 Runes pour State Management (pas de stores traditionnels)

**Rationale :**
- Svelte 5 runes ($state, $derived) offrent une réactivité native plus simple que writable/readable stores
- Performance supérieure (pas de subscription overhead)
- Code plus concis et lisible pour composants et state global
- Cohérent avec la direction future de Svelte

**Alternatives considérées :**
- Writable stores (Svelte 4 style) : Plus verbeux, API plus ancienne, moins performant
- Zustand/Jotai : Ajout d'une dépendance externe non nécessaire pour un projet simple

**Implementation :**
```typescript
// src/lib/stores/prospects.svelte.ts
let prospects = $state<Prospect[]>([]);
const kpis = $derived<KPIs>(calculateKPIs(prospects));

export function getProspects() { return prospects; }
export function getKPIs() { return kpis; }
```

### Decision 2: PapaParse pour CSV Parsing (pas d'implémentation custom)

**Rationale :**
- Bibliothèque mature (5.4) avec support RFC 4180 complet
- Gestion robuste des edge cases (quotes, newlines, encodings)
- Bundle size acceptable (~13kb gzipped)
- API simple avec support streaming (utile si migration vers gros fichiers)

**Alternatives considérées :**
- CSV parser custom : Risque de bugs sur edge cases, temps de dev supérieur
- csv-parser (Node.js) : Non compatible browser
- d3-dsv : Moins de features, parsing moins robuste

**Implementation :**
```typescript
Papa.parse<CSVRow>(file, {
  header: true,
  transformHeader: (h) => h.trim(),
  complete: (results) => { /* transform rows */ }
});
```

### Decision 3: TanStack Table v8 (préparation Phase 2, pas utilisé en MVP)

**Rationale :**
- Phase 1 MVP n'a pas de tableau → Pas d'installation immédiate
- Phase 2 nécessitera un tableau complet avec tri/recherche → TanStack Table sera ajouté à ce moment
- Décision documentée maintenant pour cohérence architecture

**Pourquoi TanStack Table (futur) :**
- Headless (contrôle total du styling avec shadcn-svelte)
- Performance excellente (virtualisation native)
- API TypeScript-first avec types solides

### Decision 4: svelte-chartjs + Chart.js pour visualisation

**Rationale :**
- Wrapper Svelte officiel pour Chart.js (bibliothèque mature)
- Bar Chart simple pour MVP (répartition par statuts)
- Bundle size acceptable (~50kb Chart.js + ~5kb svelte-chartjs)
- Extensible pour Phase 3 (variations KPIs, drill-down)

**Alternatives considérées :**
- D3.js : Overkill pour un simple bar chart, courbe d'apprentissage élevée, bundle size supérieur
- Vega-Lite : Complexité excessive pour MVP
- Echarts : Bundle size trop lourd (~150kb)

**Implementation :**
```svelte
<script>
  import { Bar } from 'svelte-chartjs';
  const data = $derived({ labels: [...], datasets: [...] });
</script>
<Bar data={data} options={options} />
```

### Decision 5: shadcn-svelte pour composants UI (pas de Material UI, Flowbite, etc.)

**Rationale :**
- Composants copiables dans le projet (pas de dépendance runtime lourde)
- Tailwind-native (cohérent avec stack)
- Personnalisable à 100% (pas de CSS overrides complexes)
- Accessibilité intégrée (ARIA, keyboard navigation)

**Alternatives considérées :**
- Skeleton UI : Moins mature, moins de composants disponibles
- Flowbite Svelte : Dépendance runtime, moins flexible
- Build from scratch : Temps de dev supérieur, risque accessibilité

**Composants utilisés (MVP) :**
- Card (KPICard wrapper)
- Button (FileUpload)
- Badge (futur : alertes Phase 2)

### Decision 6: Date-fns pour manipulation dates (pas de Day.js ou Luxon)

**Rationale :**
- Fonctions pures tree-shakable (bundle size minimal, ~2-5kb pour parseISO + isValid)
- API simple et claire
- TypeScript-first
- Pas besoin de i18n complexe pour MVP (formatage euros avec Intl.NumberFormat natif)

**Alternatives considérées :**
- Day.js : Bundle size similaire mais API moins TypeScript-friendly
- Luxon : Plus lourd (~20kb), overkill pour parsing simple
- Native Date API : Parsing ISO fragile (new Date() incohérent cross-browser)

### Decision 7: Déploiement Docker + nginx (pas de Vercel/Netlify)

**Rationale :**
- Requirement : VPS auto-hébergé (contrôle total, pas de vendor lock-in)
- Docker : Portabilité, isolation, facilite CI/CD future
- nginx : Reverse proxy mature, SSL termination, compression gzip/brotli

**Alternatives considérées :**
- Déploiement direct Node.js + PM2 : Moins portable, config serveur manuelle
- Vercel/Netlify : Interdit par contrainte projet (VPS requis)
- Caddy : Plus simple mais moins répandu (nginx = standard industrie)

**Infrastructure :**
```
Client → nginx:443 (SSL) → SvelteKit:3000 (container)
```

### Decision 8: Pas de localStorage pour persistance (délibéré)

**Rationale :**
- Simplicité MVP : Upload CSV = état frais à chaque session
- Évite bugs de synchronisation localStorage <> CSV
- Migration SQL future sera la vraie solution de persistance
- Utilisateur peut toujours re-uploader le CSV (1 clic)

**Trade-off :**
- UX : Perte de données au refresh (acceptable pour MVP)
- Gain : Simplicité code, pas de gestion de versioning localStorage

## Risks / Trade-offs

### Risk 1: Performance avec gros fichiers CSV (>500 deals)

**Risque :**
- Parsing client-side peut bloquer UI thread si fichier >2MB
- Chart.js peut ralentir avec >100 catégories

**Mitigation :**
- Phase 1 : Target ~50-200 deals (fichier démo = 50 lignes)
- PapaParse supporte streaming (worker mode) si besoin futur
- Si >500 deals → Migration SQL devient prioritaire (Phase V3)

**Monitoring :**
- Tester avec fichier 500 lignes (générer depuis démo CSV)
- Lighthouse performance score doit rester >90

### Risk 2: Compatibilité formats CSV (encodings, delimiters)

**Risque :**
- CSV exportés depuis Excel peuvent avoir encoding Windows-1252 ou délimiteurs `;`
- PapaParse ne détecte pas automatiquement l'encoding

**Mitigation :**
- Documenter format attendu (UTF-8, délimiteur `,`)
- PapaParse auto-détecte délimiteur (`,` vs `;` vs `\t`)
- Phase 2 : Ajouter message d'erreur explicite si parsing échoue

### Risk 3: Bundle size dépassant 110kb

**Risque actuel (estimations) :**
- Chart.js : ~50kb
- PapaParse : ~13kb
- date-fns (2 fonctions) : ~3kb
- shadcn components : ~10kb
- Svelte runtime : ~5kb
- **Total : ~81kb** ✅ (marge de 29kb)

**Mitigation :**
- Vite tree-shaking automatique (import spécifiques)
- Compression gzip nginx (~70% réduction)
- Monitorer avec `npm run build` + analyse bundle

### Risk 4: Migration SQL difficile (coupling fort UI/data)

**Mitigation actuelle :**
- Séparation stricte :
  - Types : `src/lib/types/prospect.ts` (réutilisable comme schema DB)
  - Business logic : `src/lib/utils/*.ts` (pure functions, aucune dépendance UI)
  - Data layer : `src/lib/stores/prospects.svelte.ts` (facile à remplacer par fetch API)
  - UI : Composants consomment via getters (pas de coupling direct)

**Exemple migration future :**
```typescript
// Avant (MVP)
export async function loadFromCSV(file: File) { /* PapaParse */ }

// Après (V3 SQL)
export async function loadFromAPI() {
  const res = await fetch('/api/prospects');
  prospects = await res.json();
}
// → Composants inchangés car utilisent getProspects() !
```

## Migration Plan

### Phase 1 → Phase 2 (V1 - Outil de pilotage)

**Changements :**
1. Ajouter TanStack Table (`npm i @tanstack/svelte-table`)
2. Créer `ProspectsTable.svelte` (tri, recherche)
3. Ajouter `pipelinePondere` dans `kpiCalculations.ts`
4. Créer `TopDealsPanel.svelte` (top 5 weightedValue)
5. Ajouter `overdueTasks` dans KPIs

**Impact code existant :**
- Aucun breaking change (ajouts uniquement)
- `+page.svelte` : Ajouter 2 nouveaux composants
- Store : Ajouter `topDeals = $derived(...)` et `searchQuery = $state('')`

### Phase 2 → Phase 3 (V2 - L'explorateur)

**Changements :**
1. Ajouter filtres (dropdown Status, Priority, Tags)
2. Export CSV avec Papa.unparse()
3. Indicateurs variations KPIs (nécessite historique → complexe)

**Complexité :**
- Variations KPIs = besoin d'historiser (localStorage temporaire ou attendre migration SQL)

### Phase V3 : Migration SQL

**Pré-requis :**
- >500 deals OU besoin multi-utilisateur OU besoin CRUD manuel

**Steps :**
1. Setup PostgreSQL + Prisma ORM
2. Créer schema DB depuis `src/lib/types/prospect.ts`
3. Créer API endpoints SvelteKit (`src/routes/api/prospects/+server.ts`)
4. Remplacer `loadFromCSV` par `loadFromAPI` dans store
5. Ajouter Auth.js pour multi-utilisateur
6. Migration progressive (garder upload CSV comme fallback temporaire)

**Rollback :**
- Conserver code CSV en parallèle (feature flag)
- Si bugs API → Fallback sur CSV upload

## Open Questions

### Q1: Faut-il ajouter un indicateur de progression pendant parsing CSV ?

**Contexte :**
- PapaParse peut prendre 1-2s pour gros fichiers
- UX : Utilisateur ne sait pas si ça charge ou si c'est bloqué

**Options :**
- A) Ajouter spinner + barre de progression (PapaParse step callback)
- B) Juste un spinner simple (isLoading = true)
- C) Rien (parsing trop rapide pour <200 lignes)

**Recommandation :** Option B pour MVP (spinner simple), Option A si feedback utilisateur demande plus de détails.

---

### Q2: Quel format de date afficher dans le graphique futur (Phase 2 drill-down) ?

**Options :**
- A) Format français : `18/01/2026`
- B) Format ISO : `2026-01-18`
- C) Format relatif : `Il y a 5 jours`

**Recommandation :** Option A (cohérent avec locale français), ajouter tooltip avec format relatif.

---

### Q3: Faut-il valider la structure du CSV avant parsing complet ?

**Contexte :**
- Actuellement : Parsing complet → skip invalid rows → affichage
- Risque : Upload d'un CSV complètement invalide → 0 prospects affichés sans message clair

**Options :**
- A) Pré-validation des colonnes requises (Task Name, Status, Montant Deal)
- B) Juste afficher erreur si 0 prospects après parsing
- C) Rien (l'utilisateur verra que ça ne marche pas)

**Recommandation :** Option B pour MVP (message "Aucun prospect valide trouvé. Vérifiez le format CSV."), Option A pour Phase 2.

---

## Conclusion

Architecture MVP validée avec decisions clés :
- ✅ Svelte 5 runes (state management)
- ✅ PapaParse (CSV parsing)
- ✅ svelte-chartjs (visualisation)
- ✅ shadcn-svelte (composants UI)
- ✅ Docker + nginx (déploiement)

**Next steps :** Implémenter tasks.md dans l'ordre, valider avec playwright-skill après chaque section majeure (2-4-6-7-9).
