# Node.js, TypeScript & MySQL Boilerplate API

A fully functional authentication API built with TypeScript, Express, Sequelize (MySQL), and JWTs.

## Features

- **User Registration & Email Verification** — Users sign up and must verify via email link
- **JWT Authentication with Refresh Tokens** — Short-lived JWTs (15 min) + long-lived refresh tokens (7 days)
- **Refresh Token Rotation** — Each use of a refresh token issues a new one and revokes the old
- **Role-Based Access Control (RBAC)** — Admin and User roles
- **Account Management** — Forgot password, reset password, CRUD operations
- **Swagger UI** — API docs at `/api-docs`

## Project Structure

```
node-mysql-api/
├── _helpers/
│   ├── db.ts               # Sequelize + MySQL connection
│   ├── role.ts             # Role enum
│   ├── send-email.ts       # Nodemailer wrapper
│   └── swagger.ts          # Swagger UI router
├── _middleware/
│   ├── authorize.ts        # JWT auth + RBAC middleware
│   ├── error-handler.ts    # Global error handler
│   └── validate-request.ts # Joi request validation
├── accounts/
│   ├── account.model.ts         # Sequelize Account model
│   ├── refresh-token.model.ts   # Sequelize RefreshToken model
│   ├── account.service.ts       # Business logic
│   └── accounts.controller.ts   # Express routes
├── config.json             # DB, JWT secret, SMTP config
├── swagger.yaml            # OpenAPI 3.0 spec
├── server.ts               # Express app entry point
├── tsconfig.json
└── package.json
```

## Setup

### 1. Prerequisites
- Node.js 16+
- MySQL running locally (default port 3306)

### 2. Configure `config.json`
```json
{
    "database": {
        "host": "localhost",
        "port": 3306,
        "user": "root",
        "password": "YOUR_MYSQL_PASSWORD",
        "database": "node_mysql_api"
    },
    "secret": "YOUR_JWT_SECRET",
    "emailFrom": "info@your-domain.com",
    "smtpOptions": {
        "host": "smtp.ethereal.email",
        "port": 587,
        "auth": {
            "user": "YOUR_ETHEREAL_USER",
            "pass": "YOUR_ETHEREAL_PASS"
        }
    }
}
```

> **Tip:** Get free test SMTP credentials at [https://ethereal.email](https://ethereal.email)

### 3. Install dependencies
```bash
npm install
```

### 4. Run the server
```bash
# Development (with auto-reload)
npm run start:dev

# Production
npm start
```

The database and tables are created automatically on first run.

## API Endpoints

| Method | Route | Auth | Description |
|--------|-------|------|-------------|
| POST | `/accounts/register` | Public | Register new account |
| POST | `/accounts/verify-email` | Public | Verify email with token |
| POST | `/accounts/authenticate` | Public | Login, get JWT + refresh token |
| POST | `/accounts/refresh-token` | Cookie | Get new JWT + refresh token |
| POST | `/accounts/revoke-token` | JWT | Revoke a refresh token |
| POST | `/accounts/forgot-password` | Public | Request password reset email |
| POST | `/accounts/validate-reset-token` | Public | Validate reset token |
| POST | `/accounts/reset-password` | Public | Reset password |
| GET | `/accounts` | Admin | List all accounts |
| GET | `/accounts/:id` | JWT | Get account by ID |
| POST | `/accounts` | Admin | Create account |
| PUT | `/accounts/:id` | JWT | Update account |
| DELETE | `/accounts/:id` | JWT | Delete account |

## Swagger UI

Once running, visit: **http://localhost:4000/api-docs**

## Security Notes

- JWT tokens expire in **15 minutes**
- Refresh tokens expire in **7 days** and are stored as HTTP-Only cookies (XSS protection)
- Refresh token rotation prevents long-lived token abuse
- The first registered account is automatically assigned the **Admin** role
- All subsequent accounts get the **User** role

## Testing with Postman

1. **Register** → POST `/accounts/register` with user details
2. **Check email** (Ethereal inbox) for verification token
3. **Verify** → POST `/accounts/verify-email` with token
4. **Authenticate** → POST `/accounts/authenticate` to get JWT
5. **Use JWT** as `Bearer Token` in Authorization header for protected routes
