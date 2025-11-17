# MicroCODE App Template

A SaaS template built from the Gravity App Boilerplate.

This template was created from Kyle's GRAVITY-APP product in 2024, when MicroCODE `purchased` it and began modifications. This template combines the best features from `gravity-current` and `ladders` to serve as a common base for building new MicroCODE SaaS applications.

This template includes all major features:

- **Multi-Lingual Support (i18n)**
- **Shadcn/UI Integration**
- **Express 5 Compatibility**
- **Enhanced Server Architecture** (SIG handling, graceful shutdown, port management)
- **MicroCODE Package Suite** (mcode-* packages)
- **Bootstrap System** for environment initialization
- **MongoDB _id as Primary Key** (no secondary UUID)

NOTE: For testing multiple Clients against one Server, remove this from `VITE Config`:

```
  strictPort: true
```

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
- mcode-* packages - MicroCODE logging, caching, data, and list processing
- ...

### Documentation

We selected Gravity as our SaaS Boilerplate because their code looked super-clean, consistent, and
they had beautiful online documentation, and training...

[Gravity Documentation](https://docs.usegravity.app/)

### Training

- MicroCODE purchased Gravity Training from Kyle, to access course: [Gravity Training Course](https://click.mlflow.com/link/c/YT0yMzg5NDM0OTg0Mzk5MTE1OTM3JmM9Zjd0NSZlPTQ2NzQ0NTIxJmI9OTczMTkwMTA1JmQ9cTFuNHoyeQ==.E8Etpdc01Kz2FeMURvW6oqmj9O6FqnopOVIiujHd1Z8)

### Installing

- How/where to download your program
- Any modifications needed to be made to files/folders

- install all dependencies...

```
npm install
```

#### MongoDB

MongoDB is our chosen DB for our SaaS.

To create a clean database for testing...

- Ensure you have admin privileges in the current MongoDB environment.
- To establish this from nothing...
- Install MongoDB and start with no security...
- Stop MongoDB - to erase current Dev environment

```
net stop mongodb
```

- UNINSTALL MongoDB from ADD/REMOVE PROGRAMS...

- Delete existing MongoDB Data Files...

```
cd "C:\Program Files\MongoDB\Server\<version>\data\db"
cd "C:\Program Files\MongoDB\Server\8.2\data\db"
del *.*
```

- Delete existing MongoDB Log Files...

```
cd "C:\Program Files\MongoDB\Server\<version>\log"
cd "C:\Program Files\MongoDB\Server\6.0\log"
del *.*
```

- DOWNLOAD & INSTALL MongoDB from scratch.
- INSTALL MONGO DB SHELL...

```
winget install MongoDB.Shell
```

- Turn off default security...
  (Edit **mongod.cfg** and comment out these lines _if_ present)

```
cd "C:\Program Files\MongoDB\Server\<version>\bin"
cd "C:\Program Files\MongoDB\Server\8.2\bin"
edit mongod.cfg

#security:
#  authorization: "enabled"
```

- Start MongoDB -- if you just installed as a Windows Service it should have start automatically...

```
net start mongodb
```

- Shell into MongoDB (mongosh) and create your admin and application accounts...

```
mongosh

# First, create the admin superuser in the admin database
use admin
db.createUser({
  user: "admin",
  pwd: "adm!nPassxord",
  roles: [
    { role: "userAdminAnyDatabase", db: "admin" },
    { role: "readWriteAnyDatabase", db: "admin" },
    { role: "dbAdminAnyDatabase", db: "admin" }
  ]
})

# Then, create the application user in the saasDatabase
use saasDatabase
db.createUser({
  user: "saasAdmin",
  pwd: "saasPassword",
  roles: [
    { role: "readWrite", db: "saasDatabase" },
    { role: "dbAdmin", db: "saasDatabase" }
  ]
})

# Verify the users were created
use admin
db.getUsers()
use saasDatabase
db.getUsers()
```

- Stop MongoDB

```
net stop mongodb
```

- Enable security... (Edit **mongod.cfg** enable these lines)
- NOTE: Remove `#` characters from both lines

```
security:
  authorization: "enabled"
```

- Restart MongoDB...

```
net start mongodb
```

- Test the saasAdmin connection...

```
mongosh "mongodb://saasAdmin:saasPassword@localhost:27017/saasDatabase" --eval "db.getName()"
```

- Move to the SERVER directory (MDS Command: **SERVER**)
- In the SERVER CLI, run the new MicroCODE scripts to create all static DB Tables

```
$env:NODE_OPTIONS = "--openssl-legacy-provider"
docker volume create mcode-mongodb-volume
npm run seed:all
```

- You should see these messages...
<p align="left"><img src=".\.github\images\mongodb-run-seeds.png" width="720" title="Init new MongoDB" style="border: 0.5px solid lightgray;"></p>

- Then check the "saasDatabase" in mongosh...
<p align="left"><img src=".\.github\images\mongodb-drop-database.png" width="720" title="Delete MongoDB" style="border: 0.5px solid lightgray;"></p>

# Production SECURITY

## What's Currently Set Up: ✅

- Authorization enabled
- User authentication required
- Role-based access control (saasAdmin has proper roles)
- Connection authentication working
- What You Should Consider Adding:

## Network Security

MongoDB is bound to 127.0.0.1 (localhost only) ✅ Good for development

- For production, keep it this way or use VPN/firewall rules

## Stronger Password

- Currently using saasPassword - change to a strong, random password for production

## Principle of Least Privilege ✅

- **Admin user (`admin`)**: Full access to all databases - use only for maintenance and user management
- **Application user (`saasAdmin`)**: Limited to `saasDatabase` only - used by the application
  - This isolation ensures leaked application credentials don't compromise other databases
- Consider creating additional users:
  - Read-only user for reporting
  - Backup user with minimal permissions

## Additional Security Measures (for production):

- Enable TLS/SSL encryption
- Enable audit logging
- Set up IP whitelisting
- Regular backups with encryption
- Rotate credentials periodically
- For your current development setup, you have everything you need. Just remember to:

Use a strong password before going to production
Never commit .env file with real credentials to git
Consider environment-specific users (dev vs production)

### Executing program

- Move to the SERVER directory (MDS Command: **SERVER**)
- Start the development server (this also starts the Web Browser Client)

```
npm run dev
```

<p align="left"><img src=".\.github\images\npm-run-dev-1.png" width="720" title="Server startup" style="border: 0.5px solid lightgray;"></p>

The Client launches here, but is not 100% available until the Server completes...

<p align="left"><img src=".\.github\images\npm-run-dev-2.png" width="720" title="Client startup" style="border: 0.5px solid lightgray;"></p>

<p align="left"><img src=".\.github\images\npm-run-dev-3.png" width="720" title="App Login" style="border: 0.5px solid lightgray;"></p>

## Help

MicroCODE has a support contract with Gravity Ltd, Kyle Gawley.

## Terminology

| Word or Acronym | Description/Definition                                                                                                                              |
| --------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| **SaaS**        | Software as a Service (SaaS) - subscription based access to a software product or service.                                                          |
| **Gravity**     | A SaaS Boilerplate, from which this App Template is constructed.                                                                                             |
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

- 0.0.0
  - Initial Release

## License

This project is licensed under a PRIVATE License - see the LICENSE.md file for details:
[MicroCODE SaaS License](./LICENSE.md)

The App Template it is based upon is licensed under their PRIVATE license,
see [GRAVITY Software License Agreement](./server/LICENSE.md)

## Acknowledgments

- [App Boilerplate](https://usegravity.app/)
