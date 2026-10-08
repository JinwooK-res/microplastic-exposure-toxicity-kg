# Research Specification

## Project

**Microplastic Exposure–Toxicity Knowledge Graph**

A provenance-aware evidence-integration prototype for structuring published foodborne microplastic occurrence data and linking it to toxicological evidence in order to identify evidence gaps relevant to environmental health risk assessment.

The project uses ontology and knowledge-graph methods as **evidence-integration tools**, not as the research goal by themselves.

---

## Primary Research Question

**How well do microplastic characteristics observed in food overlap with those investigated in human-health toxicity studies?**

### Risk-assessment subquestion

**Which frequently observed foodborne microplastic profiles lack comparable toxicological evidence when polymer, particle size, shape, and exposure route are considered?**

---

## Scientific Rationale

Foodborne microplastic occurrence studies are difficult to compare because food matrices, pretreatment procedures, filtration thresholds, identification methods, reporting units, and particle-characterization detail vary across studies.

The seed review for the exposure side is:

> Kwon J-H, Kim J-W, Pham TD, Tarafdar A, et al. (2020). *Microplastics in Food: A Review on Analytical Methods and Challenges*. International Journal of Environmental Research and Public Health, 17, 6710. https://doi.org/10.3390/ijerph17186710

This review is used as a **published, public, reproducible seed source** for reconstructing occurrence evidence. It is not treated as an unpublished internal dataset.

The toxicity side will be built separately from a small, curated set of public human-health-relevant toxicity studies.

The two evidence streams will be connected through particle-profile comparability rather than by assuming that a polymer identity alone establishes toxicological relevance.

---

## Core Evidence Model

### Exposure side

```text
Publication
     │
     ▼
OccurrenceObservation
     ├── observedIn ──────> FoodMatrix
     ├── usesMethod ──────> AnalyticalMethod
     ├── reports ─────────> ConcentrationMeasurement
     └── characterizes ───> ParticleProfile
                               │
                   ┌───────────┼───────────┐
                   ▼           ▼           ▼
                Polymer      Size        Shape
```

### Toxicity side

```text
Publication
     │
     ▼
ToxicityExperiment
     ├── tests ───────────> ParticleProfile
     ├── usesModel ───────> BiologicalModel
     ├── exposureRoute ───> ExposureRoute
     ├── hasDose ─────────> DoseMeasurement
     └── reports ─────────> ToxicityEndpoint
```

### Evidence matching layer

```text
OccurrenceObservation
          │
          ▼
     EvidenceMatch
          │
          ▼
ToxicityExperiment
```

`EvidenceMatch` must preserve the reason a pair is classified as matched, unmatched, or indeterminate.

---

## MVP v0.1 Scope

Version 0.1 is a **working prototype**, not a comprehensive microplastics knowledge base.

### Exposure evidence

Use representative records reconstructed from the published review tables.

Target:

- approximately **20–30 occurrence observations**
- coverage across representative food categories such as:
  - salts
  - fish
  - shellfish
  - processed foods

The MVP should preserve analytical context, not only concentration values.

### Toxicity evidence

Use a small manually curated public dataset.

Target:

- approximately **5–10 toxicity experiments**
- preference for studies with explicit:
  - polymer
  - particle size
  - particle shape, when available
  - exposure route
  - dose
  - biological model
  - biological endpoint
  - source provenance

### Required matching dimensions

1. Polymer
2. Particle size
3. Shape
4. Exposure route

### Explicitly excluded from v0.1

- aquatic ecotoxicity as the main toxicity domain
- full ECOTOX, ToxCast, CTD, PubChem, or other large-scale database integration
- AOP integration
- quantitative risk scoring
- derivation of RfD, MOE, or regulatory thresholds
- automated literature mining with an LLM
- Neo4j or production graph infrastructure
- comprehensive extraction of every study in the review
- unpublished or proprietary research data

---

## Core Classes

Keep the ontology intentionally small for v0.1.

1. `Publication`
2. `OccurrenceObservation`
3. `FoodMatrix`
4. `ParticleProfile`
5. `AnalyticalMethod`
6. `ConcentrationMeasurement`
7. `ToxicityExperiment`
8. `BiologicalModel`
9. `ToxicityEndpoint`
10. `EvidenceMatch`

### Controlled vocabularies

Use lightweight controlled terms for the following rather than building deep class hierarchies in v0.1.

#### Polymer

Examples:

- PE
- PP
- PS
- PET
- PVC
- PA
- PES
- other
- unknown

#### Shape

Examples:

- fiber
- fragment
- sphere
- film
- pellet
- other
- unknown

#### Exposure route

