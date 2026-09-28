# Votre architecture n'existe que si elle est exécutable

*Contraindre l'architecture front-end, pour les humains comme pour les agents.*

Titres alternatifs :
- Des règles d'architecture que personne ne peut contourner, pas même votre agent
- Architecture front-end exécutable : Sheriff, ts-arch et la boucle de feedback des agents

Format : conférence 45 minutes (≈42 min de contenu + Q&A).

## Abstract

```
Votre architecture front-end n'existe que si elle est exécutable.

Les schémas dans Confluence, les ADR, les conventions posées au kick-off :
tout ça s'érode, commit après commit. Depuis que des agents de code écrivent
une partie du code, l'érosion a juste changé de vitesse : un agent produit du
code qui marche, pas du code qui respecte vos frontières.

On commencera par la question de fond : c'est quoi une bonne architecture
front-end (et non, ce n'est pas de l'arrache-i-gonale) ? Le catalogue des
antipatterns qu'on croise tous, ce qu'ils coûtent vraiment, et les quelques
patterns qui tiennent dans la durée.

Ensuite, on passe des règles écrites aux règles exécutables : Sheriff pour les
frontières de modules, ts-arch (l'ArchUnit du TypeScript) pour les contraintes
à l'intérieur d'une feature, ESLint pour le reste. On branche tout ça sur la CI
et sur la boucle de feedback des agents, pour voir un agent corriger lui-même
sa violation d'architecture. Et on parlera de ce que ces outils ne savent pas
faire, parce qu'un agent à qui vous donnez un test à faire passer finira par
essayer de modifier le test.
```

## Fil rouge

Une seule codebase pendant tout le talk : on la dégrade, puis on la répare avec
des règles exécutables. Pas de blocs de démo isolés, les démos sont réparties
dans les sections 5 et 6.

## Plan détaillé

- **1. Intro et cadrage (2 min)**
  - qui parle, pourquoi ce sujet
  - périmètre : TypeScript d'abord, exemples Angular, ce qui est transposable à React / Vue
  - à qui ça s'adresse : équipe de 3+ devs, appli qui vit 2+ ans
  - la thèse en une phrase : une règle d'architecture non exécutable n'existe pas
- **2. Pourquoi on découpe, et ce que ça coûte (5 min)**
  - ce qu'on protège vraiment : le coût du changement, pas la beauté du schéma
  - la courbe : coût d'une feature à 3 mois vs à 3 ans
  - le rayon d'explosion : combien de fichiers touche une modif "simple"
  - onboarding : le temps pour qu'un nouveau (humain ou agent) trouve où écrire son code
  - le coût de l'archi : boilerplate, indirection, API publiques, cérémonie
  - quand ne PAS faire ça : POC, appli jetable, dev solo
- **3. Le catalogue des antipatterns (10 min)**
  - format imposé pour chaque antipattern : symptôme, ce que ça coûte, la règle qui l'aurait attrapé
  - tout dans un composant (data access + état + logique métier + template)
  - le découpage par type technique (`components/`, `services/`, `pipes/`) au lieu du découpage par feature
  - l'arborescence de fichiers qui recopie l'arbre de composants
  - le god bucket : `shared/`, `common/`, `utils.ts`
  - les dépendances circulaires entre features
  - les barrel files qui masquent le couplage
  - les deep imports cross-feature (aller piocher dans les entrailles du voisin)
  - le props drilling sur 15 couches, faute d'état ou de service
  - l'antipattern de l'ère des agents : la duplication, parce que l'agent ignore que votre helper existe déjà
  - transition : chaque ligne de ce catalogue est une règle qu'une machine peut vérifier
- **4. Les patterns qui tiennent (8 min)**
  - feature slices : c'est quoi une "feature", et où s'arrête-t-elle
  - API publique de module : `index.ts` comme seule porte d'entrée
  - le sens des dépendances : qui a le droit d'appeler qui, et pourquoi c'est unidirectionnel
  - les types de modules par tags : `domain:x` et `type:feature|ui|data|util`
  - smart / dumb, stores, coordinators : les blocs de construction à l'intérieur d'une feature
  - les conventions de nommage comme intention lisible par une machine (suffixes `-store.ts`, `-client.ts`, `-page`, `-card`)
  - la règle d'or : si vous ne savez pas nommer la contrainte, vous ne saurez pas l'outiller
