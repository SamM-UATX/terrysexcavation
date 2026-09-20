# Terry's Excavation — website

Plain HTML/CSS/JS site. No build step, no framework, no npm install. Pages use
clean, directory-based URLs (e.g. `/services/pool-removal/`) instead of `.html`
file names, which is why **local preview needs a tiny local server** — see below.

## Site structure

```
index.html                              Home
services/index.html                     Services overview
services/pool-removal/index.html        Pool removal (cave-in/partial/complete + backfill)
services/grading-excavation/index.html  Grading & excavation
services/retaining-walls/index.html     Retaining walls & hillside stabilization
services/demolition-hauling/index.html  Demolition & debris hauling
gallery/index.html                      Project gallery (with lightbox)
service-area/index.html                 Service area (city list)
about/index.html                        About Terry
faq/index.html                          General FAQ
contact/index.html                      Contact form + info
contact/thanks/index.html               Form success page (Netlify redirects here)
css/styles.css                          All styling — shared by every page
js/main.js                              Mobile menu, accordion, before/after slider,
                                         gallery lightbox, form validation highlighting,
                                         scroll reveal — shared by every page
favicon.svg                             Browser tab icon
sitemap.xml, robots.txt                 SEO plumbing
images/logo-watercolor-portrait.jpg     Header/nav logo (cropped from the Gemini
                                         watercolor concept — text-free)
images/logo-terry-portrait.jpg          Real photo of Terry, used on the About page
images/logo-concepts/                   Other Gemini-generated logo concepts, kept
                                         for reference (not used on the live site)
images/gallery/                         Web-ready photos used across the site
images/gallery/originals/               Full-resolution source photos, not tracked in
                                         git, not used directly on the site — local
                                         backup only
```

Every page links to shared assets with **root-relative paths** (`/css/styles.css`,
`/images/...`, `/services/pool-removal/`) rather than relative `../` paths. This
keeps every page's markup identical regardless of how deep it is nested, but it
means the site must be served from a domain/server root — see Local preview below.

## Local preview

Root-relative paths mean you can't just double-click `index.html` anymore (the
browser would look for `/css/styles.css` on your hard drive, not in the project).
Run a tiny local server from the project folder instead:

```
python3 -m http.server 8037
```

then visit `http://localhost:8037`. (This project also has a `.claude/launch.json`
entry named `terrysbobcat-site` if you're using Claude Code's preview tools.)
Python's server automatically serves each folder's `index.html`, so
`http://localhost:8037/services/pool-removal/` works exactly like it will in
production.

## Adding more real photos

1. Put the full-size photo in `images/gallery/originals/`.
2. Resize/compress it for the web (full phone photos are 3-8MB each, way too big for
   a fast page load). iPhone photos often carry an EXIF rotation tag that `sips`
   doesn't always respect, so use Pillow instead (`pip install Pillow` once, or see
   the venv trick below if your system Python is externally managed):
   ```
   python3 -c "from PIL import Image, ImageOps; ImageOps.exif_transpose(Image.open('images/gallery/originals/YOURPHOTO.jpeg')).convert('RGB').save('images/gallery/short-descriptive-name.jpg', quality=62)"
   ```
   This targets roughly 250-450KB per photo at 1400px wide, which keeps pages fast.
   If `pip install Pillow` refuses because Python is "externally managed":
   ```
   python3 -m venv /tmp/imgvenv && /tmp/imgvenv/bin/pip install Pillow
   /tmp/imgvenv/bin/python3 -c "from PIL import Image, ImageOps; ..."
   ```
3. Add an `<img>` tag following the existing pattern wherever it belongs — e.g. in
   `gallery/index.html`'s `gallery-grid` (search for `gallery-slot`):
   ```html
   <button type="button" class="gallery-slot reveal" data-lightbox-trigger data-caption="Full sentence description.">
     <img src="/images/gallery/short-descriptive-name.jpg" alt="Describe what's in the photo" loading="lazy" width="1400" height="1050">
     <p class="cap">Short caption</p>
   </button>
   ```
   Write a real, specific `alt` description (not just a filename) — it helps both
   accessibility and Google image search.

## Updating contact info

