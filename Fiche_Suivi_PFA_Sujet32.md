# Fiche de suivi des projets agents IA

Master • Présentation de l’avancement et plan de correction

Groupe : Binôme PFA   Projet n° : 32   Date : 15/11/2026

Intitulé du projet : Application de Navigation et Services Touristiques Intelligents

Membres : Boujrada Younes, Maldani Mostafa

Jalon examiné : Jalon 1 (Cadrage et données)   Prochain point : Jalon 2 (Prototype MVP)

## Utilisation de la fiche

Avant chaque point, chaque groupe complète les réponses et rassemble les preuves. Présenter en 10 minutes : bilan, démonstration, résultats et blocages. Prévoir ensuite 5 minutes de questions. Chaque membre explique sa contribution et un choix technique.

Pour chaque rubrique, distinguer ce qui est prévu, commencé, démontré ou bloqué. Une fonction est démontrée lorsqu’un scénario reproductible et son résultat peuvent être vérifiés. Justifier toute question sans objet. Si un élément manque, indiquer la correction et son échéance.

## Bilan et périmètre

**1. Quel problème métier traitez-vous, pour quel utilisateur et avec quel résultat attendu ?**

Preuve à présenter : Un cas d’usage précis et un critère de réussite mesurable.

Réponse du groupe : On aide un touriste en visite au Maroc (Casablanca, Marrakech, Rabat, Tanger, Agadir, Dakhla) à organiser rapidement son parcours sans chercher sur plusieurs sites. Cas d'usage : demander une visite de 3h à pied à Casablanca depuis la Place Mohammed V (monuments + pause café, budget 150 DH) et obtenir sur la carte un itinéraire ordonné réalisable en moins de 5 secondes sans dépasser les 3h.

**2. Quel périmètre MVP avez-vous retenu et quelles fonctions avez-vous reportées ?**

Preuve à présenter : Liste des fonctions incluses et exclues, validée par le groupe.

Réponse du groupe : Inclus dans le MVP : carte interactive OpenStreetMap, base locale des lieux touristiques (POI), génération d'itinéraire personnalisé via agent et tracé de la route avec OSRM. Reporté : réservation en ligne, guidage vocal temps réel et système d'avis utilisateurs.

**3. Qu’aviez-vous prévu depuis le dernier point et qu’avez-vous réellement terminé ?**

Preuve à présenter : Fonctions terminées, en cours et bloquées, avec preuve pour chacune.

Réponse du groupe : Terminé : cadrage du sujet, choix de la stack technique et extraction brute des POI des 6 villes cibles (Casablanca, Marrakech, Rabat, Tanger, Agadir, Dakhla) depuis OpenStreetMap. En cours : nettoyage du fichier JSON et intégration de la carte Leaflet. Bloqué : aucun blocage pour l'instant.

**4. Que peut-on exécuter aujourd’hui sans intervention manuelle cachée ?**

Preuve à présenter : Démonstration de bout en bout sur un scénario reproductible.

Réponse du groupe : Démontré : exécution du script Python de collecte des POI et affichage local des marqueurs sur la carte web. Commencé : connexion de l'agent avec le calcul de route OSRM (sera démontré de bout en bout au prochain point).

État général : conforme au jalon

Décision ou aide attendue de l’enseignant : Validation du périmètre sur les 6 villes touristiques retenues (Casablanca, Marrakech, Rabat, Tanger, Agadir, Dakhla).

## Données et architecture

**5. Quel dataset ou corpus utilisez-vous exactement et comment y accède-t-on ?**

Preuve à présenter : Nom, URL, version ou date, licence et fichiers retenus.

