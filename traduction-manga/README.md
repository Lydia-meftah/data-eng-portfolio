# Traduction manga / webtoon — pipeline IA

Pipeline local de traduction et de localisation de planches (manga, manhwa, webtoon)
du coreen, anglais, japonais ou chinois vers le francais. L'objectif est une
traduction de qualite et, a terme, le redessin propre du texte sur l'image, en
gardant la main sur le cout et la confidentialite grace a une execution locale.

## Chaine de traitement

1. **Detection des bulles et des zones de texte** — reperer sur la planche les
   bulles, encadres et onomatopees a traiter.
2. **OCR** — extraire le texte source de chaque zone, avec sa langue d'origine.
3. **Effacement (inpainting)** — retirer le texte original de l'image sans abimer
   le dessin, via un modele d'inpainting (LaMa).
4. **Traduction** — traduire chaque segment vers le francais (voir la couche
   multi-fournisseurs ci-dessous).
5. **Lettrage** — replacer le texte traduit dans la zone, avec une police et une
   mise en forme adaptees au support.
6. **Export / publication** — assembler les planches traduites pour relecture
   puis diffusion.

## Couche de traduction multi-fournisseurs + IA maison

Le coeur du projet est une couche de traduction interchangeable. Chaque segment
peut etre traduit par un fournisseur distant (Gemini, OpenAI, Anthropic, Mistral)
ou par un **modele maison local**, derriere une interface commune. On choisit le
moteur par un simple parametre, sans toucher au reste de la chaine.

- **Fournisseurs distants** — meilleure qualite brute sur les langues difficiles,
  utiles comme reference et pour les cas complexes.
- **Modele maison local** — execution hors-ligne, cout nul par page, confidentialite
  totale, et surtout specialisable sur le vocabulaire et le ton du scantrad
  (registres, onomatopees, conventions de ponctuation).

Strategie visee : le modele maison traite la majorite des segments ; un fournisseur
distant sert de secours sur les passages ou la confiance est basse, et de corpus de
reference pour ameliorer le modele maison au fil du temps.

## Pourquoi c'est un projet data / IA interessant

- Orchestration d'une chaine multi-etapes (vision, OCR, inpainting, NLP) avec des
  points de reprise par etape.
- Abstraction multi-fournisseurs avec bascule et repli, et comparaison de qualite
  entre moteurs.
- Boucle d'amelioration : collecter les corrections de relecture pour affiner le
  modele maison (fine-tuning / adaptation de domaine).

## Statut

Prototype personnel autour du scantrad. Ce dossier decrit l'architecture cible ;
le detail de la couche multi-fournisseurs est dans ARCHITECTURE.md.
