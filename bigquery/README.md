# Requetes BigQuery

Exemples de requetes et de bonnes pratiques BigQuery utilisees dans la couche de transformation.

## Modelisation en couches
Les donnees brutes atterrissent dans un dataset de staging. Des vues et des tables materialisees construisent ensuite une couche intermediaire nettoyee, puis une couche metier prete pour les tableaux de bord. Chaque couche a son propre dataset pour clarifier les droits d'acces et les couts.

## Maitrise des couts
BigQuery facture les octets lus. Les bonnes pratiques appliquees ici : partitionner les grandes tables par date, clusteriser sur les colonnes les plus filtrees, ne jamais selectionner toutes les colonnes en production, et preferer des tables materialisees aux vues quand une meme transformation est relue souvent.

## Deduplication
Un motif recurrent : garder la derniere version de chaque enregistrement. On numerote les lignes par cle en triant par date de mise a jour decroissante, puis on ne conserve que la premiere. Cela remplace les doublons d'ingestion par une table propre sans perdre l'historique en staging.

## Controles integres
Des requetes de controle tournent apres chaque chargement : compter les cles en double, mesurer l'ecart de volume avec la veille, et verifier que les dates ne depassent pas aujourd'hui. Leur resultat alimente le controle qualite du pipeline.

## A venir
Un fichier de requetes commentees, pretes a adapter, est ajoute a ce dossier.
