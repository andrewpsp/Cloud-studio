# Deployment architecture

| Layer | Where it runs | Contents |
| --- | --- | --- |
| Frontend (public) | Vercel (static) | Landing page / demo in `site/`, built with `npm run build:vercel` into `site/dist` (see `vercel.json`) |
| Runtime library | npm | `packages/runtime` (`mesh-avatar`), released with Changesets |
| Backend (local, private) | Developer machine | Vite server plugins in `src/server/` (`/__studio/*` project files, jobs, live relay, MediaPipe assets) and the Python tools in `tools/` |

## Vercel

1. Import the repository; Vercel reads `vercel.json` (Node 22 via `engines`/`.nvmrc`).
2. Set `SITE_BASE=/` (already set by `build:vercel`) and optionally `SITE_ORIGIN=https://<your-domain>`.
3. No server runtime is required: the deployed frontend is fully static.

GitHub Pages (`.github/workflows/pages.yml`) still uses the default `/mesh-avatar-studio/` base.

## Backend abstraction

The editor talks to its backend only through `/__studio/*` HTTP endpoints, implemented today by Vite
plugins that touch the local filesystem (user illustrations in `projects/` must stay local). The
Studio is therefore a local tool and is not deployed to Vercel. To host a backend later, implement
the same endpoints as a separate service (e.g. a Node HTTP server or serverless functions behind its
own repository/package), and proxy `/__studio` to it; keep the Python tooling as worker jobs there.
Frontend code must not import from `src/server`.

## Operations checklist

- CI (`.github/workflows/ci.yml`) runs lint, tests, builds and e2e on every PR.
- Releases of `mesh-avatar` use Changesets (`npm run changeset`).
- Never put user images or secrets into the frontend build; use Vercel environment variables for config.
