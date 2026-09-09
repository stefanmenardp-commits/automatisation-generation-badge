# Résumé global du projet

# Présentation

Le service proposé à travers ce projet, provient du besoin d’automatiser la génération de badge pour les participants des événements. Cela prend en compte la génération de badge pour plusieurs événements différents.

Les données des participants et événements proviennent directement de Hello Asso, l’outil utilisé pour la gestion des événements. La génération de badge se fait à l’aide de template spécifique à chaque événement, ces derniers sont conçus en interne.

Pour ce qui est de la technique, on retrouve 3 outils majoritaires : 

- **N8N** → Automatisation des différents processus entre les services/outils
    - **Cloud function** → Code spécifique pour des tâches complexes ou non accessible sur N8N \(génération de badge \+ Fusion des badges\)
- **Google Drive** → Stockage des différents fichiers \(template, badge généré et listing des participants\)
- **Google Sheet → **Listing des participants et sélection pour génération de badge
    - **Google Apps Script **→ Code spécifique lié au Sheet OU lien du Sheet avec N8N \(Versionning dynamique des templates \+ Génération des badges \+ Contrôle des homonymes\)

# Structure du stockage - Google Drive

Le stockage des différentes entités va s’effectuer sous Google Drive, voici l’architecture choisie : 

- template\_badge \(Dossier\)
    - Evenement\_nomEVENEMENT\_dateEVENEMENT \(sous dossier Template\_badge\) -> Créé automatiquement
        - badge\_template\_x
        - badge\_template\_x
- stockage Badge \(Dossier\)
    - Evenement\_nomEVENEMENT\_dateEVENEMENT \(sous dossier de Stockage badge\) -> Créé automatiquement
        - yyyymmdd \(sous dossier de Evenement\_nomEvenement\_dateEVENEMENT\)
            - badge\_jean\_eudes\_yyyymmdd.pdf \(Badge individuel\)
            - badge\_john\_doe\_yyyymmdd.pdf \(Badge individuel\)
            - badges.pdf \(Badges fusionnés\)
- listing\_participants.gsheet \(Google Spreadsheet\)

# Utilisation - côté client

Pour fonctionner correctement, ce service requiert une action de la part de l'utilisateur. Il offre par ailleurs diverses fonctionnalités qui répondent aux besoins pour lesquels il a été conçu. Voici les détails :

- Catégorisation des événements Hello Asso
- Intégration des templates dynamiques à la sélection
- Génération des badges \+ Téléchargement

## Catégorisation des événements Hello Asso

La catégorisation des événements est essentielle pour l’extraction unique de ceux avec une nécessité de génération de badge. La solution est l’ajout d’une option “Badge” gratuite aux tarifs de chaque événement concerné. C’est la seule action obligatoire à réaliser côté utilisateur, pour le bon fonctionnement du service.

Ajout de l’option “Badge” : 

- Dans “Billeterie”, sélectionner “Administrer” sur l’événement concerné
- Se diriger sur l’étape 2 “Tarifs et options” puis dans l’onglet “Options”
- Ajouter une option
    - Type d’option : Gratuite 
    - Nom de l’option : Badge
    - Tarifs concernés : Tous les tarifs
    - Option obligatoire : A cocher
- Enregistrer les modifications apportées à l’événement

## Intégration des templates

Cette intégration des templates se passe sur le stockage Drive, dans le dossier “stockage\_template”. Dans ce même dossier, il y sera créé automatiquement les dossiers des événements. L’action utilisateur se voit être l’intégration des templates pour chaque événement.

Le service propose de générer des badges pour de multiples événements, ce qui se traduit par la présence de plusieurs dossiers associés à ces événements.

Concernant la structure des templates, il est indispensable de suivire une charte d’homogénéisation. Ce besoin de d’uniformisation s’explique par l’automatisation, empêchant une personnalisation extrême. Ces règles interviennent sur ces sujets, détaillant les régles\) : 

- Nom des champs dynamiques
- Gras sur les champs dynamiques
- Les couleurs
- La police
- La mise en page

Pour mettre à jour la liste des templates \(Type template - colonne K\) dans l’onglet “listing” du fichier Spreadsheet “listing\_participant”, il faut : 

- Ajout de template sur les dossiers d’événements
- Cliquer sur le bouton “Refresh liste template” dans l’onglet “listing”
- Attendre le mail précisant que les templates sont bien tous refresh
- Refresh le fichier Spreadsheet dès la fin du reset de la liste des templates sur chaque participant

## Génération des badges \+ Téléchargement

A propos de la génération des badges, l’utilisateur se basera uniquement sur un fichier Google Spreadsheet, du nom de listing\_participant.  Son utilisation réside dans l’identification des participants avec le besoin de génération de badge. Lorsque c’est fait, il suffit : 

