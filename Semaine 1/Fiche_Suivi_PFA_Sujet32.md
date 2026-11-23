# Fiche de suivi des projets agents IA

Master • Présentation de l’avancement et plan de correction

Groupe : Binôme PFA   Projet n° : 32   Date : 15/11/2026

Intitulé du projet : CasaTrip — Application de Recommandation & Navigation Touristique Intelligente à Casablanca

Membres : Boujrada Younes (Front-end, Carte & Data), Maldani Mostafa (Back-end & Agent IA)

Jalon examiné : Jalon 1 (Cadrage et données)   Prochain point : Jalon 2 (Prototype MVP)

## Utilisation de la fiche

Avant chaque point, chaque groupe complète les réponses et rassemble les preuves. Présenter en 10 minutes : bilan, démonstration, résultats et blocages. Prévoir ensuite 5 minutes de questions. Chaque membre explique sa contribution et un choix technique.

Pour chaque rubrique, distinguer ce qui est prévu, commencé, démontré ou bloqué. Une fonction est démontrée lorsqu’un scénario reproductible et son résultat peuvent être vérifiés. Justifier toute question sans objet. Si un élément manque, indiquer la correction et son échéance.

## Bilan et périmètre

**1. Quel problème métier traitez-vous, pour quel utilisateur et avec quel résultat attendu ?**

Preuve à présenter : Un cas d’usage précis et un critère de réussite mesurable.

Réponse du groupe : On aide un visiteur ou touriste dans le Grand Casablanca à planifier et organiser instantanément son circuit sans naviguer entre de multiples plateformes. Cas d'usage : demander un itinéraire de 3h à pied depuis la Place Mohammed V (monuments Art déco, pause café, budget 150 DH) et obtenir sur la carte interactive un circuit optimisé et ordonné en moins de 5 secondes, respectant scrupuleusement la durée et le budget alloués.

**2. Quel périmètre MVP avez-vous retenu et quelles fonctions avez-vous reportées ?**

Preuve à présenter : Liste des fonctions incluses et exclues, validée par le groupe.

Réponse du groupe : Inclus dans le MVP : carte interactive Leaflet/OpenStreetMap sur le Grand Casablanca (13 zones / quartiers), dataset réel nettoyé et fusionné (`casablanca_places.json` / `casablanca_places.csv` de 4 946 POI), moteur de recommandation par Agent IA (FastAPI + Gemini Flash), et routage réel via OSRM avec fallback Haversine. Reporté : réservation directe en ligne, guidage vocal en temps réel, billetterie et système d'avis communautaires.

**3. Qu’aviez-vous prévu depuis le dernier point et qu’avez-vous réellement terminé ?**

Preuve à présenter : Fonctions terminées, en cours et bloquées, avec preuve pour chacune.

Réponse du groupe : Terminé : cadrage du projet CasaTrip, sélection de la stack 100 % open-source / gratuite, collecte réelle des données OSM (Overpass API) et Overture Maps (Places theme), pipeline de nettoyage, déduplication (356 doublons fusionnés) et affectation aux 13 quartiers (4 946 POI livrés en CSV et JSON avec `cleaning_report.txt` et `README_DATASET.md`). En cours : intégration des filtres par catégorie sur la carte Leaflet et interfaçage avec les endpoints FastAPI. Bloqué : aucun blocage.

**4. Que peut-on exécuter aujourd’hui sans intervention manuelle cachée ?**

Preuve à présenter : Démonstration de bout en bout sur un scénario reproductible.

Réponse du groupe : Démontré : exécution complète du script de Data Engineering (`python scripts/build_casablanca_dataset.py`) générant le dataset nettoyé de 4 946 lieux vérifiés, rapport de nettoyage automatisé et affichage des marqueurs par quartier sur la carte Leaflet. Commencé : orchestration de l'Agent IA avec les outils de recherche spatiale (`search_pois`) et de calcul de route OSRM.

État général : conforme au jalon

Décision ou aide attendue de l’enseignant : Validation du périmètre géographique centré sur le Grand Casablanca et de la structure du dataset de 4 946 POI.

## Données et architecture

**5. Quel dataset ou corpus utilisez-vous exactement et comment y accède-t-on ?**

Preuve à présenter : Nom, URL, version ou date, licence et fichiers retenus.

