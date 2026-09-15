# Renforcer les règles d'architecture front-end pour les agents comme pour les humains

## Abstract

```
Constat : les agents de code sont peut-être bon pour générer une approximation médiocre de l'état de l'art du dev front, mais sont toujours très mauvais en architecture logicielle.

Dans ce court talk, on va déjà se poser la question de "qu'est-ce qu'une bonne architecture front-end ?" (et non, ce n'est pas de l'arrache-i-gonale). Quels sont les pièges à éviter et qu'est ce qu'on veut empêcher comme erreurs avec une bonne architecture.

Et surtout, comment mettre en place des tests de validation d'architecture avec `ts-arch`, et comment les brancher à vos agents de code et à votre CI/CD.
```

## Plan

Version quickie

### 1. old man yells at clouds (5 minutes)

dissertation / râlage sur bonne vs mauvaise architecture, présence ou absence d'architecture.
antipattern : tout dans un composant
antipattern : arborescence de composants qui se retrouve dans le file system
antipattern : not managing state / not using services
antipattern : props drilling (input/outputs sur 15 couches)

### 2. ts-arch, comme ArchUnit (java), mais en TypeScript ( 5 minutes )
good pattern : types de modules / types de composants 
good pattern : limites et contraintes, sens des dépendances
good pattern : répertoires par fonctionnalités
présentation de l'outil, api, rédiger une contrainte

### 3. démo, + bonus (5 minutes)
make it fail live.
UML files as source of truth
intégration CI/CD
instructions aux agents

---

Version conférence 40 minutes , on parlerais en plus de :
- pourquoi (imho) les agents ne sont pas bons en architecture
- pourquoi on découpe ?
- nx / turborepo / outillages de monorepo
- domain driven design 
- ui / design system / qu'est ce qu'une "feature"
- autre solution que ts-arch (@nx/enforce-module-boundaries eslint rules)

---

## Sources :
- https://github.com/ts-arch/ts-arch
- https://github.com/softarc-consulting/sheriff
- https://www.angulararchitects.io/en/blog/reliable-angular-architectures-with-ai-assisted-coding/
- https://www.angulararchitects.io/en/blog/architecture-beyond-layers-tsarch-for-ai-agents/ 

