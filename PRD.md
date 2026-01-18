# Spécifications techniques : Dashboard CRM "Fondateur"

## 1. Contexte et Objectif

Développer une **single page application (SPA)** de suivi commercial pour un fondateur unique.
L'objectif est de maximiser deux axes : le **volume** de deal et la **valeur** des deals.
L'application ne nécessite pas de base de données pour l'instant (input via CSV), mais le code doit être structuré pour faciliter une migration future vers SQL.

---

## 2. Sources des données (Input)

L'application ingère un fichier CSV (crm_prospects_demo.csv) avec les colonnes suivantes :

* `Task Name` (Format : "Nom Prénom - Nom Entreprise") 
* `Status` (Valeurs : Prospect, Qualifié, Négociation, Gagné - en cours) 
* `Date Created`, `Due date`, `Start Date`(Dates) 
* `Assignee`, `Priority` (Low, Medium, High) 
* `Tags` (Séparés par des pipes, exemples : SaaS, B2B) 
* `Task Content` (Notes Contextuelles)
* `Montant Deal` (Entier/Float)

---

## 3. Règles métier & Data Engineering

### A : Nettoyage des données (Processing)

1. **Parsing Identité:** Séparer la colonne `Task Name` en deux nouvelles colonnes :
* `Contact Name` (avant le tiret)
* `Company Name` (après le tiret)

2. **Typage :**
* Convertir `Montant Deal` en numérique.
* Convertir `Due Date` en Objet Date.
* Gérer les valeurs nulles pour les dates (pour éviter les crashs).

3. **Tags : ** Parser la colonne `Tags` pour transformer "A|B" en liste `['A', 'B']`.

### B. Configuration des probabilités (pour V1)

 Le code doit définir un dictionnaire de probabilités centralisé :
 
 * `Prospect`: 10%
 * `qualifié` : 40%
 * `négociation`: 75%
 * `gagné-en-cours`: 100%
 
 ### C. Calcul des KPIs
 
 * **Pipeline brut (€) :** Somme de `Montant Deal`(tous statuts sauf perdus)
* **Pipeline Pondéré / Weighted Pipeline (€) :** Somme de (`Montant Deal`x `Probabilité de statut`)
* **Panier moyen / Average Deal Size (€) :** `Pipeline Brut` / `Nombre de deals actifs`

---

## 4. Roadmap de développement

### Phase 1 : MVP ("La météo du business")

*Objectif: Vue macro immédiate. Pas de tableau de données*

1. **Upload CSV:** Widget d'upload fichier
2. **Affichage des KPIs Macro: **
* **Volume du pipeline** (Nombre de deals actifs)
* **CA Total pipeline Brut** (Somme totale sans pondération)
* **Panier moyen** (Indicateur de valeur)

3. **Visualisation : **
* Un graphique simple (Bar Chart ou Donut) montrant la **Répartition des deals par étapes** (Status)

### Phase 2 : V1 ("L'outil de pilotage")

*Objectif: Intelligence et gestion opérationnelle*

1. **Mise à jour KPIs :** Ajout du **CA Prévisionnel(Weighted Pipeline)** calculé avec les probabilitées
2. **Focus List (Top 5) :** Petit tableau affichant les 5 deals ayant la plus forte **Valeur pondérée**
3. **Tableau Complet (Data Grid) :**
* Affichage de toutes les données nettoyées
* **Tri :** Colonnes triables (click-to-sort)
* **Recherche :** Barre de recherche textuelle globale
* *Note : Pas de filtres déroulants ici*
4. **Alertes :** Compteur visual des tâches dont la `Due Date` est passée
5. **Drill-Down :** Possibilité de voir le détail complet (Notes/Tags) d'un deal

### Phase 3 : V2 ("L'explorateur")

*Objectif: Analyse fine*

1. **Filtres avancés :** Ajout de menus déroulants au-dessus du tableau (Filtrer par Status, Priority, Tags)
2. **Export :** Bouton pour télécharger le CSV nettoyé et enrichi
3. **Feedback Visual :** Indicateurs de variation sur les KPIs (ex: évolution du panier moyen)

---

## 5. Hors périmètre (Out of Scope)

* Authentification / Login
* Base de données (SQL) pour l'instant
* Historisation des données
* Persistance locale des filtres (Session State complexe)
* Formulaires d'ajout/édition de prospects (CRUD)
* Paiement / Facturation