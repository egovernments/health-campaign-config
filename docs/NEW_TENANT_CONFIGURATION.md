# New Tenant Configuration Guide for eGov Services

## Overview
This guide provides step-by-step instructions for engineers to configure eGov-indexer and eGov-persister services for a new tenant in the health campaign system.

## Prerequisites
- Tenant code/identifier (e.g., `chad`, `chaduat`, `congob`)
- Database schema name for the tenant
- Understanding of the services to be configured
- Ensure you're working on the `DEMO` branch (`git checkout DEMO`)

## Directory Structure

```
health-campaign-config/
├── egov-indexer/
│   └── <tenant-name>/
│       ├── facility-indexer.yml
│       ├── household-indexer.yml
│       ├── individual-indexer.yml
│       ├── product-indexer.yml
│       ├── project-indexer.yml
│       ├── service-request-indexer.yml
│       └── ... (other service indexers)
└── egov-persister/
    └── <tenant-name>/
        ├── facility-persister.yml
        ├── household-persister.yml
        ├── individual-persister.yml
        ├── product-persister.yml
        ├── project-persister.yml
        ├── service-request-persister.yml
        └── ... (other service persisters)
```

## Step-by-Step Configuration Process

### Step 1: Create Tenant Directories

1. Create a new directory for your tenant in both `egov-indexer` and `egov-persister`:
   ```bash
   mkdir -p egov-indexer/<tenant-name>
   mkdir -p egov-persister/<tenant-name>
   ```

### Step 2: Configure eGov-Indexer

For each service that needs indexing, create a corresponding indexer configuration file.

#### 2.1 Indexer Configuration Template

Create files following this naming pattern: `<service-name>-indexer.yml`

**Key Configuration Elements:**
- **Topic Names**: Must include tenant prefix (e.g., `<tenant-name>-save-facility-topic`)
- **Index Names**: Must include tenant prefix (e.g., `<tenant-name>-facility-index-v1`)
- **Service Name**: Keep consistent across tenants (e.g., `facility`)

**Example: facility-indexer.yml**
```yaml
ServiceMaps:
  serviceName: facility
  version: 1.0.0
  mappings:
  - topic: <tenant-name>-save-facility-topic
    configKey: INDEX
    indexes:
    - name: <tenant-name>-facility-index-v1
      type: facility
      id: $.clientReferenceId
      isBulk: true
      jsonPath: $.*
      timeStampField: $.auditDetails.createdTime
  - topic: <tenant-name>-update-facility-topic
    configKey: INDEX
    indexes:
    - name: <tenant-name>-facility-index-v1
      type: facility
      id: $.clientReferenceId
      isBulk: true
      jsonPath: $.*
      timeStampField: $.auditDetails.lastModifiedTime
```

### Step 3: Configure eGov-Persister

For each service that needs persistence, create a corresponding persister configuration file.

#### 3.1 Persister Configuration Template

Create files following this naming pattern: `<service-name>-persister.yml`

**Key Configuration Elements:**
- **Topic Names**: Must match the indexer topics (e.g., `<tenant-name>-save-facility-topic`)
- **Database Schema**: Replace with tenant schema name (e.g., `INSERT INTO <tenant-name>.FACILITY`)
- **Query Maps**: Define INSERT and UPDATE queries with proper schema

**Example: facility-persister.yml**
```yaml
serviceMaps:
  serviceName: facility
  mappings:
  - version: 1.0
    description: Saves a facility
    fromTopic: <tenant-name>-save-facility-topic
    isTransaction: true
    isAuditEnabled: true
    module: FACILITY
    objecIdJsonPath: $.id
    tenantIdJsonPath: $.tenantId
    transactionCodeJsonPath: $.clientReferenceId
    auditAttributeBasePath: $.*
    queryMaps:
    - query: INSERT INTO <tenant-name>.FACILITY(id, clientReferenceId, tenantId, isPermanent,
        name, usage, storageCapacity, addressId, additionalDetails, createdBy, createdTime,
        lastModifiedBy, lastModifiedTime, rowVersion, isDeleted) VALUES (?,?,?,?,?,?,?,?,?,?,?,?,?,?,?);
      basePath: $.*
      jsonMaps:
      - jsonPath: $.*.id
      - jsonPath: $.*.clientReferenceId
      - jsonPath: $.*.tenantId
      - jsonPath: $.*.isPermanent
      - jsonPath: $.*.name
      - jsonPath: $.*.usage
      - jsonPath: $.*.storageCapacity
      - jsonPath: $.*.address.id
      - jsonPath: $.*.additionalFields
        type: JSON
        dbType: JSONB
      - jsonPath: $.*.auditDetails.createdBy
      - jsonPath: $.*.auditDetails.createdTime
      - jsonPath: $.*.auditDetails.lastModifiedBy
      - jsonPath: $.*.auditDetails.lastModifiedTime
      - jsonPath: $.*.rowVersion
      - jsonPath: $.*.isDeleted
```

