# CloudMaison

Application web **ASP.NET Core 10 (Razor Pages, C#)**, développée pour remplacer un service de stockage en ligne au sein de la famille. Nextcloud a été envisagé, mais sans virtualisation ni second ordinateur, une application native intégrée à Active Directory était plus simple à maintenir.

![Connexion](../images/connexion.png)

## Fonctionnalités

| Module | Ce qu'il fait |
|---|---|
| **Mes fichiers** | Envoi jusqu'à 10 fichiers à la fois, recherche, renommage, téléchargement, aperçu PDF et images, corbeille personnelle avec restauration et vidage |
| **Mes photos** | Galerie privée (JPG, PNG, WebP), aperçu en grand, corbeille |
| **Dossier commun** | Fichiers partagés par toute la famille, corbeille commune |
| **Dossiers partagés** | Navigation dans les sous-dossiers, création, renommage, déplacement de fichiers sans jamais écraser, recherche dans tout le partage |
| **Administration** | Réservée à deux comptes : état de l'application et de la surveillance, sauvegardes, occupation des disques, alertes |
| **Statistiques** | Espace et nombre de fichiers par membre et pour le dossier commun |

## Sécurité applicative

- **Authentification** : identifiants vérifiés auprès d'Active Directory, puis session par cookie (`HttpOnly`, `Secure`, `SameSite`), avec expiration glissante.
- **Revalidation des sessions** : toutes les 5 minutes, l'application vérifie que le compte est toujours actif et toujours membre du groupe autorisé. Un compte désactivé perd l'accès sans attendre la fin de sa session.
- **Autorisation par défaut** : toute page exige un utilisateur connecté et membre du groupe, sauf la page de connexion et le contrôle de santé.
- **Limitation des tentatives** de connexion (fenêtre glissante).
- **Protection CSRF** sur tous les formulaires, cookies antifalsification réservés au HTTPS.
- **Fichiers** : tailles maximales par fichier et par envoi, contrôle de la signature réelle des images, noms validés.
- **Chemins** : normalisation et confinement à la racine autorisée (pas de `../`), refus des liens symboliques et points de jonction NTFS, aucun écrasement lors des déplacements.
- **Moindre privilège** : l'application tourne sous un compte de service sans droits d'administration. Les informations système (sauvegardes, disques) sont collectées par une tâche séparée, puis lues par l'application en lecture seule.
- **Clés de session** protégées par DPAPI au niveau machine.

## Derrière un proxy inverse

L'application écoute en HTTP sur l'interface locale uniquement. Le HTTPS est assuré par Tailscale Serve, qui transmet l'en-tête `X-Forwarded-Proto`. L'application ne fait confiance à cet en-tête que s'il vient du proxy local. Sans lui, les cookies sécurisés et les jetons antifalsification sont refusés, ce qui est le comportement voulu (voir [retours d'expérience](retours-experience.md)).

## Interface V2

Refonte complète de l'interface en thème sombre « NAS professionnel » :

- feuille de style autonome (Bootstrap retiré), polices hébergées localement, icônes SVG intégrées ;
- menu latéral sur ordinateur, menu repliable sur téléphone (fermeture par Échap, clic extérieur ou choix d'un lien) ;
- zones de dépôt de fichiers, renommage dans un volet sans rechargement de page, tailles lisibles (Ko, Mo, Go) ;
- refonte limitée aux vues : **aucune ligne de logique serveur modifiée**, pour ne prendre aucun risque sur la sécurité existante.
