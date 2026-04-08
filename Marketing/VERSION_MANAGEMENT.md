# Git Version Management for Bundle Deployments

This guide explains how to ensure your Databricks Asset Bundle deployments reflect git versions.

## Overview

Version tracking ensures you can:
* Know exactly which code version is deployed
* Roll back to previous versions if needed
* Track changes and audit deployments
* Comply with governance requirements

## Quick Start

```bash
# 1. Tag your release
git tag -a v1.0.0 -m "Release version 1.0.0"
git push origin v1.0.0

# 2. Deploy with version
export GIT_TAG=$(git describe --tags --abbrev=0)
export GIT_COMMIT=$(git rev-parse --short HEAD)
databricks bundle deploy -t prod
```

## Implementation Options

### Option 1: Environment Variables (Simplest)

Use environment variables during deployment:

```bash
# Set git version variables
export GIT_TAG=$(git describe --tags --always)
export GIT_COMMIT=$(git rev-parse HEAD)
export GIT_SHORT_COMMIT=$(git rev-parse --short HEAD)
export DEPLOY_TIMESTAMP=$(date -u +"%Y-%m-%dT%H:%M:%SZ")

# Deploy
databricks bundle deploy -t prod
```

### Option 2: Bundle Configuration Variables

Update `databricks.yml` to include version variables:

```yaml
bundle:
  name: "data_product"

variables:
  warehouse_id:
    description: "SQL Warehouse ID for dashboard queries"
    default: "your_warehouse_id_here"
  
  bundle_version:
    description: "Bundle version from git tag"
    default: "dev"
  
  git_commit:
    description: "Git commit SHA"
    default: "unknown"

targets:
  dev:
    mode: development
    default: true
    workspace:
      host: https://dbc-1eff0d8e-9619.cloud.databricks.com
    variables:
      warehouse_id: "your_dev_warehouse_id"
      bundle_version: "dev-${GIT_COMMIT}"
      git_commit: "${GIT_COMMIT}"
  
  prod:
    mode: production
    workspace:
      host: https://dbc-1eff0d8e-9619.cloud.databricks.com
      root_path: /Users/melissaslattery@gmail.com/.bundle/${bundle.name}/${bundle.target}
    run_as:
      user_name: melissaslattery@gmail.com
    variables:
      warehouse_id: "your_prod_warehouse_id"
      bundle_version: "${GIT_TAG}"
      git_commit: "${GIT_COMMIT}"
```

Then deploy with:

```bash
export GIT_TAG=$(git describe --tags --abbrev=0)
export GIT_COMMIT=$(git rev-parse HEAD)
databricks bundle deploy -t prod
```

### Option 3: Resource Tagging

Add version information to resource tags in `resources/dashboards.yml`:

```yaml
resources:
  dashboards:
    abs_trade_dashboard:
      display_name: "ABS Trade Data Analysis"
      warehouse_id: "${var.warehouse_id}"
      tags:
        - key: "source"
          value: "abs"
        - key: "domain"
          value: "trade"
        - key: "bundle_version"
          value: "${var.bundle_version}"
        - key: "git_commit"
          value: "${var.git_commit}"
        - key: "deployed_at"
          value: "${DEPLOY_TIMESTAMP}"
```

### Option 4: Automated Deployment Script

Create `deploy.sh` in your bundle directory for consistent deployments with version tracking.

## Semantic Versioning

Follow semantic versioning (MAJOR.MINOR.PATCH):

* **MAJOR** (v2.0.0): Breaking changes
* **MINOR** (v1.1.0): New features (backward compatible)
* **PATCH** (v1.0.1): Bug fixes

### Examples

```bash
# Initial release
git tag -a v1.0.0 -m "Initial production release"

# Bug fix
git tag -a v1.0.1 -m "Fix dashboard query timeout"

# New feature
git tag -a v1.1.0 -m "Add state comparison widgets"

# Breaking change
git tag -a v2.0.0 -m "Migrate to new table schema"

# Push tags
git push origin --tags
```

## Git Tagging Best Practices

### Creating Tags

```bash
# Annotated tag (recommended - includes metadata)
git tag -a v1.0.0 -m "Release version 1.0.0: Initial ABS dashboard"

# Tag a specific commit
git tag -a v1.0.0 <commit-sha> -m "Release description"

# Push tag to remote
git push origin v1.0.0

# Push all tags
git push origin --tags
```

