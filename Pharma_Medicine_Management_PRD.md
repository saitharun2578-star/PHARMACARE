# Product Requirements Document (PRD)

## Pharma Medicine Management System

**Project name:** PharmaCare --- Medicine Inventory & Sales Management\
**Document version:** 1.0\
**Status:** Initial build specification\
**Target environment:** Google Antigravity\
**Database:** MySQL 8.0+\
**Application type:** Full-stack web application for a pharmacy /
medical shop\
**Primary goal:** Build a working, demonstrable application whose core
data operations are implemented correctly in SQL.

------------------------------------------------------------------------

## 1. Product Overview

PharmaCare is a medicine inventory and sales management system for a
small-to-medium pharmacy. It centralizes medicine records, batch-level
stock, expiry tracking, suppliers, purchases, customers, sales, and
reporting.

The application must be functional, not just a static UI. Forms must
save to MySQL, tables must display real database records, and stock must
update consistently when purchases and sales are recorded.

This is an educational portfolio project using synthetic demonstration
data. It is not intended to provide medical advice, prescribe medicines,
or replace regulated pharmacy software.

## 2. Problem Statement

Pharmacies may track stock, expiry dates, supplier purchases, and sales
manually or across separate files. This makes it difficult to identify
expiring stock, prevent accidental overselling, find reorder needs, and
understand sales performance.

PharmaCare provides one database-backed system for these daily
operations.

## 3. Goals and Success Criteria

### Goals

-   Maintain a searchable medicine catalogue.
-   Track inventory by batch, including expiry date and quantity.
-   Record suppliers and stock purchases.
-   Record sales with itemized bill details.
-   Automatically adjust inventory within safe database transactions.
-   Flag low-stock, near-expiry, and expired batches.
-   Provide useful reports using SQL queries, joins, aggregates, and
    views.
-   Demonstrate sound relational database design and data integrity.

### Acceptance criteria

-   The app starts locally using documented setup instructions.
-   Database schema and seed data can be installed from supplied SQL
    scripts.
-   All core forms perform validated create/read/update operations
    against MySQL.
-   A completed purchase increases the relevant batch quantity.
-   A completed sale decreases the relevant batch quantity exactly once.
-   A sale cannot exceed available eligible stock.
-   Expired batches cannot be sold.
-   Failed transactions do not leave partial sales or incorrect
    inventory.
-   Dashboard totals and reports are calculated from database records,
    not hard-coded values.
-   Empty states, validation messages, and database errors are handled
    clearly.

## 4. Target Users and Roles

### Primary user: Pharmacy administrator / owner

-   Manages medicines, suppliers, purchases, and stock.
-   Reviews sales, expiry warnings, and business reports.

### Secondary user: Pharmacy staff

-   Searches medicines and records customer sales.
-   Views stock availability and batch expiry.

For the first version, implement a simple authenticated staff/admin role
model if practical. If authentication makes the first build unstable,
keep the app local-only and clearly document that authentication is not
yet implemented. Do not present an unauthenticated deployment as
production-ready.

## 5. Scope

### In scope --- MVP

1.  Dashboard
2.  Medicine catalogue and categories
3.  Batch-level inventory
4.  Supplier management
5.  Purchase entry and purchase line items
6.  Customer records (optional customer for walk-in sales)
7.  Sales entry and itemized receipts
8.  Stock adjustment / stock movement history
9.  Low-stock, near-expiry, and expired-stock reports
10. Sales and inventory reports
11. Search, filters, sorting, and pagination where appropriate
12. SQL schema, seed data, and setup documentation

### Out of scope for MVP

-   Patient diagnosis, prescribing, or treatment recommendations
-   Drug interaction or dosage advice
-   Insurance claims and electronic prescriptions
-   Real payment gateway integration
-   Multi-branch synchronization
-   Government or wholesaler API integrations
-   Barcode hardware integration
-   Manufacturing execution, laboratory quality control, and batch
    release workflows
-   Cloud production deployment or regulatory compliance certification

## 6. Recommended Technology