### Step 4: Common Services to Configure

The following services typically need both indexer and persister configurations:

#### Core Services:
1. **facility** - Manages facility/location data
2. **household** - Manages household information
3. **individual** - Manages individual/person data
4. **product** - Manages product/inventory information
5. **project** - Manages project configurations
6. **project-task** - Manages project tasks
7. **project-staff** - Manages project staff assignments
8. **service-request** - Manages service requests
9. **stock** - Manages stock/inventory transactions

#### Supporting Services:
1. **attendance-service** - Employee attendance tracking
2. **census-service** - Census data management
3. **pgr-services** - Public grievance redressal
4. **referral-management** - Referral tracking
5. **plan-service** - Planning module
6. **audit-service** - Audit trail management
7. **boundary-management** - Geographic boundary definitions
8. **hrms-employee** - Employee management
9. **egov-workflow-v2** - Workflow management
10. **mdms** - Master data management

### Step 5: Configuration Checklist

For each service configuration, ensure:

#### Indexer Configuration:
- [ ] Replace all topic names with `<tenant-name>-<action>-<service>-topic` format
- [ ] Replace all index names with `<tenant-name>-<service>-index-v1` format
- [ ] Verify `serviceName` matches the service being configured
- [ ] Ensure `jsonPath` and `id` fields are correctly mapped
- [ ] Set appropriate `timeStampField` for create and update operations

#### Persister Configuration:
- [ ] Topic names match the corresponding indexer configuration
- [ ] Database schema name is replaced with tenant-specific schema
- [ ] All table names include the tenant schema prefix
- [ ] JSON paths correctly map to database columns
- [ ] Audit fields are properly configured
- [ ] Transaction settings are appropriate for the service

### Step 6: Testing the Configuration

1. **Validate YAML Syntax:**
   ```bash
   # Install yamllint if not available
   pip install yamllint

   # Validate all YAML files
   yamllint egov-indexer/<tenant-name>/*.yml
   yamllint egov-persister/<tenant-name>/*.yml
   ```

2. **Verify Topic Consistency:**
   Ensure that topics in indexer match those in persister:
   ```bash
   # List all topics in indexer configs
   grep -h "topic:" egov-indexer/<tenant-name>/*.yml | sort | uniq

   # List all topics in persister configs
   grep -h "fromTopic:" egov-persister/<tenant-name>/*.yml | sort | uniq
   ```

3. **Check Schema References:**
   Verify all database operations use the correct tenant schema:
   ```bash
   # Check INSERT statements
   grep -h "INSERT INTO" egov-persister/<tenant-name>/*.yml | head -5

   # Check UPDATE statements
   grep -h "UPDATE" egov-persister/<tenant-name>/*.yml | head -5
   ```

### Step 7: Common Patterns and Best Practices

#### Naming Conventions:
- **Topics:** `<tenant-name>-<action>-<service>-topic`
  - Actions: `save`, `update`, `delete`
  - Example: `newtenant-save-facility-topic`

- **Indexes:** `<tenant-name>-<service>-index-v1`
  - Example: `newtenant-facility-index-v1`

- **Database Schema:** `<tenant-name>.<TABLE_NAME>`
  - Example: `newtenant.FACILITY`

#### JSON Path Mappings:
- Always use `$.*` for bulk operations
- Nested objects: `$.*.address.id`
- Arrays: `$.*.items[*].id`
- Conditional fields: Mark as optional in persister

#### Audit Fields:
Standard audit fields that should be present in all configurations:
- `createdBy`
- `createdTime`
- `lastModifiedBy`
- `lastModifiedTime`

### Step 8: Deployment Considerations

1. **Environment-Specific Configurations:**
   - Development: May use simplified configs
   - UAT: Should mirror production closely
   - Production: Full configuration with all validations

2. **Database Prerequisites:**
   - Ensure database schema exists for the tenant
   - Required tables are created with proper structure
   - Indexes are created for performance

