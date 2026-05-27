# katted24-config

Yext **config-as-code** for the Katted24 OÜ site (entity type `location`). Yext pulls this repo
from GitHub and applies the field definitions — no Yext CLI needed.

> **This repo contains ONLY Yext-resource JSON.** No `package.json`, no scripts, no entity content,
> no seed data. Yext scans every `.json` here as a resource; anything without a valid `$schema`
> fails the Apply. Entity content lives in the site/RUNBOOK as inline curl payloads, never here.

## Layout

```
platform-config/c/km/
├── field/                       # 114 custom field definitions (one JSON each)
│   ├── c_heroTitle.json
│   ├── c_solutionBodies.json
│   └── …
└── field-eligibility-group/
    └── location.default.json    # attaches all 114 fields to the `location` entity type
```

## What's modelled

Every **viewable** on-site string is a custom field (`c_*`), all `LOCALE_SPECIFIC` (et / en / ru),
covering the full customer brief: hero, value props, use-cases, the 13-category solutions accordion,
about + values, the **Inspiratsioon** photo gallery, FAQ, inquiry form, team, map, careers, contact,
footer, plus SEO and GDPR/consent text. Repeating items are **parallel string lists** zipped at
render time (structs are unavailable on this tier). See `../katted24-build/CONTENT_MODEL.md` for the
full field table and the generator (`gen_config.py`) that produced these files.

## Apply

1. Push this repo to `main`.
2. Yext → **Account Settings → Resources** → Connect GitHub repo → `flyinghighindustries/katted24-config`, branch `main`.
3. **Repull → Apply.** Watch the log; iterate until clean.
4. Verify: **Knowledge Graph → Configuration → Custom Fields** lists all 114 fields, and
   **Field Eligibility → location.default** attaches them to `location`.

See [`RUNBOOK.md`](RUNBOOK.md) for entity creation + trilingual content load.

> **Gallery caveat:** `c_inspirationGallery` is a list-of-image ("complex photo gallery"). If your
> account tier rejects it on Apply, switch that one field to parallel URL string-lists
> (`c_inspirationImageUrls` + `c_inspirationCaptions`) — everything else is verified-safe types.
