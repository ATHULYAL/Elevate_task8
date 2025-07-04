# Elevate_task8
# Lost & Found Management System

A lightweight MySQL‑backed application for recording *lost* and *found* items on campus (or any shared space) and streamlining the claim‑approval workflow.

---

## ✨ Key Features

| Module               | Description                                                         |
| -------------------- | ------------------------------------------------------------------- |
| **Users**            | Authenticates students and staff; role‑based restrictions           |
| **Categories**       | Central list of item categories (e.g. Electronics, Books)           |
| **Lost Items**       | Records items reported *lost* by users                              |
| **Found Items**      | Records items turned in / found, with `is_claimed` toggle           |
| **Claims**           | Students can claim found items; admins approve or reject            |
| **Stored Procedure** | `register_lost_item` – one‑step insert that auto‑creates categories |
| **Stored Function**  | `unclaimed_count_by_category` – quick metric for dashboards         |

---

## 🗄️ Database Schema (DDL)

```sql
-- Create the Users table
CREATE TABLE Users (
    user_id   INT AUTO_INCREMENT PRIMARY KEY,
    name      VARCHAR(100) NOT NULL,
    email     VARCHAR(100) UNIQUE NOT NULL,
    role      VARCHAR(20) NOT NULL CHECK (role IN ('student', 'staff'))
);

-- Categories lookup
CREATE TABLE Categories (
    category_id INT AUTO_INCREMENT PRIMARY KEY,
    name        VARCHAR(50) NOT NULL UNIQUE
);

-- Lost items reported by users
CREATE TABLE Lost_Items (
    lost_id     INT AUTO_INCREMENT PRIMARY KEY,
    user_id     INT,
    category_id INT,
    item_name   VARCHAR(100),
    description TEXT,
    date_lost   DATE NOT NULL,
    location    VARCHAR(100),
    FOREIGN KEY (user_id)     REFERENCES Users(user_id),
    FOREIGN KEY (category_id) REFERENCES Categories(category_id)
);

-- Items that have been found/turned in
CREATE TABLE Found_Items (
    found_id     INT AUTO_INCREMENT PRIMARY KEY,
    user_id      INT,
    category_id  INT,
    item_name    VARCHAR(100),
    description  TEXT,
    date_found   DATE NOT NULL,
    location     VARCHAR(100),
    is_claimed   BOOLEAN DEFAULT FALSE,
    FOREIGN KEY (user_id)     REFERENCES Users(user_id),
    FOREIGN KEY (category_id) REFERENCES Categories(category_id)
);

-- Claims raised by users on found items
CREATE TABLE Claims (
    claim_id  INT AUTO_INCREMENT PRIMARY KEY,
    user_id   INT,
    found_id  INT,
    claim_date DATE DEFAULT (CURDATE()),
    status    ENUM('pending','approved','rejected') DEFAULT 'pending',
    FOREIGN KEY (user_id)  REFERENCES Users(user_id),
    FOREIGN KEY (found_id) REFERENCES Found_Items(found_id)
);
```

---

## ⚙️ Stored Routine — `register_lost_item`

*Registers a lost item and silently creates the category if it does not exist.*

```sql
CALL register_lost_item(
    p_user_id     => 3,
    p_category    => 'Electronics',
    p_item_name   => 'Dell XPS 13',
    p_desc        => 'Silver, sticker on lid',
    p_date_lost   => CURDATE(),
    p_location    => 'Library – 2nd floor',
    p_lost_id     => @new_id
);
SELECT @new_id;   -- returns newly generated lost_id
```

### ✔️ Highlights

* **IN/OUT parameters** for clean API‑like calls
* **Category auto‑provisioning** avoids duplicates in client code
* **SIGNAL** throws meaningful error if `user_id` is invalid

---

## 📈 Stored Function — `unclaimed_count_by_category`

Returns a quick count of found items **still waiting** for pickup.

```sql
SELECT unclaimed_count_by_category('Electronics')  AS pending_electronics;
```

---

## 🚀 Getting Started

### Prerequisites

* MySQL 8.x (tested on 8.2)
* An SQL client or GUI (CLI, DBeaver, phpMyAdmin, etc.)

### 1 — Clone & Load

```bash
git clone https://github.com/<your‑org>/lost‑and‑found.git
cd lost‑and‑found
mysql -u <user> -p < schema.sql   # imports all DDL & routines
```

### 2 — Seed Demo Data (optional)

```sql
INSERT INTO Users (name,email,role) VALUES
  ('Alice','alice@example.com','student'),
  ('Bob','bob@example.com','staff');
```

### 3 — Run Example Calls

```sql
-- Report a lost phone
CALL register_lost_item(1,'Mobile Phones','iPhone 15','Blue case',CURDATE(),'Cafeteria',@id);

-- See how many phones are waiting in inventory
SELECT unclaimed_count_by_category('Mobile Phones');
```

---

## 🛠️ Extending the System

| Idea                  | Where to start                                                     |
| --------------------- | ------------------------------------------------------------------ |
| **Approval workflow** | Procedure to transition `Claims.status` + trigger on `Found_Items` |
| **Audit logging**     | AFTER UPDATE triggers writing to `Claims_Audit`                    |
| **Web/API layer**     | Build a simple Flask/FastAPI service consuming these routines      |
| **Dashboard**         | Use Grafana/Superset to visualise `unclaimed_count_by_category()`  |

---

## 📜 License

MIT © 2025 Athulya L

---

## 🙋‍♀️ Questions / Contributing

Open an issue or start a discussion in the repo. PRs welcome!
