
# ORB — Ontology Runtime Bundle Quick Start

This page explains how operators and customers package, sign, install and deploy an
**Ontology Runtime Bundle (ORB)** — the config-only artifact that turns an
Ontoligent ontology + shapes + fixtures into a deployable tenant on any Kubernetes
cluster, Docker host, or Appliku app.

Backed by the `core.orb` library and the `ckos orb` CLI. Everything described here
runs without booting Django (except `install` / `export`, which touch the database).

---

## 1. Anatomy of an ORB descriptor

An ORB starts as a single `orb.yaml` file in a directory:

```yaml
apiVersion: orb.ckos.io/v1
kind: OntologyRuntimeBundle
metadata:
  slug: ontoligent-finance
  semver: 1.0.0
  title: Finance + Insurance starter
  vendor: Ontoligent
  homepage: https://ontolligent.io
ontologies:
  - name: finance
    semver: 1.0.0
    primary: true
    file: ontologies/finance.ttl
shapes:
  - name: finance-shapes
    file: shapes/finance.shacl.ttl
    enforcement: warn       # warn | quarantine | block
fixtures: []
evals: []
semantic:
  enabled: true
  store: oxigraph
  dataset: finance
deployment:
  helm:
    enabled: true
    values:
      replicas: 2
  compose:
    enabled: true
  appliku:
    enabled: true
```

`orb.yaml` is validated against `core.orb.schema.OrbDescriptor` (pydantic v2,
`extra='forbid'` so typos fail fast). Generate the JSON-Schema for your IDE with:

```powershell
python -m ckos.orb schema --out build/orb.schema.json
```

---

## 2. The lifecycle, end-to-end

```
 author orb.yaml          ckos orb validate          ckos orb package          ckos orb verify          ckos orb install
 (+ ontologies/,    ───▶  ckos orb inspect    ───▶   (signs with Ed25519) ───▶  (recipient trust)  ───▶  (per-workspace)
  shapes/, evals/)        ckos orb render
```

Every step has a stable exit code so you can script it in CI.

| Command | What it does | Exit codes |
|---|---|---|
| `ckos orb validate <path> [--check-files]` | Schema + cross-field rules; `--check-files` also verifies referenced artifacts exist | 0 ok · 3 validation · 4 missing-files |
| `ckos orb inspect <path> [--json]` | Pretty/JSON tree of the descriptor | 0 / 3 |
| `ckos orb schema [--out file]` | Emit the JSON-Schema for tooling | 0 |
| `ckos orb render <path> [--target helm\|compose\|appliku\|all] [--out DIR]` | Emit deployment manifests (no install) | 0 / 2 / 3 |
| `ckos orb package <dir> [--out DIR] [--key HEX \| --key-file FILE]` | Build a `<slug>-<semver>.orb` tarball + signature | 0 / 2 / 3 / 4 |
| `ckos orb verify <bundle> [--key HEX \| --key-file FILE]` | Verify the signature + recompute the manifest | 0 / 5 |
| `ckos orb install <bundle> --workspace <slug> [--key-file FILE] [--publish] [--allow-unsigned]` | Idempotent DB install (boots Django) | 0 / 2 / 5 |
| `ckos orb export <out_dir> --workspace <slug> [--no-shapes] [--overwrite]` | Materialize the active OntologyVersion + ShapeSets as an ORB directory | 0 / 2 |
| `ckos orb keygen [--out DIR]` | Fresh Ed25519 keypair (publishing side) | 0 |

`--allow-unsigned` is **dev-only**. Production installs MUST be signed and the
public key MUST be pinned in `settings.PACKS["TRUSTED_KEYS"]`.

---

## 3. Quick recipes

### Author → publish a bundle

```powershell
# 1. Author
mkdir my-orb ; cd my-orb
# (write orb.yaml + ontologies/ + shapes/)

# 2. Validate (no Django, no DB)
python -m ckos.orb validate .\orb.yaml --check-files

# 3. Generate a signing keypair (one time)
python -m ckos.orb keygen --out .\keys

# 4. Pack and sign
python -m ckos.orb package . --out dist --key-file .\keys\orb_signing.key

# Resulting files:
#   dist\my-orb-1.0.0.orb           (tar.gz: orb.yaml + MANIFEST.json + payload)
#   dist\my-orb-1.0.0.orb.sig       (raw 64-byte Ed25519 signature)
#   dist\my-orb-1.0.0.orb.pubkey    (hex public key, side-by-side trust hint)
```

