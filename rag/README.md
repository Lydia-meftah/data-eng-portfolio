# Mini-demo RAG

Un exemple de pipeline de RAG (Retrieval Augmented Generation) pour interroger une base documentaire en langage naturel, compatible avec les API OpenAI, Anthropic et Mistral.

## Principe
Les documents sources sont decoupes en passages, convertis en vecteurs par un modele d'embedding, et stockes dans un index. A chaque question, on recupere les passages les plus proches de la question, puis on les fournit en contexte a un LLM qui redige une reponse citant ses sources.

## Ingestion
Les documents sont nettoyes puis decoupes en morceaux de taille raisonnable, avec un leger chevauchement pour ne pas couper une idee en deux. Chaque morceau garde une reference vers son document d'origine, afin de pouvoir citer la source dans la reponse finale.

## Recherche
La question est vectorisee avec le meme modele d'embedding que les documents. On recupere les quelques passages les plus pertinents par similarite, en gardant assez de contexte sans noyer le modele sous trop de texte.

## Generation
Le LLM recoit la question et les passages retrouves, avec une consigne claire : repondre uniquement a partir du contexte fourni, et dire quand l'information est absente plutot que d'inventer. Le choix du fournisseur (OpenAI, Anthropic, Mistral) est un simple parametre, le reste du pipeline ne change pas.

## Garde-fous
Limiter la taille du contexte, filtrer les passages trop peu pertinents, et toujours afficher les sources. Ces regles reduisent les reponses hors sujet et rendent le systeme verifiable.
