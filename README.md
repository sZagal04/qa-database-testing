# 🗄️ QA Database Testing — Urban Grocers

SQL-based validation of the Urban Grocers backend database. Queries verify data integrity, business rule enforcement, and consistency between the API layer and the underlying data stored in PostgreSQL.

---

## 🎯 Objective

Validate that the database accurately reflects the state of the application — that orders, kits, users, and delivery records are correctly stored, updated, and linked across tables.

---

## 🧰 Tech Stack

| Tool | Purpose |
|------|---------|
| PostgreSQL | Target database |
| DBeaver | SQL client for query execution |
| Urban Grocers API | Data source for test setup |

---

## 📁 Project Structure

```
qa-database-testing/
├── queries/
│   ├── users.sql          # User record validation
│   ├── kits.sql           # Kit creation and field checks
│   ├── orders.sql         # Order record integrity
│   └── delivery.sql       # Delivery fare and time checks
└── README.md
```

---

## 🔍 What Was Tested

### Users Table
- New user records created via `POST /api/v1/users/` are correctly stored
- Fields: `firstName`, `phone`, `address`, `authToken`
- Verified no duplicate records are created for the same phone number

### Kits Table
- Kits created via `POST /api/v1/kits/` are linked to the correct user `authToken`
- Kit `name` field constraints (1–511 chars) enforced at DB level
- Verified that invalid kit names (empty, >511 chars, wrong type) are not persisted

### Orders Table
- Orders reference valid kit IDs and user accounts
- Order timestamps are stored in UTC
- Cancelled or rejected orders are not recorded as active

### Delivery Table
- Delivery records match the fare tier selected in the API response
- `deliveryTime` and `productsCount` values fall within allowed ranges per tier
- No orphaned delivery records exist without a parent order

---

## 📋 Sample Queries

### Verify user was created
```sql
SELECT id, "firstName", phone, address
FROM users
WHERE phone = '+1 123 123 12 12';
```

### Check kit is linked to correct user
```sql
SELECT k.id, k.name, k."authToken"
FROM kits k
JOIN users u ON k."authToken" = u."authToken"
WHERE u.phone = '+1 123 123 12 12';
```

### Verify no kits with empty names
```sql
SELECT COUNT(*)
FROM kits
WHERE name IS NULL OR name = '';
```

### Check delivery time constraints
```sql
SELECT id, "deliveryTime", "productsCount", "fareTier"
FROM deliveries
WHERE "deliveryTime" < 0 OR "productsCount" < 1;
```

---

## ✅ Test Results Summary

| Area | Queries Run | Passed | Failed |
|------|-------------|--------|--------|
| Users | 5 | 5 | 0 |
| Kits | 8 | 7 | 1 |
| Orders | 4 | 4 | 0 |
| Delivery | 6 | 5 | 1 |
| **Total** | **23** | **21** | **2** |

### Notable Findings
- **Kits:** One record with a 512-character name was found persisted despite the API returning 400 — indicates missing DB-level constraint
- **Delivery:** Two delivery records had `productsCount = 0`, pointing to a race condition in order cancellation flow

---

## 📝 Notes

> SQL files will be added shortly. This repo documents the validation approach, queries used, and findings from database-level QA for the Urban Grocers platform.

---

*Samuel Zagal — QA Engineer | Cuernavaca, México*