Réponse du groupe : Deux sources réelles et ouvertes interrogées en novembre 2026 :
1) OpenStreetMap via l'API Overpass (`out center tags;`, licence ODbL).
2) Overture Maps Foundation via CLI `overturemaps` (Places theme, bounding box `[33.45, -7.75, 33.68, -7.45]`, confiance >= 0.5, licence CDLA-Permissive v2.0).
Livrables générés : `casablanca_places.csv` (UTF-8 avec BOM), `casablanca_places.json`, script reproductible `scripts/build_casablanca_dataset.py`, rapport `cleaning_report.txt` et documentation `README_DATASET.md`.

**6. Les données couvrent-elles réellement le problème posé ? Que simulez-vous ?**

Preuve à présenter : Variables utiles, labels, limites et distinction entre réel et synthétique.

Réponse du groupe : Les données réelles couvrent strictement l'identifiant, le nom, le nom en arabe (si dispo), les coordonnées GPS (100 % réelles et validées dans la bounding box), l'adresse, le téléphone (normalisé +212), le site web (URLs validées), la catégorie normalisée parmi 12 types cibles, et la zone/quartier. En l'absence de grilles tarifaires dans les données ouvertes, des fourchettes MAD estimées par catégorie sont insérées avec l'indicateur explicite `price_is_estimated = True` (aucune donnée tarifaire n'est falsifiée ou présentée comme mesurée).

**7. Comment avez-vous nettoyé, annoté et séparé les données sans fuite ?**

Preuve à présenter : Pipeline, exemples avant/après, splits et règles anti-doublons.

Réponse du groupe : Pipeline strict via `build_casablanca_dataset.py` :
- Rejet des entités sans nom, sans coordonnées ou hors bounding box.
- Filtrage Overture Maps sur un indice de confiance >= 0.5.
- Mapping normalisé vers 12 catégories cibles (restaurant, cafe, fast_food, bar, nightclub, museum, attraction, accommodation, park, beach, shopping, entertainment).
- Déduplication spatiale et textuelle : même catégorie, distance Haversine <= 150 m, similarité Levenshtein >= 0.82 (minuscules, accents retirés, stop words éliminés) -> 356 doublons fusionnés (5 302 candidats réduits à 4 946 POI uniques).
- Affectation au centroïde du quartier le plus proche parmi 13 zones réelles de Casablanca.

**8. Quelle insuffisance des données risque de fausser votre conclusion ?**

Preuve à présenter : Biais identifié et mesure concrète pour en réduire l’effet.

Réponse du groupe : Les cafés et restaurants représentent ~58 % du dataset alors que les musées (63) et plages (49) sont plus rares. Pour éviter de générer un itinéraire saturé de restauration, l'agent IA applique un quota strict (max 1 restaurant et 1 café par circuit) et diversifie obligatoirement avec des points d'intérêt culturels ou des parcs. De plus, pour les horaires d'ouverture souvent non renseignés (97.8 % manquants), l'interface appose un badge « Horaires à vérifier ».

**9. Quels composants fonctionnent et comment échangent-ils les données ?**

Preuve à présenter : Schéma d’architecture à jour et exemple de requête API.

Réponse du groupe : Architecture 3 tiers : Front-end React/Vite + Leaflet <-> Back-end FastAPI (Python 3.12) <-> Agent IA (Gemini 2.5 Flash) + Serveur de routage OSRM. Le front-end soumet un payload JSON (`POST /api/itinerary` : `zone`, `lat`, `lon`, `budget`, `duration`, `preferences`) et reçoit l'itinéraire ordonné avec les polylignes GPS exactes calculées par OSRM.

**10. Pourquoi avez-vous choisi ces frameworks plutôt qu’une solution plus simple ?**

Preuve à présenter : Justification liée au besoin, aux compétences et aux ressources.

Réponse du groupe : Leaflet + OSM + OSRM offrent une solution cartographique et de calcul d'itinéraire 100 % gratuite et sans dépendance à une carte bancaire (contrairement à l'API Google Maps). FastAPI assure un typage rigoureux avec validation Pydantic et des temps de réponse sous les 100 ms hors appel LLM.

**11. Comment gerez-vous l’état, la mémoire, les erreurs et les reprises ?**

