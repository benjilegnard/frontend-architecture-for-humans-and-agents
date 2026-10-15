# Renforçons nos règles d'architecture frontend,<br/>pour les humains comme pour les agents 🏗️👮🤖

*Votre architecture n'existe que si elle est exécutable*

Titres alternatifs :
- Des règles d'architecture que personne ne peut contourner, pas même votre agent
- Architecture front-end exécutable : Sheriff, ts-arch et la boucle de feedback des agents

Format : conférence 45 minutes (≈40 min de contenu + Q&A).
Niveau : intermédiaire

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
des règles exécutables. Pas de blocs de démo isolés : la codebase se dégrade et
se répare bloc par bloc en section 3, le live outillé est en section 4 et la
démo agent en section 5.

Budget : 2 + 5 + 16 + 6 + 7 + 3 = 39 min de contenu, ≈6 min de Q&A.

## Plan détaillé

- **1. Intro et cadrage (2 min)**
  - qui parle, pourquoi ce sujet
  - storytelling: une armée d'agents/de juniors se lance dans la réal d'un projet
  - features `catalog`, `basket`, `orders`, puis `checkout` qui utilise les trois, et 6 mois plus tard c'est la merde
  - la thèse en une phrase : une règle d'architecture non exécutable n'existe pas
  - vos docs d'archis, adr et nfr non-exprimés ne suffisent pas
  - périmètre : Front, TypeScript d'abord, exemples Angular, ce qui est transposable à React / Vue, et au back.
- **2. Pourquoi on découpe, et ce que ça coûte (5 min)**
  - pourquoi on fait de l'architecture ? pour avoir un vocabulaire commun
  - on mets des trucs dans des boites, on a des mots, les mots on un sens (apparté: les mots ont un sens/les tokens ont un poids)
  - ce qu'on protège vraiment : le coût du changement, pas la beauté du schéma
  - le rayon d'explosion : combien de fichiers touche une modif "simple"
  - onboarding : le temps pour qu'un nouveau (humain ou agent) trouve où écrire son code
  - le coût de l'archi : boilerplate, indirection, API publiques, cérémonie
  - nos propres limites cognitves, qui si l'on ne fait pas attention vont détruire la société
  - quand ne PAS faire ça : POC, appli jetable, dev solo, projet à durée de vie de moins d'un an
