# MLOps Platform Data Architecture on Snowflake

## 1. Purpose and Scope

This document defines a target data architecture for an MLOps platform based on Snowflake. The architecture is designed to support:

- model training
- batch inference
- near-real-time scoring preparation
- feature management
- model monitoring
- retraining workflows

The first implementation scope is an MVP for one end-to-end batch ML use case. The MVP covers data ingestion, curation, feature generation, training data preparation, model training handoff, batch inference, and monitoring feedback.

## 2. Target Operating Model

### Platform principles

- Snowflake is the central data foundation for governed analytical and ML-relevant data.
- Domain teams own their data products and business semantics.
- The central platform team provides shared standards, templates, security controls, and reusable platform capabilities.
- ML artifacts must be reproducible through explicit versioning of data, features, pipelines, and models.

### Responsibility split

| Area | Primary owner | Responsibility |
| --- | --- | --- |
| Source ingestion | Data engineering | Load source data into landing and raw zones |
| Standardization and curation | Data engineering with domain owners | Harmonize schemas, quality controls, business-ready datasets |
| Feature definitions | ML engineering with domain owners | Define reusable features and serving requirements |
| Model training and registration | ML engineering | Build models, track experiments, register versions |
| Governance and security | Platform team | Access model, classification, auditing, environment isolation |
| Monitoring and retraining triggers | ML engineering with platform team | Drift, quality, and performance monitoring |

## 3. Logical Data Zones

The platform uses layered Snowflake zones with separate databases or schemas per environment.

| Zone | Purpose | Example contents |
| --- | --- | --- |
| `landing` | Initial handoff area for incoming data | staged files, ingestion manifests, connector drop-offs |
| `raw` | Immutable source-aligned persistence | source tables with minimal transformation |
| `standardized` | Conformed technical layer | normalized types, common identifiers, schema harmonization |
| `curated` | Business-ready, governed datasets | entity tables, aggregates, model input candidates |
| `feature` | Reusable feature datasets | point-in-time-safe features, feature metadata, feature snapshots |
| `training` | Training set definitions and snapshots | labels, training views, frozen training extracts |
| `inference` | Scoring inputs and outputs | batch requests, prediction results, model decision logs |
| `monitoring` | Observability and feedback signals | drift metrics, model KPIs, data quality outcomes |

### Recommended namespace pattern

For each environment:

- database per trust boundary or platform domain
- schema per zone
- object naming aligned to domain and lifecycle

Example:

- `ML_DEV.RAW_CUSTOMER`
- `ML_DEV.CURATED_SALES`
- `ML_DEV.FEATURE_CUSTOMER`
- `ML_PROD.MONITORING_MODEL`

## 4. Core Information Model

The platform should model the ML lifecycle through linked information objects.

### Core entities

| Entity | Description | Key relationships |
| --- | --- | --- |
| Source dataset | Raw technical representation of source data | feeds standardized datasets |
| Standardized dataset | Harmonized dataset with common technical rules | feeds curated datasets |
| Curated dataset | Business-consumable dataset | feeds feature calculations and training labels |
| Feature definition | Logical definition of a feature | owned by a domain or ML team |
| Feature materialization | Persisted feature values for entities and timestamps | derived from curated datasets |
| Label dataset | Supervised learning targets | joined with feature materializations |
| Training snapshot | Frozen training input for one model run | references feature version and label version |
| Model version | Registered trained model | references training snapshot and code/pipeline versions |
| Batch inference run | Execution record for a scoring run | references model version and input snapshot |
| Monitoring event | Quality or performance measurement | references model version and inference outputs |

### Reproducibility requirements

Every model version must be traceable to:

- the exact training snapshot
- the feature definition version and feature materialization timestamp
- the label dataset version
- the transformation and training pipeline version
- the hyperparameter and experiment context

Training data must be represented through frozen tables or versioned views so training can be recreated without ambiguity.

## 5. Feature Data Architecture

Features are treated as reusable data products rather than embedded one-off transformations.

### Feature architecture rules

- Separate feature definition from feature storage and feature consumption.
- Maintain point-in-time correctness for training use cases.
- Version features whenever business logic changes.
- Record feature ownership, refresh cadence, source lineage, and quality expectations.
- Reuse the same feature semantics across training and inference wherever feasible.

### Feature object categories

| Object type | Purpose |
| --- | --- |
| Feature catalog | Metadata about feature name, owner, entity, grain, refresh cadence |
| Feature definitions | Declarative or documented logic for deriving each feature |
| Feature tables | Persisted feature values by entity and event time |
| Feature snapshots | Frozen feature sets used by a given training run |
| Serving extracts | Output structures prepared for downstream scoring systems |

## 6. Pipelines and Orchestration

The architecture separates the major pipeline types while keeping lineage intact.

