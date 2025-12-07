# Club Data Collection Implementation Plan

## Executive Summary

This document outlines the detailed implementation plan for collecting and inserting missing data for all yacht clubs in `server/seed/data/club.data.js`. The plan follows the BYC (Bayview Yacht Club) entry as the reference template and adheres to the schema rules defined in `server/model/mongo/club.mongo.js`.

## 1. Current State Analysis

### 1.1 Data File Structure

- **File**: `server/seed/data/club.data.js`
- **Total Clubs**: 20 clubs
- **Complete Entries**: 3 clubs (BYC, DYC, GLYC)
- **Incomplete Entries**: 17 clubs requiring data collection

### 1.2 Reference Template (BYC)

The BYC entry serves as the complete reference with the following structure:

```javascript
{
    key: 'byc',
    name: 'Bayview Yacht Club',
    display: 'BYC',
    email: 'info@byc.com',
    email_: {
        primary: 'info@byc.com',
        office: 'matt@byc.com',
    },
    phone: '13138221853',  // String format
    phone_: {
        primary: '13138221853',
        office: '13138221853;105',
    },
    address: {
        street: '100 Clairpointe Street',
        city: 'Detroit',
        state: 'MI',
        zip: '48215',
        country: 'USA',
    },
    description: '...', // Detailed historical description
    logo: 'data:image/x-icon;base64,' // Empty base64 placeholder
}
```

### 1.3 Schema Requirements (`club.mongo.js`)

**Required Fields:**

- `id` (auto-generated)
- `name` (required)
- `type` (default: 'club')
- `state` (default: 'active')

**Optional Fields:**

- `key` (unique)
- `display`
- `email` (string)
- `email_` (object with primary, office, work, etc.)
- `phone` (Object type in schema, but String in data - follow data pattern)
- `phone_` (object with primary, office, gate, etc.)
- `address` (object with street, city, state, zip, country)
- `description` (string)
- `logo` (string - base64 data URL)

### 1.4 Clubs Requiring Data Collection

#### Fully Incomplete (Missing All Optional Fields):

1. **CSYC** - Crescent Sail Yacht Club
2. **CYC** - Chicago Yacht Club
3. **DBC** - Detroit Boat Club
4. **DSC** - Detroit Sail Club
5. **EBC** - Erie Boat Club
6. **FYC** - Ford Yacht Club
7. **GSC** - Grayhaven Sail Club
8. **GIYC** - Grosse Ile Yacht Club
9. **GPC** - Grosse Pointe Club
10. **GPFBC** - Grosse Pointe Farms Boat Club
11. **GPSC** - Grosse Pointe Sail Club
12. **GPYC** - Grosse Pointe Yacht Club
13. **LSSC** - Lake Shore Sail Club
14. **NCYC** - North Cape Yacht Club
15. **NSSC** - North Star Sail Club
16. **PYC** - Pontiac Yacht Club
17. **PHYC** - Port Huron Yacht Club
18. **SYC** - Sarina Yacht Club (Note: Likely "Sarnia Yacht Club" - Canadian)
19. **TYC** - Toledo Yacht Club
20. **WYC** - Windsor Yacht Club (Canadian)
21. **DCSC** - Detroit Community Sailing Club

#### Partially Complete (May Need Verification/Completion):

- **DYC** - Detroit Yacht Club (has email, phone, address, description)
- **GLYC** - Great Lakes Yacht Club (has email, phone, address, description)

## 2. Data Collection Strategy

### 2.1 Information Sources

1. **Official Club Websites** (Primary source)
2. **Yacht Club Directories** (US Sailing, regional associations)
3. **Google Maps/Business Listings**
4. **Social Media Profiles** (Facebook, Instagram)
5. **Wikipedia Articles** (for historical context)
6. **Local Business Directories**

### 2.2 Data Fields to Collect

For each club, collect the following information:

#### Contact Information

