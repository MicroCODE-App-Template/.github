# MicroCODE App Template

**Enterprise SaaS Application Platform**

**_⚠️ TEMPLATE REPOSITORY ONLY - ALL CHANGES FLOW DOWNSTREAM INTO OUR WEB APPS_**

This repository contains a working copy of the licensed [Gravity SaaS Boilerplate](https://usegravity.app/) that we have modified for bulding our Apps.

MicroCODE App Template is our production-ready web and mobile application framework, customized from the Gravity SaaS Boilerplate for building enterprise-grade applications.

## Overview

MicroCODE App Template serves as the foundation for all MicroCODE, Inc. SaaS products. It provides a battle-tested, feature-rich platform that eliminates boilerplate development and accelerates time-to-market for new applications.

This is our **active development platform** - the customized derivative of the licensed Gravity boilerplate that we use to build production applications.

📚 **[Developer Setup Guide](../DEVELOPER-SETUP.md)** - Complete installation and configuration instructions
📋 **[Changelog](../CHANGELOG.md)** - Version history and release notes

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

## MicroCODE Customizations

Our platform extends Gravity with enterprise-specific enhancements:

### Enhanced Architecture

- **Layered API Design**: UI, UX, DB, and IO layers for clear separation of concerns
- **Dynamic Entity System**: Single source of truth configuration driving all CRUD operations
- **HTMX Integration**: Hypermedia-driven UI following principles from "Hypermedia Systems"
- **Auto-Generated Documentation**: Swagger and JSDoc generated from configuration
- **Entity Resolution**: Automatic UUID-to-text resolution for display

### Enterprise Features

- **Advanced RBAC**: Seven-level privilege system (Super, Developer, Admin, Engineer, Group Leader, Team Leader, Operator)
- **Redis Integration**: Enhanced caching and pub/sub for real-time features
- **Health Monitoring**: Comprehensive system diagnostics and health checks
- **Support System**: Built-in ticket and help desk management
- **Admin Tools**: Cache, Redis, database, and JSON management utilities

### Developer Experience

- **Configuration-Driven Development**: Define entities once, generate everything automatically
- **Automatic API Generation**: Routes, validation, and documentation from `__config.js`
- **Consistent Patterns**: Standardized CRUD operations across all entities
- **Type Safety**: Enhanced TypeScript support and type definitions
- **CLI Tools**: Custom commands for scaffolding and entity management

## Technology Stack

- **Frontend**: React (Web), React Native (Mobile)
- **Backend**: Node.js with Express
- **UI Framework**: HTMX + HTML/CSS (Hypermedia-driven), Shadcn components
- **Database**: MongoDB, MySQL, PostgreSQL, MariaDB (multi-database support)
- **API**: RESTful with Swagger and JSDoc documentation
- **Authentication**: PassportJS with multi-provider support
- **Payments**: Stripe for subscriptions and billing
- **Caching**: Redis for performance and real-time features
- **Email**: Transactional email with multiple provider support
- **Testing**: Jest, integration tests, end-to-end testing

## Applications Built on MicroCODE App Template

Products built on this platform:

- **LADDERS® as a Service**: Cloud-based PLC compare reporting for industrial automation
- **Regatta RC™**: Race Committee management for sailing regattas
- **[Additional Applications]**: Enterprise SaaS solutions for various industries

## Key Capabilities

### For Product Teams

- Launch new SaaS products in weeks, not months
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

### Layered API Structure

**UI Layer** - Server-Side Rendering

- System operations and commands
- View and component rendering
- Modal components via HTMX
- Banner notifications
- Enum resolution and lookups

**UX Layer** - Client-Side Interactivity

- Media rendering (local/remote)
- Real-time clock and timers
- Interactive components

**DB Layer** - Data Persistence

- Entity CRUD operations
- Schema management
- Query optimization
- Multi-database support

**IO Layer** - External Integration

- Real-time data streaming
- Hardware interfacing
- Health monitoring
- External API integration

## Entity Configuration System

The `__config.js` file serves as the single source of truth:

```javascript
{
    name: 'entity',
    description: 'Entity description for documentation',
    api: ['ui', 'db'],           // Exposed APIs
    nature: 'logical',            // Entity type
    resolves: true,               // UUID resolution
    auth: {
        create: 'Engineer',
        read: 'Operator',
        update: 'Engineer',
        delete: 'Admin',
    }
}
```

This automatically generates:

- Database tables and enums
- API endpoints (GET, POST, PUT, DELETE)
- Swagger documentation
- UI views and forms
- Authorization middleware

## Getting Started

### For New Applications

1. Clone MicroCODE App Template repository
2. Configure entities in `__config.js`
3. Run entity generation CLI
4. Customize views and business logic
5. Deploy with included infrastructure code

### For Contributions

This is an internal platform. Contributions should follow MicroCODE development standards and be reviewed by the architecture team.

## Repository Structure

This GitHub Organization contains:

- **microcode-app-core**: Core platform and framework
- **microcode-app-frontend**: React web application base
- **microcode-app-mobile**: React Native mobile app base
- **microcode-app-api**: Backend API and services
- **microcode-app-docs**: Platform documentation and guides
- **microcode-app-cli**: CLI tools for scaffolding and management
- **microcode-app-components**: Shared component library
- **microcode-app-infra**: Deployment and infrastructure templates

## Maintenance and Updates

### Gravity Updates

We maintain a separate reference copy of the original Gravity boilerplate to track upstream changes. Updates are evaluated and selectively integrated into MicroCODE App Template to maintain compatibility while preserving our customizations.

### Version Management

- Platform version follows semantic versioning
- Breaking changes communicated to all product teams
- Backward compatibility maintained where possible
- Migration guides provided for major updates

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
