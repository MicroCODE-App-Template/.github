# MicroCODE App Template

A SaaS template built from the Gravity App Boilerplate.

This template was created from Kyle's GRAVITY-APP product in 2024, when MicroCODE `purchased` it and began modifications.
This template combines the best features from `gravity-current` and `ladders` to serve as a common base for building new MicroCODE SaaS applications.

This template will include all major features:

- **Multi-Lingual Support (i18n)**
- **Shadcn/UI Integration**
- **Express 5 Compatibility**
- **Enhanced Server Architecture** (SIG handling, graceful shutdown, port management)
- **JSDoc Annotations** for automatic API documentation
- **Swagger/OpenAPI Support** for API documentation and testing
- **MicroCODE Packages** (mcode-\* packages)
- **MicroCODE Bootstrap System** for environment initialization
- **MongoDB \_id as Primary Key** (no secondary UUID)

NOTE: For testing multiple Clients against one Server, remove this from `VITE Config`: `strictPort: true`

## Description

MicroCODE purchased a complete license to the Gravity SaaS Boilerplate in
January 2024. We are free to use it to build as many Web Apps as we like, we
cannot resell the boilerplate nor any derivative of it.

This `app-template` serves as the foundation for building new MicroCODE SaaS applications, including:

- **LADDERS®** - PLC Programming Platform
- **Regatta RC™** - Remote Control Application
- Future MicroCODE SaaS products

## Getting Started

### Dependencies

- Node.js 22+ - execution environment
- MongoDB - database
- Stripe - payments
- Mailgun - email service
- mcode-\* packages - MicroCODE logging, caching, data, and list processing
- ...

### Documentation

We selected Gravity as our SaaS Boilerplate because their code looked super-clean, consistent, and
they had beautiful online documentation, and training...