3. **Kafka Topics:**
   - Create all required Kafka topics before deployment
   - Set appropriate retention policies
   - Configure partitions based on expected load

### Troubleshooting Guide

#### Common Issues and Solutions:

1. **Topic Mismatch Error:**
   - Symptom: Messages not being processed
   - Solution: Verify topic names match between producer, indexer, and persister

2. **Schema Not Found:**
   - Symptom: SQL errors in persister logs
   - Solution: Ensure database schema exists and user has permissions

3. **JSON Path Resolution Failure:**
   - Symptom: Null values in database despite data in message
   - Solution: Verify JSON paths match actual message structure

4. **Index Creation Failure:**
   - Symptom: Search not returning results
   - Solution: Check Elasticsearch index naming and mapping

### Example: Adding a New Tenant "newtenant"

```bash
# 0. Ensure you're on the DEMO branch
git checkout DEMO

# 1. Create directories
mkdir -p egov-indexer/newtenant
mkdir -p egov-persister/newtenant

# 2. Copy template configurations from existing tenant
cp -r egov-indexer/chaduat/*.yml egov-indexer/newtenant/
cp -r egov-persister/chaduat/*.yml egov-persister/newtenant/

# 3. Replace tenant-specific values
# For Linux/Mac:
find egov-indexer/newtenant -name "*.yml" -exec sed -i '' 's/chaduat/newtenant/g' {} \;
find egov-persister/newtenant -name "*.yml" -exec sed -i '' 's/chaduat/newtenant/g' {} \;

# 4. Review and adjust configurations as needed

# 5. Commit and push to DEMO branch
git add egov-indexer/newtenant egov-persister/newtenant
git commit -m "Add configuration for newtenant"
git push origin DEMO
```

### Appendix A: Service Configuration Reference

| Service | Indexer Topics | Persister Tables |
|---------|---------------|------------------|
| facility | save-facility-topic, update-facility-topic | FACILITY, ADDRESS |
| household | save-household-topic, update-household-topic | HOUSEHOLD, HOUSEHOLD_MEMBER, ADDRESS |
| individual | save-individual-topic, update-individual-topic | INDIVIDUAL, INDIVIDUAL_IDENTIFIER, ADDRESS |
| product | save-product-topic, update-product-topic | PRODUCT, PRODUCT_VARIANT |
| project | save-project-topic, update-project-topic | PROJECT, PROJECT_TARGET, PROJECT_RESOURCE |
| stock | save-stock-topic, update-stock-topic | STOCK, STOCK_TRANSACTION |

### Appendix B: Validation Scripts

Create a validation script `validate-tenant-config.sh`:

```bash
#!/bin/bash

TENANT_NAME=$1

if [ -z "$TENANT_NAME" ]; then
    echo "Usage: ./validate-tenant-config.sh <tenant-name>"
    exit 1
fi

echo "Validating configuration for tenant: $TENANT_NAME"

# Check directories exist
if [ ! -d "egov-indexer/$TENANT_NAME" ]; then
    echo "ERROR: egov-indexer/$TENANT_NAME directory not found"
    exit 1
fi

if [ ! -d "egov-persister/$TENANT_NAME" ]; then
    echo "ERROR: egov-persister/$TENANT_NAME directory not found"
    exit 1
fi

# Validate YAML syntax
echo "Checking YAML syntax..."
for file in egov-indexer/$TENANT_NAME/*.yml egov-persister/$TENANT_NAME/*.yml; do
    if ! python -c "import yaml; yaml.safe_load(open('$file'))" 2>/dev/null; then
        echo "ERROR: Invalid YAML in $file"
        exit 1
    fi
done

# Check tenant name consistency
echo "Checking tenant name consistency..."
INDEXER_REFS=$(grep -h "$TENANT_NAME" egov-indexer/$TENANT_NAME/*.yml | wc -l)
PERSISTER_REFS=$(grep -h "$TENANT_NAME" egov-persister/$TENANT_NAME/*.yml | wc -l)

echo "Found $INDEXER_REFS references in indexer configs"
echo "Found $PERSISTER_REFS references in persister configs"

echo "Validation complete!"
```

### Support and Resources

- **Documentation:** Refer to eGov platform documentation
- **Templates:** Use existing tenant configurations as templates
- **Testing:** Always test in a development environment first
- **Version Control:** Commit configurations with descriptive messages

---

*Last Updated: [Current Date]*
*Version: 1.0*