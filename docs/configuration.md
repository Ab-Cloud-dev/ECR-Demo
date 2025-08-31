Configuration and Security Guidance
===================================

Overview
--------

Both Flask services contain inline defaults for convenience. For any real environment, replace hardcoded values with environment variables or a secret manager.

Environment Variables
---------------------

Recommended variables (names are suggestions):

- MONOLITH_DEFAULT_USERNAME
- MONOLITH_DEFAULT_PASSWORD
- DB_HOST
- DB_PORT (e.g., 5432)
- DB_USER
- DB_PASSWORD
- DB_NAME

Example `.env` (do not commit):

```bash
MONOLITH_DEFAULT_USERNAME=admin
MONOLITH_DEFAULT_PASSWORD=change-me
DB_HOST=localhost
DB_PORT=5432
DB_USER=postgres
DB_PASSWORD=change-me
DB_NAME=microservice
```

Loading vars in Flask
---------------------

Use `python-dotenv` or `os.environ` directly:

```python
from os import environ

DEFAULT_USERNAME = environ.get("MONOLITH_DEFAULT_USERNAME", "admin")
DEFAULT_PASSWORD = environ.get("MONOLITH_DEFAULT_PASSWORD", "password123")

db_host = environ.get("DB_HOST", "localhost")
db_port = environ.get("DB_PORT", "5432")
db_user = environ.get("DB_USER", "postgres")
db_password = environ.get("DB_PASSWORD")
db_name = environ.get("DB_NAME", "microservice")
```

Security Best Practices
-----------------------

- Never commit real credentials or secrets to source control.
- Store secrets in a vault (AWS Secrets Manager, HashiCorp Vault) or CI/CD secret store.
- Enforce TLS everywhere; avoid plain HTTP for credentials.
- Use password hashing (bcrypt/argon2) and never store plaintext passwords.
- Implement CSRF protection and secure session cookies (HttpOnly, Secure, SameSite).
- Validate and sanitize all inputs; use parameterized SQL queries.
- Restrict DB user privileges to the minimum required.

Docker and Deployment
---------------------

- Pass environment variables at runtime: `docker run -e DB_HOST=... -e DB_PASSWORD=... ...`.
- For Kubernetes, use `Secrets` and `ConfigMaps` with appropriate RBAC.
- Rotate credentials regularly and monitor for leaked secrets.

