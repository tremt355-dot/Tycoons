# Workspace

## Overview

pnpm workspace monorepo using TypeScript. Contains a City Builder Game (Tropical Tycoon) - an island-clearing and construction game with freemium monetization model.

### Isometric Rendering System
The game uses a pseudo-3D isometric engine built entirely from CSS/SVG:
- `toIso(x, y, elev)` projects grid coords to screen: `left=(x-y)*TW/2`, `top=(x+y)*TH/2 - elev*ELEV_H`
- `computeElevation(x,y,type,mapW,mapH)` assigns Chebyshev-distance height levels: center=4, edges=1, water=0
- Cliff faces: SVG parallelograms rendered per tile for each exposed edge (right-neighbor lower → right cliff; bottom-neighbor lower → left cliff)
- Tile colors vary by type AND elevation (higher = more sunlit = lighter shade)
- Z-ordering: `(x+y)*10 + elevation*2` for ground tiles, `+5000` for props

## Stack

- **Monorepo tool**: pnpm workspaces
- **Node.js version**: 24
- **Package manager**: pnpm
- **TypeScript version**: 5.9
- **API framework**: Express 5
- **Database**: PostgreSQL + Drizzle ORM
- **Validation**: Zod (`zod/v4`), `drizzle-zod`
- **API codegen**: Orval (from OpenAPI spec)
- **Build**: esbuild (CJS bundle)
- **Frontend**: React + Vite + Tailwind CSS + Framer Motion

## Structure

```text
artifacts-monorepo/
├── artifacts/              # Deployable applications
│   ├── api-server/         # Express API server
│   └── city-builder/       # React + Vite game frontend
├── lib/                    # Shared libraries
│   ├── api-spec/           # OpenAPI spec + Orval codegen config
│   ├── api-client-react/   # Generated React Query hooks
│   ├── api-zod/            # Generated Zod schemas from OpenAPI
│   └── db/                 # Drizzle ORM schema + DB connection
├── scripts/                # Utility scripts
├── pnpm-workspace.yaml
├── tsconfig.base.json
├── tsconfig.json
└── package.json
```

## Game: Tropical Tycoon

### Mechanics
- **Terrain Clearing**: Click forest/rock tiles to clear them. Costs energy (2-3), gains resources.
- **Energy System**: Players start with 20/20 energy. Regenerates 1 per 5 min. Free players hit limits quickly.
- **Buildings**: 9 building types from 2x2 Cabaña (free tier) to 16x32 Megacomplejo (premium).
- **Blueprints**: Must purchase blueprints to unlock building types above tier 1.
- **Workers**: Required for construction. Each building needs different amounts.
- **Weekly Income**: Buildings generate coins weekly. Free: tiny amounts; paid: substantial.
- **Expansions**: 6 purchasable land expansions.
- **Shop**: Energy packs, worker hiring, blueprints, land expansions.

### Building Types
| ID | Name | Size | Tier | Weekly Income |
|----|------|------|------|--------------|
| hut | Cabaña | 2x2 | 1 (free) | 5 |
| tent | Tienda | 2x4 | 1 (free) | 8 |
| market | Mercado | 4x4 | 1 (free) | 35 |
| workshop | Taller | 4x4 | 2 (200 coins blueprint) | 45 |
| inn | Posada | 6x8 | 3 (500 coins blueprint) | 150 |
| factory | Fábrica | 6x8 | 3 (800 coins blueprint) | 350 |
| mansion | Mansión | 8x8 | 4 (2000 coins blueprint) | 700 |
| tower | Torre Comercial | 8x16 | 4 (5000 coins blueprint) | 2000 |
| megaplex | Megacomplejo | 16x32 | 5 (15000 coins blueprint) | 10000 |

### API Routes (all under /api/game)
- GET `/state?playerId=xxx` - Get game state
- POST `/init` - Initialize new game
- POST `/clear-tile` - Clear a terrain tile (costs energy, gives resources)
- POST `/build` - Place a building
- POST `/buy-expansion` - Purchase land expansion
- POST `/buy-blueprint` - Unlock building blueprint
- POST `/buy-energy` - Purchase energy pack
- POST `/buy-workers` - Hire workers
- POST `/collect-income` - Collect weekly building income
- POST `/tick` - Advance game time (called every 5s by frontend)
- GET `/building-types` - List all building types
- GET `/energy-packs` - List energy packs
- GET `/blueprint-costs` - List blueprint costs

### Database Schema
- `game_states` table: stores all player game state as JSONB (tiles, buildings, expansions, etc.)

## TypeScript & Composite Projects

Every package extends `tsconfig.base.json` which sets `composite: true`. The root `tsconfig.json` lists all packages as project references.

## Root Scripts

- `pnpm run build` — runs `typecheck` first, then recursively runs `build` in all packages
- `pnpm run typecheck` — runs `tsc --build --emitDeclarationOnly` using project references
