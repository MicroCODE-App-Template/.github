# MicroCODE App Template

**Enterprise SaaS Application Platform**

**_⚠️ TEMPLATE REPOSITORY ONLY - ALL CHANGES FLOW DOWNSTREAM INTO OUR WEB APPS_**

This repository contains a working copy of the licensed [Gravity SaaS Boilerplate](https://usegravity.app/) that we have modified for bulding our Apps.

MicroCODE App Template is our production-ready web and mobile application framework, customized from the Gravity SaaS Boilerplate for building enterprise-grade applications.

## Create ISSUEs via Forms

**All issues are tracked in the centralized [`.issue`](https://github.com/MicroCODE-App-Template/.issue) repository.**

Use these links to open form-based issues for each type of work item:

### ⛔ Hotfix - defect in PRODUCTION

- **Report/Request:** [Defect Report](https://github.com/MicroCODE-App-Template/.issue/issues/new?template=defect_report.yml)
- **Template MD:** [Hotfix Template](https://github.com/MicroCODE-App-Template/.issue/issues/new?template=hotfix/hotfix-nnnn--hotfix-short-name.md)

### 🎃 Bug/Bugfix - defect in BETA or ALPHA Release

- **Report/Request:** [Bug Report](https://github.com/MicroCODE-App-Template/.issue/issues/new?template=bug_report.yml) | [Defect Report](https://github.com/MicroCODE-App-Template/.issue/issues/new?template=defect_report.yml)
- **Template MD:** [Bugfix Template](https://github.com/MicroCODE-App-Template/.issue/issues/new?template=bugfix/bugfix-nnnn--bugfix-short-name.md)

### 😎 Feature - new request for PRODUCTION

- **Report/Request:** [Feature Request](https://github.com/MicroCODE-App-Template/.issue/issues/new?template=feature_request.yml)
- **Template MD:** [Feature Template](https://github.com/MicroCODE-App-Template/.issue/issues/new?template=feature/feature-nnnn--feature-short-name.md)

### 📦 Release - request for new collection of features in PRODUCTION

- **Report/Request:** [Release Request](https://github.com/MicroCODE-App-Template/.issue/issues/new?template=release_request.yml)
- **Template MD:** [Release Template](https://github.com/MicroCODE-App-Template/.issue/issues/new?template=release/release-bM.F.0.md)

### 📋 Task - backend work in DEVELOPMENT

- **Report/Request:** [Task Request](https://github.com/MicroCODE-App-Template/.issue/issues/new?template=task_request.yml)
- **Template MD:** [Task Template](https://github.com/MicroCODE-App-Template/.issue/issues/new?template=task/task-nnnn--task-short-name.md)

### 🛡️ Security - vulnerability in PRODUCTION

- **Report/Request:** [Security Vulnerability Report](https://github.com/MicroCODE-App-Template/.issue/issues/new?template=security_vulnerability.yml)
- **Template MD:** [Security Template](https://github.com/MicroCODE-App-Template/.issue/issues/new?template=security/security-nnnn--vulnerability-short-name.md)

### ⚙️ Configuration - change to Solution set-up in DEVELOPMENT

- **Report/Request:** [Configuration Change](https://github.com/MicroCODE-App-Template/.issue/issues/new?template=config.yml)

**Note:** All issues are created in the `.issue` repository to maintain centralized tracking across all solution repositories (admin, app, client, portal, server).

## Overview

MicroCODE App Template serves as the foundation for all MicroCODE, Inc. SaaS products. It provides a battle-tested, feature-rich platform that eliminates boilerplate development and accelerates time-to-market for new applications.

This is our **active development platform** - the customized derivative of the licensed Gravity boilerplate that we use to build production applications.

This GitHub Organization has (5) repositories that form a complete App Solution:

- `admin` — Admin console/backend [Gravity's "mission-control"]
- `app` — Mobile app (React Native for iOS and Android) [Gravity's "client-react-native"]
- `client` — Web client [Gravity's "client-react-web"]
- `portal` — Portal/marketing site (Astro) [Gravity's "supernova"]
- `server` — Backend API server [Gravity's "server"]

This is a multi-repo setup where each component is versioned independently but works together.

Common benefits:

- Independent versioning and releases
- Separate access controls
- Clear separation of concerns
- Easier to scale teams per component

📚 **[Developer Setup Guide](../docs/DEV/DEVELOPER-SETUP.md)** - Complete installation and configuration instructions
📋 **[Changelog](../CHANGELOG.md)** - Version history and release notes

### Version Management

- Platform version follows semantic versioning (vM.F.p), see **[App Versioning GIT Rules](../docs/DEV/DEVELOPER-GIT.md)**
- Breaking changes communicated to all product teams
- Backward compatibility maintained whereever possible
- Migration guides provided for major updates

## Built From Gravity

MicroCODE App Template is our enhanced version of the [Gravity SaaS Boilerplate](https://usegravity.app/) by Gravity Zero Ltd., with MicroCODE-specific customizations and enterprise features.

### Core Gravity Features Included

- **Authentication System**: Email/password, magic links, 500+ social networks, 2FA
- **Subscription Billing**: Stripe integration with trials, free plans, seat/usage billing
- **UI Components**: 50+ components, 25+ views, dark mode, Shadcn components
- **Multi-Tenant Architecture**: Organizations, invitations, user roles
- **Mobile Apps**: Native iOS & Android with React Native
- **Admin Dashboard**: MicroCODE Console for metrics, user management, logs
- **Email System**: 20+ multilingual templates, 20+ email services
- **API Infrastructure**: REST API with token auth, throttling, security
- **Testing**: 40+ integration tests
- **AI Support**: Cursor & Windsurf rules for accelerated development

### MicroCODE Customizations

Our platform extends Gravity with enterprise-specific enhancements:

#### Enhanced Architecture

These features were added by MicroCODE, Inc. befoe using Gravity for production apps:

- **Layered API Design**: UI, UX, DB, and IO layers for clear separation of concerns
- **Dynamic Entity System**: Single source of truth configuration driving all CRUD operations,
  this required a complete reorganization of the Gravity code to be 'Entity Centric'
- **HTMX Integration**: Hypermedia-driven UI following principles from "Hypermedia Systems"
- **Auto-Generated Documentation**: Swagger and JSDoc generated from configuration
- **Entity Resolution**: Automatic UUID-to-text resolution for display

#### Enterprise Features

- **Redis Integration**: Enhanced caching and pub/sub for real-time features
- **Health Monitoring**: Comprehensive system diagnostics and health checks
- **Support System**: Built-in ticket and help desk management
- **Admin Tools**: Cache, Redis, database, and JSON management utilities

#### Developer Experience

- **Configuration-Driven Development**: Define entities once, generate everything automatically
- **Automatic API Generation**: Routes, validation, and documentation from `__config.js`
- **Consistent Patterns**: Standardized CRUD operations across all entities
- **CLI Tools**: Custom commands for scaffolding and entity management (setup.all and teardown.all)

## Technology Stack - MERN

- **App**: `React` (Web), `React Native` (Mobile)
- **Server**: `Node.js` with `Express` routing
- **UI Framework**: `React` with Shadcn(Radix-UI) components
- **Database**: `MongoDB`, MySQL, PostgreSQL, MariaDB (Mongo or SQL database support)
- **API**: RESTful with `Swagger` and `JSDoc` documentation
- **Authentication**: `PassportJS` with multi-provider support
- **Payments**: `Stripe` for subscriptions and billing
- **Caching**: `Redis` for performance and real-time features
- **Email**: `Mailgun` Transactional email with optional multiple provider support
- **Testing**: `Mocha` and `Chai`, integration tests, end-to-end testing

## Applications Built on MicroCODE App Template

Products built on this platform:

- **LADDERS® as a Service**: Cloud-based PLC compare reporting for industrial automation
- **Regatta RC™**: Race Committee management and Recision Scoring for sailing regattas
- **[Additional Applications]**: Enterprise SaaS solutions for various industries

## Key Capabilities

### For Product Teams

- Launch new SaaS products in days or weeks, not months
- Focus on core features instead of authentication, billing, and infrastructure
- Consistent UX patterns across all MicroCODE products
- Shared component library and design system
- Pre-built admin and user management

### For Development Teams

- Configuration-driven entity management
- Automatic API and documentation generation
- Built-in security and authentication
- Scalable multi-tenant architecture
- Mobile-ready from day one

### For Operations Teams

- Comprehensive monitoring and health checks
- Built-in support ticket system
- Admin tools for cache and database management
- Audit logging and compliance features
- Multi-database support for flexibility

## Architecture Overview

### Complete Solution Structure

- **Admin** - backend control and maintenance
- **App** - mobile Apps for iOS and Android
- **Client** - web frontend
- **Portal** - cloud marketing site
- **Server** - cloud backend

## SERVER: Entity Configuration System

The `__config.js` file serves as the single source of truth:

This automatically generates:

- Database Tables and enums for Tupe and State
- API endpoints (GET, POST, PUT, DELETE)
- Swagger documentation
- UI views and forms
- Authorization middleware

## Getting Started

### For New Applications

1. Clone MicroCODE App Template repository
2. Follow DEVERLOPER-SETUP instructions.
3. Configure entities in `__config.js`
4. Run entity generation CLI
5. Customize views and business logic
6. Deploy with included infrastructure code

### For Contributions

This is an internal platform. Contributions should follow MicroCODE development standards and be reviewed by the architecture team.

## Repository Structure

This GitHub Organization `MicroCODE-App-Template` contains:

- `admin` - backend control and maintenance
- `app` - mobile Apps for iOS and Android
- `client` - web frontend
- `portal` - cloud marketing site
- `server` - cloud backend

## Maintenance and Updates

### Gravity Updates

We maintain a separate reference copy of the original Gravity boilerplate to track upstream changes. Updates are evaluated and selectively integrated into `MicroCODE App Template` to maintain compatibility while preserving our customizations. **NOTE**: Our App Template has diveraged greatly from the `Gravity App` solution and it can only be used as a reference or detailed example going forward.

## License

© 2022-2025 MicroCODE, Inc. All rights reserved.

This software is proprietary to MicroCODE, Inc. and contains confidential information.

**License Terms:**

- ✅ Internal use for MicroCODE products
- ✅ Derivative applications allowed
- ✅ Modifications and customizations permitted
- ❌ External distribution prohibited
- ❌ No resale or licensing to third parties

**Built on:** Gravity SaaS Boilerplate (Licensed from Gravity Zero Ltd.)

## Support

For platform support, feature requests, or bug reports:

**Platform Team**
📧 dev-team@mcode.com
💬 Internal Slack: #microcode-app-template

**MicroCODE, Inc.**
55 E. Long Lake Rd #224
Troy, MI 48085

📞 +1 855.421.1010
📧 company@mcode.com
🌐 [www.mcode.com](https://www.mcode.com)

---

_The foundation for MicroCODE's SaaS product portfolio._
