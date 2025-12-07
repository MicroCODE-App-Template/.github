# Club Data Structure Diagram

## Data Structure Visualization

```
┌─────────────────────────────────────────────────────────────────┐
│                      Club Data Object                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ Required Fields (Auto-generated/Defaults)              │   │
│  ├─────────────────────────────────────────────────────────┤   │
│  │ • id: String (auto-generated)                          │   │
│  │ • name: String (required)                              │   │
│  │ • type: 'club' (default)                                │   │
│  │ • state: 'active' (default)                            │   │
│  │ • created_at: Date (auto)                               │   │
│  │ • updated_at: Date (auto)                               │   │
│  │ • rev: Number (auto)                                    │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ Basic Information (Already Present)                      │   │
│  ├─────────────────────────────────────────────────────────┤   │
│  │ • key: String (unique identifier)                       │   │
│  │   Example: 'byc', 'cyc', 'dyc'                         │   │
│  │                                                          │   │
│  │ • name: String (full club name)                         │   │
│  │   Example: 'Bayview Yacht Club'                         │   │
│  │                                                          │   │
│  │ • display: String (abbreviation)                        │   │
│  │   Example: 'BYC', 'CYC', 'DYC'                         │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ Contact Information (TO COLLECT)                        │   │
│  ├─────────────────────────────────────────────────────────┤   │
│  │                                                          │   │
│  │ email: String                                            │   │
│  │   └─> Primary email address                             │   │
│  │       Example: 'info@byc.com'                          │   │
│  │                                                          │   │
│  │ email_: Object                                          │   │
│  │   ├─> primary: String                                   │   │
│  │   ├─> office: String (optional)                        │   │
│  │   └─> work: String (optional)                          │   │
│  │                                                          │   │
│  │ phone: String                                           │   │
│  │   └─> Primary phone number                              │   │
│  │       Format: '13138221853' (no dashes/spaces)         │   │
│  │                                                          │   │
│  │ phone_: Object                                          │   │
│  │   ├─> primary: String                                   │   │
│  │   ├─> office: String (optional, may include ext)        │   │
│  │   │   Format: '13138221853;105' (number;extension)    │   │
│  │   └─> gate: String (optional)                          │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ Location Information (TO COLLECT)                       │   │
│  ├─────────────────────────────────────────────────────────┤   │
│  │                                                          │   │
│  │ address: Object                                         │   │
│  │   ├─> street: String                                    │   │
│  │   │   Example: '100 Clairpointe Street'                │   │
│  │   ├─> city: String                                     │   │
│  │   │   Example: 'Detroit'                               │   │
│  │   ├─> state: String                                    │   │
│  │   │   US: 'MI', 'IL', 'OH', etc.                      │   │
│  │   │   Canada: 'ON', 'BC', etc.                        │   │
│  │   ├─> zip: String                                      │   │
│  │   │   US: '48215' (ZIP code)                          │   │
│  │   │   Canada: 'N9A 1A1' (postal code)                │   │
│  │   └─> country: String                                  │   │
│  │       'USA' or 'CAN'                                   │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ Descriptive Information (TO COLLECT)                    │   │
│  ├─────────────────────────────────────────────────────────┤   │
│  │                                                          │   │
│  │ description: String                                     │   │
│  │   └─> Comprehensive club description                    │   │
│  │       • Minimum 100 words                               │   │
│  │       • Include founding year                           │   │
│  │       • Include location details                        │   │
│  │       • Include key features/amenities                  │   │
│  │       • Include historical context                      │   │
│  │       • Professional, factual tone                      │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ Visual Assets (TO COLLECT)                              │   │
│  ├─────────────────────────────────────────────────────────┤   │
│  │                                                          │   │
│  │ logo: String                                            │   │
│  │   └─> Base64 data URL                                   │   │
│  │       Format: 'data:image/x-icon;base64,{data}'        │   │
│  │       Or empty: 'data:image/x-icon;base64,'            │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
```

## BYC Template Structure (Reference)

```javascript
{
    // Basic (Already Present)
    key: 'byc',
    name: 'Bayview Yacht Club',
    display: 'BYC',

    // Contact (TO COLLECT)
    email: 'info@byc.com',
    email_: {
        primary: 'info@byc.com',
        office: 'matt@byc.com',
    },
    phone: '13138221853',
    phone_: {
        primary: '13138221853',
        office: '13138221853;105',
    },

    // Location (TO COLLECT)
    address: {
        street: '100 Clairpointe Street',
        city: 'Detroit',
        state: 'MI',
        zip: '48215',
        country: 'USA',
    },

    // Description (TO COLLECT)
    description: 'Founded in 1915 by four sailors, Bayview Yacht Club is located in Detroit, Michigan. The club moved to its present location at the foot of Clairpointe in 1929-30. BYC is known for hosting the annual Bayview Mackinac Race (originally Port Huron to Mackinac Boat Race) since 1925. The club features two harbors with over 100 wells for vessels, a crane for launching boats up to 50 feet, and an 8,000-square-foot clubhouse with dining room, bar, and banquet space.',

    // Logo (TO COLLECT)
    logo: 'data:image/x-icon;base64,'
}
```

## Data Collection Workflow