Use this stack unless the existing workspace already has a functioning
equivalent: - **Frontend:** React with TypeScript and a modern
responsive component system. - **Backend:** Python Flask REST API. -
**Database:** MySQL 8.0+. - **Database access:** mysql-connector-python
or SQLAlchemy; choose one and use it consistently. - **Charts:** A
lightweight chart library for sales summaries. - **Configuration:**
Environment variables in a `.env` file; commit only `.env.example`. -
**Version control:** Keep code organized and include a `.gitignore`.

Do not put database credentials in frontend code. All SQL access must
happen on the backend.

## 7. Information Architecture and UI

Use a clean, modern pharmacy operations dashboard. Prioritize clarity
and speed over decorative effects. Use consistent spacing, readable
typography, restrained colors, clear status labels, and accessible
contrast. The UI must work on desktop and tablet, with basic mobile
responsiveness.

Main navigation: - Dashboard - Medicines - Inventory & Batches -
Suppliers - Purchases - Sales - Customers - Reports - Settings / About

Each list page should include a clear page title, primary action,
search/filter controls, data table, loading state, empty state, and
actionable error feedback.

## 8. Functional Requirements

### FR-01: Dashboard

Display database-derived values: - Number of active medicine products -
Total units in non-expired inventory - Low-stock product count - Batches
expiring within the next 30 days - Expired batches still on record -
Sales total for today and the selected date range - Recent sales - A
monthly sales chart

Definitions: - "Today" uses the application/server's configured local
business timezone. - Expiry warning window defaults to 30 days and
should be configurable. - Do not count expired stock as sellable
stock. - Explain whether sales totals represent gross sales or net sales
after discounts/returns.

### FR-02: Medicine catalogue

Fields: - `medicine_id` (primary key) - `name` (required) -
`generic_name` (optional) - `category_id` (foreign key) - `manufacturer`
(optional) - `dosage_form` (e.g., tablet, capsule, syrup; descriptive
only) - `strength` (optional descriptive field) - `unit_label` (e.g.,
tablet, bottle, strip, pack) - `reorder_level` (non-negative integer) -
`is_active` (boolean) - `created_at`, `updated_at`

Capabilities: - Create, view, edit, deactivate, and search medicines. -
Prevent accidental duplicate catalogue entries where a reasonable unique
rule can be defined. - Do not hard-delete a medicine referenced by
purchase or sales history; deactivate it instead. - Store no patient
diagnosis or prescription information in the MVP.

### FR-03: Categories

Manage categories such as analgesics, antibiotics, vitamins, and
personal care for demo purposes. Categories are classification labels
only and must not be used to recommend medicine or treatment.

Fields: `category_id`, `category_name`, optional `description`.

### FR-04: Batch-level inventory

Each batch belongs to one medicine. Track: - `batch_id` (primary key) -
`medicine_id` (foreign key) - `batch_number` - `expiry_date` -
`purchase_price` - `selling_price` - `quantity_on_hand` - `supplier_id`
(optional foreign key, if provenance is useful) - `created_at`,
`updated_at`

Rules: - A batch number must be non-empty. - Quantity cannot be
negative. - Prices cannot be negative. - Keep stock by batch; do not
store a manually edited total stock value on the medicine record. -
Product availability is calculated by summing eligible batches. - Do not
sell expired batches. - Prefer FEFO (First Expiry, First Out) when
selecting batches for a sale: allocate from the earliest-expiring
eligible batch first. - If the same supplier can reuse batch numbers
across products, enforce uniqueness on the appropriate combination, such
as `(medicine_id, batch_number)`, rather than globally.

### FR-05: Supplier management

Fields: - `supplier_id` - `supplier_name` (required) - `contact_person`
(optional) - `phone` (optional) - `email` (optional) - `address`
(optional) - `is_active` - timestamps

Capabilities: - Create, edit, search, view purchase history, and
deactivate suppliers. - Validate email format if supplied. - Do not
invent supplier data; use clearly synthetic seed data.

### FR-06: Purchases / stock receiving

