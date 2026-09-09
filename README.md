# Création automatique de badges PDF pour les participants aux événements

Automatisation pour générer des badges pour des événements

## Avis de confidentialité

Ce projet a été mené dans un cadre professionnel.

Le code source, les données de production, les informations relatives aux clients et les captures d'écran originales ne sont pas accessibles au public pour des raisons de confidentialité.

Ce référentiel présente la portée du projet et l’architecture technique.

Certains diagrammes et illustrations ont pu être recréés à partir d’informations anonymisées ou fictives.

## Table des matières

- [Présentation du projet](#aperçu-projet)
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

Dans ce cas là nous détaillerons 

---

## Contexte métier

La génération manuelle des badges représentait une tâche chronophage et source d’erreurs, notamment lorsque plusieurs événements étaient organisés simultanément.
Le besoin consistait à permettre aux utilisateurs de :

Récupérer automatiquement les participants inscrits ;
Associer un template à chaque événement ;
Générer rapidement les badges au format PDF ;
Regrouper les badges dans un fichier unique ;
Identifier les éventuels homonymes ;
Retrouver facilement les fichiers générés.

---

## Objectifs

Automatiser la récupération des participants depuis HelloAsso ;
Identifier les événements nécessitant la génération de badges ;
Centraliser les templates et les badges générés ;
Permettre une sélection dynamique des templates ;
Générer automatiquement les badges individuels ;
Fusionner les badges dans un fichier PDF global ;
Détecter les homonymes ;
Supprimer automatiquement les anciennes données ;
Notifier les utilisateurs à la fin des traitements ;
Documenter le fonctionnement de la solution.

## Étapes du workflow

## Stack technique

## Compétences développées