```
START
  │
  ├─> Identify Club
  │     │
  │     ├─> Check existing data
  │     │     │
  │     │     ├─> Complete? → SKIP
  │     │     └─> Incomplete? → CONTINUE
  │     │
  │     └─> Web Search
  │           │
  │           ├─> Official Website
  │           ├─> Contact Page
  │           ├─> About Page
  │           └─> Social Media
  │
  ├─> Extract Contact Info
  │     │
  │     ├─> Email (primary)
  │     ├─> Email (office/admin)
  │     ├─> Phone (primary)
  │     └─> Phone (office/gate)
  │
  ├─> Extract Location Info
  │     │
  │     ├─> Street Address
  │     ├─> City
  │     ├─> State/Province
  │     ├─> ZIP/Postal Code
  │     └─> Country
  │
  ├─> Research History
  │     │
  │     ├─> Founding Year
  │     ├─> Historical Context
  │     ├─> Notable Features
  │     └─> Key Events/Races
  │
  ├─> Locate Logo
  │     │
  │     ├─> Found? → Convert to Base64
  │     └─> Not Found? → Use Empty Placeholder
  │
  ├─> Compile Description
  │     │
  │     ├─> Write comprehensive description
  │     ├─> Include founding year
  │     ├─> Include location details
  │     ├─> Include features/amenities
  │     └─> Include historical context
  │
  ├─> Format Data
  │     │
  │     ├─> Follow BYC template
  │     ├─> Validate formats
  │     └─> Ensure consistency
  │
  ├─> Validate
  │     │
  │     ├─> Schema compliance
  │     ├─> Field formats
  │     └─> Data quality
  │
  └─> Update File
        │
        └─> Insert into club.data.js
              │
              └─> END
```

## Field Mapping Reference

| Field             | Type   | Required | Format/Example                 | Source                    |
| ----------------- | ------ | -------- | ------------------------------ | ------------------------- |
| `key`             | String | Yes      | 'byc', 'cyc'                   | Already exists            |
| `name`            | String | Yes      | 'Bayview Yacht Club'           | Already exists            |
| `display`         | String | No       | 'BYC', 'CYC'                   | Already exists            |
| `email`           | String | No       | 'info@byc.com'                 | Website contact page      |
| `email_.primary`  | String | No       | 'info@byc.com'                 | Website contact page      |
| `email_.office`   | String | No       | 'matt@byc.com'                 | Website contact page      |
| `phone`           | String | No       | '13138221853'                  | Website contact page      |
| `phone_.primary`  | String | No       | '13138221853'                  | Website contact page      |
| `phone_.office`   | String | No       | '13138221853;105'              | Website contact page      |
| `phone_.gate`     | String | No       | '13138221853'                  | Website contact page      |
| `address.street`  | String | No       | '100 Clairpointe Street'       | Website/Google Maps       |
| `address.city`    | String | No       | 'Detroit'                      | Website/Google Maps       |
| `address.state`   | String | No       | 'MI', 'ON'                     | Website/Google Maps       |
| `address.zip`     | String | No       | '48215', 'N9A 1A1'             | Website/Google Maps       |
| `address.country` | String | No       | 'USA', 'CAN'                   | Website/Google Maps       |
| `description`     | String | No       | 100+ word text                 | Website/History/Wikipedia |
| `logo`            | String | No       | 'data:image/x-icon;base64,...' | Website/Logo finder       |

## Data Quality Checklist

### Contact Information

- [ ] Email format valid (contains @ and domain)
- [ ] Phone number format consistent (no dashes/spaces)
- [ ] Extension format correct (number;extension)
- [ ] Primary contact info present

### Location Information

- [ ] Street address complete
- [ ] City name correct
- [ ] State/Province code correct (2-letter)
- [ ] ZIP/Postal code format correct
- [ ] Country code correct ('USA' or 'CAN')
- [ ] Address verified on map

### Descriptive Information

- [ ] Description length ≥ 100 words
- [ ] Founding year included
- [ ] Location mentioned
- [ ] Key features/amenities listed
- [ ] Historical context provided
- [ ] Professional tone maintained

### Visual Assets

- [ ] Logo format correct (base64 data URL)
- [ ] Or empty placeholder used
- [ ] Image format appropriate (if logo found)

## Format Examples

### US Club Example

```javascript
{
    key: 'cyc',
    name: 'Chicago Yacht Club',
    display: 'CYC',
    email: 'info@chicagoyachtclub.org',
    email_: {
        primary: 'info@chicagoyachtclub.org',
        office: 'office@chicagoyachtclub.org',
    },
    phone: '13128610000',
    phone_: {
        primary: '13128610000',
        office: '13128610000;101',
    },
    address: {
        street: '400 East Monroe Street',
        city: 'Chicago',
        state: 'IL',
        zip: '60603',
        country: 'USA',
    },
    description: '...',
    logo: 'data:image/x-icon;base64,'
}
```

### Canadian Club Example

```javascript
{
    key: 'wyc',
    name: 'Windsor Yacht Club',
    display: 'WYC',
    email: 'info@windsoryachtclub.com',
    email_: {
        primary: 'info@windsoryachtclub.com',
    },
    phone: '15192555555',
    phone_: {
        primary: '15192555555',
    },
    address: {
        street: '1000 Riverside Drive West',
        city: 'Windsor',
        state: 'ON',
        zip: 'N9A 1A1',
        country: 'CAN',
    },
    description: '...',
    logo: 'data:image/x-icon;base64,'
}
```
