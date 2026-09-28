# Relational Database Design, Normalization, and CSV Migration Guide

This comprehensive reference guide covers the fundamental principles of relational database design, the step-by-step methodology for converting flat CSV files into relational database schemas, normalization techniques (1NF, 2NF, 3NF, BCNF), key management, referential integrity, and entity-relationship modeling based on *Beginning Database Design From Novice to Professional*.

---

## 1. Overview of Relational Databases

A **relational database management system (RDBMS)** structures and manages data using interrelated tables [4, 56]. Rather than relying on rigid, single-sheet spreadsheets or flat files, a relational database represents real-world entities as distinct classes/tables and manages connections between them using built-in database rules [4, 13, 60].

### Core Components
* **Tables, Fields (Attributes), and Rows (Records)**: Each class or entity from a conceptual data model becomes a **table**. Attributes become **fields (columns)** with specific data types (e.g., integer, text, date), and individual instances are stored as **rows (records)** [4, 60].
* **Primary Keys**: A field or minimal combination of fields that uniquely identifies every row in a table [62, 63, 128].
* **Foreign Keys**: Fields in one table that reference the primary key of another table, establishing logical connections across entities [5, 68].
* **Referential Integrity**: Standard rules built into database management software that ensure foreign key values always correspond to an existing primary key value in the referenced table [5, 70].
* **Relational Operations**: Relational databases allow retrieving and combining data efficiently through core operations:
  * **Select**: Filtering specific rows based on criteria [173, 197].
  * **Project**: Selecting specific columns [172, 197].
  * **Join**: Combining rows from multiple tables using foreign key relationships into unified virtual result tables [161, 177, 197].

---

## 2. Methodology: Converting Flat CSV Files into Relational Databases

Converting a flat CSV file into a relational database requires stepping away from flat spreadsheet views to decompose data into a normalized, multi-table schema [12, 13, 164].

```
  +-----------------------------------------------------------+
  |                   Flat Raw CSV Data                       |
  +-----------------------------------------------------------+
                               |
                               v
  +-----------------------------------------------------------+
  | Step 1: Identify Requirements & Use Cases (Input/Output)  |
  +-----------------------------------------------------------+
                               |
                               v
  +-----------------------------------------------------------+
  | Step 2: Group Columns into Distinct Entities & Classes    |
  +-----------------------------------------------------------+
                               |
                               v
  +-----------------------------------------------------------+
  | Step 3: Define Primary Keys & Relationship Cardinalities  |
  +-----------------------------------------------------------+
                               |
                               v
  +-----------------------------------------------------------+
  | Step 4: Apply Normalization (1NF, 2NF, 3NF Decomposition) |
  +-----------------------------------------------------------+
                               |
                               v
  +-----------------------------------------------------------+
  | Step 5: Implement SQL Schema & Populate Tables In Order   |
  +-----------------------------------------------------------+
```

### Step 1: Understand Use Cases and System Objectives
* **Analyze Requirements**: Identify both **input use cases** (how data will be entered, updated, and maintained) and **output use cases** (reports, queries, and analytical outputs required) [12, 16, 26, 48, 49].
* **Avoid Flat Spreadsheet Design**: Designing tables to match a single flat CSV or report format creates severe data redundancy and update anomalies later [12, 164].

### Step 2: Decompose Columns into Distinct Entities (Classes)
* **Separate Entity Classes**: Inspect CSV headers and separate unrelated information into dedicated tables (e.g., separating `Customer`, `Product`, and `Order` attributes) [14, 60].
* **Extract Category Lookups**: Fields with categorical text where consistent spelling is required (e.g., `Genus`, `DepartmentName`, `MembershipType`) should be extracted into separate lookup tables to avoid data entry errors [16, 141, 142].

### Step 3: Define Primary Keys & Establish Relationships
* **Assign Primary Keys**: Guarantee every table has a unique identifier (such as an auto-generated integer ID) [62, 130, 151].
* **Model 1–Many Relationships**: Place the primary key from the "1" side table as a **foreign key** inside the table on the "Many" side [5, 71, 128].
* **Model Many–Many Relationships**: Introduce an **intermediate (junction/pairing) table** containing foreign keys referencing both participating entities [5, 77, 79, 128].

### Step 4: Normalize the Schema
* Evaluate table structures against functional dependencies to ensure attributes are in the correct tables and eliminate update anomalies [6, 88, 92, 128].

### Step 5: Implement SQL Schema & Ingest Data
* **Write SQL `CREATE TABLE` Statements**: Specify data types, `PRIMARY KEY`, `FOREIGN KEY`, `NOT NULL`, and `UNIQUE` constraints [4, 69, 139, 141].
* **Data Ingestion Order**: Populate parent lookup tables first to generate primary key IDs before populating child tables that reference those IDs as foreign keys [20, 154].

