# PharmaCare — Medicine Inventory & Sales Management System

PharmaCare is a full-stack DBMS application for pharmacy inventory, batch tracking, expiry management, and transactional sales. It uses a normalized relational SQL schema with First-Expiry-First-Out (FEFO) batch allocation and atomic transactions.

---

## 1. System Architecture

- **Backend**: Python Flask REST API with parameterized SQL queries and database transaction contexts.
- **Frontend**: React 18 with TypeScript, Vite, and Lucide icons.
- **Database Engine**: Supports both **MySQL 8.0+** and **SQLite 3** via a database adapter.
  - SQLite mode works out of the box with zero external configuration (`pharmacare.db`).
  - MySQL mode connects to a local or remote MySQL 8.0+ database configured in `.env`.

---

## 2. Prerequisites

- **Python**: 3.10+ (tested with Python 3.14)
- **Node.js**: 18+ and `npm`
- **MySQL** (optional): MySQL 8.0+ if running with MySQL engine.

---

## 3. Installation & Setup

### A. Clone & Setup Backend
```bash
# In project root
pip install -r backend/requirements.txt
```

### B. Setup Frontend
```bash
cd frontend
npm install
npm run build
cd ..
```

### C. Configure Environment Variables
Create a `.env` file in the project root (copied from `.env.example`):
```env
# Set DB_TYPE to 'mysql' or 'sqlite'
DB_TYPE=sqlite
SQLITE_DB_PATH=pharmacare.db

# MySQL Settings (if DB_TYPE=mysql)
DB_HOST=localhost
DB_PORT=3306
DB_NAME=pharmacare
DB_USER=root
DB_PASSWORD=your_mysql_password_here

# Flask Server Configuration
FLASK_PORT=5000
FLASK_DEBUG=True
SECRET_KEY=dev-secret-key-pharmacare-secure
```

---

## 4. Database Setup & Seeding

The database auto-initializes on backend startup if tables do not exist. To manually initialize or reset:

### For MySQL:
```bash
mysql -u root -p < database/schema.sql
mysql -u root -p < database/seed.sql
```

### For SQLite:
SQLite auto-populates `pharmacare.db` from `database/schema.sql` and `database/seed.sql` on the first launch or test run.

---

## 5. Running the Application

### Start the Backend:
```bash
py backend/run.py
```
*API server runs at `http://127.0.0.1:5000`.*

### Start the Frontend:
```bash
cd frontend
npm run dev
```
*Frontend interface runs at `http://localhost:3000` with reverse proxy to `http://127.0.0.1:5000`.*

---

## 6. Running Tests

Run the automated test suite with pytest:
```bash
py -m pytest backend/tests -v
```

All 21 automated tests verify:
- Medicine CRUD, duplicates rejection, and safe deactivation.
- FEFO earliest-expiry batch allocation for sales.
- Atomic transaction rollback on partial checkout failures.
- Stock ledger movements and quantity deductions.
- SQL report join integrity without double-counting.

---

## 7. Key SQL Concepts Implemented

1. **FEFO (First-Expiry-First-Out) Querying**:
   ```sql
   SELECT batch_id, batch_number, expiry_date, selling_price, quantity_on_hand
   FROM batches
   WHERE medicine_id = ? AND quantity_on_hand > 0 AND expiry_date >= CURDATE()
   ORDER BY expiry_date ASC, batch_id ASC;
   ```
2. **Atomic Transaction Management**:
   All stock deductions and sales insertions execute within an explicit `BEGIN TRANSACTION ... COMMIT / ROLLBACK` block.
3. **Auditable Stock Movements Ledger**:
   Every stock deduction, purchase intake, or manual adjustment writes an immutable record to `stock_movements`.
4. **Relational Constraints & Views**:
   `ON DELETE RESTRICT` protects audit integrity so medicines with transaction history cannot be inadvertently deleted.
   Pre-aggregated SQL views (`v_active_stock`, `v_expiring_batches`, `v_sales_summary`) calculate live operational statistics.

---

## 8. Documentation Links

- [Entity-Relationship Diagram (ERD)](docs/erd.md)
- [REST API Reference](docs/api.md)
- [Test Plan & Verification Checklist](docs/test-plan.md)