### Recipient → trust + install

```powershell
# 1. Verify (refuses tampered manifest, wrong key, or missing signature)
python -m ckos.orb verify .\my-orb-1.0.0.orb --key-file .\trusted.pub

# 2. Install into a tenant (creates OntologyVersion + ShapeSets idempotently)
python -m ckos.orb install .\my-orb-1.0.0.orb --workspace acme --key-file .\trusted.pub --publish
```

The install runs inside one `transaction.atomic()` and writes a
`governance.AuditEvent(action="orb.installed")` row. Reinstalling the same
`(workspace, semver)` is a no-op for ontologies and a no-op per `(workspace, name)`
for shape sets.

### Export an existing tenant as an ORB

```powershell
python -m ckos.orb export .\out --workspace acme --bundle-slug acme-snapshot --bundle-version 2026.05.23 --overwrite
python -m ckos.orb package .\out --out dist --key-file .\keys\orb_signing.key
```

### Render deployment manifests without installing

```powershell
python -m ckos.orb render .\orb.yaml --target helm    --out .\out-helm
python -m ckos.orb render .\orb.yaml --target compose --out .\out-compose
python -m ckos.orb render .\orb.yaml --target appliku --out .\out-appliku
```

Every target sets the same `ORB_*` env contract on the deployed app:

| Env var | Source |
|---|---|
| `ORB_SLUG`, `ORB_VERSION`, `ORB_API_VERSION` | `metadata` |
| `ORB_PRIMARY_ONTOLOGY`, `ORB_PRIMARY_ONTOLOGY_FILE` | `ontologies[primary=true]` |
| `SEMANTIC_ENABLED`, `SEMANTIC_STORE_DATASET`, `SEMANTIC_BASE_IRI` | `semantic` |
| `ORB_FIXTURE_FILES`, `ORB_SHAPE_FILES`, `ORB_EVAL_FILES` | comma-joined paths |

So the host app boots with consistent discovery regardless of the target.

---

## 4. Trust model

* Signatures are **Ed25519** over `sha256(bundle.orb)`. Same envelope as
  `apps.packs.signer`, so ORBs ride the existing Domain Pack trust chain.
* The recipient pins trusted public keys in `settings.PACKS["TRUSTED_KEYS"]`. The
  side-by-side `.pubkey` file is a hint — never trusted automatically.
* Manifest tampering (any byte change in `orb.yaml` or any referenced file) makes
  `verify_manifest` fail before signature checks even run.

---

## 5. UI integration notes (for the studio frontend)

The studio surfaces three ORB workflows:

1. **Settings → Bundles → Install** — file uploader posts the bundle to
   `POST /api/v1/orb/install/` (planned). Until then, operators run
   `ckos orb install` on the host.
2. **Catalog → Export as ORB** — calls `POST /api/v1/orb/export/` with the
   workspace slug and optional semver. The response is a streamed zip of the
   rendered ORB directory; the user signs locally.
3. **Docs → How-to: ORB** — this page (served as JSON from
   `GET /api/v1/docs/orb-howto/`).

Read this same markdown via the JSON endpoint:

```http
GET /api/v1/docs/orb-howto/
Accept: application/json
→ {"slug": "orb-howto", "title": "ORB — Ontology Runtime Bundle Quick Start",
    "markdown": "...", "updated_at": "2026-05-23T..."}
```

Or render the raw markdown:

```http
GET /api/v1/docs/orb-howto/?format=markdown
```

---

## 6. Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| `validate` exits 3 with `extra fields not permitted` | Typo in a key | Match the schema; regenerate it with `ckos orb schema` |
| `package` exits 4 | A `file:` reference does not exist or escapes the bundle root | Move artifacts inside the bundle dir, fix the path |
| `verify` exits 5 | Wrong trusted key, missing `.sig`, or manifest tampered | Re-pull bundle from publisher, or verify with the publisher's pinned key |
| `install` exits 5 with "unsigned" | Bundle has no signature and `--allow-unsigned` not set | Sign the bundle with `ckos orb package --key-file …` |
| `install` succeeds but ontology not visible | `--publish` not used — bundle landed as a draft | Publish from Studio or rerun with `--publish` |

For the full ORB schema + library API, see `core/orb/__init__.py` and the
generated `build/orb.schema.json`.
