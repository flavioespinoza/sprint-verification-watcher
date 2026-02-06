# SPEC: Sprint Verification Watcher

**Purpose:** Prevent another TASK_6407 mistake. Verify Codex CLI and sprint requirements BEFORE starting any task.

**Trigger:** User says "merc let's go" or hits START on sprint

---

## SYSTEM OVERVIEW

### Existing Functionality (Keep)
```
tribal-knowledge/images/ → watcher.sh → process.sh → TRIBAL_KNOWLEDGE___MERCOR.md → alert.sh (war drums)
```

### New Functionality (Add)
```
"merc let's go" → sprint-guard.sh → Verification Popup → Music plays → Block until confirmed
```

---

## COMPONENT 1: Sprint Guard Script

**File:** `agents/mercor-watcher/sprint-guard.sh`

**Triggered by:** Claude detects "merc let's go" or "new merc" command

**What it does:**

1. **Popup appears** with verification checklist
2. **Music starts playing** (indie/lofi playlist)
3. **User must confirm each item** before proceeding
4. **If Codex required:** Extra verification for CLI + logs
5. **Only after ALL confirmations:** Allow sprint to start

---

## VERIFICATION CHECKLIST (In Popup)

### Pre-Sprint Verification

```
┌─────────────────────────────────────────────────┐
│  SPRINT START VERIFICATION                      │
│  (Hit ESC to cancel)                            │
├─────────────────────────────────────────────────┤
│                                                 │
│  Sprint Information:                            │
│  ┌───────────────────────────────────────────┐ │
│  │ Sprint Number: [_____]                    │ │
│  │ Deadline: [_____]                         │ │
│  └───────────────────────────────────────────┘ │
│                                                 │
│  Model Requirements:                            │
│  ┌───────────────────────────────────────────┐ │
│  │ Model A: [ ] Opus 4.6                     │ │
│  │          [ ] Sonnet 4.5                   │ │
│  │          [ ] Other: [_____]               │ │
│  │                                           │ │
│  │ Model B: [ ] GPT Codex 5.3                │ │
│  │          [ ] Claude                        │ │
│  │          [ ] Other: [_____]               │ │
│  └───────────────────────────────────────────┘ │
│                                                 │
│  Special Settings:                              │
│  ┌───────────────────────────────────────────┐ │
│  │ Reasoning Level: [_____]                  │ │
│  └───────────────────────────────────────────┘ │
│                                                 │
│  ⚠️  IF MODEL B = CODEX:                        │
│  ┌───────────────────────────────────────────┐ │
│  │ [ ] Codex CLI installed                   │ │
│  │ [ ] ChatGPT Plus/Pro active               │ │
│  │ [ ] Test session completed                │ │
│  │ [ ] Logs verified at ~/.codex/sessions/  │ │
│  └───────────────────────────────────────────┘ │
│                                                 │
│  Timer:                                         │
│  ┌───────────────────────────────────────────┐ │
│  │ [ ] 95-minute timer set on iPhone         │ │
│  └───────────────────────────────────────────┘ │
│                                                 │
│  [CANCEL]              [START SPRINT] ← (grayed)│
└─────────────────────────────────────────────────┘

🎵 Now Playing: Indie/Lofi Mix
```

**START SPRINT button:**
- Grayed out until ALL required fields filled
- If Model B = Codex: Extra checkboxes must be checked
- Once all confirmed: Button becomes active
- Click START SPRINT: Popup closes, music fades, sprint begins

---

## COMPONENT 2: Codex Verification Module

**File:** `agents/mercor-watcher/verify-codex.sh`

**Triggered by:** User selects "Model B = GPT Codex 5.3" in popup

**What it does:**

### Step 1: Check Codex CLI Installed
```bash
if ! command -v codex &> /dev/null; then
    echo "❌ Codex CLI not found"
    echo "Install instructions: CODEX_SETUP_REQUIRED.md"
    exit 1
fi
```

