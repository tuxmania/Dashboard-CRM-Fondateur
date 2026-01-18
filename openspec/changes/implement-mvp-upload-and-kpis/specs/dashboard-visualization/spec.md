# Spec Delta: Dashboard Visualization

## Purpose

Affiche les KPIs macro et la répartition des deals par statut dans une interface claire et minimaliste permettant au fondateur de visualiser instantanément la "météo du business".

## ADDED Requirements

### Requirement: System SHALL display dashboard page layout

The system SHALL afficher une page dashboard unique (/) contenant : header, zone upload CSV, grille de 3 KPI cards, et graphique de répartition.

#### Scenario: Layout complet affiché

- **WHEN** l'utilisateur accède à la page /
- **THEN** le header "Dashboard CRM Fondateur" est visible en haut de page
- **AND** la zone d'upload CSV est visible en premier (si aucun prospect chargé)
- **AND** les 3 KPI cards sont affichées en grille responsive (1 colonne mobile, 3 colonnes desktop)
- **AND** le graphique de répartition est affiché en dessous des KPIs

#### Scenario: Responsive design mobile

- **WHEN** l'écran a une largeur <768px (mobile)
- **THEN** les KPI cards s'affichent en colonne unique (1 card par ligne)
- **AND** le graphique s'adapte à la largeur de l'écran

#### Scenario: Responsive design desktop

- **WHEN** l'écran a une largeur ≥768px (tablet/desktop)
- **THEN** les KPI cards s'affichent en grille 3 colonnes
- **AND** le graphique utilise toute la largeur disponible

### Requirement: System SHALL display KPI cards

The system SHALL afficher chaque KPI dans une card shadcn-svelte avec : titre, valeur principale, et subtitle explicatif.

#### Scenario: KPI Card "Volume du Pipeline"

- **WHEN** totalDeals = 45
- **THEN** la card affiche :
  - Titre : "Volume Pipeline"
  - Valeur : "45"
  - Subtitle : "deals actifs"

#### Scenario: KPI Card "CA Total Pipeline Brut"

- **WHEN** pipelineBrut = 225000
- **THEN** la card affiche :
  - Titre : "CA Total Brut"
  - Valeur : "225 000 €" (formatCurrency)
  - Subtitle : "sans pondération"

#### Scenario: KPI Card "Panier Moyen"

- **WHEN** panierMoyen = 5000
- **THEN** la card affiche :
  - Titre : "Panier Moyen"
  - Valeur : "5 000 €" (formatCurrency)
  - Subtitle : "par deal"

#### Scenario: KPIs avec 0 prospects

- **WHEN** prospects est vide (aucun CSV uploadé)
- **THEN** les 3 cards affichent :
  - Volume : "0"
  - CA Total : "0 €"
  - Panier Moyen : "0 €"

### Requirement: System SHALL display status distribution bar chart

The system SHALL afficher un graphique Bar Chart (svelte-chartjs) montrant le nombre de deals par statut (prospect, qualifié, négociation, gagné - en cours, à relancer).

#### Scenario: Graphique avec tous les statuts

- **WHEN** prospects contient des deals avec statuts variés (10 prospects, 15 qualifiés, 12 négociations, 8 gagnés)
- **THEN** le graphique affiche 4 barres avec les hauteurs correspondantes
- **AND** les labels sont les noms de statuts ("prospect", "qualifié", "négociation", "gagné - en cours")
- **AND** les valeurs sont les counts (10, 15, 12, 8)

#### Scenario: Graphique avec statut manquant

- **WHEN** aucun prospect n'a le statut "à relancer"
- **THEN** le graphique n'affiche pas de barre pour "à relancer"
- **AND** seuls les statuts présents sont affichés

#### Scenario: Graphique responsive

- **WHEN** la fenêtre est redimensionnée
- **THEN** le graphique s'adapte automatiquement à la nouvelle largeur
- **AND** maintainAspectRatio: false permet un contrôle de hauteur fixe (h-64)

#### Scenario: Couleurs des barres

- **WHEN** le graphique est affiché
- **THEN** chaque statut a une couleur distincte :
  - Bleu (rgba(59, 130, 246, 0.8)) pour le premier statut
  - Vert (rgba(34, 197, 94, 0.8)) pour le second
  - Orange (rgba(251, 146, 60, 0.8)) pour le troisième
  - Violet (rgba(168, 85, 247, 0.8)) pour le quatrième

### Requirement: System SHALL display empty state

The system SHALL afficher un message explicite quand aucun prospect n'est chargé.

#### Scenario: État vide initial

- **WHEN** l'utilisateur accède au dashboard pour la première fois
- **AND** aucun CSV n'a été uploadé
- **THEN** un message "Aucun prospect chargé. Uploadez un fichier CSV pour commencer." est affiché
- **AND** les KPI cards et le graphique ne sont pas affichés

#### Scenario: Retour à l'état vide après erreur

- **WHEN** un CSV invalide a été uploadé (erreur de parsing)
- **THEN** le message d'état vide est affiché
- **AND** un message d'erreur est également visible au-dessus

### Requirement: System SHALL display loading state

The system SHALL afficher un indicateur de chargement pendant le parsing du CSV.

#### Scenario: Loading pendant parsing

- **WHEN** le CSV est en cours de parsing (isLoading = true)
- **THEN** un spinner ou message "Chargement..." est affiché
- **AND** les KPI cards et graphique ne sont pas visibles pendant le chargement

#### Scenario: Fin du loading

- **WHEN** le parsing est terminé (isLoading = false)
- **THEN** le spinner disparaît
- **AND** les KPIs et graphique s'affichent immédiatement