Réponse du groupe : Données géographiques OpenStreetMap extraites en novembre 2026 via Overpass API (https://overpass-turbo.eu/, licence ODbL) et résumés d'articles Wikipédia FR via l'API MediaWiki (https://fr.wikipedia.org/w/api.php, licence CC BY-SA 4.0) pour Casablanca, Marrakech, Rabat, Tanger, Agadir et Dakhla. Fichier retenu : `pois_maroc.json`.

**6. Les données couvrent-elles réellement le problème posé ? Que simulez-vous ?**

Preuve à présenter : Variables utiles, labels, limites et distinction entre réel et synthétique.

Réponse du groupe : Le nom, la catégorie, la position GPS (lat, lon) et la description sont réels. Comme la durée de visite et le tarif manquent souvent dans OpenStreetMap, on les simule avec une moyenne par catégorie (ex: 60 min pour un musée, 30 min pour un monument, 45 min pour un restaurant).

**7. Comment avez-vous nettoyé, annoté et séparé les données sans fuite ?**

Preuve à présenter : Pipeline, exemples avant/après, splits et règles anti-doublons.

Réponse du groupe : Le script `clean_pois.py` supprime les lieux sans nom et fusionne les doublons quand un même lieu existe en point et en polygone à moins de 50 mètres (~200 lieux propres par ville, soit ~1 200 POI au total dans `pois_maroc.json`). On prépare à part un fichier de 18 requêtes de test (3 par ville) utilisé uniquement pour évaluer l'agent.

**8. Quelle insuffisance des données risque de fausser votre conclusion ?**

Preuve à présenter : Biais identifié et mesure concrète pour en réduire l’effet.

Réponse du groupe : Il y a beaucoup plus de cafés/restaurants que de monuments dans OSM et les horaires d'ouverture sont souvent vides. Pour éviter de fausser l'itinéraire, on limite le résultat à 1 restaurant maximum par parcours et l'agent indique "horaires à vérifier" quand l'information est absente.

**9. Quels composants fonctionnent et comment échangent-ils les données ?**

Preuve à présenter : Schéma d’architecture à jour et exemple de requête API.

Réponse du groupe : Front-end React/Leaflet <-> API REST FastAPI (Python) <-> Agent LLM + API de routage OSRM. Le front envoie la position GPS, la durée et la demande en JSON (`POST /api/itinerary`) et reçoit la liste ordonnée des étapes avec les coordonnées du tracé.

**10. Pourquoi avez-vous choisi ces frameworks plutôt qu’une solution plus simple ?**

Preuve à présenter : Justification liée au besoin, aux compétences et aux ressources.

Réponse du groupe : Leaflet + OpenStreetMap + OSRM sont 100% gratuits et sans carte bancaire (contrairement à Google Maps). FastAPI en Python permet de manipuler simplement nos fichiers JSON et d'appeler l'API du modèle.

**11. Comment gérez-vous l’état, la mémoire, les erreurs et les reprises ?**

Preuve à présenter : Modèle d’état, journal d’une exécution et exemple d’échec traité.

Réponse du groupe : L'itinéraire généré est gardé en session pour pouvoir être modifié. Si le serveur public OSRM ne répond pas (timeout), le back-end bascule automatiquement sur un calcul de distance à vol d'oiseau (formule de Haversine) pour estimer le temps de marche sans bloquer l'exécution.

Correction prioritaire sur les données ou la conception : Terminer le nettoyage des doublons dans `pois_maroc.json`.

## Fonctionnement des agents et protocole

**12. Quelle décision l’agent prend-il et quels outils appelle-t-il ?**

Preuve à présenter : Trace d’un scénario avec entrées, décisions, appels et sorties.

Réponse du groupe : L'agent analyse la demande de l'utilisateur pour en extraire les critères (catégories, budget, durée), appelle l'outil `search_pois()` pour filtrer la base locale, puis appelle `build_route()` qui ordonne les étapes du plus proche au plus éloigné avant de formuler la réponse.

**13. Quel résultat obtenez-vous avec des règles ou une chaîne sans agent ?**

Preuve à présenter : Comparaison sur les mêmes cas, avec le même budget de calcul.

Réponse du groupe : Avec une règle simple sans agent (tri par distance uniquement), le programme choisit les points les plus proches sans comprendre le texte libre (il propose par exemple 3 cafés de suite et ignore une demande comme "endroit calme et historique").

**14. Si plusieurs agents collaborent, quel rôle distinct remplit chacun ?**

Preuve à présenter : Rôles, échanges et comparaison avec un agent unique. Sinon expliquer l’absence de multi-agents.

Réponse du groupe : Sans objet (un seul agent utilisé). On utilise un agent unique relié à nos fonctions Python car ajouter plusieurs agents augmenterait le temps de réponse alors que le calcul du trajet optimal est mieux fait par un algorithme classique (plus proche voisin).

**15. Comment vérifiez-vous une sortie du LLM avant de l’utiliser ?**

Preuve à présenter : Validation du format, contrôle des sources, test métier et cas d’abstention.

Réponse du groupe : Le serveur vérifie que la sortie respecte le schéma JSON attendu, que chaque ID de lieu existe vraiment dans `pois_maroc.json` (aucun lieu inventé) et que la durée totale ne dépasse pas la durée demandée. Si aucun lieu ne correspond, l'agent indique qu'aucun parcours ne respecte les critères.

**16. Quelles actions exigent une validation humaine et quelles limites imposez-vous ?**

Preuve à présenter : Permissions, plafond d’appels, coût maximal et scénario d’action refusée.

Réponse du groupe : L'agent a seulement un accès en lecture sur le fichier des POI, avec une limite de 3 appels d'outils maximum par requête et un timeout de 10 secondes. L'ajout ou la modification de lieux dans la base reste réservé à l'administrateur humain.

**17. Quels tests prouvent que les fonctions annoncées marchent réellement ?**

Preuve à présenter : Tests automatisés, résultat daté et anomalies encore ouvertes.

Réponse du groupe : Commencé : tests manuels sur le filtrage des POI et le calcul de distance. Prévu pour le prochain jalon : fichier de tests `pytest` vérifiant automatiquement le respect de la durée maximale et le rejet d'un ID de lieu inexistant.

**18. Quels indicateurs avez-vous choisis et pourquoi conviennent-ils au projet ?**

Preuve à présenter : Définition des métriques, baseline et protocole de mesure.

Réponse du groupe : Sur 15 scénarios de test : 1) Le pourcentage d'itinéraires qui respectent la durée et le budget demandés. 2) Le nombre de lieux inventés hors base (objectif : 0). 3) Le temps de réponse moyen en secondes.

