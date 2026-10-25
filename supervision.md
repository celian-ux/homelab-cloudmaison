# Supervision et alertes

Tout est fait avec le **Planificateur de tâches** et des scripts **PowerShell**, sans logiciel tiers à installer sur le serveur.

## Les tâches

| Tâche | Fréquence | Compte | Rôle |
|---|---|---|---|
| Démarrage automatique | Au démarrage | Compte de service | Lance CloudMaison |
| Surveillance (watchdog) | Toutes les 2 min | Système | Interroge le contrôle de santé `/healthz`. Après 3 échecs consécutifs, redémarre l'application, au plus une fois par période de sécurité (pas de redémarrage en boucle) |
| État du tableau de bord | Toutes les 5 min | Système | Collecte l'état de l'application, des sauvegardes et des disques, calcule les alertes et écrit un fichier JSON lu par l'administration |
| Notifications | Toutes les 5 min | Système | Compare les alertes avec l'état précédent et envoie une notification push seulement pour un **nouveau** problème ou un **retour à la normale** |
| Extinction du soir | Quotidienne | Système | Arrêt propre, reporté tant qu'une sauvegarde est en cours |
| Copie hors serveur | Toutes les 15 min en journée | Système | Copie hebdomadaire vers une autre machine, dès qu'elle est allumée |

## Alertes surveillées

- Application qui ne répond plus, surveillance désactivée, redémarrages automatiques trop fréquents.
- Disques : volume inaccessible, occupation au-delà de 85 % puis 95 %.
- **Santé des disques** : état SMART dégradé, erreurs de lecture, température de 55 °C ou plus.
- Sauvegardes : dernière réussite trop ancienne (plus de 36 h), dernière opération en échec, état illisible.
- Copie hors serveur : aucune réussite depuis 10 jours.
- Tableau de bord non actualisé depuis plus de 20 minutes.

## Discrétion des notifications

Les notifications push passent par un service public (ntfy). Elles ne contiennent donc **jamais** le texte détaillé des alertes ni d'informations sur la famille, seulement un libellé générique (« Alerte active dans l'administration »). Le détail se consulte dans CloudMaison, derrière l'authentification.

## Séparation des privilèges

L'application web ne peut ni arrêter le serveur, ni modifier les sauvegardes, ni lire l'état système directement. Les tâches système produisent un fichier d'état, et l'application l'affiche en lecture seule.
