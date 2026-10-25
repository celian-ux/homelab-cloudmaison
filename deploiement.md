# Méthode de déploiement

Le serveur héberge les fichiers de toute la famille. Chaque mise à jour suit donc la même procédure, entièrement scriptée en PowerShell.

## 1. Préparation (sans impact)

- Copie de travail des sources : l'original n'est jamais modifié pendant la préparation.
- **Vérification par empreinte SHA-256** que les sources du serveur sont exactement celles sur lesquelles la modification a été préparée. Au moindre écart, arrêt.
- Application des modifications, compilation et publication **dans un dossier séparé**.
- Contrôle de la présence des fichiers attendus dans la publication.

## 2. Déploiement (créneau défini)

- Refus en dehors du créneau autorisé et pendant une sauvegarde.
- Contrôle de santé **avant** : si l'application ne répond pas déjà correctement, on n'y touche pas.
- **Sauvegarde complète** de la version en production.
- Suspension du watchdog (pour qu'il ne relance pas l'application pendant la copie), arrêt de l'application, vérification que le port est bien libéré.
- Copie, redémarrage, puis contrôles **après** : contrôle de santé, et vérification que la page servie est bien la nouvelle version, avec la même requête que le proxy.
- **En cas d'échec : retour automatique à la version sauvegardée.**
- Le watchdog est toujours réactivé à la fin, quel que soit le résultat.
- Après un succès seulement, mise à jour des sources de référence (avec sauvegarde des anciennes).

## 3. Vérification

Tests fonctionnels avec un compte administrateur et un compte membre (un membre ne doit pas accéder à l'administration), sur ordinateur et sur téléphone. Un script de restauration manuelle reste disponible.

## Un retour arrière qui a servi

Lors du déploiement de l'interface V2, le contrôle après copie a échoué deux fois et le script a automatiquement remis l'ancienne version, sans intervention et en moins d'une minute de coupure. L'analyse des journaux a montré que la nouvelle version n'était pas en cause : le contrôle utilisait une ancienne version du script, qui interrogeait la page sans l'en-tête du proxy. Une fois le script corrigé, le déploiement a réussi au premier essai.

## Le même principe ailleurs

- **Portfolio** : sauvegarde du site, conservation de la configuration IIS existante, tests HTTPS, redirection, police et image, et retour arrière automatique.
- **Configuration** (règles IIS, scripts de supervision, tâches planifiées) : export ou copie de l'existant avant toute modification, vérification de la syntaxe, test immédiat, restauration automatique en cas d'erreur.
