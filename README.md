# 🍛 RasoiExpress — Full Stack Food Delivery App
### Flask Backend + Vanilla JS Frontend + SQLite Database


👤  ID    : admin123
🔑  Pass  : secure@123
---

## 📁 Project Structure

```
RasoiExpress/
│
├── backend/                    ← Flask REST API
│   ├── app.py                  ← Main entry point (run this)
│   ├── config.py               ← App settings (secret key, DB path, JWT expiry)
│   ├── database.py             ← SQLite setup + helper functions
│   ├── auth_utils.py           ← Password hashing (PBKDF2) + JWT tokens
│   ├── setup.py                ← One-click setup + DB seeding
│   ├── requirements.txt        ← Python dependencies
│   └── routes/
│       ├── auth.py             ← /api/auth  → signup, login, me, logout
│       ├── menu.py             ← /api/menu  → list dishes, categories, seed
│       ├── orders.py           ← /api/orders → place, track, list, cancel
│       └── profile.py          ← /api/profile → view, edit, change password
│
└── frontend/
    └── index.html              ← Single-page app (all HTML + CSS + JS)
```

---

## ⚡ Setup & Run (Step by Step)

### Step 1 — Prerequisites
```bash
# You need Python 3.9+ installed
python --version      # should print 3.9 or higher

# You need Flask and PyJWT
pip install Flask PyJWT
```

### Step 2 — One-click Setup
```bash
cd RasoiExpress/backend
python setup.py
```
This creates the SQLite database (`rasoi.db`) and seeds 45 dishes automatically.

### Step 3 — Start the Backend
```bash
cd RasoiExpress/backend
python app.py
```
You will see:
```
==================================================
  🍛  RasoiExpress API Server
==================================================
  📡  URL    : http://localhost:5000
  📁  DB     : backend/rasoi.db
==================================================
```

### Step 4 — Open the Frontend
**Option A — VS Code Live Server (recommended)**
1. Open `frontend/index.html` in VS Code
2. Right-click → "Open with Live Server"
3. App opens at `http://127.0.0.1:5500`

**Option B — Direct file**
1. Double-click `frontend/index.html`
2. All features work (backend must be running)

### Step 5 — Test It
1. Click **Login / Sign Up** in the navbar
2. Create an account with any email + password
3. Browse the menu, add items to cart
4. Place an order — it saves to the database
5. Watch live tracking auto-advance every 5 seconds

---

## 🔌 Complete API Reference

### Base URL: `http://localhost:5000/api`

---

### 🔐 Auth Endpoints

| Method | URL | Auth? | Description |
|--------|-----|-------|-------------|
| POST | `/auth/signup` | No | Register new user |
| POST | `/auth/login` | No | Login, get JWT token |
| GET | `/auth/me` | ✅ Yes | Get logged-in user profile |
| POST | `/auth/logout` | No | Logout (clears client token) |

**POST /auth/signup**
```json
// Request body
{ "name": "Priya Sharma", "email": "priya@example.com", "password": "secret123" }

// Response 201
{
  "message": "Welcome to RasoiExpress, Priya! 🎉",
  "token": "eyJhbGci...",
  "user": { "id": 1, "name": "Priya Sharma", "email": "priya@example.com", ... }
}
```

**POST /auth/login**
```json
// Request body
{ "email": "priya@example.com", "password": "secret123" }

// Response 200
{ "message": "Welcome back, Priya! 🎉", "token": "eyJhbGci...", "user": {...} }
```

---

### 🍽️ Menu Endpoints

| Method | URL | Auth? | Description |
|--------|-----|-------|-------------|
| GET | `/menu/items` | No | List all dishes (with filters) |
| GET | `/menu/items/<id>` | No | Get one dish by ID |
| GET | `/menu/categories` | No | List all categories |
| POST | `/menu/seed` | No | Seed 45 sample dishes (dev only) |

**GET /menu/items — Query Parameters**
```
?category=Veg Curries    filter by category
?type=veg                filter veg or nonveg
?search=paneer           keyword search
?sort=price-asc          popular / price-asc / price-desc / rating / newest
?max_price=300           price upper limit
```

---

### 📦 Order Endpoints  *(all require Authorization header)*

| Method | URL | Description |
|--------|-----|-------------|
| POST | `/orders/place` | Place a new order |
| GET | `/orders/my-orders` | List all your orders |
| GET | `/orders/<id>` | Get one order's details |
| PUT | `/orders/<id>/step` | Advance tracking step |
| DELETE | `/orders/<id>` | Cancel a pending order |

**POST /orders/place**
```json
// Request body
{
  "items": [
    { "id": 7, "name": "Butter Chicken", "price": 320, "qty": 2,
      "restaurant": "Tandoor House", "emoji": "🍗" }
  ],
  "address": "Flat 4B, MG Road, Mumbai 400050",
  "payment_method": "upi"
}
// Response 201
{
  "message": "Order RE-2024-AB12XY placed successfully! 🎉",
  "order": { "id": "RE-2024-AB12XY", "total": 689, "status": "placed", ... }
}
```

---

### 👤 Profile Endpoints  *(all require Authorization header)*