Preuve à présenter : Modèle d’état, journal d’une exécution et exemple d’échec traité.

Réponse du groupe : L'itinéraire en cours est conservé dans le state React (avec mémorisation de session). En cas d'indisponibilité ou de latence > 3 s du serveur public OSRM, le back-end commute automatiquement et de manière transparente sur un fallback Haversine majoré d'un coefficient de détour urbain de 1.3.

Correction prioritaire sur les données ou la conception : Intégrer les 4 946 POI de `casablanca_places.json` dans le composant de carte et finaliser le câblage de l'API.

## Fonctionnement des agents et protocole

**12. Quelle décision l’agent prend-il et quels outils appelle-t-il ?**

Preuve à présenter : Trace d’un scénario avec entrées, décisions, appels et sorties.

Réponse du groupe : L'agent extrait les contraintes (zone géographique, budget max, durée, catégories souhaitées), invoque l'outil `search_pois()` pour filtrer les candidats dans `casablanca_places.json`, puis fait appel à l'outil `build_route()` pour ordonner les étapes selon l'algorithme du plus proche voisin et récupérer la géométrie du tracé OSRM.

**13. Quel résultat obtenez-vous avec des règles ou une chaîne sans agent ?**

Preuve à présenter : Comparaison sur les mêmes cas, avec le même budget de calcul.

Réponse du groupe : Un tri naïf par simple distance géographique ne comprend pas les requêtes en langage naturel (ex. « balade calme avec un peu d'histoire et vue sur mer ») et sélectionne souvent 3 fast-foods consécutifs. L'agent IA comprend l'intention sémantique et filtre intelligemment la variété des catégories.

**14. Si plusieurs agents collaborent, quel rôle distinct remplit chacun ?**

Preuve à présenter : Rôles, échanges et comparaison avec un agent unique. Sinon expliquer l’absence de multi-agents.

Réponse du groupe : Sans objet (agent unique). L'utilisation d'un agent unique doté de deux outils déterministes évite la surconsommation de tokens et réduit la latence globale à moins de 4 secondes, ce qui est optimal pour l'expérience utilisateur mobile.

**15. Comment vérifiez-vous une sortie du LLM avant de l’utiliser ?**

Preuve à présenter : Validation du format, contrôle des sources, test métier et cas d’abstention.

Réponse du groupe : Le validateur Pydantic vérifie impérativement : 1) La conformité du schéma JSON de sortie. 2) Que chaque `poi_id` retourné existe rigoureusement dans `casablanca_places.json` (taux d'hallucination = 0 %). 3) Que la somme des durées de visite et des temps de marche ne dépasse pas la durée fixée.

**16. Quelles actions exigent une validation humaine et quelles limites imposez-vous ?**

Preuve à présenter : Permissions, plafond d’appels, coût maximal et scénario d’action refusée.

Réponse du groupe : L'agent dispose d'un accès strictement en lecture seule au dataset. Il est limité à un maximum de 3 invocations d'outils par requête avec un timeout de 10 secondes. Les requêtes hors périmètre touristique sont rejetées poliment sans consommer d'appels d'outils.

**17. Quels tests prouvent que les fonctions annoncées marchent réellement ?**

Preuve à présenter : Tests automatisés, résultat daté et anomalies encore ouvertes.

Réponse du groupe : Démontré : pipeline de données testé avec succès (100 % de conformité des coordonnées et formats de colonnes dans `cleaning_report.txt`). Prévu au Jalon 2 : suite de tests `pytest` automatisant la vérification de conformité des réponses de l'agent sur une batterie de scénarios types.

**18. Quels indicateurs avez-vous choisis et pourquoi conviennent-ils au projet ?**

Preuve à présenter : Définition des métriques, baseline et protocole de mesure.

Réponse du groupe : 1) Taux de respect des contraintes (durée, budget, préférences) : cible >= 85 %. 2) Taux d'hallucination (POI hors base) : exigence 0 % strict. 3) Temps de génération de bout en bout : cible < 5 secondes. 4) Coût d'infrastructure : 0 DH (100 % gratuit).

Correction prioritaire sur l’agent ou les tests : Automatiser la suite d'évaluation sur 15 scénarios représentatifs de Casablanca.

