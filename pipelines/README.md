# Pipelines ETL

Cette section regroupe des exemples de pipelines d'ingestion et de transformation de donnees, orchestres avec Airflow et charges dans BigQuery.

## Architecture type
Les donnees sources (API, fichiers, bases applicatives) sont extraites par des taches planifiees, deposees brutes dans une zone de staging sur BigQuery, puis transformees en tables propres et modelisees pour l'analyse. Chaque etape est idempotente : relancer une execution ne duplique pas les donnees.

## Orchestration avec Airflow
Chaque pipeline est un DAG. Les taches sont decoupees par responsabilite : extraction, chargement en staging, transformation, controle qualite. Les dependances entre taches garantissent qu'une transformation ne demarre jamais sur des donnees incompletes. Les echecs sont isoles par tache pour pouvoir rejouer seulement ce qui a casse.

## Couche de transformation
Les transformations sont ecrites en SQL versionne, organisees en couches : brut, intermediaire, puis tables metier. Cette separation rend le modele lisible et testable, et permet de retracer d'ou vient chaque chiffre.

## Controle qualite
Avant publication, des controles verifient la fraicheur des donnees, l'absence de doublons sur les cles, et la coherence des volumes par rapport a l'historique. Un controle qui echoue bloque la publication plutot que de diffuser des donnees fausses.

## Contenu a venir
Un exemple de DAG Airflow commente et un jeu de requetes BigQuery sont ajoutes dans les documents suivants.
