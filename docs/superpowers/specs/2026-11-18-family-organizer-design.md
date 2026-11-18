# Family Organizer: Design

Date: 2026-11-18 · Status: draft for approval

## Intent

The repository changes from the Art Gallery App to a **family organizer**: one app with several kinds of lists for a household. It is used by **two parents, each on their own smartphone**, so both must see the same data. The first focus is **cooking**: own recipes, a shopping list, then to-do lists.

The project is also a learning project: the owner wants to practice a **MongoDB backend**. The course rules of Recap Project 6 still apply (see Constraints).

## Goals (first release, in this order)

1. **Recipes:** create, view, edit and delete own recipes with any number of steps (some recipes have 2 steps, some 20).
2. **Shopping list:** one shared list with add, edit, check off and delete, grouped by preferred store.
3. **To-do lists:** several named lists with items to add, check off, edit and delete.

## Non-goals (not in the first release)

- Accounts, login, roles. A shared family code protects the data instead (see Security).
- Photos (per step or per recipe), servings scaling, cooking mode, "add recipe ingredients to the shopping list".
- Real-time sync, offline mode, push notifications.
- All other areas: daily overview, kindergarten week, calendar, wish lists, weather, finances, food inventory, recipe ranking and suggestions, barcode scanning.

These are candidates for later stories. The data model must not block servings scaling and "ingredients to the shopping list" (see Recipe).

## Anti-goals (a failure even if it works)

- **Data is open to strangers.** The app runs on a public URL from a public repository.
- **Data is lost by accident:** one wrong tap deletes a recipe or a list.
- **Entering data on the phone is tedious:** small buttons, desktop-only layouts, forms that lose input.
- **Secrets in the repository** (database URL, family code).

## Constraints

- **Course rules (Recap Project 6):** the agent writes all code; one issue, one branch, one pull request; plan with `/plan-feature`, test-first with `/tdd-feature`, `/open-pr`, review, green CI and quality gate before merge; no new dependencies without the owner's approval; only what the issue asks for.
- **Setup issues 1 to 3 stay as they are** (tooling and `CLAUDE.md`, CI, SonarQube Cloud and branch protection) and are done first. They only cover the frontend in `src/`. The backend story extends them (see Backend foundation).
- **Stack of the template stays:** Vite, React 19, TypeScript, React Router, SWR, Vitest with React Testing Library (configured in `vite.config.ts`), Node.js 22.22 or newer, deployment on Vercel.
- **Secrets** (`MONGODB_URI`, `FAMILY_CODE`) live in Vercel environment variables (separate for Production and Preview, see Environments) and in a local `.env` that the owner creates. They are never committed and never pasted into a prompt. The agent cannot read `.env` (deny rule).
- **Smartphone first:** every screen is built for about 360 px width and touch first; desktop is secondary.

## Approaches

| | Approach | Trade-off |
|---|---|---|
| **A (chosen)** | Keep the Vite frontend, add an **Express API in TypeScript** with **Mongoose** and MongoDB Atlas. Locally Vite proxies `/api` to Express; on Vercel Express runs as a function. | Keeps setup issues 1 to 3 and the template valid, separates frontend and backend, matches the course session Backend Basics (Express in TypeScript). Costs two processes in development and some deployment setup. |
| B | Switch to Next.js with API routes and Mongoose. | One framework, one dev server, native on Vercel. But the template (React Router, Vitest in `vite.config.ts`) and setup issues 1 to 3 would have to be rewritten. |
| C | Vite with plain Vercel Functions, no Express. | Less code, but local development needs `vercel dev` with a login, and the learning is tied to Vercel. |

Dropped: a separate Express server on another host (two deployments, CORS, free servers fall asleep).

The owner chose **A** to practice MongoDB while keeping everything built in Steps 1 and 2.

## Architecture

```
Smartphone browser
  └─ React app (Vite, React Router, SWR)
       └─ fetch /api/...  + header x-family-code
            └─ Express app (TypeScript)
                 └─ Mongoose ─ MongoDB Atlas
```

