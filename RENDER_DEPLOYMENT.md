# Render Deployment Guide - Multi-Service Todo App

## Overview
This guide explains how to deploy your multi-service Todo application to Render with automated builds on every Git commit.

## Prerequisites
- GitHub account with your repo pushed
- Render account (free tier available)
- Docker and Dockerfile configured for both services ✓

## Deployment Steps

### Step 1: Push Your Repository to GitHub
Ensure your code is pushed to GitHub with the `render.yaml` file at the root.

```bash
git add .
git commit -m "Add render.yaml for automated deployment"
git push origin main
```

### Step 2: Connect GitHub to Render
1. Go to [Render Dashboard](https://dashboard.render.com)
2. Click **"New +"** → **"Blueprint"**
3. Select **"Public Git repository"**
4. Paste your GitHub repository URL
5. Click **"Connect"**

### Step 3: Review Blueprint Configuration
Render will automatically read your `render.yaml` file and display:
- **Backend Service** (be-todo): Node.js API on port 5000
- **Frontend Service** (fe-todo): React app on port 80

### Step 4: Configure Environment Variables

#### For Backend Service (be-todo):
The following variables **must be set manually** in the Render dashboard:
- `DB_HOST` - Your PostgreSQL host (e.g., from Render Postgres or external DB)
- `DB_USER` - Database user
- `DB_PASSWORD` - Database password
- `DB_NAME` - Database name

**Optional:**
- `DB_PORT` - Default: 5432 (already set)
- `DB_SSL` - Default: true (already set)
- `NODE_ENV` - Default: production (already set)

#### For Frontend Service (fe-todo):
- `REACT_APP_API_URL` - Already set to: `https://be-todo.onrender.com`
  - If your backend has a different domain, update this value

### Step 5: Create PostgreSQL Database (if needed)
If you don't have an external database:
1. In Render Dashboard, go to **"New +"** → **"PostgreSQL"**
2. Create a new PostgreSQL database
3. Copy the connection details and use them for `DB_HOST`, `DB_USER`, `DB_PASSWORD`, `DB_NAME`

### Step 6: Add Environment Variables in Render Dashboard
1. After selecting "Blueprint", you'll see both services
2. For **be-todo**: 
   - Click on the service
   - Go to **"Environment"** tab
   - Add all required database variables (DB_HOST, DB_USER, DB_PASSWORD, DB_NAME)
3. For **fe-todo**:
   - Verify `REACT_APP_API_URL` matches your backend URL (will be like: `https://be-todo.onrender.com`)

### Step 7: Deploy
1. Click **"Deploy"** on the Blueprint page
2. Render will build and deploy both services automatically
3. You'll see real-time logs as it builds

## After Deployment

### Accessing Your App
- **Frontend**: `https://fe-todo.onrender.com`
- **Backend API**: `https://be-todo.onrender.com`

### Verify Services
- Open your frontend URL - React app should load
- Check backend health: `curl https://be-todo.onrender.com/api/health`
- Test API endpoints from your frontend

## Automatic Rebuilds

Your `render.yaml` includes `autoDeploy: true`, which means:
- ✅ Every time you push to GitHub, Render automatically triggers a new build
- ✅ No manual deployment needed after the initial setup
- ✅ Both services rebuild together

### Triggering a New Deployment
Simply push a commit to your repo:
```bash
git add .
git commit -m "Your changes"
git push origin main
```

## Troubleshooting

### Backend won't connect to database
- Verify all `DB_*` environment variables are set correctly
- Check database host is accessible from Render
- Review backend logs in Render dashboard

### Frontend shows blank page
- Check `REACT_APP_API_URL` in frontend environment variables
- Verify backend service is running
- Check browser console for CORS errors

### Port conflicts
- Backend: Uses port 5000 (set via `PORT` env var)
- Frontend: Uses port 80 (configured in nginx.conf)
- These don't conflict - each service has its own container

### Build fails
- Check logs in Render dashboard
- Ensure all `package.json` dependencies are installable
- Verify Dockerfile commands are correct

## Important Notes

1. **Database Credentials**: Keep DB_PASSWORD secure - use Render's secret variable feature
2. **API URL Updates**: If backend domain changes, update frontend's `REACT_APP_API_URL`
3. **Node Version**: Both services use `node:18-alpine` - change in Dockerfile if needed
4. **Free Tier**: Render free tier may have limitations - check their pricing

## Blueprint Spec Reference

The `render.yaml` file uses Render's Blueprint specification:
- Services are defined with type, environment, and Dockerfile
- Environment variables can be static or pulled from external services
- `autoDeploy: true` enables automatic rebuilds on git push
- `healthCheckPath` helps Render monitor service health

More info: https://render.com/docs/blueprint-spec
