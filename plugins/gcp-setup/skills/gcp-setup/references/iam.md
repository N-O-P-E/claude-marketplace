# GCP IAM & Permissions

## Info to Collect
- **Members**: Email addresses of users or service accounts to grant access
- **Roles**: Which predefined or custom roles to assign
- **Conditions**: Optional IAM conditions (time-based, resource-based)
- **Scope**: Project-level, folder-level, or organization-level

## Console Steps

### Adding an IAM Binding
1. Navigate to `https://console.cloud.google.com/iam-admin/iam?project=PROJECT_ID`
2. wait_for "IAM" heading and the permissions table to load
3. Click "Grant Access" button (usually top of page)
4. take_snapshot to find the form
5. fill the "New principals" field with the member email
6. Click "Select a role" dropdown
7. Search for the desired role
8. click the role
9. If multiple roles: click "Add another role" and repeat
10. Click "Save"
11. take_screenshot to verify the member appears in the IAM table

### Modifying an Existing Binding
1. Find the member in the IAM table
2. Click the edit (pencil) icon next to their name
3. Modify roles as needed
4. Click "Save"

### Removing Access
1. Find the member in the IAM table
2. Click the delete (trash) icon
3. Confirm removal

## Common Role Categories

### Basic Roles (use sparingly)
- `roles/viewer` — Read-only access to all resources
- `roles/editor` — Read/write access to most resources
- `roles/owner` — Full access including IAM management

### Recommended: Use Predefined Roles Instead
Always prefer specific predefined roles over basic roles. For example, instead of `roles/editor`, use `roles/run.admin` + `roles/storage.admin` for exactly the access needed.

## Common Issues
- **Policy conflicts**: Org policies may restrict who can be granted certain roles
- **Propagation delay**: IAM changes can take up to 60 seconds to take effect
- **Inherited roles**: Roles granted at org/folder level are inherited by all projects underneath
