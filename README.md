# Good Standings

Static site. `goodstandings.com` is the landing page, `goodstandings.com/<slug>` is a
demo site built for a prospect Johnny is about to walk into.

## Layout

```
public/index.html          -> goodstandings.com          (Johnny's landing page)
public/<slug>/index.html   -> goodstandings.com/<slug>   (generated, do not hand-edit)
templates/<name>/index.html   one template per vertical, {{TOKENS}} for business info
demos/<slug>.json             the business info for one prospect  <- this is what you edit
scripts/demo.mjs              renders demos/*.json through a template into public/
```

`demos/<slug>.json` is the source of truth for a prospect. `public/<slug>/` is build
output. Improve a template, run `node scripts/demo.mjs build`, and every demo that uses
it is regenerated.

## Adding a prospect

```bash
node scripts/demo.mjs init auto-shop joes-garage   # creates demos/joes-garage.json
# fill in the fields (or ask Claude to, from the business's Google listing)
node scripts/demo.mjs build joes-garage            # renders public/joes-garage/
git add -A && git commit -m "Add Joe's Garage demo" && git push
```

Live at `goodstandings.com/joes-garage` about 30 seconds after the push.

## Local preview

```bash
npx serve public
```

Not `open index.html` - the demo pages link to `/` for the root, which only resolves
over http.

## Rules

- Slugs are lowercase-with-hyphens and permanent once sent to a prospect.
- Demo pages carry `noindex` (set in `netlify.toml` and in each template's `<head>`).
  They must never compete with the client's real site in Google.
- Each demo is self-contained: one `index.html`, CSS inline, assets from a CDN.
  No build step, no shared stylesheet to break across 40 pages.
- The dark bar at the top of a demo ("Preview site built for X by Good Standings")
  is the thing that sends a prospect to the root domain. Only remove it when the
  site moves to the client's own domain.
