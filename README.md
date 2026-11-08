# Lab 3: REST API, Query Parameters, Validation & Debugging

**Backend Development Fundamentals (BDF)** — Year II Term II — Aurora  
**Faculty:** C. Sai Shiva Teja  
**Student Name:** Karthika Kasthuri  
**Registration Number:** 252U1R5066  
**Section / Class:** AIDE 2A  

---

## 📋 Table of Contents
1. [Project Overview](#-project-overview)
2. [Technologies Used](#-technologies-used)
3. [Project Directory Structure](#-project-directory-structure)
4. [Database Setup & Schema](#-database-setup--schema)
5. [Installation & Setup](#-installation--setup)
6. [Running the Application](#-running-the-application)
7. [REST API Endpoints](#-rest-api-endpoints)
8. [Query Parameters & Filters (GET /books)](#-query-parameters--filters-get-books)
9. [Request Validation Rules](#-request-validation-rules)
10. [HTTP Status Codes & Response Formats](#-http-status-codes--response-formats)
11. [Debugging Guide (4 Common Errors)](#-debugging-guide-4-common-errors)
12. [Postman Testing Guide (16 Test Cases)](#-postman-testing-guide-16-test-cases)
13. [Automated Testing](#-automated-testing)
14. [Viva Examination Questions & Answers](#-viva-examination-questions--answers)
15. [Security & Version Control Best Practices](#-security--version-control-best-practices)

---

## 📌 Project Overview

This project converts an earlier **Lab 2** backend into a production-grade, clean **RESTful API** for managing a collection of books. It features:
- **Noun-based RESTful routing** (`/books`).
- **Dynamic query parameters** for search (`search_q` with SQL `LIKE`), multi-column sorting (`order_by`, `order`), and pagination (`limit`, `offset`).
- **Price range filtering** (`min_price`, `max_price`) utilizing SQL `BETWEEN` and comparison operators.
- **Strict request validation** with descriptive error messages and HTTP `400 Bad Request`.
- **Proper HTTP status codes** (`200 OK`, `201 Created`, `400 Bad Request`, `404 Not Found`, `500 Internal Server Error`).
- **SQL Injection Prevention** by whitelisting sort columns/directions and using parameterized queries (`?`).
- **Asynchronous database operations** with `sqlite` and `sqlite3` using `async/await`.
- **Debugging solutions** for common developer pitfalls (`EADDRINUSE`, wrong imports, table name typos, missing `await`).

---

## 🛠 Technologies Used

- **Node.js** (v18+) - JavaScript runtime environment.
- **Express.js** (v5) - Fast, minimalist web framework.
- **SQLite3 & sqlite** - File-based SQL relational database engine with promise-based async/await support.
- **dotenv** - Environment variable management.
- **Postman** - API testing and validation.

---

## 📁 Project Directory Structure

```text
lab-3-rest-api/
├── config/
│   └── db.js                 # SQLite database initialization & auto-seeding
├── controllers/
│   └── bookController.js     # Business logic, query construction, and validation
├── routes/
│   └── bookRoutes.js         # RESTful router endpoints for /books
├── .env.example              # Template environment variables
├── .gitignore                # Excludes node_modules, .env, and .db files
├── DEBUGGING_GUIDE.md        # Detailed breakdown of the 4 debugging problems
├── package.json              # Project scripts and dependencies
├── server.js                 # Server entry point, middleware, and error handlers
├── test-api.js               # 16-point automated test suite
└── README.md                 # Complete documentation (this file)
```

---

## 🗄 Database Setup & Schema

The application uses an embedded **SQLite** database (`books.db`). The database schema and initial seed data are managed automatically on server start via `config/db.js`.

### Table Schema: `books`
```sql
CREATE TABLE IF NOT EXISTS books (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title TEXT NOT NULL,
    author TEXT NOT NULL,
    price REAL NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

### Auto-Seeded Data
When the database starts for the first time, 8 sample books are automatically seeded if the table is empty:

| # | Title | Author | Price |
|---|-------|--------|-------|
| 1 | The Great Gatsby | F. Scott Fitzgerald | ₹350.00 |
| 2 | To Kill a Mockingbird | Harper Lee | ₹420.00 |
| 3 | 1984 | George Orwell | ₹299.00 |
| 4 | Pride and Prejudice | Jane Austen | ₹250.00 |
| 5 | The Catcher in the Rye | J.D. Salinger | ₹380.00 |
| 6 | The Hobbit | J.R.R. Tolkien | ₹499.00 |
| 7 | Clean Code | Robert C. Martin | ₹850.00 |
| 8 | JavaScript: The Good Parts | Douglas Crockford | ₹620.00 |

---

## ⚙️ Installation & Setup

### Prerequisites
- **Node.js** v18 or higher installed ([nodejs.org](https://nodejs.org))
- **npm** (bundled with Node.js)

### Steps

**1. Clone or download the project:**
```bash
cd "lab 3 bds"
```

**2. Install dependencies:**
```bash
npm install
```

**3. Configure environment variables:**
```bash
# Copy the example file and edit if needed
copy .env.example .env
```

The `.env` file defaults:
```
PORT=3000
DB_FILE=./books.db
```

---

## ▶️ Running the Application

### Start the server:
```bash
npm start
```

### Start with hot-reload (development):
```bash
npm run dev
```

**Expected console output:**
```
Database initialized and seeded with 8 sample books.
====================================================
 Books REST API is running!
 Student: Karthika Kasthuri (252U1R5066) | AIDE 2A
 Base URL: http://localhost:3000
 Try: http://localhost:3000/books
====================================================
```

---

## 🔗 REST API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/` | Welcome message & API info |
| `GET` | `/books` | Get all books (supports filters, sorting, pagination) |
| `GET` | `/books/:id` | Get a single book by ID |
| `POST` | `/books` | Create a new book |
| `PUT` | `/books/:id` | Update an existing book |
| `DELETE` | `/books/:id` | Delete a book |

---

## 🔍 Query Parameters & Filters (GET /books)

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `search_q` | string | `''` | Partial title search (SQL `LIKE %value%`) |
| `order_by` | string | `id` | Sort column: `id`, `title`, `author`, `price`, `created_at` |
| `order` | string | `ASC` | Sort direction: `ASC` or `DESC` |
| `limit` | integer | `10` | Max records per page (must be > 0) |
| `offset` | integer | `0` | Records to skip for pagination (must be >= 0) |
| `min_price` | float | — | Minimum price filter (inclusive) |
| `max_price` | float | — | Maximum price filter (inclusive) |

### Example URLs

```
GET /books                                   → All books (default: id ASC, limit 10)
GET /books?search_q=Gatsby                   → Search by title containing "Gatsby"
GET /books?order_by=price&order=DESC         → Sort by price, highest first
GET /books?limit=3&offset=2                  → Page 2 with 3 books per page
GET /books?min_price=300&max_price=500       → Books priced between ₹300 and ₹500
GET /books?min_price=400                     → Books priced ₹400 or more
GET /books?max_price=400                     → Books priced ₹400 or less
```

---

## ✅ Request Validation Rules

### POST /books & PUT /books/:id (Request Body)

| Field | Type | Rule |
|-------|------|------|
| `title` | string | Required, non-empty string |
| `author` | string | Required, non-empty string |
| `price` | number | Required, valid non-negative number |

**Example valid request body:**
```json
{
  "title": "Clean Architecture",
  "author": "Robert C. Martin",
  "price": 750.00
}
```

---

## 📊 HTTP Status Codes & Response Formats

| Status Code | Meaning | When Used |
|-------------|---------|-----------|
| `200 OK` | Success | GET, PUT, DELETE operations succeed |
| `201 Created` | Resource created | POST /books succeeds |
| `400 Bad Request` | Validation failure | Missing/invalid fields, bad query params |
| `404 Not Found` | Resource missing | Book ID doesn't exist; unknown endpoint |
| `500 Internal Server Error` | Server crash | Unhandled exceptions |

### Response Format Examples

**Success (200):**
```json
{
  "message": "Books retrieved successfully",
  "count": 8,
  "data": [
    { "id": 1, "title": "The Great Gatsby", "author": "F. Scott Fitzgerald", "price": 350, "created_at": "..." }
  ]
}
```

**Validation Error (400):**
```json
{
  "message": "Title is required and must be a non-empty string"
}
```

**Not Found (404):**
```json
{
  "message": "Book not found"
}
```

---

## 🐛 Debugging Guide (4 Common Errors)

See the full details in [DEBUGGING_GUIDE.md](./DEBUGGING_GUIDE.md).

| # | Error | Cause | Fix |
|---|-------|-------|-----|
| 1 | `EADDRINUSE` | Port 3000 already occupied | Use `process.env.PORT`, graceful error handler |
| 2 | `Cannot find module` | Wrong relative import path | Use `'../config/db'` from controllers folder |
| 3 | `no such table: book` | Table name typo (singular vs plural) | Standardize all SQL to use `books` |
| 4 | Returns `undefined` / empty | Missing `await` on async DB calls | Add `await` before all `db.all()`, `db.get()`, `db.run()` |

---

## 🧪 Postman Testing Guide (16 Test Cases)

### Test 1: Get All Books
- **Method:** `GET`
- **URL:** `http://localhost:3000/books`
- **Expected Status:** `200 OK`
- **Expected Response:** Array of books with `count` and `data`

### Test 2: Search Books by Title
- **Method:** `GET`
- **URL:** `http://localhost:3000/books?search_q=Gatsby`
- **Expected Status:** `200 OK`
- **Expected Response:** Books where title contains "Gatsby"

### Test 3: Sort Books Ascending (by Price)
- **Method:** `GET`
- **URL:** `http://localhost:3000/books?order_by=price&order=ASC`
- **Expected Status:** `200 OK`
- **Expected Response:** Books sorted by price lowest to highest

### Test 4: Sort Books Descending (by Price)
- **Method:** `GET`
- **URL:** `http://localhost:3000/books?order_by=price&order=DESC`
- **Expected Status:** `200 OK`
- **Expected Response:** Books sorted by price highest to lowest

### Test 5: Pagination (Limit and Offset)
- **Method:** `GET`
- **URL:** `http://localhost:3000/books?limit=3&offset=2`
- **Expected Status:** `200 OK`
- **Expected Response:** At most 3 books, skipping first 2

### Test 6: Filter by Minimum Price
- **Method:** `GET`
- **URL:** `http://localhost:3000/books?min_price=400`
- **Expected Status:** `200 OK`
- **Expected Response:** Books with price >= 400

### Test 7: Filter by Maximum Price
- **Method:** `GET`
- **URL:** `http://localhost:3000/books?max_price=400`
- **Expected Status:** `200 OK`
- **Expected Response:** Books with price <= 400

### Test 8: Filter by Price Range (Min & Max)
- **Method:** `GET`
- **URL:** `http://localhost:3000/books?min_price=300&max_price=500`
- **Expected Status:** `200 OK`
- **Expected Response:** Books priced between 300 and 500

### Test 9: Create Book — Valid Data
- **Method:** `POST`
- **URL:** `http://localhost:3000/books`
- **Headers:** `Content-Type: application/json`
- **Body:**
  ```json
  {
    "title": "Node.js Design Patterns",
    "author": "Mario Casciaro",
    "price": 599.00
  }
  ```
- **Expected Status:** `201 Created`
- **Expected Response:** `{"message": "Book created successfully", "data": {...}}`

### Test 10: Create Book — Missing Title
- **Method:** `POST`
- **URL:** `http://localhost:3000/books`
- **Body:**
  ```json
  { "author": "Anonymous", "price": 299.00 }
  ```
- **Expected Status:** `400 Bad Request`
- **Expected Response:** `{"message": "Title is required and must be a non-empty string"}`

### Test 11: Create Book — Missing Author
- **Method:** `POST`
- **URL:** `http://localhost:3000/books`
- **Body:**
  ```json
  { "title": "Nameless Chronicles", "price": 199.00 }
  ```
- **Expected Status:** `400 Bad Request`
- **Expected Response:** `{"message": "Author is required and must be a non-empty string"}`

### Test 12: Create Book — Invalid Price
- **Method:** `POST`
- **URL:** `http://localhost:3000/books`
- **Body:**
  ```json
  { "title": "Free Knowledge", "author": "Open Source", "price": "invalid_price_string" }
  ```
- **Expected Status:** `400 Bad Request`
- **Expected Response:** `{"message": "Price is required and must be a valid non-negative number"}`

### Test 13: Update Existing Book
- **Method:** `PUT`
- **URL:** `http://localhost:3000/books/1`
- **Body:**
  ```json
  {
    "title": "Node.js Design Patterns - 3rd Edition",
    "author": "Mario Casciaro & Luciano Mammino",
    "price": 720.00
  }
  ```
- **Expected Status:** `200 OK`
- **Expected Response:** `{"message": "Book updated successfully", "data": {...}}`

### Test 14: Update Nonexistent Book
- **Method:** `PUT`
- **URL:** `http://localhost:3000/books/999999`
- **Body:**
  ```json
  { "title": "Ghost Book", "author": "Nobody", "price": 500.00 }
  ```
- **Expected Status:** `404 Not Found`
- **Expected Response:** `{"message": "Book not found"}`

### Test 15: Delete Existing Book
- **Method:** `DELETE`
- **URL:** `http://localhost:3000/books/1`
- **Expected Status:** `200 OK`
- **Expected Response:** `{"message": "Book deleted successfully"}`

### Test 16: Delete Nonexistent Book
- **Method:** `DELETE`
- **URL:** `http://localhost:3000/books/999999`
- **Expected Status:** `404 Not Found`
- **Expected Response:** `{"message": "Book not found"}`

---

## 🤖 Automated Testing

An automated test script `test-api.js` is included to execute all 16 test cases against the live server:

```bash
npm run test:api
```

**Expected Output:**
```text
===============================================================
 Starting Lab 3 REST API Automated Test Suite (16 Test Cases)
 Student: Karthika Kasthuri (252U1R5066) | AIDE 2A
===============================================================

Test server running on port 3099

[PASS] Test 1: Get all books (GET /books)
[PASS] Test 2: Search books by title (GET /books?search_q=Gatsby)
[PASS] Test 3: Sort books in ascending order (GET /books?order_by=price&order=ASC)
[PASS] Test 4: Sort books in descending order (GET /books?order_by=price&order=DESC)
[PASS] Test 5: Pagination using limit and offset (GET /books?limit=3&offset=2)
[PASS] Test 6: Filter by minimum price (GET /books?min_price=400)
[PASS] Test 7: Filter by maximum price (GET /books?max_price=400)
[PASS] Test 8: Filter by price range (GET /books?min_price=300&max_price=500)
[PASS] Test 9: Create a book with valid data (POST /books)
[PASS] Test 10: Create a book with missing title (POST /books)
[PASS] Test 11: Create a book with missing author (POST /books)
[PASS] Test 12: Create a book with invalid price (POST /books)
[PASS] Test 13: Update an existing book (PUT /books/9)
[PASS] Test 14: Update a nonexistent book (PUT /books/999999)
[PASS] Test 15: Delete an existing book (DELETE /books/9)
[PASS] Test 16: Delete a nonexistent book (DELETE /books/999999)

===============================================================
 Test Summary: 16 PASSED, 0 FAILED out of 16 tests
===============================================================
```

---

## ❓ Viva Examination Questions & Answers

### Q1: What is the difference between a query parameter and a path parameter?
- **Path Parameter (`/books/:id`):** Part of the URL path itself used to identify a **specific resource** (e.g., `/books/5` uniquely points to the book with ID 5).
- **Query Parameter (`/books?search_q=Gatsby&limit=10`):** Added after the `?` delimiter in the URL as key-value pairs used to **filter, sort, search, or paginate** a collection without altering the resource endpoint identity.

### Q2: What does "Stateless" mean in RESTful API architecture?
- **Statelessness** means that every HTTP request from a client must contain **all the context and information** needed for the server to understand and process it.
- The server does not store client session state between requests in memory. Each request is independent and self-contained.

### Q3: Which HTTP status code should be returned for bad input vs a missing record?
- **Bad Input:** `400 Bad Request` — Used when client data fails validation (e.g., missing title, negative price, invalid parameters).
- **Missing Record:** `404 Not Found` — Used when client requests a valid operation on a resource that does not exist in the database (e.g., requesting or deleting ID `999999`).

### Q4: Why must we use parameterized queries instead of string concatenation in SQL?
- String concatenation (`"SELECT * FROM books WHERE title = '" + input + "'"`) is vulnerable to **SQL Injection Attacks**, allowing an attacker to inject malicious SQL commands (e.g., `' OR '1'='1`).
- Parameterized queries (`SELECT * FROM books WHERE title = ?`, `[input]`) treat user input strictly as data values, never as executable SQL code.

### Q5: Why is `await` necessary when interacting with SQLite in Node.js?
- Database queries involve disk I/O and are asynchronous. They return a JavaScript `Promise`.
- Without `await`, JavaScript executes the next line immediately before the query has finished, leading to unhandled promises, undefined values, or empty responses.

---

## 🔐 Security & Version Control Best Practices

1. **`.gitignore` configured:**
   - `node_modules/` is excluded to keep repository size small.
   - `.env` is excluded to protect private configuration and secrets.
   - `*.db` is excluded to avoid committing local binary database files.
2. **SQL Injection Prevention:**
   - Whitelist validation for dynamic table columns and sorting directions.
   - Parameterized SQL placeholders (`?`) for all user-supplied search and filter queries.
3. **Graceful Error Handling:**
   - Production error messages never expose database credentials or internal stack traces to the client.

