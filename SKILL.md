---
name: nuxt-init
description: Use when scaffolding a new Nuxt 4 project with standard config files (prettier, eslint, gitignore, husky, vitest, tsconfig, sops) and bun scripts.
---

# Nuxt Init

Scaffolds a new Nuxt 4 project with the standard config files, dev dependencies, lint-staged hooks, and bun scripts used across all Puretec Nuxt apps.

## When to Use

- Starting a brand-new Nuxt 4 project.
- Auditing an existing project to ensure all standard configs are present.
- Onboarding a new repo to the same tooling baseline.

## Workflow

1. Create the config files listed below (skip any that already exist unless the user asks to overwrite).
2. Install the dev dependencies (including `terraform-scaffold`).
3. Add the scripts and `lint-staged` block to `package.json`.
4. Run `bun install` and `bun run prepare` to initialise husky and nuxt types.
5. Run `bunx terraform-scaffold init` to scaffold the Terraform directory structure.

---

## Config Files

### `prettier.config.js`

```js
export default {
    trailingComma: 'es5',
    tabWidth: 4,
    semi: false,
    singleQuote: true,
    bracketSpacing: true,
    bracketSameLine: false,
    arrowParens: 'always',
    vueIndentScriptAndStyle: false,
    singleAttributePerLine: true,
    endOfLine: 'crlf',

    // Pug-specific config
    plugins: ['@prettier/plugin-pug', 'prettier-plugin-terraform-formatter'],
    pugFramework: 'vue',
    pugSortAttributes: 'asc',
    pugWrapAttributesThreshold: 1,
    pugClassNotation: 'attribute', // Ensures `class="..."` inside ()
    pugIdNotation: 'literal',
    pugAttributeSeparator: 'always',

    pugSingleQuote: false,
    pugPrintWidth: 100,
}
```

### `.prettierignore`

```
node_modules
*.log*
.nuxt
.nitro
.cache
.output
.env
.agents
.claude
.githooks
dist
jsons/**/*
public/**/*
ios
android
terraform/lambda/src/
pnpm-lock.yaml
```

### `.gitignore`

```
# Nuxt dev/build outputs
.output
.data
.nuxt
.nitro
.cache
dist

# Node dependencies
node_modules

# Logs
logs
*.log

# Misc
.DS_Store
.fleet
.idea

# Local env files
.env
.env.*
!.env.example

# Local scratch
.tmp/

# Claude Code
.claude/worktrees/

# Terraform local artifacts
**/.terraform/*
*.tfstate
*.tfstate.*
*.tfplan
terraform/envs/*/terraform.tfvars
*.tsbuildinfo
```

### `eslint.config.ts`

```ts
import withNuxt from './.nuxt/eslint.config.mjs'
// @ts-expect-error: module uses export = syntax
import prettierConfig from 'eslint-config-prettier'

export default withNuxt(
    {
        name: 'project/ignores',
        ignores: [
            '.agents/**',
            '**/.agents/**',
            '.claude/**',
            '**/.claude/**',
            '.nuxt/**',
            '**/.nuxt/**',
            '.output/**',
            '**/.output/**',
            'node_modules/**',
            '**/node_modules/**',
            'vendor/**',
            '**/vendor/**',
            'terraform/functions/**',
        ],
    },
    {
        name: 'project/rules',
        languageOptions: {
            parserOptions: {
                templateTokenizer: {
                    pug: 'vue-eslint-parser-template-tokenizer-pug',
                },
            },
        },
        rules: {
            '@typescript-eslint/no-unused-vars': [
                'error',
                {
                    varsIgnorePattern: '^(_|on[A-Z].*|props|emit)$',
                    argsIgnorePattern: '^_',
                },
            ],
            'vue/max-attributes-per-line': 'off',
            'vue/attributes-order': 'off',
            'vue/html-quotes': 'off',
            'vue/multiline-html-element-content-newline': 'off',
            'vue/singleline-html-element-content-newline': 'off',
            'vue/html-self-closing': 'off',
            'vue/component-tags-order': [
                'error',
                { order: ['script', 'template', 'style'] },
            ],
            'func-style': ['error', 'declaration'],
        },
    },
    prettierConfig
)
```

