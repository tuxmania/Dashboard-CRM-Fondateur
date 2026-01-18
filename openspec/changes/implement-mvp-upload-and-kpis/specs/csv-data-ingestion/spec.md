# Spec Delta: CSV Data Ingestion

## Purpose

Permet au fondateur d'uploader un fichier CSV contenant ses prospects/deals commerciaux et de les transformer en données structurées exploitables par le dashboard.

## ADDED Requirements

### Requirement: System SHALL provide CSV file upload

The system SHALL permettre à l'utilisateur d'uploader un fichier CSV via une interface de drag-and-drop ou un sélecteur de fichier standard.

#### Scenario: Upload réussi via drag-and-drop

- **WHEN** l'utilisateur glisse un fichier CSV valide dans la zone de drop
- **THEN** le fichier est accepté et le parsing démarre immédiatement
- **AND** un indicateur de chargement s'affiche pendant le parsing

#### Scenario: Upload réussi via file input

- **WHEN** l'utilisateur clique sur le bouton d'upload et sélectionne un fichier CSV
- **THEN** le fichier est accepté et le parsing démarre immédiatement
- **AND** un indicateur de chargement s'affiche pendant le parsing

#### Scenario: Fichier non-CSV rejeté

- **WHEN** l'utilisateur tente d'uploader un fichier avec extension .xlsx, .pdf, ou autre (pas .csv)
- **THEN** le système affiche un message d'erreur "Format de fichier non supporté. Veuillez uploader un fichier CSV."
- **AND** le fichier n'est pas parsé

### Requirement: System SHALL parse CSV with PapaParse

The system SHALL parser le fichier CSV avec la bibliothèque PapaParse en mode header (première ligne = noms de colonnes).

#### Scenario: Parsing avec colonnes valides

- **WHEN** le CSV contient toutes les colonnes requises (Task Name, Status, Date Created, Due Date, Start Date, Assignee, Priority, Tags, Task Content, Montant Deal)
- **THEN** le parsing réussit et retourne un tableau d'objets avec ces propriétés
- **AND** les lignes vides sont ignorées (skipEmptyLines: true)

#### Scenario: Parsing avec colonnes manquantes

- **WHEN** le CSV manque une ou plusieurs colonnes optionnelles (ex: Tags, Start Date)
- **THEN** le parsing continue et les valeurs manquantes sont traitées comme vides ou null
- **AND** les prospects sont créés avec des valeurs par défaut pour ces champs

#### Scenario: Erreur de parsing

- **WHEN** le fichier CSV est corrompu ou mal formaté (quotes non fermées, délimiteurs incorrects)
- **THEN** le système affiche un message d'erreur "Erreur lors du parsing du fichier CSV. Vérifiez le format."
- **AND** l'état de chargement se termine (isLoading = false)

### Requirement: System SHALL parse Task Name into contact and company

The system SHALL séparer la colonne Task Name (format "Contact Name - Company Name") en deux champs distincts : contactName et companyName.

#### Scenario: Task Name bien formaté

- **WHEN** Task Name est "Sophie Martin - TechStart"
- **THEN** contactName = "Sophie Martin"
- **AND** companyName = "TechStart"
- **AND** les espaces avant/après le tiret sont supprimés (trim)

#### Scenario: Task Name sans tiret

- **WHEN** Task Name est "Sophie Martin" (pas de tiret)
- **THEN** contactName = "Sophie Martin"
- **AND** companyName = "" (chaîne vide)

#### Scenario: Task Name avec multiples tirets

- **WHEN** Task Name est "Jean-Pierre Dupont - Tech-Start Inc."
- **THEN** contactName = "Jean-Pierre Dupont"
- **AND** companyName = "Tech-Start Inc."
- **AND** seul le premier tiret est utilisé comme séparateur

### Requirement: System SHALL parse and validate date fields

The system SHALL convertir les chaînes de dates (Date Created, Due Date, Start Date) en objets Date JavaScript, et gérer les valeurs nulles/invalides sans crasher.

#### Scenario: Date ISO valide

- **WHEN** Due Date est "2026-02-15"
- **THEN** le champ dueDate est un objet Date représentant le 15 février 2026
- **AND** la date est utilisable pour les calculs (overdue, tri)

#### Scenario: Date vide ou nulle

- **WHEN** Due Date est "" (chaîne vide) ou null
- **THEN** le champ dueDate est null
- **AND** aucune erreur n'est levée
- **AND** les calculs overdue traitent null comme "pas de deadline"

#### Scenario: Date invalide

- **WHEN** Due Date est "32/13/2026" (format invalide)
- **THEN** le champ dueDate est null
- **AND** un warning est loggé en console
- **AND** la row reste valide (pas de skip total)

### Requirement: System SHALL parse Montant Deal as numeric value

The system SHALL convertir la colonne Montant Deal (string) en nombre (float) pour les calculs KPI.

#### Scenario: Montant valide entier

- **WHEN** Montant Deal est "5000"
- **THEN** montantDeal = 5000 (number)

#### Scenario: Montant valide décimal

- **WHEN** Montant Deal est "5000.50"
- **THEN** montantDeal = 5000.50 (number)

#### Scenario: Montant vide ou invalide

- **WHEN** Montant Deal est "" ou "N/A" ou "TBD"
- **THEN** montantDeal = 0
- **AND** la row reste valide (pas de skip)