- **Phone** appears on every page (nav, hero, footer, mobile call button, `tel:`
  links, and each page's JSON-LD). Search the whole project for `4086054224` and
  `(408) 605-4224` and replace both the digits-only version and the display version.
- **Email** (`Terrysbobcatservices@gmail.com`) is in the footer of every page and
  in the homepage's `LocalBusiness` JSON-LD. Search for `Terrysbobcatservices@gmail.com`
  if it ever needs to change. **Note:** the business was renamed from "Terry's Bobcat"
  to "Terry's Excavation" on the site itself, but this email address (and nothing
  else about the actual mailbox) was left as-is since it's a real, working inbox —
  update it here once/if a new address is set up.

## Contact form — what's configured vs. what you must do before launch

The `/contact/` form is wired for **Netlify Forms** (matches the Netlify deploy path
below) using a real, JS-free HTML submission — no fake success states:

- `data-netlify="true"`, a hidden `form-name` field, and `netlify-honeypot="bot-field"`
  are already on the `<form>` in `contact/index.html`.
- A real (but visually hidden, `aria-hidden`) honeypot field is included for spam
  protection.
- The optional project-photo upload uses `enctype="multipart/form-data"`, which
  Netlify Forms supports natively.
- Required-field validation is native HTML5 (`required`, `type="email"`, etc.) —
  works with no JavaScript; `js/main.js` only adds a visual red-border highlight on
  top of the browser's own validation messages.
- On success, Netlify redirects to `/contact/thanks/`, a real static confirmation
  page — there is no fake "submitted!" message shown without an actual submission.

**Before launch, you must:**
1. Deploy to Netlify (see below) — Netlify Forms only works once the site is built
   and deployed through Netlify's own system. It will **not** work from
   `python3 -m http.server`, GitHub Pages, or any other host.
2. In the Netlify dashboard, go to **Site configuration → Forms** and turn on an
   email notification (or Slack/Zapier integration) so submissions actually reach
   Terry — otherwise they'll sit unread in the Netlify dashboard.
3. If you'd rather use a different form backend (Formspree, Basin, etc.) instead of
   Netlify Forms, remove the `data-netlify`/`netlify-honeypot` attributes and point
   `action` at that provider's endpoint per their docs.

## SEO basics already in place

- Unique page title, meta description, canonical URL, and Open Graph tags on every page
- `BreadcrumbList` JSON-LD + a visible breadcrumb nav on every internal page
- `LocalBusiness` JSON-LD (home page) and `Service` JSON-LD (each service page) —
  update the `url`/`address` fields once the domain and shop address are finalized
- `FAQPage` JSON-LD on `/faq/`, matching the visible questions exactly
- `sitemap.xml` and `robots.txt` at the project root
- Semantic headings and real city names in text (not just images) for local search

**Domain:** `terrysexcavation.com` is used as the canonical/OG URL, `sitemap.xml`,
and `robots.txt` domain throughout — no find-and-replace needed once it's connected
in Netlify (see Deploying below). **This domain has not been verified as registered/owned**
— confirm that before connecting it. (The site previously ran as "Terry's Bobcat" at
`terrysbobcatpoolremoval.com`; that repo/site was left untouched — this is a separate project.)

## Deploying (GitHub → Netlify)

This repo already has a GitHub remote (`origin`), and the plan is to deploy through
Netlify's Git integration rather than dragging a folder onto the dashboard:

1. Push the latest commit to GitHub (`git push origin main`), if it isn't already there.
2. In Netlify, create a free account at [netlify.com](https://netlify.com), then
   **Add new site → Import an existing project → Deploy with GitHub**, and pick
   this repo.
3. Build settings: leave the build command blank and the publish directory as `/`
   (or `.`) — this is a static site with no build step.
4. Every push to `main` will auto-redeploy from then on, and Netlify Forms (see
   above) starts working as soon as the first deploy finishes.
5. Go to **Domain settings → Add a domain**, enter `terrysexcavation.com`,
   and follow the DNS instructions Netlify gives you (a couple of records added
   wherever the domain is registered). Netlify handles HTTPS automatically once
   DNS points to it.

## Content that still needs to be filled in before launch

These are flagged rather than guessed, per the project brief:

- **Domain** — set throughout the site as `terrysexcavation.com`, but **not confirmed
  as actually registered** (unlike the old `terrysbobcatpoolremoval.com`, which was
  verified). Register/confirm ownership before connecting it in Netlify per the
  Deploying section.
- **Business street address** — the `PostalAddress` in each page's JSON-LD only has
  region/country. Add `streetAddress`/`postalCode` once available (or omit entirely
  if Terry operates without a public-facing shop address).
- **Contractor license / business name note** — CA C-12 license #618640 is shown
  sitewide (footer + About page), confirmed live on the CSLB lookup as of the previous
  "Terry's Bobcat" version of this site. That license is registered to "A-1 HAULING,"
  with no DBA on file for "Terry's Bobcat" — and now the site has been renamed again to
  "Terry's Excavation," which also has no confirmed DBA on file. The site intentionally
  shows the license number alone without pairing it to a business name. Resolving the
  DBA question (if desired) is between Terry and CSLB, not a website change — but it's
  worth resolving given the site now displays a business name that doesn't match either
  the license holder or the (also mismatched) contact email/original domain.
- **More customer reviews** — only one real testimonial exists (David, San Jose,
  carried over from the previous site). The reviews section says "more reviews
  coming soon" rather than inventing others. No review structured data has been
  added — add it only once there are enough genuine, policy-eligible reviews.
- **Social media links** — none exist yet; add them to the footer once accounts are live.
- **Service area** — currently the 10 South Bay cities already verified from the
  previous site. Do not add more cities without Terry confirming he actively serves them.
