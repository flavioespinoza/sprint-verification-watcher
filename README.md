# Sprint Verification Watcher

**Two Modes:**
1. **Tribal Knowledge Mode** - Auto-process Slack screenshots for insights
2. **Don't Do Dumb Shit Mode** - Verify sprint requirements before starting

---

## Mode 1: Tribal Knowledge

Watches `tribal-knowledge/images/` directory. When screenshots dropped:
- GPT-4V extracts insights
- Updates tribal knowledge document
- Plays war drums alert

**Purpose:** Capture platform changes, sprint announcements, edge cases from Slack

---

## Mode 2: Don't Do Dumb Shit

Prevents wrong model selection in Mercor sprints.

**Problem Solved:** TASK_6407 was invalidated because wrong models were used for both Model A and Model B (Sonnet 4.5 instead of Opus 4.6 + GPT Codex 5.3). Result: $0 payment, all work wasted.

---

## What It Does

When you say **"merc let's go"**:

1. Popup appears with verification checklist
2. Indie/lofi music starts playing
3. You fill in sprint requirements
4. If Model B = GPT Codex: Extra verification runs
5. START SPRINT button activates only when ALL verified
6. Music fades out, sprint begins

**If verification fails: Sprint BLOCKED until fixed.**

---

## Components

```
agents/mercor-watcher/
├── sprint-guard.sh      # Verification popup
├── verify-codex.sh      # Codex CLI checker
├── indie-player.sh      # Music player
└── config/
    └── playlist.m3u     # Indie/lofi tracks
```

---

## Verification Checklist

- Sprint number and deadline
- Model A requirements (Opus 4.6? Sonnet 4.5?)
- Model B requirements (GPT Codex 5.3? Claude?)
- Special settings (reasoning level)
- 95-minute timer set
- **IF Model B = Codex:**
  - Codex CLI installed
  - ChatGPT Plus/Pro authenticated
  - Test session completed
  - Logs verified at `~/.codex/sessions/rollout-*.jsonl`

---

## Installation

```bash
# Clone repo
git clone <repo_url> sprint-verification-watcher
cd sprint-verification-watcher

# Install dependencies (macOS)
brew install jq

# Make scripts executable
chmod +x agents/mercor-watcher/*.sh
```

---

## Usage

```bash
# Manual launch (for testing)
./agents/mercor-watcher/sprint-guard.sh

# Integrated with Claude Code
# Add to CLAUDE.md trigger command:
# "merc let's go" → launches sprint-guard.sh automatically
```

---

## Success Criteria

- ✓ Popup launches within 2 seconds
- ✓ Music plays during verification
- ✓ Codex checks run when Model B = Codex
- ✓ START button only active when all confirmed
- ✓ Sprint blocked if verification fails
- ✓ No more wrong-model submissions

---

## Priority

**HIGH** - Prevents $0 submissions from wrong model usage

**Effort:** 4-6 hours to build and test

**Impact:** Saves hours of wasted work + prevents payment loss

---

## Documentation

See [SPEC.md](SPEC.md) for complete technical specification.

---

**Status:** Specification complete - ready for implementation