[Gravity Documentation](https://docs.usegravity.app/)

### Training

- MicroCODE purchased Gravity Training from Kyle, to access course: [Gravity Training Course](https://click.mlflow.com/link/c/YT0yMzg5NDM0OTg0Mzk5MTE1OTM3JmM9Zjd0NSZlPTQ2NzQ0NTIxJmI9OTczMTkwMTA1JmQ9cTFuNHoyeQ==.E8Etpdc01Kz2FeMURvW6oqmj9O6FqnopOVIiujHd1Z8)

### Initial Setup

1. Install MongoDB Community Edition

   - [MongoDB Download Center](https://www.mongodb.com/try/download/community)

2. Install Node Version Manager (NVM)

   - [NVM for Windows](https://github.com/coreybutler/nvm-windows)

3. Install Node.js v22+

   - Using NVM: `nvm install 22.24.0` (or later)
   - Using NVM: `nvm use 22.24.0` (or later)

4. Clone the MicroCODE App Template repository

   - `git clone https://github.com/MicroCODE-App-Template/app-template.git`

5. Create the required `.env` configuration files

   - See [DEVELOPER-ENV.md](./DEVELOPER-ENV.md) for details

6. Fill out the required `config/default.json` configuration files

   - See [DEVELOPER-CONFIG.md](./DEVELOPER-CONFIG.md) for details

7. Ensure MongoDB is running with a 'appDatabase' and 'appAdmin' user and SEED data

   - See [DEVELOPER-MONGO.md](./DEVELOPER-MONGO.md) for details
   - Move to the CONSOLE directory (`cd console`)
   - Create the MicroCODE Console Admin Account (`npm run create:admin`)

8. Install NPM dependencies and Setup proper DB Models in Source Code

   - Move to the SERVER directory (`cd server`)
   - Run the setup scripts `npm run setup.all` to create DB Models and install dependencies
   - NOTE: `setup.all` runs the 'setup' scripts for both the SERVER and CONSOLE
   - To place the entire `app-template` in a GitHub repo state, run `npm run teardown.all` from the SERVER directory

   - You should see these messages like these...
   <p align="left"><img src=".\images\mongodb-run-seeds.png" width="720" title="Init new MongoDB" style="border: 0.5px solid lightgray;"></p>

   - Then check the "appDatabase" in mongosh for the default tables, similar to these...
   <p align="left"><img src=".\images\mongodb-drop-database.png" width="720" title="Delete MongoDB" style="border: 0.5px solid lightgray;"></p>

---

## Running the Frontends (MicroCODE Console and User App)

MicroCODE Console is your admin dashboard for managing users, viewing metrics, and monitoring the application.
Instead of using the legacy setup wizard, follow this manual configuration process above, and then start the servers.

### Start the App Server

```bash
cd server
npm run dev
```

The App will be available at `http://localhost:3000`

### First User Login

1. Navigate to `http://localhost:3000/signup`
2. Create a login with email validation
3. Select Plan and Invite Users as needed

### Start MicroCODE Console

```bash
cd console
npm run dev
```

MicroCODE Console will be available at `http://localhost:5002`

### First Admin Login

1. Navigate to `http://localhost:5002/signin`
2. Log in with `admin@mcode.com` / `admin123`
3. **Immediately change your password** in the account settings

---

You should see `Server console` output like this...

<p align="left"><img src=".\images\npm-run-dev-1.png" width="720" title="Server startup" style="border: 0.5px solid lightgray;"></p>

The `Client` launches here, but is not 100% available until the Server completes...

<p align="left"><img src=".\images\npm-run-dev-2.png" width="720" title="Client startup" style="border: 0.5px solid lightgray;"></p>

And the `App` should automatically open to the Login screen, similar to this...

<p align="left"><img src=".\images\npm-run-dev-3.png" width="720" title="App Login" style="border: 0.5px solid lightgray;"></p>

---

# App Branding and Colors

The primary color for the App can be changed in these two files:

- `client/tailwind.config.js`

```
'primary': '#6363ac',  // Purple
```

- `app/components/global/global.js`

```
const Color = {
  primary: "#546DB7",      // Blue-purple (line 12)
  purple: "#576CB5",       // Purple (line 22)
  darkpurple: "#556CB6",   // Dark purple (line 25)
  // ... other colors
};
```

- `console/client/tailwind.config.js`

```
'primary': '#6363ac',  // Purple (same as web app)
```

The current primary color is a purple: `#6363ac`

---

# Production Security

## What's Currently Set Up:

- Authorization enabled
- User authentication required
- Role-based access control (appAdmin has proper roles)
- Connection authentication working
- What You Should Consider Adding:

## Network Security

MongoDB is bound to 127.0.0.1 (localhost only) ✅ Good for development

- For production, keep it this way or use VPN/firewall rules

## Stronger Password

- Currently using `"appPassword"` - change to a strong, random password for production

## Principle of Least Privilege

- **Admin user (`admin`)**: Full access to all databases - use only for maintenance and user management
- **Application user (`appAdmin`)**: Limited to `appDatabase` only - used by the application
  - This isolation ensures leaked application credentials don't compromise other databases
- Consider creating additional users:
  - Read-only user for reporting
  - Backup user with minimal permissions

## Additional Security Measures (for production):

- Force HTTPS connections
- Force 2-Factor Authentication (2FA) for Admin / Master users
- Enable TLS/SSL encryption
- Enable audit logging
- Set up IP whitelisting
- Regular backups with encryption
- Rotate credentials periodically
- For your current development setup, you have everything you need. Just remember to:

Use a strong password before going to production
Never commit .env file with real credentials to git
Consider environment-specific users (dev vs production)

## Help

MicroCODE has a support contract with Gravity Ltd, Kyle Gawley.

## Terminology

| Word or Acronym | Description/Definition                                                                                                                              |
| --------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| **SaaS**        | Software as a Service (SaaS) - subscription based access to a software product or service.                                                          |
| **Gravity**     | A SaaS Boilerplate, from which this App Template is constructed.                                                                                    |
| **API**         | An Application Programming Interface, or API, is the set of functions/objects that a developer will provide in order to make use of their services. |
| **NPM**         | Node Package Manager, actually "Node PM", "Node pkgmakeinst" a system to deploy, install, and maintain NodeJS Apps. (PM was a BASH utility).        |
| **NVM**         | Node Version Manager, a tool that supports changing NodeJS versions.                                                                                |
| **MERN**        | MongoDB, Express, React, Node JS.                                                                                                                   |
| **MongoDB**     | A 'NoSQL' database designed for Cloud applications, also referred to as a 'Document Store'.                                                         |
| **ExpressJS**   | Express is _not_ a database but rather an 'extensible routing language' for communication between a Client and a Server.                            |
| **React**       | A Web UI development system, a JavaScript library developed by Facebook and made public—and Open Source—since 2013.                                 |
| **NodeJS**      | A development stack that executes from a local file store—on a local Server—instead of from a network of servers.                                   |
| **JSDocs**      | A toolset to automatically generate API-style documentation from source code tagging.                                                               |

## Authors

Contributors names and contact info

- Timothy J McGuire [@TimothyMcGuire](https://twitter.com/TimothyMcGuire)

## Version History

- See [CHANGELOG.md](./CHANGELOG.md) for details

## License

This project is licensed under a PRIVATE License - see the LICENSE.md file for details:
[MicroCODE SaaS License](./LICENSE.md)

The App Template it is based upon is licensed under their PRIVATE license,
see [GRAVITY Software License Agreement](./server/LICENSE.md)

## Acknowledgments

- [App Boilerplate](https://usegravity.app/)
