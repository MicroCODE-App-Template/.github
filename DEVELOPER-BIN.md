# BIN/ Utilities

The `bin/` directories contain utility scripts for scaffolding, setup, and maintenance. All scripts follow the new file naming conventions:

- `{entity}.model.js` for models
- `{entity}.controller.js` for controllers
- `{entity}.route.js` for API routes
- `{entity}.{locale}.json` for locale files

## server/bin/

### Scaffolding & Code Generation

- **`view.js`** — Scaffolds new MVC entities (model, view, controller, API routes, locales).

  - Usage: `node bin/view.js <entity-name> [-ui] [-db]`
  - Creates: `{entity}.model.js`, `{entity}.controller.js`, `{entity}.route.js`, `{entity}.{locale}.json`
  - `-ui` flag: Also creates React UI views and routes
  - `-db` flag: Creates SQL migration file (SQL databases only, not MongoDB)

- **`component.js`** — Creates new React components. Usage: `node bin/component.js <component-name>`

  - Creates component file and adds export to `lib.jsx`

- **`master.js`** — Creates master admin accounts. Usage: `node bin/master.js <email>:<password>`
  - Creates account with `master` plan and admin user with `master` permissions

### Setup & Configuration

- **`setup.js`** — Initial database setup and model file copying

  - Reads `DB_CLIENT` from `.env` to determine database type
  - Copies model files from `model/sql/` or `model/mongo/` to `model/` root
  - Renames files to `.model.js` format during copy
  - Runs database migrations (SQL) or prepares MongoDB models

- **`cleanup.js`** — Removes setup/demo files after initial setup
  - Deletes setup controller, model, route, and locale files
  - Removes setup views from client

### CLI & Tooling

- **`gravity.js`** — Interactive CLI toolbelt for common tasks

  - `gravity create view` — Scaffold new MVC (interactive)
  - `gravity create component` — Create React component (interactive)
  - `gravity create master` — Create master account (interactive)
  - `gravity help` — Show interactive help menu
  - `gravity test` — Run test suite (server only)

- **`start.js`** — Starts development servers concurrently
  - Runs `npm run server`, `npm run app` (if exists), `npm run client` (if exists)

### Installation Checks

- **`installcheck.js`** — Validates Node.js version (requires 15+)
- **`appcheck.js`** — Checks for mobile app directory and renames if needed
  - Looks for: `app`, `client-react-native`, `client-react-native-main`
- **`clientcheck.js`** — Checks for web client directory and renames if needed
  - Looks for: `client`, `client-react-web`, `client-react-web-main`

## console/bin/

### Scaffolding & Code Generation

- **`view.js`** — Scaffolds new MVC entities (same as server version), creates:

  - `{entity}.model.js`
  - `{entity}.controller.js`
  - `{entity}.route.js`
  - `{entity}.{locale}.json`

- **`component.js`** — Creates new React components (same as server version)

- **`master.js`** — Creates master admin accounts (same as server version)

- **`create-admin.js`** — Creates MicroCODE Console admin account directly in MongoDB
  - Usage: `npm run create:admin` or `node bin/create-admin.js`
  - Creates default admin: `admin@mcode.com` / `admin123`
  - Bypasses setup wizard for manual configuration
  - Uses mongoose schemas directly (no model file dependencies)

### Setup & Configuration

- **`setup.js`** — Initial database setup and model file copying (same as server version)

  - Copies model files and renames to `.model.js` format

- **`cleanup.js`** — Removes setup/demo files (same as server version)

### CLI & Tooling

- **`gravity.js`** — Interactive CLI toolbelt (same as server, without test command)

### Installation Checks

- **`installcheck.js`** — Validates Node.js version (requires 15+)

---

`{entity}.route.js`, `{entity}.{locale}.json`

- `-ui` flag: Also creates React UI views and routes
- `-db` flag: Creates SQL migration file (SQL databases only, not MongoDB)

- **`component.js`** — Creates new React components. Usage: `node bin/component.js <component-name>`

  - Creates component file and adds export to `lib.jsx`

- **`master.js`** — Creates master admin accounts. Usage: `node bin/master.js <email>:<password>`
  - Creates account with `master` plan and admin user with `master` permissions

### Setup & Configuration

- **`setup.js`** — Initial database setup and model file copying

  - Reads `DB_CLIENT` from `.env` to determine database type
  - Copies model files from `model/sql/` or `model/mongo/` to `model/` root
  - Renames files to `.model.js` format during copy
  - Runs database migrations (SQL) or prepares MongoDB models

- **`cleanup.js`** — Removes setup/demo files after initial setup
  - Deletes setup controller, model, route, and locale files
  - Removes setup views from client

### CLI & Tooling

- **`gravity.js`** — Interactive CLI toolbelt for common tasks

  - `gravity create view` — Scaffold new MVC (interactive)
  - `gravity create component` — Create React component (interactive)
  - `gravity create master` — Create master account (interactive)
  - `gravity help` — Show interactive help menu
  - `gravity test` — Run test suite (server only)

- **`start.js`** — Starts development servers concurrently
  - Runs `npm run server`, `npm run app` (if exists), `npm run client` (if exists)

### Installation Checks

- **`installcheck.js`** — Validates Node.js version (requires 15+)
- **`appcheck.js`** — Checks for mobile app directory and renames if needed
  - Looks for: `app`, `client-react-native`, `client-react-native-main`
- **`clientcheck.js`** — Checks for web client directory and renames if needed
  - Looks for: `client`, `client-react-web`, `client-react-web-main`

## console/bin/

### Scaffolding & Code Generation

- **`view.js`** — Scaffolds new MVC entities (same as server version)

  - Creates: `{entity}.model.js`, `{entity}.controller.js`,
  - `{entity}.route.js`, `{entity}.{locale}.json`

- **`component.js`** — Creates new React components (same as server version)

- **`master.js`** — Creates master admin accounts (same as server version)

- **`create-admin.js`** — Creates MicroCODE Console admin account directly in MongoDB
  - Usage: `npm run create:admin` or `node bin/create-admin.js`
  - Creates default admin: `admin@mcode.com` / `admin123`
  - Bypasses setup wizard for manual configuration
  - Uses mongoose schemas directly (no model file dependencies)

### Setup & Configuration

- **`setup.js`** — Initial database setup and model file copying (same as server version)

  - Copies model files and renames to `.model.js` format

- **`cleanup.js`** — Removes setup/demo files (same as server version)

### CLI & Tooling

- **`gravity.js`** — Interactive CLI toolbelt (same as server, without test command)

### Installation Checks

- **`installcheck.js`** — Validates Node.js version (requires 15+)

---