| Pipeline | Main outcome | Primary execution location |
| --- | --- | --- |
| Ingestion | Load source data into landing/raw | connectors or ingestion tools plus Snowflake |
| Standardization | Conform technical structure | Snowflake |
| Curation | Build business-ready datasets | Snowflake |
| Feature generation | Compute reusable features | Snowflake |
| Training set assembly | Create point-in-time-safe snapshots | Snowflake |
| Model training | Train and evaluate models | external ML runtime with Snowflake data access |
| Batch scoring | Produce predictions at scale | external ML runtime or Snowflake-adjacent execution |
| Monitoring | Compute drift and quality metrics | Snowflake plus observability integrations |

### Orchestration principles

- Use scheduled pipelines for predictable recurring workloads.
- Use event-driven triggers for source arrivals, model promotion, or retraining conditions.
- Keep orchestration metadata separate from business data, but link runs through stable run identifiers.
- Persist status and lineage for every pipeline run.

## 7. Governance, Security, and Access

### Security model

- Isolate development, test, and production environments.
- Separate experimental, governed, and sensitive workloads.
- Apply role-based access controls to schemas, tables, and views.
- Use row-level and column-level protections for restricted attributes.
- Enforce auditable access to sensitive training and inference data.

### Role model

| Role | Access pattern |
| --- | --- |
| Platform admin | Platform-wide administration and policy control |
| Data engineer | Write access in landing/raw/standardized/curated zones |
| ML engineer | Read curated data, manage feature/training/inference assets |
| Analyst | Read-only access to approved curated and monitoring outputs |
| Domain steward | Approval and governance oversight for domain datasets |
| Service role | Restricted runtime access for automated pipelines |

### Governance controls

- Data classification for public, internal, confidential, and restricted data.
- Approval process for promoting assets into curated and feature zones.
- Full lineage from source to feature to model to inference output.
- Audit trail for schema changes, access, promotions, and model usage.

## 8. Observability and Data Quality

The platform should manage technical and ML-specific observability as separate but linked concerns.

### Data quality controls

- freshness checks for inbound and curated datasets
- schema change detection
- completeness and null-threshold checks
- referential integrity on shared entity identifiers
- distribution checks for critical features and labels

### ML monitoring controls

- feature drift
- prediction drift
- model performance degradation
- data volume anomalies
- batch scoring failures and retries

### Monitoring storage

Monitoring results should be persisted in the `monitoring` zone with references to:

- dataset version
- feature version
- model version
- inference run identifier
- observation timestamp

## 9. Cost and Performance Model

### Cost principles

- separate compute workloads by ingestion, transformation, training preparation, and monitoring
- keep heavy feature computations materialized when reuse justifies the cost
- archive inactive intermediate assets based on retention policy
- distinguish development experimentation from production-grade workloads

### Performance principles

- cluster or optimize large fact-style tables around dominant entity and time access patterns
- use incremental processing for standardized, curated, and feature layers where possible
- materialize expensive joins that are reused across training and scoring
- limit broad access to raw data in production workloads

## 10. Environment and Lifecycle Model

The architecture requires strict lifecycle separation.

| Environment | Purpose |
| --- | --- |
| Dev | Fast experimentation for pipelines, features, and model development |
| Test | Controlled integration and release validation |
| Prod | Governed production workloads and monitoring |

### Promotion path

1. Define or change a dataset, feature, or pipeline in dev.
2. Validate structure, quality, and access controls in test.
3. Promote approved assets to prod.
4. Register production model versions with explicit links to approved data and feature assets.

### Required lifecycle traceability

- code version
- pipeline version
- data object version or snapshot identifier
- feature version
- model version
- deployment and scoring run identifier

## 11. Logical Architecture Diagram

```mermaid
flowchart LR
    A[Operational Sources] --> B[Landing Zone]
    B --> C[Raw Zone]
    C --> D[Standardized Zone]
    D --> E[Curated Zone]
    E --> F[Feature Zone]
    E --> G[Label Datasets]
    F --> H[Training Snapshots]
    G --> H
    H --> I[External Training Runtime]
    I --> J[Model Registry]
    J --> K[Batch Inference Runs]
    E --> K
    F --> K
    K --> L[Prediction Outputs]
    L --> M[Monitoring Zone]
    F --> M
    E --> M
    J --> M
```

## 12. MVP Scope

The first MVP should implement one batch-oriented ML use case with the following scope:

1. ingest one operational source into landing and raw
2. standardize and curate the source into model-ready business entities
3. define a first reusable feature set with ownership and refresh cadence
4. create versioned training snapshots with labels
5. train and register one model version outside Snowflake with lineage back to Snowflake assets
6. execute batch inference and persist predictions
7. store monitoring outputs for data freshness, feature drift, and prediction quality

## 13. Suggested Initial Deliverables

- zone and naming standard for Snowflake objects
- domain ownership matrix
- feature catalog template
- training snapshot and model lineage specification
- role and access matrix
- monitoring KPI catalog