---

## 3. Normalization (1NF, 2NF, 3NF, BCNF) with Examples

**Normalization** is a formal, mathematical process introduced by E. F. Codd to analyze table structures based on **functional dependencies** and eliminate **update anomalies** (insertion, modification, and deletion errors) [86, 87, 92].

### Bill Kent's Summary
> *"A table is based on **the key, the whole key, and nothing but the key** (so help me Codd)."* [115, 128]

---

### Functional Dependencies
A **functional dependency** ($A \rightarrow B$) states that knowing the value of attribute $A$ uniquely determines the value of attribute $B$ across all possible records [6, 92, 93, 128].
* **Primary Key Definition**: A minimal set of fields that functionally determines *all* non-key fields in a table ($Key \rightarrow \text{All Attributes}$) [97, 128].
* **Determinant**: Any field or set of fields that determines another attribute [114, 128].

---

### Normal Forms Overview

| Normal Form | Core Rule | Problem Solved | Remediation / Decomposition |
| :--- | :--- | :--- | :--- |
| **First Normal Form (1NF)** | Every field must contain single, **atomic** values. No multivalued attributes or repeating columns [7, 101, 103]. | Inability to query, filter, or index individual array/list values [102]. | Remove multivalued columns; place them in a separate child table linked by parent primary key [103, 104]. |
| **Second Normal Form (2NF)** | Must be in 1NF **AND** every non-key field must depend on the *entire* composite primary key [7, 107]. | **Partial Dependencies**: Non-key fields depending on only part of a composite key cause duplicate attribute storage [105, 107]. | Remove partially dependent non-key attributes into a new table with the key segment they depend on [108]. |
| **Third Normal Form (3NF)** | Must be in 2NF **AND** no non-key field can depend on another non-key field [7, 111]. | **Transitive Dependencies**: Non-key attributes determining other non-key attributes cause data redundancy across rows [110, 111]. | Move the dependent non-key attributes into a separate table, keeping the determinant as a foreign key [112]. |
| **Boyce-Codd (BCNF)** | Every determinant must be a candidate primary key [7, 113, 114]. | Anomalies when multiple overlapping candidate keys exist [113, 114]. | Decompose tables so that every field that determines another is a primary key [114, 128]. |

---

### Step-by-Step Example of Normalization

#### Raw Unnormalized Data (Spreadsheet / CSV Layout)
Suppose a raw CSV contains employee project assignments and department details:

| empID | empName | deptNum | deptName | projNum | projName | hours | usages (multivalued) |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| 1001 | John Smith | 2 | Marketing | 3, 6 | ABCPromo, Smith&Co | 8, 20 | shelter, hedging |
| 1005 | Susan Jones | 2 | Marketing | 1 | JenningsLtd | 8 | shelter |

---

#### Step 1: Conversion to First Normal Form (1NF) — "The Key"
* **Rule**: Eliminate repeating groups and multivalued list fields [101, 103].
* **Action**: Decompose multivalued lists into separate records so every cell holds an atomic value [104].
* **1NF Table**: `Assignment_1NF` (`empID`, `projNum`, `empName`, `deptNum`, `deptName`, `projName`, `hours`)
  * **Composite Primary Key**: (`empID`, `projNum`)

---

#### Step 2: Conversion to Second Normal Form (2NF) — "The Whole Key"
* **Check Dependencies**:
  * (`empID`, `projNum`) $\rightarrow$ `hours` *(Full Dependency — requires both employee and project)*
  * `empID` $\rightarrow$ `empName`, `deptNum`, `deptName` *(Partial Dependency on `empID`)*
  * `projNum` $\rightarrow$ `projName` *(Partial Dependency on `projNum`)*
* **Action**: Decompose into three tables to eliminate partial dependencies [108]:
  1. `Employee_2NF` (`empID`, `empName`, `deptNum`, `deptName`) — Key: `empID`
  2. `Project_2NF` (`projNum`, `projName`) — Key: `projNum`
  3. `ProjectAssignment_2NF` (`empID`, `projNum`, `hours`) — Composite Key: (`empID`, `projNum`)

---

#### Step 3: Conversion to Third Normal Form (3NF) — "Nothing But the Key"
* **Check Transitive Dependencies in `Employee_2NF`**:
  * `empID` $\rightarrow$ `deptNum`
  * `deptNum` $\rightarrow$ `deptName` *(Transitive Dependency: non-key field determining non-key field)* [110, 111]
* **Action**: Decompose `Employee_2NF` into two separate tables [112]:
  1. `Department` (`deptNum`, `deptName`) — Key: `deptNum`
  2. `Employee` (`empID`, `empName`, `deptNum`) — Key: `empID`, Foreign Key: `deptNum`

