
# Ontology Packs — entities, relationships & complementary packs

This document describes **every ontology pack** Ontoligent ships, the **entities**
and **relationships** each one gives you, and a set of **complementary packs** you
can install side-by-side to extend a base pack as your needs grow.

Packs are the fastest path to value: instead of starting from a blank canvas you
seed a versioned **DRAFT** ontology from an industry standard, then customize,
publish, diff and roll back like code.

- **Install in Studio:** *Studio → Ontology → "Start from standard"*.
- **Install via API:**
  - `GET /api/v1/ontology/standards/` — list available packs (filter by `industry`/`scope`).
  - `POST /api/v1/ontology/standards/seed/` — seed a draft version from a pack id.
  - `GET /api/v1/ontology/versions/<id>/card/` — one-page Markdown documentation with provenance + change log.
  - `GET /api/v1/ontology/versions/<id>/export/?fmt=jsonld|turtle|owl|cypher|csv|graph-json` — export to any RDF store / graph DB.

Source of truth for these definitions:
[apps/ontology/standards/catalog.py](../apps/ontology/standards/catalog.py).

## Pack types

| Scope | Meaning | Examples |
| --- | --- | --- |
| **bundled** | Curated subset shipped in-repo. Seeds **offline**, no license, no network call. | Org-Memory, FIBO-core, ACORD-core, FHIR-core, schema.org-core, SKOS-core, PROV-O-core |
| **external** | Link-only. We never redistribute license-gated data; import it yourself via *Studio → Ontology → Import OWL*. | SNOMED CT, ISO 20022, GS1, BioPortal, LOV |

Bundled subsets are intentionally small (10–30 concepts) as a **starting point**,
not a ceiling — small enough that a first-time user sees a real model in seconds.
They are not limited by the visualizer: the interactive graph explorer
(*Ontology → Card → Explore*) scales to large ontologies with search, focus, and
lazy neighbourhood drill-down. Grow any pack to production breadth three ways:
**import the full upstream standard** (*Ontology → Standards → Bring your own
ontology*), **add entities/relations on demand** (*Versions → draft → Add*), or
**layer a complementary pack**. Each pack links to the full official source.

## Pack catalog at a glance

| Pack id | Name | Industry | Scope | Entities | Relations | License |
| --- | --- | --- | --- | ---: | ---: | --- |
| `org-memory` | Enterprise Memory Graph — Org-Memory core | cross-domain | bundled | 20 | 26 | MIT |
| `fibo-core` | FIBO — Finance Industry Business Ontology | finance | bundled | 18 | 14 | Open — MIT (EDM Council / OMG) |
| `acord-core` | ACORD — Insurance reference model | insurance | bundled | 17 | 14 | ACORD membership for full schemas |
| `fhir-core` | HL7 FHIR — Clinical resources | healthcare | bundled | 16 | 12 | Open — CC0 (HL7) |
| `schema-org-core` | schema.org — Web semantics | cross-domain | bundled | 11 | 6 | Open — CC-BY-SA-3.0 |
| `skos-core` | SKOS — Simple Knowledge Organization System | cross-domain | bundled | 3 | 5 | Open — W3C |
| `prov-o-core` | PROV-O — W3C Provenance Ontology | governance | bundled | 6 | 6 | Open — W3C |
| `snomed-ct` | SNOMED CT — Clinical terminology | healthcare | external | — | — | License required (UMLS / IHTSDO) |
| `iso-20022` | ISO 20022 — Payment messaging semantics | finance | external | — | — | Open registry |
| `gs1-core` | GS1 — Supply chain / product identity | retail | external | — | — | Open (GS1 terms) |
| `bioportal` | BioPortal — Biomedical ontology repository | healthcare | external | — | — | Per-ontology |
| `lov` | LOV — Linked Open Vocabularies catalog | cross-domain | external | — | — | Per-vocabulary |

---

## `org-memory` — Enterprise Memory Graph (core)

