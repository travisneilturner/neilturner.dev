# neilturner.dev — Hugo Site

## What's in here

- `content/blog/event-log-agent-economy.md` — your first blog post
- `content/consulting.md` — Work With Me / consulting page
- `content/about.md` — About page
- `hugo.toml` — site configuration
- `themes/PaperMod/` — theme (git submodule)

## To deploy on Netlify

### 1. Push to GitHub

```bash
cd neilturner.dev
git add .
git commit -m "Initial site with first blog post"
git remote add origin git@github.com:YOUR_USERNAME/neilturner.dev.git
git push -u origin master
```

### 2. Connect to Netlify

1. Go to https://app.netlify.com
2. Click "Add new site" → "Import an existing project"
3. Connect your GitHub account and select the `neilturner.dev` repo
4. Build settings:
   - **Build command:** `hugo --minify`
   - **Publish directory:** `public`
   - **Environment variable:** `HUGO_VERSION` = `0.147.6`
5. Click "Deploy site"

### 3. Point your domain

1. In Netlify: Site settings → Domain management → Add custom domain → `neilturner.dev`
2. In Cloudflare: DNS → Add CNAME record:
   - Name: `@` (or `neilturner.dev`)
   - Target: `YOUR_SITE_NAME.netlify.app` (Netlify will tell you this)
   - Proxy status: DNS only (turn OFF the orange cloud)
3. In Netlify: Enable HTTPS (it will provision a Let's Encrypt cert automatically)

Note: Turn off Cloudflare's proxy (orange cloud) for the domain pointing to Netlify. 
Netlify needs to handle SSL directly. Using Cloudflare proxy causes redirect loops.

### 4. Verify

Visit https://neilturner.dev — you should see the site.
Visit https://neilturner.dev/blog/event-log-agent-economy/ — your first post.

## To write a new post

```bash
hugo new content blog/my-new-post.md
```

Edit the file in `content/blog/`, set `draft: false` when ready, commit and push.
Netlify auto-deploys on every push to master.

## To preview locally

```bash
hugo server -D
```

Opens at http://localhost:1313. Live reloads on file changes.

## Theme

Using [PaperMod](https://github.com/adityatelange/hugo-PaperMod) — clean, fast, 
widely used for developer blogs. Supports dark mode, table of contents, share buttons,
reading time, tags, and search (JSON output enabled in config).
