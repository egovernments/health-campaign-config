# EGOV Health Campaign Configuration

This repository contains configuration files for eGov services used in health campaign management systems.

## Important: Branch Information
- **Working Branch**: `DEMO` - All new tenant configurations should be added to this branch
- **Main Branch**: `master` - Production configurations

## Repository Structure

```
health-campaign-config/
├── egov-indexer/       # Elasticsearch indexer configurations
│   ├── chad/          # Chad tenant configurations
│   ├── chaduat/       # Chad UAT tenant configurations
│   └── congob/        # Congo tenant configurations
├── egov-persister/     # Database persister configurations
│   ├── chad/          # Chad tenant configurations
│   ├── chaduat/       # Chad UAT tenant configurations
│   └── congob/        # Congo tenant configurations
└── docs/              # Documentation
    └── NEW_TENANT_CONFIGURATION.md  # Guide for adding new tenants
```

## Quick Start

### Adding a New Tenant
For detailed instructions on adding configurations for a new tenant, see [New Tenant Configuration Guide](docs/NEW_TENANT_CONFIGURATION.md).

### Existing Tenants
- **chad** - Production configuration for Chad
- **chaduat** - UAT environment for Chad
- **congob** - Configuration for Congo

## Services Configured

### Core Services
- **Facility Management** - Location and facility data
- **Household Management** - Household registration and tracking
- **Individual Management** - Beneficiary registration
- **Product Management** - Inventory and product tracking
- **Project Management** - Campaign project configuration
- **Stock Management** - Inventory transactions

### Supporting Services
- Attendance tracking
- Census data collection
- Public grievance redressal (PGR)
- Referral management
- Workflow management
- Master data management (MDMS)

## Configuration Components

### eGov-Indexer
Handles indexing of data into Elasticsearch for search capabilities. Each service has an indexer configuration that:
- Listens to Kafka topics for data changes
- Transforms and indexes data into Elasticsearch
- Maintains search indices for quick data retrieval

### eGov-Persister
Manages database persistence operations. Each service has a persister configuration that:
- Listens to Kafka topics for data events
- Executes database operations (INSERT/UPDATE)
- Maintains audit trails
- Handles transactional integrity

## Documentation

- [New Tenant Configuration Guide](docs/NEW_TENANT_CONFIGURATION.md) - Step-by-step guide for adding new tenants
- [Service Configuration Reference](docs/NEW_TENANT_CONFIGURATION.md#appendix-a-service-configuration-reference) - Detailed service configuration mappings

## Contributing

When adding or modifying configurations:
1. Follow the naming conventions documented in the configuration guide
2. Test configurations in a development environment first
3. Ensure all related services are configured (both indexer and persister)
4. Update documentation if adding new services

## Support

For issues or questions regarding configurations, please refer to the documentation in the `docs/` folder or contact the platform team.