Examples:

- oral
- dietary
- gavage
- inhalation
- in_vitro
- other
- unknown

---

## Exposure Data Schema

Canonical processed file:

`data/processed/exposure_observations.csv`

Recommended fields:

```text
occurrence_id
publication_id
review_table
review_reference
food_category
food_item
species_or_product
sample_origin
sample_matrix
pretreatment
density_separation
filter_pore_size_um
identification_method
chemical_confirmation
concentration_min
concentration_max
concentration_mean
concentration_sd
concentration_unit
polymer
shape
particle_size_min_um
particle_size_max_um
source_page
source_note
extraction_note
```

### Exposure-side rules

- Missing information must remain missing; do not infer unsupported values.
- Use `unknown` only for controlled categorical terms when absence must be represented explicitly.
- Use empty/NA values for unavailable numeric values.
- Keep `filter_pore_size_um` separate from particle-size fields.
- Do **not** treat a filtration cutoff as an observed particle size.
- Preserve the original concentration unit.
- Do not silently convert non-detects to zero in the canonical dataset.
- If a display or analysis layer converts `n.d.` to zero, keep the original reported value and document the transformation separately.
- `chemical_confirmation` should distinguish spectroscopy-confirmed measurements from visual/staining-only observations when this can be supported from the source.

---

## Toxicity Data Schema

Canonical processed file:

`data/processed/toxicity_experiments.csv`

Recommended fields:

```text
toxicity_id
publication_id
polymer
shape
particle_size_min_um
particle_size_max_um
particle_size_nominal_um
biological_system
species
cell_line
exposure_route
dose_value
dose_unit
duration_value
duration_unit
endpoint
effect_direction
effect_measure
effect_value
human_relevance_note
source_page
source_note
extraction_note
```

### Toxicity-side rules

- All records must be traceable to a public source.
- Do not infer particle shape, dose, route, or model details if they are not reported.
- Keep in vivo and in vitro evidence distinguishable.
- Exposure route must be explicit because route relevance is part of Level 4 matching.
- A toxicity record without sufficient particle characterization may still be included, but uncertainty must propagate into matching as `UNKNOWN` / `indeterminate`.

---

## Publication Schema

Canonical processed file:

`data/processed/publications.csv`

Recommended fields:

```text
publication_id
doi
title
authors
year
journal
url
evidence_type
source_dataset
```

### Provenance rule

Every occurrence observation and every toxicity experiment must link to a `publication_id`.

No provenance-free record should enter the canonical graph.

---

## Evidence Match Schema

Derived file:

`data/derived/evidence_matches.csv`

Recommended fields:

```text
match_id
occurrence_id
toxicity_id
polymer_match
size_match
shape_match
route_match
match_level
match_status
size_overlap_fraction
match_rationale
```

### Three-state matching

Do not collapse missing evidence into disagreement.

Each comparison dimension should support:

- `TRUE`
- `FALSE`
- `UNKNOWN`

Examples:

- Exposure shape not reported + toxicity shape = sphere → `shape_match = UNKNOWN`
- Exposure shape = fiber + toxicity shape = sphere → `shape_match = FALSE`
- Exposure shape = fiber + toxicity shape = fiber → `shape_match = TRUE`

### Match status

Use:

- `matched`
- `unmatched`
- `indeterminate`

A pair with missing required information at a given match level should generally become `indeterminate`, not automatically `unmatched`.

---

## Matching Levels

Retain the staged matching concept already defined in the repository.

### Level 1

```text
Polymer
```

Rule:

- exact or normalized polymer match

### Level 2

```text
Polymer
+ Particle size
```

Rule:

- Level 1 satisfied
- numeric particle-size ranges overlap

### Level 3

```text
Polymer
+ Particle size
+ Shape
```

Rule:

- Level 2 satisfied
- shape is exact or explicitly compatible

### Level 4

```text
Polymer
+ Particle size
+ Shape
+ Relevant exposure route
```

Rule:

- Level 3 satisfied
- toxicity exposure route is relevant to foodborne/dietary human exposure

Potential route-compatible toxicity terms for dietary exposure may include:

- oral
- dietary
- gavage

Inhalation should not count as a Level 4 dietary-route match.

`unknown` route must remain indeterminate.

---

## Particle-Size Matching

Normalize particle size to micrometers when supported by the source.

For two numeric intervals:

```python
max(exposure_min, toxicity_min) <= min(exposure_max, toxicity_max)
```

indicates overlap.

For a nominal single size:

```text
min = nominal
max = nominal
```

may be used for MVP matching.

Do not equate:

- analytical detection cutoff
- filter pore size
- reported particle-size interval