Correction prioritaire sur l’agent ou les tests : Finaliser la rédaction des 15 scénarios de test dans un fichier séparé pour évaluer l'agent au prochain point.

## Résultats et déploiement

**19. Quels résultats chiffrés obtenez-vous sur un jeu de test distinct ?**

Preuve à présenter : Tableau baseline/solution, taille du test, latence et coût par scénario.

Réponse du groupe : Commencé : la baseline par distance fonctionne en moins de 0.5s mais ne couvre qu'environ 50% des contraintes en texte libre. Les mesures complètes de l'agent sur les 15 scénarios de test seront présentées au Jalon 2.

**20. Quels sont vos trois échecs les plus importants et comment les expliquez-vous ?**

Preuve à présenter : Entrées, sorties observées, causes probables et correction envisagée.

Réponse du groupe : 1) Demande avec trop de visites pour une durée très courte (ex: 1h) -> limiter le nombre d'étapes. 2) Lieux trop éloignés à pied -> filtrer par rayon selon le mode de transport. 3) Lenteur occasionnelle de l'API publique OSRM -> fallback sur la distance de Haversine.

**21. Quelles entrées difficiles ou hostiles avez-vous testées ?**

Preuve à présenter : Données manquantes, outil indisponible, prompt malveillant, réponse non sourcée.

Réponse du groupe : Prévu dans les tests : demande sur une autre ville absente du dataset, durée ou budget négatif, coupure de l'API OSRM (test du fallback), et question hors sujet touristique pour vérifier que l'agent refuse de répondre.

**22. Un autre groupe peut-il installer et lancer votre application ?**

Preuve à présenter : README vérifié, dépendances fixées et commande de lancement.

Réponse du groupe : En cours : mise en place du fichier `README.md` et du fichier `requirements.txt` avec les commandes pour lancer le back-end FastAPI et le front-end React en local.

**23. Quelle version est déployée et quel parcours avez-vous validé ?**

Preuve à présenter : Commit ou version, environnement, démonstration et résultat du test de bout en bout.

Réponse du groupe : Version locale de développement : extraction des données et affichage des POI sur la carte validés. Le parcours complet de bout en bout sera validé au prochain point.

**24. Comment protégez-vous les secrets, les accès et les données ?**

Preuve à présenter : Configuration sans secrets, contrôle d’accès et politique de journalisation.

Réponse du groupe : La clé API du modèle est stockée uniquement côté serveur dans un fichier `.env` (ajouté au `.gitignore`) et n'est jamais exposée côté navigateur.

