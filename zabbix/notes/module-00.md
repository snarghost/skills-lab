# Zabbix — Module 00 : Diagnostic

**Date :** 2026-09-26 · **Durée :** ~1 h

## Objectif
Valider l'environnement de lab, mesurer le niveau Docker et adapter le parcours à l'entretien du 1er octobre 2026.

## Concepts clés
- Stack Zabbix : PostgreSQL, serveur, frontend, agent 2, proxy. Ports : 10050 (agent, checks passifs), 10051 (serveur/trappeur, checks actifs et proxy actif), 5432 (PostgreSQL, jamais publié).
- Compose intégré (`docker compose`) : le binaire `docker-compose` v1 est abandonné.
- Volumes : sans volume nommé, une image qui déclare `VOLUME` crée un volume anonyme orphelin.
- DNS Docker : résolution par nom de service, uniquement sur un réseau défini par l'utilisateur.
- `expose` est documentaire ; `ports` publie sur 0.0.0.0 par défaut (préférer 127.0.0.1:port:port en lab).
- Git : un secret commité reste dans l'historique ; rotation d'abord, puis réécriture.
- SSH : la clé privée signe un défi, l'empreinte dans known_hosts protège contre l'homme du milieu.
- Version retenue : Zabbix 7.0 LTS (5 ans de support ; 8.0 encore en bêta mi-2026).
- Proxy : collecte déléguée par site, tampon local si le WAN coupe, mode actif = flux sortant unique vers 10051, n'évalue pas les triggers.

## Ce que j'ai fait dans le lab
```bash
wsl --version && wsl -l -v                 # PowerShell : distro en VERSION 2
lsb_release -ds; ps -p 1 -o comm=          # Ubuntu 24.04.5, systemd actif
nproc; free -h; df -h ~                    # 4 vCPU, 7,8 Go, ~950 Go libres
docker version; docker compose version     # 29.8.1 / Compose v5.5.1
docker run --rm hello-world
ss -ltn | grep -E ':(80|8080|10050|10051|5432)\b' || echo "ports libres"
git -C ~/projets/skills-lab remote -v      # remote SSH
```

## Vérifications
- Environnement validé : WSL2, systemd, cgroup v2, groupe docker, ports libres.
- Dépôt `~/projets/skills-lab` propre et synchronisé, `.gitignore` couvrant `.env`, clés et données.

## Ce que je retiens (en mes mots)
- Vérifier l'invite (PowerShell ou Ubuntu) avant de coller un bloc.
- Une base de données a toujours un volume nommé et n'est jamais publiée sur l'hôte.
- Choisir une LTS pour la durée de support et le rythme de migration.
- Le proxy est une décision d'architecture multi-sites, pas un détail.

## Plan accéléré (entretien jeudi 1er octobre)
S1 architecture · S2-S4 socle, agent, templates · S5 alerting · S6-S7 SNMP, proxy, sécurisation · S8 synthèse et simulation.

## Question type entretien
**Q :** Vous devez superviser un siège et plusieurs sites distants reliés par un WAN. Comment placez-vous les composants Zabbix ?
**Éléments de réponse :** serveur et base au siège ; un proxy par site distant en mode actif (seul flux sortant vers 10051, pas d'ouverture entrante, compatible NAT) ; tampon local du proxy pendant une coupure WAN ; agents configurés vers leur proxy ; triggers et alertes restent sur le serveur ; groupes de proxies (7.0) pour la redondance ; PSK/TLS entre proxy et serveur ; choix d'une LTS pour le support long.