### Requirement: System SHALL display error state

The system SHALL afficher les erreurs de parsing dans une zone d'erreur visible en haut du dashboard.

#### Scenario: Erreur de parsing CSV

- **WHEN** error = "Erreur parsing CSV: <message>"
- **THEN** une zone d'erreur rouge (bg-destructive/10 text-destructive) est affichée
- **AND** le message complet de l'erreur est visible
- **AND** l'utilisateur peut réessayer en uploadant un autre fichier

#### Scenario: Erreur cleared après upload réussi

- **WHEN** un CSV valide est uploadé après une erreur
- **THEN** error = null
- **AND** la zone d'erreur disparaît
- **AND** les KPIs normaux s'affichent

### Requirement: System SHALL integrate file upload component

The system SHALL intégrer le composant FileUpload dans la page dashboard avec gestion d'événements.

#### Scenario: Upload déclenché

- **WHEN** l'utilisateur uploade un fichier via FileUpload
- **THEN** l'événement on:upload est déclenché avec le fichier
- **AND** la fonction handleFileUpload() est appelée
- **AND** loadFromCSV() est exécuté pour parser le fichier

#### Scenario: Réactivité après upload

- **WHEN** loadFromCSV() termine avec succès
- **THEN** prospects est mis à jour
- **AND** kpis est recalculé automatiquement
- **AND** l'UI se met à jour immédiatement (réactivité Svelte)

### Requirement: System SHALL configure Chart.js

The system SHALL configurer Chart.js avec les paramètres : responsive = true, beginAtZero = true pour l'axe Y, stepSize = 1 pour afficher des nombres entiers.

#### Scenario: Axe Y commence à 0

- **WHEN** le graphique est affiché avec des counts (5, 10, 15)
- **THEN** l'axe Y commence à 0 (pas de troncation)
- **AND** les barres sont proportionnelles aux valeurs

#### Scenario: Axe Y avec stepSize = 1

- **WHEN** les counts sont des petits nombres (1, 2, 3)
- **THEN** l'axe Y affiche des graduations entières : 0, 1, 2, 3
- **AND** pas de graduations décimales (0.5, 1.5, etc.)

### Requirement: System SHALL use Tailwind CSS styling

The system SHALL utiliser Tailwind CSS pour tous les styles avec les classes utilitaires.

#### Scenario: Container responsive

- **WHEN** le contenu du dashboard est affiché
- **THEN** le container utilise : container mx-auto p-6
- **AND** le contenu est centré avec padding responsive

#### Scenario: Espacement entre sections

- **WHEN** plusieurs sections sont affichées (header, KPIs, graphique)
- **THEN** space-y-6 crée un espacement vertical de 1.5rem entre chaque section

#### Scenario: Grille KPIs responsive

- **WHEN** les KPI cards sont affichées
- **THEN** la grille utilise : grid grid-cols-1 md:grid-cols-3 gap-4
- **AND** 1 colonne sur mobile, 3 colonnes sur desktop

### Requirement: System SHALL use shadcn-svelte Card component

The system SHALL utiliser le composant Card de shadcn-svelte pour les KPIs et le graphique.

#### Scenario: Card KPI avec padding

- **WHEN** une KPI card est affichée
- **THEN** elle utilise <Card> avec classe p-6 pour padding interne
- **AND** le contenu est bien espacé et lisible

#### Scenario: Card graphique avec titre

- **WHEN** le graphique est affiché
- **THEN** il est enveloppé dans une <Card> avec :
  - Titre h2 : "Répartition des Deals par Statut"
  - Padding : p-6
  - Hauteur fixe pour chart : h-64 (256px)

### Requirement: System SHALL enforce TypeScript type safety

The system SHALL garantir la sécurité de types TypeScript pour tous les composants et props.

#### Scenario: Props typées KPICard

- **WHEN** <KPICard> est utilisé
- **THEN** les props title, value, subtitle sont typées (title: string, value: string | number, subtitle: string)
- **AND** une erreur TypeScript est levée si un prop manquant ou mal typé

#### Scenario: Props typées StatusChart

- **WHEN** <StatusChart> est utilisé
- **THEN** le prop prospects est typé : prospects: Prospect[]
- **AND** une erreur TypeScript est levée si on passe un tableau de type incorrect

### Requirement: System SHALL implement accessibility with ARIA labels

The system SHALL inclure des labels ARIA pour les composants interactifs afin d'assurer l'accessibilité.

#### Scenario: KPI Cards avec rôle sémantique

- **WHEN** les KPI cards sont rendues
- **THEN** chaque card utilise une structure sémantique HTML (h3 pour titre, p pour valeur)
- **AND** le texte est lisible par les lecteurs d'écran

#### Scenario: Graphique avec label

- **WHEN** le graphique Bar Chart est rendu
- **THEN** un titre descriptif est présent au-dessus du graphique
- **AND** Chart.js génère automatiquement les attributs ARIA pour le canvas

### Requirement: System SHALL achieve First Contentful Paint under 1 second

The system SHALL charger et afficher le dashboard initial (empty state) en moins de 1 seconde.

#### Scenario: Chargement initial rapide

- **WHEN** l'utilisateur accède à / pour la première fois
- **THEN** le header et la zone d'upload sont visibles en <1s
- **AND** le bundle JavaScript est <110kb (gzipped)
- **AND** pas de blocking render

#### Scenario: Réactivité après upload

- **WHEN** un CSV de 50 lignes est uploadé
- **THEN** les KPIs et le graphique s'affichent en <500ms
- **AND** pas de lag perceptible dans l'UI
