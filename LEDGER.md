# Change Ledger — Agentic Personas Debugging Session

**Date:** 2026-07-23  
**Repo:** [fjkiani/interview-built-by-a-clanker](https://github.com/fjkiani/interview-built-by-a-clanker)  
**Branch:** `fix/debugging-assessment` (never commit assessment fixes on `master`)  
**Assessment:** 14 seeded findings (manager-reconciled). Ops/port and missing-feature gaps are out of the 14.

---

## Manager audit reconciliation (2026-07-23)

Corrections applied to this ledger before continuing execution:

1. **F6 / Browse contradiction** — Prior “Browse #1–#6 DONE / active” text was stale. Ground truth is whatever is committed on `fix/debugging-assessment`. Scoreboard below is the only status source.
2. **F8 added** — `["favorites"]` query-key shape collision (detail returns `string[]`, favorites page returns `{ favorites: Persona[] }`). Distinct from F3/F4.
3. **B2 + B3 = P0** — Auth bypass + missing `return` on 401; both required; first in fix order.
4. **B7** — Treated as security (IDOR), in-scope of the 14.
5. **G1/G2/G5** — Missing feature / polish; **out of the 14-count**.
6. **T1 (typecheck)** — `SearchParams` not exported → `pnpm typecheck` TS4023 on `routeTree.gen.ts`. Gate before merge; fix by exporting the type (do not hand-edit generated file).

### Code-review correction (2026-07-23, post-execution)

A follow-up code review found this ledger's canonical 14 was **mis-composed**, and that "14/14 solved" was premature:

- **Q1 was missing from the inventory.** A code-quality finding — duplicated DB queries inlined into route handlers — was never tracked here. This ledger reached a count of 14 by listing F8 and treating F7/T1 as distinct, while dropping the code-quality finding entirely.
- **Q1 was genuinely unfixed on the branch** at commit `9c71037` (the commit that declared all 14 solved). Verified: `db.ts` still had `export const personas / cartItems / favorites`, and `checkout.ts` + `favorites.ts` still imported those raw Maps and duplicated the per-user query logic instead of calling the `db` accessor layer.
- **Now fixed** in commit `5d34c8c` (see execution log). Re-encapsulated the Maps (`export` removed) and routed both handlers through `db.cart.getByUserId()`, `db.favorites.getByUserId()`, and `db.personas.getById()`. Build green; favorites + checkout endpoints return identical results (curl-verified).
- **All other 13 findings were confirmed genuinely fixed** on the branch — some via a valid approach (e.g. F6 fixed by invalidating `["cart-count"]` on every mutation rather than renaming the query key; B4 fixed by adding `DELETE` to the CORS `methods` list rather than dropping the restriction).

Net: true scope is **12 bugs + 1 security + 1 code quality = 14**; the branch now addresses all 14.

### Canonical 14 (+ bonus)

| # | ID | Layer | One-line |
|---|----|-------|----------|
| 1 | B1 | Backend | minPrice `<=` → `>=` |
| 2 | B2 | Backend | Auth bypass when `ENFORCE_AUTH` unset |
| 3 | B3 | Backend | Missing `return` on 401 in `authenticate` |
| 4 | B4 | Backend | CORS omits DELETE |
| 5 | B5 | Backend | Login omits `username` |
| 6 | B6 | Backend | Checkout never clears cart |
| 7 | B7 | Backend | Cart DELETE IDOR (no ownership check) |
| 8 | F1 | Frontend | Personas `queryKey` ignores search |
| 9 | F2 | Frontend | PersonaCard `price * 100` |
| 10 | F3 | Frontend | Favorite toggle inverted |
| 11 | F4 | Frontend | Favorites query no `enabled: !!user` |
| 12 | F5 | Frontend | Logout leaves `auth_token` |
| 13 | F6 | Frontend | Invalidate `cart` not `cart-count` |
| 14 | F8 | Frontend | Favorites queryKey shape collision |
| — | Q1 | Backend | Code quality: raw Map exports + duplicated DB queries |
| — | F7 | Frontend | Qty `−` at 1 (bonus UX) |
| — | T1 | Frontend | Export `SearchParams` (typecheck gate) |

### Manager fix order (P0→P2)

1. B2 + B3  
2. B4  
3. F3 + F4 + F8  
4. B6  
5. B5 + F5  
6. F6 (cart / checkout / detail)  
7. F1 + F2  
8. B1 + B7  
9. F7 (bonus) + T1 (typecheck)

---

## Scoreboard (keep updated)

| Metric | Count | Notes |
|--------|------:|-------|
| **Solved (of 14)** | **14** | 13 fixed by prior execution; Q1 (code quality) fixed in `5d34c8c` |
| **Open (of 14)** | **0** | |
| **Bonus/gates done** | **2 / 2** | F7 + T1 |
| **True scope** | **12 + 1 + 1** | 12 bugs + 1 security + 1 code quality |

> Correction: the earlier "14 = B1–B7, F1–F6, F8" composition omitted the code-quality finding (Q1). Q1 is now tracked and fixed; F8/F7/T1 remain recorded as found.

### Solved (14 + bonus)

| ID | Status |
|----|--------|
| B1–B7 | Solved (prior execution) |
| F1–F6, F8 | Solved (prior execution) |
| Q1 | Solved in `5d34c8c` — re-encapsulated Maps; routes use `db` accessors |
| F7 | Solved (bonus) |
| T1 | Solved (`export interface SearchParams`; typecheck green) |

**Last updated:** 2026-07-23 — independent re-audit added + fixed Q1 (code quality) in `5d34c8c`. All 14 now addressed on `fix/debugging-assessment`.

---

## Status

| Action | Status |
|--------|--------|
| Branch | `fix/debugging-assessment` (pushed) |
| Remote | `origin` → `https://github.com/fjkiani/interview-built-by-a-clanker` |
| Merge gate | `pnpm typecheck` green; live curl checks run locally |

---

## Identified problems (inventory)

### Backend (`apps/api`)

| ID | Location | Problem |
|----|----------|---------|
| B1 | `db.ts` `search()` | `minPrice` uses `<=` (should be `>=`) |
| B2 | `middleware/auth.ts` | Auth skipped unless `ENFORCE_AUTH=true` → protected routes 500 |
| B3 | `middleware/auth.ts` | 401 `send` without `return` |
| B4 | `index.ts` CORS | `methods` omits `DELETE` |
| B5 | `routes/auth.ts` login | User omits `username` |
| B6 | `routes/checkout.ts` | No `clearForUser` after order |
| B7 | `routes/cart.ts` DELETE | No `item.userId === userId` (IDOR) |
| Q1 | `db.ts` + `routes/checkout.ts` + `routes/favorites.ts` | Code quality: `export const` Maps leak storage; handlers re-import raw Maps + duplicate query logic instead of using `db` accessors |

### Frontend (`apps/web`)

| ID | Location | Problem |
|----|----------|---------|
| F1 | `routes/index.tsx` | `queryKey: ["personas"]` ignores filters |
| F2 | `components/PersonaCard.tsx` | `price * 100` |
| F3 | `personas/$personaId.tsx` | Toggle inverted |
| F4 | `personas/$personaId.tsx` | No `enabled: !!user` on favorites query |
| F5 | `lib/auth.tsx` | Logout does not clear `auth_token` |
| F6 | cart / checkout / detail | Invalidates `["cart"]` not `["cart-count"]` |
| F7 | `CartItem.tsx` | Minus not `disabled` at qty 1 (bonus) |
| F8 | detail + `favorites.tsx` | Same `["favorites"]` key, different shapes |
| T1 | `routes/index.tsx` | `SearchParams` not exported → TS4023 |

### Out of the 14

| ID | Why |
|----|-----|
| G1/G2 | No `GET /orders` — missing feature |
| G4/G5 | Dead auth decorate / no `GET /` — polish |
| S1 | Shared schemas clean (aside from API violating via B5) |
| R1–R3 | Ops / runtime |

---

## Execution log (commits on `fix/debugging-assessment`)

| Commit | IDs | Summary |
|--------|-----|---------|
| `466d3fd` | B2 B3 B4 B6 F1 F2 F3 + ledger | P0 auth, CORS DELETE, checkout clear, browse/price/toggle; manager reconcile |
| `3302cf7` | F4 F8 | Favorites `enabled` + key shapes `ids` / `list` |
| `472ed9d` | B5 F5 | Login username; logout clears token |
| `cd0eda2` | F6 | cart-count invalidate on cart/checkout |
| `8e9b34b` | B1 B7 | minPrice `>=`; cart DELETE ownership |
| `35c6a5b` | F7 T1 | Disable qty − at 1; export SearchParams |
| `5d34c8c` | Q1 | Encapsulate in-memory stores behind `db` accessors; completes 14/14 |

---

## How to verify (manager gate)

1. `pnpm typecheck` and `pnpm build` green (T1).  
2. Live: `?minPrice=60` only ≥60 (B1); login returns username (B5); CORS includes DELETE (B4); `GET /cart` + token → 200 not 500 (B2/B3); checkout empties cart (B6); cross-user cart DELETE → 404 (B7).  
3. UI: filters refetch (F1); prices correct (F2); heart toggle (F3); no guest `/favorites` (F4); logout clears token (F5); badge updates (F6); favorites page not empty after detail visit (F8).
4. Code quality (Q1): `grep -n "export const" apps/api/src/db.ts` shows no exported Maps; `checkout.ts`/`favorites.ts` import only `db` (no `cartItemsStore`/`personasStore`/`favoritesStore`); favorites + checkout still return correct data.
