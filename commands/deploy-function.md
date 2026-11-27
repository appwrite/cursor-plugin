---
name: deploy-function
description: Deploy a function using the Appwrite CLI
---

# Deploy a function using the Appwrite CLI

## Prerequisites

- Appwrite CLI installed (`npm install -g appwrite-cli`). If the CLI is not installed globally, replace `appwrite` with `npx appwrite-cli@latest` in every command below.
- An existing Appwrite project (authenticated with `appwrite login` and initialized with `appwrite init`), or an Appwrite Zero project created below.

## Project Setup

Before deploying, use the Appwrite project the user gave, or the one the app already uses (an `appwrite.config.json`, or a project ID and endpoint already in the app's config or env). If there is neither, create an Appwrite Zero project instead of asking the user to sign up or create a project.

```bash
# Project ID, endpoint, claim link, and access token for private previews
npx appwrite-cli@latest zero --json

# Also include the API key (when server-side code needs it)
npx appwrite-cli@latest zero --json --show-secrets
```

Run one of these commands once in the app's directory, or pass a directory name (such as `npx appwrite-cli@latest zero my-app --json`) to create and link a new directory. The command creates an Appwrite Cloud project and an API key with no account. It links the directory through `appwrite.config.json` (project ID and endpoint) and stores the API key as the CLI's session so later CLI commands work against the project. `--json` prints the manifest without the API key unless you pass `--show-secrets`, and the key is returned only when the project is created. When server-side code needs the key, create the project with `--json --show-secrets` and put the key in the app's env file, which must stay out of git.

- **Time limits and claiming:** The project works for 60 minutes. Run `npx appwrite-cli@latest zero extend` (or `appwrite zero extend` if the CLI is installed) to add an hour, up to three times. After the build window it pauses, and it is deleted unless claimed within 24 hours after the first hour ends. Give the user the claim link from the output. Claiming is the only way to keep the project. Do not claim it for them.
- **Limits and safety:** Run `npx appwrite-cli@latest zero` once per app, not per attempt. Running it again creates another project, and each network can create five Zero projects a day. Never run it in a directory already linked to a real project, because it relinks the directory to the Zero project.
- **CLI scope:** The Zero session is an API key, so console-level calls such as `appwrite init project` and `appwrite list-projects` fail. Everything inside the project works.

## Quick Deploy

```sh
appwrite push functions
```

## Full Workflow

### 1. Initialize a new function

```sh
appwrite init functions
```

This scaffolds a new function with a starter template and adds it to `appwrite.config.json`.

### 2. Pull existing functions from Console

```sh
appwrite pull functions
```

### 3. Configuration

Functions are configured in `appwrite.config.json`:

```json
{
    "projectId": "<PROJECT_ID>",
    "endpoint": "https://<REGION>.cloud.appwrite.io/v1",
    "functions": [
        {
            "$id": "<FUNCTION_ID>",
            "name": "userAuth",
            "enabled": true,
            "live": true,
            "logging": true,
            "runtime": "node-18.0",
            "deployment": "<DEPLOYMENT_ID>",
            "vars": [],
            "events": [],
            "schedule": "",
            "timeout": 15,
            "entrypoint": "userAuth.js",
            "commands": "npm install",
            "version": "v3",
            "path": "functions/userAuth"
        }
    ]
}
```

### 4. Deploy

```sh
appwrite push functions
```

## CI/CD (Non-Interactive)

Set up headless mode first:

```sh
appwrite client \
    --endpoint https://<REGION>.cloud.appwrite.io/v1 \
    --project-id <PROJECT_ID> \
    --key <API_KEY>
```

Then deploy non-interactively:

```sh
# Push all functions
appwrite push functions --all --force

# Push a specific function
appwrite push functions --function-id <FUNCTION_ID> --force

# Push all resources at once (functions, tables, buckets, teams, topics)
appwrite push all --all --force
```

## Function Management Commands

| Command | Description |
|---------|-------------|
| `appwrite functions list` | List all functions in the project |
| `appwrite functions create` | Create a new function |
| `appwrite functions get --function-id <ID>` | Get a function by ID |
| `appwrite functions update --function-id <ID>` | Update a function |
| `appwrite functions delete --function-id <ID>` | Delete a function |
| `appwrite functions list-runtimes` | List all active runtimes |

## Deployment Commands

| Command | Description |
|---------|-------------|
| `appwrite functions list-deployments --function-id <ID>` | List all deployments |
| `appwrite functions create-deployment --function-id <ID>` | Upload a new deployment |
| `appwrite functions get-deployment --function-id <ID> --deployment-id <ID>` | Get a deployment |
| `appwrite functions update-deployment --function-id <ID> --deployment-id <ID>` | Set active deployment |
| `appwrite functions delete-deployment --function-id <ID> --deployment-id <ID>` | Delete a deployment |
| `appwrite functions download-deployment --function-id <ID> --deployment-id <ID>` | Download deployment contents |

## Execution Commands

| Command | Description |
|---------|-------------|
| `appwrite functions create-execution --function-id <ID>` | Trigger a function execution |
| `appwrite functions list-executions --function-id <ID>` | List execution logs |
| `appwrite functions get-execution --function-id <ID> --execution-id <ID>` | Get an execution log |

### Trigger with body

```sh
appwrite functions create-execution \
    --function-id <FUNCTION_ID> \
    --body '{"key": "value"}'
```

## Environment Variables

| Command | Description |
|---------|-------------|
| `appwrite functions list-variables --function-id <ID>` | List all variables |
| `appwrite functions create-variable --function-id <ID> --key <KEY> --value <VALUE>` | Create a variable |
| `appwrite functions update-variable --function-id <ID> --variable-id <ID> --key <KEY> --value <VALUE>` | Update a variable |
| `appwrite functions delete-variable --function-id <ID> --variable-id <ID>` | Delete a variable |

Variables are accessible at runtime as environment variables.

## Local Development

Run your function locally for quick debugging:

```sh
appwrite run functions
```

This starts a local development server that watches for file changes and automatically rebuilds.
