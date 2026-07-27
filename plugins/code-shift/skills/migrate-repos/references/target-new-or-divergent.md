# Target reference — new or divergent history

Use this reference when the target is **empty/new** (nothing to overwrite) **or** its history **diverges** from the source. Both cases share the same two safe outcomes: a clean create, or — when conflicting data already exists — **abort (default)** or **override (destructive, force-replace)**.

## Dry-run report

### Empty / new target

```markdown
## Migration plan — NEW target: <source coords> → <target coords>
Target repo: CREATE (private)  ·  LFS: yes (M objects) | no
Default branch: <name> (source) → <name> (target, to be set)

Branches: <total> total, all new (created)
Tags:     <total> total, all new (created)

Key branch tips (only main/master and develop, shown when they exist):
| Branch      | Source tip | Target tip | State |
| ----------- | ---------- | ---------- | ----- |
| main/master | abc1234    | —          | new   |
| develop     | def5678    | —          | new   |

Cannot migrate (hidden refs, skipped): refs/pull/*, refs/merge-requests/*, refs/notes/*
```

### Divergent / unrelated target

```markdown
## Migration plan — DIVERGENT target ⚠️: <source coords> → <target coords>
Target repo: EXISTS ⚠️  ·  Histories diverge  ·  Options: ABORT (default) or OVERRIDE (force-replace)
Default branch: <name> (source) → <name> (target now) → <name> (after override)

If OVERRIDE:
Branches: <n> created (new on target)  ·  <n> deleted (target-only, pruned)  ·  <n> force-updated (divergent, replaced)
Tags:     <n> created  ·  <n> deleted (target-only, pruned)  ·  <n> repointed

Key branch tips (only main/master and develop, shown when they exist):
| Branch      | Source tip | Target tip | Common ancestor | −lost commits | State     |
| ----------- | ---------- | ---------- | --------------- | ------------- | --------- |
| main/master | abc1234    | zzz9999    | 111aaaa         | 3             | diverged  |
| develop     | def5678    | 44ccdd0    | (none)          | —             | unrelated |

−lost = commits currently on the target that OVERRIDE will discard.
OVERRIDE requires a second explicit confirmation naming exactly these refs.
```

## Execute (confirm each write)

- In the new-target case, create the target repository as a completely empty repository and do not allow the host to auto-initialize it with a README, .gitignore, or license, because any seeded commit would turn the very first push into a divergent override that then has to be forced.
- Create the empty target repository on whichever host is being used before any refs are pushed.
- On Azure DevOps, a new project already ships an empty same-named repo, so reuse that repo or create a new one named after the source repo exactly as the user chose in preflight, rather than blindly creating a duplicate.
- For an empty or new target, push every in-scope branch and tag with a plain non-force push, since the operation is purely additive and nothing already on the target can be lost.
- For a divergent target, only proceed once the second explicit confirmation has been given, and then force-replace the target's refs while pruning any target-only branches and tags so the target ends up as an exact mirror of the source, keeping in mind that this step is destructive and effectively irreversible and must never happen silently.
- Fetch all of the LFS content from the source before pushing, because mirror clones download blob content lazily, and then push that LFS content across to the target so the large files resolve.
- Set the target's default branch explicitly, because pushing refs does not by itself update which branch the target treats as its default.
- Restrict the push to branches and tags only, and list the host-managed hidden refs such as `refs/pull/*`, `refs/merge-requests/*`, and `refs/notes/*` under **Cannot migrate** rather than attempting to push them, since they fail with a `deny updating a hidden ref` error.

## Verify

- `git ls-remote target` — confirm every source branch/tag tip is present on the target (mirror equality after an override, or the created set for a new target).
- For an override, confirm the previously divergent target commits are gone (expected) and no unintended branch/tag was pruned.
- Confirm the target default branch equals `<name>`.
- Confirm LFS objects resolved (`git lfs ls-files` on a fresh clone of the target).
