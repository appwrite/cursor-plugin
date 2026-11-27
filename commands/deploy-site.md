---
name: deploy-site
description: Deploy a site using the Appwrite CLI
---

# Deploy a site using the Appwrite CLI

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
appwrite push sites
```

## Full Workflow

### 1. Initialize a new site

```sh
appwrite init sites
```

### 2. Pull existing sites from Console

```sh
appwrite pull sites
```

### 3. Configuration

Sites are configured in `appwrite.config.json`:

```json
{
    "projectId": "<PROJECT_ID>",
    "endpoint": "https://<REGION>.cloud.appwrite.io/v1",
    "sites": [
        {
            "$id": "<SITE_ID>",
            "name": "My Site",
            "enabled": true,
            "logging": true,
            "framework": "astro",
            "timeout": 30,
            "installCommand": "npm install",
            "buildCommand": "npm run build",
            "outputDirectory": "./dist",
            "specification": "s-1vcpu-512mb",
            "buildRuntime": "node-22",
            "adapter": "ssr",
            "fallbackFile": "",
            "path": "sites/my-site"
        }
    ]
}
```

### 4. Deploy

```sh
appwrite push sites
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
# Push all sites
appwrite push sites --all --force

# Push a specific site
appwrite push sites --site-id <SITE_ID> --force
```

## Site Management Commands

| Command | Description |
|---------|-------------|
| `appwrite sites list` | List all sites in the project |
| `appwrite sites create` | Create a new site |
| `appwrite sites get --site-id <ID>` | Get a site by ID |
| `appwrite sites update --site-id <ID>` | Update a site |
| `appwrite sites delete --site-id <ID>` | Delete a site |
| `appwrite sites list-frameworks` | List available frameworks |
| `appwrite sites list-specifications` | List allowed site specifications |

## Deployment Commands

| Command | Description |
|---------|-------------|
| `appwrite sites list-deployments --site-id <ID>` | List all deployments |
| `appwrite sites create-deployment --site-id <ID>` | Create a new deployment |
| `appwrite sites get-deployment --site-id <ID> --deployment-id <ID>` | Get a deployment |
| `appwrite sites delete-deployment --site-id <ID> --deployment-id <ID>` | Delete a deployment |
| `appwrite sites update-site-deployment --site-id <ID> --deployment-id <ID>` | Set active deployment |
| `appwrite sites update-deployment-status --site-id <ID> --deployment-id <ID>` | Cancel an ongoing build |

## Environment Variables

| Command | Description |
|---------|-------------|
| `appwrite sites list-variables --site-id <ID>` | List all variables |
| `appwrite sites create-variable --site-id <ID> --key <KEY> --value <VALUE>` | Create a variable |
| `appwrite sites update-variable --site-id <ID> --variable-id <ID> --key <KEY> --value <VALUE>` | Update a variable |
| `appwrite sites delete-variable --site-id <ID> --variable-id <ID>` | Delete a variable |

Variables are accessible during build and runtime (server-side rendering) as environment variables.

## Logs

| Command | Description |
|---------|-------------|
| `appwrite sites list-logs --site-id <ID>` | List site request logs |
| `appwrite sites get-log --site-id <ID> --log-id <ID>` | Get a specific log |
| `appwrite sites delete-log --site-id <ID> --log-id <ID>` | Delete a log |

## Templates

| Command | Description |
|---------|-------------|
| `appwrite sites list-templates` | List available site templates |
| `appwrite sites get-template --template-id <ID>` | Get template details |
| `appwrite sites create-template-deployment --site-id <ID>` | Deploy from a template |