### Requirement: System SHALL normalize status values

The system SHALL normaliser les variations de statuts (casse, accents, variations textuelles) vers les valeurs canoniques : prospect, qualifié, négociation, gagné - en cours, à relancer.

#### Scenario: Statut avec variation de casse

- **WHEN** Status est "PROSPECT" ou "Prospect" ou "prospect"
- **THEN** status = "prospect" (lowercase)

#### Scenario: Statut avec variation textuelle

- **WHEN** Status est "Qualif" ou "qualifié" ou "Qualifié"
- **THEN** status = "qualifié"

#### Scenario: Statut "gagné" avec variations

- **WHEN** Status est "Gagné" ou "gagné - en cours" ou "Gagné en cours"
- **THEN** status = "gagné - en cours"

#### Scenario: Statut inconnu

- **WHEN** Status est "En attente" ou toute autre valeur non reconnue
- **THEN** status = "prospect" (valeur par défaut)
- **AND** un warning est loggé en console

### Requirement: System SHALL parse tags into array

The system SHALL parser la colonne Tags (format "Tag1|Tag2|Tag3") en tableau de strings.

#### Scenario: Tags multiples

- **WHEN** Tags est "SaaS|B2B|Enterprise"
- **THEN** tags = ["SaaS", "B2B", "Enterprise"]
- **AND** les espaces avant/après chaque tag sont supprimés (trim)

#### Scenario: Tag unique

- **WHEN** Tags est "SaaS"
- **THEN** tags = ["SaaS"]

#### Scenario: Pas de tags

- **WHEN** Tags est "" (vide) ou null
- **THEN** tags = [] (tableau vide)

#### Scenario: Tags avec séparateur et espaces

- **WHEN** Tags est " SaaS | B2B | Enterprise "
- **THEN** tags = ["SaaS", "B2B", "Enterprise"]
- **AND** les espaces parasites sont nettoyés

### Requirement: System SHALL parse and normalize priority

The system SHALL convertir la colonne Priority en valeurs normalisées : low, medium, high.

#### Scenario: Priority valide

- **WHEN** Priority est "High" ou "HIGH" ou "high"
- **THEN** priority = "high"

#### Scenario: Priority manquante

- **WHEN** Priority est "" ou null
- **THEN** priority = "medium" (valeur par défaut)

### Requirement: System SHALL calculate closure probability

The system SHALL calculer automatiquement la probabilité de closure (probability) pour chaque prospect en fonction de son statut, selon la configuration STATUS_PROBABILITIES.

#### Scenario: Prospect avec statut "qualifié"

- **WHEN** status = "qualifié"
- **THEN** probability = 0.40
- **AND** la probabilité est utilisée pour calculer weightedValue

#### Scenario: Prospect avec statut "gagné - en cours"

- **WHEN** status = "gagné - en cours"
- **THEN** probability = 1.00

### Requirement: System SHALL calculate weighted value

The system SHALL calculer la valeur pondérée (weightedValue) de chaque prospect comme montantDeal × probability.

#### Scenario: Calcul weighted value standard

- **WHEN** montantDeal = 10000 ET status = "négociation" (probability = 0.75)
- **THEN** weightedValue = 7500

#### Scenario: Calcul weighted value pour deal gagné

- **WHEN** montantDeal = 10000 ET status = "gagné - en cours" (probability = 1.00)
- **THEN** weightedValue = 10000

### Requirement: System SHALL calculate overdue status

The system SHALL déterminer si un prospect est overdue (isOverdue) en comparant dueDate avec la date actuelle, sauf pour les deals gagnés.

#### Scenario: Deal overdue

- **WHEN** dueDate = "2026-01-10" (dans le passé) ET status ≠ "gagné - en cours"
- **THEN** isOverdue = true

#### Scenario: Deal non overdue

- **WHEN** dueDate = "2026-03-01" (dans le futur)
- **THEN** isOverdue = false

#### Scenario: Deal gagné (ignoré pour overdue)

- **WHEN** dueDate = "2026-01-10" (dans le passé) ET status = "gagné - en cours"
- **THEN** isOverdue = false

#### Scenario: Pas de due date

- **WHEN** dueDate = null
- **THEN** isOverdue = false

### Requirement: System SHALL handle invalid rows gracefully

The system SHALL ignorer les rows invalides (erreurs de parsing critiques) et continuer le traitement des autres rows.

#### Scenario: Row avec Task Name vide

- **WHEN** Task Name est "" (vide)
- **THEN** la row est ignorée (skipped)
- **AND** un warning est loggé en console

#### Scenario: Parsing réussi avec quelques rows invalides

- **WHEN** le CSV contient 50 rows dont 3 invalides
- **THEN** 47 prospects sont créés et stockés dans le state
- **AND** les 3 rows invalides sont loggées en console (warning)

### Requirement: System SHALL manage error state

The system SHALL gérer les erreurs de parsing et mettre à jour l'état global (error, isLoading) pour affichage dans l'UI.

#### Scenario: Parsing réussi

- **WHEN** le CSV est parsé avec succès
- **THEN** isLoading = false
- **AND** error = null
- **AND** prospects contient les données parsées

#### Scenario: Erreur de parsing

- **WHEN** une erreur PapaParse se produit
- **THEN** isLoading = false
- **AND** error = "Erreur parsing CSV: <message détaillé>"
- **AND** prospects reste vide (tableau vide)
