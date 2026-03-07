# SourceVerify Skill for Claude Code (Beta)

Verify academic and professional citations directly from Claude Code using the [SourceVerify](https://sourceverify.ai) API.

## Install

1. Clone the repo:
```bash
git clone https://github.com/SourceVerify-ai/sourceverify-skill.git
```

2. Launch Claude Code with the plugin:
```bash
claude --plugin-dir /path/to/sourceverify-skill
```

## Setup

1. Get an API key at [sourceverify.ai/developers](https://sourceverify.ai/en/developers)
2. Set it in your shell profile:

```bash
export SOURCEVERIFY_API_KEY="sk_your_key_here"
```

## Usage

### Verify references

```
/sourceverify Smith, J. (2020). The Art of Citation. Academic Press.
```

Or point to a file:

```
/sourceverify references.txt
```

Or just paste multiple references and ask Claude to verify them — the skill triggers automatically.

### Check token balance

```
/sourceverify balance
```

### View verification history

```
/sourceverify history
```

### Cancel in-progress jobs

```
/sourceverify cancel doc_abc123
```

## How It Works

1. References are submitted to the SourceVerify API
2. Each reference is verified against multiple sources (OpenAlex, Google Scholar, direct URL checks)
3. An AI review agent analyzes the evidence and assigns a status
4. Results include corrected APA7 formatting and field-level accuracy details

## Verification Statuses

| Status | Meaning |
|--------|---------|
| **VERIFIED** | Reference confirmed as accurate |
| **VERIFIED (with errors)** | Reference exists but has minor inaccuracies |
| **NEEDS HUMAN REVIEW** | Ambiguous results requiring manual review |
| **UNVERIFIED** | Could not confirm the reference exists |

## Pricing

Each reference verification costs 1 token. Get tokens at [sourceverify.ai/pricing](https://sourceverify.ai/en/pricing).

## Links

- [SourceVerify](https://sourceverify.ai)
- [API Documentation](https://sourceverify.ai/en/developers/docs/api)
- [Get API Key](https://sourceverify.ai/en/developers)
