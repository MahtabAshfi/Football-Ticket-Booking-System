# Football Ticket Booking System — Database Design & SQL Queries

A relational database project simulating a simplified football ticket booking platform. The system tracks registered users, upcoming tournament matches, and individual seat booking transactions.

## Overview

This project demonstrates database schema design and SQL querying techniques including joins, subqueries, aggregations, pattern matching, null handling, and pagination.

## Entity Relationship Diagram

The schema consists of three core tables connected through foreign key relationships:

- **Users (1) → Bookings (Many)**: A single user can place multiple ticket bookings across different matches.
- **Matches (1) → Bookings (Many)**: A single match can be associated with many individual booking records from different users.
- **Bookings**: Acts as the junction, where each row maps exactly one user to one match for a specific seat.


> A full visual ERD is available here: **[]**

## Schema Design

### 1. Users Table

This table tracks all administrative staff and customers who use the platform.

| Column | Type | Constraints |
|---|---|---|
| `user_id` | `INT` | `PRIMARY KEY` |
| `full_name` | `VARCHAR(100)` | `NOT NULL` |
| `email` | `VARCHAR(100)` | `UNIQUE`, `NOT NULL` |
| `role` | `VARCHAR(20)` | `CHECK (role IN ('Ticket Manager', 'Football Fan'))` |
| `phone_number` | `VARCHAR(20)` | — |

### 2. Matches Table

Catalogs tournament fixtures, categories, and pricing/status information.

| Column | Type | Constraints |
|---|---|---|
| `match_id` | `INT` | `PRIMARY KEY` |
| `fixture` | `VARCHAR(150)` | `NOT NULL` |
| `tournament_category` | `VARCHAR(50)` | — |
| `base_ticket_price` | `DECIMAL(10,2)` | `CHECK (base_ticket_price >= 0)` |
| `match_status` | `VARCHAR(20)` | `CHECK (match_status IN ('Available', 'Selling Fast', 'Sold Out', 'Postponed'))` |

### 3. Bookings Table

This transactional table records individual ticket purchases by linking users to their chosen matches.  

| Column | Type | Constraints |
|---|---|---|
| `booking_id` | `INT` | `PRIMARY KEY` |
| `user_id` | `INT` | `FOREIGN KEY → Users(user_id)` |
| `match_id` | `INT` | `FOREIGN KEY → Matches(match_id)` |
| `seat_number` | `VARCHAR(10)` | — |
| `payment_status` | `VARCHAR(20)` | `CHECK (payment_status IN ('Pending', 'Confirmed', 'Cancelled', 'Refunded'))` |
| `total_cost` | `DECIMAL(10,2)` | `CHECK (total_cost >= 0)` |

## Sample Data

**Users**

| user_id | full_name | email | role | phone_number |
|---|---|---|---|---|
| 1 | Tanvir Rahman | tanvir@mail.com | Football Fan | +8801711111111 |
| 2 | Asif Haque | asif@mail.com | Football Fan | +8801722222222 |
| 3 | Sajjad Rahman | sajjad@mail.com | Ticket Manager | +8801733333333 |
| 4 | Jannat Ara | jannat@mail.com | Football Fan | NULL |

**Matches**

| match_id | fixture | tournament_category | base_ticket_price | match_status |
|---|---|---|---|---|
| 101 | Real Madrid vs Barcelona | Champions League | 150 | Available |
| 102 | Man City vs Liverpool | Premier League | 120 | Selling Fast |
| 103 | Bayern Munich vs PSG | Champions League | 130 | Available |
| 104 | AC Milan vs Inter Milan | Serie A | 90 | Sold Out |
| 105 | Juventus vs Roma | Serie A | 80 | Available |

**Bookings**

| booking_id | user_id | match_id | seat_number | payment_status | total_cost |
|---|---|---|---|---|---|
| 501 | 1 | 101 | A-12 | Confirmed | 150 |
| 502 | 1 | 102 | B-04 | Confirmed | 120 |
| 503 | 2 | 101 | A-13 | Confirmed | 150 |
| 504 | 2 | 101 | NULL | NULL | 150 |
| 505 | 3 | 102 | C-20 | Pending | 120 |

## SQL Queries

All queries are implemented in `QUERY.sql`.

### Query 1 — Available Champions League Matches

Retrieve all upcoming football matches belonging to the 'Champions League' where the match status is 'Available'.

```sql
SELECT
  match_id,
  fixture,
  base_ticket_price
FROM
  Matches
WHERE
  tournament_category = 'Champions League'
  AND match_status = 'Available';
```

---

### Query 2 — Pattern-Matched User Search

Search for all users whose full names start with 'Tanvir' or contain the phrase 'Haque'

**Concepts used:** `LIKE`, `ILIKE`

```sql
SELECT
  user_id,
  full_name,
  email
FROM
  Users
WHERE
  full_name LIKE 'Tanvir%'
  OR full_name ILIKE '%Haque%';
```

---

### Query 3 — Bookings With Missing Payment Status

Retrieve all booking records where the payment status is missing (`NULL`), replacing the empty result with `Action Required`.

**Concepts used:** `IS NULL`, `COALESCE`

```sql
SELECT
  booking_id,
  user_id,
  match_id,
  COALESCE(payment_status, 'Action Required') AS systematic_status
FROM
  Bookings
WHERE
  payment_status IS NULL;
```

---

### Query 4 — Booking Details With User & Match Info

Retrieve match booking details along with the User's full name and the scheduled Match fixture teams.

**Concepts used:** `INNER JOIN`

```sql
SELECT
  b.booking_id,
  u.full_name,
  m.fixture,
  b.total_cost
FROM
  Bookings b
  INNER JOIN Users u ON b.user_id = u.user_id
  INNER JOIN Matches m ON b.match_id = m.match_id;
```

---

### Query 5 — All Users and Their Bookings

Display a comprehensive list of all users and their booking IDs, ensuring that fans who have never bought a ticket are still listed.

**Concepts used:** `LEFT JOIN`

```sql
SELECT
  u.user_id,
  u.full_name,
  b.booking_id
FROM
  Users u
  LEFT JOIN Bookings b ON u.user_id = b.user_id
```

---

### Query 6 — Above-Average Bookings

Find all ticket bookings where the total cost is strictly higher than the average cost of all ticket bookings.

**Concepts used:** Aggregate subquery (`AVG`)

```sql
SELECT
  booking_id,
  match_id,
  total_cost
FROM
  Bookings
WHERE
  total_cost > (
    SELECT
      AVG(total_cost)
    FROM
      Bookings
  );
```

---

### Query 7 — Second and Third Most Expensive Matches

Retrieve the top 2 most expensive matches sorted by base ticket price, skipping the absolute highest premium match.

**Concepts used:** `ORDER BY`, `LIMIT`, `OFFSET`

```sql
SELECT
  match_id,
  fixture,
  base_ticket_price
FROM
  Matches
ORDER BY
  base_ticket_price DESC
LIMIT
  2
OFFSET
  1;
```


## Tech Stack

- **Database:** PostgreSQL
- **Tools:** SQL DDL (table creation, constraints, foreign keys), DML (data seeding), DQL (querying)

## How to Run

1. Open the `QUERY.sql` file in your preferred SQL client .
2. Execute the script top to bottom — it will drop existing tables (if any), recreate the schema, seed sample data, and run all seven queries in order.
3. Review each query's output against the expected results documented above.

