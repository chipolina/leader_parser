# Lead Pipeline — Implementation Plan

## 1. Project Goal

Build a simple, maintainable MVP for automated discovery and qualification
of Russian B2B companies as potential customers for AI automation services.

The final pipeline should be able to:

1. obtain company data from verified external sources;
2. preserve original source data;
3. normalize verified data;
4. filter companies using configurable rules;
5. enrich only filtered companies;
6. store facts together with their sources;
7. calculate an explainable lead score;
8. optionally use an LLM for inference;
9. export qualified leads to CSV/XLSX.

The system must be designed so that industry, region, filtering criteria,
scoring rules and enrichment sources can be changed without rewriting
the entire application.

---

# 2. Mandatory Development Rules

These rules apply to every implementation step.

## 2.1 Work incrementally

Implement ONLY the current step.

Never implement future steps automatically.

After completing the current step:

1. run tests;
2. run the relevant validation;
3. update documentation;
4. report changed files;
5. report validation results;
6. STOP.

The next step must be started explicitly by the user.

---

## 2.2 Verify external sources before implementation

For every external data source:

1. use the official source first;
2. verify the currently published version/release;
3. verify the actual download URL or API endpoint;
4. verify the actual format;
5. verify the actual schema;
6. verify authentication if an API is used;
7. verify documented limitations;
8. verify update frequency if documented;
9. verify pricing/free availability if relevant;
10. document the findings.

Do NOT assume:

- API availability;
- API endpoints;
- filenames;
- URL patterns;
- XML structure;
- JSON structure;
- CSV structure;
- field names;
- authentication methods;
- rate limits;
- pagination;
- update frequency;
- pricing;
- commercial usage rights.

If something cannot be verified:

`NOT CONFIRMED`

Do not build critical functionality around unconfirmed information.

Official documentation has priority over third-party sources.

---

## 2.3 Keep the MVP simple

Do not introduce unnecessary infrastructure.

Preferred initial architecture:

- Python;
- PostgreSQL;
- Docker Compose;
- pytest;
- simple configuration files;
- simple CLI;
- simple source-specific modules.

Do NOT introduce unless a demonstrated requirement appears:

- Kubernetes;
- Kafka;
- Celery;
- Airflow;
- Redis;
- message queues;
- microservices;
- distributed workers;
- event-driven architecture;
- complex dependency injection;
- generic data-source frameworks.

Prefer simple explicit code.

---

## 2.4 Code quality

Use:

- Python;
- type hints for public functions;
- small functions;
- clear names;
- docstrings for non-trivial functions;
- comments explaining WHY rather than obvious WHAT.

Avoid:

- unnecessary abstractions;
- premature optimization;
- generic frameworks;
- excessive class hierarchies;
- giant functions;
- duplicated business logic.

Code should be easy for a developer to debug.

---

## 2.5 Error handling

External integrations must handle relevant failures explicitly.

Depending on the source:

- timeout;
- connection errors;
- HTTP errors;
- retries;
- rate limits;
- pagination;
- malformed responses;
- invalid data;
- duplicate data;
- partial failures.

Do not use broad exception handling that hides the original error.

Errors must contain enough information to understand what failed.

---

## 2.6 Idempotency

Repeated execution of the same ingestion operation must not create
duplicate company records.

The pipeline should be safe to run repeatedly.

---

## 2.7 Raw data preservation

Never modify original source files.

Raw source data must be preserved exactly as downloaded.

Processing must happen on copies/parsing streams.

Every normalized fact must eventually be traceable to its source.

---

## 2.8 Documentation

After every step:

- update README.md if behavior changed;
- update source documentation when a source is added;
- document commands;
- document inputs;
- document outputs;
- document assumptions;
- document limitations.

Do not document assumptions as facts.

---

# 3. Project Structure

