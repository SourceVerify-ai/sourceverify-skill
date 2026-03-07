---
name: sourceverify
description: Verify academic and professional citations using the SourceVerify API. Use when the user wants to check if references or citations are real, accurate, or properly formatted. Triggers on keywords like "verify references", "check citations", "citation check", "reference verification", "sourceverify".
argument-hint: [references or "balance" or "history"]
user-invocable: true
disable-model-invocation: false
allowed-tools: Bash, Read
---

# SourceVerify — Citation Verification Skill

Verify academic and professional references against real-world sources using the SourceVerify API.

## Setup

The user must set their API key as an environment variable:

```bash
export SOURCEVERIFY_API_KEY="sk_..."
```

If `SOURCEVERIFY_API_KEY` is not set, tell the user to get one at https://sourceverify.ai/en/developers and set it in their shell profile.

## Commands

Based on `$ARGUMENTS`, determine which action to take:

### 1. Verify References (default)

If arguments contain reference text, a file path, or the user asks to verify/check citations:

**Step 1 — Extract references**

If the user provided a file path, read the file and extract the references from it.
If the user pasted text, parse individual references from it (split by numbered list, newlines between entries, or other clear delimiters).

Limit to 10 references per request. If more than 10, batch them into multiple requests.

**Step 2 — Submit to API**

```bash
curl -s -X POST https://sourceverify.ai/api/verify-reference \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $SOURCEVERIFY_API_KEY" \
  -d '{"references": ["ref1", "ref2", ...]}'
```

Parse the JSON response to get `jobIds` and `documentId`.

If the response has an error (401, 402, 403, 429), report it clearly:
- 401/403: Invalid API key
- 402: Insufficient tokens — direct user to https://sourceverify.ai/en/pricing
- 429: Rate limited — wait and retry

**Step 3 — Poll for results**

Poll every 5 seconds until all jobs are completed or failed:

```bash
curl -s -X POST https://sourceverify.ai/api/verify-reference/results \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $SOURCEVERIFY_API_KEY" \
  -d '{"jobIds": ["id1", "id2", ...]}'
```

While polling, show a brief status update (e.g., "3/5 references verified, polling...").

**Step 4 — Display results**

For each completed job, display a formatted report:

```
## Reference 1: [STATUS]

**Original:** [raw reference text]
**Corrected (APA7):** [corrected reference from result]

**Field Results:**
- Title: MATCH
- Authors: MATCH_WITH_TYPO
- Year: MATCH
- ...

**Explanation:**
[message from the API — already formatted as markdown]
```

Status should be displayed with clear indicators:
- `verified` — VERIFIED
- `verified-with-error` — VERIFIED (with minor errors)
- `unverified` — UNVERIFIED
- `needs-human` — NEEDS HUMAN REVIEW
- `failed` — ERROR

At the end, show a summary:
```
## Summary
- Total: X references
- Verified: X
- Unverified: X
- Needs Review: X
- Tokens remaining: [from balance check]
```

### 2. Check Balance

If `$ARGUMENTS` is "balance" or the user asks about tokens/balance:

```bash
curl -s -X GET https://sourceverify.ai/api/verify-reference/balance \
  -H "Authorization: Bearer $SOURCEVERIFY_API_KEY"
```

Display:
```
## SourceVerify Token Balance
- Available: X tokens
- Subscription: X tokens
- Purchased: X tokens
- Pending (in-progress): X tokens
- Plan: [plan name]
```

### 3. View History

If `$ARGUMENTS` is "history" or the user asks about past verifications:

```bash
curl -s -X GET "https://sourceverify.ai/api/verify-reference/history?limit=10" \
  -H "Authorization: Bearer $SOURCEVERIFY_API_KEY"
```

Display a table of recent verification documents with their status and reference counts.

### 4. Cancel Jobs

If `$ARGUMENTS` contains "cancel" followed by a document ID or job IDs:

```bash
curl -s -X POST https://sourceverify.ai/api/verify-reference/cancel \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $SOURCEVERIFY_API_KEY" \
  -d '{"documentId": "..."}'
```

Report how many jobs were cancelled and tokens refunded.

## Important Notes

- Always check that `SOURCEVERIFY_API_KEY` is set before making any API call
- Each reference consumes 1 token
- Verification typically takes 10-45 seconds per reference
- The API returns corrected APA7 formatted references when possible
- Never expose the API key in output
- If polling takes more than 3 minutes total, warn the user and ask if they want to continue waiting
