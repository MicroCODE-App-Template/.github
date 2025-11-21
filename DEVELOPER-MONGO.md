# MongoDB Set-Up for Development

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
cd "C:\Program Files\MongoDB\Server\8.2\log"
del *.*
```

- DOWNLOAD & INSTALL MongoDB from scratch.
- INSTALL MONGO DB SHELL...

```
winget install MongoDB.Shell
```

- Turn off default security...
  (Edit **mongod.cfg** and comment out (`#`) these lines _if_ present)

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

# First, confirm the existing admin/saas accounts (do not delete them)
use admin
db.getUsers()
# If the `admin` superuser is missing, create it with the command below, otherwise skip.
# db.createUser({ ... })

# Next, create the new application user in the `appDatabase`
use appDatabase
db.createUser({
  user: "appAdmin",
  pwd: "appPassword",
  roles: [
    { role: "readWrite", db: "appDatabase" },
    { role: "dbAdmin", db: "appDatabase" }
  ]
})

# Verify the users were created (admin + appDatabase coexist with existing saasDatabase)
use admin
db.getUsers()
use appDatabase
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

- Test the new `appAdmin` connection for this template...

```
mongosh "mongodb://appAdmin:appPassword@localhost:27017/appDatabase" --eval "db.getName()"
```

- Seed the MongoDB database with initial data
  - Move to the SERVER directory (`cd server`)
  - In the SERVER CLI, run the new MicroCODE scripts to create all static DB Tables

```
npm run seed:all
```

---

**Database Connection Issues:**

Check your `console/.env` file has:

```bash
DB_CLIENT=mongo
DB_HOST=localhost
DB_USER=appAdmin
DB_PASSWORD=appPassword
DB_NAME=appDatabase
DB_PORT=27017
```

Or use the MONGO_URI format:

```bash
MONGO_URI=mongodb://appAdmin:appPassword@localhost:27017/appDatabase
```