- **Email**: Primary contact email
- **Email\_ Object**:
  - `primary`: Main contact email
  - `office`: Office/admin email (if different)
  - `work`: Work email (if applicable)
- **Phone**: Primary phone number (string format, E.164 or US format)
- **Phone\_ Object**:
  - `primary`: Main phone number
  - `office`: Office phone (may include extension, e.g., "13138221853;105")
  - `gate`: Gate/security phone (if applicable)

#### Location Information

- **Address Object**:
  - `street`: Full street address
  - `city`: City name
  - `state`: State/province code (e.g., "MI", "ON" for Ontario)
  - `zip`: Postal/ZIP code
  - `country`: Country code (e.g., "USA", "CAN")

#### Descriptive Information

- **Description**: Historical and descriptive text (similar length/style to BYC entry)
  - Include founding year
  - Location details
  - Notable features/amenities
  - Historical significance
  - Notable events/races hosted

#### Visual Assets

- **Logo**: Club logo as base64 data URL
  - Format: `'data:image/x-icon;base64,{base64_string}'`
  - If logo unavailable, use empty placeholder: `'data:image/x-icon;base64,'`

### 2.3 Data Quality Standards

1. **Email Validation**: Verify email format and domain existence
2. **Phone Format**: Standardize to E.164 format (e.g., +13138221853) or US format (13138221853)
3. **Address Verification**: Cross-reference with Google Maps for accuracy
4. **Description Quality**:
   - Minimum 100 words
   - Include founding year, location, key features
   - Professional, factual tone
   - Similar style to BYC description

## 3. Implementation Architecture

### 3.1 Data Collection Workflow

```
┌─────────────────────────────────────────────────────────────┐
│                    Data Collection Phase                     │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  For each incomplete club:                                     │
│  1. Web Search → Find official website                       │
│  2. Extract contact information                              │
│  3. Extract address information                              │
│  4. Research historical information                          │
│  5. Locate logo (if available)                               │
│  6. Compile description                                      │
│  7. Validate data completeness                               │
│  8. Format according to BYC template                         │
│                                                               │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Data Validation Phase                     │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  1. Schema compliance check                                  │
│  2. Field format validation                                  │
│  3. Required field verification                              │
│  4. Data consistency review                                  │
│                                                               │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Data Insertion Phase                      │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  1. Update club.data.js file                                 │
│  2. Maintain existing structure                              │
│  3. Preserve existing complete entries                       │
│  4. Follow BYC template format                               │
│                                                               │
└─────────────────────────────────────────────────────────────┘
```

### 3.2 File Structure

**Target File**: `server/seed/data/club.data.js`

**Structure Preservation**:

- Maintain existing `prefix` constant
- Maintain existing `club` array structure
- Preserve order of clubs
- Keep existing complete entries unchanged
- Add missing fields to incomplete entries

### 3.3 Data Format Specifications

#### Email Format

```javascript
email: 'primary@example.com',
email_: {
    primary: 'primary@example.com',
    office: 'office@example.com',  // Optional
    work: 'work@example.com'        // Optional
}
```

#### Phone Format

```javascript
phone: '13138221853',  // String, no dashes/spaces
phone_: {
    primary: '13138221853',
    office: '13138221853;105',  // Extension format: number;extension
    gate: '13138221853'         // Optional
}
```

#### Address Format

```javascript
address: {
    street: '123 Main Street',
    city: 'Detroit',
    state: 'MI',  // US state code or Canadian province code
    zip: '48215', // US ZIP or Canadian postal code
    country: 'USA' // 'USA' or 'CAN'
}
```

#### Description Format

- Minimum 100 words
- Include: founding year, location, key features, historical context
- Professional tone
- Similar style to BYC entry

#### Logo Format

```javascript
logo: "data:image/x-icon;base64,{base64_encoded_image}";
// Or empty placeholder if logo unavailable:
logo: "data:image/x-icon;base64,";
```

## 4. Data Collection Checklist

### 4.1 Per-Club Checklist

For each club, verify collection of:

