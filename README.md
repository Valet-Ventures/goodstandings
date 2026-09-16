# Goodstandings

Static site. The root domain is the landing page, and `/<slug>` is a demo site built
for a prospect before Johnny walks in the door.

Reputation management for aesthetic practices and med spas. Every review on Google,
Yelp, RealSelf and Healthgrades answered, unfair ones fought, new five stars weekly.
HIPAA safe. From $500 a month.

## Layout

```
public/index.html          -> the landing page (Johnny's design, hand-maintained)
public/<slug>/index.html   -> a prospect's demo site (generated, do not hand-edit)
templates/<name>/index.html   one template per vertical, {{TOKENS}} for business info
demos/<slug>.json             the business info for one prospect  <- this is what you edit
scripts/demo.mjs              renders demos/*.json through a template into public/
```

`demos/<slug>.json` is the source of truth for a prospect. `public/<slug>/` is build
output. Improve a template, run `node scripts/demo.mjs build`, and every demo using it
is regenerated. That is the whole point: 40 prospects, one template fix.

## Adding a prospect

```bash
node scripts/demo.mjs init med-spa radiance-aesthetics   # creates demos/radiance-aesthetics.json
# fill in the fields, or ask Claude to from the business's Google listing
node scripts/demo.mjs build radiance-aesthetics        # renders public/radiance-aesthetics/
git add -A && git commit -m "Add radiance-aesthetics demo" && git push
```

Live about 30 seconds after the push.

## Local preview

```bash
npx serve public
```

Not `open index.html`: demo pages link to `/` for the root, which only resolves over http.

## Booking requests (med-spa template)

The med-spa template includes a slot picker. Four fields in the demo JSON drive it:

```
BOOKING_OPEN_DAYS      "2,3,4,5,6"   weekday numbers, 0=Sun. Must match HOURS.
BOOKING_START_HOUR     "9"           first slot starts here
BOOKING_END_HOUR       "18"          last slot ENDS here
BOOKING_SLOT_MINUTES   "60"          slot length
```

It offers the next 21 days, skips closed days, and never offers today.

**It submits a request, not a booking.** The page has no access to the practice's
calendar, so every slot shows as free and the copy under the button says the time
is not held until the practice confirms. Do not reword that into a confirmation,
and do not fake "taken" slots to look busy: a patient can act on either, and it
is the practice that eats the empty chair.

**Never put medical information in the form.** Name, phone and treatment interest
only. Netlify Forms is not covered by a BAA, so anything resembling a medical
history does not belong in it. The placeholder text tells patients this; leave it in.

## Rules

- Slugs are lowercase-with-hyphens and permanent once sent to a prospect.
- Demo pages carry `noindex`, set in `netlify.toml` and in each template's `<head>`.
  A demo must never compete with the prospect's real site in Google.
- **Never invent reviews, ratings or credentials for a real business.** Pull their actual
  Google reviews or leave the block out. A page branded with their name carrying made-up
  testimonials is the fastest way to lose the room.
- Each demo is self-contained: one `index.html`, CSS inline, assets from a CDN. No build
  step and no shared stylesheet to break across 40 pages at once.
- The bar at the top of a demo ("Preview site built for X") is what sends a prospect to
  the root domain. Only remove it when the site moves to the client's own domain.
