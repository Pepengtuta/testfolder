# Deploying Ugyon to Your Own Hosting (Railway)

This guide walks you through running the Ugyon system on **your own** Railway account, step by step. Follow the steps in order.

---

## Before You Start (Prerequisites)

- A **Railway account** — sign up free at [railway.com](https://railway.com) (you can use your GitHub account).
- **Your own GitHub repository** containing this project's code.
- The **database file** — `database/Railway/capstone.sql` (already included in this repo).
- **HeidiSQL** (or any MySQL client) — for importing the database. Optional but recommended.
- The server uses **PHP 8.2 or newer**. Railway installs this automatically; you do not need to set it up.

---

## Step 1: Create a Railway Project

1. Go to [railway.com](https://railway.com) and sign in.
2. Click **+ New Project**. Keep this project open — you will add services to it.

---

## Step 2: Add a MySQL Database

1. Inside your project, click **+ Add Service** → **Database** → **MySQL**.
2. Wait until the MySQL service finishes deploying (watch the **Deployments** tab).

Railway automatically creates these variables on the MySQL service: `MYSQLHOST`, `MYSQLUSER`, `MYSQLPASSWORD`, `MYSQLDATABASE`, `MYSQLPORT`. You will use them in Step 5.

> The database is named **`railway`** by default. We will import the data into this `railway` database in Step 4.

---

## Step 3: Deploy the Code from GitHub

1. In the same project, click **+ Add Service** → **GitHub Repo**.
2. Authorize Railway to access your GitHub account.
3. Select the repository that contains this project, and choose the **`main`** branch.
4. Click **Deploy**.

Railway detects that this is a PHP project because of the `composer.json` and `railpack.json` files in the project root. It installs the PHP dependencies automatically.

---

## Step 4: Import the Database

Importing gives your deployment the tables and data it needs (users, barangays, reports, etc.).

### 4.1 — Turn on public access to MySQL

1. Open the **MySQL** service → **Settings** → **Networking**.
2. If needed, click **Public Access** (also called **TCP Proxy**) to enable it. This creates a public address like `xxxx.proxy.rlwy.net:PORT`.
3. Note the **host** and **port**.

### 4.2 — Get the database password

1. Open the **MySQL** service → **Variables** tab.
2. Copy the value of **`MYSQLPASSWORD`**. The username is **`root`**.

### 4.3 — Prepare the SQL file (important)

The included file forces the database name to `capstone`, but Railway's database is named `railway`. Before importing, **open `database/Railway/capstone.sql` in a text editor and delete these 3 lines** (near the top):

```sql
DROP DATABASE IF EXISTS `capstone`;
CREATE DATABASE IF NOT EXISTS `capstone` /*!40100 DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci */;
USE `capstone`;
```

Save the file. Now the tables will be created inside whichever database you have selected.

### 4.4 — Import with HeidiSQL

1. In HeidiSQL, create a new connection:
   - **Network type:** MySQL or MariaDB (TCP/IP)
   - **Hostname:** the proxy host from 4.1 (e.g. `xxxx.proxy.rlwy.net`)
   - **Port:** the proxy port from 4.1
   - **User:** `root`
   - **Password:** the `MYSQLPASSWORD` from 4.2
2. Click **Open**.
3. In the left panel, select (click) the **`railway`** database.
4. Go to **File → Run SQL file**, choose your edited `capstone.sql`, set encoding to **UTF-8**, and run it.
5. Press **F5** to refresh, then open a query tab and run:

   ```sql
   SELECT COUNT(*) FROM users;
   ```

   A number greater than 0 means the import succeeded.

### Alternative: Railway CLI

If you have the Railway CLI and `mysql` client installed, you can tunnel in instead of enabling public access:

```bash
railway connect mysql
```

Then, in the new connection, select the `railway` database and run the edited `capstone.sql`.

---

## Step 5: Connect the App to the Database

Tell the PHP app how to reach the database by adding variables to the **GitHub service** (the service running your code).

1. Open your **GitHub service** in Railway.
2. Go to the **Variables** tab.
3. Add the following variables. When you type `${{`, Railway suggests values from the MySQL service — select them. **Do not type the password by hand.**

| Variable Name | Value |
|---------------|-------|
| `DB_HOST` | `${{MySQL.MYSQLHOST}}` |
| `DB_USER` | `${{MySQL.MYSQLUSER}}` |
| `DB_PASS` | `${{MySQL.MYSQLPASSWORD}}` |
| `DB_NAME` | `${{MySQL.MYSQLDATABASE}}` |
| `DB_PORT` | `${{MySQL.MYSQLPORT}}` |

4. **(Optional but recommended) Evidence uploads.** The system stores incident photos/videos on Cloudinary. Without this, uploads are saved to the server's temporary disk and are **lost every time the app redeploys**. Add:

   | Variable Name | Value |
   |---------------|-------|
   | `CLOUDINARY_URL` | `cloudinary://API_KEY:API_SECRET@CLOUD_NAME` |

   Get these from your own [Cloudinary](https://cloudinary.com) account (free plan).

5. Click **Save**. Railway **automatically redeploys** the app with the new settings.

> You do **not** need to edit any code. The app already reads these variables at runtime. For local development it reads them from a `.env` file instead (see `.env.example`).

---

## Step 6: Get Your Public URL

1. Open your **GitHub service** → **Settings** → **Networking**.
2. Under **Public Networking**, click **Generate Domain**.
3. Railway creates a free address like `yourservice-production.up.railway.app`.

---

## Step 7: Log In and Test

1. Open `https://your-domain/public/login.php` in your browser.
   - Note: opening just `https://your-domain/` shows a **404** — that is expected. The login page lives under `/public/`.
2. Log in with an account from the imported database (use the test credentials provided by the developer).
3. If login succeeds, you are taken to the dashboard.

If login fails with **"Invalid username or password"**:
- Confirm the database was imported: run `SELECT COUNT(*) FROM users;` — it should return more than 0.
- Confirm you imported into the **`railway`** database (see Step 4).

---

## Step 8 (Optional): Point the Flutter App to Your Hosting

If you use the Flutter mobile app, point it at your new Railway domain. The cleanest way is a build-time flag (no file edits):

```bash
flutter build apk --dart-define=API_BASE_URL=https://your-domain
```

Or edit the default in `flutter/flutter_saster/lib/constants/api_config.dart`:

```dart
static const String _root = String.fromEnvironment(
  'API_BASE_URL',
  defaultValue: 'https://your-domain',
);
```

Then rebuild and test the app on your phone.

---

## Cleanup

1. **Optional:** Once everything works, disable the MySQL **Public Access** again (MySQL → Settings → Networking). The app uses an internal connection, so the public proxy is not needed. Re-enable it only when you want to connect from outside.
2. **Billing:** Check your Railway billing/usage page. The free trial has an end date; after it, Railway charges for usage. A capstone project with light traffic costs very little.

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| **PHP is not detected** (build says "Detected Staticfile") | Make sure `composer.json` and `railpack.json` exist in the project root. |
| **404 on every URL** | Make sure the **Root Directory** setting on the GitHub service is empty (not `/public/` or similar). Then open `/public/login.php`. |
| **Database connection error** | Check that all five variables in Step 5 are set on the **GitHub service** (not just the MySQL service), then redeploy. |
| **Login fails / "Invalid username or password"** | The database was not imported into the right place. Re-import into the **`railway`** database after removing the 3 `capstone` lines (Step 4.3). Verify with `SELECT COUNT(*) FROM users;`. |
| **"No database selected" during import** | Select the `railway` database in HeidiSQL *before* running the SQL file. |
| **Photos/videos disappear after a redeploy** | Uploads are stored on temporary disk. Add `CLOUDINARY_URL` (Step 5.4) so evidence is saved to Cloudinary. |
| **Phone app cannot connect** | The app must point at the Railway URL, not `localhost`. See Step 8. |

---

## Support

- [Railway Docs](https://docs.railway.com)
- [Railpack PHP Guide](https://railpack.com/languages/php)
- [Railway Help Station](https://station.railway.com)

---

**Last updated:** October 2026
