# Zabbix — supervision d'infrastructure
> Lab Zabbix 7.0 LTS (Long Term Support) sous Docker Compose. Il sera intégré au projet de supervision de Kerlab (P3, dépôt `monitoring-zabbix`).

## Périmètre actuel
- Serveur Zabbix 7.0 avec PostgreSQL 16 et interface web Nginx.
- Interface web publiée uniquement en local (`127.0.0.1:8080`).
- Données persistées dans un volume nommé.

## Architecture
Architecture cible multi-sites et choix techniques : [docs/architecture.md](docs/architecture.md).

## Lancer le lab
```bash
cd lab
cp .env.example .env   # renseigner les mots de passe
docker compose up -d
```

## Évolutions
Les agents Windows et Linux, SNMP (Simple Network Management Protocol), le proxy à l'agence, l'alerting et le chiffrement des flux seront réalisés dans le dépôt `monitoring-zabbix` (P3).