### `vitest.config.ts`

```ts
import { resolve } from 'node:path'
import { defineConfig } from 'vitest/config'

export default defineConfig({
    test: {
        globals: true,
        environment: 'node',
    },
    resolve: {
        conditions: ['vitest'],
        alias: {
            '~~': resolve(__dirname),
            '~': resolve(__dirname, 'app'),
        },
    },
})
```

### `tsconfig.json`

```json
{
    "files": [],
    "references": [
        { "path": "./.nuxt/tsconfig.app.json" },
        { "path": "./.nuxt/tsconfig.server.json" },
        { "path": "./.nuxt/tsconfig.shared.json" },
        { "path": "./.nuxt/tsconfig.node.json" }
    ]
}
```

### `.husky/pre-commit`

```sh
bunx lint-staged
```

### `.husky/pre-push`

```sh
bun run lint
bun run test
```

### `.sops.yaml`

```yaml
# Replace the placeholder recipients below with real age public keys (age1...)
# for your developer team and CI before encrypting secrets.
creation_rules:
    - path_regex: ^(.+[\\/])?secrets[\\/].*\.sops\.json$
      age: >-
          age1REPLACE_WITH_YOUR_AGE_PUBLIC_KEY
    - path_regex: ^(.+[\\/])?secrets[\\/]env-bundle\.sops\.json$
      age: >-
          age1REPLACE_WITH_YOUR_AGE_PUBLIC_KEY
```

---

## `package.json` Additions

### Scripts

```json
{
    "dev": "nuxt dev",
    "build": "nuxt build",
    "generate": "nuxt generate",
    "preview": "nuxt preview",
    "lint": "eslint . && vue-tsc --noEmit",
    "lint:fix": "eslint . --fix",
    "pretty": "prettier --write .",
    "test": "vitest run",
    "test:watch": "vitest",
    "prepare": "husky",
    "postinstall": "nuxt prepare"
}
```

### lint-staged

```json
{
    "lint-staged": {
        "*.{js,ts,vue}": "eslint --fix",
        "*.{js,ts,vue,json,md,css,scss,yml,yaml}": "prettier --write"
    }
}
```

---

## Dev Dependencies

Install with:

```bash
bun add -d eslint @nuxt/eslint eslint-config-prettier prettier @prettier/plugin-pug prettier-plugin-terraform-formatter vue-eslint-parser-template-tokenizer-pug husky lint-staged vue-tsc @types/node tsx tailwindcss terraform-scaffold
```

---

## Terraform Scaffold

[terraform-scaffold](https://github.com/ralphcrisostomo/terraform-scaffold) is a CLI tool that automates AWS infrastructure setup with Terraform, AppSync, and Lambda.

### What It Provides

- Complete Terraform directory layout with environment configs (staging, production)
- GraphQL/AppSync resolver scaffolding (JS or Lambda-backed)
- Standalone Lambda function generation
- Lambda bundling with esbuild
- Terraform execution wrapper and output export scripts

### Setup

After installing as a dev dependency (included in the install command above), initialise the Terraform structure:

```bash
bunx terraform-scaffold init
```

### Key Commands

| Command | Description |
|---------|-------------|
| `bunx terraform-scaffold init` | Create config file and full Terraform directory structure |
| `bunx terraform-scaffold graphql` | Generate a GraphQL resolver with schema, resolver, and Terraform files |
| `bunx terraform-scaffold lambda` | Create a standalone Lambda function |
| `bunx terraform-scaffold build --env=<env>` | Bundle Lambda functions with esbuild |
| `bunx terraform-scaffold tf <env> <action>` | Execute Terraform operations (plan, apply, etc.) |
| `bunx terraform-scaffold tf-output <env>` | Export Terraform outputs to environment files |
| `bunx terraform-scaffold sync-modules` | Copy Terraform modules into your project |

---

## Post-Setup

After creating all files and installing dependencies:

```bash
bun install
bun run prepare              # initialises husky + nuxt types
bunx terraform-scaffold init # scaffolds Terraform directory structure
```
