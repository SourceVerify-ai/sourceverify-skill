# SourceVerify Skill for Claude Code (Beta)

Verify academic and professional citations directly from Claude Code using the [SourceVerify](https://sourceverify.ai) API.

## Install

**1. Get an API key**

[Sign up](https://sourceverify.ai/signup) and grab your key from [Account → Developer](https://sourceverify.ai/account/developer). Set it in your shell profile:

```bash
export SOURCEVERIFY_API_KEY="sk_your_key_here"
```

**2. Install the skill**

```bash
curl -o ~/.claude/commands/sourceverify.md \
  https://raw.githubusercontent.com/SourceVerify-ai/sourceverify-skill/main/skills/sourceverify/SKILL.md
```

That's it. Restart Claude Code and `/sourceverify` is available.

## Usage

### Verify a reference

```
/sourceverify "Kuhn, T. S. (1962). The Structure of Scientific Revolutions. University of Chicago Press."
```

Or point to a file (one reference per line):

```
/sourceverify references.txt
```

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
2. Each reference is checked against OpenAlex, Google Scholar, and direct URL verification
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

Each reference verification costs 1 token. Get tokens at [sourceverify.ai/pricing](https://sourceverify.ai/pricing).

## Links

- [SourceVerify](https://sourceverify.ai)
- [CLI Tool](https://sourceverify.ai/developers/docs/cli)
- [API Documentation](https://sourceverify.ai/developers/docs/api)
- [Get API Key](https://sourceverify.ai/account/developer)
