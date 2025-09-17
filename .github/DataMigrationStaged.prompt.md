# Data Migration Template - UV SQL Tool MCP

## Overview
This template provides a standardized approach for migrating data from legacy systems to Dynamics 365 Finance & Operations (F&O) using the UV SQL Tool MCP. Follow this template for consistent, repeatable migrations.

## Template Variables
Replace the following placeholders with your specific migration details:

- `{CSV_FILE_NAME}`: Name of the source CSV file
- `{DICTIONARY_FILE_NAME}`: Name of the data dictionary Excel file
- `{SOURCE_TABLE_NAME}`: Name of the source table (usually src{EntityName})
- `{STAGING_TABLE_NAME}`: Name of the staging table (usually stg{EntityName})
- `{REFERENCE_SP_PATH}`: Path to reference stored procedure for template

## Prerequisites Checklist

### Required Files
- [ ] Source CSV file: `DataSamples/{CSV_FILE_NAME}.csv`
- [ ] Data dictionary: `Mapping/{DICTIONARY_FILE_NAME}.xlsx`
- [ ] Reference stored procedure (if specific formatting needed)

### Required Tools
- [ ] UV SQL Tool MCP configured and available
- [ ] Access to D365 F&O migration database
- [ ] VS Code with appropriate extensions

### Environment Setup
- [ ] Database connection established
- [ ] Workspace folder structure in place
- [ ] Permissions verified for file operations

## Migration Process

### Step 1: Analyze Source Data
Before starting the migration, examine the source data structure:

```
1. Open the CSV file: DataSamples/{CSV_FILE_NAME}.csv
2. Review the first 10-20 rows to understand:
   - Column names and data types
   - Data patterns and formats
   - Null values or special characters
   - Delimiter types (comma, pipe, tab)
3. Document any data quality issues
```

### Step 2: Create Source Table
Use the UV SQL Tool to create the source table from CSV:

**Command:**
```
Tool: mcp_uv_sql_tool_m_create_table
Parameters:
- csv_file_path: "c:\D365DataMigration\DataSamples\{CSV_FILE_NAME}.csv"
- table_name: "{SOURCE_TABLE_NAME}"
```

**Expected Output:**
- Creates `Tables/{SOURCE_TABLE_NAME}.sql`
- Table structure matches CSV columns
- Appropriate SQL Server data types assigned

### Step 3: Create Staging Stored Procedure
Use the UV SQL Tool to create the transformation stored procedure:

**Command:**
```
Tool: mcp_uv_sql_tool_m_create_stored_procedure
Parameters:
- dictionary_path: "c:\D365DataMigration\Mapping\{DICTIONARY_FILE_NAME}.xlsx"
- reference_sp_path: "c:\D365DataMigration\stored_procedures\{REFERENCE_SP_PATH}"
- table_name: "{SOURCE_TABLE_NAME}"
```

**Expected Output:**
- Creates/Updates `stored_procedures/{STAGING_TABLE_NAME}_create.sql`
- Follows UV SQL Tool conventions
- Includes column mapping transformations

## UV SQL Tool Conventions

The UV SQL Tool follows specific patterns that differ from project coding standards:

### Stored Procedure Format
```sql
ALTER PROCEDURE sp_{STAGING_TABLE_NAME}
AS
BEGIN
    SET NOCOUNT ON;
    IF OBJECT_ID('dbo.{STAGING_TABLE_NAME}', 'U') IS NOT NULL
        DROP TABLE dbo.{STAGING_TABLE_NAME};

    CREATE TABLE dbo.{STAGING_TABLE_NAME} (
        [COLUMN1] NVARCHAR(MAX),
        [COLUMN2] NVARCHAR(MAX),
        -- ... additional columns
        [DATAAREAID] NVARCHAR(4)
    );

    INSERT INTO dbo.{STAGING_TABLE_NAME} (
        [COLUMN1], [COLUMN2], ..., [DATAAREAID]
    )
    SELECT
        [source_column1] AS [COLUMN1],
        [source_column2] AS [COLUMN2],
        -- ... additional mappings
        'USMF' AS [DATAAREAID]
    FROM dbo.{SOURCE_TABLE_NAME};
END
```

### Key Characteristics
- Uses `ALTER PROCEDURE` instead of `CREATE PROCEDURE`
- Implements `IF OBJECT_ID()` for table existence checks
- Creates staging table with explicit `CREATE TABLE`
- Uses `INSERT INTO ... SELECT` pattern
- Column names converted to UPPERCASE
- Uses `NVARCHAR(MAX)` for most text fields
- Adds `DATAAREAID` field when appropriate

## Column Naming Conventions

### Standard Transformations
Common source to target column mappings:

| Source Pattern | Target Pattern | Example |
|---------------|----------------|---------|
| snake_case | UPPERCASE | customer_id → CUSTOMERID |
| camelCase | UPPERCASE | customerId → CUSTOMERID |
| Mixed Case | UPPERCASE | Customer_ID → CUSTOMERID |
| Abbreviated | EXPANDED | cust_id → CUSTOMERID |

### D365 F&O Standard Fields
Always include these standard fields when applicable:

- `DATAAREAID` (NVARCHAR(4)) - Company identifier, usually 'USMF'
- `RECID` (BIGINT) - Record identifier (if needed)
- `CREATEDBY` (NVARCHAR(20)) - Created by user
- `CREATEDDATETIME` (DATETIME) - Creation timestamp
- `MODIFIEDBY` (NVARCHAR(20)) - Modified by user
- `MODIFIEDDATETIME` (DATETIME) - Modification timestamp

## Data Validation

### Pre-Migration Checks
```sql
-- 1. Verify source data load
SELECT COUNT(*) as SourceRecordCount FROM {SOURCE_TABLE_NAME};

-- 2. Check for null/empty critical fields
SELECT 
    COUNT(*) as TotalRecords,
    COUNT(CASE WHEN [key_field] IS NULL OR [key_field] = '' THEN 1 END) as NullKeyFields
FROM {SOURCE_TABLE_NAME};

-- 3. Examine data distribution
SELECT 
    [key_field], 
    COUNT(*) as RecordCount 
FROM {SOURCE_TABLE_NAME} 
GROUP BY [key_field] 
ORDER BY RecordCount DESC;
```

### Post-Migration Validation
```sql
-- 1. Compare record counts
SELECT 
    (SELECT COUNT(*) FROM {SOURCE_TABLE_NAME}) as SourceCount,
    (SELECT COUNT(*) FROM {STAGING_TABLE_NAME}) as StagingCount;

-- 2. Verify data types and structure
SELECT 
    COLUMN_NAME, 
    DATA_TYPE, 
    CHARACTER_MAXIMUM_LENGTH,
    IS_NULLABLE
FROM INFORMATION_SCHEMA.COLUMNS 
WHERE TABLE_NAME = '{STAGING_TABLE_NAME}'
ORDER BY ORDINAL_POSITION;

-- 3. Sample data verification
SELECT TOP 10 * FROM {STAGING_TABLE_NAME};

-- 4. Check for data transformation accuracy
SELECT 
    s.[source_column],
    t.[TARGET_COLUMN]
FROM {SOURCE_TABLE_NAME} s
JOIN {STAGING_TABLE_NAME} t ON s.[key_field] = t.[KEY_FIELD]
WHERE s.[source_column] <> t.[TARGET_COLUMN]
LIMIT 10;
```

## Error Handling and Troubleshooting

### Common Issues and Solutions

#### 1. CSV Import Issues
**Problem:** CSV file not reading correctly
**Solutions:**
- Check file encoding (UTF-8, ANSI)
- Verify delimiter consistency
- Look for embedded quotes or special characters
- Ensure file is not locked by another process

#### 2. Data Type Mismatches
**Problem:** Source data doesn't fit target data types
**Solutions:**
- Review data dictionary mappings
- Check for unexpected data formats
- Consider data cleansing requirements
- Adjust target column definitions if necessary

#### 3. Column Mapping Issues
**Problem:** UV SQL Tool doesn't map columns correctly
**Solutions:**
- Verify data dictionary file is accessible
- Check Excel file format and structure
- Ensure column names match between CSV and dictionary
- Review reference stored procedure format

#### 4. Performance Issues
**Problem:** Large datasets causing timeouts
**Solutions:**
- Process in batches if necessary
- Add appropriate indexes
- Consider partitioning strategies
- Monitor database performance during execution

## Best Practices

### File Organization
- Keep source files in `DataSamples/` folder
- Store data dictionaries in `Mapping/` folder
- Place generated tables in `Tables/` folder
- Put stored procedures in `stored_procedures/` folder

### Documentation
- Document any manual data transformations
- Record business rules applied during migration
- Note any data quality issues discovered
- Maintain change log for iterative improvements

### Testing Strategy
- Test with sample data first
- Validate against business requirements
- Perform end-to-end testing in target system
- Document test results and any issues

### Version Control
- Commit each migration step separately
- Use descriptive commit messages
- Tag releases for production deployments
- Maintain backup of source files

## Execution Checklist

### Pre-Execution
- [ ] Source CSV file validated and accessible
- [ ] Data dictionary reviewed and confirmed
- [ ] Database connection tested
- [ ] Backup of existing data (if applicable)
- [ ] All prerequisites met

### During Execution
- [ ] Monitor UV SQL Tool output for errors
- [ ] Verify each step completes successfully
- [ ] Check intermediate results
- [ ] Document any issues encountered

### Post-Execution
- [ ] Run all validation queries
- [ ] Compare source and target record counts
- [ ] Verify data quality and transformations
- [ ] Update documentation with results
- [ ] Notify stakeholders of completion

## Template Usage Instructions

1. **Copy this template** for each new migration
2. **Replace all {PLACEHOLDER}** values with your specific details
3. **Customize validation queries** for your data structure
4. **Add entity-specific business rules** as needed
5. **Document deviations** from the standard process
6. **Update the template** based on lessons learned

## Example Usage

For a Customer migration:
- {CSV_FILE_NAME} = Customers_conca
- {DICTIONARY_FILE_NAME} = CustomersDictionary
- {SOURCE_TABLE_NAME} = srcCustomers
- {STAGING_TABLE_NAME} = stgCustomers

This would result in commands like:
```
mcp_uv_sql_tool_m_create_table with csv_file_path: "c:\D365DataMigration\DataSamples\Customers_conca.csv"
```

---
*Template Version: 1.0*
*Created: August 15, 2025*
*Tool: UV SQL Tool MCP*
*Target: Dynamics 365 Finance & Operations*