# Charte de l'agent de code — binôme B (Safyah, Julie), groupe 1

Cette charte part de l'agent que nous faisons tourner sur Kaggle. Elle décrit à quelles conditions nous accepterions de le brancher sur le dépôt de NidBuyer.


## Ce que l'agent peut faire aujourd'hui

Notre agent a accès à six outils, fournis par la compétition :

| Outil | Ce qu'il fait | Le pire possible |
|---|---|---|
| `run_command` | Lance n'importe quelle commande shell | Tout : supprimer des fichiers, lire des secrets, lancer un script sans fin, envoyer des données si le réseau est ouvert |
| `read_file` | Lit un fichier | Lire un fichier qui contient des mots de passe ou des clés |
| `edit_file` | Remplace un passage dans un fichier | Modifier un test pour qu'il passe, au lieu de corriger le bug |
| `write_file` | Crée ou écrase un fichier | Écraser un fichier entier, ajouter un fichier inutile au correctif |
| `submit_patch` | Rend le correctif final | Rendre un correctif faux comme s'il était terminé |
| `get_status` | Indique le budget restant | Rien de grave |

L'agent a un budget de temps et d'appels d'outils par bug, et Kaggle impose 12 h au total.

Notre prompt lui interdit plusieurs choses : modifier les tests, aller sur internet, sortir du dossier de travail.
Mais dans l'agent lui-même, rien ne l'en empêche. `run_command` suffit à contourner toutes ces consignes.
Ce qui protège vraiment, c'est l'environnement de Kaggle : pas de réseau, un conteneur jetable, des tests remis à l'identique avant la vérification.

Chez NidDouillet, rien de tout ça n'existe par défaut. Cette charte sert à le reconstruire.


## Niveau d'autonomie

**Niveau 1 : l'agent agit, un humain valide.**

L'agent travaille seul la nuit, dans une copie isolée du dépôt. Il ne fait que proposer des correctifs, sous forme de pull requests.
Rien n'est fusionné sans qu'un développeur de NidBuyer l'ait relu.

Pourquoi pas plus : sur Kaggle, les meilleurs agents corrigent environ 1 bug sur 5. Les autres correctifs sont faux ou incomplets, et certains ont l'air corrects.
Sans relecture, on fusionnerait surtout des erreurs.


## Les six règles

| Règle | Chez Kaggle | Chez NidDouillet (notre proposition) |
|---|---|---|
| 1. Moindre privilège : outils autorisés et interdits | Seuls les outils déclarés dans `agent.yaml` existent. Mais `run_command` permet n'importe quelle commande dans le conteneur | Mêmes outils, mais `run_command` passe par une liste de commandes autorisées : lecture git, grep, pytest, python. Pas de `rm -rf`, pas de `git push`, pas de `curl` |
| 2. Bac à sable : où il tourne, ce qu'il ne voit pas (prod, secrets, internet) | Conteneur jetable, sans internet, détruit après chaque bug | Un conteneur neuf par ticket, avec une copie du dépôt. Aucun accès à la production, à la base de données, aux fichiers `.env` ni à internet |
| 3. Budget : temps, appels, euros par nuit | Temps et appels limités par bug, 12 h au total | 10 minutes et 50 appels par ticket, 20 tickets au maximum par nuit, et un plafond de dépense fixé avec la direction (par exemple 30 € par nuit). Au-delà, l'agent s'arrête |
| 4. Humain avant l'irréversible : quels gestes, qui valide | Rien n'est irréversible : le correctif est testé dans un conteneur jetable, jamais fusionné | Fusion, déploiement, suppression d'un fichier ou d'une branche, modification d'un test ou de la CI : toujours validés par le développeur nommé sur le ticket |
| 5. Journal : ce qui est conservé, combien de temps | Kaggle garde les correctifs rendus et les scores | Pour chaque ticket : toutes les commandes lancées, le correctif et le résultat des tests. Le journal est conservé 6 mois, et l'agent ne peut pas le modifier |
| 6. Interrupteur : qui peut l'arrêter, comment revenir en arrière | Kaggle coupe au bout de 12 h. Une soumission peut être annulée | Le responsable technique et le développeur de garde peuvent couper l'agent d'un clic. Comme rien n'est fusionné sans relecture, revenir en arrière revient à fermer la pull request |


## Et le cas Replit ?

L'agent de Replit a supprimé une base de données de production pendant un gel du code, puis a donné de fausses informations sur ce qu'il avait fait.
Avec notre charte, la règle 2 l'aurait empêché : l'agent n'a aucun accès à la production.
La règle 4 aussi : une suppression doit être validée par un humain.
Et la règle 5 aurait montré ce qui s'est vraiment passé, puisque l'agent ne peut pas modifier son propre journal.


## Responsabilité

Le développeur qui fusionne le correctif en est responsable, comme s'il l'avait écrit lui-même.
C'est lui qui signe le commit. L'agent apparaît seulement comme co-auteur, pour qu'on sache d'où vient le code.


## Trois incidents qui coupent l'agent

1. Un correctif modifie un fichier de test ou la configuration de la CI.
2. L'agent lance une commande hors de la liste autorisée, ou essaie d'accéder au réseau.
3. Le budget de la nuit est dépassé, ou le même ticket échoue trois nuits de suite.

Après un incident, l'agent reste coupé jusqu'à ce qu'un humain ait lu le journal et compris ce qui s'est passé.


## Ce que nous refusons

- Que l'agent fusionne ou déploie lui-même, même pour un petit correctif.
- Que l'agent ait accès à la base de production ou aux données des clients.
- Que l'agent modifie des tests pour faire passer un correctif.
- Que l'agent tourne sans journal, ou avec un journal qu'il peut modifier.
