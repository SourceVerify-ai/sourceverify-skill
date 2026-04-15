---
name: sourceverify
description: Verify academic and professional citations using the SourceVerify API. Use this skill whenever the user mentions checking references, verifying citations, citation accuracy, fake references, hallucinated citations, reference lists, bibliographies, works cited, APA formatting, or anything related to whether a source actually exists. Also trigger when users paste academic references, share bibliography files, or ask about citation correctness — even if they don't explicitly say "verify."
argument-hint: [references or "balance" or "history"]
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

If `SOURCEVERIFY_API_KEY` is not set, tell the user to get one at https://sourceverify.ai/account/developer and set it in their shell profile.

## Commands

Based on `$ARGUMENTS`, determine which action to take:

### 1. Verify References (default)

If arguments contain reference text, a file path, or the user asks to verify/check citations:

**Step 1 — Extract references**

If the user provided a file path, read the file and extract the references from it.
If the user pasted text, parse individual references from it (split by numbered list, newlines between entries, or other clear delimiters).

Limit to 20 references per request. If more than 20, batch them into multiple requests of up to 20.

**Step 2 — Submit to API**

```bash
curl -s -X POST https://sourceverify.ai/api/verify-reference \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $SOURCEVERIFY_API_KEY" \
  -d '{"references": ["ref1", "ref2", ...]}'
```

Parse the JSON response to get `jobIds` and `documentId`.

If the response has an error (401, 402, 403, 429), report it clearly:
- 401/403: Invalid API key — direct user to https://sourceverify.ai/account/developer
- 402: Insufficient tokens — direct user to https://sourceverify.ai/pricing
- 429: Rate limited — wait 30 seconds and retry once

**Step 3 — Poll for results**

Verification typically takes 15–60 seconds per batch. Poll every 8 seconds until all jobs are completed or failed:

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
- `verified` — ✅ VERIFIED
- `verified-with-error` — ⚠️ VERIFIED (with minor errors)
- `unverified` — ❌ UNVERIFIED
- `needs-human` — 🔍 NEEDS HUMAN REVIEW
- `failed` — ⚠️ ERROR

**Step 5 — Show summary and balance**

After displaying all results, check the token balance and show a summary:

```bash
curl -s -X GET https://sourceverify.ai/api/verify-reference/balance \
  -H "Authorization: Bearer $SOURCEVERIFY_API_KEY"
```

```
## Summary
- Total: X references
- ✅ Verified: X
- ❌ Unverified: X
- 🔍 Needs Review: X
- Tokens remaining: X
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
  -d '{"jobIds": ["id1", "id2", ...]}'
```

Report how many jobs were cancelled and tokens refunded.

## Important Notes

- Always check that `SOURCEVERIFY_API_KEY` is set before making any API call
- Each reference consumes 1 token
- Verification typically takes 15–60 seconds per batch — this is normal, keep polling
- The API returns corrected APA7 formatted references when possible
- Never expose the API key in output
- If polling takes more than 3 minutes total, warn the user and suggest checking `history` later
