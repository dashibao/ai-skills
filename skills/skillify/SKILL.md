---
name: skillify
description: Capture this session's repeatable process into a skill. Call at end of the process you want to capture with an optional description.
---

# Skillify: Create a Skill from This Session

You are capturing this session's repeatable process as a reusable skill.

## Your Task

### Step 1: Analyze the Session

Before asking any questions, analyze the session to identify:
- What repeatable process was performed
- What the inputs/parameters were
- The distinct steps (in order)
- The success artifacts/criteria for each step
- Where the user corrected or steered you
- What tools and permissions were needed
- What agents were used
- What the goals and success artifacts were

### Step 2: Interview the User

Use AskUserQuestion for ALL questions. Never ask questions via plain text.

**Round 1: High level confirmation**
- Suggest a name and description for the skill based on your analysis
- Suggest high-level goal(s) and specific success criteria for the skill

**Round 2: More details**
- Present the high-level steps you identified as a numbered list
- If you think the skill will require arguments, suggest arguments based on what you observed
- Ask if this skill should run inline (in the current conversation) or forked (as a sub-agent with its own context)
- Ask where the skill should be saved:
  - **This repo** (`.claude/skills/<name>/SKILL.md`) — for workflows specific to this project
  - **Personal** (`~/.claude/skills/<name>/SKILL.md`) — follows you across all repos

**Round 3: Breaking down each step**
For each major step, if it's not glaringly obvious, ask:
- What does this step produce that later steps need? (data, artifacts, IDs)
- What proves that this step succeeded, and that we can move on?
- Should the user be asked to confirm before proceeding?
- Are any steps independent and could run in parallel?
- How should the skill be executed?
- What are the hard constraints or hard preferences?

You may do multiple rounds, one per step, especially if there are more than 3 steps.

**Round 4: Final questions**
- Confirm when this skill should be invoked, and suggest trigger phrases
- Ask for any other gotchas or things to watch out for

### Step 3: Write the SKILL.md

Create the skill file using this format:

```markdown
---
name: {{skill-name}}
description: {{one-line description}}
allowed-tools:
  {{list of tool permission patterns}}
when_to_use: {{when Claude should automatically invoke this skill}}
argument-hint: "{{hint showing argument placeholders}}"
arguments:
  {{list of argument names}}
context: {{inline or fork}}
---

# {{Skill Title}}

## Inputs
- `$arg_name`: Description of this input

## Goal
Clear goal for this workflow with defined artifacts/criteria for completion.

## Steps

### 1. Step Name
What to do in this step. Be specific and actionable.

**Success criteria**: [REQUIRED] What proves this step is done.

**Execution**: Direct (default), Task agent, Teammate, or [human]

**Artifacts**: Data this step produces that later steps need

**Human checkpoint**: When to pause and ask the user

**Rules**: Hard rules for the workflow
```

**Per-step annotations:**
- **Success criteria** is REQUIRED on every step
- **Execution**: `Direct` (default), `Task agent`, `Teammate`, or `[human]`
- **Artifacts**: Data this step produces that later steps need
- **Human checkpoint**: When to pause before proceeding
- **Rules**: Hard rules for the workflow

### Step 4: Confirm and Save

Before writing the file, output the complete SKILL.md content as a yaml code block so the user can review it. Then ask for confirmation using AskUserQuestion.

After writing, tell the user:
- Where the skill was saved
- How to invoke it: `/{{skill-name}} [arguments]`
- That they can edit the SKILL.md directly to refine it

## Usage

```
/skillify                     # Capture current session process
/skillify PR review workflow  # Capture with description
```