### Viewing Tags

```bash
# List all tags
git tag

# Show tag details
git show v1.0.0

# List tags matching pattern
git tag -l "v1.*"

# Get current version
git describe --tags
git describe --tags --abbrev=0  # Just the tag name
```

### Deleting Tags

```bash
# Delete local tag
git tag -d v1.0.0

# Delete remote tag
git push origin --delete v1.0.0
```

## Deployment Workflow

### Development Workflow

```bash
# 1. Make changes
git add .
git commit -m "feat: Add new dashboard widget"

# 2. Deploy to dev (uses commit SHA)
export GIT_COMMIT=$(git rev-parse --short HEAD)
databricks bundle deploy -t dev

# 3. Test in dev environment
```

### Production Workflow

```bash
# 1. Create release tag
git tag -a v1.0.0 -m "Production release 1.0.0"
git push origin v1.0.0

# 2. Deploy to prod (uses tag)
export GIT_TAG=$(git describe --tags --abbrev=0)
export GIT_COMMIT=$(git rev-parse HEAD)
databricks bundle deploy -t prod

# 3. Verify deployment
databricks dashboards list | grep "ABS Trade"
```

## CI/CD Integration

### Key Environment Variables

Export these in your CI/CD pipeline:

```bash
export GIT_TAG=$(git describe --tags --always)
export GIT_COMMIT=$(git rev-parse HEAD)
export GIT_SHORT_COMMIT=$(git rev-parse --short HEAD)
export GIT_BRANCH=$(git rev-parse --abbrev-ref HEAD)
export DEPLOY_TIMESTAMP=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
```

### GitHub Actions Snippet

```yaml
- name: Get version info
  id: version
  run: |
    echo "git_tag=$(git describe --tags --always)" >> $GITHUB_OUTPUT
    echo "git_commit=$(git rev-parse HEAD)" >> $GITHUB_OUTPUT

- name: Deploy to Prod
  if: startsWith(github.ref, 'refs/tags/v')
  env:
    GIT_TAG: ${{ steps.version.outputs.git_tag }}
    GIT_COMMIT: ${{ steps.version.outputs.git_commit }}
  run: databricks bundle deploy -t prod
```

## Verification

After deployment, verify version information:

```bash
# Check dashboard tags (if tagged)
databricks dashboards get <dashboard-id> --output json | \
  jq '.tags[] | select(.key | contains("version"))'
```

## Rollback Procedure

If you need to rollback to a previous version:

```bash
# 1. Find the version to rollback to
git tag -l

# 2. Checkout the tag
git checkout v1.0.0

# 3. Deploy
export GIT_TAG=v1.0.0
export GIT_COMMIT=$(git rev-parse HEAD)
databricks bundle deploy -t prod

# 4. Return to main branch
git checkout main
```

## Troubleshooting

### Version Variables Not Interpolated

**Problem**: `${GIT_TAG}` appears literally in deployed resources

**Solution**: Ensure environment variables are exported before deployment

```bash
export GIT_TAG=$(git describe --tags)
export GIT_COMMIT=$(git rev-parse HEAD)
databricks bundle deploy -t prod
```

### No Git Tags Found

**Problem**: `git describe --tags` returns an error

**Solution**: Create your first tag

```bash
git tag -a v1.0.0 -m "Initial release"
git push origin v1.0.0
```

### Production Deploy Requires Tag

**Problem**: Deploy script enforces tagged releases for production

**Solution**: Tag your release before deploying

```bash
git tag -a v1.1.0 -m "New features"
git push origin v1.1.0
databricks bundle deploy -t prod
```

## Best Practices Summary

1. ✅ **Always tag production releases** with semantic versions
2. ✅ **Include version info** in bundle variables and resource tags
3. ✅ **Automate deployments** with scripts or CI/CD
4. ✅ **Track deployment history** in logs
5. ✅ **Document changes** in git commit messages
6. ✅ **Verify deployments** by checking deployed version tags
7. ✅ **Test in dev** before promoting to production
8. ✅ **Keep deployment scripts** in version control

## Additional Resources

* [Semantic Versioning](https://semver.org/)
* [Git Tagging Documentation](https://git-scm.com/book/en/v2/Git-Basics-Tagging)
* [Databricks Asset Bundles](https://docs.databricks.com/en/dev-tools/bundles/)
* [Conventional Commits](https://www.conventionalcommits.org/)
