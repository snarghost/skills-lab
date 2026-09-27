# Zabbix — Module 01 : Fondamentaux

**Date :** 2026-09-26 → 2026-09-27 · **Durée :** ~2 h · **Version :** Zabbix 7.0 LTS (7.0.31)

## Objectif
Comprendre le rôle de la supervision, les composants de Zabbix, les flux et les ports, le fonctionnement du proxy, situer Zabbix face à Centreon et Prometheus, et produire le schéma d'architecture de référence.

## Concepts clés
- Supervision : détecter avant l'utilisateur, alerter la bonne personne, historiser pour diagnostiquer et anticiper.
- Serveur : collecte (pollers), réception (trappers), évaluation des triggers, actions et alertes, housekeeping. Seul composant qui juge et alerte.
- Base (PostgreSQL) : configuration, history, trends, événements. Point critique de toute l'architecture.
- Frontend : interface web et API JSON-RPC, lit et écrit en base, contacte le serveur (10051). Ne collecte rien.
- Agent 2 (Go) : passif = écoute sur 10050 ; actif = se connecte au 10051. Plugins, tampon persistant optionnel. `Hostname` doit correspondre au nom déclaré.
- Pollers vs trappers : aller chercher vs recevoir. Pollers agent/HTTP/SNMP asynchrones en 7.0. Saturation surveillée via `zabbix[process,poller,avg,busy]` (> 75 %).
- Triggers : expression sur les valeurs d'un item, sévérités, événements, recovery expression (hystérésis), dépendances.
- History vs trends : valeurs brutes gardées peu de temps / agrégats horaires min, avg, max, count gardés longtemps. Arbitrage volume et performance.
- Proxy : collecte déléguée, base locale (SQLite), tampon (`ProxyOfflineBuffer`, `ProxyBufferMode` en 7.0). N'évalue pas les triggers, n'alerte pas. Actif = flux sortant vers 10051 du serveur. Proxy groups en 7.0.
- Alternatives : Centreon (modèle à états, plugins Nagios, « poller » = serveur de collecte), Prometheus (pull HTTP, exporters, PromQL, Kubernetes). Complémentaires avec Grafana.

## Ce que j'ai fait dans le lab
```bash
# Ports Zabbix enregistrés dans la base locale des services
getent services 10050 10051
# Image du proxy 7.0 LTS et vérification de la version (7.0.31)
docker run --rm --entrypoint zabbix_proxy zabbix/zabbix-proxy-sqlite3:alpine-7.0-latest -V
# Schéma d'architecture multi-sites (Mermaid)
cat zabbix/docs/architecture.md
```

## Vérifications
- `getent` : zabbix-agent 10050/tcp et zabbix-trapper 10051/tcp.
- `zabbix_proxy -V` : Zabbix 7.0.31, OpenSSL 3.5.8, licence AGPLv3.
- `zabbix/docs/architecture.md` : bloc Mermaid présent, rendu vérifié.

## Ce que je retiens (en mes mots)
- Le frontend est une vitrine : s'il tombe, collecte et alertes continuent.
- Sans la base, plus d'alertes fiables pour tout le périmètre : c'est le point unique de défaillance le plus grave, pas le lien d'un site.
- En passif, l'agent ne collecte rien seul, il répond. Le mode actif est obligatoire pour les logs.
- Pour le pare-feu, seul compte qui initie la connexion. Proxy actif = une seule règle par site (IP du proxy → serveur 10051/TCP).
- Coupure de lien avec proxy : pas de perte si elle reste sous `ProxyOfflineBuffer`, mais alertes en retard. `nodata()` ne se déclenche pas, seule l'alerte « proxy injoignable » doit sortir.
- Un proxy web relaie des requêtes, un proxy Zabbix exécute la collecte à la place du serveur.
- On fige la version des images (7.0.31) pour un lab reproductible.

## Question type entretien
**Q :** Vous devez superviser un siège et plusieurs sites distants reliés par des liaisons parfois instables. Quelle architecture Zabbix proposez-vous et quels flux demandez-vous à l'équipe réseau ?
**Éléments de réponse :**
- Serveur Zabbix, PostgreSQL et frontend au siège, frontend derrière un reverse proxy HTTPS.
- Un proxy actif par site : une seule règle par site, IP du proxy → serveur 10051/TCP, flux sortant, rien à exposer sur le site.
- Flux 10050 et SNMP 161/UDP locaux au site, entre le proxy et les hôtes. Mode actif pour les logs, voire 100 % actif pour ne rien exposer sur les hôtes.
- Liaisons instables : `ProxyOfflineBuffer` dimensionné avec marge, `ProxyBufferMode` disk ou hybrid, disque vérifié. Pas de perte, alertes différées.
- Anti-bruit : trigger « proxy injoignable », `nodata()` conscient du proxy, dépendances de triggers.
- Résilience : la base est le point critique (supervision, sauvegarde, redondance), HA native du serveur, proxy groups (7.0) pour les sites importants.
- Chiffrement PSK sur les mêmes ports, déploiement des agents et proxies par Ansible.
