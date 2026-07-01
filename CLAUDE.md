# CLAUDE.md — Sales Demo Menu

> **Role in the live system:** This repo is the **design source** for the Birch & Bramble demo restaurant in Supabase. The script `product-menuadmin/scripts/seed-demo.ts` reads `config.js` and `menuData.js` from here and loads them into the database. This file itself is **never deployed** and is never touched by the live system — it is kept as-is for offline sales pitching. If you change the menu content here, the database is **not** automatically updated; the seed script would need to be re-run (which is destructive — see `product-menuadmin/scripts/README.md` before doing that).



Guidance for Claude when working in this project, and the **playbook for personalising
the menu for a sales lead**.

## What this is

A mobile-first **digital restaurant menu demo** used to sell a digital-menu service to
restaurant owners. It is a self-contained single-page app — open `index.html` in a
browser, no build step, no server required.

- **`index.html`** — the whole app: an inline `<script type="text/babel">` React 18 app
  (React + Babel loaded from CDN). Contains the components, `THEMES`, `LANGS`, and the
  CSS-variable styling. Edit this for layout/theme/UI changes.
- **`config.js`** — restaurant **identity** (name, tagline, monogram, logo, currency) and
  the **Earthy** theme's palette/fonts. Loaded as a classic script; defines `const CONFIG`.
- **`menuData.js`** — categories, dishes, allergen-icon map, and filter config. Defines
  `const ALLERGEN_ICONS`, `CATEGORIES`, `FILTERS`, `DISHES`.
- **`images/allergens/`** — 14 EU allergen icons (`*.png`, dark glyphs on transparent).
- **`images/dishes/`** — dish photos (`<id-ish>.jpg`). Missing photo ⇒ neutral placeholder.
- **`docs/superpowers/`** — the original design spec + implementation plan (background).

### Key facts
- **Two existing themes** live in `THEMES` (in `index.html`): `A` = **Earthy** (warm, card
  style, reads its colours/fonts from `CONFIG`), `B` = **Minimal** (fixed, sparse, serif).
  The bottom pill (`ThemeToggle`) switches them; default theme is set in `MenuApp`.
- **Four languages**: `LANGS` (EN, ES, FR, DE) in `index.html`. Dish `name`/`desc` are
  per-language objects `{ EN, ES, FR, DE }`, resolved by the `dishText(field, t)` helper
  (`EN` is the fallback). The tagline in `config.js` is the same per-language shape.
- **EU-14 allergen codes**: `GL CR EG FI PN SY MK NU CY MD SE SU LP ML`.
- The app is centred to a **720px column** on wide screens (`#appcol`).

### Verifying changes (no test runner)
After editing `index.html`, **compile-check the inline JSX** (catches syntax errors):
```bash
TMP=$(mktemp -d); cd "$TMP"; echo '{"name":"t","private":true}' > package.json
npm install --no-audit --no-fund @babel/standalone@7 >/dev/null 2>&1
node -e 'const fs=require("fs");const h=fs.readFileSync(process.argv[1],"utf8");const m=h.match(/<script type="text\/babel">([\s\S]*?)<\/script>/);require("@babel/standalone").transform(m[1],{presets:["react"]});console.log("JSX_COMPILE_OK");' "<abs path>/index.html"
cd /; rm -rf "$TMP"
```
After editing `menuData.js`, validate it:
```bash
node -e 'const vm=require("vm"),fs=require("fs");const c={out:{}};vm.runInNewContext(fs.readFileSync("menuData.js","utf8")+"\n;out.D=DISHES;",c);const D=c.out.D,ids=new Set(D.map(d=>d.id));for(const d of D){for(const L of["EN","ES","FR","DE"]){if(!d.name[L]||!d.desc[L])console.log("MISSING",d.id,L);}(d.pairsWith||[]).forEach(p=>{if(!ids.has(p))console.log("BAD pair",d.id,p)});}console.log("dishes",D.length);'
```
There is **no automated browser check** — the human verifies visually. Don't claim it
"looks right"; state what you compiled/validated and ask them to refresh `index.html`.

---

# Playbook: personalise the menu for a lead

When the user asks to personalise for a specific restaurant/lead, do the following. The
headline requirement is a **new "Personalised" theme tab that the app defaults to**,
tailored to the client. This is **identity + theme only** — do **not** swap the demo dishes
or photos. Work from whatever brand details the user gives (name, colours, fonts, logo). Ask
for anything essential that's missing (at minimum: name + brand colours).

