# Org Management Reference

## Listing Orgs

```bash
# List all authenticated orgs (fast, no network check)
sf org list --skip-connection-status --json

# List with live connection status (slower)
sf org list --json
```

JSON output structure:

```json
{
  "result": {
    "nonScratchOrgs": [
      {
        "alias": "my-sandbox",
        "username": "user@company.com.sandbox",
        "instanceUrl": "https://company--sandbox.sandbox.my.salesforce.com",
        "isDefaultUsername": true
      }
    ],
    "scratchOrgs": [
      {
        "alias": "dev-scratch",
        "username": "test-xyz@example.com",
        "expirationDate": "2024-03-25",
        "isDefaultUsername": false
      }
    ]
  }
}
```

Prefer **alias** over username in all commands — aliases are shorter and don't change.

## Setting Target Org

```bash
# Set for current project only (stored in .sf/config.json)
sf config set target-org my-sandbox

# Set globally across all projects
sf config set target-org my-sandbox --global

# Check current target org
sf config get target-org

# Remove target org config
sf config unset target-org
```

Project-level config takes precedence over global config.

## Opening Orgs

```bash
# Open org home page
sf org open -o my-sandbox

# Open a specific metadata record in Setup
sf org open -f force-app/main/default/classes/MyClass.cls -o my-sandbox

# Open Setup directly
sf org open --path /lightning/setup/ApexClasses/home -o my-sandbox
```

## Scratch Orgs vs Sandboxes

|          | Scratch Org              | Sandbox                             |
| -------- | ------------------------ | ----------------------------------- |
| Source   | Created from config file | Cloned from production              |
| Lifetime | 1–30 days (expires)      | Persistent                          |
| Data     | Empty by default         | Production data copy (full/partial) |
| Use case | Feature development, CI  | UAT, integration testing            |
| Auth     | `sf org create scratch`  | `sf org login web`                  |

### Scratch Org Lifecycle

```bash
# Create (requires Dev Hub configured)
sf org create scratch -f config/project-scratch-def.json -a my-scratch -d 30 -v DevHub

# Check scratch org info
sf org display -o my-scratch --json

# Delete when done
sf org delete scratch -o my-scratch --json
```

### Sandbox Authentication

```bash
# Authenticate via browser
sf org login web -a my-sandbox -r https://test.salesforce.com

# Authenticate production
sf org login web -a production -r https://login.salesforce.com
```

## Org Display

Get full details about an authenticated org:

```bash
sf org display -o my-sandbox --json
```

Output includes: instance URL, access token, org ID, username, alias, expiration date (scratch orgs).

## Dev Hub

Dev Hub is the production org used to create scratch orgs. Authenticate it separately:

```bash
sf org login web -a DevHub -r https://login.salesforce.com
sf config set target-dev-hub DevHub
```

## Org Config Files

**`.sf/config.json`** (project-level, git-ignored):

```json
{
  "target-org": "my-sandbox"
}
```

**`~/.sf/config.json`** (global):

```json
{
  "target-org": "default-sandbox",
  "target-dev-hub": "DevHub"
}
```

**`config/project-scratch-def.json`** (scratch org definition, committed to git):

```json
{
  "orgName": "My Dev Org",
  "edition": "Developer",
  "features": ["EnableSetPasswordInApi"],
  "settings": {
    "lightningExperienceSettings": {
      "enableS1DesktopEnabled": true
    }
  }
}
```
