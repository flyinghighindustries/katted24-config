# Runbook — apply Katted24 config + load content

Account ID: **4744809** (flyinghighindustries). Entity id: **393880** (created below with a custom
`meta.id`). Domain: **katted24.ee**. Languages: **et** (primary) + **en** + **ru**.

No Yext CLI — Yext pulls this repo from GitHub. Entity content is created via inline `curl` against
the Management API.

---

## Step 1 — Push + connect this repo

```bash
cd ~/katted24-config
git add . && git commit -m "Katted24 Yext config-as-code"
git push -u origin main
```

Yext → **Account Settings → Resources** → Connect GitHub repo → `flyinghighindustries/katted24-config`,
branch `main` → **Save**.

## Step 2 — Repull → Apply

Same screen: **Repull**, then **Apply**. Expect 114 custom fields created and attached to `location`.

- If `c_inspirationGallery` errors (list-of-image not supported on tier): change it to two fields
  `c_inspirationImageUrls` (list<string SIMPLE>) + `c_inspirationCaptions`, re-push, re-Apply, and
  the site falls back to URL-list rendering.

## Step 3 — Set the account primary language to Estonian

Yext → **Account Settings → Languages** → make **Estonian (et)** the **primary** profile language,
add **English (en)** and **Russian (ru)** as alternates.

> The POST below writes into the **primary** locale. If primary isn't `et`, the Estonian text lands
> in the wrong profile. Set this **before** Step 5.

## Step 4 — Short-lived Management API key

Yext → **Account Settings → Developer → API Credentials** → new key, Knowledge Graph read+write,
**≤24h expiry**.

```bash
export YEXT_API_KEY="paste_key_here"   # never commit; rotate after use
```

## Step 5 — Create the entity (primary = Estonian) with id 393880

The entity id is set via `meta.id` in the body; `entityType` is a **query param** (not in the body).
Content payloads are generated to `/tmp/` by the build step (see `katted24-build`).

```bash
curl -X POST \
  "https://api.yextapis.com/v2/accounts/me/entities?api_key=${YEXT_API_KEY}&v=20231201&entityType=location" \
  -H "Content-Type: application/json" \
  -d @/tmp/katted24-et-content.json
```

The `et` payload includes built-in fields (`name`, `address`, `mainPhone`, `emails`,
`facebookPageUrl`) + `"meta": { "id": "393880" }` + every `c_*` field. Confirm the response
`meta.id` is `393880`.

## Step 6 — Populate the English + Russian profiles

Alternate payloads contain **only** `slug` + `c_*` fields (no built-ins).

```bash
curl -X PUT \
  "https://api.yextapis.com/v2/accounts/me/entityprofiles/393880/en?api_key=${YEXT_API_KEY}&v=20231201" \
  -H "Content-Type: application/json" -d @/tmp/katted24-en-content.json

curl -X PUT \
  "https://api.yextapis.com/v2/accounts/me/entityprofiles/393880/ru?api_key=${YEXT_API_KEY}&v=20231201" \
  -H "Content-Type: application/json" -d @/tmp/katted24-ru-content.json
```

Slug per profile: `et` → empty (root `/`), `en` → `en` (`/en`), `ru` → `ru` (`/ru`).

## Step 7 — Images

Upload in Yext UI (**Knowledge Graph → Entities → Katted24 OÜ**):

- `c_heroImage`, `c_aboutPhoto` — 1–2 examples staged in `katted24-build/images/`
- `c_ogImage` (1200×630), `c_favicon` — branded; static across locales
- `c_inspirationGallery` — stock PVC-usage photos (replace the placeholder set)

## Step 8 — Deploy the site

Yext → **Pages → Sites → New Site** → connect `flyinghighindustries/katted24`, branch `main`,
build config `config.yaml`. Env vars:

| Name | Value |
|---|---|
| `YEXT_PUBLIC_LOCATION_ENTITY_ID` | `393880` |
| `YEXT_PUBLIC_LOCATION_LOCALE_CODE` | `et,en,ru` |
| `YEXT_PUBLIC_EVENTS_API_KEY` | (Events-only key, Phase 7) |

## Step 9 — Rotate the key

Delete the Step-4 key in Yext → Developer → API Credentials once content is loaded. **The key shared
in chat during the build session is compromised — rotate it regardless.**

---

## Troubleshooting

| Symptom | Fix |
|---|---|
| `Resource file is missing $schema` | A non-resource JSON crept into the repo. Remove it. |
| `Invalid $schema field` | Schema URI not recognised — paste the rejecting line. |
| `c_inspirationGallery` rejected | Tier lacks list-of-image; fall back to URL string-lists (Step 2 note). |
| POST `EMPTY_PARAMETER 1515` | `entityType` must be the URL query param, not in the body. |
| `entityprofiles/{id}/en` 404 | English not enabled (Step 3). |
| Estonian text in English profile | Account primary wasn't `et` at POST time (Step 3). |
