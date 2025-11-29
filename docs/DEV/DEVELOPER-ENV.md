# Environment Variables

Create a required .env files in the appropriate directories with sensitive credentials:

1. Create a shared `.env` file

   - Clone `.env.example` to `.env.shared` in the root directory and fill in the required values
   - Note: `DB_CLIENT` controls '`npm run setup`' behavior in Steps 6 and 7
   - the `.env` files should never be committed to source control with real credentials!
   - the `.env` files are ignored by `.gitignore` to prevent accidental commits, **_STORE them securely!_**
   - the `.env` files are required for both `server` and `console` directories.
   - the `.env` files in `server` and `console` must be identical.

2. Created a `.env` file in the `server` directory

   - Clone `.env.shared` to `.env` and fill in the required values

3. Created a `.env` file in the `console` directory

   - Clone `.env.shared` to `.env` and fill in the required values

```bash
# Mailgun API credentials
MAILGUN_API_KEY=your_mailgun_api_key_here

# Stripe payment processing
STRIPE_SECRET_API_KEY=sk_test_xxxxxxxxxxxxx

# Session security
SESSION_SECRET=your_random_session_secret

# MongoDB (if not using config files)
MONGO_URI=mongodb://appAdmin:appPassword@localhost:27017/appDatabase
```

**Find your Mailgun API key:**

- Dashboard → **Settings** → **API Keys**
- Copy the **Private API Key** (starts with `key-`)

## Configuration Validation

The application validates configuration on startup:

- ✅ **Green/No banner**: All required configuration is present
- ⚠️ **Orange banner**: Non-critical warnings (check console)
- 🛑 **Red banner**: Critical errors - app may not function (check `/api/health/config`)

If you see validation errors:

1. Check the browser console and server logs
2. Verify `server/config/default.json` exists and has valid JSON
3. Ensure `email.domain` is not empty
4. Verify `MAILGUN_API_KEY` is set in `.env`
5. Restart the server after fixing configuration

---