## 1. Identity — `config.js`
Update only the identity block:
```js
name:     'Client Name',
tagline:  { EN:'…', ES:'…', FR:'…', DE:'…' },   // or a plain string for all languages
monogram: 'C',          // 1–2 letters for the logo circle when no logoUrl
logoUrl:  '',           // optional image URL/path; fills the logo circle
currency: '€',          // or '£', '$', …
```
Leave the `primaryColor / accentColor / bgColor / headingFont / bodyFont / themeALabel`
fields **as-is** — those drive the fixed **Earthy** showcase theme, not the personalised one.

## 2. Add the "Personalised" theme — `index.html`, `THEMES`
Add a `P` entry to the `THEMES` object (alongside `A` and `B`). Put the **client's brand
colours/fonts as literals** here. Pick `minimal:false` for the warm card layout (recommended),
or `minimal:true` for the sparse serif list layout.
```js
  P: {
    key:'P', name:'Personalised', minimal:false,
    swatch:['#PRIMARY', '#ACCENT'],          // the two dots shown in the toggle
    vars:{
      '--primary':'#PRIMARY','--on-primary':'#TEXT_ON_PRIMARY',   // header/tabs/buttons + text on them
      '--accent':'#ACCENT','--accent-ink':'#TEXT_ON_ACCENT',      // logo/badge/price/Filters pill + text on them
      '--bg':'#PAGE_BG','--surface':'#CARD_BG','--surface-2':'#SUBTLE_BG',
      '--text':'#BODY_TEXT','--muted':'#MUTED_TEXT','--line':'rgba(0,0,0,0.09)',
      '--pill-bg':'#PILL_BG','--pill-ink':'#PILL_TEXT',           // inactive chips
      '--card-radius':'18px','--img-radius':'14px','--chip-radius':'7px',
      '--shadow':'0 8px 20px rgba(0,0,0,0.07), 0 1.5px 3px rgba(0,0,0,0.05)',
      '--shadow-bar':'0 6px 18px rgba(0,0,0,0.10)',
      '--font-display':"'Heading Font', system-ui, sans-serif",
      '--font-body':"'Body Font', system-ui, sans-serif",
      '--display-weight':'700','--name-spacing':'-0.01em',
    },
  },
```
**Colour guidance:** `--primary` is the header/tabs/active-buttons; `--on-primary` must read
clearly on it (usually a near-white/cream). `--accent` is the highlight (logo fill, Popular
badge, price, Filters pill); `--accent-ink` is text/icons sitting on the accent. `--bg` is
the page; `--surface` the dish cards. Aim for contrast and brand fidelity. Reuse the radius/
shadow/weight values above unless the brand calls for sharper/rounder.

**Fonts:** if the brand fonts aren't already loaded, add them to the Google-Fonts `<link>`
URL in `<head>` (the font-loader script). Cormorant Garamond, Jost, and the `CONFIG` fonts
are already loaded.

## 3. Show it in the toggle & default to it — `index.html`
In `ThemeToggle`'s return, add `P` (keep all three):
```js
      {opt('P', THEMES.P.name, THEMES.P.swatch)}
      {opt('B', THEMES.B.name, THEMES.B.swatch)}
      {opt('A', THEMES.A.name, THEMES.A.swatch)}
```
In `MenuApp`, make `P` the default (both fallbacks on that line):
```js
  const [themeKey, setThemeKey] = useState(() => { try { return localStorage.getItem('menu-theme') || 'P'; } catch { return 'P'; } });
```
(A returning visitor whose browser stored a prior choice keeps it; a fresh lead opens on
Personalised. If the user wants it to *always* open on Personalised, ignore localStorage in
that initialiser.)

## 4. Verify & hand off
Run the JSX compile check above. Tell the user exactly what you changed, that the
**Personalised tab is now the default**, and to refresh `index.html`. (The demo menu's
dishes and photos are left as-is.)

## Don'ts
- Don't add a build step, framework, or server dependency — it must open by double-click.
- Don't rename/restructure files or touch `images/allergens/` unless asked.
- Don't swap the demo dishes/photos or edit `menuData.js` as part of personalisation.
- Don't claim visual correctness you haven't verified — compile, then ask to refresh.
