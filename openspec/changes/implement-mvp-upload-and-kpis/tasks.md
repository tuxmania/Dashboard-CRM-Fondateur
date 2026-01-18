# Tasks - MVP Dashboard CRM (Phase 1)

## 1. Setup Projet SvelteKit

- [ ] 1.1 Initialiser projet SvelteKit avec TypeScript
- [ ] 1.2 Configurer Tailwind CSS + PostCSS
- [ ] 1.3 Installer shadcn-svelte et ajouter composants de base (button, card, badge)
- [ ] 1.4 Configurer adapter-node pour déploiement VPS
- [ ] 1.5 Créer structure de dossiers (lib/types, lib/utils, lib/components, lib/stores)
- [ ] 1.6 Configurer TypeScript strict mode (tsconfig.json)

## 2. Modèle de Données & Types

- [ ] 2.1 Créer `src/lib/types/prospect.ts` avec interfaces (Prospect, CSVRow, KPIs, ProspectStatus, Priority)
- [ ] 2.2 Définir STATUS_PROBABILITIES dans `src/lib/utils/probabilities.ts`
- [ ] 2.3 Valider types avec `npm run check`

## 3. Parsing CSV & Transformation

- [ ] 3.1 Installer PapaParse et date-fns
- [ ] 3.2 Créer `src/lib/utils/csvParser.ts` avec fonction parseCSV()
- [ ] 3.3 Implémenter parseTaskName() pour séparer Contact Name / Company Name
- [ ] 3.4 Implémenter parseDate() avec gestion des valeurs nulles
- [ ] 3.5 Implémenter normalizeStatus() pour normaliser les variations de statuts
- [ ] 3.6 Calculer probability et weightedValue dans transformRow()
- [ ] 3.7 Tester avec crm_prospects_demo.csv

## 4. Calculs KPIs

- [ ] 4.1 Créer `src/lib/utils/kpiCalculations.ts`
- [ ] 4.2 Implémenter calculateKPIs() (totalDeals, pipelineBrut, panierMoyen)
- [ ] 4.3 Implémenter formatCurrency() pour formatage euros
- [ ] 4.4 Écrire tests unitaires Vitest pour kpiCalculations
- [ ] 4.5 Écrire tests unitaires Vitest pour csvParser

## 5. State Management (Svelte 5 Runes)

- [ ] 5.1 Créer `src/lib/stores/prospects.svelte.ts`
- [ ] 5.2 Définir state réactif avec $state (prospects, isLoading, error)
- [ ] 5.3 Créer $derived pour KPIs automatiques
- [ ] 5.4 Implémenter loadFromCSV() avec gestion erreurs
- [ ] 5.5 Exporter getters (getProspects, getKPIs, getLoadingState, getError)

## 6. Composants UI

- [ ] 6.1 Créer `src/lib/components/FileUpload.svelte` (drag & drop + input file)
- [ ] 6.2 Créer `src/lib/components/KPICard.svelte` (card réutilisable avec titre, valeur, subtitle)
- [ ] 6.3 Installer svelte-chartjs + Chart.js
- [ ] 6.4 Créer `src/lib/components/StatusChart.svelte` (Bar chart répartition par statut)
- [ ] 6.5 Créer layout global `src/routes/+layout.svelte` (header, title app)

## 7. Page Dashboard Principale

- [ ] 7.1 Créer `src/routes/+page.svelte`
- [ ] 7.2 Intégrer FileUpload component
- [ ] 7.3 Afficher 3 KPICards (Volume pipeline, CA Total brut, Panier moyen)
- [ ] 7.4 Afficher StatusChart
- [ ] 7.5 Gérer états (loading, error, empty state)
- [ ] 7.6 Tester responsive design (mobile, tablet, desktop)

## 8. Configuration Déploiement

- [ ] 8.1 Créer Dockerfile (multi-stage build)
- [ ] 8.2 Créer docker-compose.yml (crm-dashboard + nginx)
- [ ] 8.3 Créer nginx.conf (reverse proxy + HTTPS)
- [ ] 8.4 Ajouter .env.example avec variables d'environnement
- [ ] 8.5 Tester build local (`npm run build`)

## 9. Tests & Validation

- [ ] 9.1 Exécuter tous les tests unitaires (`npm run test`)
- [ ] 9.2 Vérifier bundle size (<110kb avec `npm run build`)
- [ ] 9.3 Tester avec playwright-skill (upload CSV, affichage KPIs, graphique)
- [ ] 9.4 Vérifier accessibilité et responsive design
- [ ] 9.5 Valider conformité avec PRD Phase 1

## 10. Documentation

- [ ] 10.1 Ajouter README.md avec instructions setup et dev
- [ ] 10.2 Documenter format CSV attendu
- [ ] 10.3 Ajouter commentaires JSDoc dans utils/
- [ ] 10.4 Vérifier cohérence avec ARCHITECTURE.md

---

**Dépendances :**
- Task 3 dépend de Task 2 (types doivent exister avant parsing)
- Task 5 dépend de Tasks 3 et 4 (store utilise parsing et calculs)
- Task 6 et 7 dépendent de Task 5 (composants consomment store)
- Task 9 dépend de tous les tasks précédents

**Parallelisable :**
- Tasks 1 et 2 peuvent être faites en parallèle
- Task 8 (Docker) peut être fait indépendamment après Task 1
- Tests unitaires (4.4, 4.5) peuvent être écrits pendant le développement

**Validation finale :**
- L'application doit permettre d'uploader crm_prospects_demo.csv et afficher :
  - Volume du pipeline (nombre total)
  - CA Total pipeline Brut (somme en euros)
  - Panier moyen (moyenne en euros)
  - Graphique Bar Chart avec répartition par statuts (Prospect, Qualifié, Négociation, Gagné - en cours, À relancer)
