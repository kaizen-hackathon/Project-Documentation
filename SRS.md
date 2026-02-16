# Software Requirements Specification (SRS)
## Project: LLM Evaluation Platform

---

# 1. Introduction

## 1.1 Purpose

This document defines the requirements for the LLM Evaluation Platform, a modular system for benchmarking Large Language Models (LLMs) using domain-specific datasets, structured multi-metric scoring, and configurable LLM-as-Judge personalities.

The system enables reproducible, parameterized evaluation of target LLMs across multiple scoring dimensions.

---

## 1.2 Scope

The system shall:

- Allow users to configure evaluation jobs
- Evaluate target LLMs across curated datasets
- Support subjective and objective scoring metrics
- Support configurable Judge LLM personalities
- Support customizable judgement styles
- Store evaluation data in DuckDB
- Provide detailed analytical reporting

---

# 2. System Overview

## 2.1 Architecture Components

- Frontend UI
- FastAPI Backend
- Background Task Engine
- DuckDB (embedded analytical database)
- External LLM Providers (via boto3 / REST APIs)
- Judge LLM Module

---

## 2.2 High-Level Workflow

1. User selects:
   - Target LLM
   - Judge LLM
   - Judgement style
   - Use case
   - Dataset/category
2. Evaluation job is created.
3. Background tasks:
   - Generate responses from target LLM.
   - Evaluate responses using Judge LLM.
4. Metric scores stored.
5. Aggregations computed.
6. Results displayed on UI.

---

# 3. Functional Requirements

---

# 3.1 Evaluation Metrics

The system shall support the following scoring dimensions:

- Accuracy
- Hallucination
- Truthfulness
- Coherence
- Subjective Quality

Each metric must:

- Store numeric score (configurable scale, e.g., 0–5 or 0–10)
- Store textual reasoning
- Support aggregation
- Be independently queryable

---

# 3.2 Evaluation Identity Model

Each metric score shall be uniquely identified by:

- usecase_id
- category_id
- question_id
- judge_llm_id
- target_llm_id
- judgement_style_id
- metric_name

Composite uniqueness constraint:

(usecase_id, category_id, question_id, judge_llm_id, target_llm_id, judgement_style_id, metric_name)

---

# 3.3 Judge LLM Configuration

## 3.3.1 Judge Model Requirements

Each Judge LLM shall include:

- judge_llm_id
- base_model_name
- personality_name
- personality_description
- judgement_style_id
- version

---

## 3.3.2 Judgement Style Requirements

Each Judgement Style shall define:

- style_name
- scoring_scale
- metric_definitions
- prompt_template
- output JSON schema
- version

Judgement styles must enforce structured JSON output.

---

# 3.4 UI Requirements

## 3.4.1 Configuration Interface

The UI shall allow:

- Selection of target LLM
- Selection of judge LLM
- Selection of judgement style
- Selection of use case
- Selection of category/dataset
- Enabling/disabling metrics
- Triggering evaluation job

---

## 3.4.2 Results Interface

The UI shall display:

- Job status (CREATED, RUNNING, SCORING, COMPLETED, FAILED)
- Per-question scores
- Per-metric breakdown
- Judge reasoning text
- Aggregated scores
- Cross-model comparison
- Cross-judge comparison

---

# 3.5 API Requirements

## 3.5.1 Evaluation APIs

POST /evaluations  
GET /evaluations  
GET /evaluations/{job_id}  
GET /evaluations/{job_id}/results  

---

## 3.5.2 Configuration APIs

GET /usecases  
GET /categories  
GET /questions  
GET /target-llms  
GET /judge-llms  
GET /judgement-styles  

---

# 4. Background Task Requirements

---

## 4.1 Phase 1 – Target Response Generation

For each question:

- Send prompt to target LLM
- Store response in database

---

## 4.2 Phase 2 – Judge Evaluation

For each response:

- Retrieve question and reference answer
- Retrieve judge personality definition
- Retrieve judgement style template
- Construct judge prompt
- Send to Judge LLM
- Parse structured JSON output
- Store metric scores

---

## 4.3 Phase 3 – Aggregation

- Compute metric-level aggregation
- Compute category-level aggregation
- Compute use-case level aggregation
- Support weighted scoring

---

# 5. Data Model (DuckDB)

---

## 5.1 usecases

| Column | Type |
|--------|------|
| id | UUID |
| name | VARCHAR |
| description | TEXT |

---

## 5.2 categories

| Column | Type |
|--------|------|
| id | UUID |
| usecase_id | UUID |
| name | VARCHAR |

---

## 5.3 questions

| Column | Type |
|--------|------|
| id | UUID |
| category_id | UUID |
| prompt_text | TEXT |
| reference_answer | TEXT |

---

## 5.4 target_llms

| Column | Type |
|--------|------|
| id | UUID |
| model_name | VARCHAR |
| provider | VARCHAR |
| version | VARCHAR |

---

## 5.5 judge_llms

| Column | Type |
|--------|------|
| id | UUID |
| base_model | VARCHAR |
| personality_name | VARCHAR |
| personality_description | TEXT |
| version | VARCHAR |

---

## 5.6 judgement_styles

| Column | Type |
|--------|------|
| id | UUID |
| style_name | VARCHAR |
| scoring_scale | VARCHAR |
| prompt_template | TEXT |
| version | VARCHAR |
| created_at | TIMESTAMP |

---

## 5.7 responses

| Column | Type |
|--------|------|
| id | UUID |
| question_id | UUID |
| evaluation_id | UUID |
| target_llm_id | UUID |
| response_text | TEXT |
| created_at | TIMESTAMP |

---

## 5.8 metric_scores

| Column | Type |
|--------|------|
| id | UUID |
| usecase_id | UUID |
| category_id | UUID |
| question_id | UUID |
| judge_llm_id | UUID |
| target_llm_id | UUID |
| judgement_style_id | UUID |
| metric_name | VARCHAR |
| score | FLOAT |
| reasoning | TEXT |
| created_at | TIMESTAMP |

---

# 6. Non-Functional Requirements

---

## 6.1 Performance

- Must support 10,000+ questions per evaluation
- Must support parallel background processing
- Must not block API threads during evaluation

---

## 6.2 Scalability

- Support horizontal scaling of workers
- DuckDB used for analytical workload
- Migration path to distributed DB must be feasible

---

## 6.3 Reliability

- Retry LLM calls on transient failure
- Persist intermediate results
- Maintain idempotent job execution

---

## 6.4 Observability

- Structured logging
- Per-job logs
- Per-question evaluation trace
- Error tracking

---

# 7. Versioning & Reproducibility

Each evaluation shall store:

- Target LLM version
- Judge LLM version
- Judgement style version
- Prompt template snapshot
- Metric definitions snapshot

Evaluations must be reproducible.

---

# 8. Assumptions

- LLM calls are stateless
- Datasets are curated
- Judge templates are version-controlled
- DuckDB storage is persistent

---

# 9. Constraints

- DuckDB is file-based storage
- Evaluation speed depends on external LLM latency
- Heavy concurrency may require distributed architecture

---

# 10. Design Principle

Model output is treated as a hypothesis.  
Judge output is treated as structured interpretation under defined constraints.

The system must not conflate evaluation with objective truth.