- [ ] **Basic Information**

  - [ ] `key` (already exists)
  - [ ] `name` (already exists)
  - [ ] `display` (already exists)

- [ ] **Contact Information**

  - [ ] `email` (primary email address)
  - [ ] `email_.primary`
  - [ ] `email_.office` (if available)
  - [ ] `phone` (primary phone number string)
  - [ ] `phone_.primary`
  - [ ] `phone_.office` (if available, may include extension)
  - [ ] `phone_.gate` (if applicable)

- [ ] **Location Information**

  - [ ] `address.street`
  - [ ] `address.city`
  - [ ] `address.state` (or province)
  - [ ] `address.zip` (or postal code)
  - [ ] `address.country`

- [ ] **Descriptive Information**

  - [ ] `description` (comprehensive, 100+ words)

- [ ] **Visual Assets**
  - [ ] `logo` (base64 data URL or empty placeholder)

## 5. Special Considerations

### 5.1 Canadian Clubs

- **SYC** (Sarnia Yacht Club) - Likely in Sarnia, Ontario, Canada
- **WYC** (Windsor Yacht Club) - Likely in Windsor, Ontario, Canada

**Address Format for Canadian Clubs**:

```javascript
address: {
    street: '123 Main Street',
    city: 'Windsor',
    state: 'ON',  // Ontario province code
    zip: 'N9A 1A1',  // Canadian postal code format
    country: 'CAN'
}
```

### 5.2 Phone Number Formatting

- **US Clubs**: Use format `1{area_code}{number}` (e.g., `13138221853`)
- **Canadian Clubs**: Use format `1{area_code}{number}` (e.g., `15191234567`)
- **Extensions**: Format as `{number};{extension}` (e.g., `13138221853;105`)

### 5.3 Missing Information Handling

- If email not found: Leave `email` and `email_` fields undefined (not empty strings)
- If phone not found: Leave `phone` and `phone_` fields undefined
- If address incomplete: Collect as much as possible, verify with maps
- If description unavailable: Create basic description from available information
- If logo unavailable: Use empty placeholder `'data:image/x-icon;base64,'`

### 5.4 Data Verification

- Cross-reference addresses with Google Maps
- Verify phone numbers format
- Check email domain validity
- Ensure descriptions are factual and accurate

## 6. Implementation Steps

### Phase 1: Research & Data Collection

1. For each incomplete club, perform web search
2. Identify official website
3. Extract contact information
4. Extract address information
5. Research historical information
6. Locate logo (if available)
7. Compile comprehensive description

### Phase 2: Data Validation

1. Verify all collected data against schema
2. Check field formats (email, phone, address)
3. Validate description quality
4. Ensure consistency with BYC template

### Phase 3: Data Formatting

1. Format phone numbers consistently
2. Structure email objects correctly
3. Format addresses according to country
4. Convert logos to base64 (if found)
5. Write descriptions in consistent style

### Phase 4: File Update

1. Open `server/seed/data/club.data.js`
2. For each incomplete club entry:
   - Add missing fields following BYC template
   - Maintain proper indentation
   - Preserve existing structure
3. Verify file syntax
4. Ensure no breaking changes

### Phase 5: Verification

1. Review updated file structure
2. Verify all clubs have consistent format
3. Check for syntax errors
4. Validate against schema requirements

## 7. Risk Assessment

### 7.1 Potential Risks

1. **Inaccurate Information**: Web sources may be outdated

   - **Mitigation**: Cross-reference multiple sources, prioritize official websites

2. **Missing Information**: Some clubs may not have public contact info

   - **Mitigation**: Use best available information, leave fields undefined if truly unavailable

3. **Logo Availability**: Logos may not be publicly available

   - **Mitigation**: Use empty placeholder, can be updated later

4. **Data Format Inconsistencies**: Different sources may use different formats

   - **Mitigation**: Standardize all data to match BYC template format

5. **Canadian vs US Format Differences**: Address and phone formats differ
   - **Mitigation**: Follow country-specific formatting rules

