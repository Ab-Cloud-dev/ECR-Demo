Monolith Service API and Usage
==============================

Overview
--------

Flask application providing a simple login flow, product landing, and order submission pages. Templates are served from `templates/` with static assets in `static/`.

- App module: `monolith-aws-microservice-project/app.py`
- Helper: `monolith-aws-microservice-project/helper_scripts/db_script.py`
- Default credentials: username `admin`, password `password123` (demo only)

Base URL
--------

- Local: `http://localhost:5000`
- Container: `http://<container-host>:5000`

Endpoints
---------

1) `GET /` (Login page)

- Renders `login.html` and optionally displays an `error` message when credentials are invalid.
- No parameters.

2) `POST /` (Login submit)

- Form fields:
  - `username` (string, required)
  - `password` (string, required)
- Behavior:
  - If `username == admin` and `password == password123`, redirects to `/welcomepage`.
  - Otherwise, re-renders `login.html` with an error message.
- Example:

```bash
curl -i -X POST \
  -d "username=admin" \
  -d "password=password123" \
  http://localhost:5000/
```

3) `GET /welcomepage`

- Renders `product.html` (landing page after successful login).
- Example:

```bash
curl -i http://localhost:5000/welcomepage
```

4) `GET /place_order`

- Query parameters:
  - `product` (string, optional): product identifier to pre-fill order.
- Renders `place_order.html` with `product_id` injected.
- Example:

```bash
curl -i "http://localhost:5000/place_order?product=SKU-123"
```

5) `POST /submit_order`

- Form fields:
  - `product_id` (string, required)
  - `name` (string, required)
  - `address` (string, required)
  - `quantity` (integer as string, required)
- Behavior:
  - Processes order (placeholder), then renders `order_confirmation.html` with submitted fields.
- Example:

```bash
curl -i -X POST http://localhost:5000/submit_order \
  -d "product_id=SKU-123" \
  -d "name=Alice" \
  -d "address=221B Baker St" \
  -d "quantity=2"
```

6) `GET /external_service`

- Renders `external_service.html` (placeholder for an external integration).
- Example:

```bash
curl -i http://localhost:5000/external_service
```

Templates
---------

- `templates/login.html`: Login form posting to `/` with fields `username`, `password`.
- `templates/product.html`: Landing page after successful login.
- `templates/place_order.html`: Order form with fields `product_id`, `name`, `address`, `quantity` that posts to `/submit_order`.
- `templates/order_confirmation.html`: Confirmation view showing submitted order details.
- `templates/external_service.html`: Placeholder page.

Helper: Database Bootstrap Script
---------------------------------

File: `helper_scripts/db_script.py`

- Public function: `create_table_clear_and_insert_default()`
  - Creates `users` table if not exists with columns `(id SERIAL PRIMARY KEY, username VARCHAR(255), password VARCHAR(255))`.
  - Deletes existing rows and inserts a default user `(user, user)`.
  - Prints table structure and data to stdout.
- Usage:

```bash
python monolith-aws-microservice-project/helper_scripts/db_script.py
```

Notes and Best Practices
------------------------

- Default credentials are for demonstration only; replace with real auth.
- Use environment variables or a secret manager for database configuration.
- Consider CSRF protection for form submissions and input validation.

