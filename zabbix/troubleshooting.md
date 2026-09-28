# Journal de dépannage — Zabbix

## Bloc Bash exécuté dans PowerShell
- **Symptôme :** `uname`, `free`, `docker` non reconnus ; erreurs sur `&&` et `||`.
- **Diagnostic :** invite `PS C:\WINDOWS\system32>` au lieu de `user@machine:~$`.
- **Cause :** commandes Linux lancées dans PowerShell ; Docker n'existe que dans WSL.
- **Correction :** ouvrir le terminal Ubuntu (`wsl` ou onglet Ubuntu) et relancer.
- **Leçon :** vérifier l'invite avant de coller un bloc. Git Windows et Git WSL sont deux installations distinctes.

## Dépôt introuvable dans WSL
- **Symptôme :** `ls -d ~/skills-lab` ne renvoie rien.
- **Diagnostic :** `ls ~/projets` montre le dépôt.
- **Cause :** chemin réel `~/projets/skills-lab`.
- **Correction :** chemin corrigé dans le profil et dans les commandes.
- **Leçon :** documenter le chemin exact du dépôt dans le profil.

## Variables .env non chargées après réouverture du terminal
- **Symptôme :** `WARN The "POSTGRES_DB" variable is not set. Defaulting to a blank string.` puis `service "postgres" has neither an image nor a build context specified: invalid compose project`.
- **Diagnostic :** `pwd`, `ls -la`, `docker compose ls`, `docker ps`.
- **Cause :** commandes lancées hors de `zabbix/lab` : Compose n'a pas trouvé le `.env` du projet, toutes les variables (dont l'image) étaient vides.
- **Correction :** `cd ~/projets/skills-lab/zabbix/lab` puis relancer les commandes.
- **Leçon :** vérifier l'invite (environnement et dossier) avant de coller un bloc ; le `.env` est lu dans le répertoire du projet.

## Coupure de connexion serveur → base (19 h 38, 28/09)
- **Symptôme :** logs serveur `[Z3005] query failed: PGRES_FATAL_ERROR: server closed the connection unexpectedly`, puis reprise normale (housekeeper OK).
- **Diagnostic :** `docker compose ps` (postgres Up 2 hours, non redémarré), `docker compose logs postgres | grep -iE "fatal|terminat|shutdown"` (à faire).
- **Cause :** non déterminée à ce jour (postgres n'a pas redémarré).
- **Correction :** aucune nécessaire : reconnexion automatique du serveur.
- **Leçon :** horodatages des logs conteneur en UTC (Temps universel coordonné) ; depends_on n'intervient pas après le démarrage.
