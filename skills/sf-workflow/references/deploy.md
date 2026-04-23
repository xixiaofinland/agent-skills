# Deploy & Retrieve Reference

## Three Deploy Modes

### 1. Single File Deploy (most common)

Deploy one metadata file after editing it:

```bash
sf project deploy start -d force-app/main/default/classes/MyClass.cls -o <org>
sf project deploy start -d force-app/main/default/lwc/myComponent -o <org>
```

The `-d` flag accepts a file path or directory. For LWC/Aura, pass the component folder.

Pass only one file per component — the sf CLI resolves all associated files automatically:

- **Apex**: pass the `.cls` or `.trigger` file; the sf CLI includes the `-meta.xml` companion automatically
- **LWC**: pass any one file inside the component folder (`.js`, `.html`, `.css`, or `-meta.xml`); the sf CLI deploys/retrieves the entire component (all files in the folder)

### 2. Tracked Changes Deploy

Deploy all locally modified files at once (requires source tracking):

```bash
sf project deploy start -o <org>
```

Source tracking records which files have changed since the last sync. Run `sf project deploy preview` first to see what will be deployed without actually deploying.

### 3. Manifest Deploy/Retrieve

Use a `package.xml` manifest to specify exactly what to retrieve:

```bash
sf project retrieve start -x manifest/package.xml -o <org>
sf project deploy start -x manifest/package.xml -o <org>
```

## Retrieve Patterns

Retrieve mirrors deploy syntax:

```bash
# Single file
sf project retrieve start -d force-app/main/default/classes/MyClass.cls -o <org>

# By metadata spec
sf project retrieve start -m ApexClass:MyClass -o <org>
sf project retrieve start -m "ApexClass:MyClass,ApexClass:OtherClass" -o <org>

# All tracked remote changes
sf project retrieve start -o <org>

# By manifest
sf project retrieve start -x manifest/package.xml -o <org>
```

## Source Tracking

Source tracking keeps a local record of what's in sync with the org. It works automatically when you deploy or retrieve.

Check tracking status:

```bash
sf project deploy preview -o <org>    # shows local changes not yet deployed
sf project retrieve preview -o <org>  # shows org changes not yet retrieved
```

If tracking is out of sync:

```bash
# Force re-sync tracking (destructive — use carefully)
sf project reset tracking -o <org> --json
```

## Conflict Handling

When both local and org have changes to the same file, deploy/retrieve will fail with a conflict error. Resolve by:

1. Retrieve the org version to inspect: `sf project retrieve start -m ApexClass:MyClass -o <org>`
2. Compare with local version
3. Decide which version to keep, then deploy or retrieve explicitly

## package.xml Format

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Package xmlns="http://soap.sforce.com/2006/04/metadata">
    <types>
        <members>MyClass</members>
        <members>OtherClass</members>
        <name>ApexClass</name>
    </types>
    <types>
        <members>MyTrigger</members>
        <name>ApexTrigger</name>
    </types>
    <version>59.0</version>
</Package>
```

Use `*` as a member to retrieve all of a type:

```xml
<members>*</members>
<name>ApexClass</name>
```

## Common Flags

| Flag                 | Purpose                                  |
| -------------------- | ---------------------------------------- |
| `-d <path>`          | Source file or directory                 |
| `-m <type:name>`     | Metadata component spec                  |
| `-x <file>`          | Manifest (package.xml)                   |
| `-o <org>`           | Target org alias or username             |
| `--dry-run`          | Preview without executing                |
| `--ignore-conflicts` | Deploy despite conflicts (use carefully) |
| `--ignore-errors`    | Continue on partial failures             |

## Common Errors & Recovery

| Error pattern                             | Cause                                    | Recovery                                                              |
| ----------------------------------------- | ---------------------------------------- | --------------------------------------------------------------------- |
| `INVALID_SESSION_ID` or `Session expired` | Auth token expired                       | Re-authenticate: `sf org login web -a <alias>`                        |
| `CONFLICT` in deploy/retrieve response    | Both local and org changed the same file | Retrieve org version to temp dir, diff, let user decide               |
| Compile error (`Line X, Column Y`)        | Syntax or type error in Apex             | Fix the reported line, redeploy the single file                       |
| `UNKNOWN_EXCEPTION`                       | Transient server error                   | Safe to retry the same command once                                   |
| `Test run timed out`                      | Tests exceeded `-w` value                | Re-run with a higher `-w` (e.g., `-w 300`)                            |
| `Entity is locked by another deployment`  | Another deploy is in progress            | Wait and retry — do not cancel the other deploy without user approval |

## Idempotency & Retries

- **Deploy and retrieve** are safe to retry — redeploying the same file overwrites with the same content.
- **Delete is NOT idempotent** — deleting an already-deleted component fails. Always check existence first with `sf org list metadata -m Type:Name -o <alias> --json` before retrying a delete.
- After any transient failure, retry once. If it fails again, report the full error to the user rather than looping.