### 7.2 Quality Assurance

- Double-check all collected data
- Verify addresses with mapping services
- Ensure phone numbers are in correct format
- Review descriptions for accuracy and completeness

## 8. Success Criteria

### 8.1 Completion Criteria

- [ ] All 20 clubs have complete data entries (or best available)
- [ ] All entries follow BYC template structure
- [ ] All data validates against schema requirements
- [ ] File maintains proper JavaScript syntax
- [ ] No breaking changes to existing complete entries

### 8.2 Quality Criteria

- [ ] All email addresses are properly formatted
- [ ] All phone numbers follow consistent format
- [ ] All addresses are complete and accurate
- [ ] All descriptions are comprehensive (100+ words)
- [ ] All entries maintain consistent formatting style

## 9. Questions for Clarification

Before proceeding with implementation, please clarify:

1. **Logo Handling**:

   - Should I attempt to find and convert logos to base64, or always use empty placeholder?
   - What image format should logos be in? (PNG, JPG, SVG?)
   - What size/resolution should logos be?

2. **Phone Number Format**:

   - Should I use E.164 format (+13138221853) or US format without + (13138221853)?
   - The schema shows `phone` as Object type, but data uses String - should I follow the data pattern (String) or schema (Object)?

3. **Missing Data**:

   - If a club has no public email/phone, should I:
     - Leave fields undefined?
     - Use empty strings?
     - Use placeholder values?

4. **Description Length**:

   - Is there a maximum length for descriptions?
   - Should descriptions include specific information (founding year, amenities, etc.)?

5. **Canadian Clubs**:

   - For Canadian clubs (WYC, SYC), should I use Canadian postal code format?
   - Should state field use province codes (ON, BC, etc.)?

6. **Data Sources**:

   - Are there preferred sources I should prioritize?
   - Should I avoid certain sources?

7. **Verification**:

   - Do you want me to verify data accuracy before insertion?
   - Should I create a validation report?

8. **Existing Complete Entries**:
   - Should I verify/update DYC and GLYC entries, or leave them as-is?

## 10. Timeline Estimate

- **Research Phase**: ~2-3 hours (20 clubs × 6-9 minutes each)
- **Data Validation**: ~30 minutes
- **Formatting & File Update**: ~1 hour
- **Verification**: ~30 minutes
- **Total**: ~4-5 hours

## 11. Deliverables

1. **Updated `club.data.js` file** with all missing data filled in
2. **Data collection log** documenting sources used for each club
3. **Validation report** showing data completeness and quality metrics

---

## Appendix A: Club List Reference

### Complete Entries (Reference)

- BYC - Bayview Yacht Club ✓
- DYC - Detroit Yacht Club (partial - verify)
- GLYC - Great Lakes Yacht Club (partial - verify)

### Incomplete Entries (To Complete)

1. CSYC - Crescent Sail Yacht Club
2. CYC - Chicago Yacht Club
3. DBC - Detroit Boat Club
4. DSC - Detroit Sail Club
5. EBC - Erie Boat Club
6. FYC - Ford Yacht Club
7. GSC - Grayhaven Sail Club
8. GIYC - Grosse Ile Yacht Club
9. GPC - Grosse Pointe Club
10. GPFBC - Grosse Pointe Farms Boat Club
11. GPSC - Grosse Pointe Sail Club
12. GPYC - Grosse Pointe Yacht Club
13. LSSC - Lake Shore Sail Club
14. NCYC - North Cape Yacht Club
15. NSSC - North Star Sail Club
16. PYC - Pontiac Yacht Club
17. PHYC - Port Huron Yacht Club
18. SYC - Sarina/Sarnia Yacht Club
19. TYC - Toledo Yacht Club
20. WYC - Windsor Yacht Club
21. DCSC - Detroit Community Sailing Club

---

**Document Version**: 1.0
**Created**: 2024
**Status**: Planning Phase - Awaiting Approval