### Step 2: Check ChatGPT Plus/Pro
```bash
# Check if authenticated
codex auth status || {
    echo "❌ Not authenticated with ChatGPT Plus/Pro"
    echo "Run: codex auth login"
    exit 1
}
```

### Step 3: Verify Logs Directory
```bash
if [ ! -d ~/.codex/sessions ]; then
    echo "❌ Codex sessions directory not found"
    echo "Create test session first"
    exit 1
fi
```

### Step 4: Check for Recent Logs
```bash
# Check if any rollout-*.jsonl files exist
if ! ls ~/.codex/sessions/rollout-*.jsonl &> /dev/null; then
    echo "❌ No Codex session logs found"
    echo "Run test session to verify logs generate"
    exit 1
fi
```

**If all pass:** ✓ Green checkmarks appear in popup
**If any fail:** ❌ Red X appears with error message and link to fix

---

## COMPONENT 3: Music Player

**File:** `agents/mercor-watcher/indie-player.sh`

**Music starts when popup appears**

**Playlist options:**
1. Lofi hip hop beats
2. Indie electronic
3. Chill instrumental

**Commands:**
```bash
# Start playing
./indie-player.sh start

# Stop playing
./indie-player.sh stop

# Fade out (when sprint starts)
./indie-player.sh fadeout
```

**Implementation:**
- Use `afplay` (macOS) or `mpg123` (Linux)
- Loop playlist until popup closed
- Fade out over 3 seconds when START SPRINT clicked

---

## COMPONENT 4: Tribal Knowledge Watcher (Existing - Enhanced)

**Keep existing functionality:**
- Watch `tribal-knowledge/images/`
- Process screenshots with GPT-4V
- Extract insights
- Update `TRIBAL_KNOWLEDGE___MERCOR.md`
- Play war drums alert

**Enhancement:**
- When new insight added, show notification
- Include sprint-relevant warnings from tribal knowledge
- Example: "⚠️ New sprint announcement detected in images/ - verify requirements"

---

## FILE STRUCTURE

```
agents/mercor-watcher/
├── watcher.sh              # Existing - polls images/ every 30s
├── process.sh              # Existing - GPT-4V analysis
├── alert.sh                # Existing - war drums notification
├── logs.sh                 # Existing - colorized log viewer
├── sprint-guard.sh         # NEW - verification popup
├── verify-codex.sh         # NEW - Codex CLI verification
├── indie-player.sh         # NEW - music player
└── config/
    ├── popup-template.txt  # NEW - verification UI template
    └── playlist.m3u        # NEW - indie/lofi playlist
```

---

## INTEGRATION WITH CLAUDE CODE

**In `CLAUDE.md` - Trigger Command section:**

```markdown
## TRIGGER COMMAND

**"merc let's go"** or **"new merc"** = Start Mercor eval sprint process

When Flavio says this, Claude MUST:
1. **Launch sprint-guard.sh** (verification popup)
2. **Wait for user confirmation** from popup
3. **After all verified:** Begin Step 1 of workflow
```

**Claude behavior:**
- Detects "merc let's go" command
- Calls `bash agents/mercor-watcher/sprint-guard.sh`
- Waits for exit code
- Exit code 0 = All verified, proceed
- Exit code 1 = User cancelled or verification failed, abort

---

## USER FLOW

### Scenario 1: All Requirements Met

1. User: "merc let's go"
2. Popup appears, music starts
3. User fills in sprint info
4. User selects Model A: Opus 4.6
5. User selects Model B: Claude (not Codex)
6. User checks "95-minute timer set"
7. START SPRINT button activates
8. User clicks START SPRINT
9. Music fades out, popup closes
10. Sprint begins

**Time:** ~2 minutes

---

### Scenario 2: Codex Required But Not Set Up