A decision-centric **upper ontology** that turns ingested artefacts (Slack threads,
Confluence ADRs, Salesforce approvals, calendar events) into a queryable memory
graph. It answers the questions that separate a Knowledge OS from document search:
*"Why was this decision made?"*, *"Who approved exception X?"*, *"Who is the expert
on topic Y?"*

- **Industry:** cross-domain · **License:** MIT · **Scope:** bundled
- **Source:** <https://github.com/ontoligent/org-memory>
- **Designed to be the upper ontology you align vertical packs to.** Subclass any
  vertical class (FIBO `Counterparty`, ACORD `Claim`, FHIR `Patient`) from these
  classes and the causal/competency questions still resolve.

### Entities

| Entity | Parent | Description |
| --- | --- | --- |
| Agent | — | Anything that can act — a person or an organization. |
| Person | Agent | A natural person. |
| Organization | Agent | A team, department, company, or external party. |
| Role | — | A position a Person holds in an Organization at a point in time. |
| Topic | — | A subject area used for expertise discovery and topic-modelling. |
| Document | — | Any artefact: contract, ADR, policy doc, slide deck, page. |
| Policy | Document | A normative document binding behaviour. |
| Contract | Document | Legally binding agreement. |
| Event | — | Something that happened at a point in time. |
| Meeting | Event | A synchronous discussion among Agents. |
| Conversation | Event | Async exchange (Slack thread, email chain). |
| Decision | — | A choice made among alternatives, attributable to one or more Agents. |
| Approval | — | A formal sign-off granting permission for an action or exception. |
| Exception | — | A deviation from a Policy that requires explicit Approval. |
| Hypothesis | — | A claim under investigation; supersedable as evidence accumulates. |
| Risk | — | A potential negative outcome, with likelihood and impact. |
| Project | — | A bounded body of work with goals and a timeline. |
| Task | — | A unit of work assigned to an Agent. |
| Metric | — | A measurable signal (revenue, NPS, churn, MTTR). |
| ExpertiseSignal | — | Derived edge weight: how strongly an Agent is associated with a Topic. |

### Relationships

| Relation | Source → Target | Card. | Description |
| --- | --- | --- | --- |
| HAS_ROLE | Person → Role | 1-N | Roles a person holds. |
| ROLE_IN | Role → Organization | N-N | Organization the role is in. |
| REPORTS_TO | Person → Person | N-N | Reporting line. |
| AUTHORED | Agent → Document | 1-N | Document authorship. |
| OWNS | Agent → Project | N-N | Project ownership. |
| ASSIGNED | Task → Agent | N-N | Task assignee. |
| PART_OF | Task → Project | N-N | Task belongs to project. |
| ATTENDED | Agent → Meeting | N-N | Meeting attendance. |
| PARTICIPATED_IN | Agent → Conversation | N-N | Conversation participation. |
| DECIDED_IN | Decision → Event | N-N | The Meeting/Conversation where the Decision was made. |
| DECIDED_BY | Decision → Agent | N-N | Decision-makers. |
| MOTIVATED_BY | Decision → Hypothesis | N-N | Reasons / hypotheses behind the decision. |
| EVIDENCED_BY | Hypothesis → Document | N-N | Documents supporting a hypothesis. |
| SUPERSEDES | Decision → Decision | N-N | A later decision overrides an earlier one. |
| APPROVES | Approval → Exception | 1-N | Approval grants the exception. |
| GRANTED_BY | Approval → Agent | N-N | The approver. |
| VIOLATES | Exception → Policy | N-N | Policy the exception deviates from. |
| GOVERNS | Policy → Project | N-N | Policy applicable to a project. |
| MITIGATES | Decision → Risk | N-N | Decision intended to mitigate a risk. |
| AFFECTS | Risk → Project | N-N | Project the risk applies to. |
| TRIGGERED_BY | Event → Event | N-N | Causal predecessor event. |
| REFERENCES | Document → Document | N-N | Inter-document reference. |
| ABOUT | Document → Topic | N-N | Topic of a document. |
| DISCUSSED_TOPIC | Event → Topic | N-N | Topic of a meeting/conversation. |
| EXPERT_IN | Agent → Topic | N-N | Derived expertise edge. |
| INFLUENCED | Decision → Metric | N-N | Metric the decision affected (or was meant to affect). |

