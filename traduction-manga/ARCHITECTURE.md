# Architecture — couche de traduction

## Point d'entree unique

Toute la traduction passe par un seul appel : donner une consigne systeme, le
texte a traduire, l'image de la page (le modele voit la planche) et une taille de
reponse ; recevoir une traduction structuree en JSON. Le reste du pipeline
(detection, effacement, lettrage) ignore quel moteur repond.

## Etat actuel

Un seul parametre de configuration choisit le moteur :

- **Gemini** (par defaut, offre gratuite) : le code tente plusieurs modeles dans
  l'ordre (par ex. un modele rapide, puis un modele plus leger), bascule tout seul
  quand un modele est sature (erreur 429 par minute), marque les modeles dont le
  quota du jour est epuise, et repete avec des pauses. C'est deja une logique de
  repli, mais interne a un seul fournisseur.
- **Claude** (option payante) : meme interface, pour la meilleure qualite quand on
  en a besoin.

Les cles d'API restent hors du depot (fichier d'environnement local).

## Cible : vraie abstraction multi-fournisseurs

Garder le meme point d'entree et ajouter des adaptateurs interchangeables :

- **Fournisseurs distants** : Gemini, OpenAI, Anthropic, Mistral. Chaque
  adaptateur traduit l'interface commune vers l'API du fournisseur et renvoie, en
  plus de la traduction, un indicateur de confiance.
- **Modele maison local** : un modele de traduction execute sur le GPU, specialise
  sur le vocabulaire et le ton du scantrad (registres, honorifiques, conventions
  de ponctuation). Meme interface, sans appel reseau ni cout par page.

## Politique de selection et de repli

1. Le modele maison traite le segment en premier (cout nul, hors-ligne,
   confidentiel).
2. Si sa confiance est basse, ou si le segment est marque difficile (langue rare,
   jeu de mots, onomatopee), on bascule vers un fournisseur distant.
3. En cas d'echec ou d'indisponibilite d'un moteur, on passe au suivant dans un
   ordre de repli defini — en reutilisant la logique de bascule et de quota deja
   presente pour Gemini, mais etendue a l'ensemble des fournisseurs.

Le moteur et les seuils de confiance sont des parametres, pas du code en dur, pour
pouvoir comparer les moteurs sans rien reecrire.

## Coherence sur une page

La traduction depend du contexte : noms propres, tutoiement, registre. On traduit
donc une page entiere avec son image et le glossaire de la serie en contexte,
plutot que bulle par bulle isolee.

## Boucle d'amelioration du modele maison

L'interface de correction produit, pour chaque bulle corrigee, un triplet
precieux : texte source, traduction proposee, traduction validee. Collectes, ces
triplets forment un corpus propre au style du projet, utilisable pour affiner le
modele maison et reduire progressivement le recours aux fournisseurs distants.

## Mesure de la qualite

Garder quelques pages de reference traduites a la main permet de comparer
objectivement les moteurs entre eux (fournisseurs distants et modele maison) a
chaque evolution.
