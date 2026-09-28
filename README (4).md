## DATABASE_DESIGN

# 1. General Methodology to Convert a CSV File into a Database
Step 1: Understand Use Cases and Requirements: Identify your input use cases (data entry and maintenance tasks) and output use cases (reports and queries needed now and in the future) rather than designing tables to match a single flat report

# Step 2: Identify Entities (Classes) and Attributes: Group related CSV columns into distinct entity classes (such as Customer, Order, or Product) rather than keeping all fields in one universal table

# Step 3: Establish Primary Keys and Relationships: Assign a primary key to uniquely identify every row in each entity table -Connect entities using foreign keys for 1–Many relationships, and introduce intermediate junction tables for Many–Many relationships
.
#Step 4: Apply Normalization: Evaluate functional dependencies across attributes to ensure tables avoid update anomalies (insertion, modification, and deletion errors)
.
#Step 5: SQL Schema Implementation and Data Ingestion: Write SQL CREATE TABLE statements with appropriate data types and referential integrity constraints
. Populate parent lookup tables first to generate primary key IDs before inserting child records referencing those foreign keys
.