A purchase records stock received from a supplier.

Purchase header: - `purchase_id` - `supplier_id` - `purchase_date` -
`invoice_number` (optional) - `status` (e.g., draft, received,
cancelled) - `total_amount` - `notes` (optional) - timestamps

Purchase items: - `purchase_item_id` - `purchase_id` - `batch_id` or
medicine/batch details sufficient to create or update a batch -
`quantity` - `unit_cost` - `line_total`

Rules: - Validate all quantities and costs. - On confirming a received
purchase, add quantities to the appropriate batches exactly once. - Use
a database transaction for purchase confirmation. - Prevent duplicate
confirmation from increasing stock twice. - Cancelled purchases must not
count as received stock. - Preserve historical purchase details even if
the medicine or supplier is later deactivated.

### FR-07: Customer management

Fields: - `customer_id` - `customer_name` (optional for walk-in
customer) - `phone` (optional) - timestamps

Keep customer information minimal. Do not collect sensitive medical
history. A walk-in sale must be possible without creating a customer
record.

### FR-08: Sales and receipts

Sale header: - `sale_id` - `sale_datetime` - `customer_id` (nullable) -
`subtotal` - `discount_amount` (default 0) - `tax_amount` (default 0,
configurable and not assumed to represent legal tax advice) -
`grand_total` - `status` (completed, cancelled, or refunded according to
implemented workflow) - `created_by` (if users/authentication are
implemented) - timestamps

Sale items: - `sale_item_id` - `sale_id` - `batch_id` - `quantity` -
`unit_price` - `line_total`

Rules: - Allow searching medicines by name and viewing available
stock. - Validate quantity and price on the server, not only in the
browser. - Allocate stock by FEFO from eligible batches. - Refuse a sale
if eligible stock is insufficient. - Refuse a sale from expired
batches. - Record the batch used for every sale item. - Create sale
header, sale items, and inventory movements in one transaction. -
Prevent duplicate submission from creating duplicate sales. - Calculate
totals on the backend using decimal-safe numeric types; do not use
floating-point arithmetic for money. - Provide a printable receipt view
with sale number, date, items, quantities, unit prices, totals, and a
clear demo/business identity. - Do not implement medicine
recommendations or medical advice.

### FR-09: Stock movement ledger

Maintain an auditable history of stock changes.

Fields: - `movement_id` - `batch_id` - `movement_type` (purchase, sale,
adjustment, return, or reversal) - `quantity_change` (positive for
additions, negative for deductions) - `reference_type` -
`reference_id` - `reason` - `created_at` - `created_by` if
authentication is enabled

Rules: - Record a movement for each committed stock change. - Do not
silently overwrite stock history. - Manual adjustments require a
reason. - Reconcile current batch quantity against opening stock plus
all committed movements. - Corrections should use reversal/adjustment
entries rather than deleting historical movements.

### FR-10: Alerts and inventory reports

Provide filters for: - Low stock: sellable stock at or below the
medicine's reorder level. - Near expiry: expiry date from today through
the next 30 days, inclusive. - Expired stock: expiry date before today
and quantity greater than zero. - Out of stock: no sellable quantity
available. - Batch stock: quantities by medicine, batch, supplier, and
expiry date.

Use a consistent date rule and explain it in the UI. Alerts are
operational reminders, not clinical advice.

### FR-11: Sales and business reports

Include: - Sales by day, week, month, and custom date range -
Top-selling medicines by quantity - Highest-grossing medicines by sales
amount - Revenue by category - Purchase totals by supplier - Current
inventory valuation using recorded purchase cost, with the valuation
method documented - Expired and near-expiry stock value - Sales detail
report with receipt links

All reports must be based on SQL data. Use correct joins that do not
multiply totals when joining headers and line items. Document report
definitions.

### FR-12: Search and usability

-   Search medicines by name or generic name.
-   Filter batches by expiry status and supplier.
-   Filter sales by date and status.
-   Provide sorting and pagination for large lists.
-   Display helpful validation messages beside invalid fields.
-   Confirm before destructive or irreversible actions.
-   Use deactivation and status transitions rather than deleting
    historical records.