### Résultats à comparer

| Indicateur | Baseline | Solution | Jeu ou scénario |
| --- | --- | --- | --- |
| Respect des contraintes (durée, catégories) | ~50% (ignore le texte libre) | Objectif > 85% (mesure au J2) | 15 scénarios de test |
| Latence et coût | < 0.5 s / gratuit | ~3 à 4 s / API gratuite | Moyenne par requête |

Pour un projet industriel, démontrer en simulation et expliquer les limites du passage au réel. Pour un projet informatique, montrer les droits accordés aux outils et les actions qui exigent une validation humaine.

## Contributions et corrections

**25. Quelle contribution chaque membre peut-il présenter et expliquer ?**

Preuve à présenter : Tâches, commits, résultats et revue croisée. Chaque membre explique au moins un choix technique.

Réponse du groupe : Boujrada Younes : développement de l'interface web React, intégration de la carte Leaflet/OpenStreetMap, affichage des marqueurs, tracé d'itinéraire et collecte/nettoyage du dataset OSM (pois_maroc.json). Maldani Mostafa : développement de l'API FastAPI, outils de calcul d'itinéraire OSRM/Haversine, intégration de l'Agent IA LLM (Gemini Flash) et validation stricte Pydantic.

**26. Quel blocage menace le prochain jalon et de quelle aide avez-vous besoin ?**

Preuve à présenter : Cause précise, tentatives déjà faites et décision attendue.

Réponse du groupe : Pas de blocage technique bloquant, seulement le manque d'horaires d'ouverture sur certains lieux dans OpenStreetMap qu'on gère avec des estimations par défaut par catégorie.

**27. Quelle affirmation ou fonctionnalité devez-vous corriger après cette revue ?**

Preuve à présenter : Écart entre prévu et observé, cause et preuve de correction attendue.

Réponse du groupe : Au lieu de couvrir tout le pays sans filtre, on cible 6 villes principales (Casablanca, Marrakech, Rabat, Tanger, Agadir, Dakhla) pour garder une base de lieux bien nettoyée et vérifiable (~1 200 POI).

**28. Quelles sont vos trois prochaines actions prioritaires ?**

Preuve à présenter : Action observable, responsable nommé, échéance et test d’acceptation.

Réponse du groupe : 1) Finaliser le nettoyage du fichier `pois_maroc.json` (6 villes) et l'affichage des marqueurs sur la carte Leaflet (Boujrada Younes). 2) Développer les endpoints API FastAPI et le calcul de route OSRM (Maldani Mostafa). 3) Connecter l'agent IA à l'interface et aux outils pour la démo du Jalon 2 (Younes & Mostafa).

**29. Faut-il réduire le périmètre ou revoir le choix des données ou des outils ?**

Preuve à présenter : Décision argumentée et effet sur le planning et les livrables.

Réponse du groupe : Non, le périmètre actuel (carte + génération d'itinéraire sur les 6 villes avec un seul fichier JSON, OpenStreetMap et OSRM) reste simple et réaliste en binôme.

**30. Quelle preuve présenterez-vous lors du prochain point d’avancement ?**

Preuve à présenter : Jalon vérifiable et scénario exact qui permettra de le valider.

Réponse du groupe : Démonstration en direct d'un scénario complet : sélection d'une ville (ex: Casablanca ou Tanger), saisie d'une demande ("visite de 3h à pied, monuments et café") et affichage du parcours tracé sur la carte avec les étapes choisies par l'agent.

### Plan de correction à valider

| Action prioritaire | Responsable | Échéance | Preuve de correction |
| --- | --- | --- | --- |
| 1. Nettoyage `pois_maroc.json` (6 villes) + sélecteur Leaflet | Boujrada Younes | Semaine S3-S4 | Fichier JSON propre + carte multi-villes |
| 2. Back-end FastAPI, routage OSRM et intégration Agent IA | Maldani Mostafa | Semaine S4-S5 | Endpoints fonctionnels + validation Pydantic |
| 3. Connexion Front-Back et démo complète sur scénario | Younes & Mostafa | Semaine S5 | Démo de bout en bout en local |

Décision de la revue : poursuivre

Commentaire de l’enseignant : [à compléter]