Use a simple structure similar to:

    lead-pipeline/
    ├── AGENTS.md
    ├── IMPLEMENTATION_PLAN.md
    ├── README.md
    ├── pyproject.toml
    ├── config/
    ├── data/
    │   ├── raw/
    │   └── processed/
    ├── docs/
    │   └── sources/
    ├── src/
    │   ├── ingestion/
    │   ├── normalization/
    │   ├── filtering/
    │   ├── enrichment/
    │   ├── scoring/
    │   ├── database/
    │   └── export/
    └── tests/

Do not create all modules immediately.

Create directories/modules when the corresponding implementation step
starts.

---

# Step 0 — Project Bootstrap

## Goal

Create the minimal Python project.

## Tasks

Create:

- pyproject.toml;
- src/;
- tests/;
- config/;
- data/raw/;
- data/processed/;
- docs/;
- README.md.

Configure pytest.

Create a minimal test proving that the project can run.

## Do not implement

- FNS integration;
- XML parsing;
- PostgreSQL;
- filtering;
- enrichment;
- scoring.

## Validation

Run:

    pytest

## Done when

- project structure exists;
- Python project can be executed;
- pytest runs successfully;
- test suite passes.

STOP.

---

# Step 1 — Discover and Download Current FNS RSMP Dataset

## Goal

Download the currently published FNS Unified Register of Small and Medium
Enterprises dataset.

Official source:

https://www.nalog.gov.ru/rn26/opendata/7707329152-rsmp/

## Critical requirement

Do NOT permanently hardcode the current release filename or URL.

At runtime, determine the currently published release from the official
FNS dataset page.

Verify:

- dataset ID;
- current release;
- download URL;
- filename;
- format;
- XSD availability;
- XSD URL;
- XSD filename;
- publication/update date;
- actuality date.

Do not rely on values from this plan if the FNS website has changed.

## Tasks

Implement source discovery and download.

Store raw files under:

    data/raw/rsmp/

Expected structure:

    data/raw/rsmp/
    ├── <current-release>.zip
    ├── <current-xsd>.xsd
    └── manifest.json

The ZIP and XSD must not be modified.

## manifest.json

Store verified metadata:

- source_name;
- source_url;
- dataset_id;
- release_filename;
- release_url;
- xsd_filename;
- xsd_url;
- format;
- last_update_date;
- actuality_date;
- downloaded_at;
- sha256;
- file_size_bytes.

If a value is not available from the official source:

- use null;
- document why;
- do not invent a value.

## Download requirements

Implement:

- timeout;
- HTTP error handling;
- streaming download;
- SHA-256 calculation;
- file-size calculation;
- ZIP integrity validation;
- idempotent download;
- `--force` option.

## Do not implement

- XML parsing;
- normalization;
- PostgreSQL;
- filtering;
- enrichment;
- scoring.

## Tests

Test:

1. manifest generation;
2. SHA-256;
3. ZIP integrity;
4. existing-file detection;
5. `--force`;
6. HTTP failure handling.

Use local fixtures/mocks.

Do not store the full FNS dataset in the repository.

## README

Document:

- official source;
- how current release discovery works;
- where files are stored;
- how to run ingestion;
- `--force`;
- manifest contents;
- limitations.

Clearly state that XML parsing has not yet been implemented.

## Validation

Run the downloader against the real official FNS source.

Verify:

- ZIP exists;
- XSD exists;
- ZIP is valid;
- SHA-256 is recorded;
- manifest is correct;
- release metadata corresponds to the official FNS page.

STOP.

---

# Step 2 — Inspect Actual RSMP XML/XSD Structure

## Goal

Determine the actual structure of the current RSMP dataset before writing
normalization code.

Use ONLY the files downloaded during Step 1.

## Tasks

Inspect:

- XSD;
- representative XML files from the actual archive.

Determine the actual:

- XML elements;
- XML attributes;
- XML paths;
- data types;
- required/optional fields;
- identifiers;
- company name;
- INN;
- OGRN;
- KPP if present;
- OKVED;
- region;
- address;
- registration date;
- company status;
- SME category;
- employee information if present;
- other useful fields.

For each field document:

