Login Microservice API and Usage
===============================

Overview
--------

Flask microservice handling login via a PostgreSQL-backed `users` table and redirecting to a product page upon success. Logging is configured at DEBUG level.

- App module: `login-microservice-app/app.py`
- Optional helper: `login-microservice-app/helper_script.py` (initializes demo data)

Base URL
--------

- Local: `http://localhost:5000`

Database
--------

- Connection parameters are defined in code (replace with environment variables for production): `db_host`, `db_port`, `db_user`, `db_password`, `db_name`.
- Table: `users` with columns `(id SERIAL PRIMARY KEY, name VARCHAR(100), email VARCHAR(100) UNIQUE, password VARCHAR(100))`.

Endpoints
---------

1) `GET /` (Login page)

- Renders `login.html`.

2) `POST /login` (Authenticate)

- Form fields:
  - `email` (string, required)
  - `password` (string, required)
- Behavior:
  - Queries `users` for a row with matching `email` and `password`.
  - On success, redirects to `/product`.
  - On failure or DB error, redirects back to `/`.
- Example:

```bash
curl -i -X POST http://localhost:5000/login \
  -d "email=alice_dummy@example.com" \
  -d "password=password123_dummy"
```

3) `GET /product` (Redirect to Product)

- Redirects to the monolith service product landing page
  `http://crypto-app-882103207.eu-central-1.elb.amazonaws.com:5000/welcomepage`.
- Example:

```bash
curl -i http://localhost:5000/product
```

Helper Script
-------------

File: `login-microservice-app/helper_script.py`

- Initializes the `users` table and inserts three demo users.
- Usage (for local/dev only):

```bash
python login-microservice-app/helper_script.py
```

Notes and Best Practices
------------------------

- Replace hardcoded DB credentials with environment variables or a secret manager.
- Hash and salt passwords instead of storing plaintext.
- Use parameterized queries (already used) and validate inputs.
- Add CSRF protection for forms and secure cookies/sessions for logged-in state.

