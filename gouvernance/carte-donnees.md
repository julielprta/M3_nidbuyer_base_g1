# Carte des données de NidBuyer

TP 0 du M3. Cas fictif : Mme Martin (aucune vraie donnée personnelle dans ce repo).

État du code au 16/10/2026. Les routes `/chat` et `/alerte` sont encore des TODO (erreur 501) : plusieurs flux ci-dessous sont donc prévus mais pas encore actifs. Les cases « à vérifier » sont des points que nous n'avons pas encore confirmés dans le code.

## Le tableau

| Donnée | Entre par | Passe par | Écrite où ? | Combien de temps ? | Qui d'autre la voit ? |
|---|---|---|---|---|---|
| Texte libre du profil (`description_libre`) | `POST /rechercher`, modèle `ProfilAcheteur` (main.py:41, 87) | `search_similar` (main.py:102), puis embeddings SentenceTransformer (rag.py) | Nulle part aujourd'hui dans `/rechercher`. Futur : `/chat` et `/alerte` (TODO, 501) | Durée de la requête | Pas envoyé au LLM dans `/rechercher` (le prompt n'utilise que `profil.intention`, main.py:114) |
| E-mail + profil d'alerte | `POST /alerte`, modèle `AlerteProfil` (main.py:50, 130), route non implémentée | `sauvegarder_profil` (alert.py:16), pas encore appelé par la route | `data/alertes.json`, en clair (alert.py:9, 20). Cible prévue : Supabase (TODO main.py:132) | Indéfinie, aucune fonction de suppression | Quiconque accède au disque ou au repo (à vérifier dans git) |
| E-mail dans le rapport de synchro | Lu depuis `alertes.json` (ingestion.py:110) | `rapport["erreurs"]` en cas d'échec d'alerte (ingestion.py:121) | Logs applicatifs (ingestion.py:125) | Selon la configuration des logs (à vérifier) | Toute personne ayant accès aux logs |
| E-mail vers SMTP / Slack | Même profil d'alerte | `notifier_email` / `notifier_slack` (alert.py:22-29, TODO) | Chez le prestataire SMTP ou chez Slack | Hors de notre contrôle | Prestataire SMTP, Slack |
| Contenu envoyé au LLM | Prompt de `/rechercher` (main.py:114) ; futur `/chat` | `llm.generer`, `generate_content` (llm.py) | Chez le fournisseur : Google ou Ollama en local (à vérifier avec `provider()`) | Selon les conditions du fournisseur | Google si le fournisseur est externe |
| Journal des appels LLM (`JOURNAL`) | Chaque appel (llm.py:75) | `_journaliser` | Mémoire du serveur, 1000 derniers appels (llm.py:81) | Jusqu'au redémarrage | Endpoint d'état (50 derniers). Ne contient aucun texte : seulement fournisseur, modèle, statut, latence, tokens |
| Résultats d'évaluation | Scénarios du harnais M2, éventuellement des questions réelles copiées du chat | `m2_eval.py` | `eval_resultats.json` (id, réponse du modèle, noms d'outils, verdicts, durée) + sortie console | Jusqu'à suppression manuelle | Toute l'équipe, et git si le fichier est suivi (à vérifier) |
| Annonces collectées (peuvent contenir des coordonnées de vendeurs) | `sources/*.py`, `generique.py` (texte d'e-mails, PDF d'agences) | `ingestion.py` | Supabase (db.py:52) et base vectorielle (rag.py:56) | À vérifier | Équipe, Supabase |
| Base vectorielle | Annonces uniquement (rag.py:56) | `rag.py`, `get_collection` | `./chroma_db` sur disque (variable `CHROMA_PATH`) | Persistante | Équipe. Pas de donnée acheteur aujourd'hui |

## TODO = décisions pas encore prises

- main.py:75, 82, 125 : routes `/biens`, `/biens/{id}`, `/chat` non écrites. Que fera-t-on du texte de l'acheteur ?
- main.py:132 : persister les alertes dans Supabase plutôt que dans un fichier local.
- alert.py:24, 29, 37 : envoi par SMTP, webhook Slack, filtrage des profils. Chaque envoi ajoute un prestataire.
- rag.py:37 : quels champs indexer, la description entière ou non ?
- bienici.py:18, leboncoin.py:19 : construction des requêtes de collecte.

## Les trois questions

**1. Laquelle de ces données est la plus sensible ? Pourquoi ?**
Le champ libre « Parlez-nous de votre projet ». Dans l'exemple de Mme Martin, il révèle une situation familiale (divorce, deux enfants) et une information de santé sur un mineur (le fils en fauteuil). Ce sont des données de catégorie particulière au sens de l'art. 9 du RGPD. Aujourd'hui `/rechercher` ne l'écrit nulle part, mais les routes `/chat` et `/alerte`, encore à écrire, vont le faire circuler vers le LLM et vers un fichier ou une base.

**2. Laquelle est la plus inutile pour conseiller Mme Martin ?**
La partie biographique du champ libre (le divorce, les enfants). Pour la conseiller, il suffit de critères : logement accessible, nombre de chambres, budget, quartiers. Des cases à cocher donneraient le même service sans collecter le reste. Les coordonnées de vendeurs présentes dans certaines annonces ne servent pas non plus.

**3. Si Mme Martin demande « effacez tout », dans combien d'endroits faut-il aller ? Y en a-t-il un où nous ne saurions pas le faire ?**
Une fois `/alerte` et `/chat` branchées, au moins sept : `alertes.json` (ou la table Supabase des alertes), les logs de synchro, `eval_resultats.json`, l'historique git, la sortie console, le fournisseur du LLM et le prestataire SMTP ou Slack. Nous ne saurions pas effacer : les logs (l'e-mail y est dispersé sans identifiant de recherche), l'historique git si un fichier a été commité, et ce qui est chez le fournisseur du LLM. Dans `alert.py`, il n'existe aucune fonction de suppression.

## À retenir pour la présentation

Les décisions de gouvernance se prennent avant d'écrire les routes `/chat` et `/alerte`, pas après : une fois le texte libre dans un fichier et dans les logs, l'effacer devient difficile.