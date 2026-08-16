# Kevin's Blog

Live at: <https://sjyangkevin.github.io/>

Built with [Hugo](https://gohugo.io/) and the
[PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme, deployed to
GitHub Pages by GitHub Actions.

The repository name is what puts this site at the domain root: GitHub serves
`<user>.github.io` as the account's one *user site*. Any other repository can
still publish its own site at `sjyangkevin.github.io/<repo>/`, so this doesn't
foreclose future ones — but those paths are carved out of this site's URL
space. Avoid creating a content section here that collides with the name of a
repository you might publish (this site already claims `/posts/`, `/tags/`,
`/archives/` and `/search/`).

## Setup

1. Install [Hugo Extended](https://gohugo.io/installation/) locally (used only
   for previewing; CI installs its own copy for deploys).
2. Clone the repo and initialize the theme submodule:

   ```bash
   git clone https://github.com/sjyangkevin/sjyangkevin.github.io.git
   cd sjyangkevin.github.io
   git submodule update --init --recursive
   ```

## Writing a post

```bash
hugo new content posts/my-post-title/index.md
```

That creates a page bundle from `archetypes/default.md`. Write the post, fill
in `summary` and `tags`, then flip `draft: false` when it's ready. Keep images
next to `index.md` inside the same folder and reference them relatively
(`![alt](diagram.png)`) — Hugo bundles them with the post.

`summary` is worth filling in: it's the blurb on list pages *and* the page's
SEO meta description. Left blank, Hugo falls back to the first 30 words of the
post, which reads poorly if the post opens with a code block.

Draft posts are excluded from the production build and never reach the live
site, so it's safe to commit and push work in progress.

## Previewing locally

```bash
hugo server -D
```

Serves at `http://localhost:1313/` including drafts (`-D`), with live reload.
Stop with `Ctrl+C`.

## Publishing

```bash
git add .
git commit -m "add: my post title"
git push
```

Pushing to `main` triggers the GitHub Actions workflow, which builds the site
and deploys it to GitHub Pages. The live site updates within about a minute —
no other steps required.

## Customization

Styling lives in `assets/css/extended/`. PaperMod concatenates every file in
that directory **in filename order**, after its own stylesheets — hence the
numeric prefixes, which make the cascade explicit:

| File | Covers |
| --- | --- |
| `00-tokens.css` | colours, borders, focus ring, type scale — start here |
| `05-surfaces.css` | page and TOC backgrounds |
| `10-links.css` | accent colour for links |
| `15-flat-row.css` | shared row surface for post entries and search results |
| `20-navigation.css` | header menu |
| `30-post-list.css` | post entries on list pages |
| `40-tags.css` | tag pills |
| `50-pagination.css` | prev/next navigation |
| `60-search.css` | search input |

Retuning the palette or spacing should only mean editing `00-tokens.css`.

Two theme templates are forked into `layouts/` because PaperMod offers no hook
for the change. Each carries a header comment naming the upstream file and the
commit it was taken from. **When you update the theme submodule, re-diff them:**

```bash
diff themes/PaperMod/layouts/single.html layouts/single.html
diff themes/PaperMod/layouts/_partials/header.html layouts/_partials/header.html
```

Each should show only the delta described in its header comment. Anything else
is an upstream change you've silently reverted.

## Updating the theme

```bash
git submodule update --remote themes/PaperMod
hugo server -D   # check the site still looks right
git add themes/PaperMod && git commit -m "chore: bump PaperMod"
```

Re-run the two `diff` commands above afterwards.
