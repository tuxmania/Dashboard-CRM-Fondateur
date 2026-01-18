# Spec Delta: KPI Calculation

## Purpose

Calcule les indicateurs clés de performance (KPIs) du pipeline commercial à partir des données prospects pour permettre au fondateur de piloter volume et valeur de son activité.

## ADDED Requirements

### Requirement: System SHALL calculate volume pipeline

The system SHALL calculer le nombre total de deals actifs (totalDeals) en comptant tous les prospects dont le statut n'est pas "perdu".

#### Scenario: Calcul avec tous deals actifs

- **WHEN** prospects contient 45 prospects avec statuts variés (prospect, qualifié, négociation, gagné - en cours, à relancer)
- **THEN** totalDeals = 45

#### Scenario: Calcul avec 0 prospects

- **WHEN** prospects est un tableau vide []
- **THEN** totalDeals = 0

#### Scenario: Exclusion des deals perdus (future-proof)

- **WHEN** prospects contient 50 prospects dont 5 avec status = "perdu"
- **THEN** totalDeals = 45
- **AND** les deals perdus sont exclus du calcul

### Requirement: System SHALL calculate pipeline brut

The system SHALL calculer le pipeline brut (pipelineBrut) en sommant les montantDeal de tous les deals actifs, sans pondération.

#### Scenario: Somme de plusieurs deals

- **WHEN** prospects contient 3 deals avec montantDeal = [5000, 10000, 7500]
- **THEN** pipelineBrut = 22500

#### Scenario: Pipeline brut avec 0 prospects

- **WHEN** prospects est un tableau vide []
- **THEN** pipelineBrut = 0

#### Scenario: Deals avec montant 0

- **WHEN** prospects contient 2 deals avec montantDeal = [5000, 0]
- **THEN** pipelineBrut = 5000
- **AND** les deals à 0 sont inclus dans le calcul (pas d'erreur)

### Requirement: System SHALL calculate panier moyen

The system SHALL calculer le panier moyen (panierMoyen) en divisant pipelineBrut par totalDeals.

#### Scenario: Calcul panier moyen standard

- **WHEN** pipelineBrut = 100000 ET totalDeals = 10
- **THEN** panierMoyen = 10000

#### Scenario: Panier moyen avec 0 deals

- **WHEN** totalDeals = 0
- **THEN** panierMoyen = 0
- **AND** aucune erreur de division par zéro

#### Scenario: Panier moyen avec décimales

- **WHEN** pipelineBrut = 75000 ET totalDeals = 7
- **THEN** panierMoyen = 10714.285714285714
- **AND** l'affichage UI arrondira à 2 décimales (formatCurrency)

### Requirement: System SHALL calculate pipeline pondéré (V1 preparation)

The system SHALL calculer le pipeline pondéré (pipelinePondere) en sommant les weightedValue de tous les deals actifs.

#### Scenario: Somme weighted values

- **WHEN** prospects contient 3 deals avec weightedValue = [1000, 4000, 5625]
- **THEN** pipelinePondere = 10625

#### Scenario: Pipeline pondéré avec 0 prospects

- **WHEN** prospects est un tableau vide []
- **THEN** pipelinePondere = 0

**Note :** pipelinePondere est calculé en MVP mais non affiché (sera ajouté en Phase 2).

### Requirement: System SHALL count overdue tasks (V1 preparation)

The system SHALL calculer le nombre de tâches overdue (overdueTasks) en comptant les prospects avec isOverdue = true.

#### Scenario: Comptage overdue tasks

- **WHEN** prospects contient 50 deals dont 8 avec isOverdue = true
- **THEN** overdueTasks = 8

#### Scenario: Aucune tâche overdue

- **WHEN** tous les prospects ont isOverdue = false
- **THEN** overdueTasks = 0

**Note :** overdueTasks est calculé en MVP mais non affiché (sera ajouté en Phase 2 sous forme de badge alerte).

### Requirement: System SHALL auto-update KPIs with reactive state

The system SHALL recalculer automatiquement les KPIs à chaque modification du tableau prospects via $derived (Svelte 5 runes).

#### Scenario: KPIs mis à jour après upload CSV

- **WHEN** un nouveau fichier CSV est uploadé
- **THEN** prospects est mis à jour avec les nouveaux prospects
- **AND** kpis est recalculé automatiquement via $derived
- **AND** l'UI affiche les nouveaux KPIs immédiatement

#### Scenario: KPIs recalculés après suppression (future)

- **WHEN** un prospect est supprimé du tableau prospects (Phase V1+)
- **THEN** kpis est recalculé automatiquement
- **AND** totalDeals diminue de 1
- **AND** pipelineBrut et panierMoyen sont ajustés

### Requirement: System SHALL format currency for display

The system SHALL fournir une fonction formatCurrency() pour formater les montants en euros avec le format français (séparateur milliers espace, symbole €, 0 décimales).

#### Scenario: Formatage montant entier

- **WHEN** formatCurrency(10000) est appelé
- **THEN** retourne "10 000 €"

#### Scenario: Formatage montant décimal

- **WHEN** formatCurrency(10750.75) est appelé
- **THEN** retourne "10 751 €"
- **AND** les décimales sont arrondies (maximumFractionDigits: 0)

#### Scenario: Formatage montant 0

- **WHEN** formatCurrency(0) est appelé
- **THEN** retourne "0 €"

#### Scenario: Formatage montant négatif (edge case)

- **WHEN** formatCurrency(-5000) est appelé
- **THEN** retourne "-5 000 €"

### Requirement: System SHALL implement KPI calculations as pure functions

The system SHALL implémenter calculateKPIs() comme une fonction pure (pas d'effets de bord, déterministe).

#### Scenario: Fonction pure déterministe

- **WHEN** calculateKPIs() est appelé 2 fois avec le même tableau prospects
- **THEN** retourne exactement le même objet KPIs (valeurs identiques)

#### Scenario: Pas de mutation du tableau d'entrée

- **WHEN** calculateKPIs(prospects) est appelé
- **THEN** le tableau prospects passé en paramètre n'est pas modifié
- **AND** la fonction ne fait que des lectures (pas de push, splice, etc.)

### Requirement: System SHALL return structured KPIs object

The system SHALL retourner un objet KPIs structuré avec les propriétés : totalDeals, pipelineBrut, pipelinePondere, panierMoyen, overdueTasks.

#### Scenario: Structure complète

- **WHEN** calculateKPIs() est appelé avec prospects valides
- **THEN** retourne un objet avec les 5 propriétés
- **AND** toutes les valeurs sont des nombres (pas de string, null, undefined)

#### Scenario: Types TypeScript validés

- **WHEN** le code est compilé avec `npm run check`
- **THEN** aucune erreur TypeScript n'est levée
- **AND** l'interface KPIs est respectée