| Method | URL | Description |
|--------|-----|-------------|
| GET | `/profile` | Get full profile + order stats |
| PUT | `/profile` | Update name, phone, address, avatar, color |
| PUT | `/profile/password` | Change password |
| DELETE | `/profile` | Delete account permanently |

---

### 🔑 How to Send the Auth Token

Every protected endpoint needs this header:
```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

The frontend stores the token in `localStorage` after login/signup:
```javascript
localStorage.setItem('rasoiToken', data.token);
```

And sends it automatically with every protected API call.

---

## 🗄️ Database Schema (SQLite)

```sql
-- 4 tables total

users          id, name, email, password(hashed), phone, address,
               picture, profile_color, created_at

menu_items     id, name, description, price, category, type(veg/nonveg),
               restaurant, rating, image, emoji, is_spicy, is_new,
               is_best, time, available, created_at

orders         id(RE-YYYY-XXXX), user_id→users, items(JSON),
               total, restaurant, address, status, current_step,
               placed_at

cart_items     id, user_id→users, dish_id→menu_items, qty
```

---

## 🔒 Security Measures

| Feature | Implementation |
|---------|----------------|
| Password storage | PBKDF2-SHA256, 600,000 iterations, random salt |
| API authentication | JWT (HS256), 7-day expiry |
| Timing attack prevention | `hmac.compare_digest()` for password comparison |
| Token validation | Signature + expiry checked on every request |
| Input validation | All inputs validated before DB operations |
| SQL injection | 100% parameterized queries (`?` placeholders) |
| CORS | Controlled headers — only needed methods allowed |
| Error messages | Vague on auth failures (no "email not found" leak) |

---

## 📝 Viva Questions & Answers

**Q1: Why Flask over Django?**
Flask is a micro-framework — lightweight and flexible, perfect for a REST API.
Django has an ORM, admin panel, and templating built-in, which we don't need.
Flask lets us build exactly what we need without overhead.

**Q2: What is a JWT token and why use it?**
JWT (JSON Web Token) is a signed, self-contained token.
After login the server signs the user's ID into a token and sends it to the client.
The client sends this token with every API request in the `Authorization` header.
The server verifies the signature — no session or database lookup needed.
This makes it stateless and scalable.

**Q3: How is the password stored securely?**
We use PBKDF2-SHA256 (Password Based Key Derivation Function 2).
A random 16-byte salt is generated per password, mixed with the password,
then hashed 600,000 times. Even if the database is stolen, brute-forcing
is computationally infeasible. `hmac.compare_digest()` prevents timing attacks.

**Q4: What is a Blueprint in Flask?**
A Blueprint is a way to organize routes into separate modules.
Instead of putting all 20 routes in one file, we split them:
`auth_bp`, `menu_bp`, `orders_bp`, `profile_bp`.
Each blueprint is registered on the app with a URL prefix like `/api/auth`.

**Q5: What is REST API?**
REST (Representational State Transfer) is an architectural style for APIs.
Rules: use HTTP methods (GET=read, POST=create, PUT=update, DELETE=remove),
stateless requests, and JSON responses.
Example: `POST /api/orders/place` creates a new order.

**Q6: Why SQLite and not MySQL/PostgreSQL?**
SQLite is serverless — the entire database is a single `.db` file.
Zero installation, zero configuration. Perfect for development and small apps.
For production with multiple users you would switch to PostgreSQL.

**Q7: What does `@login_required` do?**
It is a Python decorator that wraps a route function.
Before running the actual route, it checks the `Authorization` header,
decodes the JWT, and either returns 401 (unauthorized) or injects the
decoded user info as `current_user` into the function.

**Q8: How does the frontend know the user is still logged in after refresh?**
On page load the frontend calls `restoreSession()`.
It reads the JWT from `localStorage` and calls `GET /api/auth/me`.
If the token is valid, the server returns the user profile and the
frontend restores `state.user` — the user appears logged in.

**Q9: What is CORS and why do we need it?**
CORS (Cross-Origin Resource Sharing) is a browser security policy.
The frontend is on `http://localhost:5500` and the API on `http://localhost:5000`.
These are different origins, so by default the browser blocks the request.
We add `Access-Control-Allow-Origin: *` headers in Flask's `after_request`
hook to tell the browser the API is safe to call from any origin.

**Q10: Explain the order lifecycle.**
```
placed (step 0) → preparing (step 1) → on_the_way (step 2)
  → nearby (step 3) → delivered (step 4)
```
Each step is stored as an integer in the `orders` table.
The frontend auto-advances the step every 5 seconds using `setInterval`.
In a real app, the delivery partner's mobile app would call
`PUT /api/orders/<id>/step` to trigger each advance.

---

## 🔧 Common Issues & Fixes

| Issue | Fix |
|-------|-----|
| `ModuleNotFoundError: flask` | Run `pip install Flask PyJWT` |
| CORS error in browser | Make sure `python app.py` is running on port 5000 |
| `rasoi.db not found` | Run `python setup.py` first |
| Login button does nothing | Open browser DevTools → Console for error details |
| Already seeded error | Safe to ignore — means dishes are already in DB |
| Port 5000 in use | Change port in `app.py`: `app.run(port=5001)` and update `API_BASE` in `index.html` |
