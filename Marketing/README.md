# ABS Trade Data Product Bundle

A Databricks Asset Bundle for Australian Bureau of Statistics (ABS) trade data analytics with built-in data contracts and quality enforcement.

## Overview

This bundle provides a complete data product for analyzing ABS international trade data, including:

* **Table References**: Links to ABS trade datasets in Unity Catalog
* **Dashboard**: Interactive analytics dashboard for trade data visualization
* **Data Contract**: Formal schema definitions, quality rules, and SLAs
* **Quality Monitoring**: Automated validation and alerting
* **Documentation**: Deployment guides and configuration references

## Contents

### Tables

The bundle references the following ABS trade data tables:

* `workspace.abs_trade.metadata` - Structured metadata about ABS trade dataflows
* `workspace.abs_trade.raw_json` - Raw API responses from ABS
* `workspace.default.abs_trade_serv_state_cy_metadata` - State-level metadata
* `workspace.default.abs_trade_serv_state_cy_raw_json` - State-level raw data

### Dashboard

**ABS Trade Data Analysis** - An interactive dashboard featuring:

* Total record counters across all tables
* Metadata summary tables with dataset descriptions
* Record count visualizations by table
* State-level trade data details

Dashboard ID: `01f1321d5f29145c8dbff9df0ddf464c`

### Data Contract

The `data_contract.yml` file defines:

* **Schema Specifications**: Expected column names, types, and descriptions
* **Quality Rules**: Validation rules including:
  - Not null constraints
  - Unique value checks
  - Valid value lists
  - Pattern matching (regex)
  - Row count minimums
  - JSON format validation
* **SLAs**: 
  - 99.9% availability
  - 24-hour data freshness
  - 95% completeness
  - 99% accuracy
  - 7-year retention
* **Governance**: Data classification, ownership, and access control
* **Breaking Change Policy**: 90-day deprecation period with approval requirements

### Quality Monitoring

Automated quality checks run daily via scheduled jobs:

* **Schema Validation**: Verifies tables match contract specifications
* **Data Quality Checks**: Enforces all quality rules defined in contract
* **Alerting**: Email notifications on failures or SLA breaches
* **Reporting**: Detailed pass/fail reports with severity levels

Quality check notebook: `/Users/melissaslattery@gmail.com/quality_checks`

## Quick Start

### Prerequisites

1. Databricks CLI installed
2. Access to Databricks workspace: `https://dbc-1eff0d8e-9619.cloud.databricks.com`
3. SQL Warehouse ID for query execution (default: `6d2e141d82a63d88`)
4. Git for version tracking (recommended)

### Deploy

```bash
# 1. Validate bundle
databricks bundle validate

# 2. Deploy to dev (includes quality jobs)
databricks bundle deploy -t dev

# 3. For production, tag and deploy with version
git tag -a v1.0.0 -m "Release 1.0.0"
export GIT_TAG=$(git describe --tags)
databricks bundle deploy -t prod
```

See [DEPLOYMENT.md](DEPLOYMENT.md) for detailed instructions and [VERSION_MANAGEMENT.md](VERSION_MANAGEMENT.md) for git versioning.

## Bundle Structure

```
data_product/
├── databricks.yml              # Main bundle configuration
├── README.md                   # This file
├── DEPLOYMENT.md               # Deployment guide
├── VERSION_MANAGEMENT.md       # Git version tracking guide
├── .gitignore                  # Git ignore patterns
└── resources/
    ├── tables.yml              # Table references and metadata
    ├── dashboards.yml          # Dashboard configuration
    ├── data_contract.yml       # Data contract with quality rules
    └── jobs.yml                # Quality monitoring jobs
```

## Configuration

### Environments

* **dev**: Development environment (default)
* **prod**: Production environment with run_as configuration

### Variables

* `warehouse_id`: SQL Warehouse ID for dashboard queries (default: `6d2e141d82a63d88`)
* `bundle_version`: Version from git tag (optional, for tracking)
* `git_commit`: Git commit SHA (optional, for tracking)

## Data Quality & Governance

### Quality Rules

The data contract enforces quality rules on all tables:

| Rule Type | Description | Severity |
|-----------|-------------|----------|
| not_null | Required fields must have values | error |
| unique | Key fields must be unique | error |
| valid_values | Fields must match allowed values | error |
| pattern | Fields must match regex patterns | warning |
| valid_json | JSON fields must be parseable | error |
| row_count | Minimum row count requirements | error |

### SLA Guarantees

| Metric | Target | Monitoring |
|--------|--------|------------|
| Availability | 99.9% | Continuous |
| Freshness | < 24 hours | Hourly |
| Completeness | ≥ 95% | Daily |
| Accuracy | ≥ 99% | Daily |
| Retention | 7 years | N/A |

### Running Quality Checks

```python
# Manual quality check execution
databricks jobs run-now --job-name "ABS Trade Data - Quality Checks"

# Or run the notebook directly
# Open /Users/melissaslattery@gmail.com/quality_checks
```

Quality checks run automatically:
* **Daily at 2 AM UTC** - Full validation suite
* **On-demand** - Schema validation job

### Breaking Changes

The bundle enforces a breaking change policy:

* **Breaking changes** require approval and 90-day deprecation notice
* **Examples**: Column removal, type changes, renames
* **Non-breaking**: Column additions, constraint additions
* **Notifications**: Sent to product owner

## Data Sources

This bundle works with ABS trade data covering:

* **International Trade in Services by Classification (EBOPS) and State**
* Calendar year statistics
* Extended Balance of Payments Services classification
* State-level breakdowns

Catalogue number: 5368.0.55.004

## Deployment Targets

| Target | Mode | Purpose |
|--------|------|---------|
| dev | development | Development and testing |
| prod | production | Production deployment with access controls |

## Version Control

This bundle supports git-based version tracking. To ensure deployed versions match git versions:

1. **Tag releases** with semantic versioning (v1.0.0, v1.1.0, etc.)
2. **Export version variables** before deploying
3. **Track deployments** in git history

See [VERSION_MANAGEMENT.md](VERSION_MANAGEMENT.md) for complete guide.

## Monitoring and Alerts

Quality monitoring includes:

* **Automated Checks**: Daily validation of all quality rules
* **Email Alerts**: Failures sent to `melissaslattery@gmail.com`
* **Severity Levels**:
  - **Error**: Immediate attention required, may halt downstream processes
  - **Warning**: Should be addressed, but not blocking
* **Reports**: Detailed validation results with pass/fail status

## Support & Documentation

### Internal Documentation
* [DEPLOYMENT.md](DEPLOYMENT.md) - Deployment procedures
* [VERSION_MANAGEMENT.md](VERSION_MANAGEMENT.md) - Version control best practices
* [data_contract.yml](resources/data_contract.yml) - Complete data contract specification
* [Quality Checks Notebook](/Users/melissaslattery@gmail.com/quality_checks) - Quality validation implementation

### External Resources
* [Databricks Asset Bundles](https://docs.databricks.com/en/dev-tools/bundles/)
* [Bundle Configuration](https://docs.databricks.com/en/dev-tools/bundles/settings.html)
* [ABS Data Information](https://www.abs.gov.au/)
* [Semantic Versioning](https://semver.org/)

## Tags

* Source: `abs`
* Domain: `trade`
* Type: `analytics`
* Quality: `enforced`
* Contract: `v1.0.0`
