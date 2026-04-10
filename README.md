# ai-skills

Private skill repository for local AI workflows.

This repo stores reusable skills in a stable, portable layout so they can be:

- versioned in Git
- reviewed like normal documentation/code
- synced into `~/.agents/skills/`
- expanded over time with more skills

## Layout

```text
skills/
  npc-bug-fix/
    SKILL.md
    references/
    examples/
scripts/
```

## Current Skills

- `npc-bug-fix`: Teambition bug handling workflow with review, compile checks, smoke checks, commit/push gates, and task status update guidance

## Install / Sync

Sync all repo skills into the local agent skill directory:

```bash
./scripts/sync_skills.sh
```

By default the script syncs to:

```bash
~/.agents/skills
```

You can override the target:

```bash
TARGET_SKILLS_DIR=/path/to/skills ./scripts/sync_skills.sh
```

## Add a New Skill

1. Create a new directory under `skills/<skill-name>/`
2. Add a `SKILL.md`
3. Add `references/` and `examples/` when needed
4. Run `./scripts/sync_skills.sh`
5. Commit and push

## Notes

- This repo is intended to stay private.
- Keep each skill self-contained.
- Avoid storing local OS junk such as `.DS_Store`.
