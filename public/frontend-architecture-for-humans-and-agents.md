# Renforçons nos règles d'architecture frontend,<br/>pour les humains comme pour les agents 🏗️👮🤖

*Votre architecture n'existe que si elle est exécutable*


## Introduction


## Qu'est-ce qu'une bonne architecture frontend ?

---

## Exemples mermaid


## Six mois plus tard

```mermaid
flowchart TD
  D[feature D] --> A[feature A]
  D --> B[feature B]
  D --> C[feature C]
  A --> B
  B --> A
  C --> S[shared/]
  A --> S
  B -.->|deep import| C
```

Notes:
Exemple mermaid : la feature D qui utilise A, B et C, et le spaghetti qui s'installe.


## Une feature, cinq rôles

```mermaid
flowchart TD
  feature[booking/feature] --> ui[booking/ui]
  feature --> state[booking/state]
  feature --> model[booking/model]
  ui --> model
  state --> client[booking/client]
  state --> model
  client --> model
```

Notes:
Exemple mermaid : les depRules Sheriff entre types (section 3.3).


## Les features entre elles

```mermaid
flowchart TD
  D[feature D] -->|index.ts| A[feature A]
  D -->|index.ts| B[feature B]
  A --> S[domain:shared]
  B --> S
  A x--x|interdit| B
```

Notes:
Exemple mermaid : API publique par index.ts, dépendances unidirectionnelles, jamais de lien horizontal A ↔ B (section 3.4).


## La boucle de feedback de l'agent

```mermaid
sequenceDiagram
  participant Agent
  participant Hook as Stop hook
  participant Rules as Règles
  Agent->>Hook: fin de tâche
  Hook->>Rules: ci-checks.mjs
  Rules-->>Hook: violation (source -> target)
  Hook-->>Agent: exit code 2 + message
  Agent->>Agent: corrige l'import
  Agent->>Hook: fin de tâche
  Hook->>Rules: ci-checks.mjs
  Rules-->>Hook: OK
```

Notes:
Exemple mermaid : le stop hook qui rattrape l'agent (section 5).
