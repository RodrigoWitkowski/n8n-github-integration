# n8n + GitHub API learning exercise

A small educational project I made for experimenting with **n8n**, the **GitHub API**, scheduled workflows, Base64 file content, and automated commits.

The workflow:

1. Starts manually or from an n8n schedule.
2. Reads a Markdown file from a GitHub repository.
3. Decodes the Base64 file content returned by GitHub.
4. Appends the workflow execution timestamp.
5. Updates the file and creates a new commit.

> This repository is intended as an automation/API learning exercise, not as a production system.

## Workflow preview

<!--
Add your n8n workflow screenshot to:

docs/workflow-screenshot.png

Then remove the surrounding HTML comment from the image below.
-->

<!-- ![n8n workflow](docs/workflow-screenshot.png) -->

<img width="888" height="534" alt="image" src="https://github.com/user-attachments/assets/d7d839d4-0b32-44b5-8e2c-a45876891bd9" />

## Workflow structure

```text
Manual Trigger ───────┐
                      ├──> Get GitHub file
Schedule Trigger ─────┘          │
                                 ▼
                         Decode file content
                                 │
                                 ▼
                          Append timestamp
                                 │
                                 ▼
                         Update GitHub file
```

## Repository files

```text
.
├── README.md
├── n8n-github-workflow-sanitized.json
├── Commit Timestamps.md
└── docs/
    └── workflow-screenshot.png
```

Only `README.md` and the sanitized workflow export are required. Create the other files as described below.

## Requirements

- An n8n instance, either self-hosted or n8n Cloud.
- A GitHub account.
- A GitHub repository where you have permission to update files.
- A GitHub personal access token with access to the target repository.

## Setup

### 1. Create the target Markdown file

In the repository that the workflow will update, create:

```text
Commit Timestamps.md
```

Example initial content:

```md
# Execution timestamps
```

The filename can be changed, but the same path must be configured in both GitHub nodes.

### 2. Create a GitHub access token

Create a fine-grained personal access token in GitHub.

Recommended configuration:

```text
Resource owner: your GitHub account or organization
Repository access: only the repository used by this project
Repository permission:
  Contents: Read and write
```

Use the minimum repository access required for the experiment.

Do not place the token directly inside the workflow JSON, source code, README, screenshots, or Git history.

### 3. Import the workflow into n8n

1. Open n8n.
2. Create a new workflow.
3. Choose **Import from File**.
4. Select:

```text
n8n-github-workflow-sanitized.json
```

The public export contains placeholders instead of account-specific configuration.

### 4. Add the GitHub credential in n8n

Create a GitHub API credential in n8n using your personal access token.

Then open both GitHub nodes and select that credential:

```text
Get a file
Edit a file
```

Credentials are stored by n8n and should not be committed to this repository.

### 5. Configure the repository

In both GitHub nodes, replace:

```text
YOUR_GITHUB_USERNAME
YOUR_REPOSITORY
```

with the target repository owner and repository name.

Example:

```text
Owner: octocat
Repository: automation-playground
File path: Commit Timestamps.md
```

Both nodes must point to the same file.

### 6. Configure the schedule

Open the **Schedule Trigger** node and choose when the automation should run.

For example, to run every day at 10:00 using n8n's six-field cron format:

```cron
0 0 10 * * *
```

The fields are:

```text
second minute hour day-of-month month day-of-week
```

Also confirm the workflow timezone in n8n. For São Paulo:

```text
America/Sao_Paulo
```

The sanitized export intentionally does not contain a working account-specific schedule, so configure the trigger in the n8n editor before activating the workflow.

### 7. Test manually

Before activating the schedule:

1. Open the workflow.
2. Click **Execute workflow**.
3. Confirm that the **Get a file** node reads the Markdown file.
4. Confirm that the content is decoded.
5. Confirm that the **Edit a file** node creates a commit.
6. Open the Markdown file on GitHub and verify that a timestamp was appended.

Expected result:

```md
# Execution timestamps

Execution timestamp:2026-07-07 10:00:00
```

### 8. Activate the workflow

After the manual test succeeds, save and activate the workflow.

The Schedule Trigger only runs automatically while the workflow is active.

## How the update works

GitHub returns file content encoded in Base64. The workflow decodes it with:

```javascript
{{ $json.content.base64Decode() }}
```

It then removes trailing blank lines and appends a timestamp:

```javascript
{{ 
  $json.decodedContent.replace(/\n+$/, '') +
  "\n\n" +
  $json.string +
  $now.toFormat('yyyy-MM-dd HH:mm:ss')
}}
```

The GitHub node sends the updated content back to the repository and creates a commit with:

```text
I studied today!
```

You can change the appended text and commit message in the **Edit Fields** and **Edit a file** nodes.

## Troubleshooting

### `Resource not accessible by personal access token`

Check that:

- The token has access to the selected repository.
- The repository owner is correct.
- The token has **Contents: Read and write** permission.
- The token is approved by the organization, when organization approval is required.
- Both GitHub nodes use the correct n8n credential.

### File not found

Confirm that:

- The file already exists in the repository.
- The path and capitalization match exactly.
- Both nodes use the same repository and file path.
- The selected branch contains the file.

### Schedule does not run

Confirm that:

- The workflow is saved and active.
- The cron expression is valid.
- The workflow timezone is correct.
- The Schedule Trigger is connected to the GitHub nodes.

## Security

Before publishing an n8n workflow export, remove or replace:

- Repository owner names when privacy is desired.
- Repository URLs.
- Credential IDs and credential names.
- Webhook IDs.
- n8n instance IDs.
- Tokens, secrets, headers, and environment values.
- Pinned execution data containing private API responses.

A token should be stored only in n8n's credential manager or another appropriate secret store.

## References

- [n8n Schedule Trigger](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.scheduletrigger/)
- [GitHub personal access tokens](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)
- [GitHub repository contents API](https://docs.github.com/en/rest/repos/contents)
