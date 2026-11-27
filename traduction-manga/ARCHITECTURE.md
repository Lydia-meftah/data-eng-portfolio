# Architecture — couche de traduction

## Principe

Un seul contrat pour tous les moteurs de traduction. Le reste du pipeline
(detection, OCR, effacement, lettrage) ignore quel moteur est utilise : il envoie
des segments et recoit des traductions.

## Interface commune

Chaque moteur expose la meme operation : prendre un segment (texte source, langue
source, langue cible, et un peu de contexte) et renvoyer une traduction avec un
score de confiance. Un segment porte au minimum :

- le texte source et sa langue detectee a l'OCR ;
- la langue cible (francais) ;
- un contexte optionnel : type de zone (dialogue, recitatif, onomatopee),
  et les segments voisins de la meme planche pour la coherence.

## Moteurs

- **Fournisseurs distants** : Gemini, OpenAI, Anthropic, Mistral. Chaque fournisseur
  est un adaptateur qui traduit l'interface commune vers son API. On garde une cle
  par fournisseur, hors du code (variables d'environnement).
- **Modele maison local** : un modele de traduction execute en local, specialise sur
  le domaine scantrad. Meme interface que les fournisseurs distants, mais sans appel
  reseau ni cout par page.

## Selection et repli

1. Le modele maison traite le segment en premier.
2. Si sa confiance est sous un seuil, ou si le segment est marque difficile
   (langue rare, jeu de mots, onomatopee), on bascule vers un fournisseur distant.
3. En cas d'echec ou d'indisponibilite d'un moteur, on passe au suivant dans un
   ordre de repli defini.

Ce fonctionnement garde le cout bas par defaut tout en gardant la qualite des gros
modeles disponible au besoin.

## Coherence sur une planche

La traduction d'un segment peut dependre de ses voisins (noms propres, tutoiement,
registre). On traduit donc une planche comme un lot, en passant les segments
deja traduits en contexte, plutot que segment par segment isole.

## Boucle d'amelioration du modele maison

Les corrections faites au lettrage et a la relecture sont une source de donnees
precieuse. En les collectant (source, traduction proposee, traduction corrigee),
on constitue un corpus propre au style du projet, utilisable pour affiner le modele
maison et reduire progressivement le recours aux fournisseurs distants.

## Points d'attention

- Les cles d'API restent hors du depot (environnement, secrets).
- Le choix du moteur et les seuils de confiance sont des parametres, pas du code en
  dur, pour pouvoir comparer les moteurs sans rien reecrire.
- Mesurer la qualite : garder quelques planches de reference traduites a la main
  pour comparer objectivement les moteurs entre eux.
