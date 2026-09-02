
 Transactions
Model configuration
Database connection


Database Configuration
appsettings.json
appsettings.Development.json
appsettings.Production.json
        ↓
Configuration
        ↓
DI
        ↓
DbContext

 DbContext configuration
    
- Dependency Injection
    
- Environment-specific configuration

Then learn migrations:
Model change
     ↓
Migration
     ↓
Migration SQL
     ↓
Database schema change




Learn:

- Primary key conventions
    
- Foreign key conventions
    
- Table naming
    
- Column naming
    
- Required/optional properties
    
- String length
    
- Relationships
    
- Shadow properties


Data Annotations


Change Tracking & context.Entry(entity).State   , Explicit Loading


Transaction
 ├── Atomicity
 ├── Consistency
 ├── Isolation
 └── Durability

- Commit
    
- Rollback
    
- Isolation levels
    
- Ambient transactions
    
- Multiple operations
    
- Transaction boundaries

Concurrency  Learn concurrency tokens: