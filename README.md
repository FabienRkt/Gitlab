# Déploiement local GitLab + Runner

Ce dépôt contient une configuration Docker Compose pour déployer une instance GitLab locale et un GitLab Runner. Fichiers concernés :
- RUNNER/docker-compose.yml  
- LOCAL/docker-compose.yml  
- RUNNER/config.toml

Prérequis
- Docker et Docker Compose installés.
- Vérifier le contenu des docker-compose.yml pour le nom du réseau ou des ports (les volumes nécessaires sont créés automatiquement par Docker Compose).

Démarrage des services
1) Démarrer GitLab
```bash
cd LOCAL
docker-compose up -d
```
- Les volumes sont créés automatiquement.
- Vérifier l'URL et le port exposé dans LOCAL/docker-compose.yml (ex: http://gitlab.local:8081).

2) Démarrer le Runner
```bash
cd RUNNER
docker-compose up -d
```
- RUNNER/docker-compose.yml définit le volume (ex: gitlab-runner-config) monté dans /etc/gitlab-runner dans le conteneur.

Enregistrement des runners
- Récupérer le token d'enregistrement depuis l'interface GitLab (Admin ou projet).
- Exemple d'enregistrement non-interactif depuis l'hôte (adapter le nom du conteneur) :
```bash
docker exec -it <container> gitlab-runner register \
  --non-interactive \
  --url "http://gitlab.local:8081" \
  --registration-token "VOTRE_TOKEN_ICI" \
  --name "workflow" \
  --executor "docker" \
  --docker-image "docker:latest" \
  --docker-network-mode "network-git" \
  --tag-list "workflow" \
  --run-untagged="true"
```

Extraction / modification / réinjection de config.toml
Objectif : copier config.toml depuis le volume géré par Docker, le modifier hors du conteneur, puis le renvoyer pour écraser la configuration dans le conteneur runner.

1) Identifier le conteneur runner
```bash
docker ps
# repérer le conteneur (ex: gitlab-runner)
```

2) Extraire config.toml depuis le conteneur vers le répertoire local RUNNER/
```bash
docker cp <container>:/etc/gitlab-runner/config.toml RUNNER/config.toml
```

3) Modifier RUNNER/config.toml localement avec votre éditeur.

4) Renvoyer le fichier modifié dans le conteneur pour écraser la configuration
```bash
docker cp RUNNER/config.toml <container>:/etc/gitlab-runner/config.toml
docker restart <container>
```

Remarques importantes
- Adaptez <container> au nom réel du conteneur défini par docker-compose (voir `docker-compose ps` ou `docker ps`).
- Sauvegardez config.toml avant toute modification.
- Évitez de committer des tokens sensibles dans le dépôt ; préférez l'enregistrement via l'interface GitLab ou l'utilisation de variables d'environnement.
- Si votre docker-compose utilise un réseau externe, créez-le au préalable (ex: `docker network create network-git`) ; sinon, Docker Compose gère le réseau automatiquement.