- D’indiquer les champs manuels \(Voie, Numéro et Nom table - Colonne F, G et H\)
- De sélectionner les templates sur lesquels baser la génération des badges \(Type template - Colonne K\)
- De passer à “OUI” pour la génération des badges \(Génération de badge - Colonne L\)
- Cliquer sur le bouton “Générer”
- Attendre de recevoir la notification par mail signifiant la fin de la génération. Se diriger dans “stockage\_badge”, puis vers les événements concernés et la date du jour.
- Téléchargement des fichiers des badges fusionnés \(badges\) puis impression

# Authentification des entités

Pour ce qui est de l’authentification des différentes entités pour exécuter les différentes étapes : 

- Modules Google N8N \(Drive, Spreadsheet\) \(sur le compte Google du client\) → Service Account
- Module Google N8N \(Drive\) - Upload Generate PDF and merge PDF → OAuth - Projet GCP \(n8n-hub-automatisation\) - Externe Production
- Module N8N Mail \(Gmail d’xxx\) \(temporaire, à changer\) → OAuth - Projet GCP - Interne
- Exécution script Apps Script sur le compte Google du client → OAuth - Projet GCP \(n8n-hub-automatisation\) - Externe Production
- Exécution Cloud function \(Create Badge et Badge merger\) → Projet GCP \(xxx-PROJECT-DATA\)

# Différents processus - côté back

## Process 1 - Intégration des templates dynamiques à la sélections

Le processus 1 répond au besoin d’obtenir dynamiquement les templates lors de la sélection de ces derniers sur le fichier listing\_participant. La structure technique de ce processus se base sur : 

- **Drive** → Intégration des templates par l’utilisateur
- **N8N** → Workflow qui permet de récupérer tous les noms des templates du Drive pour chaque événement
    - Déclencheur automatique : Tous les jours à 4h
    - Déclencheur manuel : Au clic du bouton Refresh liste template dans l’onglet “listing” du fichier Spreadsheet “listing\_participant” - Besoin de refresh à la fin
    - Outils utilisés : 
        - Google Spreadsheet
        - Drive
        - Apps Script
- **Apps Script** → Intégration de la sélection dynamique des templates sur chaque participation via Validation de données \(Fichier Apps script “Type template dynamique.gs” ET fonction main “applyTemplateValidation”\)
    - Déclencheur : La fin du Workflow de N8N

**Détail sur le fonctionnement du Workflow N8N _\(Nom workflow : xxx - Template dynamique\)_ : **

- Récupération des lignes des templates existants sur l’onglet “template” du fichier listing\_participant
- Suppression des lignes avec les templates existants
- Récupération sur le Drive de tous les sous dossiers des événements du dossier “stockage\_template”
- Boucle pour parcourir tous les dossiers
    - Récupération de tous les fichiers template dans les sous dossiers 
    - Transformation des données pour obtenir un tableau avec le nom du template ET l’événement lié
- Intégration de tous les événements et les templates dans l’onglet “template”
- Exécution du script Apps Script qui va intégrer la sélection adaptée des templates pour chaque participant. Utilisation de la validation de donnée, pour éviter à l’utilisateur d’intégrer un template qui n’existe pas.

## Process 2 - Récupération des participants ET suppression des événéments passés

Le processus 2 se présente sous 2 phases, la récupération automatique des derniers participants d’Hello Asso sur les événements cibles à la génération de badge. Puis, la gestion des données existantes, avec la suppression des participants et événements passés.

La structure technique se présente sous cette forme : 

- **N8N** → Workflow exécutant les 2 phases
    - Déclencheur : 1 fois par jour à 1h du matin
    - Outils utilisés : 
        - Google Spreadsheet 
        - Drive
        - Gmail
        - API Hello Asso

**Détail sur le fonctionnement du Workflow N8N - Phase 1 - Récupération des participants _\(Nom workflow : xxx - Récupération/Suppression Participant Hello Asso\)_ :**

- Extraction API Hello Asso
    - Récupération de l’Access Token
    - Récupération des orders à J-1 \(participants\) 
    - Récupération des événements \(événement concerné par la génération des badges - via l’option “Badge” sur Hello Asso\)
    - Transformation, Merge et filtre de donnée pour obtenir tous les participants en lien avec les événements cibles pour la génération de badge
- Intégration des participants sur le fichier Spreadsheet dans l’onglet “listing”
- Récupération des valeurs uniques des événements liés aux participants intégrés \(Utile pour la création des dossiers sur le Drive\)
- Boucle pour parcourir les dossiers du Drive pour l’intégration des nouveaux événements sous forme de dossier dans le Drive \(dans les dossiers “stockage\_badge” et “template\_badge”\)
- Création du nom de dossier \(Evenement\_nomEvenement\_dateEvenement\) - Pour tester si déjà présent OU l’intégrer
    - Uniformation du nom d’événement \(suppression des espaces, caractères spéciaux et en minuscule\)
    - Uniformisation de la date d’événement \(passage en string et suppression des “-”\)
