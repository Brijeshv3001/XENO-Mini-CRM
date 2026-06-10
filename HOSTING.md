# XenoCRM Live Hosting Manual

This manual details the step-by-step instructions to host both the Next.js CRM frontend (Vercel) and the Channel campaign microservice (Railway/Render) live.

---

## 1. Database Migration: SQLite to PostgreSQL (Neon/Supabase)

Next.js is hosted on Vercel's serverless environment, which has a **stateless, read-only filesystem**. Storing data in a local SQLite file (`dev.db`) will cause data loss whenever the serverless container spins down or restarts. To deploy live, you must migrate to a persistent cloud PostgreSQL database (e.g. Neon or Supabase free tiers).

### Steps:
1. Create a free PostgreSQL database on [Neon.tech](https://neon.tech) or [Supabase.com](https://supabase.com).
2. Copy the database connection URL string (it will start with `postgresql://...`).
3. Open `prisma/schema.prisma` in your code and change the provider from `sqlite` to `postgresql`:
   ```prisma
   datasource db {
     provider = "postgresql"
     url      = env("DATABASE_URL")
   }
   ```
4. Update the `.env` file with the database URL:
   ```bash
   DATABASE_URL="postgresql://user:password@host/dbname?sslmode=require"
   ```
5. Push the schemas and run the seeder script on your new live database:
   ```bash
   npx prisma db push
   node prisma/seed.js
   ```

---

## 2. Deploy the Frontend (Vercel)

Vercel natively integrates with your GitHub repository to enable automatic deployment:

1. Go to [Vercel](https://vercel.com) and sign in with your GitHub account.
2. Click **Add New** -> **Project**.
3. Import the repository: `XENO-Mini-CRM`.
4. Under **Environment Variables**, configure the following variables:
   - `DATABASE_URL` : *Your Neon/Supabase PostgreSQL connection string.*
   - `ANTHROPIC_API_KEY` : *Your Anthropic Claude API key.*
   - `CHANNEL_SERVICE_URL` : *The live URL of your Channel Service (created in Step 3, e.g. `https://xeno-channel-service.up.railway.app/send`).*
   - `CRM_RECEIPT_URL` : *The callback receipt URL pointing back to Vercel (e.g. `https://xeno-mini-crm.vercel.app/api/receipts`).*
5. Click **Deploy**. Vercel will build and host the CRM dashboard.

---

## 3. Deploy the Channel Service (Railway or Render)

The Express campaign simulator needs a persistent Node.js environment to handle staggered setTimeout timers:

### Deploying on Railway:
1. Go to [Railway.app](https://railway.app) and sign in.
2. Click **New Project** -> **Deploy from GitHub repo** -> Select `XENO-Mini-CRM`.
3. Go to the service's **Settings** and set the **Root Directory** to: `channel-service`.
4. Under **Variables**, configure:
   - `CRM_RECEIPT_URL` : *The live URL of your Vercel deployment (e.g. `https://xeno-mini-crm.vercel.app/api/receipts`).*
5. Click deploy. Railway will launch the simulation container.

### Deploying on Render:
1. Go to [Render.com](https://render.com) and sign in.
2. Click **New** -> **Web Service** -> Link your GitHub repo.
3. Set the **Root Directory** to: `channel-service`.
4. Set the **Build Command** to: `npm install`.
5. Set the **Start Command** to: `node server.js`.
6. Add the environment variable `CRM_RECEIPT_URL` pointing to your Vercel app URL.