- **5. Des règles écrites aux règles exécutables (10 min)**
  - l'échelle de latence du feedback, c'est le vrai critère de choix
    - ESLint : à la frappe, portée locale au fichier
    - Sheriff : dans l'IDE et en CLI, portée graphe d'imports (domaines, couches, encapsulation)
    - ts-arch : au run de tests, portée intra-feature (conventions de nommage, cycles, slices PlantUML)
    - nx enforce-module-boundaries : portée workspace, si vous êtes déjà en monorepo nx
    - CI : le filet de sécurité, jamais la boucle de feedback principale
  - la grille de décision : ce que chaque outil attrape, ce qu'il rate, ce que coûte son adoption
  - live : écrire une règle Sheriff (tags + dependencyRules) et la faire échouer
  - live : écrire une règle ts-arch (`filesOfProject().matchingPattern().shouldNot().dependOnFiles()`) et la faire échouer
  - le setup réel : `tsconfig.arch.json`, le runner de tests, les scripts npm, le job CI
  - bonus si le temps le permet : le diagramme PlantUML comme source de vérité des slices
- **6. La boucle de feedback des agents (7 min)**
  - pourquoi un agent dérive : il optimise "du code qui marche", pas "du code qui respecte vos frontières"
  - les règles comme contexte partagé humains / agents : `AGENTS.md` pour les lignes rouges courtes
  - le context layering : always-on (`AGENTS.md`, `CLAUDE.md`) vs task-specific (`docs/architecture-boundaries.md`)
  - le stop hook : un `ci-checks.mjs` commun, la traduction du résultat par outil (exit code 2 pour Claude Code, JSON sur stdout pour Cursor)
  - démo : l'agent viole une frontière, le hook le rattrape, il se corrige tout seul
  - le mode d'échec qui compte : donnez un test à faire passer à un agent, il finira par modifier le test
    - relâcher la contrainte d'archi plutôt que corriger le code
    - renommer un fichier pour échapper au pattern de suffixe
    - la parade : propose-and-confirm, jamais de relâchement silencieux de règle
- **7. Limites, et plan du lundi matin (3 min)**
  - ce que ces outils ne savent pas faire
    - les suffixes sont une intention déclarée, pas une garantie
    - les re-exports en barrel masquent les chaînes de dépendances à ts-arch
    - les messages d'erreur sont laconiques (`source -> target`), ça compte quand le lecteur est un agent
    - santé des projets à revérifier la semaine du talk : dernière release npm de `tsarch` en 5.4.1 (décembre 2024) alors que le repo committe encore en 2026 ; Sheriff en 0.19.6 (septembre 2025)
  - les échappatoires : comment on déroge, comment une règle évolue, qui valide
  - le plan en 3 étapes
    - taguer ses modules et activer les frontières
    - n'encoder que les 3 règles qui correspondent à une douleur déjà vécue
    - brancher le stop hook et la CI
  - conclusion : une règle non exécutable n'est pas une règle, c'est un souhait

## À trancher avant de soumettre

- périmètre framework : Angular assumé, ou démo neutre en TypeScript ? (les 3 sources sont Angular, Sheriff et ts-arch sont agnostiques)
- retour d'expérience : est-ce qu'on a un vrai projet avec des chiffres avant / après, ou est-ce qu'on assume l'hypothèse ?
- les démos agent sont non déterministes : scripter le terminal ou pré-enregistrer avec narration live, et ne garder en vrai live que la partie déterministe (Sheriff / ts-arch qui échoue)

## Hors périmètre (assumé)

nx / turborepo comme outillage monorepo, DDD, design system : trois autres talks.
`@nx/enforce-module-boundaries` n'apparaît que dans la grille de comparaison.

## Sources

- https://github.com/ts-arch/ts-arch
- https://github.com/softarc-consulting/sheriff
- https://www.angulararchitects.io/en/blog/reliable-angular-architectures-with-ai-assisted-coding/
- https://www.angulararchitects.io/en/blog/architecture-beyond-layers-tsarch-for-ai-agents/
