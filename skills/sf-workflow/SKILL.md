---
name: salesforce-development
description: >
  Use when the project contains sfdx-project.json, .forceignore, or a force-app/ directory,
  or when working with .cls, .trigger, or Lightning Web Component files.
  Use when the user asks to "deploy to org", "push to org", "retrieve from org", "pull from org",
  "run apex tests", "run tests", "create apex class", "create lwc", "create trigger",
  "create aura component", "delete metadata", "open org", "set target org", "list orgs",
  "diff against org", "compare with org", "check what changed in org",
  "pull metadata from org", "browse org metadata", "fix this apex class",
  "why is my test failing", or any task involving Salesforce, Apex, SFDX, LWC,
  Lightning Web Components, metadata deployment, or the sf CLI.
---

# Salesforce Development

This skill guides Salesforce development tasks using the modern **sf CLI** (Salesforce CLI v2+). All commands use `sf`, never the legacy `sfdx`.

**Always pass `--json` to sf commands.** JSON output is structured, unambiguous, and far easier to parse than human-formatted output. Exceptions: `sf apex run test` with `-r human` when showing results directly to the user, and `sf generate` commands (which produce local files, not JSON).

## Key Concepts

**The local repo is intentionally sparse.** A Salesforce project does not contain all the metadata that exists in the org — only the components relevant to the current work are pulled down. This is normal. Do not treat an empty or partial `force-app/` directory as an error. When the user needs to work on something not yet local, use the "Retrieve Org Metadata Not in Local Repo" workflow below.

**Always verify command results.** After every deploy, retrieve, or test command, parse the `--json` output. If the `status` field is not `Succeeded` (or the command returns a non-zero exit code), report the error to the user before proceeding. See `references/deploy.md` for common errors and recovery steps.

## Safety Rules

These rules must never be violated without explicit user instruction:

**Deploy/Retrieve:**

- Always target a specific file (`-d <file>`) or a named component (`-m Type:Name`) — this is the default for all deploy and retrieve operations
- Never use a `package.xml` manifest (`-x`) unless the user explicitly asks for it
- Never use wildcard component specs (`-m "ApexClass:*"`) unless the user explicitly asks for it
- Never run `sf project deploy start` or `sf project retrieve start` without `-d` or `-m` unless the user explicitly says "deploy all tracked changes" or "retrieve all tracked changes"

**Temp Directories:**

- Always create temp directories **inside the project root** (e.g. `./temp/sf-diff-XXXXXX`) — never at the system root (`/tmp/`)
- The sf CLI requires metadata operations to run within a directory tree containing `sfdx-project.json`; using a path outside the project (e.g. `/tmp/`) will cause the CLI to fail
- Add `temp/` to `.gitignore` if it is not already there

**Apex Tests:**

- Never run `-l RunAllTestsInOrg` or `-l RunLocalTests` without explicit user instruction
- Default to running only the tests the user mentions: a specific method (`-t`) or a specific class (`-n`)
- If the user says "run all tests", confirm which scope they mean before proceeding

## Using the sf CLI Help System

When unsure about a command, its flags, or its behaviour — use the sf CLI's built-in help. Every command and subcommand has its own help page with descriptions, flags, and working examples.

```bash
sf --help                          # top-level commands
sf project --help                  # subcommands under "project"
sf project deploy --help           # subcommands under "project deploy"
sf project deploy start --help     # full flag reference with examples
```

Drill down progressively until you find the right command. The examples in the help output are authoritative — prefer them over guessing.

## Project Detection

Before any operation, verify the project root contains `sfdx-project.json` or `.forceignore`. If neither exists, the current directory is not a Salesforce project — inform the user.

The standard package directory is `force-app/main/default/`. Read `sfdx-project.json` to confirm:

```json
{
  "packageDirectories": [{ "path": "force-app", "default": true }]
}
```

## Pre-Implementation Formatter Check

Before starting any implementation, verify the Apex formatter toolchain is working.

