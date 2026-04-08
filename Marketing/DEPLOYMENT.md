# Deploying the ABS Trade Data Product Bundle

This guide explains how to deploy the `data_product` Databricks Asset Bundle with data contracts and quality enforcement.

## Prerequisites

1. **Databricks CLI** installed and configured
   ```bash
   # Install Databricks CLI (if not already installed)
   pip install databricks-cli
   
   # Or use the newer unified CLI
   curl -fsSL https://raw.githubusercontent.com/databricks/setup-cli/main/install.sh | sh
   ```

2. **Authentication** configured
   ```bash
   # Configure using OAuth (recommended)
   databricks configure --token
   
   # Or set environment variables
   export DATABRICKS_HOST=https://dbc-1eff0d8e-9619.cloud.databricks.com
   export DATABRICKS_TOKEN=your_token_here
   ```

3. **SQL Warehouse ID** - Get this from your Databricks workspace
   - Go to SQL Warehouses in your workspace
   - For dev: Using warehouse ID `6d2e141d82a63d88`
   - For prod: Update to your production warehouse ID if different

4. **Catalog and Schema Access** - Ensure you have permissions to:
   - Read from `workspace` catalog
   - Access `abs_trade` and `default` schemas
   - Query the ABS trade tables
   - Create and run jobs (for quality monitoring)

## Bundle Structure

```
data_product/
├── databricks.yml              # Main bundle configuration
├── README.md                   # Bundle documentation
├── DEPLOYMENT.md               # This file
├── VERSION_MANAGEMENT.md       # Git version tracking guide
└── resources/
    ├── tables.yml              # ABS table references
    ├── dashboards.yml          # Dashboard configuration
    ├── data_contract.yml       # Data contract & quality rules
    └── jobs.yml                # Quality monitoring jobs
```

## Important: What This Bundle Deploys

**This bundle does NOT create or manage the source tables.** It assumes the following tables already exist:

- `workspace.abs_trade.metadata`
- `workspace.abs_trade.raw_json`
- `workspace.default.abs_trade_serv_state_cy_metadata`
- `workspace.default.abs_trade_serv_state_cy_raw_json`

The bundle deploys:
✅ Table reference documentation
✅ Dashboard configuration
✅ **Data contract with schema and quality rules**
✅ **Automated quality monitoring jobs**
✅ **Quality validation notebook**
✅ Metadata and tags
✅ **Email alerts for quality failures**
❌ Does NOT create or migrate tables
❌ Does NOT deploy actual dashboard widgets (references existing dashboard)

### Data Contract Features

The bundle includes a comprehensive data contract (`resources/data_contract.yml`) that defines:

* **Schema specifications** for all tables
* **Quality rules**: not_null, unique, valid_values, patterns, row counts
* **SLAs**: 99.9% availability, 24h freshness, 95% completeness
* **Governance**: data classification, ownership, breaking change policy
* **Monitoring**: Daily automated checks with email alerts

### Quality Monitoring

Two jobs are deployed:

1. **ABS Trade Data - Quality Checks** (Scheduled)
   - Runs daily at 2 AM UTC
   - Validates all quality rules from data contract
   - Sends email alerts on failures
   - Notebook: `/Users/melissaslattery@gmail.com/quality_checks`

2. **ABS Trade Data - Schema Validation** (On-demand)
   - Manual trigger only
   - Validates table schemas match contract
   - Quick validation without full data checks

## Configuration Steps

### 1. Verify Catalog and Table Access

Before deploying, verify you can access the required tables:

```bash
# Test table access using Databricks CLI or SQL
databricks sql query execute \
  --warehouse-id 6d2e141d82a63d88 \
  --query "SELECT COUNT(*) FROM workspace.abs_trade.metadata"
```

Or in a SQL notebook/query:
```sql
-- Verify all required tables exist
SHOW TABLES IN workspace.abs_trade;
SHOW TABLES IN workspace.default LIKE 'abs_trade%';

-- Test table access
SELECT * FROM workspace.abs_trade.metadata LIMIT 1;
SELECT * FROM workspace.default.abs_trade_serv_state_cy_metadata LIMIT 1;
```

### 2. Review Data Contract (Optional)

The data contract is pre-configured but you can customize it:

```bash
# View the data contract
cat /Workspace/Users/melissaslattery@gmail.com/dbx-test2/data_product/resources/data_contract.yml
```

Key sections to review:
* **SLAs**: Adjust availability, freshness, completeness targets
* **Quality rules**: Modify or add validation rules
* **Email notifications**: Update contact email addresses
* **Breaking change policy**: Adjust deprecation periods

### 3. Update Variables (Optional)

The bundle is pre-configured with warehouse ID `6d2e141d82a63d88`. To use a different warehouse, edit `databricks.yml`:

```yaml
variables:
  warehouse_id:
    default: "6d2e141d82a63d88"  # Change if needed

targets:
  dev:
    variables:
      warehouse_id: "6d2e141d82a63d88"  # Dev warehouse ID
  
  prod:
    variables:
      warehouse_id: "6d2e141d82a63d88"  # Update for production if different
```

