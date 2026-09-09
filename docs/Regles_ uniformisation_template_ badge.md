# Règles / Uniformisation template badge

Pour faciliter la génération automatique des badges, il est essentiel d’instaurer un cadre pour les templates des badges.

## Le nom des champs

Intégrer une logique identique sur le nom des champs utilisés, permet de faciliter la détection automatique des parties de texte à remplacer. Proposition de l’uniformisation des champs : 

- PRENOM
- NOM
- ENTREPRISE
- VOIE
- NUMERO 
- ZONE
- NOM TABLE

## Le gras

Mettre en place d’une règle d’uniformisation sur l’utilisation du gras sur les différents champs. Proposition de gras à intégrer : 

- PRENOM = Pas gras
- NOM = Gras
- ENTREPRISE = Gras
- VOIE = Gras
- NUMERO  = Gras
- NOM TABLE = Gras

## Couleur

Eviter d’utiliser des couleurs complexes sur les champs dynamiques, tels que des dégradés. Impossibilité de répliquer la couleur avec la génération automatique.

## Police

Assigner une police par défaut pour les champs dynamiques. La partager pour que l’on puisse l’appliquer lors de la génération automatique. Lorsque qu’une police est demandé, elle sera utilisé pour tous les badges générés.

Police actuel pour la génération des badges : Gotham

## Placement du texte

Affecter par défaut un placement de texte à partir de la gauche

## Structure

Le template doit être une page, avec seulement des écritures à l’endroit, pas de version inversée/pliable. 

Sur les champs à utiliser dans les templates, prévenir dans le cas d’ajout d’un nouveau, pour que l’on puisse l’intégrer au Google Spreadsheet et à la génération automatique.


