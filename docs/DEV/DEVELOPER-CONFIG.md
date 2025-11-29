# Configuration Files (⚠️ CRITICAL - DO NOT SKIP)

The application requires proper configuration files in `server/config/` that are **NOT tracked in git** for security. Missing or incomplete configuration will cause runtime errors.

## Required Configuration Files

Create these files in the `server/config/` directory:

### 1. `default.json` (Development Configuration)

This file is used for local development. Create it with the following structure:

```json
{
  "app": {
    "name": "Your App Name",
    "url": "http://localhost:3000"
  },
  "email": {
    "domain": "sandbox-xxxxxxxxxxxxx.mailgun.org",
    "sender": "noreply@yourdomain.com"
  },
  "database": {
    "connection": "mongodb://appAdmin:appPassword@localhost:27017/appDatabase"
  }
}
```

### 2. `production.json` (Production Configuration)

This file is used when `NODE_ENV=production`. It should contain production-specific values:

```json
{
  "app": {
    "url": "https://yourdomain.com"
  },
  "email": {
    "domain": "mg.yourdomain.com",
    "sender": "noreply@yourdomain.com"
  }
}
```

## Critical: Mailgun Email Configuration

The `email.domain` field is **required** for email functionality. If missing or empty, you'll see:

- ❌ Red banner in the UI warning about configuration errors
- ❌ Server startup warnings in the console
- ❌ 404 errors when attempting to send emails

**Where to find your Mailgun domain:**

1. Log into your [Mailgun dashboard](https://app.mailgun.com/)
2. Navigate to **Sending** → **Domains**
3. Copy your sandbox domain (format: `sandbox-xxxxxxxxxxxxx.mailgun.org`) for development
4. For production, use your verified custom domain

**Important:** Do not leave `email.domain` as an empty string `""` - this will cause Mailgun to return 404 errors.
