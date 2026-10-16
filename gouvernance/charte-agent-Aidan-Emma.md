# Charte de l’agent de code — binôme Aidan–Emma, groupe 1

## Périmètre et niveau d’autonomie

Cette charte propose les conditions d’utilisation d’un agent Gemma 4 Developer Agent pour traiter des tickets du dépôt NidBuyer. **Niveau 1 : l’agent peut analyser, modifier et tester du code dans un bac à sable, mais une personne doit examiner et valider chaque correctif avant toute fusion.** L’agent ne reçoit aucun accès de production.

## Inventaire de départ

Les consignes du Projet 1 indiquent un modèle Gemma 4 31B, un environnement Kaggle isolé et jetable, sans Internet, avec les tests remis à l’identique avant vérification et un budget total de 12 heures. Les outils mentionnés comprennent `read_file`, `edit_file`, `run_command` et `submit_patch`; leurs capacités exactes et les limites par bug dépendent de la version de l’agent du binôme.

| Élément inventorié | Valeur connue pour le binôme |
|---|---|
| Modèle et environnement de compétition | Gemma 4 31B; environnement Kaggle isolé et jetable, sans Internet; tests remis à l’identique avant vérification; plafond total de 12 h |
| Outils | Le sujet cite `read_file`, `edit_file`, `run_command` et `submit_patch`. La liste effective et les paramètres de `agent.yaml` restent à vérifier dans le kit Kaggle privé. |
| Limites par bug, état après E02 | `max_time_minutes: 4`, `max_tool_calls: 30`, `max_turns: 50`, `timeout_seconds: 60`. E01 utilisait 4 minutes et 10 appels; son score était 0,01. E02 conserve 4 minutes et augmente le plafond à 30 appels. La soumission E02 a été faite aujourd’hui; son évaluation est en attente. |
| Sous-agents activés et actions possibles | À vérifier dans `agent.yaml` et `sub_agents/` du kit privé. |
| Restrictions du prompt et contrôles réellement appliqués | À vérifier dans `prompts/system.md`. Une consigne du prompt n’est pas à elle seule un contrôle technique. |

Une interdiction écrite dans le prompt ne constitue pas à elle seule une barrière technique : les permissions, l’isolation et les contrôles doivent aussi l’imposer.

## Les six règles

| Règle | Chez Kaggle | Chez NidDouillet — conditions proposées |
|---|---|---|
| **1. Moindre privilège** | N’utiliser que les outils déclarés dans `agent.yaml`. La lecture permet d’inspecter le dépôt; l’édition et les commandes peuvent modifier des fichiers ou exécuter du code; `submit_patch` remet le correctif à l’évaluation. Vérifier dans le fichier du binôme la liste complète et les paramètres avant de conclure à l’inventaire. | Autoriser la lecture, la modification et les tests uniquement dans une branche de travail isolée. Bloquer l’accès aux secrets, aux données clients et aux identifiants de déploiement; interdire fusion, push vers la branche principale, suppression de données et déploiement. |
| **2. Bac à sable** | Travailler dans le conteneur Kaggle jetable, sans Internet; les tests sont rétablis à l’identique avant vérification. | Créer un conteneur éphémère depuis un commit connu, sans accès réseau par défaut, sans secrets ni données réelles, avec le dépôt monté en écriture uniquement dans la branche de travail. Détruire le conteneur après le ticket. |
| **3. Budget** | Respecter le plafond de 12 heures au total. D’après le journal, après E02 le budget par bug est de 4 minutes, 30 appels d’outils, 50 tours et 60 secondes par commande. Vérifier ces valeurs dans le zip avant chaque soumission. | Plafonner à 20 minutes et 40 appels d’outils par ticket, et à 60 minutes d’exécution cumulée par nuit. À la limite, arrêter et remettre le diff partiel pour revue; aucun service payant externe ne peut être appelé. |
| **4. Humain avant l’irréversible** | L’agent remet un patch à l’évaluateur; l’environnement de compétition vérifie le correctif, il ne constitue pas une autorisation de modifier NidBuyer en production. | Un ingénieur désigné examine le diff, vérifie que les tests existants n’ont pas été affaiblis et relance la CI. Seul un humain peut approuver la fusion et le déploiement; l’agent ne peut ni fusionner ni déployer. |
| **5. Journal** | Conserver l’identifiant du ticket, la configuration/version soumise, les limites appliquées et le résultat disponible de l’évaluation. Le dépôt de journal Kaggle reste privé au binôme; ne pas y copier de secrets ou de données personnelles. | Enregistrer ticket, commit de départ, version du modèle/configuration, commandes, diff et résultats de tests dans un journal d’audit à accès restreint. Exclure secrets et données personnelles; conserver 90 jours puis supprimer automatiquement, sauf obligation d’enquête documentée. |
| **6. Interrupteur et retour arrière** | Arrêter l’exécution à la fin du budget ou si le conteneur sort du périmètre prévu; abandonner la soumission en cas d’anomalie. | L’astreinte peut interrompre le processus et révoquer son jeton. Désactiver le déclencheur nocturne, fermer la branche de travail et revenir au commit stable connu; ne jamais laisser l’agent reprendre seul après un arrêt. |

## Responsabilité

L’équipe NidDouillet reste responsable du correctif intégré. Un ingénieur humain nommé examine et signe le commit; un second réviseur approuve la fusion selon les règles habituelles du dépôt. L’agent ne signe pas et sa proposition n’équivaut pas à une validation.

## Trois incidents qui arrêtent l’agent

1. Le diff modifie, supprime ou désactive un test existant, ou tente de contourner la CI.
2. L’agent tente de lire un secret, des données personnelles, un fichier hors du dépôt de travail, ou d’accéder au réseau ou à la production.
3. Le budget de temps ou d’appels est atteint, ou l’agent tente une fusion, un push vers la branche principale ou un déploiement.

Dans chaque cas : arrêter le processus, conserver le journal expurgé pour l’analyse, isoler le diff et demander une revue humaine avant toute nouvelle exécution.

## Ce que nous refusons

Nous refusons l’auto-fusion et le déploiement sans validation humaine, l’accès direct à la production ou à des données clients, la modification des tests dans le but de faire passer la CI, l’utilisation de secrets par l’agent et toute action irréversible ou hors du périmètre du ticket.