**Finding Warehouse IDs:**
1. Go to SQL Warehouses in your Databricks workspace
2. Click on your warehouse
3. Copy the Warehouse ID from the URL or details page
4. Use the ID in databricks.yml

### 4. Validate the Bundle

```bash
# Navigate to the bundle directory
cd /Workspace/Users/melissaslattery@gmail.com/dbx-test2/data_product

# Validate the bundle configuration
databricks bundle validate

# Validate for a specific target
databricks bundle validate -t dev
databricks bundle validate -t prod
```

### 5. Deploy to Development

```bash
# Deploy to dev environment (default target)
databricks bundle deploy

# Or explicitly specify the dev target
databricks bundle deploy -t dev
```

This will:
- Validate bundle configuration
- Create bundle metadata in your workspace
- Deploy quality monitoring jobs
- Set up email notifications
- Document table references and dashboard configuration

### 6. Verify Quality Jobs

After deployment, verify the quality monitoring jobs:

```bash
# List deployed jobs
databricks jobs list | grep "ABS Trade"

# Get job details
databricks jobs get --job-id <job-id>
```

Or in the Databricks UI:
1. Go to Workflows → Jobs
2. Look for "ABS Trade Data - Quality Checks"
3. Verify schedule is set to daily at 2 AM UTC
4. Check email notification settings

### 7. Run Initial Quality Check

Run a manual quality check to verify everything works:

```bash
# Trigger quality check job
databricks jobs run-now --job-name "ABS Trade Data - Quality Checks"

# Or open the quality checks notebook and run it manually
```

### 8. Deploy to Production

```bash
# For version tracking (recommended)
git tag -a v1.0.0 -m "Production release 1.0.0 with data contract"
export GIT_TAG=$(git describe --tags --abbrev=0)

# Deploy to production environment
databricks bundle deploy -t prod
```

The production deployment will:
- Run as the specified service principal or user
- Deploy to the production root path
- Use production warehouse configuration
- Set up production quality monitoring

## Deployment Commands Reference

| Command | Description |
|---------|-------------|
| `databricks bundle validate` | Check bundle configuration for errors |
| `databricks bundle deploy` | Deploy the bundle to the workspace |
| `databricks bundle deploy -t prod` | Deploy to production target |
| `databricks bundle destroy` | Remove deployed bundle resources |
| `databricks bundle run` | Execute jobs defined in the bundle |
| `databricks jobs run-now --job-name "..."` | Trigger quality check job manually |

## Verifying Deployment

After deployment, verify in your Databricks workspace:

1. **Dashboard**: The existing dashboard (ID: 01f1321d5f29145c8dbff9df0ddf464c) should still be accessible
2. **Tables**: Verify table references are accessible:
   ```sql
   SELECT * FROM workspace.abs_trade.metadata;
   SELECT * FROM workspace.default.abs_trade_serv_state_cy_metadata;
   ```
3. **Permissions**: Confirm appropriate access controls
4. **Bundle Resources**: Check deployed bundle in your workspace
5. **Quality Jobs**: Verify jobs are scheduled and configured
   - Go to Workflows → Jobs
   - Check "ABS Trade Data - Quality Checks"
   - Verify schedule and notifications
6. **Quality Notebook**: Open `/Users/melissaslattery@gmail.com/quality_checks` and verify it has content
7. **Run Test**: Manually trigger quality check job to test

## Data Contract Enforcement

### Understanding the Data Contract

The data contract (`resources/data_contract.yml`) defines:

```yaml
# Example quality rules from contract
tables:
  - name: "workspace.abs_trade.metadata"
    schema:
      columns:
        - name: "id"
          type: "STRING"
          nullable: false
          quality_rules:
            - type: "not_null"
              severity: "error"
            - type: "unique"
              severity: "error"
```

### Modifying Quality Rules

To add or modify quality rules:

1. Edit `resources/data_contract.yml`
2. Add quality rules under the relevant column
3. Set severity: `error` (blocks) or `warning` (alerts only)
4. Redeploy: `databricks bundle deploy`
5. Quality checks will enforce new rules on next run

### Severity Levels

* **error**: Validation failure blocks downstream processes, sends immediate alert
* **warning**: Validation failure logs warning, sends alert but doesn't block

## Troubleshooting

### Common Issues

#### Issue: Catalog or Schema Not Found

**Error**: `[SCHEMA_NOT_FOUND] The schema 'workspace.abs_trade' cannot be found`

**Solutions**:
1. Verify the catalog and schema exist:
   ```sql
   SHOW CATALOGS;
   SHOW SCHEMAS IN workspace;
   ```

2. Ensure you have USE CATALOG permission:
   ```sql
   USE CATALOG workspace;
   ```

3. Check your permissions:
   ```sql
   SHOW GRANTS ON CATALOG workspace;
   SHOW GRANTS ON SCHEMA workspace.abs_trade;
   ```

#### Issue: Table Not Found

**Error**: `[TABLE_OR_VIEW_NOT_FOUND] The table or view 'workspace.abs_trade.metadata' cannot be found`

**Solutions**:
1. List all tables in the schema:
   ```sql
   SHOW TABLES IN workspace.abs_trade;
   SHOW TABLES IN workspace.default;
   ```

