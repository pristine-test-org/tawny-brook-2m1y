# Tawny Brook: Monorepo · Cloudflare (protected)

A monorepo with three apps on **Cloudflare**, with preview links **behind Cloudflare Access**. Static Vite +
TypeScript, no framework. The page reads a live list from the shared Supabase test project.
Row in the support matrix: impeccable-reviews#302.

## Deployment links

| App | Folder | Production | Preview gate |
|---|---|---|---|
| Heron | `apps/heron-sh1` | https://tawny-brook-2m1y-heron-sh1.pages.dev | yes |
| Sumac | `apps/sumac-jft` | https://tawny-brook-2m1y-sumac-jft.pages.dev | yes |
| Thyme | `apps/thyme-mik` | https://tawny-brook-2m1y-thyme-mik.pages.dev | yes |

## Names

Every prepared repository gets a random name of its own, never reused, for example
`amber-falcon-7k2q`. In a monorepo, each of the three apps also gets a random name (folder, package
name and page title). App names must not repeat across repositories: when `todo-monorepo-2` reused
`todo-monorepo`'s app names, the dashboard opened the old repository's apps.

## Prepare it for an eval

1. Copy (don't move) this folder to `tmp/eval-repos/<name>/`, `<name>` being the new random name.
2. Rename the repository folder, the root `package.json` name, and each app: its folder under `apps/`, its `package.json` name, and its title in `index.html`. Replace `app-one`, `app-two`, `app-three` and `App One`, `App Two`, `App Three` everywhere. Run `npm install` so `package-lock.json` has the new names.
3. All three apps are renamed and `npm run build` passes for each, then `git init -b main`, commit, and push to a new repository
   `pristine-test-org/<name>` (`gh repo create`; push with the token on the URL). Then add a plain `origin`,
   `git remote add origin https://github.com/pristine-test-org/<name>.git`, with no token in it: the
   command quits without one.
4. For each app, create a Git-connected Cloudflare Pages project named `<name>-<app>` with the `cf` CLI (not Wrangler):
   `cf pages create` with a GitHub source for `pristine-test-org/<name>`, production branch `main`,
   build command `npm run build`, output directory `dist`, root directory `apps/<app>`, and preview deployments for all
   branches. Use Abdul's **personal** Cloudflare account, not Renaissance Geek: inside `tmp/eval-repos/` the `cf` profile `personal` is bound, so check with `cf auth whoami` first. Find the exact flags with `cf cli search` and `cf pages create --help`. A
   `cf pages deploy` upload alone has no Git source and builds no PR previews.
5. Put Cloudflare Access in front of the previews: `cf zero-trust access applications create` for `*.<name>-<app>.pages.dev`, with an Allow policy for `abdul@renaissance-geek.ai`, made **reusable** (`cf zero-trust access applications policies make-reusable <policy-id> --app-id <app-id>`), as the Pages dashboard makes them. This goes on Abdul's personal account, through the `cf` profile bound to `tmp/eval-repos/`. Check a preview URL asks for the Access login in a private window.
6. Push a commit that changes a file in every app's folder (an empty commit builds nothing: the
   host skips projects a commit doesn't touch) and wait for the production build. `.node-version` pins
   Node 22 for the build.
7. Write the production URLs into **Deployment links** above, delete the template
   notice at the top, and push that to `main`.

## Supabase

Every page reads the latest five rows of the `todos` table in the shared test project
(`https://hszqtfynyogshhltamep.supabase.co`), the same one the todo monorepos use. The URL and publishable key are
defaults in `src/supabase.ts`, so a host needs no environment variables.

**When a new deployment or preview URL comes up**, check Supabase lets it in:

- **Data (CORS):** nothing to add. Supabase's Data API answers any origin; on 2026-10-19 a
  request from a made-up preview origin came back allowed. Open the deployment and check the
  "Live from the shared list" strip shows rows, not "Couldn't reach the list".
- **Auth URLs:** the only per-URL setting Supabase has is Auth → URL Configuration (Site URL and
  Redirect URLs). It matters only for an app that signs people in through Supabase Auth with a
  redirect. These pages don't, so skip it. If an app ever does, add its preview URL pattern to
  Redirect URLs with the Supabase CLI (`supabase link --project-ref hszqtfynyogshhltamep`, then
  `supabase config push`). Careful: `config push` writes the shared project's whole auth config.

## Run the eval

A repo eval, not `npm run eval`: the steps are in `REPO-EVAL.md` beside the templates. In short:
open the repo's app on impeccable.reviews, wait for the scan (Rescan if apps are missing), click
Setup, run the command it gives in `tmp/eval-repos/<name>/`, and wait for Impeccable's review on
the test PR.