## 9. Database Design Requirements

Use a normalized relational schema. Suggested tables: 1. `categories` 2.
`medicines` 3. `suppliers` 4. `batches` 5. `purchases` 6.
`purchase_items` 7. `customers` 8. `sales` 9. `sale_items` 10.
`stock_movements` 11. `users` (only if authentication is implemented)

Use: - Primary keys for every table. - Foreign keys for relationships. -
`NOT NULL`, `UNIQUE`, and `CHECK` constraints where supported. -
`DECIMAL`, not `FLOAT` or `DOUBLE`, for currency. - `DATE` for expiry
dates and appropriate `DATETIME`/`TIMESTAMP` fields for transactions. -
Indexes for common searches, foreign keys, sale dates, expiry dates,
medicine names, and batch identifiers. - Explicit transactions for
stock-changing operations. - Parameterized SQL queries to prevent SQL
injection. - Consistent naming conventions and documented relationships.

Important modeling decision: inventory is batch-level. A medicine can
have multiple batches with different expiry dates and prices. Sales must
reference the batch actually sold. Do not model a single expiry date or
stock quantity on the medicine table.

## 10. API / Backend Expectations

Use a REST-style API. Suggested endpoints: - `GET /api/dashboard` -
`GET, POST /api/medicines` - `GET, PUT, PATCH /api/medicines/{id}` -
`GET, POST /api/categories` - `GET, POST /api/suppliers` -
`GET, PUT /api/suppliers/{id}` - `GET /api/batches` -
`GET /api/inventory/low-stock` - `GET /api/inventory/near-expiry` -
`GET /api/inventory/expired` - `GET, POST /api/purchases` -
`POST /api/purchases/{id}/receive` - `GET, POST /api/sales` -
`GET /api/sales/{id}` - `GET /api/reports/sales` -
`GET /api/reports/top-medicines` -
`GET /api/reports/inventory-valuation`

Use consistent JSON responses and appropriate HTTP status codes.
Validate request payloads and handle expected errors. The exact route
structure may be adjusted to fit the chosen framework, but all MVP
functions must be available.

## 11. Security and Reliability

-   Keep database credentials and secrets in environment variables.
-   Include `.env.example` with placeholders only.
-   Never expose credentials in logs, frontend bundles, or repository
    files.
-   Use parameterized queries.
-   Validate and normalize inputs on the backend.
-   Use transactions and row-level locking or equivalent concurrency
    protection when deducting stock.
-   Avoid race conditions where two simultaneous sales oversell the same
    batch.
-   Return safe error messages to the user and log diagnostic details
    server-side.
-   Add authentication and role checks if included in the build.
-   Do not claim compliance with any pharmacy, privacy, or tax
    regulation without a separate verified implementation and review.

## 12. Demo Data

Provide synthetic seed data for: - 8--12 medicines across several
categories - 3--5 suppliers - Multiple batches for at least some
medicines - Batches with future expiry dates, near-expiry dates, and
expired dates - Several purchases - Several customers, including walk-in
sales - At least 10 sample sales with sale items - Stock movements that
reconcile with the starting inventory

Seed data must be clearly fictional and must not imply clinical advice.
Make dates relative to a documented demo date or include a repeatable
seed/reset procedure so expiry dashboard examples remain understandable.

## 13. Testing Requirements

Include automated tests or a documented repeatable test checklist for:

1.  Create a medicine with valid data.
2.  Reject a medicine with missing required name.
3.  Reject negative quantity and negative price.
4.  Receive a purchase and verify stock increases exactly once.
5.  Complete a sale and verify stock decreases by the sold quantity.
6.  Reject a sale when stock is insufficient.
7.  Reject a sale from an expired batch.
8.  Verify FEFO chooses the earliest-expiring eligible batch.
9.  Simulate a failure during sale creation and verify the transaction
    rolls back.
10. Verify cancelled purchases do not increase stock.
11. Verify low-stock, near-expiry, and expired reports use correct
    conditions.
