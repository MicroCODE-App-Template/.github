# Changelog

All notable changes to the MicroCODE App Template will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Changed
- Migrated organization from MicroCODE-Web-App to MicroCODE-App-Template
- Updated all repository remote URLs to new organization

### Added
- Developer Setup Guide with comprehensive MongoDB installation and configuration
- Organization profile README with enterprise positioning and architecture overview

## [12.0.18] - 2024-12-XX

### Added
- Multi-lingual support (i18n) across all clients
- Shadcn/UI component integration
- Enhanced server architecture with graceful shutdown and SIG handling
- MicroCODE package suite (mcode-* packages)
- Bootstrap system for environment initialization
- MongoDB _id as primary key (removed secondary UUID)

### Changed
- Upgraded to Express 5
- Enhanced error handling and logging
- Improved port management and configuration

### Fixed
- Client strictPort configuration for multi-client testing

## [11.0.10] - 2024-XX-XX

### Added
- Layered API architecture (UI, UX, DB, IO layers)
- Dynamic entity configuration system
- HTMX integration for hypermedia-driven UI
- Auto-generated Swagger and JSDoc documentation
- Entity resolution for UUID-to-text conversion

### Enhanced
- Seven-level RBAC privilege system
- Redis integration for caching and pub/sub
- Health monitoring and diagnostics
- Support ticket system
- Admin management tools

## [8.0.0] - 2024-XX-XX

### Added
- React Native mobile client implementation
- Expo integration for iOS and Android
- Multi-platform navigation system
- Mobile-optimized components

### Changed
- Unified component library across web and mobile
- Cross-platform state management

## [1.0.0] - 2024-01-XX

### Added
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

### MicroCODE Customizations
- Configuration-driven entity management
- Automatic API and documentation generation
- Enhanced security and role-based access control
- Custom admin tools and utilities

---

## Version History Notes

### Version Numbering
- **Major.Minor.Patch** (Semantic Versioning)
- Major: Breaking changes requiring migration
- Minor: New features, backward compatible
- Patch: Bug fixes and minor improvements

### Component Versions
- **Client Web**: 12.0.18
- **Client Mobile**: 8.0.0
- **Server**: 11.0.10
- **Mission Control**: 2.1.1
- **Free Boilerplate**: 1.0.1
- **Supernova**: 1.0.0

### Maintenance
- Platform updates evaluated quarterly
- Security patches applied as needed
- Gravity upstream changes reviewed monthly
- Breaking changes communicated 30 days in advance

---

**Note:** This changelog covers the MicroCODE App Template platform. Individual applications built on this platform maintain their own changelogs.

[Unreleased]: https://github.com/MicroCODE-App-Template/.github/compare/v12.0.18...HEAD
[12.0.18]: https://github.com/MicroCODE-App-Template/.github/releases/tag/v12.0.18
[11.0.10]: https://github.com/MicroCODE-App-Template/.github/releases/tag/v11.0.10
[8.0.0]: https://github.com/MicroCODE-App-Template/.github/releases/tag/v8.0.0
[1.0.0]: https://github.com/MicroCODE-App-Template/.github/releases/tag/v1.0.0