- source;
- XML path;
- field name;
- type;
- optional/required;
- meaning;
- example.

## Create

    docs/sources/rsmp.md

## Critical requirement

Do not infer field names.

Do not copy field structures from old documentation.

Do not use third-party schemas as the source of truth.

If a field cannot be confirmed:

    NOT CONFIRMED

## Validation

Every documented field must be traceable to the actual XSD/XML.

## Done when

The current RSMP schema is documented.

STOP.

---

# Step 3 — Implement RSMP XML Parser

## Goal

Convert the verified RSMP XML structure into Python records.

## Tasks

Implement a simple parser based only on the schema documented in Step 2.

Parse only fields required for the MVP.

Prefer streaming XML parsing if required by actual file size.

Handle:

- missing optional values;
- malformed XML;
- unexpected values;
- duplicate records.

Use small Python data structures/dataclasses where useful.

Do not introduce a generic XML framework.

## Tests

Create small fixtures based on the real source structure.

Test:

- valid record;
- optional missing fields;
- malformed XML;
- duplicate records;
- relevant identifier extraction.

## Validation

Run the parser against a real XML file from the downloaded dataset.

## Done when

Real RSMP XML can be converted into validated Python records.

STOP.

---

# Step 4 — Source and Ingestion Metadata

## Goal

Make ingestion reproducible and traceable.

## Tasks

Record:

- source;
- dataset;
- release;
- filename;
- download timestamp;
- SHA-256;
- parser version.

Implement a simple ingestion metadata mechanism.

Do not over-engineer this.

## Validation

Running the same source twice must not create ambiguous or duplicate
ingestion records.

## Done when

Every imported dataset can be identified and reproduced.

STOP.

---

# Step 5 — PostgreSQL Foundation

## Goal

Introduce PostgreSQL only after the source schema has been verified.

## Technology

Use:

- PostgreSQL;
- Docker Compose.

Do not add additional infrastructure.

## Initial tables

Create only the tables required for the current source:

    companies
    company_identifiers
    source_records
    ingestion_runs

Exact columns must be based on verified source data.

## Company identity

Use an internal database identifier:

    companies.id

Do not use INN or OGRN as the internal primary key.

Store government identifiers separately.

Example conceptual structure:

    company_identifiers
    ├── company_id
    ├── identifier_type
    ├── identifier_value
    └── source

Create appropriate uniqueness constraints and indexes.

## Tasks

Implement:

- database schema;
- connection configuration;
- insertion;
- idempotent upsert;
- duplicate protection.

## Validation

Load a sample of real RSMP data.

Verify:

- records inserted;
- duplicate identifiers do not create duplicate companies;
- foreign keys work;
- indexes/constraints work.

STOP.

---

# Step 6 — FNS Employee Data

## Goal

Add official FNS average employee-count data.

Official source:

https://www.nalog.gov.ru/rn69/opendata/7707329152-sshr2019/

## Before implementation

Verify from the current official FNS page:

- current release;
- download URL;
- filename;
- format;
- XSD;
- update date;
- actuality date;
- actual XML structure.

Do not use values from this plan as permanent source configuration.

## Tasks

Follow the same sequence:

    verify
      ↓
    download
      ↓
    preserve raw data
      ↓
    inspect XSD/XML
      ↓
    document
      ↓
    parse
      ↓
    connect to companies

## Store

Employee count together with:

- reporting year;
- source;
- source release;
- retrieval/import metadata.

Do not store only the current employee count without its time context.

## Done when

Verified FNS employee data is linked to companies.

STOP.

---

# Step 7 — FNS Financial Data

## Goal

Add official financial information available from the FNS open dataset.

Official source:

https://www.nalog.gov.ru/opendata/7707329152-revexp/

## Before implementation

Verify:

- current release;
- download URL;
- filename;
- format;
- XSD;
- update date;
- actuality date;
- actual fields.

## Important

Do not assume that the dataset provides:

- profit;
- EBITDA;
- assets;
- liabilities;
- cash;
- margin.