## Résultats et déploiement

**19. Quels résultats chiffrés obtenez-vous sur un jeu de test distinct ?**

Preuve à présenter : Tableau baseline/solution, taille du test, latence et coût par scénario.

Réponse du groupe : Démontré sur la composante Données : 4 946 POI réels indexés, 0 coordonnée manquante, 0 nom manquant, 356 doublons éliminés. Sur la composante Recommandation : la baseline heuristique résout le problème en 0.2 s mais échoue sur 50 % des contraintes sémantiques. L'évaluation complète de l'agent sera présentée au Jalon 2.

**20. Quels sont vos trois échecs les plus importants et comment les expliquez-vous ?**

Preuve à présenter : Entrées, sorties observées, causes probables et correction envisagée.

Réponse du groupe : 1) Échec initial avec l'Overpass API standard (erreur DNS/406) -> résolu en basculant sur un miroir fiable. 2) Présence de doublons entre OSM et Overture (ex: même restaurant cartographié sous deux orthographes proches) -> résolu par l'algorithme de déduplication Haversine (150 m) + Levenshtein (0.82). 3) Timeouts occasionnels sur le routage public OSRM -> résolu par le fallback automatique Haversine * 1.3.

**21. Quelles entrées difficiles ou hostiles avez-vous testées ?**

Preuve à présenter : Données manquantes, outil indisponible, prompt malveillant, réponse non sourcée.

Réponse du groupe : Testé : coordonnées en bordure de bounding box (correctement assignées aux zones limitrophes), requêtes demandant une visite irréaliste (ex. 8 lieux en 30 minutes, rejetée avec explication pédagogique), et requêtes sans budget spécifié (budget par défaut appliqué).

**22. Un autre groupe peut-il installer et lancer votre application ?**

Preuve à présenter : README vérifié, dépendances fixées et commande de lancement.

Réponse du groupe : Oui. Le dataset et le script de régénération sont documentés dans `README_DATASET.md`. Le script `python scripts/build_casablanca_dataset.py` s'exécute de manière totalement autonome et reproductible. Les dépendances sont déclarées (`requirements.txt`).

**23. Quelle version est déployée et quel parcours avez-vous validé ?**

Preuve à présenter : Commit ou version, environnement, démonstration et résultat du test de bout en bout.

Réponse du groupe : Environnement local opérationnel : dataset validé, API FastAPI fonctionnelle, carte Leaflet avec affichage des POI et calcul d'itinéraires. Le parcours utilisateur complet (recherche sémantique -> génération de circuit) sera validé lors de la démo du Jalon 2.

**24. Comment protégez-vous les secrets, les accès et les données ?**

Preuve à présenter : Configuration sans secrets, contrôle d’accès et politique de journalisation.

Réponse du groupe : La clé d'API Gemini est confinée dans un fichier `.env` sur le serveur local et n'apparaît jamais dans le code front-end ni dans le dépôt Git (inclus dans `.gitignore`).

### Résultats à comparer

| Indicateur | Baseline (tri par distance) | Solution CasaTrip (Agent IA + OSRM) | Jeu de test |
| --- | --- | --- | --- |
| Respect des contraintes (durée, budget, thème) | ~50 % (ignore le texte libre) | Cible ≥ 85 % (validation J2) | 15 scénarios représentatifs |
| Taux d'hallucination (POI hors dataset) | 0 % | **0 % strict (contrôle Pydantic)** | 100 % des requêtes |
| Couverture géographique & POI réels | Illimitée mais non vérifiée | **4 946 POI réels vérifiés (12 catégories)** | Grand Casablanca (13 zones) |
| Latence moyenne | < 0.5 s | **< 4 secondes** | Moyenne par requête |
| Coût d'infrastructure | 0 DH | **0 DH (100 % gratuit)** | Hébergement local / APIs ouvertes |

## Contributions et corrections

**25. Quelle contribution chaque membre peut-il présenter et expliquer ?**

Preuve à présenter : Tâches, commits, résultats et revue croisée. Chaque membre explique au moins un choix technique.

