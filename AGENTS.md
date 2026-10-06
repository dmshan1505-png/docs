# Notes for agents working in this repo

This is `github/docs` — the source of docs.github.com. It is a Next.js app served by a
custom Express server, not a plain `next dev` project.

## Running it (Base44 sandbox)

```bash
docker compose -f docker-compose.base44.yml up -d --build
```

- Everything runs in one `web` service (`docker-compose.base44.yml`). Source is
  bind-mounted at `/app`; dependencies are installed from `package-lock.json` on
  startup, so no image rebuild is needed after editing code.
- The dev entry point is `npm start` -> `nodemon src/frame/server.ts`
  (`NODE_ENV=development`, `ENABLED_LANGUAGES=en`) with Next.js dev-mode compilation.
  nodemon restarts the server when `.ts` files change; Next recompiles pages on request.
- The app listens on **port 4000** inside the container (`PORT`); the compose file maps
  host **3000** -> container 4000, which is the public preview port.
- Node 24 is required (`engines.node: ^24`) — the compose image is `node:24-bookworm-slim`.
- Health endpoint: `GET /healthcheck` (see `src/frame/middleware/healthcheck.ts`).
- The public repo has no `translations/` checkout; `ENABLED_LANGUAGES=en` keeps warmup to
  English so the server boots without it.

## Environment

- No credentials are required to boot or browse the docs.
- `ELASTICSEARCH_URL` unset -> general search is proxied to GitHub's production endpoint
  (`src/search/middleware/general-search-middleware.ts`).
- `CSE_COPILOT_SECRET` is only needed for AI answers; `HYDRO_SECRET`/`HYDRO_ENDPOINT` only
  for sending local analytics events. All optional.
- `next.config.ts` sets `allowedDevOrigins` from `BASE44_PUBLIC_HOST_SUFFIX` so the proxied
  preview origin can load dev assets/HMR. Keep that env var wired in compose.

## Verifying changes

- `curl -s -o /dev/null -w '%{http_code}' http://localhost:3000/` should return 200 with
  real HTML; the first request after a restart can take a while while the content tree warms.
- Check logs with `docker compose -f docker-compose.base44.yml logs -f web`.
- Restart after backend/server edits: `docker compose -f docker-compose.base44.yml restart web`
  (nodemon usually handles it, but a restart is the reliable fallback).
