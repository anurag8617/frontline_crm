# 🏝️ Frontline Island Bar & Grill — CRM

Production-ready CRM for Frontline Island Bar & Grill.  
Built with **Node.js + Express + MongoDB Atlas + Nodemailer**.  
Frontend served as static files from the same Express server.

---

## 📋 Project Structure

```
frontline-crm/
├── backend/
│   ├── config/
│   │   └── seed.js          ← Seeds 146 leads + 2 users on first boot
│   ├── middleware/
│   │   └── auth.js          ← JWT protection middleware
│   ├── models/
│   │   ├── Lead.js
│   │   ├── User.js
│   │   └── Other.js         ← EmailLog, CustomModule, Game
│   ├── routes/
│   │   ├── auth.js          ← POST /api/auth/login, GET /api/auth/me
│   │   ├── leads.js         ← CRUD + bulk import
│   │   ├── email.js         ← Send + log emails
│   │   └── data.js          ← Dashboard stats, modules, games
│   ├── server.js            ← Main entry point
│   ├── package.json
│   └── .env.example
├── frontend/
│   └── index.html           ← Complete single-page app
├── .env.example
├── .gitignore
├── netlify.toml
└── README.md
```

---

## 🔐 Default Credentials

| User | Password | Role |
|------|----------|------|
| `ravi` | `Frontline2026!@#$%` | Admin |
| `pam`  | `Frontline@2026!@#$%` | User |

---

## 🚀 DEPLOYMENT GUIDE

### Option A: Render (Backend) + Netlify (Frontend) — RECOMMENDED

This is the setup for your existing `frontlinecateringcrm.netlify.app`.

---

### STEP 1 — Set Up MongoDB Atlas

