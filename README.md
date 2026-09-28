# DATABASE_DESIGN

## Basic Methodology: CSV Tables to Relational Database

### 1) Profile and prepare the CSV data
- Collect all CSV files and identify the business entity each file represents (for example: customers, orders, products).
- Standardize column names, data types, date formats, and null handling rules before import.
- Define a data dictionary with each field's meaning, type, allowed values, and quality constraints.

### 2) Define entities and candidate keys
- For each CSV, identify unique identifiers and natural keys.
- If no stable natural key exists, create surrogate keys (for example: `customer_id`, `order_id`).
- Remove duplicate rows and document deduplication logic.

### 3) Normalize the schema
- **1NF**: Ensure each column is atomic (no comma-separated lists, no repeating groups).
- **2NF**: Move attributes that depend on only part of a composite key into separate tables.
- **3NF**: Remove transitive dependencies by placing derived or lookup attributes in reference tables.
- Use junction tables for many-to-many relationships (for example: `order_items`, `student_courses`).

### 4) Model relationships and constraints
- Define primary keys for all tables.
- Add foreign keys for one-to-many and many-to-many relationships.
- Set referential actions intentionally (`RESTRICT`, `CASCADE`, `SET NULL`) based on business rules.
- Add `UNIQUE`, `NOT NULL`, and `CHECK` constraints to protect data quality.

### 5) Plan and execute ETL loading
- Load dimension/reference tables first, then parent tables, then child tables.
- Validate referential integrity before loading dependent tables.
- Log rejected rows with reasons (type mismatch, missing key, invalid format).
- Use idempotent loads so reruns do not duplicate data.

### 6) Validate and optimize
- Reconcile record counts between CSV sources and database tables.
- Run quality checks for orphan records, duplicates, null violations, and domain violations.
- Add indexes on primary keys, foreign keys, and frequently filtered columns.
- Document the final ER model and loading process for maintainability.

### 7) Governance and change management
- Version-control schema migrations and ETL scripts.
- Add automated validation tests for key constraints and row-level quality rules.
- Monitor data drift and update mappings when CSV source structure changes.