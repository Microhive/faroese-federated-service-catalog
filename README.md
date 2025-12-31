# Proposed Faroese Federated Service Catalog

Federated service & process catalog for the Faroe Islands, modeled in **CPSV-AP (JSON-LD)** and maintained as **docs-as-code in Git**.  
Designed to enable cross-organisation discovery, duplicate detection, and measurable process/compliance checks (e.g., missing SLA per step, missing data sharing agreements), and to be exposed to AI agents via **MCP**.

---

## What problem this solves

Public-sector services are often documented in many places and formats. This project aims to make services:

- **Findable** (one cross-agency overview)
- **Comparable** (spot duplicates/redundant services)
- **Auditable** (who changed what, when, and why)
- **Measurable** (SLA/DSA/legal/control completeness)
- **AI-consumable** (structured records + step-by-step execution plans)
- **Federated** (each organisation owns and maintains its own catalog)

---

## Standards-first approach

We model service catalog entries using:

- **CPSV-AP 3.2.0** as the canonical service description format (JSON-LD)
  - Spec: https://semiceu.github.io/CPSV-AP/releases/3.2.0/
  - JSON-LD context: https://semiceu.github.io/CPSV-AP/releases/3.2.0/context/cpsv-ap.jsonld
  - SHACL shapes: https://semiceu.github.io/CPSV-AP/releases/3.2.0/shacl/cpsv-ap-SHACL.ttl

CPSV-AP is excellent for “what the service is”.  
For “how the service executes” (step-by-step), we attach an **Execution Plan** as JSON-LD (MVP extension vocabulary). This keeps the catalog CPSV-AP compliant while enabling workflow-level checks.

---

## Repository scope (what lives here)

This repository is the **federation hub**:

- **Standards & profiles**
  - CPSV-AP profile decisions (what is required for MVP)
  - Controlled vocabularies (channels, categories, data classes, etc.)
  - Extension vocabulary for execution plans and compliance metadata

- **Templates**
  - Starter template for organisation catalogs (folder structure + examples)

- **Registry**
  - List of participating organisation repositories (for the crawler)

- **Tools (reference implementation)**
  - Validator (JSON-LD + SHACL + MVP rules)
  - Crawler/aggregator (index services from org repos)
  - MCP adapter (expose aggregated catalog to AI agents)

> Note: Tools may be implemented iteratively. The MVP starts with validation + crawling + basic search.

---

## Federation model

### Ownership
Each organisation maintains its own Git repository with its services.

Example:
- `almannaverkid-service-catalog`
- `heilsuverid-service-catalog`
- `kommunur-service-catalog`

### Central overview
Government (or a central unit) provides an overview by:
1. reading a registry of repo sources,
2. cloning/pulling those repos,
3. validating CPSV-AP JSON-LD,
4. indexing records centrally,
5. exposing search + explanations (and later: deduplication analytics, compliance reporting).

---

## Folder structure (suggested)

### Organisation repository layout

Each organisation repository follows this structure:

```
org-service-catalog/
├── catalog.jsonld             # DCAT Dataset describing this organization's catalog
├── services/
│   └── <service-id>/
│       ├── service.jsonld           # CPSV-AP core record
│       └── execution-plan.jsonld    # MVP extension (optional step-by-step plan)
└── README.md
```

### Federation hub (this repository)

```
faroese-federated-service-catalog/
├── README.md
├── profiles/
│   └── mvp-profile.md         # MVP-required fields documentation
├── vocabularies/
│   └── extension-vocab.jsonld # Extension vocabulary (ex:ExecutionPlan, etc.)
├── registry/
│   └── repos.json             # List of participating org repositories
├── services/                  # Example services (reference implementations)
│   └── aettleidingarstudul/
│       ├── service.jsonld
│       └── execution-plan.jsonld
├── tools/
│   ├── validator/             # JSON-LD + SHACL validation
│   ├── crawler/               # Repository aggregator
│   └── mcp-adapter/           # MCP server wrapper
└── .github/
    └── workflows/
        └── validate.yml       # CI validation pipeline
```

---

## MVP-required fields (CPSV-AP profile)

Every service must include these fields:

| Entity | Required Fields |
|--------|----------------|
| `PublicService` | `identifier`, `title`, `description`, `hasCompetentAuthority`, `processingTime` |
| `Channel` | `identifier`, optionally `type`, `hasInput` |
| `Evidence` | `identifier`, `title`, optionally `page` (link to forms/docs) |
| `hasLegalResource` | At least one legal resource (policy/DSA references) |

### MVP extension (Execution Plan)

For process/workflow-level checks, attach an `ex:ExecutionPlan` as JSON-LD with:

