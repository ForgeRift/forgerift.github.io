# forgerift.io — static landing page

Single-file static site for `https://forgerift.io`, served via GitHub Pages.

## Contents

- `index.html` — full landing page, inline CSS, no build step.
- `CNAME` — custom-domain directive for GitHub Pages.

## Deploy

1. Create a repo `forgerift/forgerift.github.io` (org-level Pages site).
2. Push this directory's contents to the repo's `main` branch.
3. In the repo's **Settings → Pages**, source = `main` / root.
4. Configure DNS for `forgerift.io`:
   - `A` records pointing at GitHub Pages IPs (185.199.108.153, 185.199.109.153, 185.199.110.153, 185.199.111.153), or
   - `CNAME` record on `www.forgerift.io` pointing at `forgerift.github.io.`
5. In repo Settings → Pages → Custom domain, enter `forgerift.io` and enable **Enforce HTTPS** once the cert provisions.

The `CNAME` file in this directory is what tells GitHub Pages to serve the custom domain.

## Updates

When pricing, plugin feature list, or security posture changes, edit `index.html` directly. There is no build step, no framework, no npm install. `git commit && git push` publishes.