- Analyse des dossiers Drive \(“stockage\_badge” et “template\_badge”\)
    - Si événement déjà présent - Aucune action
    - Si événement non présent - Création du dossier et notification de la xxx par mail \(pour les mettre au courant de la possibilité d’ajouter les templates de badge\)

**Détail sur le fonctionnement du Workflow N8N - Phase 2 - Gestion des données existantes :**

- Récupération des participants sur le fichier Spreadsheet, de l’onglet “listing”
- Filtre pour obtenir uniquement les participants avec une date d’événement supérieure à J\+5 par rapport à la date du jour d’éxécution
- Suppression des participants sur l’onglet “listing” du fichier Spreadsheet
    - Trie des participants à supprimer - Pour faciliter la suppression
    - Suppression des participants
- Récupération des valeurs uniques des événements liés aux participants intégrés \(Utile pour la suppression des dossiers sur le Drive\)
    - Détection et suppression du dossier événement dans le Drive \(dans les dossiers “stockage\_badge” et “template\_badge”\)

## Process 3 - Génération des badges

Le processus 3 se présente également sous 2 phases, la génération de badge individuel, puis la génération d’un badge merge reprenant tous les badges individuels.

La structure technique se présente sous cette forme : 

- **N8N** → Workflow exécutant les 2 phases
    - Déclencheur : Au clic du bouton “Générer” dans l’onglet “listing” du fichier Spreadsheet “listing\_participant”
    - Outils utilisés : 
        - Google Spreadsheet  
        - Drive
        - Gmail
        - Google Cloud Function

**Détail sur le fonctionnement du Workflow N8N - Commun aux 2 phases _\(Nom workflow : xxx - Génération de badge\)_ : **

- Récupération des participants de l’onglet “listing” du fichier Spreadsheet - Seulement ceux à OUI pour la génération de badge
- Filtre pour garder uniquement les participants avec un type de template indiqué
- Boucle pour parcourir chaque participant - Pour effectuer les 2 phases

**Détail sur le fonctionnement du Workflow N8N - Phase 1 - Génération des badges individuels :**

- Suppression des valeurs de génération sur l’onglet “listing” - Passage à vide pour la génération de badge et le type de template - Indication de la date de génération pour l’historique
- Analyse du dossier Drive \(“stockage\_badge”\) - Récupération dossier date du jour ET nettoyage des badges individuels
    - Récupération du dossier de l’événement lié au participant
    - Récupération du sous dossier avec la date du jour
        - Si dossier avec date du jour déjà présente - Suite workflow
        - Si dossier avec date du jour non présent - Création du dossier
    - Analyse des badges individuels 
        - Si badge déjà généré pour le participant - Suppression
        - Si badge pas généré - Suite workflow
- Analyse du dossier Drive \(“template\_badge”\) - Récupération du template sélectionné pour la génération du badge du participant
    - Récupération du dossier de l’événement lié au participant
    - Récupération et téléchargement du fichier template sélectionné
- Génération du badge individuel - Appel de la Cloud Function
- Intégration du badge individuel dans le dossier de la date du jour de l’événement \(“stockage\_badge” → Evenement → Date du jour\)

**Détail sur le fonctionnement du Workflow N8N - Phase 2 - Génération des badges merges :**

- Phase suivant la fin des itérations de la boucle \(Done\)
- Récupération des valeurs uniques des événements liés aux participants intégrés \(Utile pour effectuer la génération des badges pour chaque événement\)
- Analyse du dossier Drive \(“stockage\_badge”\) - Récupération des dossiers des événements et des dates du jour
    - Récupération du dossier de l’événement
    - Récupération du dossier de la date du jour
- Boucle pour parcourir tous les fichiers des dossiers de date du jour
    - Récupération et suppression si présence d’un fichier fusionné déjà présent
    - Récupération et téléchargement de tous les badges individuels existants
    - Transformation de données pour rassembler tous les fichiers dans une même entité et passage sous fichier ZIP
    - Génération du badge fusionné - Appel de la Cloud Function
    - Intégration du badges fusionné dans le dossier Drive de la date du jour de l’événement traité
- Phase suivant la fin des itérations de la boucle \(Done\)
    - Aggrégation pour obtenir un item \(Pour effectuer qu’une seule fois l’action qui va suivre\)
    - Notification de la xxx par mail \(pour les mettre au courant de la fin de la génération des badges individuels et fusionnés\)

## Process 4 - Contrôle des homonymes

Ce contrôle permet d'identifier les participants ayant des noms et prénoms identiques et de les mettre en avant en les affichant en priorité et en leur attribuant une couleur distinctive.

Déclenchement de ce script \(Fichier Apps script “Controle homonyme.gs” ET fonction main “detectAndSortHomonymsColors”\) entre 2h et 3h. Cela s’exécute 1h après le processus 2 récupérant les participants d’Hello Asso.