2. Verify the exact table names (case-sensitive):
   ```sql
   DESCRIBE TABLE workspace.abs_trade.metadata;
   ```

3. Check if tables need to be created first (see Data Setup section below)

#### Issue: Quality Check Job Failed

**Error**: Quality monitoring job fails or reports errors

**Solutions**:
1. Check job run history in Workflows UI
2. Review error messages in job logs
3. Open quality checks notebook and run manually to debug
4. Verify warehouse permissions
5. Check data contract YAML syntax is valid
6. Ensure all referenced tables exist and are accessible

#### Issue: Email Alerts Not Received

**Error**: Quality failures occur but no email sent

**Solutions**:
1. Verify email address in `resources/jobs.yml`
2. Check Databricks workspace email settings
3. Verify job notification settings in UI
4. Check spam folder
5. Test email notifications:
   ```bash
   databricks jobs run-now --job-name "ABS Trade Data - Schema Validation"
   ```

#### Issue: `warehouse_id` not found

**Error**: Variable interpolation failed for `${var.warehouse_id}`

**Solution**: Update the warehouse_id in databricks.yml with an actual warehouse ID from your workspace. The default is set to `6d2e141d82a63d88`. To find other warehouse IDs:
1. Navigate to SQL Warehouses
2. Select your warehouse
3. Copy the warehouse ID from the details or URL

#### Issue: Permission denied

**Error**: `PERMISSION_DENIED: User does not have access`

**Solutions**:
1. Ensure your authentication token has sufficient permissions
2. Request access to the required catalogs and schemas
3. For production deployments, verify the `run_as` user has necessary permissions
4. Check permissions to create and run jobs

#### Issue: Dashboard Not Deploying

**Note**: The current bundle configuration documents an existing dashboard but does not create new dashboards. To work with the dashboard:

1. Access the existing dashboard directly: Dashboard ID `01f1321d5f29145c8dbff9df0ddf464c`
2. Or create new dashboards manually and reference them in the bundle

### Data Setup

If the source tables don't exist yet, you need to create them first. This bundle assumes they already exist. To set up the tables:

```sql
-- Create catalog and schema if needed
CREATE CATALOG IF NOT EXISTS workspace;
CREATE SCHEMA IF NOT EXISTS workspace.abs_trade;

-- Create tables (example structure)
CREATE TABLE IF NOT EXISTS workspace.abs_trade.metadata (
  agency_id STRING,
  description STRING,
  id STRING,
  is_final BOOLEAN,
  name STRING,
  structure_urn STRING,
  urn STRING,
  version STRING
);

CREATE TABLE IF NOT EXISTS workspace.abs_trade.raw_json (
  raw_json STRING
);

-- Similar setup for default schema tables
CREATE TABLE IF NOT EXISTS workspace.default.abs_trade_serv_state_cy_metadata (
  agency_id STRING,
  description STRING,
  id STRING,
  is_final BOOLEAN,
  name STRING,
  structure_urn STRING,
  urn STRING,
  version STRING
);

CREATE TABLE IF NOT EXISTS workspace.default.abs_trade_serv_state_cy_raw_json (
  raw_json STRING
);
```

## Updating the Bundle

To update after making changes:

```bash
# 1. Validate changes
databricks bundle validate

# 2. Deploy updates to dev
databricks bundle deploy -t dev

# 3. Test quality checks
databricks jobs run-now --job-name "ABS Trade Data - Quality Checks"

# 4. After testing, tag and deploy to prod
git tag -a v1.1.0 -m "Update: description"
export GIT_TAG=$(git describe --tags)
databricks bundle deploy -t prod
```

See [VERSION_MANAGEMENT.md](VERSION_MANAGEMENT.md) for version control best practices.

### Updating the Data Contract

When modifying the data contract:

1. Edit `resources/data_contract.yml`
2. Document changes in git commit
3. Deploy to dev and test
4. If breaking changes, follow breaking change policy (90-day notice)
5. Deploy to prod after validation

## Cleanup

To remove the deployed bundle:

```bash
# Remove dev deployment
databricks bundle destroy -t dev

# Remove prod deployment
databricks bundle destroy -t prod
```

**Note**: This removes:
- Bundle metadata
- Quality monitoring jobs
- Job schedules and notifications

**Does NOT delete**:
- Source tables
- Dashboard
- Quality checks notebook
- Historical job run logs

## Additional Resources

- [Databricks Asset Bundles Documentation](https://docs.databricks.com/en/dev-tools/bundles/)
- [Bundle Configuration Reference](https://docs.databricks.com/en/dev-tools/bundles/settings.html)
- [Bundle Permissions Guide](https://docs.databricks.com/dev-tools/bundles/permissions.html)
- [Unity Catalog Documentation](https://docs.databricks.com/en/data-governance/unity-catalog/)
- [Databricks Jobs](https://docs.databricks.com/en/workflows/jobs/)
- [Data Quality Monitoring](https://docs.databricks.com/en/lakehouse-monitoring/)
- [VERSION_MANAGEMENT.md](VERSION_MANAGEMENT.md) - Git version tracking guide
