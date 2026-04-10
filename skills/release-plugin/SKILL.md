---
name: release-plugin
description: Ship a Claude Code plugin update — detect the plugin, determine the version bump, update plugin.json and CHANGELOG, stage surgically, commit, push, and (for multi-branch repos) merge to the default branch via a worktree. Prints the /plugin update and /reload-plugins commands for the user to run in Claude Code. Use this skill whenever the user has finished editing a plugin's skills and wants to publish the changes, or says "release the plugin", "ship the skill update", "publish this", "bump and push", "push the new version", or any phrase about getting plugin changes live. Also use this preemptively whenever you're about to commit changes inside a `.claude-plugin/` or `plugins/*/` directory — the version-bump step is the most-forgotten part of the workflow and silently leaves changes unshipped.
---

# Release Plugin

Ship a Claude Code plugin update cleanly. The workflow is five steps (edit, bump, commit, push, update) but has sharp edges — the version-bump step is the silent failure mode, and repos with uncommitted WIP need a worktree to avoid disturbing your working state. This skill encodes the discipline.

---

## Contract

**Input:** the current state of the plugin source on disk (you've already edited the skill files). Optionally a starting directory.

**Output:** committed and pushed changes, with the plugin's version bumped and the CHANGELOG updated. Plus a printed pair of commands for the user to run in Claude Code's UI.

**What this skill does NOT do:**
- Run `/plugin update` or `/reload-plugins` — these are Claude Code UI commands, not shell-reachable. You print them for the user.
- Write your skill content for you — this skill ships what's on disk, it doesn't author changes.
- Decide whether your changes are good — it just ships them. Run `max:verify-before-done` beforehand if the changes are load-bearing.

---

## Phases

### Phase 1: Detect the plugin

Find the plugin.json that governs the changes being shipped.

1. Start from the current working directory. If cwd is inside a `.claude-plugin/` sibling directory or inside a `skills/<name>/` directory, walk up until you find `.claude-plugin/plugin.json`.
2. Read that plugin.json and note:
   - `name` — the plugin's canonical name (e.g., `max`, `bugbook`)
   - `version` — current version (e.g., `0.3.0`)
   - `repository` — the git remote URL
3. Check for a sibling `.claude-plugin/marketplace.json` in the same repo (may be at repo root). Note its `name` field — this is the marketplace suffix used in `/plugin install` and `/plugin update` commands.
4. If the plugin lives inside a subdirectory of a larger repo (e.g., `plugins/bugbook/` inside the Bugbook app repo), record that subdirectory path — you'll need it for surgical staging later.

If any of these can't be resolved, stop and ask the user to clarify which plugin they're updating.

### Phase 2: Verify there are changes to ship

Run `git status` in the repo root. Look for modified or new files under the plugin's path.

- If nothing has changed: tell the user there's nothing to ship and exit.
- If only `plugin.json` has changed: that's a version-bump-only commit, which is unusual — ask the user what content change they meant to include.
- If substantive changes exist: proceed.

**Critical check:** if the repo also has unrelated modified files (a working tree mixed with WIP), note them now. You'll need to avoid staging them in Phase 6.

### Phase 3: Determine the version bump

Look at the git diff of the changes being shipped (`git diff <plugin-subdir>` if the plugin is in a subdirectory, `git diff` otherwise) and recommend a version bump based on this rubric:

| Bump type | When to use | Examples |
|---|---|---|
| **Patch** (0.x.y → 0.x.y+1) | Description tweaks, typo fixes, README corrections, frontmatter polish, CHANGELOG updates, minor clarifications | "Fix install command in README", "Improve grill-me description pushiness" |
| **Minor** (0.x.y → 0.(x+1).0) | New skills, new phases in existing skills, new behaviors, new bundled resources, new composition links | "Add release-plugin skill", "Add ambiguity report persistence to grill-me" |
| **Major** (0.x.y → 1.0.0) | Breaking changes to how skills are invoked, removed skills, renamed skills, changed composition contracts | "Rename write-prd to draft-prd", "Remove tech-spec grill gate requirement" |

Present the recommended bump to the user with 1-2 sentences of reasoning. Let them override.

### Phase 4: Update plugin.json and CHANGELOG

Update `plugin.json`:
```json
{
  "version": "<new version>"
}
```

Update `CHANGELOG.md` if it exists. Add a new section at the top in the same style as the previous entries:

```markdown
## v<new version> — <YYYY-MM-DD>

<Short paragraph summarizing the change.>

### <Category (Added / Changed / Fixed / Removed)>

- **<skill or area>** — <what changed and why>
```

If there's no CHANGELOG, skip this step silently — not every plugin has one.

### Phase 5: Draft the commit message

Read the git diff and compose a commit message:
- **First line:** imperative summary under 72 chars (e.g., "Add release-plugin skill")
- **Blank line**
- **Body:** 2-5 bullet points or a short paragraph explaining what changed and why. Mention the version bump.

Present the draft to the user and let them edit. If they approve, proceed.

### Phase 6: Stage surgically and commit

Change directory to the repo root. Then:

**If the repo has no WIP** (only the plugin changes are modified):
```bash
git add -A
```

**If the repo has WIP unrelated to the plugin** (the critical case — this is what the bugbook plugin hits because the Bugbook app repo has ongoing Swift work):
```bash
git add <specific plugin paths>
# e.g., git add .claude-plugin plugins/bugbook
```

Never use `git add -A` or `git add .` when WIP is present. You'll sweep up the user's in-progress work and commit it accidentally.

Then:
```bash
git commit -m "<message from Phase 5>"
```

### Phase 7: Push to the right branch

Determine which branch to push to:

1. Check `git symbolic-ref refs/remotes/origin/HEAD` for the remote's default branch (usually `main`).
2. Check the current branch with `git branch --show-current`.
3. If current branch is the default branch: push directly.
4. If current branch is a feature branch (e.g., `dev` in the Bugbook case): push the feature branch first, then merge to default via worktree (Phase 8).

```bash
git push origin <current-branch>
```

### Phase 8: (Conditional) Merge to default branch via worktree

**Only run this phase if the current branch is NOT the default branch** (i.e., you pushed to `dev` and need to get to `main`).

The point of using a worktree is to avoid disturbing the user's WIP by switching branches in the main working tree.

```bash
# Create a temporary worktree for the merge
git worktree add /tmp/plugin-release-merge <default-branch>

# Move into it and merge
cd /tmp/plugin-release-merge
git pull origin <default-branch> --ff-only
git merge <current-branch> --no-edit
git push origin <default-branch>

# Clean up
cd <original-repo-path>
git worktree remove /tmp/plugin-release-merge --force
git worktree prune
```

If the merge produces conflicts (rare for plugin-only changes, since plugins don't usually collide with app code), stop and hand off to the user.

### Phase 9: Print the follow-up commands

Print the exact commands the user needs to type in Claude Code's UI:

```
/plugin update <plugin-name>@<marketplace-name>
/reload-plugins
```

Where:
- `<plugin-name>` is the `name` field from the plugin's `plugin.json`
- `<marketplace-name>` is the `name` field from the marketplace's `marketplace.json`

Example for the `max` plugin:
```
/plugin update max@max4c-skills
/reload-plugins
```

Example for the `bugbook` plugin:
```
/plugin update bugbook@max4c-bugbook
/reload-plugins
```

Explicitly tell the user: "These are Claude Code UI commands — you need to type them yourself, I can't run them for you."

### Phase 10: (Optional) Cache cleanup

Offer to clean up old cached versions of the plugin that are no longer needed:

```bash
ls ~/.claude/plugins/cache/<marketplace-name>/<plugin-name>/
```

For each directory that isn't the new version, offer to delete it:
```bash
rm -rf ~/.claude/plugins/cache/<marketplace-name>/<plugin-name>/<old-version>
```

Only delete after the user confirms the new version loaded correctly — deleting prematurely could strand the user if the new version has a bug and they need to roll back.

---

## Red flags

- **"I'll bump the version later."** No. The version bump is what makes the update land. Forgetting it means your changes sit unused and `/plugin update` reports "already at latest". This is the most common failure mode and the main reason this skill exists.
- **"Let me just `git add -A` real quick."** Not in a repo with WIP. Stage specific paths.
- **"I already pushed, let me skip the version bump."** You need to amend or make a new commit that bumps the version. A push without a bump is effectively a no-op from the plugin-updater's perspective.
- **"The commit message can be `update skill`."** No. Future-you will want to know what changed without reading the diff. Write a real message.

---

## Example invocation

User: *"I just tuned the grill-me description, ship it"*

Skill:
1. Detects cwd is inside `~/Code/skills/skills/grill-me/` → plugin is `max` at `~/Code/skills`, marketplace `max4c-skills`
2. `git status` shows `skills/grill-me/SKILL.md` modified — has changes to ship
3. Reads the diff: it's a description edit (frontmatter only). Recommends **patch bump** (0.3.0 → 0.3.1)
4. Updates `plugin.json` version to `0.3.1` and adds a CHANGELOG entry
5. Drafts commit message: "Tune grill-me description for better undertriggering coverage"
6. User approves
7. `git add -A` (clean working tree), `git commit`, `git push origin main`
8. Current branch IS default branch, skip Phase 8
9. Prints:
   ```
   /plugin update max@max4c-skills
   /reload-plugins
   ```
10. Offers to clean up `~/.claude/plugins/cache/max4c-skills/max/0.3.0/` after user confirms the new version loaded

Total elapsed: ~30 seconds of work you didn't have to do yourself.

---

## Composition

**Pairs naturally with:**
- `max:verify-before-done` — run BEFORE release to confirm your skill changes actually do what you expect. Releasing a broken skill is worse than not releasing it.
- `max:grill-me` — run on the CHANGELOG entry if it feels hand-wavy. Ambiguous release notes mean you won't remember what changed when you look back.

**Does NOT compose with:**
- `max:write-prd` or `max:tech-spec` — those are about authoring, this is about shipping. Different phase.

---

## Further reading

- [Claude Code plugin marketplace docs](https://docs.claude.com/en/docs/claude-code/plugins) — the authoritative source on marketplace schema and install commands
- Anthropic's official plugin marketplace at `.claude/plugins/marketplaces/claude-plugins-official/.claude-plugin/marketplace.json` — a real-world example of a multi-plugin marketplace with diverse source types
