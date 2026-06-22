# Ontoligent

> Governed, ontology-driven knowledge & agent platform — public SDK, API spec, and ontology packs.

This is the **public** face of Ontoligent. It gives developers everything they
need to build on the platform — a typed client SDK, the OpenAPI contract, ready
-to-install ontology packs, connector descriptors, and quickstart guides.

---

## What's in this repo (public)

```
clients/python/         # Typed Python SDK (ckos_client.py) + usage README
docs/                   # Developer & ontology docs (incl. ontology-packs.md)
build/orb.schema.json   # ORB pack JSON schema
build/schema.yml        # OpenAPI 3 contract for the public API
examples/               # Example ORB packs, connector configs, quickstarts
LICENSE
```

Proprietary components (control-plane orchestration, the LangGraph runtime,
graph supervisor, governance ledgers, signer keys, and deployment secrets) live
in a **separate private repository** and are not published here.

---

## Quickstart

### 1. Install the SDK

```bash
pip install ontoligent-client      # or: pip install -e clients/python
```

### 2. Authenticate

```python
from ckos_client import Client

client = Client(
    base_url="https://api.ontoligent.com/api/v1",
    token="<your-api-token>",
    workspace="<your-workspace-id>",
)
```

### 3. Seed an ontology and ingest

```python
# Seed a bundled standard (FIBO, ACORD, FHIR, schema.org, SKOS, PROV-O...)
version = client.ontology.seed_standard("fibo-core")

# Ingest a document into the governed pipeline
client.ingestion.ingest(source="s3://bucket/contract.pdf")

# Ask a grounded, cited question
answer = client.retrieval.ask("What obligations does clause 7 impose?")
print(answer.text, answer.citations)
```

See [clients/python/README.md](clients/python/README.md) for the full SDK
reference.

---

## API contract

The public REST API is described by an OpenAPI 3 document:

- Machine-readable spec: [build/schema.yml](build/schema.yml)
- Interactive docs (hosted): `https://api.ontoligent.com/api/schema/swagger-ui/`

Generate clients in any language with `openapi-generator` against
[build/schema.yml](build/schema.yml).

---

## Ontology packs & the marketplace

Ontoligent ships curated, versioned ontology packs you can seed in one call.
The full catalogue — entities, relationships, competency questions, and
licensing — is documented in [docs/ontology-packs.md](docs/ontology-packs.md).

**Bundled (seedable directly):**
`org-memory` · `fibo-core` · `acord-core` · `fhir-core` · `schema.org-core`
· `skos-core` · `prov-o-core`

**External / reference (license-gated):**
SNOMED CT · ISO 20022 · GS1 · BioPortal vocabularies · LOV

### Bring your own ontology (the unblocker)

Need a standard we don't bundle — MITRE ATT&CK, a national health code set, or
your own private ontology? Anything exportable as **OWL / RDF-XML, Turtle,
JSON-LD, Notation3, or N-Triples** can be imported directly:

```python
draft = client.ontology.create_draft(semver="0.1.0-import")
summary = client.ontology.import_file(draft.id, "snomed-subset.ttl", format="turtle")
client.ontology.publish(draft.id)
```

In the web app this lives under **Ontology → Standards → "Bring your own
ontology"**. License-gated sources (e.g. SNOMED CT) just need converting to one
of the supported serializations first. If you can't produce a file, request a
pack and our team will help onboard it.

---

## Documentation

| Guide | Path |
| --- | --- |
| Ontology packs catalogue | [docs/ontology-packs.md](docs/ontology-packs.md) |
| ORB how-to | [docs/orb-howto.md](docs/orb-howto.md) |
| Tenant onboarding | [docs/tenant-onboarding-example.md](docs/tenant-onboarding-example.md) |
| Data sovereignty | [docs/data-sovereignty.md](docs/data-sovereignty.md) |
| Deployment | [docs/appliku-deployment.md](docs/appliku-deployment.md) |

---

## Support & contributing

- Issues and feature requests: open a GitHub issue.
- Ontology pack requests: `support@ontoligent.com`.
- Security disclosures: please email privately rather than filing a public
  issue.

---

## License

See [LICENSE](LICENSE). The SDK, API spec, docs, and example packs are released
under the repository's open-source license; the hosted platform and its
proprietary internals are not.
