# Feuille de route

## Kerlab, entreprise fictive
Kerlab est une PME (petite ou moyenne entreprise) fictive d'environ 80 salariés : un **siège**, une **agence**, puis une **extension dans le cloud AWS** (Amazon Web Services). Chaque projet construit une brique de son SI (système d'information) et s'appuie sur les précédentes : la supervision surveille le réseau, le SIEM (Security Information and Event Management) collecte les journaux de l'Active Directory, Ansible redéploie ce qui a d'abord été fait manuellement.

**Conventions**
- Domaine interne : `kerlab.internal`. Le TLD (Top-Level Domain) `.internal` est réservé à l'usage privé.
- Domaine Active Directory : `ad.kerlab.internal`.
- Adressage RFC 1918, sans aucune donnée réelle :

| Zone | Plage |
|---|---|
| Siège | `10.10.0.0/16` |
| Agence | `10.20.0.0/16` |
| AWS | `10.30.0.0/16` |
| Interconnexion opérateur simulée | `172.16.0.0/24` |

## Projets

### P0 — Socle de virtualisation Hyper-V
- **v1 :** commutateurs virtuels, VLAN (Virtual LAN), images de référence, déploiement des VM (machines virtuelles) par PowerShell, exploitation courante.
- **v2 :** cluster de basculement imbriqué, stockage iSCSI (Internet Small Computer Systems Interface), Live Migration.
- **v3 :** images construites avec Packer.

### P1 — Réseau multi-sites
- **v1 :** OPNsense au siège : VLAN, routage inter-VLAN, DHCP (Dynamic Host Configuration Protocol), DNS (Domain Name System), filtrage.
- **v2 :** agence, VPN (Virtual Private Network) site-à-site IPsec ou WireGuard, OSPF (Open Shortest Path First) avec FRRouting.
- **v3 :** haute disponibilité avec CARP (Common Address Redundancy Protocol), IDS/IPS (détection et prévention d'intrusion) Suricata, configuration versionnée.

### P2 — Identité et services Windows
- **v1 :** AD DS (Active Directory Domain Services) avec deux contrôleurs de domaine, DNS, DHCP, OU (unités d'organisation), GPO (Group Policy Objects), partages NTFS et DFS (Distributed File System).
- **v2 :** modèle en tiers, Windows LAPS (Local Administrator Password Solution), gMSA (group Managed Service Accounts), PKI (Public Key Infrastructure) avec AD CS (Active Directory Certificate Services).
- **v3 :** sauvegarde Veeam et PRA (plan de reprise d'activité) testé, hybridation Entra ID.

### P3 — Supervision
- **v1 :** Zabbix LTS (Long Term Support), agents Windows et Linux, templates, alerting.
- **v2 :** SNMP (Simple Network Management Protocol), proxy à l'agence, LLD (Low-Level Discovery), dépendances entre déclencheurs.
- **v3 :** Prometheus, Grafana et Loki ; comparaison avec Centreon.

### P4 — Sécurité et détection
- **v1 :** Wazuh : agents, journaux de l'Active Directory et d'OPNsense, FIM (File Integrity Monitoring), SCA (Security Configuration Assessment) selon les benchmarks CIS (Center for Internet Security).
- **v2 :** alertes Suricata dans Wazuh, règles personnalisées, réponse active.
- **v3 :** scénarios d'attaque simulés et détection documentée.

### P5 — Automatisation et IaC (Infrastructure as Code)
- **v1 :** inventaire Kerlab, rôles Linux et Windows, durcissement.
- **v2 :** intégration continue : lint, tests Molecule, mode check.
- **v3 :** reconstruction complète de l'agence en une exécution.

### P6 — Cloud hybride AWS
- **v1 :** Terraform : VPC (Virtual Private Cloud) multi-AZ (Availability Zones), IAM (Identity and Access Management), EC2, S3, état distant.
- **v2 :** VPN site-à-site avec OPNsense, DNS hybride, sauvegardes vers S3.
- **v3 :** supervision, maîtrise des coûts, extension Azure.

### P7 — Plateforme conteneurs
- **v1 :** k3s multi-nœuds, ingress, cert-manager.
- **v2 :** GitOps avec Argo CD, NetworkPolicies, observabilité.
- **v3 :** chaîne CI/CD (intégration et déploiement continus) complète, de la construction de l'image au déploiement.

## Ordre de réalisation
1. **Socle :** P0, P1, P2.
2. **Exploitation :** P3, P4.
3. **Automatisation et cloud :** P5, P6.
4. **Plateforme :** P7.
5. **Versions avancées :** les v2 et v3 restantes.

## Principes
- **Manuel, puis documenté, puis automatisé.**
- **Preuve :** chaque version est livrée avec une validation reproductible.
- **Exploitation :** supervision, sauvegarde, restauration, sécurité et mises à jour sont traitées pour chaque brique.
- **Sécurité :** aucun secret versionné, adressage RFC 1918, données entièrement fictives.