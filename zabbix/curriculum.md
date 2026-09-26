# Curriculum — Zabbix (7.0 LTS)

Notions annexes révisées : Docker/Compose, réseaux Docker, PostgreSQL, SNMP, TLS/PSK, reverse proxy.

| Module | Titre | Contenu | Livrable lab |
|---|---|---|---|
| 0 | Diagnostic | Environnement, niveau Docker, adaptation du parcours | Profil complété |
| 1 | Fondamentaux | Rôle de la supervision ; serveur, base, frontend, agent 2, proxy ; flux et ports ; Centreon/Prometheus | Schéma d'architecture |
| 2 | Socle Docker Compose | Réseau dédié, PostgreSQL + volume, serveur, frontend, tests à chaque ajout | docker-compose.yml v1 |
| 3 | Premier hôte | Agent 2, checks passifs/actifs, zabbix_get, logs | Hôte supervisé |
| 4 | Collecte et affichage | Items, triggers, templates, macros, graphiques, dashboards | Template personnalisé |
| 5 | Alerting | Actions, médias, escalades, maintenances, dépendances | Alerte fonctionnelle |
| 6 | Réseau | SNMP v2c puis v3 (conteneur snmpd), Low-Level Discovery | Équipement SNMP supervisé |
| 7 | Proxy | Site distant sur réseau Docker séparé, proxy actif, coupure de lien | Architecture multi-site |
| 8 | Sécurisation | PSK/TLS, comptes et rôles, HTTPS via reverse proxy, secrets | Lab durci |
| 9 | Synthèse | Proposer une architecture pour une organisation multi-sites, critique type recruteur | Fiche de révision + README final |