**1. Confirm `.prettierrc` exists at the project root:**

```bash
ls .prettierrc
```

If it is missing, stop and inform the user — prettier will silently fall back to defaults and Apex formatting cannot be trusted.

**2. Confirm prettier and the apex plugin load correctly** with a fast, non-destructive stdin smoke test:

```bash
echo "public class _Check {}" | npx prettier --stdin-filepath _Check.cls --plugin=prettier-plugin-apex
```

- If it exits 0 and prints formatted output, the toolchain is confirmed. Proceed.
- If it fails, stop and report the error to the user before writing any code. Common causes: `node_modules` not installed (`npm install`), `.prettierrc` missing from the project root, or `prettier-plugin-apex` not declared in `package.json`.

## Target Org

**Before any deploy, retrieve, or test operation, resolve the target org and always pass it explicitly as `-o <alias>` in every command.** This makes every command unambiguous and human-readable — never rely on implicit org resolution.

Check the current default:

```bash
sf config get target-org --json
```

If a target org is set, use its value as the `-o` argument in all commands.

If no target org is set, the JSON output will show an empty or missing value. In that case:

1. List the authenticated orgs so the user can choose:
   ```bash
   sf org list --skip-connection-status --json
   ```
2. Ask the user which org to use, then set it:
   ```bash
   sf config set target-org <alias> --json
   ```
   Omit `--global` to scope it to the current project only.

Do not proceed with operations until a target org is confirmed.

If the user explicitly asks to run against a **different** org (not the default), use the specified org alias as `-o` for that command only — do not change the configured default.

## Deploy & Retrieve

Three deploy modes — choose based on scope:

| Mode            | Command                                                      | When to use                                          |
| --------------- | ------------------------------------------------------------ | ---------------------------------------------------- |
| Single file     | `sf project deploy start -d <file> -o <alias> --json`        | Default — after editing a specific file              |
| Named component | `sf project retrieve start -m Type:Name -o <alias> --json`   | Retrieve a specific component by metadata spec       |
| Tracked changes | `sf project deploy start -o <alias> --json`                  | Only when user explicitly asks to deploy all changes |
| Manifest (`-x`) | `sf project retrieve start -x package.xml -o <alias> --json` | Only when user explicitly asks to use package.xml    |

Single file is the most common. Pass the **absolute path** or path relative to project root.

When deploying or retrieving metadata, pass only one file — the sf CLI resolves all associated files automatically:

- **Apex**: pass the `.cls` or `.trigger` file — the sf CLI includes the `-meta.xml` automatically
- **LWC**: pass any one file inside the component folder (`.js`, `.html`, `.css`, or `-meta.xml`) — the sf CLI deploys/retrieves the entire component

For retrieve: same pattern with `retrieve` instead of `deploy`. Always substitute `<alias>` with the resolved default org value.

Before performing a deploy or retrieve for the first time in a session, read `references/deploy.md` for source tracking concepts, conflict resolution, error recovery, and manifest format.

## Apex Tests

| Goal                   | Command                                                               |
| ---------------------- | --------------------------------------------------------------------- |
| Run one test method    | `sf apex run test -t ClassName.MethodName -o <alias> -w 180 -r human` |
| Run all tests in class | `sf apex run test -n ClassName -o <alias> -w 180 -r human`            |
| Run with code coverage | `sf apex run test -n ClassName -o <alias> -w 180 -r json -c`          |
| Fetch coverage report  | `sf apex get test -i <runId> -o <alias> -c --json`                    |

Always use `-w 180` (3-minute wait). Use `-r human` when showing results to the user; use `-r json` when parsing results programmatically (e.g., to extract the run ID for a coverage fetch).

See `references/testing.md` for coverage workflows and reading test results.

## Metadata Type Map

