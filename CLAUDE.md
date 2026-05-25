# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an Angular 16 single-page application — a stat and damage calculator for Ragnarok Online (RO). The live site is at https://turugrura.github.io/tong-calc-ro-host/#/. It uses PrimeNG as the component library, PrimeFlex for layout, and connects to a backend API for user accounts and preset storage.

## Commands

```bash
npm start          # Dev server on 0.0.0.0:4200 with HMR
npm run build      # Production build (output: dist/sakai-ng/)
npm test           # Run tests with Karma/Jasmine
npm run lint       # ESLint with auto-fix
```

To run a single test file, Karma runs all specs by default. Narrow tests by temporarily using `fdescribe`/`fit` (focused) inside the spec file.

Deploy sequence (builds to sibling `tong-calc-ro-host/docs/`):
```bash
npm run predeploy  # ng build with output-path to ../tong-calc-ro-host/docs/
npm run deploy     # copies index.html → 404.html
```

## Architecture

### Core Calculation Engine

The calculator logic lives entirely in `src/app/layout/pages/ro-calculator/` and consists of four plain TypeScript classes (no Angular DI):

- **`Calculator`** (`calculator.ts`) — top-level orchestrator. Call chain: `setMasterItems` → `setHpSpTable` → `setClass` → `setMonster` → `loadItemFromModel` → `prepareAllItemBonus` → compute methods. It builds the full equipment bonus (`EquipmentSummaryModel`) by aggregating item scripts, cards, enchants, refines, and grade bonuses.
- **`DamageCalculator`** (`damage-calculator.ts`) — consumes `Calculator` output; computes per-skill and basic attack damage including elemental multipliers, size penalties, critical hits, and DPS.
- **`HpSpCalculator`** (`hp-sp-calculator.ts`) — computes MaxHP/MaxSP from the HP/SP factor table and equipment bonuses.
- **`BaseStateCalculator`** (`base-state-calculator.ts`) — tracks allocated/remaining status points and trait points across base levels 1–275.

### Job/Class System

`src/app/jobs/` contains one file per playable class. Every class extends the abstract `CharacterBase` (`_character-base.abstract.ts`) and must implement:

- `JobBonusTable` — `Record<jobLevel, [str, agi, vit, int, dex, luk]>` cumulative deltas.
- `TraitBonusTable` — same shape for 4th-job trait stats (pow, sta, wis, spl, con, crt).
- `_atkSkillList: AtkSkillModel[]` — each entry carries `formula`, optional `customFormula`, ASPD timings (`acd`, `fct`, `vct`, `cd`), element, hit count, and flags like `isMatk`, `isMelee`, `canCri`.
- `_activeSkillList` / `_passiveSkillList` — togglable UI skill panels that feed bonuses into the calculator.

`_class-list.ts` instantiates all classes and exposes `getClassDropdownList()` — the single source of truth for which classes are selectable in the UI.

### Game Data Flow

Static game data (items, monsters, HP/SP tables) is loaded once from JSON assets at runtime by `RoService` (`src/app/api-services/ro.service.ts`) using `shareReplay(1)`:

- `assets/demo/data/item.json` — all equippable items with their parsed `script` bonus maps.
- `assets/demo/data/monster.json` — monster stats.
- `assets/demo/data/hp_sp_table.json` — per-class base HP/SP progression.

In development mode, `RoService` logs any item `script` bonus keys or class names that are not recognized by the validator (`validClassNameSet`, `createRawTotalBonus` key set) to catch data mismatches.

### Constants

`src/app/constants/` centralises game rule tables:
- `enchant_item/` — per-item enchant option lists.
- `share-active-skills/` and `share-passive-skills/` — skill definitions shared across multiple classes.
- Other files: element/race/size mappers, weapon-type/ammo compatibility, food stat lists, item option tables.

### API Layer

`BaseAPIService` (`src/app/api-services/base-api.service.ts`) is an abstract base that provides authenticated `get`, `post`, and `delete` wrappers with automatic JWT refresh (checks expiry before every request via `@auth0/angular-jwt`). Tokens are stored in `localStorage` as `accessToken` and `refreshToken`.

Concrete services:
- `AuthService` — login/logout, profile fetching; exposes `loggedInEvent$` and `profileEventObs$` as `ReplaySubject`-backed observables.
- `PresetService` — CRUD for saved character builds; supports bulk create, sharing/publishing, and tagging.

### Routing & Pages

`HashLocationStrategy` is used (URLs contain `#`). Lazy-loaded modules:

| Path | Module |
|---|---|
| `/` | `RoCalculatorModule` — main calculator UI |
| `/shared-presets` | `SharedPresetModule` — community-shared builds |
| `/preset-summary` | `PresetSummaryModule` — comparison view |
| `/login` | `AuthModule` |

### Environments

`src/environments/environment.ts` (dev) points `roBackendUrl` to `http://localhost:3001`. Production swaps this to the Cloud Run URL via `fileReplacements` in `angular.json`.

## Key Conventions

- **`any` is intentional** — ESLint rule `@typescript-eslint/no-explicit-any` is disabled. Game data scripts are dynamically shaped, so `any` is used throughout the bonus-processing pipeline.
- **Unused imports are errors** — `unused-imports/no-unused-imports: error`. Unused variables are warnings; prefix with `_` to suppress.
- **Components use `app-` prefix** in kebab-case; directives use `app` prefix in camelCase.
- **Styles** are SCSS; each component has a paired `.css` file (Angular uses the CSS extension despite SCSS in angular.json for inline styles).
- **`EquipmentSummaryModel`** is the canonical stat accumulator — any new bonus type must be added to both the model and `createRawTotalBonus()` to be recognized.
- When adding a new job class, create a file in `src/app/jobs/`, export it from `jobs/index.ts`, and register an instance in `_class-list.ts`.