1. User: "merc let's go"
2. Popup appears, music starts
3. User fills in sprint info
4. User selects Model B: **GPT Codex 5.3**
5. Extra Codex checkboxes appear
6. User tries to check "Codex CLI installed"
7. Verification runs: `verify-codex.sh`
8. **❌ FAILED: Codex CLI not found**
9. Error message appears: "Install Codex CLI first. See CODEX_SETUP_REQUIRED.md"
10. START SPRINT button stays grayed out
11. User must cancel and fix issue first

**Result:** PREVENTED another TASK_6407 mistake

---

### Scenario 3: User Cancels

1. User: "merc let's go"
2. Popup appears, music starts
3. User realizes they're not ready
4. User hits ESC or clicks CANCEL
5. Music stops, popup closes
6. Sprint does NOT start
7. User can prepare and try again later

---

## TECHNICAL REQUIREMENTS

### Dependencies
- `bash` (shell scripts)
- `osascript` (macOS popup dialogs) or `zenity` (Linux dialogs)
- `afplay` (macOS audio) or `mpg123` (Linux audio)
- `jq` (JSON parsing for Codex verification)

### Installation
```bash
# macOS (already have osascript and afplay)
brew install jq

# Linux
sudo apt-get install zenity mpg123 jq
```

### Permissions
```bash
# Make scripts executable
chmod +x agents/mercor-watcher/sprint-guard.sh
chmod +x agents/mercor-watcher/verify-codex.sh
chmod +x agents/mercor-watcher/indie-player.sh
```

---

## ERROR HANDLING

### If Popup Fails to Launch
```bash
echo "⚠️ Sprint guard popup failed to launch"
echo "Fallback: Manual verification required"
echo "Check: MERCOR_WORKFLOW__REALTIME.md - PRE-START section"
```

### If Music Fails to Play
```bash
# Continue anyway - music is nice but not critical
# Just log the error
echo "⚠️ Music player failed - continuing without audio"
```

### If Codex Verification Fails
```bash
echo "❌ BLOCKED: Codex CLI verification failed"
echo "Model B requires GPT Codex 5.3"
echo "Fix: See CODEX_SETUP_REQUIRED.md"
exit 1  # Do not allow sprint to start
```

---

## TESTING

### Test 1: Popup Launches
```bash
./agents/mercor-watcher/sprint-guard.sh
# Expected: Popup appears, music plays
```

### Test 2: Codex Verification (Not Installed)
```bash
# Simulate Codex not installed
mv /usr/local/bin/codex /usr/local/bin/codex.bak 2>/dev/null
./agents/mercor-watcher/verify-codex.sh
# Expected: ❌ Error message, exit code 1
```

### Test 3: Codex Verification (Installed)
```bash
# Restore codex
mv /usr/local/bin/codex.bak /usr/local/bin/codex 2>/dev/null
./agents/mercor-watcher/verify-codex.sh
# Expected: ✓ All checks pass, exit code 0
```

### Test 4: User Cancels
```bash
./agents/mercor-watcher/sprint-guard.sh
# User hits ESC
# Expected: Music stops, exit code 1, sprint does not start
```

---

## SUCCESS CRITERIA

**Sprint Guard is successful when:**

1. ✓ Popup appears within 2 seconds of "merc let's go"
2. ✓ Music starts playing
3. ✓ All verification fields present
4. ✓ Codex checks run when Model B = Codex
5. ✓ START SPRINT button only activates when all confirmed
6. ✓ Music fades out when sprint starts
7. ✓ Sprint blocked if verification fails
8. ✓ No more TASK_6407 mistakes

---

## FUTURE ENHANCEMENTS

### Phase 2 (Optional)
- Remember last sprint settings
- Auto-fill common configurations
- Sprint history tracking
- Success rate dashboard

### Phase 3 (Optional)
- Integration with Mercor Airtable API
- Auto-fetch sprint requirements from Slack
- Real-time sprint announcement monitoring

---

**Priority:** HIGH - Prevents $0 submissions from wrong model usage

**Effort:** MEDIUM - ~4-6 hours to build and test

**Impact:** HIGH - Saves hours of wasted work + prevents payment loss

---

**Status:** SPEC COMPLETE - Ready for implementation