- **3. Des antipatterns aux règles (16 min)**
  - format imposé pour chaque bloc : antipattern → pattern → règle
    - antipattern : le symptôme et ce qu'il coûte, montré sur la codebase de l'intro (`catalog`, `basket`, `orders`, `checkout`)
    - pattern : ce qui le corrige
    - règle : un snippet exécutable de 3 à 5 lignes
  - progression du plus petit au plus grand : un composant → une feature → les features entre elles
  - la formule qui cadre la section : la règle n'attrape pas toujours l'antipattern, elle verrouille le pattern
  - **3.1 Le composant qui fait tout (4 min)**
    - antipattern : data access + état + logique métier + template dans un composant de 600 lignes ; intestable, non réutilisable, un agent qui le modifie touche à tout
    - pattern : smart / dumb porté par le nommage, `-page` orchestre, `-card` (et les autres composants de présentation) affichent
    - pattern : le composant éclaté en 4 rôles, ui / model / state / client
    - pattern : le nommage comme intention lisible par une machine (`-page.ts`, `-card.ts`, `-store.ts`, `-client.ts`)
    - règle : ESLint `no-restricted-imports`, `HttpClient` interdit hors des fichiers `*-client.ts`
    - règle : ts-arch, les `*-card.ts` ne dépendent ni des `*-client.ts` ni des `*-store.ts`
  - **3.2 Le props drilling (3 min)**
    - antipattern : un `@Input` qui traverse 6 à 15 couches ; chaque composant intermédiaire connaît une donnée dont il n'a pas besoin, ajouter un champ modifie toute la chaîne
    - pattern : le symptôme d'une gestion d'état absente, on introduit un `-store.ts` (signal store) ; seul le `-page` injecte le store, les composants dumb restent en inputs / outputs
    - règle : ts-arch, `ui/**` ne dépend pas de `state/**`
    - elle ne détecte pas le drilling, elle verrouille le pattern qui le rend inutile (première application de la formule)
  - **3.3 Organiser une feature (3 min)**
    - antipattern : le découpage par type technique (`components/`, `services/`, `pipes/`)
    - antipattern : l'arborescence qui recopie l'arbre de composants, et casse à chaque refacto d'UI
    - même cause pour les deux : on range par technique ou par UI, pas par domaine
    - pattern : d'abord le domaine, ensuite le rôle ; les 4 rôles de 3.1 deviennent des dossiers (`orders/feature`, `orders/ui`, `orders/state`, `orders/client`, `orders/model`)
    - pattern : c'est quoi une feature, et où s'arrête-t-elle
    - règle : Sheriff, tags `domain:orders` et `type:feature|ui|state|client|model`, `depRules` entre types
      - `feature` → `ui`, `state`, `model`
      - `ui` → `model`
      - `state` → `client`, `model`
      - `client` → `model`
  - **3.4 Ne pas mélanger les features (5 min)**
    - le cœur du storytelling de l'intro : `checkout` qui utilise `catalog`, `basket` et `orders`
    - antipattern : les deep imports, `checkout` va piocher `orders/state/internal-helper.ts`
    - antipattern : les dépendances circulaires, `catalog` importe `basket` qui importe `catalog`
    - antipattern : le god bucket, `shared/` devient le dépotoir de ce qu'on n'a pas su ranger et toute l'app en dépend
    - pattern : API publique de module, `index.ts` comme seule porte d'entrée, on choisit ce qu'on n'exporte pas
    - pattern : le sens des dépendances, unidirectionnel, et pourquoi
    - pattern : quand `checkout` a besoin de `catalog` et `basket`, on remonte dans un module partagé typé ou on orchestre au niveau de `checkout`, jamais de lien horizontal `catalog` ↔ `basket`
    - pattern : `shared/` sous discipline, `domain:shared` n'accepte que `ui`, `model` et `util`, jamais `state` ni `feature`
    - slide de clarification sur les barrels : le barrel interne qui ré-exporte tout cache le couplage (antipattern), le barrel `index.ts` comme API publique est un contrat (pattern) ; teaser de la limite ts-arch (section 6)
    - règle : Sheriff, encapsulation par `index.ts` (les deep imports sont bloqués par défaut) et règles `domain:*`
    - règle : ts-arch `shouldBeFreeOfCycles()` (ou `import/no-cycle`) pour les cycles
  - **clôture (1 min)**
    - la règle d'or : si vous ne savez pas nommer la contrainte, vous ne saurez pas l'outiller
    - transition : "on vient d'écrire 6 règles, laquelle tourne où, et quand ?"
