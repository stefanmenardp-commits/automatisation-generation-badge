# Création automatique de badges PDF pour les participants aux événements

Automatisation pour générer des badges pour des événements

## Avis de confidentialité

Ce projet a été mené dans un cadre professionnel.

Le code source, les données de production, les informations relatives aux clients et les captures d'écran originales ne sont pas accessibles au public pour des raisons de confidentialité.

Ce référentiel présente la portée du projet et l’architecture technique.

Certains diagrammes et illustrations ont pu être recréés à partir d’informations anonymisées ou fictives.

## Table des matières

- [Présentation du projet](#présentation-du-projet)
- [Contexte métier](#contexte-métier)
- [Objectifs](#objectifs)
- [Architecture technique](#architecture-technique)
- [Étapes du workflow](#étapes-du-workflow)
- [Stack technique](#stack-technique)
- [Compétences développées](#compétences-développées)

## Présentation du projet

Mise en place d’un service automatisé permettant de générer des badges au format PDF pour les participants de plusieurs événements.

Les données des participants et des événements sont récupérées depuis HelloAsso. Les badges sont générés à partir de templates spécifiques à chaque événement, puis stockés et organisés automatiquement dans Google Drive.

Le projet comprenait également la création d’une documentation utilisateur et technique afin de faciliter l’utilisation, la maintenance et la compréhension de la solution.

---

## Contexte métier

La génération manuelle des badges représentait une tâche chronophage et source d’erreurs, notamment lorsque plusieurs événements étaient organisés simultanément.

---

## Objectifs

Le besoin consistait à permettre aux utilisateurs de :

- récupérer automatiquement les participants inscrits
- associer un template à chaque événement
- générer rapidement les badges au format PDF
- regrouper les badges dans un fichier unique
- identifier les éventuels erreurs (doublon)

---

## Architecture technique

L’architecture reposait sur plusieurs services complémentaires :

- HelloAsso : source des événements et des participants
- n8n : orchestration des workflows et automatisation
- Google Sheets : listing des participants, sélection des templates et déclenchement de la génération
- Google Drive : stockage des templates, badges individuels et fichiers fusionnés
- Google Apps Script : gestion dynamique des templates, contrôle des homonymes et interaction avec Google Sheets
- Google Cloud Functions : génération et fusion des fichiers PDF
- Gmail : envoi des notifications aux utilisateurs

### Structure du stockage - Google Drive

Le stockage des différentes entités s’effectuait sous Google Drive, voici la structure choisie : 

```text
(Dossier) template_badge
│   ├── (Dossier) Evenement_nomEvenement_dateEvenement
  │   ├── (Fichier) badge_template_x.pdf
(Dossier) stockage_badge
│   ├── (Dossier) Evenement_nomEvenement_dateEvenement
  │   ├── (Dossier) yyyymmdd
    │   ├── (Fichier - badge individuel) badge_jean_eudes_yyyymmdd.pdf
    │   ├── (Fichier - badge individuel) badge_jeohn_doe_yyyymmdd.pdf
    │   ├── (Fichier - badges fusionnés) badges.pdf
(Fichier) listing_participant.gsheet
```
### Fichier central

Tout ce qui concerne la génération des badges se faisait sur un fichier Google Sheet central, qui se nomme "listing_participant". Il contenait toutes les informations des participants aux événements, avec la possibilité d'exécuter les scénarios pour mettre à jour les templates et pour générer les badges sur les participants sélectionnés. Il se composait de : 

- date inscription
- nom
- prénom
- entreprise
- date Evenement
- nom Evenement
- zone
- numéro
- nom table
- dernière version
- historique génération
- type template
- génération de badge

### Règle d'uniformisation des templates des badges

Pour faciliter la génération automatique des badges, il était essentiel d’instaurer un cadre pour les templates des badges de tous les événements. Pour éviter les erreurs de génération lors des utilisations, voici les axes d'uniformisation identifiés pour les champs à changer pour chaque participant sur les badges : 

- champs à changer et nom attribué dans les templates (<<PRENOM>>, <<NOM>>, etc.)
- style sur les champs (gras, couleur, police)
- placement des champs
- structure (A4, sur une page, a plier ou non, etc.)

Pour la génération des badges, un script Python a été mis en place pour mener les changements de champs pour chaque participant. Il était impacté pleinement par les différents changements, le cadre s'est vu créé, pour stabiliser ce script et le rendre utilisable pour les templates de chaque événement. Il était hébergé sous Google Cloud Function, ou des exécutions était mené dans des scénarios n8n. Possibilité de le retrouver ici [docs/]()

A savoir, de manière indépendante aux règles d'uniformisation, un autre script Python était hébergé sous Google Cloud Function, pour effectuer la fusion de tous les fichiers PDF générés. 

---

## Étapes du workflow

Il existe 3 scénarios créés avec le logiciel low code n8n, qui vient découper les grandes étapes pour le bon fonctionnement de la solution :

- Récupération et maj des participants entre Hello Asso et le fichier central
- Synchronisation des templates de badge présent sur Google Drive et le fichier central
- Génération des badges

### Récupération et maj des participants entre Hello Asso et le fichier central

```text
Hello Asso : Extraction API Hello Asso
        │
        ▼
Google Sheets : Intégration des participants sur le fichier central dans l’onglet “listing”
        │
        ▼
Google Drive
│   ├── Si nouvel événement -> Création du dossier de l'événement dans les dossiers "template_badge" et "stockage_badge"
│   └── Si pas nouvel événement -> Rien
        │
        ▼
Mail
│   ├── Si nouvel événement -> Notification pour prévenir la possibilité d'ajouter un template de badge pour l'événement
│   └── Si pas nouvel événement -> Rien
```

### Synchronisation des templates de badge présent sur Google Drive et le fichier central

```text
Google Drive : Intégration par l'utilisation des templates de badge de l'événement
        │
        ▼
Google Sheets : Clic de l'utilisateur sur le bouton "Refresh liste template" dans le fichier central
        │
        ▼
Apps Script : Exécution du workflow n8n
        │
        ▼
n8n / Google Sheet : Ajout dynamique des noms de template badge disponible pour chaque participant selon l'événement indiqué
```

### Génération des badges

Segmentation en 2 du workflow, pour une meilleure compréhension. Ils se déroulaient à la suite, la génération des badges individuels -> génération d'un fichier fusionnant les badges individuels créés -> Alerte par mail de la fin des générations.

#### Génération des badges individuels

```text
Google Sheets : Clic de l'utilisateur sur le bouton "Générer"
        │
        ▼
n8n / Google Sheets : Récupération des participants sélectionnés
│   ├── Conservation des lignes avec "Génération de badge" = "OUI"
│   └── Conservation des participants possédant un template
        │
        ▼
n8n : Parcours des participants à traiter
        │
        ▼
Google Sheets : Mise à jour du listing
│   ├── Réinitialisation du champ "Génération de badge"
│   ├── Réinitialisation du champ "Type template"
│   └── Ajout de la date de génération
        │
        ▼
Google Drive : Recherche du dossier de l'événement dans "stockage_badge"
        │
        ▼
Google Drive
│   ├── Si le dossier du jour existe -> Utilisation du dossier existant
│   └── Si le dossier du jour n'existe pas -> Création du dossier
        │
        ▼
Google Drive : Recherche des badges individuels existants
│   ├── Si un badge existe déjà -> Suppression du badge
│   └── Si aucun badge n'existe -> Poursuite de la génération
        │
        ▼
Google Drive : Recherche du template sélectionné et téléchargement
        │
        ▼
Google Cloud Function : Appel de la fonction, pour la génération du badge PDF individuel
        │
        ▼
Google Drive : Enregistrement du badge généré
│   └── stockage_badge / Événement / Date du jour
        │
        ▼
n8n : Fin du parcours des participants
```

#### Génération des badges individuels

```text
n8n : Récupération des événements concernés
        │
        ▼
Google Drive : Accès aux dossiers événement et date du jour
        │
        ▼
Google Drive : Récupération des badges individuels
│   ├── Suppression d'un fichier fusionné déjà présent
│   └── Téléchargement des badges individuels
        │
        ▼
n8n : Regroupement des badges et création d'une archive ZIP
        │
        ▼
Google Cloud Function : Fusion des badges PDF
        │
        ▼
Google Drive : Enregistrement du fichier "badges.pdf"
        │
        ▼
n8n : Fin de la fusion des badges pour tous les événements
        │
        ▼
Gmail : Envoi d'une notification
│   └── Confirmation de la génération des badges individuels et fusionnés
```

---

## Stack technique

| **Domaines** | **Technologies** |
|---|---|
| **Logiciel d'automatisation** | `n8n` |
| **Langage** | `Python`, `JS` |
| **API** | `Hello Asso API REST` |
| **Organisation des fichiers** | `Google Drive` |
| **Gestion des tableurs** | `Google Sheet`, `Google Apps Script` |

---

## Compétences développées

Ce projet m'a permis de développer et de mettre en œuvre mes compétences dans les domaines suivants :

- autonomnie dans la gestion d'un projet et la communication avec un client
- analyse métier et recueil des besoins
- utilisation de logiciel no code/low code
- utilisation d'API
- documentation technique et fonctionnelle

Pour ce qui est des outils, j'ai développé des compétences dans ceux qui sont présents dans la partie stack technique.