Only implement fields confirmed in the actual source.

## Tasks

Follow:

    verify
      ↓
    download
      ↓
    raw storage
      ↓
    inspect XSD/XML
      ↓
    documentation
      ↓
    parser
      ↓
    database

Store reporting year and source metadata.

## Done when

Financial facts can be associated with companies and reporting periods.

STOP.

---

# Step 8 — Configurable Company Filtering

## Goal

Reduce the company universe using deterministic configurable rules.

## Configuration

Use a simple configuration file.

Example:

    filters:
      regions: []

      okved:
        include: []
        exclude: []

      employees:
        min: null
        max: null

      revenue:
        min: null
        max: null

These are examples only.

Do not treat these values as recommended business thresholds.

## Requirements

Filtering must be:

- deterministic;
- configurable;
- testable;
- independent from enrichment;
- independent from LLM.

Changing thresholds must not require code changes.

## Filtering reasons

For excluded companies store or report reasons such as:

    employees_below_minimum
    employees_above_maximum
    region_not_allowed
    okved_not_allowed
    revenue_below_minimum

Only use reasons supported by actual available data.

## Validation

Given identical input and configuration, output must be identical.

Test boundary values and missing values.

## Done when

A configurable filter can reduce the company dataset reproducibly.

STOP.

---

# Step 9 — Website Enrichment Research and Implementation

## Goal

Obtain useful public company website information only for companies that
passed initial filtering.

## First research

Before coding, determine how websites can actually be obtained.

Possible sources may include:

- official company data;
- public company directories;
- search results;
- company websites themselves.

For every candidate source verify:

- availability;
- format;
- access method;
- API availability;
- usage restrictions;
- automation restrictions;
- relevant legal/ToS considerations.

Do not assume a website field exists in FNS data.

## Implementation

After the source is verified, implement the simplest viable approach.

HTTP fetching should support:

- timeout;
- reasonable retry;
- HTTP error handling;
- content size limits;
- logging;
- retrieval timestamp.

Do not build a general-purpose crawler.

Initially fetch only the minimum pages needed.

## Facts

Store evidence separately from inference.

Examples of possible facts:

- website URL;
- page title;
- company description;
- services;
- career page;
- technology indicators.

Only store facts that are actually extracted.

## Legal/technical limitations

Document:

- robots.txt considerations;
- Terms of Service;
- rate limits if applicable;
- personal-data risks;
- scraping limitations.

## Done when

A filtered company can be enriched with verified public website information.

STOP.

---

# Step 10 — Vacancy Enrichment Research

## Goal

Determine whether vacancy data can provide useful signals for lead scoring.

Primary candidate:

HeadHunter API.

Official documentation:

https://api.hh.ru/openapi/redoc

## Critical requirement

Do NOT implement the API before verifying current official documentation
and current usage conditions.

Verify:

- required endpoint;
- authentication;
- actual request format;
- actual response format;
- pagination;
- rate limits;
- restrictions;
- commercial use;
- data usage rights;
- current Terms of Service.

## Decision

If commercial use for this lead-generation scenario is clearly allowed:

Implement the smallest useful integration.

If commercial use is not clearly allowed:

Do not implement the source.

Document:

    NOT IMPLEMENTED

and explain why.

Do not search for an unofficial workaround.

## If implemented

Store only signals needed for scoring.

Do not collect unnecessary vacancy information.

Potential signals may include:

- vacancy count;
- vacancy categories;
- recent vacancy activity;

but only if confirmed by the actual API response.

## Done when

The source is either safely implemented or explicitly rejected based on
verified current conditions.

STOP.

---

# Step 11 — Evidence / Facts Layer

## Goal

Make every important data point traceable to a source.

## Concept

Separate:

    FACT

from:

    INFERENCE

A fact should contain, where available:

- value;
- source;
- source URL;
- source release;
- retrieved_at;
- reporting period;
- company ID.

Example:

    FACT
    employees = 85
    source = FNS
    reporting_year = 2025

