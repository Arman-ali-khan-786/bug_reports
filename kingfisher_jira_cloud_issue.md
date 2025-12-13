### 📋 Checklist

- [✅] I'm running **the latest `main` or a recent release**.
- [✅] I've searched existing issues and **no open issue covers this bug**.
- [✅] If this is a build problem, I attached the **full error log**.

---

### What version?

```
![kingfisher_version](https://github.com/user-attachments/assets/b0dd04cc-44e1-4891-9b21-306d033130fa)
```

### 🐞  What happened?

Jira Cloud scan fails with `missing field 'issues'` error because Atlassian has **removed** the old API endpoint.
Kingfisher calls: `GET /rest/api/2/search?jql=...`
But Jira Cloud now requires: `POST /rest/api/3/search/jql` with JQL in JSON body.
Jira returns HTTP 410 Gone with an error JSON (not the expected `issues` array), causing the decode failure.

**Root cause location:** `vendor/jira_query/src/access.rs`
- Line 26: Uses deprecated `rest/api/2`
- Lines 83-84: Uses removed `search?jql=` endpoint
- Lines 155-162: Uses GET instead of required POST

### ✅  What did you expect to happen?

Kingfisher should fetch issues from Atlassian Jira Cloud and scan them for secrets successfully

### 🔢  Reproduction steps

1. Export Jira API token: `export KF_JIRA_TOKEN="your-atlassian-api-token"`
2. Run: `kingfisher scan jira --url https://your-org.atlassian.net --jql "project = TEST" --max-results 10`
3. Observed the `missing field 'issues error

### 💻  Environment

| Item            | Value |
| --------------- | ----- |
| OS / Distro     | macOS 14.x (Sonoma) arm64 |
| Rust toolchain  | N/A (pre-built binary via Homebrew) |
| kingfisher ver. | 1.69.0 |

### 📎  Log / stack trace

<details>
<summary>Click to expand</summary>

```text
❯ kingfisher scan jira --url https://<org_name>.atlassian.net --jql "project = <PROJECT_KEY> ORDER BY created DESC" --max-results 10

Kingfisher 1.69.0 is up to date
 INFO kingfisher: Launching with 16 concurrent scan jobs. Use --num-jobs to override.
 INFO kingfisher::rule_loader: Loaded 350 rules
Error: Failed to run scan command

Caused by:
    0: Error in accessing the Jira REST API.
    1: error decoding response body
    2: missing field `issues` at line 1 column 55
```

**Jira API response (verified via curl):**
```
HTTP/2 410
{"errorMessages":["The requested API has been removed. Please migrate to the /rest/api/3/search/jql API."]}
```

</details>