### Competency questions (gates)

Org-Memory ships SPARQL `ASK`/`SELECT` competency questions that gate the ontology
before publish, e.g.:

- Every `Decision` must be attributable to at least one `Agent` (`DECIDED_BY`).
- Every `Approval` must be attributable to an approver (`GRANTED_BY`).
- The model can express *why* a decision was made (`MOTIVATED_BY` → `EVIDENCED_BY`).
- The model can express that a `Decision` overrides an earlier one (`SUPERSEDES`).
- The model can identify experts on a topic (`EXPERT_IN` + `Topic`).
- The model can trace an `Exception` to the `Policy` it `VIOLATES` and the `Agent`
  who `APPROVES` it.

---

## `fibo-core` — Finance Industry Business Ontology (core subset)

Top-level FIBO classes for legal entities, financial instruments, counterparties,
transactions, and regulatory agents.

- **Industry:** finance · **License:** Open — MIT (EDM Council / OMG) · **Scope:** bundled
- **Source:** <https://spec.edmcouncil.org/fibo/>
- **Full ontology:** import the OWL files from the [FIBO GitHub](https://github.com/edmcouncil/fibo)
  via *Studio → Ontology → Import OWL* (subset = 18 classes / 14 relations).

### Entities

| Entity | Parent | Description |
| --- | --- | --- |
| LegalEntity | — | An entity with legal personhood. |
| Organization | LegalEntity | A legally constituted organization. |
| Person | LegalEntity | A natural person. |
| FinancialInstitution | Organization | Bank, insurer, asset manager. |
| RegulatoryAgency | Organization | Issuer of binding rules. |
| Counterparty | LegalEntity | Party to a financial contract. |
| FinancialInstrument | — | Tradable financial asset. |
| Security | FinancialInstrument | Tradable security. |
| Equity | Security | Ownership share. |
| DebtInstrument | Security | Debt obligation (bond, note). |
| Derivative | FinancialInstrument | Contract whose value derives from an underlying. |
| FinancialAccount | — | Account holding instruments or balances. |
| Transaction | — | Movement of value between accounts. |
| Trade | Transaction | Execution of a financial transaction. |
| Payment | Transaction | Settlement of an obligation. |
| Position | — | Holding of an instrument by a party at a point in time. |
| Issuer | LegalEntity | Party that issued an instrument. |
| MarketDataProvider | Organization | Source of quotes / reference data. |

### Relationships

| Relation | Source → Target | Card. | Description |
| --- | --- | --- | --- |
| ISSUES | Issuer → FinancialInstrument | 1-N | Issuer creates an instrument. |
| OWNS | LegalEntity → FinancialAccount | 1-N | Account ownership. |
| HOLDS_POSITION | FinancialAccount → Position | 1-N | Account holds a position. |
| REFERENCES_INSTRUMENT | Position → FinancialInstrument | N-N | Position references an instrument. |
| HAS_COUNTERPARTY | Transaction → Counterparty | N-N | Transaction parties. |
| SETTLES_THROUGH | Transaction → FinancialAccount | N-N | Settlement account. |
| EXECUTES_TRADE | Counterparty → Trade | 1-N | Trade execution. |
| REGULATED_BY | LegalEntity → RegulatoryAgency | N-N | Regulatory oversight. |
| REPORTS_TO | FinancialInstitution → RegulatoryAgency | N-N | Regulatory reporting line. |
| UNDERLYING_OF | FinancialInstrument → Derivative | 1-N | Underlying for a derivative. |
| MARKET_DATA_FROM | FinancialInstrument → MarketDataProvider | N-N | Reference data provenance. |
| EMPLOYS | Organization → Person | 1-N | Employment. |
| SUBSIDIARY_OF | Organization → Organization | N-N | Ownership hierarchy. |
| PAID_TO | Payment → Counterparty | N-N | Payment recipient. |

---

## `acord-core` — Insurance reference model (core subset)

Top-level ACORD concepts for policy administration, claims, underwriting, and
reinsurance ceded business — the FNOL → Claim → Reserve → Reinsurance flow.

- **Industry:** insurance · **License:** ACORD membership for full schemas (subset paraphrased) · **Scope:** bundled
- **Source:** <https://www.acord.org/standards-architecture/information-models>
- **Full model:** available to ACORD members; load via *Studio → Ontology → Import OWL*
  (subset = 17 classes / 14 relations).

### Entities

| Entity | Parent | Description |
| --- | --- | --- |
| Party | — | A legal or natural party. |
| Insurer | Party | Carrier underwriting the policy. |
| Insured | Party | Named insured / policyholder. |
| Claimant | Party | Party making a claim. |
| Adjuster | Party | Loss adjuster handling a claim. |
| Underwriter | Party | Underwriter assessing risk. |
| Reinsurer | Party | Reinsurance counterparty. |
| Broker | Party | Producer / intermediary. |
| Policy | — | Master insurance contract. |
| Coverage | — | A specific peril / limit on a policy. |
| Endorsement | — | Amendment to a policy. |
| Claim | — | Loss event filed against a policy. |
| FNOL | Claim | First Notice of Loss event. |
| Reserve | — | Money set aside to pay a claim. |
| Payment | — | Indemnity / expense payment. |
| ReinsuranceTreaty | — | Cession agreement between insurer and reinsurer. |
| RiskAssessment | — | Underwriting risk rating. |

### Relationships

| Relation | Source → Target | Card. | Description |
| --- | --- | --- | --- |
| ISSUED_BY | Policy → Insurer | N-N | Carrier of the policy. |
| INSURES | Policy → Insured | 1-N | Named insured(s). |
| COVERS | Policy → Coverage | 1-N | Coverages on the policy. |
| AMENDED_BY | Policy → Endorsement | 1-N | Endorsement chain. |
| FILED_AGAINST | Claim → Policy | N-N | Claim → policy. |
| FILED_BY | Claim → Claimant | N-N | Claim filer. |
| ASSIGNED_TO | Claim → Adjuster | N-N | Adjuster assignment. |
| HAS_RESERVE | Claim → Reserve | 1-N | Reserves on a claim. |
| PAID_VIA | Claim → Payment | 1-N | Indemnity payments. |
| UNDERWRITTEN_BY | Policy → Underwriter | N-N | Underwriting attribution. |
| PRODUCED_BY | Policy → Broker | N-N | Producer / broker of record. |
| RISK_RATED_BY | Policy → RiskAssessment | 1-N | Risk rating attached to a policy. |
| CEDED_TO | Policy → ReinsuranceTreaty | N-N | Policy ceded under a treaty. |
| REINSURED_BY | ReinsuranceTreaty → Reinsurer | N-N | Treaty counterparty. |

---

## `fhir-core` — HL7 FHIR clinical resources (core subset)

Top-level FHIR R4 resource types for patients, encounters, observations,
conditions, procedures, and medications.

- **Industry:** healthcare · **License:** Open — CC0 (HL7) · **Scope:** bundled
- **Source:** <https://hl7.org/fhir/>
- **Full spec:** 150+ resources at hl7.org/fhir; import any FHIR StructureDefinition
  (converted to RDF/Turtle) via *Studio → Ontology → Import OWL* (subset = 16 classes / 12 relations).

### Entities

| Entity | Parent | Description |
| --- | --- | --- |
| Resource | — | Base FHIR resource. |
| Patient | Resource | Person receiving care. |
| Practitioner | Resource | Clinician delivering care. |
| Organization | Resource | Healthcare provider organization. |
| Location | Resource | Physical location of care. |
| Encounter | Resource | Episode of care. |
| Observation | Resource | Measurement or finding. |
| Condition | Resource | Clinical condition / diagnosis. |
| Procedure | Resource | Action performed on a patient. |
| Medication | Resource | Drug product. |
| MedicationRequest | Resource | Prescription order. |
| MedicationAdministration | Resource | Drug administered to a patient. |
| AllergyIntolerance | Resource | Patient allergy or intolerance. |
| DiagnosticReport | Resource | Lab / imaging report. |
| CarePlan | Resource | Plan for patient care. |
| Coverage | Resource | Insurance coverage. |

### Relationships

| Relation | Source → Target | Card. | Description |
| --- | --- | --- | --- |
| HAS_ENCOUNTER | Patient → Encounter | 1-N | Patient encounters. |
| PERFORMED_BY | Encounter → Practitioner | N-N | Clinician on encounter. |
| AT_LOCATION | Encounter → Location | N-N | Where care happened. |
| OBSERVES | Encounter → Observation | 1-N | Observations during encounter. |
| DIAGNOSED_WITH | Patient → Condition | 1-N | Patient diagnoses. |
| UNDERWENT | Patient → Procedure | 1-N | Procedures done. |
| PRESCRIBED | Practitioner → MedicationRequest | 1-N | Prescription authorship. |
| REQUESTS_MEDICATION | MedicationRequest → Medication | N-N | Drug requested. |
| ADMINISTERED | MedicationAdministration → Patient | N-N | Admin event. |
| REPORTS | DiagnosticReport → Observation | 1-N | Findings rolled up into a report. |
| HAS_ALLERGY | Patient → AllergyIntolerance | 1-N | Patient allergies. |
| COVERED_BY | Patient → Coverage | 1-N | Insurance coverage. |

---

## `schema-org-core` — schema.org web semantics (core subset)

Top-level schema.org types useful for general entity linking and JSON-LD APIs.

- **Industry:** cross-domain · **License:** Open — CC-BY-SA-3.0 · **Scope:** bundled
- **Source:** <https://schema.org/>

### Entities

| Entity | Parent | Description |
| --- | --- | --- |
| Thing | — | Most generic schema.org type. |
| Person | Thing | A person, alive or deceased. |
| Organization | Thing | An organization. |
| Place | Thing | A physical location. |
| Event | Thing | An event occurring at a place and time. |
| Product | Thing | A product or service. |
| Offer | Thing | An offer to transact. |
| Review | Thing | A review of a product / service. |
| CreativeWork | Thing | An intellectual work. |
| Article | CreativeWork | Article-shaped CreativeWork. |
| WebSite | CreativeWork | A website. |

### Relationships

| Relation | Source → Target | Card. | Description |
| --- | --- | --- | --- |
| AUTHOR | CreativeWork → Person | N-N | Authorship. |
| PUBLISHER | CreativeWork → Organization | N-N | Publisher. |
| LOCATED_AT | Event → Place | N-N | Event location. |
| OFFERS | Organization → Offer | 1-N | Seller offers. |
| OFFERED_PRODUCT | Offer → Product | 1-N | Product behind an offer. |
| REVIEW_OF | Review → Thing | N-N | Subject of a review. |

---

## `skos-core` — Simple Knowledge Organization System

Primitives for thesauri and taxonomies. Use it to add controlled vocabularies and
broader/narrower hierarchies to **any** other pack.

- **Industry:** cross-domain · **License:** Open — W3C · **Scope:** bundled
- **Source:** <https://www.w3.org/TR/skos-reference/>

### Entities

| Entity | Description |
| --- | --- |
| Concept | An idea / unit of thought. |
| ConceptScheme | A grouping of concepts (a thesaurus). |
| Collection | A meaningful collection of concepts. |

### Relationships

| Relation | Source → Target | Card. | Description |
| --- | --- | --- | --- |
| BROADER | Concept → Concept | N-N | Hierarchical broader. |
| NARROWER | Concept → Concept | N-N | Hierarchical narrower. |
| RELATED | Concept → Concept | N-N | Associative relation. |
| IN_SCHEME | Concept → ConceptScheme | N-N | Membership in a scheme. |
| TOP_CONCEPT_OF | Concept → ConceptScheme | N-N | Top of a scheme. |

---

## `prov-o-core` — W3C Provenance Ontology

Primitives for lineage. Use it to make any pack's facts traceable to the agent and
activity that produced them — the backbone of governance and audit.

- **Industry:** governance · **License:** Open — W3C · **Scope:** bundled
- **Source:** <https://www.w3.org/TR/prov-o/>

### Entities

| Entity | Parent | Description |
| --- | --- | --- |
| Agent | — | Something that bears responsibility for an activity. |
| Entity | — | A thing — physical, digital, or conceptual. |
| Activity | — | Something that occurs over a period and acts on entities. |
| SoftwareAgent | Agent | Software acting as an agent. |
| Person | Agent | A human agent. |
| Organization | Agent | An organization acting as an agent. |

### Relationships

| Relation | Source → Target | Card. | Description |
| --- | --- | --- | --- |
| WAS_GENERATED_BY | Entity → Activity | N-N | Entity produced by activity. |
| USED | Activity → Entity | N-N | Activity used an entity. |
| WAS_DERIVED_FROM | Entity → Entity | N-N | Entity derived from another. |
| WAS_ATTRIBUTED_TO | Entity → Agent | N-N | Attribution of an entity to an agent. |
| WAS_ASSOCIATED_WITH | Activity → Agent | N-N | Activity associated with an agent. |
| ACTED_ON_BEHALF_OF | Agent → Agent | N-N | Delegation of responsibility. |

---

## External (link-only) standards

We never redistribute license-gated data. These packs appear in the catalog as
pointers plus integration notes; import them yourself via
*Studio → Ontology → Import OWL*.

| Pack id | Name | Industry | License | How to integrate |
| --- | --- | --- | --- | --- |
| `snomed-ct` | SNOMED CT — Clinical terminology | healthcare | UMLS / IHTSDO license required | Obtain a UMLS license, download RF2, convert to OWL/Turtle (e.g. snomed-owl-toolkit), import OWL. |
| `iso-20022` | ISO 20022 — Payment messaging | finance | Open registry | Download schemas from iso20022.org, transform the message component(s) to OWL, import OWL. |
| `gs1-core` | GS1 — Supply chain / product identity | retail | Open (GS1 terms) | Fetch the GS1 JSON-LD vocabulary (gs1.org/voc), import OWL with `format=json-ld`. |
| `bioportal` | BioPortal — Biomedical ontology repository | healthcare | Per-ontology | Download from bioportal.bioontology.org (ICD-10, RxNorm, GO, ChEBI, …), import OWL. Free API key for programmatic access. |
| `lov` | LOV — Linked Open Vocabularies | cross-domain | Per-vocabulary | Pick a vocabulary from lov.linkeddata.es, import OWL. |

---

## Complementary packs — combine to extend a base pack

Packs are designed to be **layered**. Install a vertical base pack, then add a
complementary pack to cover an adjacent need without re-modelling from scratch.
Alignment is done by subclassing: point your base-pack class at the complementary
pack's class (e.g. ACORD `Party` `rdfs:subClassOf` Org-Memory `Agent`).

| If your base pack is… | Add this complementary pack… | …to unlock |
| --- | --- | --- |
| `fibo-core` (banking / capital markets) | `acord-core` | Bancassurance & insured-lending: one graph spanning instruments **and** policies. |
| `acord-core` (insurance) | `fibo-core` | Investment-side of the balance sheet, reinsurer counterparties as financial entities. |
| `fhir-core` (clinical) | `snomed-ct` (external) | Production-grade clinical coding for `Condition`/`Procedure`/`Observation`. |
| `fhir-core` (clinical) | `fibo-core` + `acord-core` | Payer/claims context — link FHIR `Coverage` to ACORD `Policy` for prior-auth & claims. |
| **any vertical pack** | `prov-o-core` | Lineage on every fact: `WAS_GENERATED_BY` / `WAS_ATTRIBUTED_TO` for audit & regulators. |
| **any vertical pack** | `skos-core` | Controlled vocabularies & broader/narrower taxonomies (synonym collapse, code lists). |
| **any vertical pack** | `org-memory` | Decision/approval/expertise layer — *who decided, why, and who signed off.* |
| `org-memory` (cross-domain) | `prov-o-core` | Turn decision motivation chains into formally provable provenance. |
| `schema-org-core` (web/entity-linking) | `org-memory` | Promote web-extracted entities into the organizational memory graph. |
| `gs1-core` (external, retail) | `schema-org-core` | Pair product identity with web `Product`/`Offer`/`Review` semantics for commerce. |

### Recommended stacks by industry

- **Insurance:** `acord-core` + `fibo-core` + `prov-o-core` + `org-memory`.
- **Banking / KYC:** `fibo-core` + `org-memory` + `prov-o-core` (+ `skos-core` for risk taxonomies).
- **Healthcare / Pharma:** `fhir-core` + `snomed-ct` + `prov-o-core` (+ `skos-core` for code lists).
- **Capital markets:** `fibo-core` + `iso-20022` + `prov-o-core`.
- **Retail / supply chain:** `gs1-core` + `schema-org-core` + `org-memory`.
- **Cross-functional governance:** `org-memory` + `prov-o-core` over any vertical base.

---

## Proposed enhancement packs (roadmap variations)

These are natural next packs that extend the bundled set. Each is a candidate
complementary pack a customer could install based on their need; they reuse the
upper-ontology alignment pattern above.

| Proposed pack | Extends | Adds (entities / relations) | Use case |
| --- | --- | --- | --- |
| `mitre-attack-core` | `prov-o-core` + `org-memory` | Asset, Identity, Alert, Technique, Incident, Runbook · TARGETS, USES_TECHNIQUE, MITIGATED_BY | SecOps threat-hunt & incident enrichment. |
| `legal-clause-core` | `org-memory` | Contract, Clause, Obligation, PlaybookRule · CONTAINS, DEVIATES_FROM, GOVERNS | Contract review & playbook deviation. |
| `orb-med-core` | `fhir-core` + `snomed-ct` | Disease, Gene, Mutation, Pathway, Drug, ClinicalTrial · ENCODES, TARGETS, INHIBITS/ACTIVATES, CAUSES | Biomedical research / drug repurposing. |
| `fibo-aml-ext` | `fibo-core` | BeneficialOwner, AdverseMediaEvent, SAR · CONTROLS, FLAGGED_BY | KYC / AML beneficial-ownership traversal. |
| `acord-subrogation-ext` | `acord-core` | RecoveryOpportunity, Liablity Party · RECOVERS_FROM, AT_FAULT | Subrogation scouting via graph hops. |
| `fhir-payer-ext` | `fhir-core` + `acord-core` | PriorAuthorization, ExplanationOfBenefit · AUTHORIZES, ADJUDICATED_AS | Payer prior-auth & claims adjudication. |
| `cdisc-trials-ext` | `fhir-core` | Study, Arm, Endpoint, Subject · ENROLLS, MEASURES | Clinical-trial evidence synthesis. |

> When these ship they will live in
> [apps/ontology/standards/catalog.py](../apps/ontology/standards/catalog.py)
> as new `StandardSpec` entries and appear automatically under
> *Studio → Ontology → "Start from standard"* and `GET /api/v1/ontology/standards/`.