- **Folders:** frontend stays in `src/`. Backend lives in `server/` with its own `tsconfig.json` for Node.
- **One Express app** (`server/app.ts`) is used in two ways: a local entry starts it on port 3001; a Vercel function entry exports it. Vite's dev server proxies `/api` to `http://localhost:3001`.
- **Same origin in production:** frontend and API are one Vercel project, so `/api/*` goes to Express and all other paths to the React app (the existing SPA rewrite in `vercel.json` must not catch `/api`). No CORS.
- **Data fetching:** SWR, as in the template. SWR revalidates when the app gets focus again; that is how the second phone sees changes. A visible refresh is enough; no real-time sync.
- **UI language:** German (the users are German; stores like Budni). Code, tests, commits and issues stay in English. *(builder's call, confirm)*
- **Styling:** plain CSS as in the template, mobile first. Touch targets at least 44 × 44 px. Bottom navigation with the three areas Rezepte, Einkauf, To-dos.

## Security: family code

Not a login, but strangers cannot read or change data.

- `FAMILY_CODE` is a **long random code**, for example four random words, at least 16 characters. It is typed once per phone, so length costs little, and guessing it is not practical. The server refuses to start if `FAMILY_CODE` is missing or shorter than 16 characters. No rate limiting.
- The server reads `FAMILY_CODE` from the environment. Every `/api` request except `GET /api/health` needs the header `x-family-code` with this value, compared in constant time. Otherwise: `401`.
- The app asks for the code once per phone, stores it in `localStorage` and sends it with every request. On a `401` it asks again.
- HTTPS comes from Vercel. The code can be changed in Vercel at any time; both phones then enter it again.
- Later, a real login can replace this middleware without touching the routes.

## Environments: real data stays separate

Unmerged code must never run against the family's real data.

| Where | Database (one free Atlas cluster) | Set by |
|---|---|---|
| Vercel **Production** (merges to `main`) | `family-organizer` | owner, Vercel env vars scoped to Production |
| Vercel **Preview** (every pull request) | `family-organizer-dev` | owner, Vercel env vars scoped to Preview |
| Local development | `family-organizer-dev` | owner, local `.env` |

- `MONGODB_URI` and `FAMILY_CODE` are set separately for Production and Preview. Preview gets its own family code, so a preview link shown to coaches never exposes the real code or real data.
- Story 2 documents these owner steps in the README (Atlas: create both databases and a database user; Vercel: env vars per environment). The agent never sees the values.

## Data model

All documents have `_id`, `createdAt` and `updatedAt` (Mongoose timestamps). Strings are trimmed; required strings must not be empty after trimming.

### Recipe

```ts
type Ingredient = {
  amount?: number; // > 0, decimals allowed (0.5)
  unit?: string; // free text: "g", "EL", "Prise"
  name: string; // required
};

type Step = {
  description: string; // required, free text: what to do
  ingredients: Ingredient[]; // 0 or more
};

type Recipe = {
  title: string; // required
  servings: number; // required, integer >= 1
  steps: Step[]; // 1 or more, order = array order, no upper limit
};
```

- Ingredients are recorded **per step**, as structured fields (amount, unit and name separately). This keeps servings scaling and "ingredients to the shopping list" possible later.
- A step may have no ingredients ("Ofen auf 180 °C vorheizen").
- **Total ingredient list** of a recipe is computed, not stored: a pure function over all steps. Ingredients with the same name and the same unit (case-insensitive, trimmed) are added up; an ingredient without an amount appears once by name. It has unit tests.
- Steps are stored embedded in the recipe document.

### Store

```ts
type Store = {
  name: string; // required, unique (case-insensitive)
};
```

The owners create their own stores (Budni, Lidl, Rewe …). Deleting a store does not delete items: their store is removed, and they move to "Egal wo".

### ShoppingItem

```ts
type ShoppingItem = {
  name: string; // required
  amount?: number;
  unit?: string;
  storeId?: string; // preferred store; empty = "Egal wo"
  done: boolean; // default false
};
```

A separate collection, one document per item, so both phones can change different items at the same time without overwriting each other.

### TodoList and TodoItem

```ts
type TodoList = { name: string }; // required
type TodoItem = { listId: string; text: string; done: boolean }; // text required, done default false
```

Items are a separate collection for the same reason as shopping items. Deleting a list deletes its items.

## API

JSON over REST under `/api`. Validation errors: `400` with a message per field. Unknown ids: `404`. Missing or wrong family code: `401`.

| Method and path | Purpose |
|---|---|
| `GET /api/health` | No family code. Checks that the server and the database connection work. |
| `GET, POST /api/recipes` | List (title, servings, number of steps), create |
| `GET, PUT, DELETE /api/recipes/:id` | Read, replace, delete one recipe |
| `GET, POST /api/stores` | List, create |
| `PUT, DELETE /api/stores/:id` | Rename, delete (items move to "Egal wo") |
| `GET, POST /api/shopping-items` | List all, create |
| `PATCH, DELETE /api/shopping-items/:id` | Change fields or `done`, delete |
| `DELETE /api/shopping-items?done=true` | Remove all checked items |
| `GET, POST /api/todo-lists` | List, create |
| `PUT, DELETE /api/todo-lists/:id` | Rename, delete with its items |
| `GET, POST /api/todo-lists/:id/items` | List, create items of one list |
| `PATCH, DELETE /api/todo-items/:id` | Change text or `done`, delete |

## Screens

```
Rezepte                     Rezept                     Einkauf
┌───────────────────┐      ┌───────────────────┐      ┌───────────────────┐
│ Rezepte       [+] │      │ ‹ Linsen-Dal  [✎] │      │ Einkauf [Bei:Alle▾]│
│ Linsen-Dal  · 4 P │      │ für 4 Personen    │      │ LIDL              │
│ Pfannkuchen · 2 P │      │ Zutaten (gesamt)  │      │ ☐ 1 l Hafermilch  │
│ …                 │      │ • 250 g Linsen …  │      │ EGAL WO           │
│                   │      │ 1. Zwiebel …      │      │ ☐ 500 g Nudeln    │
│                   │      │ 2. …              │      │ ☑ Butter  (grau)  │
├───────────────────┤      ├───────────────────┤      ├───────────────────┤
│ Rezepte Einkauf ✓ │      │ Rezepte Einkauf ✓ │      │ [+ Artikel]       │
└───────────────────┘      └───────────────────┘      └───────────────────┘
```

- **Recipe list → recipe detail** (title, servings, total ingredients, numbered steps with their ingredients and description).
- **Recipe form** (create and edit): title, servings, steps. Add, remove and reorder steps (move up and down); inside a step, add and remove ingredient rows. The form keeps the input when saving fails.
- **Shopping list:** grouped by preferred store, "Egal wo" last. Filter "Bei: <store>" shows that store's items plus "Egal wo". Checked items turn grey and move to the end of their group; "Erledigte löschen" removes them. Add and edit with name, amount, unit, store. Manage stores from this screen.
- **To-do lists:** overview of lists → one list with its items: add, check, edit, delete.
- **Deleting** a recipe, a store or a to-do list asks for confirmation. Single shopping and to-do items are deleted without a dialog.

## Testing

- **Frontend:** Vitest and React Testing Library as in the setup issues; the API is mocked in component tests.
- **Backend:** route tests against the Express app (e.g. with `supertest`) and a database that is **never** the real Atlas database (e.g. `mongodb-memory-server` or a mocked model layer). CI has no database secret. Which tool: the builder proposes it in the plan of the backend story; it is a new dependency and needs approval.
- Server tests run in the Node environment, frontend tests in jsdom.
- Pure functions (total ingredients, grouping by store, family code check) have unit tests.

## Proposed story order

Each line becomes one issue, written with `/create-issue`. The art gallery stories 4 to 13 are closed (by the owner, in GitHub).

1. **README and app shell:** README describes the family organizer; bottom navigation with three empty areas; mobile-first base styles.
2. **Backend foundation:** Express app, MongoDB connection, `GET /api/health`, Vite proxy, Vercel function entry. Extends the tooling from issues 1 to 3 to `server/`: TypeScript config, lint, tests in the Node environment, coverage and SonarQube sources, CI.
3. **Family code:** middleware, code prompt in the app, `401` handling. *(Split from story 2 to keep pull requests reviewable; builder's call.)*
4. **Create a recipe** (form with steps and ingredients, `POST`).
5. **Recipe list and detail** with total ingredients.
6. **Edit a recipe.**
7. **Delete a recipe.**
8. **Stores:** create, rename, delete.
9. **Shopping list:** add, edit, check off, delete, grouped by store.
10. **Shopping list:** "Bei: <store>" filter and "Erledigte löschen" (no confirmation: only checked items are removed; builder's call).
11. **To-do lists:** create, rename, delete lists.
12. **To-do items:** add, check, edit, delete.

Stories 4 and 5 may be swapped or merged by the owner when writing the issues.

## Left to the builder

- Exact Vercel setup for Express next to Vite (function entry, `vercel.json` rewrites); checked against the current Vercel documentation in story 2.
- Reusing the Mongoose connection across serverless calls.
- Folder structure inside `src/` and `server/`, component names, CSS structure.
- The backend test tool (see Testing), proposed in the plan of story 2.
- Field length limits (sensible maximums for titles and texts).
