# Microplastic Exposure–Toxicity Knowledge Graph

An ontology-driven research project for connecting **foodborne microplastic occurrence, human exposure, particle characteristics, and toxicological evidence**.

The project is currently in the **research-design and data-acquisition stage**.

## Research Question

**How well do microplastic characteristics observed in food overlap with those investigated in human-health toxicity studies?**

## Motivation

Microplastic occurrence, exposure, and toxicity information is commonly reported across separate studies and databases using heterogeneous terminology and experimental conditions.

This project aims to develop a semantic framework connecting:

```text
Food
  ↓
Microplastic Occurrence
  ↓
Particle Characteristics
(polymer, size, shape, etc.)
  ↓
Exposure Evidence
  ↓
Toxicity Experiments
  ↓
Biological Endpoints
```

The objective is not simply to construct an ontology.

The longer-term goal is to use the resulting knowledge graph to identify gaps between **microplastic profiles observed in food** and **available toxicological evidence**.

## Initial Scope

Version 0.1 will focus on:

- foodborne microplastic occurrence
- human exposure
- polymer identity
- particle size
- particle shape
- human-health toxicity evidence
- study-level provenance

The initial version will not include:

- aquatic ecotoxicity
- AOP integration
- quantitative risk scoring
- unpublished or proprietary research data

## Planned Evidence Matching

Toxicity evidence will eventually be compared with occurrence data at increasing levels of specificity:

```text
Level 1
Polymer

Level 2
Polymer
+ Particle size

Level 3
Polymer
+ Particle size
+ Shape

Level 4
Polymer
+ Particle size
+ Shape
+ Relevant exposure route
```

This framework is intended to examine how toxicity-evidence coverage changes as more exposure-relevant particle characteristics are considered.

## Example Competency Questions

The future knowledge graph should support questions such as:

- Which polymers are reported in each food category?
- Which particle characteristics dominate foodborne microplastic occurrence?
- Which polymers have corresponding human-health toxicity evidence?
- Which toxicity experiments use particle sizes overlapping those observed in food?
- How much toxicity evidence remains when polymer, size, and shape are matched simultaneously?
- Which frequently observed microplastic profiles have limited toxicological evidence?
- Which biological endpoints are associated with exposure-relevant particle profiles?

## Planned Technical Stack

```text
Python
pandas
RDF / OWL
rdflib
SPARQL
SHACL
matplotlib
```

RDF/OWL is planned as the canonical semantic representation.

Additional graph databases or visualization tools may be added later if useful.

## Planned Outputs

1. Microplastic exposure–toxicity ontology
2. Harmonized occurrence and toxicity data model
3. RDF knowledge graph
4. SPARQL competency queries
5. SHACL validation
6. Exposure–toxicity evidence coverage analysis
7. Evidence-gap visualizations
8. Reproducible research workflow

## Current Status

**Research design / data acquisition**

Current next steps:

1. retrieve and inspect the original food microplastic occurrence dataset
2. inspect the structure of candidate toxicity databases
3. define the intermediate data model
4. finalize competency questions
5. design the minimal ontology
6. build the first RDF knowledge graph
7. conduct exposure–toxicity evidence-gap analysis

The detailed ontology structure will remain intentionally minimal until the source-data schema has been verified.

## Research Direction

The longer-term objective is to explore how semantic data integration can support transparent and traceable environmental exposure and risk assessment.

## License

MIT License. See `LICENSE`.