## Requirements

Do not allow scoring or LLM inference to rely on unexplained values.

Every scoring input should be traceable to a stored fact.

## Done when

A developer can trace an important company attribute back to its source.

STOP.

---

# Step 12 — Deterministic Lead Scoring

## Goal

Create an explainable 0–100 lead score without an LLM.

## Important

Do not invent final business weights before examining real data.

Define the scoring system so weights can be configured.

Possible dimensions:

- company size;
- financial capacity;
- growth indicators;
- operational complexity;
- industry fit;
- enrichment signals.

These dimensions are hypotheses, not confirmed scoring criteria.

## Requirements

Each score component must be based on actual facts.

Example:

    lead_score = 72

    components:
      company_size: 18
      financial_capacity: 15
      growth_signal: 20
      operational_complexity: 11
      industry_fit: 8

Store:

- total score;
- components;
- scoring version;
- timestamp.

## Validation

The same facts and scoring configuration must produce the same result.

## Done when

The score is deterministic and explainable.

STOP.

---

# Step 13 — LLM Lead Analysis

## Goal

Use an LLM only for analysis that cannot be handled reliably with
deterministic rules.

## Input

The LLM receives verified facts only.

Example:

    Company facts:

    Employees: 85
    Revenue: ...
    Vacancies: ...
    Industry: ...
    Website: ...

## LLM may infer

- potential AI automation opportunities;
- likely operational pain points;
- prioritization;
- confidence.

## LLM must NOT invent

- employee count;
- revenue;
- customers;
- technologies;
- business processes;
- contacts;
- decision makers;
- other company facts.

## Output

Store separately:

    FACTS

and:

    INFERENCE

Store:

- model;
- prompt version;
- input facts;
- output;
- created_at.

## Validation

Create test examples where the LLM must distinguish fact from inference.

## Done when

LLM output is traceable to the facts provided to it.

STOP.

---

# Step 14 — Lead Export

## Goal

Produce a human-readable lead database.

Start with CSV.

Potential fields:

    company
    inn
    ogrn
    industry
    okved
    region
    employees
    employees_year
    revenue
    revenue_year
    website
    lead_score
    score_reason
    potential_use_cases
    source_urls
    last_updated

Do not include fields that are not actually available.

## Requirements

Every important output value must be traceable to stored data.

## Done when

A user can open the CSV and manually evaluate the leads.

STOP.

---

# Step 15 — Data Quality Validation

## Goal

Measure the quality of the pipeline.

## Implement checks for

- total companies;
- unique INN;
- unique OGRN;
- duplicates;
- missing values;
- invalid identifiers;
- invalid numeric values;
- source coverage;
- enrichment coverage;
- scoring coverage;
- score distribution.

## Manual validation

Select a small sample of companies and manually compare the pipeline
results with the original official sources.

Document:

- sample size;
- discrepancies;
- error types;
- correction actions.

## Done when

The pipeline produces a repeatable data-quality report.

STOP.

---

# Step 16 — End-to-End MVP

## Goal

Run the complete pipeline from source ingestion to lead export.

Expected conceptual flow:

    Official Sources
          ↓
    Raw Data
          ↓
    Parsing
          ↓
    Normalization
          ↓
    PostgreSQL
          ↓
    Filtering
          ↓
    Enrichment
          ↓
    Verified Facts
          ↓
    Rule-based Score
          ↓
    LLM Inference
          ↓
    Lead Export

## Requirements

The final MVP must:

- be reproducible;
- be idempotent;
- preserve raw data;
- preserve source information;
- have configurable filters;
- have versioned scoring;
- clearly separate facts and inference;
- provide useful logs;
- provide tests;
- provide documentation.

## Final validation

Run the pipeline from a clean environment on a deliberately limited
dataset first.

Verify every stage independently.

Only after the limited run succeeds should the pipeline be tested against
a larger dataset.

## Done when

The complete pipeline can be executed reproducibly and produces a
reviewable lead dataset.

STOP.