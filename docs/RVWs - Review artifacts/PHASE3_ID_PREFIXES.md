# Phase 3: ID Generation Prefixes

## Entity Prefix Mapping

Each entity will use a 4-character prefix for `utility.unique_id()` generation:

| Entity        | Prefix | Example ID Format                         |
| ------------- | ------ | ----------------------------------------- |
| account       | `acct` | `acct_l8k2j3m4n5o6p7q8r9s0t1u2v3w4x5y6z7` |
| user          | `user` | `user_l8k2j3m4n5o6p7q8r9s0t1u2v3w4x5y6z7` |
| event         | `evnt` | `evnt_l8k2j3m4n5o6p7q8r9s0t1u2v3w4x5y6z7` |
| feedback      | `fdbk` | `fdbk_l8k2j3m4n5o6p7q8r9s0t1u2v3w4x5y6z7` |
| invite        | `invt` | `invt_l8k2j3m4n5o6p7q8r9s0t1u2v3w4x5y6z7` |
| key (API key) | `apik` | `apik_l8k2j3m4n5o6p7q8r9s0t1u2v3w4x5y6z7` |
| log           | `log_` | `log__l8k2j3m4n5o6p7q8r9s0t1u2v3w4x5y6z7` |
| login         | `logn` | `logn_l8k2j3m4n5o6p7q8r9s0t1u2v3w4x5y6z7` |
| notification  | `notf` | `notf_l8k2j3m4n5o6p7q8r9s0t1u2v3w4x5y6z7` |
| pushtoken     | `psht` | `psht_l8k2j3m4n5o6p7q8r9s0t1u2v3w4x5y6z7` |
| token         | `tokn` | `tokn_l8k2j3m4n5o6p7q8r9s0t1u2v3w4x5y6z7` |
| usage         | `used` | `used_l8k2j3m4n5o6p7q8r9s0t1u2v3w4x5y6z7` |
| email         | `mail` | `mail_l8k2j3m4n5o6p7q8r9s0t1u2v3w4x5y6z7` |

## Non-Database Entity Prefixes

Non-database entities also use 4-character prefixes for consistency:

| Entity        | Prefix | Example ID Format                         | Storage      |
| ------------- | ------ | ----------------------------------------- | ------------ |
| job (queue)   | `jobs` | `jobs_l8k2j3m4n5o6p7q8r9s0t1u2v3w4x5y6z7` | Redis (Bull) |
| file (upload) | `file` | `file_l8k2j3m4n5o6p7q8r9s0t1u2v3w4x5y6z7` | S3/File Sys  |

## ID Format

The `utility.unique_id()` function generates IDs in the format:

```
<prefix>_<base36-timestamp><random-16-hex>
```

Example: `user_l8k2j3m4n5o6p7q8r9s0t1u2v3w4x5y6z7`

Where:

- `prefix` = 4-character entity identifier (e.g., `user`, `acct`, `apik`)
- `base36-timestamp` = Current timestamp in base36 encoding
- `random-16-hex` = 16 random hexadecimal characters

## Benefits

1. **Human-readable**: Prefix makes it easy to identify entity type from ID
2. **Sortable**: Timestamp component provides chronological ordering
3. **Unique**: Random component ensures uniqueness even at same millisecond
4. **No conflicts**: Avoids conflicts with MongoDB's `_id` field
5. **Consistent**: All entities use same generation pattern