| Directory          | Salesforce Metadata Type   |
| ------------------ | -------------------------- |
| `classes/`         | `ApexClass`                |
| `triggers/`        | `ApexTrigger`              |
| `lwc/`             | `LightningComponentBundle` |
| `aura/`            | `AuraDefinitionBundle`     |
| `objects/`         | `CustomObject`             |
| `flows/`           | `Flow`                     |
| `layouts/`         | `Layout`                   |
| `permissionsets/`  | `PermissionSet`            |
| `profiles/`        | `Profile`                  |
| `staticresources/` | `StaticResource`           |
| `pages/`           | `ApexPage`                 |

Metadata spec format for CLI flags: `MetadataType:ApiName` — e.g., `ApexClass:MyClass`.

## Create Metadata

```bash
# Apex class (creates .cls + .cls-meta.xml)
sf apex generate class -d force-app/main/default/classes -n ClassName

# Apex trigger
sf apex generate trigger -d force-app/main/default/triggers -n TriggerName

# Lightning Web Component
sf lightning generate component -d force-app/main/default/lwc -n ComponentName --type lwc

# Aura component
sf lightning generate component -d force-app/main/default/aura -n ComponentName --type aura
```

See `references/metadata.md` for delete, rename, and file naming conventions.

## Diff Local File Against Org

Use this to compare a local metadata file against the version currently in the org — useful before deploying or after suspecting someone changed the org directly.

**Full workflow:**

1. Derive the metadata type and component name from the file path (e.g. `force-app/main/default/classes/MyClass.cls` → `ApexClass:MyClass`)
2. Create an isolated temp directory **inside the project root**:
   ```bash
   mkdir -p ./temp && mktemp -d ./temp/sf-diff-XXXXXX
   ```
   The temp dir must live inside the project because the sf CLI requires a `sfdx-project.json` ancestor to resolve metadata operations. Using `/tmp/` or any path outside the project will fail.
3. Retrieve the org version into the temp dir (does NOT overwrite the local file):
   ```bash
   sf project retrieve start -m ApexClass:MyClass -r <tmpdir> -o <alias> --json
   ```
4. Diff and show the output:
   ```bash
   diff force-app/main/default/classes/MyClass.cls <tmpdir>/force-app/main/default/classes/MyClass.cls
   ```
5. Ask the user what to do: keep local version (deploy), take org version (retrieve into project), or handle manually
6. Clean up: `rm -rf <tmpdir>`

The `-r` flag is critical — it redirects the retrieved files to a temp directory so the local working copy is never touched during the comparison.

Ensure `temp/` is in `.gitignore` so these ephemeral directories are never committed.

To diff against a **different org**, substitute its alias for `-o <alias>` — do not change the configured default.

## Retrieve Org Metadata Not in Local Repo

Use this when the user wants to pull down metadata that exists in the org but has no local file yet.

**Two-step flow — list then retrieve:**

Step 1: List all components of a type from the org:

```bash
sf org list metadata -m ApexClass -o <alias> --json
```

Parse the JSON response — each entry has a `fullName` field (the component name) and `manageableState`. Filter out `"manageableState": "installed"` (managed package components — these cannot be edited).

Step 2: Once the user selects a name, retrieve it:

```bash
sf project retrieve start -m ApexClass:ChosenName -o <alias> --json
```

**If the user doesn't know which metadata type to browse**, first list all available types in the org:

```bash
sf org list metadata-types -o <alias> --json
```

Parse the result, present the type names, let the user pick a type, then proceed with Step 1 above using that type.

See `references/metadata.md` for the full JSON response structure and filtering rules.

## Org Management

```bash
# Check current default org
sf config get target-org --json

# List all authenticated orgs
sf org list --skip-connection-status --json

# Set default org (project-scoped)
sf config set target-org <alias> --json

# Open org in browser
sf org open -o <alias> --json

# Open a specific metadata file in org UI
sf org open -f force-app/main/default/classes/MyClass.cls -o <alias> --json
```

See `references/org.md` for scratch org vs sandbox differences and org config details.