- **4. Des règles écrites aux règles exécutables (6 min)**
  - le vrai critère de choix : ce que l'outil voit (portée), puis quand il vous le dit (moment)
    - ESLint natif (angular-eslint, `no-restricted-imports`, `@ngrx/eslint-plugin`) : voit un fichier et ses imports directs ; à la frappe, pre-commit, CI
    - Sheriff : voit le graphe de modules (dossiers avec `index.ts`, tags, `depRules`, encapsulation) ; à la frappe via son plugin ESLint, en CLI (`sheriff verify`) en CI
    - ts-arch : voit les fichiers et leurs conventions dans tout le projet (nommage, dossiers, cycles), y compris à l'intérieur d'un module ; au run de tests uniquement, pas de feedback à la frappe
    - alternatives : eslint-plugin-boundaries (frontières par chemins, en pur ESLint), dependency-cruiser (graphe complet, CLI, pas
  d'IDE), nx `enforce-module-boundaries` (si vous êtes déjà en monorepo nx)
    - CI : le filet de sécurité, jamais la boucle de feedback principale
  - la grille de décision : ce que chaque outil attrape, ce qu'il rate, ce que coûte son adoption
  - les snippets ont déjà été montrés en section 3 : ici, un seul live (optionnel) qui casse une règle de 3.4 sur le fil rouge
  - le setup réel : `tsconfig.arch.json`, le runner de tests, les scripts npm, le job CI
- **5. La boucle de feedback des agents (7 min)**
  - pourquoi un agent dérive : il optimise "du code qui marche", pas "du code qui respecte vos frontières"
  - les règles comme contexte partagé humains / agents : `AGENTS.md` pour donner les commandes de vérifications, hook et CI pour les empêcher. 
  - `.skills/` pour cadrer les compétences / ce que connait l'agent (partagez-les!).
  - le context layering : always-on (`AGENTS.md`, `CLAUDE.md`) vs task-specific (`docs/architecture-boundaries.md` / skills)
  - le stop hook : un `ci-checks.mjs` commun, la traduction du résultat par outil (exit code 2 pour Claude Code, JSON sur stdout pour Cursor)
  - démo : l'agent viole une frontière, le hook le rattrape, il se corrige tout seul
  - le mode d'échec qui compte : donnez un test à faire passer à un agent, il finira par modifier le test
    - relâcher la contrainte d'archi plutôt que corriger le code (les règles de 3.4)
    - renommer un fichier pour échapper au pattern de suffixe (les règles de 3.1 et 3.2)
    - la parade : propose-and-confirm, jamais de relâchement silencieux de règle (CODEOWNERS, deny en écriture sur les fichiers de règles)
  - on ne fait que mitiger les problèmes, mais c'est déjà ça de pris.
- **6. Limites, et plan du lundi matin (3 min)**
  - ce que ces outils ne savent pas faire
    - les suffixes sont une intention déclarée, pas une garantie
    - les re-exports en barrel masquent les chaînes de dépendances à ts-arch
    - les messages d'erreur sont laconiques (`source -> target`), ça compte quand le lecteur est un agent
  - les échappatoires : comment on déroge, comment une règle évolue, qui valide
  - le plan en 3 étapes
    - taguer ses modules et activer les frontières
    - n'encoder que les 3 règles qui correspondent à une douleur déjà vécue
    - brancher le stop hook et la CI
  - activer des régles de lint test sur une codebase existante : tips & tricks (eslint suppressions)
  - conclusion : une règle non exécutable n'est pas une règle, c'est un souhait

## Tranché

- framework : Angular assumé pour les exemples ; Sheriff, ts-arch et ESLint s'appliquent à tout l'écosystème Node.js (à reporter dans un slide)
- les démos agent sont non déterministes : scripter le terminal ou pré-enregistrer avec narration live, et ne garder en vrai live que la partie déterministe (Sheriff / ts-arch qui échoue)

## Hors périmètre (assumé)

nx / turborepo comme outillage monorepo, DDD, design system : trois autres talks.
`@nx/enforce-module-boundaries` n'apparaît que dans la grille de comparaison.

## Sources

- https://github.com/ts-arch/ts-arch
- https://github.com/softarc-consulting/sheriff
- https://www.angulararchitects.io/en/blog/reliable-angular-architectures-with-ai-assisted-coding/
- https://www.angulararchitects.io/en/blog/architecture-beyond-layers-tsarch-for-ai-agents/

## Citations

> Affabulating... I'll look into the codebase for existing patterns... 
> *Claude*

> Chaque ligne de code écrite est un échec.

> echo "Make no mistakes" >> AGENTS.md
> *David Goodenough*

> un dev dans son coin, avec son llm, ne font pas une équipe
> un llm dans son coin, avec son dev, ne font pas une convention
> *BenjiLeGnard*

> On fait de l'architecture pour se protéger des gens qui n'en ont rien à foutre
> Un LLM de base n'en a rien à foutre
> *BenjiLeGnard
