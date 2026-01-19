# Installation Checklist

Use this checklist to verify your Feature Builder plugin is correctly installed and ready to use.

---

## Pre-Installation

- [ ] Claude Code CLI is installed and working
- [ ] You have access to Opus 4.5 and Sonnet 4.5 models
- [ ] You understand the basic workflow (see [QUICKSTART.md](QUICKSTART.md))

---

## File Structure Verification

Run this command to verify all files are present:

```bash
cd "/Users/jonestrinh/mas plugin"
find . -type f -name "*.md" | sort
```

You should see exactly these 10 files:

```
./MULTIAGENT_SYSTEM_PLAN.md           # Original architecture plan
./QUICKSTART.md                       # Quick start guide
./README.md                           # Full documentation
./skills/feature-builder/evaluator.md # Evaluator agent prompt
./skills/feature-builder/feature-builder.md # Entry point skill
./skills/feature-builder/orchestrator.md # Orchestrator agent prompt
./skills/feature-builder/subagents/backend.md # Backend agent prompt
./skills/feature-builder/subagents/frontend.md # Frontend agent prompt
./skills/feature-builder/subagents/testing.md # Testing agent prompt
./templates/plan-template.md          # Plan file template
```

### Verify Directory Structure

```bash
tree -L 3 "/Users/jonestrinh/mas plugin"
```

Expected structure:

```
mas plugin/
├── INSTALLATION_CHECKLIST.md (this file)
├── MULTIAGENT_SYSTEM_PLAN.md
├── QUICKSTART.md
├── README.md
├── skills/
│   └── feature-builder/
│       ├── evaluator.md
│       ├── feature-builder.md
│       ├── orchestrator.md
│       └── subagents/
│           ├── backend.md
│           ├── frontend.md
│           └── testing.md
└── templates/
    └── plan-template.md
```

---

## Installation Steps

### Step 1: Copy to Claude Code Skills Directory

```bash
# Create the target directory if it doesn't exist
mkdir -p ~/.claude/skills

# Copy the entire plugin
cp -r "/Users/jonestrinh/mas plugin" ~/.claude/skills/feature-builder
```

### Step 2: Verify Installation

```bash
# Check that the skill is recognized
ls -la ~/.claude/skills/feature-builder/skills/feature-builder/
```

You should see:
- `feature-builder.md` (entry point)
- `evaluator.md`
- `orchestrator.md`
- `subagents/` directory

### Step 3: Create Plans Directory

The plugin stores plans in `.mas/plans/`. Create this directory in your project root:

```bash
# In your project directory
mkdir -p .mas/plans
```

Or let the plugin create it automatically on first use.

---

## Verification Checks

### Check 1: Skill Entry Point

```bash
cat ~/.claude/skills/feature-builder/skills/feature-builder/feature-builder.md | head -5
```

Should show:
```markdown
# Feature Builder - Multiagent System for Product Development

You are the **Entry Point** for the Feature Builder multiagent system...
```

### Check 2: Evaluator Agent Prompt

```bash
wc -l ~/.claude/skills/feature-builder/skills/feature-builder/evaluator.md
```

Should show approximately **800-900 lines** (comprehensive prompt).

### Check 3: Subagents Present

```bash
ls ~/.claude/skills/feature-builder/skills/feature-builder/subagents/
```

Should show:
- `backend.md`
- `frontend.md`
- `testing.md`

### Check 4: Template File

```bash
ls ~/.claude/skills/feature-builder/templates/
```

Should show:
- `plan-template.md`

---

## Test Run

### Minimal Test

Try invoking the skill with a simple request:

```bash
# In Claude Code CLI
/feature-builder Create a simple hello world component
```

**Expected behavior**:
1. Evaluator spawns
2. Asks clarifying questions (or proceeds if request is clear)
3. Creates a plan file
4. Waits for your approval

If this works, the plugin is correctly installed!

### Expected Output

```
I'll initiate the Feature Builder multiagent system to develop this feature.

[Spawning Evaluator agent with model: opus]

Evaluator: I'll help you create a hello world component. Let me gather some requirements first.

[AskUserQuestion appears OR plan is created directly]
```

---

## Troubleshooting

### Issue: Skill not recognized

**Symptom**: `/feature-builder` command not found

**Fix**:
```bash
# Verify the skill file exists
ls ~/.claude/skills/feature-builder/skills/feature-builder/feature-builder.md

# If missing, re-copy the plugin
cp -r "/Users/jonestrinh/mas plugin" ~/.claude/skills/feature-builder
```

### Issue: Evaluator doesn't spawn

**Symptom**: Entry point runs but doesn't spawn Evaluator

**Fix**: Check that the entry point skill (`feature-builder.md`) has the correct Task tool invocation:
```bash
grep -A 3 "Task tool:" ~/.claude/skills/feature-builder/skills/feature-builder/feature-builder.md
```

Should show:
```
Task tool:
- subagent_type: "Plan"
- model: "opus"
```

### Issue: Plan files not created

**Symptom**: Evaluator runs but plan.md doesn't appear

**Fix**: Create the plans directory manually:
```bash
mkdir -p .mas/plans
chmod 755 .mas/plans
```

### Issue: Agents fail to spawn

**Symptom**: Orchestrator tries to spawn agents but fails

**Fix**: Verify subagent prompt files exist:
```bash
ls -la ~/.claude/skills/feature-builder/skills/feature-builder/subagents/
```

All three files (backend.md, frontend.md, testing.md) must be present.

---

## Post-Installation

### Recommended: Add to .gitignore

If using version control, consider adding plan directories:

```bash
# In your project's .gitignore
echo ".mas/plans/*.md" >> .gitignore  # If you don't want to track plans
```

Or commit plans for documentation:

```bash
# Keep plans in version control
git add .mas/plans/
```

### Recommended: Review Documentation

Before using in production:

1. Read [QUICKSTART.md](QUICKSTART.md) (5 minutes)
2. Skim [README.md](README.md) (10 minutes)
3. Review [MULTIAGENT_SYSTEM_PLAN.md](MULTIAGENT_SYSTEM_PLAN.md) for architecture details

---

## Checklist Summary

- [ ] All 10 files present (see "File Structure Verification")
- [ ] Plugin copied to `~/.claude/skills/feature-builder`
- [ ] Entry point skill exists and is readable
- [ ] All subagent prompts exist (backend, frontend, testing)
- [ ] Plan template exists
- [ ] Test invocation works (`/feature-builder` responds)
- [ ] `.mas/plans/` directory created in your project (or will be auto-created)
- [ ] Documentation reviewed

---

## Ready to Use!

If all checks pass, you're ready to start building features:

```bash
/feature-builder [your feature description]
```

See [QUICKSTART.md](QUICKSTART.md) for your first feature walkthrough.

---

## Version Information

- **Plugin Version**: 1.0.0
- **Architecture**: Evaluator-Orchestrator Multiagent System
- **Required Models**: Opus 4.5 (Evaluator), Sonnet 4.5 (Orchestrator & Subagents)
- **Platform**: Claude Code CLI

---

## Support

- **Issues**: Check [Troubleshooting](#troubleshooting) section
- **Questions**: See [README.md](README.md) FAQ section
- **Examples**: See [MULTIAGENT_SYSTEM_PLAN.md](MULTIAGENT_SYSTEM_PLAN.md) usage examples

---

*Installation checklist complete. Happy building!* 🚀
