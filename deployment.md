# Deployment Guide

This guide covers deploying the application to production using Netlify (frontend) and Railway (backend).

## Frontend Deployment (Netlify)

### Configuration Files

Two key files handle routing and API proxying in production:

1. `frontend/netlify.toml`:
```toml
[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200

[[redirects]]
  from = "/api/*"
  to = "https://your-backend.railway.app/api/:splat"
  status = 200
  force = true
```

2. `frontend/public/_redirects`:
```
/api/*  https://your-backend.railway.app/api/:splat  200
/*      /index.html     200
```

### Environment Configuration

The frontend uses relative API paths in production (`frontend/src/environments/environment.ts`):
```typescript
export const environment = {
  production: true,
  apiUrl: '/api'  // Use relative path for Netlify proxy
};
```

### Critical Settings

1. **API Proxying**: All `/api/*` requests are proxied to your Railway backend through Netlify.
2. **SPA Routing**: Non-API routes fall back to `index.html` for Angular routing.
3. **Build Output**: `_redirects` and `netlify.toml` must be included in the build output.

## Backend Deployment (Railway)

### Required Environment Variables

- `NODE_ENV=production` - Enables secure cookie settings
- `CORS_ORIGIN=https://your-frontend-domain.com` - Your Netlify domain
- JWT secrets (see Environment Variables section in main README)

### Cookie Security

In production:
- `Secure: true` - Requires HTTPS
- `SameSite: 'none'` - Allows cross-origin requests
- `HttpOnly: true` - Prevents JavaScript access

### CORS Configuration

The backend allows credentials and specific origins:
```typescript
app.use(cors({
  origin: process.env.CORS_ORIGIN,
  credentials: true
}));
```

## Common Issues

### Missing Cookies

If authentication cookies aren't sent on API requests:
1. Verify `NODE_ENV=production` on Railway (enables proper cookie settings)
2. Confirm frontend uses relative `/api` paths in production
3. Check Netlify includes redirect rules in build
4. Verify CORS_ORIGIN matches your Netlify domain exactly

### CORS Errors

If you see CORS errors:
1. Check `CORS_ORIGIN` on Railway matches your Netlify domain
2. Ensure credentials are enabled in both frontend and backend
3. Verify cookies use `SameSite=None; Secure` in production

### Cookie Domains

- Cookies are automatically scoped to the Netlify domain when using the proxy
- No explicit `Domain` attribute needed in cookie settings
- All requests appear same-origin to the browser

## Deployment Checklist

1. Frontend (Netlify):
   - [ ] Push code with `netlify.toml` and `_redirects`
   - [ ] Verify build includes redirect rules
   - [ ] Update API URL in environment.ts
   - [ ] Deploy and verify routing works

2. Backend (Railway):
   - [ ] Set `NODE_ENV=production`
   - [ ] Configure `CORS_ORIGIN` with Netlify domain
   - [ ] Set all required JWT secrets
   - [ ] Deploy and verify API responds

3. Verify Integration:
   - [ ] Test login flow end-to-end
   - [ ] Confirm cookies are set and sent
   - [ ] Check protected routes work
   - [ ] Verify email verification flow