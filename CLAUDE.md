<!-- OPENSPEC:START -->
# OpenSpec Instructions

These instructions are for AI assistants working in this project.

Always open `@/openspec/AGENTS.md` when the request:
- Mentions planning or proposals (words like proposal, spec, change, plan)
- Introduces new capabilities, breaking changes, architecture shifts, or big performance/security work
- Sounds ambiguous and you need the authoritative spec before coding

Use `@/openspec/AGENTS.md` to learn:
- How to create and apply change proposals
- Spec format and conventions
- Project structure and guidelines

Keep this managed block so 'openspec update' can refresh the instructions.

<!-- OPENSPEC:END -->

---

## Aperçu de l'Objectif du Projet

Dashboard CRM Single Page Application pour fondateur solo permettant le suivi de deals commerciaux avec focus sur **volume** et **valeur**. L'application ingère des données CSV et affiche des KPIs macro, des graphiques de répartition, un tableau complet de prospects, et des alertes opérationnelles.

## Aperçu de l'Architecture Globale

- **Framework** : SvelteKit 2.x + Svelte 5 (runes)
- **Language** : TypeScript 5.6 (strict mode)
- **UI Components** : shadcn-svelte + Tailwind CSS
- **Data Table** : TanStack Table v8
- **Charts** : svelte-chartjs + Chart.js
- **CSV Parsing** : PapaParse 5.4
- **State Management** : Svelte 5 Runes ($state, $derived)
- **Déploiement** : VPS auto-hébergé (Docker + nginx)

## Style Visuel

- Interface claire et minimaliste
- Pas de mode sombre pour le MVP

## Contraintes et Politiques

- **NE JAMAIS exposer les clés API au client**
- TypeScript strict mode obligatoire
- Code structuré pour migration SQL future
- Éviter l'over-engineering : solutions simples et focalisées

## Dépendances

- Préférer les composants existants (shadcn-svelte) plutôt que d'ajouter de nouvelles bibliothèques UI
- Limiter les dépendances pour maintenir un bundle size < 110kb

## À la Fin de Chaque Développement

**Pour les développements impliquant l'interface graphique** :
- Tester avec playwright-skill
- L'interface doit être responsive
- L'interface doit être fonctionnelle
- L'interface doit répondre au besoin développé

## Documentation

Pour comprendre le projet en détail, consulter :
- **Spécifications produit** : [@PRD.md](/Users/fred/Documents/Programming/claude-projets/projet-C/PRD.md)
- **Architecture technique** : [@ARCHITECTURE.md](/Users/fred/Documents/Programming/claude-projets/projet-C/ARCHITECTURE.md)

## Context7

Utilise toujours Context7 lorsque j'ai besoin de :
- Génération de code
- Étapes de configuration ou d'installation
- Documentation de bibliothèque/API

Cela signifie que tu dois automatiquement utiliser les outils MCP Context7 pour résoudre l'identifiant de bibliothèque et obtenir la documentation de bibliothèque **sans que j'aie à le demander explicitement**.

## Langue et Conventions

**Toutes les spécifications doivent être rédigées en français**, y compris les specs OpenSpec (sections Purpose et Scenarios).

**Exception** : Seuls les titres de Requirements dans OpenSpec doivent rester en anglais avec les mots-clés SHALL/MUST pour la validation OpenSpec.