- `schema:step` array of `schema:HowToStep` entries
- Per-step `ex:stepSla` (ISO8601 duration)
- `ex:requiresDataSharingAgreement` for cross-org data sharing

---

## Validation pipeline (CI)

The validation pipeline checks:

1. **JSON-LD parsing** - Valid JSON-LD syntax
2. **SHACL validation** - CPSV-AP conformance
3. **MVP completeness rules**:
   - Missing `processingTime` ⇒ "Missing SLA"
   - Missing `hasLegalResource` when cross-org participation exists ⇒ "Missing DSA/legal basis"
   - Channels missing `type` ⇒ "Ambiguous access method"
   - Evidence missing `page` ⇒ "No application/form link"
   - `ex:stepSla: null` ⇒ "Missing step SLA"
   - `ex:involvesExternalOrganisation` without `ex:requiresDataSharingAgreement` ⇒ "Missing DSA"

---

## MCP adapter (AI agent interface)

The MCP server wrapper exposes:

| Type | Endpoint | Description |
|------|----------|-------------|
| Resource | `service://<id>` | Returns CPSV-AP JSON-LD graph for a service |
| Tool | `search_services(query, filters)` | Returns IDs + short summaries |
| Tool | `explain_service(serviceId, persona)` | Plain-language walkthrough |
| Tool | `compliance_report(serviceId)` | Flags missing SLA/DSA/form links |

---

## Example: Ættleiðingarstuðul

See the complete example in [`services/aettleidingarstudul/`](services/aettleidingarstudul/):

- **Service**: Support up to 100,000 kr for foreign child adoption
- **Processing time**: 14 days once all documents are received
- **Competent authority**: Almannaverkið

### Key files:
- [`service.jsonld`](services/aettleidingarstudul/service.jsonld) - CPSV-AP core record
- [`execution-plan.jsonld`](services/aettleidingarstudul/execution-plan.jsonld) - Step-by-step process with SLAs

> **Note**: The execution plan example intentionally includes a step with `ex:stepSla: null` (the "Prepare documents" step) to demonstrate what triggers the "Missing step SLA" validation error. In a production service, all steps should have defined SLAs.

---

## Quick checklist (MVP definition of done)

- [ ] Org repos contain CPSV-AP JSON-LD for each service
- [ ] CI validates via official SHACL + MVP completeness rules
- [ ] Central crawler builds an aggregated index
- [ ] MCP adapter supports: search → fetch service → fetch execution plan → compliance report
- [ ] Duplicate candidate report runs nightly (text similarity + evidence/output overlap)

---

## Getting started

### 1. Create a service record

Copy the template from `services/aettleidingarstudul/service.jsonld` and customize:

```json
{
  "@context": [
    "https://semiceu.github.io/CPSV-AP/releases/3.2.0/context/cpsv-ap.jsonld",
    { "ex": "https://example.gov.fo/vocab#" }
  ],
  "@graph": [
    {
      "@id": "https://services.example.gov.fo/id/publicservice/your-service-id",
      "@type": "PublicService",
      "identifier": "org:your-service-id",
      "title": "Your Service Title",
      "description": "Service description...",
      "hasCompetentAuthority": { "@id": "https://services.example.gov.fo/id/org/your-org" },
      "processingTime": "P14D",
      "hasChannel": [],
      "hasInput": [],
      "produces": [],
      "hasLegalResource": []
    }
  ]
}
```

### 2. Add an execution plan (optional)

Create `execution-plan.jsonld` with step-by-step process:

```json
{
  "@context": {
    "ex": "https://example.gov.fo/vocab#",
    "schema": "https://schema.org/"
  },
  "@type": "ex:ExecutionPlan",
  "schema:step": [
    {
      "@type": "schema:HowToStep",
      "name": "Step name",
      "text": "Step description",
      "ex:stepSla": "P0D"
    }
  ]
}
```

### 3. Run validation

```bash
# TODO: Validation tooling will be added
npm run validate
```

---

## References

- [CPSV-AP 3.2.0 Specification](https://semiceu.github.io/CPSV-AP/releases/3.2.0/)
- [CPSV-AP JSON-LD Context](https://semiceu.github.io/CPSV-AP/releases/3.2.0/context/cpsv-ap.jsonld)
- [CPSV-AP SHACL Shapes](https://semiceu.github.io/CPSV-AP/releases/3.2.0/shacl/cpsv-ap-SHACL.ttl)
- [Almannaverkið - Ættleiðingarstuðul](https://www.av.fo/fo/utgjoeld-og-veitingar/aettleidingarstudul)
- [Almannaverkið - Dátuvernd](https://www.av.fo/fo/um-almannaverkid/datuvernd)