These are different concepts and must remain separate.

---

## Optional Future Exposure Estimate

The seed review describes dietary intake conceptually as:

```text
TDI = Σ(IR × EF × C) / BW
```

However, v0.1 should **not** calculate TDI systematically because that would require additional intake-rate, frequency, body-weight, population, and regional assumptions.

Reserve an optional `ExposureEstimate` concept for v0.2.

Future integration may connect this project with the existing `microplastic-exposure-calculator` repository.

---

## SHACL Validation for v0.1

Keep validation minimal and useful.

### OccurrenceObservation

Require:

- `Publication`
- `FoodMatrix`
- concentration unit
- analytical/identification method

### ToxicityExperiment

Require:

- `Publication`
- `ParticleProfile`
- `BiologicalModel`
- `ToxicityEndpoint`

### EvidenceMatch

Require:

- linked `OccurrenceObservation`
- linked `ToxicityExperiment`
- `match_status`

Validation failures should be reported clearly and should not be silently ignored.

---

## Competency Questions

The MVP must support at least the following.

### Q1 — Occurrence profile

Which particle profiles are most frequently reported in food-occurrence studies?

### Q2 — Toxicity coverage

Which food-occurrence particle profiles have matching human-health-relevant toxicity evidence?

### Q3 — Evidence gap

Which frequently observed particle profiles have little or no comparable toxicity evidence?

Additional useful questions may include:

- Which polymers are reported in each food category?
- Which analytical methods are associated with particular food matrices?
- Which toxicity experiments use particle-size ranges overlapping those observed in food?
- How much evidence remains as the match requirement becomes stricter from Level 1 to Level 4?

---

## Canonical Technical Stack

For v0.1:

```text
Python
pandas
RDF / OWL
rdflib
SPARQL
SHACL
matplotlib
pytest
```

RDF/OWL is the canonical graph representation.

Do not add a graph database unless it materially improves the MVP.

---

## Target Repository Structure

```text
microplastic-exposure-toxicity-kg/
│
├── README.md
├── RESEARCH_SPEC.md
├── requirements.txt
│
├── data/
│   ├── processed/
│   │   ├── publications.csv
│   │   ├── exposure_observations.csv
│   │   └── toxicity_experiments.csv
│   │
│   └── derived/
│       └── evidence_matches.csv
│
├── ontology/
│   ├── mpetkg.ttl
│   └── shapes.ttl
│
├── src/
│   ├── build_graph.py
│   ├── match_evidence.py
│   ├── validate_graph.py
│   └── visualize.py
│
├── queries/
│   ├── q1_particle_profiles.sparql
│   ├── q2_matched_evidence.sparql
│   └── q3_evidence_gaps.sparql
│
├── outputs/
│   ├── graph.ttl
│   ├── evidence_coverage.csv
│   └── figures/
│
└── tests/
    ├── test_graph.py
    └── test_matching.py
```

---

## MVP Acceptance Criteria

Version 0.1 is complete only when all of the following are true:

1. At least about 20 representative food-occurrence observations are structured from public review evidence.
2. At least about 5 curated toxicity experiments are included with source provenance.
3. The processed datasets conform to documented schemas.
4. An RDF graph is generated successfully with `rdflib`.
5. Provenance links are retained for exposure and toxicity evidence.
6. The staged Level 1–4 matching logic runs.
7. `TRUE`, `FALSE`, and `UNKNOWN` are distinguished in matching.
8. SHACL validation runs and reports graph/data issues.
9. At least three competency queries execute successfully.
10. At least one evidence-coverage or evidence-gap output is generated.
11. Tests cover core graph-building and matching behavior.
12. README demonstrates a working prototype rather than only a future plan.

---

## Interpretation Boundary

The MVP identifies **comparability and evidence coverage**, not causality and not a complete health-risk estimate.

Do not claim:

- that a matched toxicity experiment proves risk at observed food concentrations
- that an unmatched profile is safe
- that absence of toxicity evidence is evidence of absence of harm
- that polymer identity alone determines toxicity

The intended output is a transparent map of **what evidence exists, how comparable it is, and where important gaps remain**.

---

## Longer-Term Direction

Potential v0.2+ extensions:

- connection to the existing microplastic exposure calculator
- dietary intake estimates
- broader toxicity evidence ingestion
- uncertainty and evidence-quality scoring
- controlled ontology alignment
- AOP or mechanistic integration
- cross-route exposure comparison
- decision-support views for quantitative risk assessment

These extensions must remain secondary to the central research goal:

> **transparent, traceable, uncertainty-aware integration of environmental exposure and toxicological evidence for risk assessment.**