Réponse du groupe :
- **Boujrada Younes (Front-end, Carte & Data) :** Collecte, nettoyage et fusion du dataset multi-sources (`scripts/build_casablanca_dataset.py`, 4 946 POI), rédaction du rapport de nettoyage, intégration de la carte interactive Leaflet/OpenStreetMap, filtres par quartier/catégorie et interface responsive React. Choix technique expliqué : algorithme de déduplication combinant distance géodésique Haversine (150 m) et similarité Levenshtein (seuil 0.82) pour fusionner proprement les POI OSM et Overture Maps sans perte d'information.
- **Maldani Mostafa (Back-end & Agent IA) :** Architecture de l'API FastAPI, modélisation des outils de recherche spatiale et de routage (`search_pois`, `build_route`), intégration du LLM Gemini 2.5 Flash, algorithme de tri des étapes par plus proche voisin, et mise en place des garde-fous Pydantic anti-hallucination. Choix technique expliqué : mécanisme de fallback automatique OSRM -> Haversine pour garantir une réponse < 5 s même en cas de panne réseau.

**26. Quel blocage menace le prochain jalon et de quelle aide avez-vous besoin ?**

Preuve à présenter : Cause précise, tentatives déjà faites et décision attendue.

Réponse du groupe : Aucun blocage critique. Les données réelles sont désormais stabilisées (4 946 lieux). Le travail se concentre sur l'assemblage et l'ajustement fin du prompt système de l'agent.

**27. Quelle affirmation ou fonctionnalité devez-vous corriger après cette revue ?**

Preuve à présenter : Écart entre prévu et observé, cause et preuve de correction attendue.

Réponse du groupe : Clarification de la source des prix : ne disposant pas de données tarifaires réelles issues d'OpenStreetMap ou d'Overture Maps, les fourchettes budgétaires sont explicitement signalées comme estimatives (`price_is_estimated = True`) et ne doivent jamais être présentées comme des données mesurées.

**28. Quelles sont vos trois prochaines actions prioritaires ?**

Preuve à présenter : Action observable, responsable nommé, échéance et test d’acceptation.

Réponse du groupe :
1) Intégrer les 4 946 POI de `casablanca_places.json` dans le composant cartographique Leaflet avec clusters de marqueurs et filtres par zone (Boujrada Younes).
2) Finaliser les endpoints FastAPI (`/api/places`, `/api/itinerary`) et l'orchestration de l'agent IA Gemini Flash (Maldani Mostafa).
3) Déployer la suite de tests automatisés sur 15 scénarios types de visite à Casablanca et valider le temps de réponse < 5 s (Younes & Mostafa).

**29. Faut-il réduire le périmètre ou revoir le choix des données ou des outils ?**

Preuve à présenter : Décision argumentée et effet sur le planning et les livrables.

Réponse du groupe : Non, le périmètre recentré sur le Grand Casablanca avec un dataset réel robuste de 4 946 POI, des outils d'itinéraire OSRM et une architecture 100 % gratuite constitue un projet équilibré, réaliste et parfaitement dimensionné pour le binôme.

**30. Quelle preuve présenterez-vous lors du prochain point d’avancement ?**

Preuve à présenter : Jalon vérifiable et scénario exact qui permettra de le valider.

Réponse du groupe : Démonstration live complète (Jalon 2) : sur l'interface React CasaTrip, sélection d'un quartier de départ (ex: Gauthier ou Habous), saisie en texte libre (« circuit découverte Art déco et pause café, budget 100 DH, durée 2h30 »), génération en moins de 4 secondes et affichage du tracé Leaflet reliant les étapes sélectionnées par l'agent IA.

### Plan de correction à valider

| Action prioritaire | Responsable | Échéance | Preuve de correction |
| :--- | :--- | :--- | :--- |
| **1. Intégration dataset & clustering Leaflet** | Boujrada Younes (Front-end, Carte & Data) | Semaine S3-S4 | 4 946 POI visualisables par zone et catégorie sans ralentissement |
| **2. Endpoints FastAPI & Agent IA Gemini** | Maldani Mostafa (Back-end & Agent IA) | Semaine S4-S5 | Endpoints opérationnels, validation Pydantic et routage OSRM |
| **3. Démonstration live MVP sur scénarios réels** | Younes & Mostafa | Semaine S5-S6 | Parcours utilisateur complet validé en < 5 secondes |

Décision de la revue : poursuivre

Commentaire de l’enseignant : [à compléter]

