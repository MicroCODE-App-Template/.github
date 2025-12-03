# Mongoose Schema Updates - Best Practices

## Overview

When adding new fields to existing MongoDB schemas, there are important considerations about how Mongoose handles updates.

## The Issue

Mongoose has **strict mode enabled by default**. This means:

1. **During inserts**: Mongoose will only save fields that are defined in the schema
2. **During updates**: Mongoose may strip fields that aren't in the schema, OR if there are validation issues

## Solution: Two Approaches

### 1. Regular CRUD Operations (Application Code)

For normal application CRUD operations, **Mongoose works fine** as long as:

- The new fields are **added to the schema definition** in the model file
- You use standard Mongoose methods like `findOneAndUpdate()`, `save()`, etc.

**Example:**
```javascript
// In model/mongo/email.mongo.js
const EmailSchema = new Schema({
    // ... existing fields ...
    new_field: {
        type: String,
        required: false
    }
});

// In your CRUD function
exports.update = async function({ name, data }) {
    // This will work fine - Mongoose respects schema fields
    return await Email.findOneAndUpdate({ name: name }, data);
}
```

### 2. Bulk Operations (Seeding/Migration)

For bulk operations like seeding or migrations, we use the **native MongoDB driver** to bypass Mongoose validation:

**Example (from seed/seeder.js):**
```javascript
// Use native MongoDB driver to bypass Mongoose validation
const mongoose = require('mongoose');
const collectionName = model.schema.collection.name;
const collection = mongoose.connection.db.collection(collectionName);

const updateResult = await collection.updateOne(
    { _id: objectId },
    { $set: updatePayload }
);
```

## Why This Matters

When you add a new field to a schema:

1. **Add it to the schema definition** in the model file (e.g., `model/mongo/email.mongo.js`)
2. **Regular CRUD operations** will work automatically - Mongoose will accept the field
3. **Bulk seeding operations** use the native driver to ensure fields are always saved

## Best Practices

1. **Always update the schema first** when adding new fields
2. **Test regular CRUD operations** - they should work with Mongoose
3. **Seeding uses native driver** - this is intentional and ensures reliability
4. **If you encounter issues** with Mongoose stripping fields during updates, check:
   - Is the field defined in the schema?
   - Is `required: true` but the value is missing?
   - Are you using the correct update method?

## When to Use Native Driver

Use the native MongoDB driver (bypassing Mongoose) for:
- Bulk seeding operations
- Data migrations
- Any operation where you need 100% certainty that fields will be saved

Use Mongoose for:
- Regular application CRUD operations
- Operations that benefit from Mongoose validation
- Operations that need Mongoose middleware/hooks

## Current Implementation

The seeding system (`server/seed/seeder.js`) uses the native driver for updates to ensure:
- New fields are always saved, even during bulk operations
- No schema validation issues prevent field updates
- Consistent behavior when adding fields to existing records
