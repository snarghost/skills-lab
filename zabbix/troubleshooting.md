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
