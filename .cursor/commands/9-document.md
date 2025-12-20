# DOCUMENT - Document the solution

## Phase: 9: DOCUMENT

**Purpose**: Document the final solution, including usage, API changes, and any important notes.

## Instructions

DOCUMENT the implemented and successfully tested solution.

### Documentation Requirements

1. **Include updates to all affected README.md files**

2. **Ensure new or updated APIs are covered** in relevant OpenAPI/Swagger schemas where applicable

3. **Document the solution**:

   - Summary of what was implemented
   - API changes (if applicable) - endpoints, request/response formats
   - Database changes (if applicable) - schema, migrations
   - Component changes - new components, modified components
   - Configuration changes - env variables, config files
   - Usage instructions - how to use the new feature
   - Breaking changes (if any)

4. **Update related documentation**:

   - API documentation (Swagger/spec.yaml if applicable)
   - README.md files in affected repos
   - Developer documentation if needed

5. **Document important notes**:
   - Known limitations
   - Future improvements
   - Dependencies
   - Migration notes (if applicable)

### Document the Documentation

- **If using Issue Template**: Update the working file in `.issue/.github/ISSUE_TEMPLATE/`
- **If using AIN document**: Navigate to the M: DOCUMENT section and update it
- **Update Status in Metadata** to reflect completion

## Context

- **API Docs**: `server/api/spec.yaml` - Update OpenAPI/Swagger schemas where applicable
- **README Files**: Update all affected `README.md` files
- **Developer Docs**: `.github/docs/DEV/`
- **AI Rules**: `.github/docs/AI/AI-RULES.md`
- **Issue Template**: `.issue/.github/ISSUE_TEMPLATE/` - Update the working file
- **AIN Workflow Template**: `.github/docs/AI/AIN [20YY-MM-DD] WORKFLOW_TEMPLATE.md` (alternative workflow)

## Important

- All workflow phases go in **ONE document** - do NOT create separate files
- Documentation should be clear and complete
- Include examples if helpful
- Document any breaking changes clearly

## Documentation Checklist

- [ ] Solution summary written
- [ ] API changes documented (if applicable)
- [ ] Database changes documented (if applicable)
- [ ] Usage instructions provided
- [ ] Breaking changes noted (if any)
- [ ] Related docs updated
- [ ] Status updated in Metadata
