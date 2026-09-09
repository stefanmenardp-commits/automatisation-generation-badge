# Procédures de test

Suivi du protocole de test des différents processus mis en place.

## Process 1 - Intégration des templates dynamiques à la sélections


- En amont, créer des participants tests avec des événements existant dans le dossier Drive “template\_badge”
- Ajouter un fichier PDF dans le dossier de l’événement \(présent dans le dossier “template\_badge”\) - Tester avec un ou plusieurs templates et événements différents \(tester une version avec un ajout ET multiple\)
- Exécuter le workflow N8N _French tech - Template dynamique_
- A la fin de l’éxécution - Vérifier la colonne Type template \(colonne K\) sur les participants de test \(sur l’onglet “listing” du fichier Spreadsheet “listing\_participant”\). Les participants se doit d’avoir en validation de donnée la proposition des templates liés à leur événement.

## Process 2 - Récupération des participants ET suppression des événéments passés


**Teste de la récupération des participants : **

- Exécuter le workflow N8N _French tech - Récupération/Suppression Participant Hello Asso_
- A la fin de l’éxécution - Vérifier la présence de nouvelles données
- Comparer les données ajoutés dans l’onglet “listing” du fichier Spreadsheet “listing\_participant” avec les données sur l’interface d’Hello Asso 
    - Sur l’interface administrateur d’Hello Asso → Accueil → Descendre et sélectionner l’événement souhaité → Cliquer sur “Voir mes participants”

**Teste de l’intégration des dossiers d’un nouvel événement \(tester une version avec un ajout ET multiple\)**

- Intégrer des participants avec un événement non présent OU supprimer les dossiers événements déjà présent \(Dans les dossiers Drive “template\_badge” et “stockage\_badge”\)
- Intégrer les participants - Exécuter le workflow N8N _French tech - Récupération/Suppression Participant Hello Asso_
- Vérifier la présence des dossier événements nouveaux dans les dossiers Drive “stockage\_badge” et “template\_badge”
- Vérifier la réception d’un mail notifiant la création d’un nouvel événement dans le dossier “template\_badge”

**Teste de la suppression des entités avec des événements passés \(participants et événements\) \(tester une version avec un ajout ET multiple\) : **

-  Intégrer / modifier la date d’événement de participant dans l’onglet “listing” du fichier Spreadsheet “listing\_participant”
- Intégrer un dossier drive de l’événement / Modifier le nom du dossier de l’événement dans “stockage\_badge” et “template\_badge”
- Exécuter le workflow N8N _French tech - Récupération/Suppression Participant Hello Asso_
- A la fin de l’éxécution - Vérifier la non présence des participants dans l’onglet “listing” et des dossiers événements dans les dossiers “stockage\_badge” et “template\_badge”

## Process 3 - Génération des badges


**Tester la génération des badges \(tester une version avec un ajout ET multiple\) :**

- En amont, ajouter des participants, avec des dossiers d’événements \(dans les dossiers “template\_badge” et “stockage\_badge”\) et avec des templates de badge \(dans les dossiers événements du dossier “template\_badge”\)
- Exécuter le workflow N8N _French tech - Génération de badge_
- A la fin de l’éxécution - Contrôler : 
    - La présence du/des badges individuels dans le dossier “stockage\_badge” → Evenement concerné → Date du jour ET du contenu \(template correct, champ complété, police, gras, etc.\)
    - La présence du fichier avec les badges merged, dans le même dossier ET du contenu
    - La réception d’un mail notifiant la fin de la génération des badges

## Process 4 - Contrôle des homonymes


- Créer des participants fictifs avec le même nom et prénom \(tester une version avec un cas ET multiple\)
- Ouvrir Apps Script du fichier Spreadsheet “listing\_participant” et éxécuter la fonction “detectAndSortHomonymsColors” provenant du fichier “Controle homonyme.gs”


