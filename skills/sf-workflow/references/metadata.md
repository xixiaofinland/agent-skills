# Metadata Management Reference

## File Naming Conventions

Every metadata file has a companion `-meta.xml` file. Both must exist and stay in sync.

| Metadata Type | File pattern                  | Meta file                              |
| ------------- | ----------------------------- | -------------------------------------- |
| Apex Class    | `MyClass.cls`                 | `MyClass.cls-meta.xml`                 |
| Apex Trigger  | `MyTrigger.trigger`           | `MyTrigger.trigger-meta.xml`           |
| LWC           | `myComponent/` (folder)       | `myComponent/myComponent.js-meta.xml`  |
| Aura          | `MyComponent/` (folder)       | `MyComponent/MyComponent.cmp-meta.xml` |
| Custom Object | `MyObject__c.object-meta.xml` | — (single file)                        |
| Flow          | `MyFlow.flow-meta.xml`        | — (single file)                        |

## Create Metadata

### Apex Class

```bash
sf apex generate class -d force-app/main/default/classes -n MyClass
```

Creates:

- `force-app/main/default/classes/MyClass.cls`
- `force-app/main/default/classes/MyClass.cls-meta.xml`

Default template is a bare class. For a test class, create manually or modify the generated file.

### Apex Trigger

```bash
sf apex generate trigger -d force-app/main/default/triggers -n MyTrigger
```

Creates:

- `force-app/main/default/triggers/MyTrigger.trigger`
- `force-app/main/default/triggers/MyTrigger.trigger-meta.xml`

The generated trigger is empty — add the `on ObjectName (before insert, ...)` header manually.

### Lightning Web Component

```bash
sf lightning generate component -d force-app/main/default/lwc -n myComponent --type lwc
```

Creates the `myComponent/` folder with:

- `myComponent.js` — controller
- `myComponent.html` — template
- `myComponent.css` — styles
- `myComponent.js-meta.xml` — metadata config

LWC names use **camelCase** in the CLI but **kebab-case** in HTML markup: `<c-my-component>`.

### Aura Component

```bash
sf lightning generate component -d force-app/main/default/aura -n MyComponent --type aura
```

Creates the `MyComponent/` folder with Aura-specific files.

## Delete Metadata

Delete from both the org and local filesystem:

```bash
# Delete from org and local
sf project delete source -m ApexClass:MyClass -o <org> --json

# Delete from org only (keep local)
sf project delete source -m ApexClass:MyClass -o <org> --no-source-tracking --json

# Preview what will be deleted
sf project delete source -m ApexClass:MyClass -o <org> --dry-run --json
```

Always review the dry-run output before deleting. This is irreversible in the org.

## Rename Metadata

The sf CLI has no rename command. The workflow is:

1. Create the new file locally with the new name (copy content)
2. Update the class/trigger name inside the file to match
3. Deploy the new file: `sf project deploy start -d <new-file> -o <org>`
4. Delete the old file: `sf project delete source -m ApexClass:OldName -o <org>`

For Apex classes, ensure any references (test classes, other classes) are updated before deleting.

## Listing and Retrieving Org Metadata Not in Local Repo

The local repo typically starts empty. Metadata is pulled selectively from the org as needed for the current work — not all at once.

### Step 1: List available components of a type

```bash
sf org list metadata -m ApexClass -o <alias> --json
```

Response structure:

```json
{
  "result": [
    {
      "fullName": "MyClass",
      "type": "ApexClass",
      "manageableState": "unmanaged"
    },
    {
      "fullName": "ManagedClass",
      "type": "ApexClass",
      "manageableState": "installed"
    }
  ]
}
```

**Filter out `"manageableState": "installed"`** — these are managed package components and cannot be edited locally.

Present only the `fullName` values with `manageableState` of `"unmanaged"` or `"deprecatedEditable"` to the user.

### Step 2: Retrieve the chosen component

```bash
sf project retrieve start -m ApexClass:ChosenName -o <alias> --json
```

The file lands in `force-app/main/default/classes/` (or the appropriate subdirectory for the type).

### Browsing all metadata types first

If the user doesn't know which type to look in:

```bash
sf org list metadata-types -o <alias> --json
```

Response includes a `result.metadataObjects` array — each entry has `xmlName` (the type name to use with `-m`) and `directoryName` (where files land locally). Present `xmlName` values for the user to pick from, then proceed with Step 1.

### Retrieve multiple components

```bash
sf project retrieve start -m "ApexClass:MyClass,ApexTrigger:MyTrigger" -o <alias> --json
```

Never use wildcard retrieval (`ApexClass:*`) unless the user explicitly requests it — see Safety Rules in SKILL.md.

## meta.xml API Versions

Each `-meta.xml` file contains an `<apiVersion>`. Keep this consistent with your project's target API version (set in `sfdx-project.json` under `"sourceApiVersion"`).

Example class meta:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ApexClass xmlns="http://soap.sforce.com/2006/04/metadata">
    <apiVersion>59.0</apiVersion>
    <status>Active</status>
</ApexClass>
```