#### Final 3NF Relational Schema:
* **`Department`**: (`deptNum`, `deptName`)
* **`Employee`**: (`empID`, `empName`, `deptNum`*)
* **`Project`**: (`projNum`, `projName`)
* **`ProjectAssignment`**: (`empID`*, `projNum`*, `hours`)

---

## 4. Primary Keys, Foreign Keys, and Referential Integrity

### Primary Keys
* **Surrogate Keys (Generated IDs)**: Automatically generated numeric identifiers (e.g., `AUTO_INCREMENT`, `IDENTITY`) are preferred when natural attributes (names, addresses) change over time [8, 130, 131, 151].
* **Concatenated (Composite) Keys**: Keys composed of multiple attributes, common in junction tables representing Many–Many relationships [63, 79, 133].
* **Unique Indexes**: Defining a primary key automatically creates a unique clustered/nonclustered index to enforce row uniqueness [62, 153].

### Foreign Key Placement Rules
1. **1–Many Relationships**: Place the primary key from the "1" end table as a foreign key column in the "Many" end table [5, 71, 128].
2. **Many–Many Relationships**: Create a dedicated junction table containing foreign keys to both parent tables [5, 77, 79, 128].
3. **1–1 Relationships**: Place the foreign key in the table where the relationship is compulsory, and add a `UNIQUE` constraint to ensure a 1-to-1 match [5, 81, 139, 141].

### Referential Integrity & Foreign Key Deletion Options
Referential integrity ensures foreign key values match valid primary keys in the referenced parent table [5, 70, 144]. When a parent record is deleted, three strategies handle child foreign keys [145, 151]:

```sql
-- Disallow Delete (Default standard): Prevents parent deletion if child records exist [146]
CREATE TABLE Team (
    teamID INT PRIMARY KEY,
    captainID INT FOREIGN KEY REFERENCES Member(memberID)
);

-- Nullify Delete: Sets child foreign key fields to NULL when parent is deleted [146, 148]
CREATE TABLE Team (
    teamID INT PRIMARY KEY,
    captainID INT FOREIGN KEY REFERENCES Member(memberID) ON DELETE SET NULL
);

-- Cascade Delete: Automatically deletes all referencing child records [147, 148]
CREATE TABLE OrderItem (
    orderID INT,
    itemID INT,
    PRIMARY KEY (orderID, itemID),
    FOREIGN KEY (orderID) REFERENCES Orders(orderID) ON DELETE CASCADE
);
```

---

## 5. Weak vs. Strong Entities and Relationships

```
  +-----------------------+                    +-----------------------+
  |    Strong Entity      |                    |      Weak Entity      |
  |     (e.g., Farm)      |                    |    (e.g., Paddock)    |
  +-----------------------+                    +-----------------------+
  | PK: farmID            |<---- 1---n ------->| PK: (farmID, padNum)  |
  | name, address         |   Ownership Link   | FK: farmID            |
  +-----------------------+                    +-----------------------+
```

### Strong vs. Weak Entities
* **Strong Entities**: Independent entity classes that exist on their own without relying on another class for identification (e.g., `Customer`, `Farm`, `Product`) [132].
* **Weak (Dependent/Ownership) Entities**: Classes that cannot exist or be uniquely identified without a parent entity (e.g., `Paddock` owned by a `Farm`, `Sample` collected during a `Visit`) [132, 135].
  * **Key Propagation**: Weak entities often use composite primary keys consisting of the parent's primary key plus a local discriminator (e.g., `farmID` + `paddockNum`) [133, 136, 137].

### Relationship Cardinality and Optionality
* **Cardinality**: The maximum number of related records (1 or Many) [27, 60].
* **Optionality (Minimum Cardinality)**:
  * **Mandatory Relationship (Optionality = 1)**: Child entity *must* be linked to a parent entity. Enforced in SQL by setting `NOT NULL` on the foreign key field [27, 73, 144].
  * **Optional Relationship (Optionality = 0)**: Child entity can exist without a parent entity. Allowed by permitting `NULL` values in the foreign key field [27, 73, 144].

---

## Summary Checklist for Relational Database Conversion

1. **Decompose** flat CSV headers into clear, logical entity classes [14, 60].
2. **Assign** a surrogate integer primary key to every entity table [62, 130, 151].
3. **Link** 1–Many relationships using foreign keys [5, 71].
4. **Build** junction tables with foreign keys for Many–Many relationships [5, 79].
5. **Verify** 1NF (atomic values), 2NF (no partial key dependencies), and 3NF (no transitive non-key dependencies) [115, 128].
6. **Enforce** referential integrity with appropriate `ON DELETE` rules (`DISALLOW`, `SET NULL`, or `CASCADE`) [145, 148].
