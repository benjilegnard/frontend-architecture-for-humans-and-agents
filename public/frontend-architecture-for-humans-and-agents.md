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

---

## Exemples de code


## ESLint : `HttpClient` dans les clients

```js
// eslint.config.js
{
  files: ['**/*.ts'],
  ignores: ['**/*-client.ts'],
  rules: {
    'no-restricted-imports': ['error', {
      paths: [{
        name: '@angular/common/http',
        importNames: ['HttpClient'],
        message: 'HttpClient uniquement dans les fichiers *-client.ts',
      }],
    }],
  },
}
```

Notes:
Règle 3.1 : le data access reste dans les `*-client.ts`.


## ts-arch : les cards restent dumb

```ts [|6|7-9|10]
import { filesOfProject } from 'tsarch';
import { expect, it } from 'vitest';

it('les cards ne dépendent ni des clients ni des stores', async () => {
  const rule = filesOfProject('tsconfig.arch.json')
    .matchingPattern(String.raw`-card\.ts$`)
    .shouldNot()
    .dependOnFiles()
    .matchingPattern(String.raw`-(client|store)\.ts$`);
  expect(await rule.check()).toEqual([]);
});
```

Notes:
Règle 3.1 : le nommage comme intention lisible par une machine. `check()` renvoie la liste des violations, ce qui marche avec n'importe quel runner (le matcher `toPassAsync()` n'existe que pour Jest).


## ts-arch : `ui` ne dépend pas de `state`

```ts
it('ui ne dépend pas de state', async () => {
  const rule = filesOfProject('tsconfig.arch.json')
    .inFolder('ui')
    .shouldNot()
    .dependOnFiles()
    .inFolder('state');
  expect(await rule.check()).toEqual([]);
});
```

Notes:
Règle 3.2 : elle ne détecte pas le props drilling, elle verrouille le pattern qui le rend inutile.


## Sheriff : domaines et types

```ts [|6-8|11|12-15]
// sheriff.config.ts
import { sameTag, SheriffConfig } from '@softarc/sheriff-core';

export const config: SheriffConfig = {
  entryFile: './src/main.ts',
  modules: {
    'src/app/<domain>/<type>': ['domain:<domain>', 'type:<type>'],
  },
  depRules: {
    root: 'type:feature',
    'domain:*': [sameTag, 'domain:shared'],
    'type:feature': ['type:ui', 'type:state', 'type:model'],
    'type:ui': 'type:model',
    'type:state': ['type:client', 'type:model'],
    'type:client': 'type:model',
  },
};
```

Notes:
Règles 3.3 et 3.4 : les placeholders taguent toute l'arbo d'un coup, `sameTag` interdit les liens entre domaines. Les deep imports sont bloqués par défaut : seul l'`index.ts` d'un module est accessible.


## Sheriff : dans l'IDE et en CLI

```js
// eslint.config.js
const sheriff = require('@softarc/eslint-plugin-sheriff');

module.exports = tseslint.config({
  files: ['**/*.ts'],
  extends: [sheriff.configs.all],
});
```

```bash
npx sheriff verify   # vérifie les règles
npx sheriff list     # liste les modules et leurs tags
```

Notes:
Même règles, deux boucles de feedback : à la frappe via ESLint, en CI via la CLI.


## ts-arch : pas de cycles

```ts
it('pas de dépendances circulaires', async () => {
  const rule = filesOfProject('tsconfig.arch.json')
    .inFolder('src/app')
    .should()
    .beFreeOfCycles();
  expect(await rule.check()).toEqual([]);
});
```

Notes:
Règle 3.4 : A importe B qui importe A. Alternative côté ESLint : `import/no-cycle`.


## ts-arch : le setup

```ts
// vitest.arch.config.ts
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    include: ['arch/**/*.spec.ts'],
    environment: 'node',
    testTimeout: 60_000,
  },
});
```

```json
"test:arch": "vitest run --config vitest.arch.config.ts"
```

Notes:
`tsconfig.arch.json` est une copie du tsconfig principal : ts-arch ne résout pas `extends`.


## Le stop hook : Claude Code

```js [|4|6-7]
// scripts/hooks/claude-stop-hook.mjs
import { runChecks } from '../ci-checks.mjs';

const result = await runChecks();
if (result.status !== 'success') {
  process.stderr.write(result.message);
  process.exit(2);
}
```

```json
// .claude/settings.json
{ "hooks": { "Stop": [{ "hooks": [{
  "type": "command",
  "command": "node scripts/hooks/claude-stop-hook.mjs",
  "timeout": 600
}] }] } }
```

Notes:
Exit code 2 : Claude Code bloque l'arrêt et renvoie le stderr à l'agent comme feedback.


## Le stop hook : Cursor

```js
// scripts/hooks/cursor-stop-hook.mjs
const input = await readInput();
if (input.status !== 'aborted') {
  const result = await runChecks();
  if (result.status !== 'success') {
    process.stdout.write(JSON.stringify({ followup_message: result.message }));
    process.exit(0);
  }
}
process.stdout.write('{}');
process.exit(0);
```

Notes:
Même cœur `ci-checks.mjs`, autre convention : toujours exit 0, le `followup_message` devient la prochaine tâche de l'agent. `loop_limit: 3` dans `.cursor/hooks.json` borne les tentatives.


## Sources

- [ts-arch/ts-arch](https://github.com/ts-arch/ts-arch)
- [softarc-consulting/sheriff](https://github.com/softarc-consulting/sheriff)
- [Reliable Angular Architectures with AI-Assisted Coding](https://www.angulararchitects.io/en/blog/reliable-angular-architectures-with-ai-assisted-coding/)
- [Architecture Beyond Layers: tsarch for AI Agents](https://www.angulararchitects.io/en/blog/architecture-beyond-layers-tsarch-for-ai-agents/)
