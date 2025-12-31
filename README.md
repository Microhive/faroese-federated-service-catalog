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

