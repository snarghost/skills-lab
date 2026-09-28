# Zabbix — Module 02 : Socle Docker Compose

**Date :** 2026-09-28 · **Durée :** ~2 h 30

## Objectif
Monter le socle Zabbix 7.0 LTS (Long Term Support) en Docker Compose, un composant à la fois, testé avant le suivant : réseau dédié, PostgreSQL avec volume nommé, serveur, frontend.

## Concepts clés
- Réseau bridge utilisateur `zbx-core` : DNS (Domain Name System) intégré, résolution par nom de service (`postgres`, `zabbix-server`).
- Volume nommé `pgdata` : les données survivent à `down`/`up` ; sans montage, l'image crée un volume anonyme (base vide au prochain `up`). `down -v` supprime les volumes nommés.
- `.env` (ignoré par Git) : tags figés (`alpine-7.0.31`, `postgres:16-alpine`) et secrets ; `.env.example` versionné avec valeurs factices. Lu dans le répertoire du projet.
- `$VAR` = interpolé par Compose ; `$$VAR` = évalué dans le conteneur (healthcheck).
- Healthcheck `pg_isready` + `depends_on: condition: service_healthy` : ordre de démarrage au `up` uniquement ; ensuite le serveur gère seul sa reconnexion.
- Premier démarrage : le serveur importe le schéma (203 tables) ; ensuite il lit la table `dbversion` pour vérifier ou mettre à jour le schéma.
- Frontend : deux flux sortants, vers la base (5432/TCP) et vers le serveur (10051/TCP).
- Publication : base et serveur non publiés (seulement « exposés ») ; frontend publié sur `127.0.0.1:8080` uniquement. Les ports publiés par Docker contournent UFW (Uncomplicated Firewall).
- nginx non-root dans le conteneur → écoute sur 8080 (ports < 1024 privilégiés).

## Ce que j'ai fait dans le lab
```bash
cd ~/projets/skills-lab/zabbix/lab
cp .env.example .env
sed -i "s/^POSTGRES_PASSWORD=.*/POSTGRES_PASSWORD=$(openssl rand -hex 20)/" .env
git check-ignore -v .env
docker compose up -d postgres
docker compose up -d zabbix-server
docker compose logs -f zabbix-server
docker compose up -d zabbix-web
```

## Vérifications
- `docker compose ps` : postgres (healthy, 5432/tcp exposé), zabbix-server (10051/tcp exposé), zabbix-web (healthy, 127.0.0.1:8080->8080/tcp).
- `docker compose exec zabbix-server getent hosts postgres` → 172.18.0.2.
- Schéma : 203 tables dans `public`.
- `curl -sI http://127.0.0.1:8080` → HTTP/1.1 200 OK ; interface : « Zabbix server is running : Yes ».

## Ce que je retiens (en mes mots)
- Le serveur joint la base par son nom de service, parce qu'ils sont sur le même réseau utilisateur.
- Les données vivent dans le volume, pas dans l'image : volume nommé obligatoire pour la base.
- depends_on ne sert qu'au démarrage ; une coupure de base rend la supervision aveugle pour tout le périmètre (point unique de défaillance).
- Je ne publie un port que si un client hors Docker en a besoin, et sur une adresse explicite.
- Serveur arrêté : le frontend affiche encore des données, mais figées.
- Compte Admin par défaut conservé dans ce lab (écoute sur 127.0.0.1 uniquement) ; en production, changé dès l'installation.

## Question type entretien
**Q :** Vous déployez Zabbix avec Docker Compose. Quels choix faites-vous pour la base de données et pour l'exposition réseau ?
**Éléments de réponse :**
- Volume nommé pour PostgreSQL + sauvegardes régulières (pg_dump) testées ; la base est le point unique de défaillance.
- Healthcheck + depends_on service_healthy pour l'ordre de démarrage ; reconnexion gérée par le serveur ensuite.
- Réseau dédié, base et serveur non publiés ; 10051 publié seulement pour les proxys/agents externes, sur une IP d'interface filtrée, avec PSK (Pre-Shared Key).
- Frontend derrière un reverse proxy HTTPS ; attention aux ports Docker qui contournent UFW.
- Tags d'images figés (pas de latest), secrets hors du dépôt (.env ignoré), compte Admin changé puis remplacé par des comptes nominatifs.
