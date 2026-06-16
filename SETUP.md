# Joblet Backend Setup Guide - Render Deployment

This document guides you through setting up the `jobletserver` repository as a separate backend deployment on Render.

## Overview

**Architecture**:
- **Client**: `job-main` repository → Vercel deployment (https://joblet-eight.vercel.app)
- **Server**: `jobletserver` repository → Render deployment (https://jobletserver.onrender.com)
- **Database**: Firebase + Supabase (shared)

**Local Development**:
- Client: `http://localhost:5173` (Vite)
- Server: `http://localhost:3000` (Express)

---

## Step 1: Initialize jobletserver Git Repository

```bash
cd c:\Users\lenovo\Downloads\jobletserver

# Initialize git
git init
git add .
git commit -m "Initial commit: Backend server setup"
git branch -M main

# Add remote (replace with your GitHub URL)
git remote add origin https://github.com/CcOORPSE/jobletserver.git

# Push to GitHub
git push -u origin main
```

---

## Step 2: Set Up Environment Variables

### Local Development (.env file)

```bash
cd jobletserver

# Copy template
cp .env.example .env

# Edit .env and fill in:
# - FIREBASE_SERVICE_ACCOUNT_JSON (get from Firebase Console)
# - GEMINI_API_KEY (from Google Cloud Console)
# - CLOUDINARY_* (from Cloudinary dashboard)
# - SUPABASE_* (from Supabase dashboard)
```

**Local .env Example**:
```
NODE_ENV=development
PORT=3000
FIREBASE_PROJECT_ID=jobfinder-b817d
FIREBASE_SERVICE_ACCOUNT_JSON={'type':'service_account',...}
GEMINI_API_KEY=AIza...
CLOUDINARY_NAME=...
CLOUDINARY_API_KEY=...
CLOUDINARY_API_SECRET=...
SUPABASE_URL=https://...
SUPABASE_KEY=...
FRONTEND_URL=http://localhost:5173
```

---

## Step 3: Test Locally

```bash
# Install dependencies
npm install

# Start development server
npm run dev
# Server runs on http://localhost:3000

# Verify API is working
curl http://localhost:3000/api/debug-firebase-init
# Should return JSON with status info
```

---

## Step 4: Connect to Render

### 4.1 Create Render Account
- Go to [render.com](https://render.com)
- Sign up with GitHub account

### 4.2 Deploy Backend

1. **Create Web Service**:
   - Render Dashboard → New → Web Service
   - Select GitHub repo: `jobletserver`
   - Name: `jobletserver`
   - Environment: `Node`
   - Build Command: `npm install`
   - Start Command: `node server.js`
   - Plan: Free (or paid for production)

2. **Add Environment Variables** (in Render Dashboard):
   ```
   NODE_ENV=production
   FIREBASE_PROJECT_ID=jobfinder-b817d
   FIREBASE_SERVICE_ACCOUNT_JSON=(paste from Firebase)
   GEMINI_API_KEY=(from Google Cloud)
   CLOUDINARY_NAME=(your name)
   CLOUDINARY_API_KEY=(your key)
   CLOUDINARY_API_SECRET=(your secret)
   SUPABASE_URL=(your URL)
   SUPABASE_KEY=(your key)
   FRONTEND_URL=http://localhost:5173,https://joblet-eight.vercel.app
   ```

3. **Deploy**:
   - Click "Deploy"
   - Wait for build to complete (~3-5 minutes)
   - Check logs for errors

### 4.3 Verify Render Deployment

```bash
# Test API endpoint (replace with your Render URL)
curl https://jobletserver.onrender.com/api/debug-firebase-init

# Should return JSON confirming API is running
```

---

## Step 5: Update Client (job-main)

**Already Done**: Environment files created in `job-main/client/`:
- `.env.local` → `VITE_BACKEND_URL=http://localhost:3000` (development)
- `.env.production` → `VITE_BACKEND_URL=https://jobletserver.onrender.com` (production)

**Verify**:
```bash
cd job-main/client

# Development build uses .env.local
npm run dev
# Open http://localhost:5173

# Production build uses .env.production
npm run build
# Check that VITE_BACKEND_URL is set to Render URL
```

---

## Step 6: Verify Full Stack Integration

### Local Development
1. **Terminal 1 - Backend**:
   ```bash
   cd jobletserver
   npm run dev
   # Runs on http://localhost:3000
   ```

2. **Terminal 2 - Frontend**:
   ```bash
   cd job-main/client
   npm run dev
   # Runs on http://localhost:5173
   ```

3. **Test** (open browser):
   - Navigate to http://localhost:5173
   - Open DevTools → Network tab
   - Perform any API action
   - Verify API calls go to http://localhost:3000/api/* ✅

### Production Verification
1. Open https://joblet-eight.vercel.app
2. Open DevTools → Network tab
3. Verify API calls go to https://jobletserver.onrender.com/api/* ✅
4. Check console for CORS errors ❌ (should have none)

---

## Step 7: Continuous Deployment

**Auto-Deploy is Enabled**:
- Push changes to `jobletserver` main branch → Render auto-deploys
- Push changes to `job-main` main branch → Vercel auto-deploys

**Monitor Deployments**:
- Render: render.com → jobletserver service → Logs
- Vercel: vercel.com → job-main project → Deployments

---

## Troubleshooting

### CORS Errors
**Error**: "Access to XMLHttpRequest blocked by CORS"
- **Cause**: Client origin not in CORS whitelist
- **Fix**: Update `FRONTEND_URL` in Render environment variables

### Cold Start Delays
**Issue**: First request takes 30+ seconds
- **Cause**: Render free tier has cold starts
- **Solution**: Upgrade to paid plan or add keep-alive monitoring

### Firebase Initialization Error
**Error**: "Cannot initialize Firebase"
- **Cause**: Invalid `FIREBASE_SERVICE_ACCOUNT_JSON`
- **Fix**: 
  1. Download fresh service account JSON from Firebase Console
  2. Paste entire JSON as string in environment variable
  3. Verify no newlines in the env var (should be one-liner)

### Database Connection Issues
**Error**: "Cannot connect to Supabase"
- **Cause**: Invalid credentials or firewall blocking
- **Fix**:
  1. Verify `SUPABASE_URL` and `SUPABASE_KEY` are correct
  2. Check Supabase dashboard for connection limits
  3. Test connection locally first

### Gemini API Errors
**Error**: "Invalid API key"
- **Cause**: Expired or incorrect key
- **Fix**: 
  1. Regenerate key in Google Cloud Console
  2. Update in Render environment variables
  3. Restart service

---

## File Structure

```
jobletserver/
├── config/
│   ├── cloudinary.js
│   ├── firebaseAdmin.js
│   ├── instrument.js
│   ├── loadEnv.js
│   ├── multer.js
│   └── supabase.js
├── controller/
│   ├── applicantChatbotController.js
│   ├── chatbotController.js
│   ├── jobController.js
│   ├── recruiterChatbotController.js
│   ├── userController.js
│   └── ...
├── middleware/
│   ├── authMiddleware.js
│   └── rateLimiters.js
├── routes/
│   ├── jobRoutes.js
│   ├── userRoutes.js
│   ├── chatbotRoutes.js
│   └── ...
├── services/
│   └── [business logic]
├── utils/
│   └── [utility functions]
├── uploads/
│   └── [file uploads storage]
├── server.js (main entry point)
├── package.json
├── .env.example
├── render.yaml
├── Dockerfile
└── README.md
```

---

## Important Notes

1. **Git Ignore**: Node modules and `.env` are in `.gitignore` - don't commit them
2. **Secrets**: Store sensitive keys only in environment variables, never in code
3. **FRONTEND_URL**: Update when adding new domains (e.g., staging, preview)
4. **Database**: Firebase and Supabase are shared - changes affect both environments
5. **Rate Limiting**: API has rate limiting for chat endpoints - adjust if needed

---

## Next Steps

1. ✅ Backend repository created (jobletserver)
2. ✅ Client environment files configured (job-main)
3. ⏳ Push jobletserver to GitHub
4. ⏳ Connect jobletserver to Render
5. ⏳ Test local development with separated servers
6. ⏳ Verify production deployment

---

**Last Updated**: 2026-06-16
**Status**: Ready for GitHub push and Render deployment

