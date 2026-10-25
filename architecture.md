# Architecture

## Matériel

Une ancienne tour HP réemployée en serveur. Pas de virtualisation (le processeur ne la gère pas) : tous les services tournent directement sur Windows Server.

| Disque | Rôle |
|---|---|
| SSD 128 Go | Système et applications |
| Disque dur 1 To | Espaces personnels de la famille |
| Disque dur 1 To | Dossier commun familial |
| Disque dur 1 To | Sauvegardes Windows Server (dédié) |

Séparer les données, le partage et les sauvegardes sur trois disques physiques différents permet qu'une panne isolée n'emporte jamais à la fois les données et leur sauvegarde.

## Système et rôles

- **Windows Server 2022**, en adresse IP fixe réservée sur la box (mise en place avec une tâche de retour arrière automatique, pour ne jamais perdre l'accès Bureau à distance pendant le changement).
- **Active Directory Domain Services** et DNS : un domaine local, une unité d'organisation dédiée aux comptes de la famille, et un groupe de sécurité qui donne accès à CloudMaison.
- **IIS** : publication du portfolio statique.
- **Sauvegarde Windows Server** : sauvegarde planifiée quotidienne.

## Comptes et droits

| Type de compte | Usage | Principe |
|---|---|---|
| Comptes de la famille | Connexion à CloudMaison | Membres du seul groupe CloudMaison, aucun droit d'administration |
| Compte de service | Exécute CloudMaison | Non administrateur, droit d'ouverture de session en tant que tâche accordé par une GPO dédiée |
| Comptes système | Supervision, sauvegardes | Tâches qui ont besoin de lire l'état du serveur, séparées de l'application web |

Chaque espace personnel a ses propres permissions NTFS : seuls son propriétaire, le compte de service et les administrateurs y ont accès. L'application vérifie en plus l'identité côté serveur, à chaque requête.

## Exposition réseau

| Service | Exposé à Internet ? | Comment |
|---|---|---|
| Portfolio | Oui | Redirection du seul trafic web depuis la box vers IIS, HTTPS obligatoire (redirection permanente 301), certificat Let's Encrypt renouvelé automatiquement |
| CloudMaison | **Non** | L'application n'écoute qu'en local. L'accès distant passe par un VPN maillé (Tailscale) qui fournit le HTTPS. Seuls les appareils autorisés de la famille peuvent la joindre |
| Administration du serveur | Non | Bureau à distance uniquement depuis le réseau local, règles de pare-feu restreintes |

Ce choix évite d'exposer une application maison, avec des fichiers familiaux, directement sur Internet.

## Cycle d'alimentation

Le serveur fonctionne de jour : une tâche l'éteint proprement le soir (en attendant la fin d'une éventuelle sauvegarde), une prise connectée coupe puis rétablit l'alimentation, et le BIOS le redémarre automatiquement le matin. Au démarrage, CloudMaison et la supervision repartent seuls.
