# DICP Insurance — Public Deployment Guide

This version is prepared for a real public URL using **Vercel (React/Vite)** and **Railway (Spring Boot + Python analytics + MySQL)**.

## Production architecture

- Frontend: Vercel (`frontend/`)
- Backend: Railway (`backend/`)
- Analytics / Python prediction: Railway (`salary_service/`)
- Database: Railway MySQL
- Uploaded profile/application/claim files: Railway Volume mounted at `/data`

## 1. Push this project to GitHub

Create a GitHub repository and push the `v5work` folder. Do **not** commit real `.env` files or passwords.

## 2. Create Railway MySQL

In Railway create a project, then **New -> Database -> MySQL**. Railway exposes `MYSQLHOST`, `MYSQLPORT`, `MYSQLUSER`, `MYSQLPASSWORD`, and `MYSQLDATABASE`. Keep the database private; the Spring Boot service can connect inside the Railway project.

## 3. Deploy Spring Boot backend on Railway

Create a Railway service from the same GitHub repository and set **Root Directory** to `/backend`. Railway will detect `backend/Dockerfile`.

Set these variables in the backend service:

```text
DB_HOST=${{MySQL.MYSQLHOST}}
DB_PORT=${{MySQL.MYSQLPORT}}
DB_NAME=${{MySQL.MYSQLDATABASE}}
DB_USER=${{MySQL.MYSQLUSER}}
DB_PASSWORD=${{MySQL.MYSQLPASSWORD}}
JWT_SECRET=<long-random-secret>
ADMIN_EMAIL=<your-admin-email>
ADMIN_PASSWORD=<strong-password>
FILE_STORAGE_DIR=/data/uploads
CORS_ALLOWED_ORIGINS=https://YOUR-FRONTEND.vercel.app
```

Attach a **Railway Volume** to the backend service and mount it at `/data`. This prevents uploaded profile images and supporting documents from disappearing after redeploys.

After deployment: **Settings -> Networking -> Generate Domain**. Your backend URL will look like:

```text
https://your-backend.up.railway.app
```

Health check:

```text
https://your-backend.up.railway.app/api/health
```

## 4. Deploy Python analytics on Railway

Create another service from the same GitHub repo and set **Root Directory** to `/salary_service`.

Set:

```text
SALARY_HOST=0.0.0.0
SALARY_ALLOWED_ORIGINS=https://YOUR-FRONTEND.vercel.app
ANALYTICS_MAX_BODY_BYTES=1048576
```

Railway provides `PORT` automatically and this project now reads it. Generate a public domain. Test:

```text
https://your-analytics.up.railway.app/health
```

## 5. Deploy React/Vite frontend on Vercel

Import the same GitHub repo in Vercel and set **Root Directory** to `frontend`. Vercel detects Vite.

Add environment variables before deploying:

```text
VITE_API_BASE_URL=https://your-backend.up.railway.app/api
VITE_SALARY_API_URL=https://your-analytics.up.railway.app
```

Also add EmailJS / Google variables only if your deployment uses those features.

`frontend/vercel.json` contains SPA routing fallback, so refreshing `/admin/dashboard`, `/customer/profile`, etc. continues to load the React app.

## 6. Put the final Vercel URL back into CORS

When Vercel gives your final URL, update both Railway services:

Backend:

```text
CORS_ALLOWED_ORIGINS=https://your-project.vercel.app
```

Analytics:

```text
SALARY_ALLOWED_ORIGINS=https://your-project.vercel.app
```

Redeploy/restart both services. For a custom domain, add that origin too, separated by a comma.

## 7. Import / migrate your existing MySQL data

If you need your existing XAMPP data in Railway MySQL, export the `insurance_portal` database from phpMyAdmin as SQL, then import it into Railway MySQL using a MySQL client. Apply `database/performance_15000.sql` after import for the large-dataset indexes.

Do not make the database public unless you specifically need external database access.

## Production checklist

- Change the default admin password before opening the site publicly.
- Use a long random `JWT_SECRET`.
- Never put DB passwords, JWT secrets, or private API keys in Vite variables; Vite variables are visible to browser users.
- Configure database backups.
- Keep uploaded files on the Railway Volume (`/data`).
- Use the Vercel HTTPS URL in backend and analytics CORS lists.
- Verify Login, Register, Plans, Applications, Payments, Claims, Feedback, AI assistant, Admin Reports, and Predictions after deployment.

## Local development still works

The existing local defaults remain available. You can still run:

```text
backend: mvn spring-boot:run
analytics: salary_service\run.bat
frontend: npm run dev
```

Or use `docker-compose.local.yml` if Docker Desktop is installed.
