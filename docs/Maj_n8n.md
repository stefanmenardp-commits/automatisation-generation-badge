# MAJ N8N

Pour se connecter au serveur contenant N8N, création d’une clé SSH publique à ajouter pour accèder a ce dernier.

Voici les différentes étapes à mener : 

- Création de la clé SSH publique de l’ordinateur voulant se connecter au serveur
- Donner les droits de connexion à la clé SSH publique sur le serveur
- Sur le CLI du serveur : 
    - Se diriger sur le dossier contenant le docker compose qui permet d’héberger N8N
        - Détecter le dossier : _find / -name docker-compose.yml 2>/dev/null_
        - Se placer dans le dossier : cd xx
    - Vérification de la version du N8N hébergé
        - _docker exec -it n8n-docker-caddy-n8n-1 n8n --version_
    - Enregistrer les workflows existants sur N8N
        - _docker exec -it n8n-docker-caddy-n8n-1 \  
n8n export:workflow --all --output=/tmp/workflows.json_
    - Vérifier la manière d’intégrer la version de N8N sur le docker compose
        - _nano  docker-compose.yml_
    - SI image = “image: caddy:latest” \(MAJ qui va prendre automatiquement la version la plus récente\) → Pas optimisé, car peut prendre des versions Beta qui ne sont pas stables
        - _docker compose pull_
        - _docker compose down_
        - _docker compose up -d_
    - SI image = “image: caddy:1.86” → Intégration manuelle de la version à sélectionner
        - _nano  docker-compose.yml → Changer la version_
        - _docker compose pull_
        - _docker compose down_
        - _docker compose up -d_
    - Vérification de la version du N8N hébergé
        - _docker exec -it n8n-docker-caddy-n8n-1 n8n --version_