12. Verify sales totals are correct for multiple line items.
13. Verify report joins do not duplicate totals.
14. Verify SQL queries use parameters.
15. Verify duplicate form submission does not create duplicate
    transactions.

## 14. Deliverables

The finished project must include: - Working frontend and backend source
code - SQL schema script (`database/schema.sql`) - Seed data script
(`database/seed.sql`) - Optional demo reset script with clear warnings -
`.env.example` - `README.md` with prerequisites, installation, database
setup, run commands, and troubleshooting - Database relationship diagram
(ERD) in Mermaid or a diagram file - API endpoint documentation - Test
checklist or automated tests - Sample screenshots only after the app is
actually running - A concise explanation of important SQL concepts used

## 15. Suggested Project Structure

``` text
pharmacare/
  frontend/
  backend/
    app/
      routes/
      services/
      repositories/
      schemas/
    tests/
    requirements.txt
  database/
    schema.sql
    seed.sql
  docs/
    erd.md
    api.md
    test-plan.md
  .env.example
  .gitignore
  README.md
```

Adjust this structure if the selected framework has a better convention,
but keep database scripts, backend logic, tests, and documentation
clearly separated.

## 16. Build Plan for Antigravity

Build incrementally. Do not attempt to generate the whole application in
one unverified pass.

### Phase 1 --- Plan and database

-   Inspect the workspace before modifying files.
-   Create an implementation plan and confirm the schema.
-   Build the ERD, schema, constraints, indexes, and seed scripts.
-   Verify foreign keys and sample queries.

### Phase 2 --- Backend

-   Configure the database connection using environment variables.
-   Implement validation and API endpoints.
-   Implement transactional purchase receiving and sales.
-   Add automated tests for stock integrity.

### Phase 3 --- Frontend

-   Build the navigation and dashboard.
-   Implement medicine, batch, supplier, purchase, customer, and sales
    pages.
-   Connect all screens to backend APIs.
-   Add loading, empty, success, and error states.

### Phase 4 --- Reports and hardening

-   Add expiry, stock, purchase, and sales reports.
-   Verify report calculations against seed data.
-   Test transaction rollback and duplicate submissions.
-   Fix errors before polishing visuals.

### Phase 5 --- Final verification

-   Follow README instructions in a clean local setup.
-   Run database scripts and tests.
-   Start frontend and backend and verify core workflows end to end.
-   Document known limitations honestly.

## 17. Instructions to the AI Development Agent

Act as a careful senior full-stack engineer and SQL database designer.

1.  First inspect the current workspace and report existing files and
    technologies.
2.  Present a concise implementation plan and database relationship
    design before making major changes.
3.  Prefer working, understandable code over unnecessary abstraction.
4.  Implement the MVP in phases and run checks after each phase.
5.  Do not claim a feature is complete unless it is implemented and
    verified.
6.  Do not use hard-coded dashboard numbers or fake API responses in the
    final working application.
7.  Keep SQL in the database layer and business rules in backend
    services.
8.  Use parameterized queries and transactions for all stock-changing
    operations.
9.  Never silently delete sales, purchases, or stock movement history.
10. If a requirement is ambiguous, choose the safest reasonable default
    and document it instead of blocking progress.
11. If a dependency, credential, or local service is missing, explain
    exactly what is needed and continue with independent tasks.
12. Finish by providing setup instructions, files created, tests run,
    test results, and remaining limitations.

## 18. Definition of Done

The MVP is complete when: - The database can be created from the
provided SQL scripts. - The application connects to MySQL through the
backend. - Medicines and suppliers can be managed. - Batches can be
tracked by expiry and quantity. - Purchases add stock once, and sales
deduct stock once. - Expired and insufficient stock cannot be sold. -
Stock movement history is recorded. - Dashboard and reports show
database-derived results. - The core test checklist passes. - Another
student can follow the README and run the project locally.

**Final product principle:** correctness of inventory and transaction
data is more important than visual complexity. Build a reliable
SQL-backed system first, then improve the interface.
