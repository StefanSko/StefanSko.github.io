# stefansko.com

Personal essay blog. Hugo + [hugo-bearblog](https://github.com/janraasch/hugo-bearblog), deployed to GitHub Pages via Actions on every push to `main`.

## Publish an essay

1. Copy the finished markdown into `content/blog/<slug>.md`
2. Add front matter at the top:

   ```yaml
   ---
   title: "Thing Apart"
   date: 2026-08-01
   ---
   ```

3. Preview locally: `hugo server -D` → http://localhost:1313
4. Commit and push to `main`. Live in ~1 minute.

Drafts: add `draft: true` to the front matter — rendered locally with `-D`, never deployed.

## Custom domain (after registering stefansko.com)

1. Repo → Settings → Pages → Custom domain → `stefansko.com`, save, tick "Enforce HTTPS" once the check passes
2. At the DNS provider:
   - `A` records for the apex (`stefansko.com`) → `185.199.108.153`, `185.199.109.153`, `185.199.110.153`, `185.199.111.153`
   - `CNAME` record `www` → `stefansko.github.io`
3. Change `baseURL` in `hugo.toml` to `https://stefansko.com/` and push
