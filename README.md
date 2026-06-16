# Joblet Server

Backend API server for the Joblet job portal application. Deployed on Render.

## Setup

### Local Development

```bash
# Install dependencies
npm install

# Copy and configure environment variables
cp .env.example .env

# Start development server
npm run dev
```

Server runs on `http://localhost:3000`

### Production Deployment

Deployed on Render at `https://jobletserver.onrender.com`

#### Environment Variables Required on Render:
- `NODE_ENV=production`
- `FIREBASE_PROJECT_ID`
- `FIREBASE_SERVICE_ACCOUNT_JSON`
- `GEMINI_API_KEY`
- `CLOUDINARY_NAME`, `CLOUDINARY_API_KEY`, `CLOUDINARY_API_SECRET`
- `SUPABASE_URL`, `SUPABASE_KEY`
- `FRONTEND_URL` (for CORS)

#### Render Deployment:
1. Connect this GitHub repo to Render
2. Set Build Command: `npm install`
3. Set Start Command: `node server.js`
4. Add all environment variables in Render dashboard
5. Render auto-deploys on push to main branch

## API Endpoints

All API endpoints are prefixed with `/api/`

### Jobs
- `GET /api/jobs` - Get all jobs
- `POST /api/company/post-job` - Create new job (requires auth)
- `GET /api/jobs/:id` - Get job details
- `PUT /api/jobs/:id` - Update job (requires auth)
- `DELETE /api/jobs/:id` - Delete job (requires auth)

### Users
- `POST /api/auth/register` - Register user
- `POST /api/auth/login` - Login user
- `GET /api/users/profile` - Get user profile (requires auth)
- `PUT /api/users/profile` - Update profile (requires auth)

### Chatbot
- `POST /api/chat` - Send chat message
- `POST /api/chat/company` - Company chatbot

### More endpoints available - see routes/ directory

## CORS

CORS is configured to allow:
- `http://localhost:5173` (local development)
- `https://joblet-eight.vercel.app` (production)
- Any other origins in `FRONTEND_URL` env variable

## Testing

```bash
# Run tests
npm test

# Test chatbot
npm run test:chat
```

## Project Structure

```
├── config/              # Configuration files (Firebase, Supabase, etc.)
├── controller/          # Request handlers
├── middleware/          # Express middleware (auth, rate limiting)
├── routes/              # API route definitions
├── services/            # Business logic
├── utils/               # Utility functions
├── server.js            # Main server file
└── package.json         # Dependencies
```

## Troubleshooting

### Cold Start Issues
Render free tier has slower cold starts. Monitor at render.com dashboard.

### CORS Errors
Check that your client origin is in `FRONTEND_URL` env variable.

### Firebase Connection
Check `FIREBASE_SERVICE_ACCOUNT_JSON` is properly formatted JSON string.

### Gemini API
Verify `GEMINI_API_KEY` is valid and has appropriate permissions.

## References

- [Render Docs](https://render.com/docs)
- [Express.js](https://expressjs.com/)
- [Firebase Admin SDK](https://firebase.google.com/docs/admin/setup)
- [Supabase](https://supabase.com/docs)
