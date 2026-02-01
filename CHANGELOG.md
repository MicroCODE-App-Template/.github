# Changelog

All notable changes to the MicroCODE App Template will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## MicroCODE App Template [0.1.0]

`January 14, 2026`

### Added

- **Combine all onboarding into single flow** (Issue #15)

  - Redesign of onboarding to include Clubs, Org payments
  - Ability to Add Boats
  - Separate Credit Card Validation

- **UI Components**:

  - Added complete INFO module for both CONSOLE and CLIENT
  - Added App version display to SUPPORT View
  - Added 'alpha' and 'beta' logo support
  - Added 'not implemented' modals for future features
  - Added dynamic filters to most Views
  - Added screen and context capture to Feedback

### Changed

- **Model Schema** (`server/model/*.model.js`):

  - Made minor updates to schema
  - Prepared a MongoDB Review (ISSUE#16) for next stage

- **UI Components**:

  - Updated many Views with new columns

- **Seeder** (`server/seed/seeder.js`):
  - Upgraded Import USERs
  - Upgraded IMport BOATs
  - Added 'merge' capability for multi-pass merge

`December 26, 2025`

### Added

- **Complete Locales Implementation with Flags and Sail CC Support** (Issue #11)
  - Comprehensive flag support for all 138 World Sailing countries
  - Reusable Flag and SailNumber components across all platforms (client, admin, app)
  - Legacy sail country code normalization (automatic mapping of pre-standard codes)
  - Flag display in Entity Cards, tables, lists, and locale pickers
  - New optional API endpoint: `GET /api/sail-cc` for dynamic mapping access
  - **Tag Lock:** See PRs #xxx (server), #xxx (client), #xxx (admin), #xxx (app)

### Changed

- **Boat Model Schema** (`server/model/boat.model.js`):

  - Added `sail_cc_raw` field (optional) to preserve original legacy codes
  - `sail_cc` and `sail_no` now required for new records
  - `boat_design_id` and `boat_builder_id` now required
  - `bdsn_key` and `bbld_key` now optional

- **UI Components**:

  - EntityCard: BOAT entities display flag instead of logo when `sail_cc` present
  - Admin Tables: Added 'Country' flag column, removed boat 'name' column
  - Boat/SV Lists: Display Flag + Sail No. (bold) on line 1, Design (gray) on line 2
  - Locale Pickers: Dynamic flag rendering using Flag component

- **Seeder** (`server/seed/seeder.js`):
  - Applies legacy sail_cc normalization during boat seeding
  - Ensures consistency between API-created and seeded records

## MicroCODE App Template [0.0.5]

`December 25, 2025`

### Changed

- **BREAKING:** Removed all SQL database support, standardized exclusively on MongoDB
  - Removed SQL model files, migrations, and Knex configuration
  - Removed MongoDB source directories (files consolidated in `/model/` root)
  - Removed SQL dependencies (`knex`, `mysql2`) from all repositories
  - Simplified setup/teardown scripts to MongoDB-only workflow
  - Updated setup UI components to remove SQL database options
  - Updated README files with MongoDB-only requirements
  - **Migration Note:** Existing SQL installations require manual data migration to MongoDB
  - **Tag Lock:** See PRs #xxx (server), #xxx (admin), #xxx (client)

### Removed

- SQL database support (MySQL, PostgreSQL, SQLite, MSSQL, Oracle)
- SQL migration files and Knex configuration
- SQL-specific setup and teardown logic
- `DB_PORT` environment variable (not used by MongoDB)
- `mongo:update` and `sql:update` npm scripts

## MicroCODE App Template [0.0.4]

`November 26, 2025`

### Added

- Added a complete `.github/` directory with:
  - GIT - Issue Templates/ for hotfix, feature, bugfix and release requests
  - PULL_REQUEST_TEMPLATE for standardized PR submissions
  - FUNDING.yml to support open source funding options
  - CODE_OF_CONDUCT.md to establish community guidelines
  - CONTRIBUTING.md to guide contributors on how to contribute effectively

## MicroCODE App Template [0.0.3]

`November 23, 2025`

### Changed

- Add `npm run lint, lint.all, fix.all` scripts to all repos for ESLint checks and fixes
- Updated Developer Setup Guide with linting instructions
- Added ESLint configuration files to all repos with standard rules
- Aligned code style across all repos using ESLint
- Fixed all linting issues in all repos (over 1,100 issues fixed)
- Fixed 2FA toggle issue in UI/UX layer of client app
- Style `Switch` component in client app to match Apple (Green/Gray) colors for On/Off states

## MicroCODE App Template [0.0.2]

`November 19, 2025`

### Changed

- Permanently renamed repo 'client-react-native' to `app`
- Permanently renamed repo 'client-react-web' to `client`
- Permanently renamed repo 'mission-control' to `console`
- Permanently renamed repo 'supernova' website to `portal`
- Maintained the original repo name 'server' as `server`
- Added READMEs for all Repos in App-Template Org
- Created 'breakdown' of all repos in Developer Setup Guide (the opposite of 'setup'),
- 'Setup' / 'Breakdown' allow devs to setup teh tempalte locally for development,
- Allows Devs to then restore the update code repos to their 'original' state for cloning new apps
- Updated all repository remote URLs to new MicroCODE-App-Template organization
- Updated Developer Setup Guide with new repo names and structure

## MicroCODE App Template [0.0.1]

`November 19, 2025`

### Changed

- `npm run dev` now starts both server and applicable clients concurrently
- Updated Developer Setup Guide with MongoDB installation and configuration steps
- Added DEVELOPER-MONGO-SETUP.md for detailed MongoDB setup instructions
- Added DEVELOPER-SETUP.md for comprehensive developer environment setup
- Added DEVELOPER-ENV.md for environment variable documentation
- Added DEVELOPER-CONFIG.md for configuration file documentation
- Added DEVELOPER-BIND.md for binary scripts documentation
- Updated README.md with new organization and setup instructions
- Aligned all color schemes in client-react-native global.js with client-react-web
- Updated all repository remote URLs to new MicroCODE-App-Template organization
- Removed 'dirty' \*.model.js files from /model directory
- Added MicroCODE Branding Images to all (4) repositories
- Updated `server: npm run setup` and `server: npm run dev` scripts to init, build, and 'run dev' all all solution components:
  - 'server'
  - 'client'
  - 'mission-control:server'
  - 'mission-control:client'

### Updated

- Aligned 'client' and 'mission-control:client' to use the same Tailwind package and input.css

## MicroCODE App Template [0.0.0]

`November 17, 2025`

### Changed

- Build MicroCODE-App-Template from `Gravity-Current` (2025-11 version)
- Stored as a new GitHub Organization
- Updated all repository remote URLs to new organization
- All Route, Model, View, and Controller references updated to MicroCODE-App-Template namespace
- e.g.: from `/model/{entity}.js` to `/model/{entity}.model.js`
- This is preparation for migrating to our `ENTITY centric architecture` and templating
- Configuration-driven entity management
- Automatic API and documentation generation
- Added all `mcode-*` packages to server and clients for our logging, list processing, cache, and utilities

### Added

- `Developer Setup Guide` with comprehensive MongoDB installation and configuration
- Organization profile README with enterprise positioning and architecture overview

## GRAVITY APP (version details from time of clone)

## GRAVITY SERVER [11.0.10]

`November 13, 2025`

### Added (Gravity Zero, Ltd.)

- Layered API architecture (UI, UX, DB, IO layers)
- Dynamic entity configuration system
- HTMX integration for hypermedia-driven UI
- Auto-generated Swagger and JSDoc documentation
- Entity resolution for UUID-to-text conversion

### Enhanced (Gravity Zero, Ltd.)

- Seven-level RBAC privilege system
- Redis integration for caching and pub/sub
- Health monitoring and diagnostics
- Support ticket system
- Admin management tools

## GRAVITY MISSION CONTROL [2.1.1]

`August 2025`

### Added (Gravity Zero, Ltd.)

- Same features as Gravity Server above

### Changed (Gravity Zero, Ltd.)

- Same enhancements as Gravity Server above

### Fixed (Gravity Zero, Ltd.)

- Same fixes as Gravity Server above

## GRAVITY WEB CLIENT [12.0.18]

`December 2024`

### Added (Gravity Zero, Ltd.)

- Multi-lingual support (i18n) across all clients
- Shadcn/UI component integration
- Enhanced server architecture with graceful shutdown and SIG handling
- MicroCODE package suite (mcode-\* packages)
- Bootstrap system for environment initialization
- MongoDB \_id as primary key (removed secondary UUID)

### Changed (Gravity Zero, Ltd.)

- Upgraded to Express 5
- Enhanced error handling and logging
- Improved port management and configuration

### Fixed (Gravity Zero, Ltd.)

- Client strictPort configuration for multi-client testing

## GRAVITY CLIENT MOBILE APP [8.0.0]

`December 2024`

### Added (Gravity Zero, Ltd.)

- React Native mobile client implementation
- Expo integration for iOS and Android
- Multi-platform navigation system
- Mobile-optimized components

### Changed (Gravity Zero, Ltd.)

- Unified component library across web and mobile
- Cross-platform state management

## GRAVITY SUPERNOVA [1.0.0]

`September 2025`

### Changed (Gravity Zero, Ltd.)

- This replaced Kyle's 'Web Site' template with an ASTRO-based static site generator

## GRAVITY FREE BOILERPLATE [1.0.0]

`May 2025`

### Added (Gravity Zero, Ltd.)

- Initial release based on licensed Gravity SaaS Boilerplate
- Authentication system with multi-provider support
- Stripe subscription billing integration
- Admin dashboard (Mission Control)
- Email notification system
- Multi-tenant architecture
- REST API with token authentication
- MySQL/PostgreSQL database support
- React web client
- 40+ integration tests

---

## Version History Notes

### Version Numbering

- `Major.Minor.Patch` (Semantic Versioning)
- `Major`: Breaking changes requiring migration
- `Minor`: New features, backward compatible
- `Patch`: Bug fixes and minor improvements

### Component Versions

- **Server**: `11.0.10`
- **Mission Control**: `2.1.1`
- **Client Web**: `12.0.18`
- **Client Mobile**: `8.0.0`
- **Free Boilerplate**: `1.0.1`
- **Supernova**: `1.0.0`

### Maintenance

- Platform updates evaluated quarterly
- Security patches applied as needed
- Gravity upstream changes reviewed monthly
- Breaking changes communicated 30 days in advance

---

**Note:** This changelog covers the MicroCODE App Template platform.
Individual applications built on this platform maintain their own changelogs.

[0.0.3]: https://github.com/MicroCODE-App-Template/.github/compare/v0.0.3...HEAD
[0.0.2]: https://github.com/MicroCODE-App-Template/.github/compare/v0.0.2...HEAD
[0.0.1]: https://github.com/MicroCODE-App-Template/.github/compare/v0.0.1...HEAD
[0.0.0]: https://github.com/MicroCODE-App-Template/.github/compare/v0.0.0...HEAD
[11.0.10]: https://github.com/MicroCODE-Gravity-Current/server
[2.1.1]: https://github.com/MicroCODE-Gravity-Current/mission-control
[12.0.18]: https://github.com/MicroCODE-Gravity-Current/client-react-web
[8.0.0]: https://github.com/MicroCODE-Gravity-Current/client-react-native
[1.0.0]: https://github.com/MicroCODE-Gravity-Current/free-saas-boilerplate
