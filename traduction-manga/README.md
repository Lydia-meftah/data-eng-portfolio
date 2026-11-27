# Traduction manga / webtoon — pipeline IA

Application locale qui traduit un chapitre (coreen, anglais, japonais, chinois
vers le francais) **sans abimer le dessin**, puis le met en forme et le publie.
Elle tourne entierement sur le PC (127.0.0.1, protegee par mot de passe) et
s'appuie sur un GPU NVIDIA pour les modeles de vision.

Ce dossier documente l'architecture du projet au niveau conception (pas de code
ni de secret) : c'est un exemple d'orchestration d'une chaine IA multi-etapes et
multi-fournisseurs.

## Chaine de traitement

1. **Detection** — un detecteur specialise bande dessinee (RT-DETR-v2, entraine
   sur des milliers de pages manga / webtoon / comics) repere chaque bulle, le
   texte dans les bulles et le texte hors bulle ; un OCR (EasyOCR) extrait les
   lignes exactes et leur langue.
2. **Effacement (inpainting)** — une bulle unie est repeinte avec sa couleur
   exacte ; le texte pose sur le dessin est reconstruit par un modele LaMa
   affine sur un grand corpus manga / anime, sur le GPU local.
3. **Traduction** — un LLM recoit la page entiere (pour saisir qui parle, le ton,
   le contexte) et le glossaire de la serie, puis renvoie les traductions en JSON.
4. **Lettrage** — le francais epouse la forme reelle de la bulle (lignes plus
   courtes en haut et en bas), avec une taille de police homogene sur la page.
5. **Relecture / correction** — une interface liste chaque bulle (texte d'origine
   relu + traduction) ; on corrige ou on ignore une zone, puis on re-lettre la
   page instantanement, sans rappeler le LLM.
6. **Publication** — les pages validees sont poussees vers le depot du site, qui
   se redeploie tout seul.

## Glossaire par serie

Un glossaire par serie garde la coherence d'un chapitre a l'autre : noms propres,
termes imposes, et notes de style (tutoiement, honorifiques). Il est enrichi au
fil des chapitres et passe en contexte au LLM.

## Evolution visee : multi-fournisseurs + modele maison

Aujourd'hui la traduction passe par un moteur choisi en configuration (Gemini par
defaut, avec bascule automatique entre modeles Gemini quand l'un sature ; Claude
en option payante pour la meilleure qualite). L'objectif est d'ouvrir cette couche
a plusieurs fournisseurs (Gemini, OpenAI, Anthropic, Mistral) **et** a un modele
maison local specialise scantrad, derriere une interface commune. Detail dans
ARCHITECTURE.md.

## Pourquoi c'est un projet data / IA interessant

- Orchestration d'une chaine multi-etapes (vision, OCR, inpainting, NLP) avec
  reprise possible a chaque etape.
- Couche LLM unifiee, deja tolerante aux pannes (bascule de modeles, gestion de
  quota, reprise), a etendre en vraie abstraction multi-fournisseurs.
- Boucle d'amelioration : les corrections de relecture forment un corpus propre au
  style du projet, reutilisable pour affiner un modele maison.
