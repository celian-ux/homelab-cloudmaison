# Sauvegardes

## Deux niveaux

| Niveau | Quoi | Où | Fréquence | Historique |
|---|---|---|---|---|
| 1. Sauvegarde Windows Server | Système, état Active Directory, volumes de données | Disque dédié, dans le serveur | Quotidienne | Plusieurs versions, gérées par Windows |
| 2. Copie hors serveur | Données irremplaçables : espaces de la famille, dossier commun, code et scripts de CloudMaison, portfolio | Disque d'un autre ordinateur de la maison | Hebdomadaire | 4 semaines |

Le premier niveau protège contre la panne d'un disque de données ou une erreur de manipulation. Le second protège contre une panne qui emporterait tout le serveur (alimentation, plusieurs disques, logiciel malveillant). Une copie dans un autre lieu reste une amélioration possible.

## Copie hors serveur

- Le serveur vérifie toutes les 15 minutes en journée si la copie de la semaine est faite. Sinon, et si l'autre ordinateur est allumé, il la lance.
- Chaque copie va dans un **dossier daté** et n'est validée qu'avec un fichier témoin écrit en fin de copie. Seules les copies complètes comptent pour la rotation.
- La rotation garde les **4 dernières copies complètes**. Si les fichiers du serveur sont corrompus ou chiffrés, les copies plus anciennes restent intactes.
- Côté ordinateur de destination : un compte local dédié, caché, sans autre droit que l'écriture dans ce dossier, un partage caché accessible à ce seul compte, et une règle de pare-feu qui n'autorise que le serveur.
- Côté serveur : le mot de passe de ce compte est stocké chiffré (DPAPI machine), dans un dossier réservé au système et aux administrateurs.
- Alerte push si aucune copie n'a réussi depuis 10 jours.

## Restauration testée

Une sauvegarde qui n'a jamais été restaurée n'est qu'une hypothèse. Un dossier personnel a été restauré depuis la sauvegarde Windows vers un emplacement séparé, puis comparé à l'original. Les seuls écarts étaient les fichiers créés après la sauvegarde.

## Coexistence avec l'extinction nocturne

- L'extinction du soir est reportée tant qu'une sauvegarde est en cours.
- Le rattrapage automatique des tâches manquées a été désactivé pour la sauvegarde : un redémarrage manuel en soirée ne relance plus une sauvegarde de plus d'une heure peu avant la coupure d'alimentation (voir [retours d'expérience](retours-experience.md)).