1. Go to [mongodb.com/atlas](https://mongodb.com/atlas) → **Create free account**
2. Create a **free M0 cluster** (choose AWS us-east-1 for lowest latency to Render)
3. **Database Access** → Add a database user:
   - Username: `frontline-admin`
   - Password: generate a strong password (save it!)
   - Role: `Atlas admin`
4. **Network Access** → Add IP Address → **Allow access from anywhere** (`0.0.0.0/0`)
   - (Required for Render's dynamic IPs)
5. **Connect** → Connect your application → Copy the connection string:
   ```
   mongodb+srv://frontline-admin:<password>@cluster0.xxxxx.mongodb.net/?retryWrites=true&w=majority
   ```
6. Replace `<password>` with your actual password and add the database name:
   ```
   mongodb+srv://frontline-admin:YOURPASS@cluster0.xxxxx.mongodb.net/frontline_crm?retryWrites=true&w=majority
   ```
   **Save this — you'll need it in Step 2.**

---

### STEP 2 — Deploy Backend on Render

1. Go to [render.com](https://render.com) → **New** → **Web Service**
2. Connect your GitHub repository (push this project to GitHub first)
3. Configure the service:
   | Setting | Value |
   |---------|-------|
   | **Name** | `frontline-crm-api` |
   | **Root Directory** | `backend` |
   | **Runtime** | `Node` |
   | **Build Command** | `npm install` |
   | **Start Command** | `node server.js` |
   | **Instance Type** | Free |

4. **Environment Variables** — Add ALL of these:

   | Key | Value |
   |-----|-------|
   | `NODE_ENV` | `production` |
   | `MONGO_URI` | Your Atlas connection string from Step 1 |
   | `JWT_SECRET` | Run `node -e "console.log(require('crypto').randomBytes(64).toString('hex'))"` and paste result |
   | `JWT_EXPIRES_IN` | `7d` |
   | `SMTP_HOST` | `smtp.gmail.com` |
   | `SMTP_PORT` | `587` |
   | `SMTP_SECURE` | `false` |
   | `SMTP_USER` | `frontlineislandbarandgrill1@gmail.com` |
   | `SMTP_PASS` | Your Gmail App Password (see Step 3) |
   | `SMTP_FROM_NAME` | `Frontline Island Bar & Grill` |
   | `SMTP_FROM_EMAIL` | `frontlineislandbarandgrill1@gmail.com` |
   | `FRONTEND_URL` | `https://frontlinecateringcrm.netlify.app` |

5. Click **Create Web Service**
6. Wait for deployment. Note your Render URL:  
   `https://frontline-crm-api.onrender.com` (example)
7. Test it: visit `https://your-render-url.onrender.com/health` — should return `{"status":"ok"}`

---

### STEP 3 — Set Up Gmail App Password

Gmail requires an **App Password** for SMTP (not your regular Gmail password):

1. Go to [myaccount.google.com](https://myaccount.google.com)
2. **Security** → **2-Step Verification** → Make sure it's enabled
3. **Security** → **App passwords** (search for it)
4. Select app: **Mail**, Select device: **Other** → name it `Frontline CRM`
5. Copy the 16-character password (e.g. `abcd efgh ijkl mnop`)
6. Use this as `SMTP_PASS` in Render (no spaces needed)

---

### STEP 4 — Connect Frontend on Netlify

**Option A: Same-origin (Render serves frontend)**  
If you want Render to serve both frontend and backend from one URL:
- No changes needed to `index.html`
- Access your CRM at: `https://your-render-url.onrender.com`

**Option B: Netlify frontend + Render backend (your current setup)**

1. The `index.html` already has smart API detection built in
2. On Netlify, go to your site → **Site configuration** → **Environment variables**
3. Add:
   | Key | Value |
   |-----|-------|
   | `BACKEND_URL` | `https://your-render-url.onrender.com` |

4. Alternatively, add a `_headers` file to your `frontend/` folder:
   ```
   /*
     X-Backend-URL: https://your-render-url.onrender.com
   ```

5. **Simplest method** — Edit `frontend/index.html`, find this line:
   ```javascript
   const BACKEND_URL = (typeof window !== 'undefined' && window.BACKEND_URL) ? ...
   ```
   Change it to hard-code your Render URL:
   ```javascript
   const BACKEND_URL = 'https://frontline-crm-api.onrender.com';
   ```

6. Push to GitHub → Netlify auto-deploys
7. Your Netlify site should now connect to the Render backend

---

### STEP 5 — Verify Everything Works

1. Visit `https://frontlinecateringcrm.netlify.app`
2. Login with `ravi` / `Frontline2026!@#$%`
3. Dashboard should show **146 leads** loaded from MongoDB Atlas
4. Test:
   - ✅ Dashboard stats load
   - ✅ Catering leads show
   - ✅ Soccer leads show
   - ✅ Add a new lead
   - ✅ Edit a lead
   - ✅ CSV import (use `Test Import` button)
   - ✅ Email send (requires Gmail App Password)

---

## 🏠 Option B: Render Serves Everything (Simpler)

If you want ONE URL for the entire app (no Netlify needed):

1. In Render, set **Root Directory** to the project root (not `/backend`)
2. Set **Start Command** to `cd backend && node server.js`
3. The backend already serves `frontend/index.html` as a static SPA
4. Access everything at `https://your-render-url.onrender.com`
5. No API URL change needed — frontend uses relative `/api` paths

---

## 💻 Local Development

```bash
# 1. Clone the repo
git clone https://github.com/YOUR_REPO/frontline-crm.git
cd frontline-crm

# 2. Install backend dependencies
cd backend && npm install

# 3. Create .env file
cp .env.example backend/.env
# Edit backend/.env with your MongoDB Atlas URI and SMTP credentials

# 4. Start the server
npm start
# or for hot-reload:
npm run dev

# 5. Open browser
open http://localhost:3001
```

---

## 🌱 Database Seeding

The 146 Kimi AI research leads are **automatically seeded** on first startup when the database is empty.

To **re-seed** (wipe and start fresh):
```bash
# Connect to MongoDB Atlas, drop the 'leads' and 'users' collections,
# then restart the server. The seed will run automatically.
```

To **import additional leads via CSV**:
1. Log in to the CRM
2. Go to any module → **Import CSV**
3. Upload a CSV with columns: Organization, Contact, Email, Phone, Type, Event Date, Notes

---

## 🔌 API Reference

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| POST | `/api/auth/login` | ❌ | Login → returns JWT |
| GET | `/api/auth/me` | ✅ | Verify token + get user |
| POST | `/api/auth/logout` | ✅ | Logout |
| GET | `/api/leads` | ✅ | Get all leads (filterable) |
| POST | `/api/leads` | ✅ | Create lead |
| PUT | `/api/leads/:id` | ✅ | Update lead |
| DELETE | `/api/leads/:id` | ✅ | Delete lead |
| POST | `/api/leads/import/bulk` | ✅ | Bulk import leads |
| POST | `/api/email/send` | ✅ | Send email via SMTP |
| GET | `/api/email/logs` | ✅ | Email send history |
| GET | `/api/data/stats` | ✅ | Dashboard statistics |
| GET | `/api/data/modules` | ✅ | Custom modules |
| GET | `/api/data/games` | ✅ | Calendar events |
| GET | `/health` | ❌ | Health check |

---

## ⚠️ Render Free Tier Notes

- Free tier servers **spin down after 15 min of inactivity**
- First request after spin-down takes ~30 seconds (cold start)
- The app shows a loading spinner during this time — that's normal
- Consider upgrading to Render Starter ($7/mo) for always-on service
- MongoDB Atlas M0 (free) has 512MB storage — enough for thousands of leads

---

## 🔒 Security Notes

- JWT tokens expire in 7 days (configurable via `JWT_EXPIRES_IN`)
- All API routes are rate-limited (300 req/min global)
- Login endpoint has additional rate limit (10 attempts per 15 min)
- Passwords are hashed with bcrypt (12 rounds)
- CORS is restricted to your configured `FRONTEND_URL`
- Helmet.js security headers enabled
