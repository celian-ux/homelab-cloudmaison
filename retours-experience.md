# Retours d'expérience

Les problèmes concrets rencontrés en mettant le serveur en service, et ce que j'en ai appris.

## Proxy inverse et cookies sécurisés

**Symptôme** : la page de connexion plantait quand on l'appelait en local, et une boucle de redirections vers la page d'erreur apparaissait.
**Cause** : l'application exige des cookies `Secure`, donc une requête HTTPS. Derrière le proxy, elle ne sait qu'elle est en HTTPS que grâce à l'en-tête `X-Forwarded-Proto`, et seulement s'il vient d'un proxy de confiance.
**Leçon** : ne jamais affaiblir la politique des cookies pour « faire marcher ». Il faut configurer correctement les en-têtes transmis et tester avec la même requête que le proxy.

## Liaison IIS attachée à une adresse précise

**Symptôme** : un test local du site en HTTPS renvoyait `400 Bad Request`.
**Cause** : la liaison HTTPS d'IIS n'écoute que sur l'adresse réseau du serveur, pas sur l'adresse de bouclage.
**Leçon** : lire les liaisons avant de tester. Mes scripts déterminent maintenant l'adresse à tester à partir de la liaison elle-même.

## La redirection HTTPS disparue

**Symptôme** : le site répondait en HTTP sans rediriger. Les navigateurs récents passant d'eux-mêmes en HTTPS, le problème était invisible sur téléphone.
**Cause** : le fichier de configuration du site avait été remplacé par une ancienne copie, qui ne contenait plus la règle de réécriture.
**Leçon** : tester avec `curl`, pas seulement avec un navigateur. Une copie de fichier garde sa date d'origine : une date ancienne ne prouve pas que le fichier n'a pas été remplacé.

## Une sauvegarde qui se lançait au mauvais moment

**Symptôme** : rallumer le serveur en soirée déclenchait une sauvegarde de plus d'une heure, peu avant la coupure d'alimentation programmée.
**Cause** : l'option « exécuter la tâche dès que possible si une exécution planifiée a été manquée ». Le journal du Planificateur montrait toute une série de tâches relancées à la même seconde, quelques minutes après le démarrage.
**Leçon** : le journal opérationnel du Planificateur de tâches répond directement à la question « qui a lancé quoi ? ».

## Les retours arrière automatiques

Deux déploiements ont été annulés automatiquement par les contrôles. L'application est restée disponible, et les journaux d'événements .NET ont permis de trouver la vraie cause. Le temps passé à écrire des contrôles et des retours arrière a été largement rentabilisé.

## PowerShell 5.1

- Un bloc `try / catch / finally` collé morceau par morceau dans la console ne se comporte pas comme dans un fichier : les procédures sont désormais toujours livrées sous forme de scripts.
- Scripts écrits en ASCII pour éviter les problèmes d'encodage entre éditeurs et consoles.
- `Invoke-WebRequest` suit jusqu'à 50 redirections par défaut : un seul appel peut produire des dizaines d'erreurs dans les journaux.
