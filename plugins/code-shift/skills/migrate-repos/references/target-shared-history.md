# Target reference — shared history, target behind (fast-forward add)

Use this reference when the target **already shares the source's history up to a point** and is simply **behind** — the source has new commits on top and the target has **no commits the source lacks**. The migration is **additive and non-destructive**: new commits are appended, new branches/tags are created, up-to-date refs are skipped. Nothing is force-replaced.

## Dry-run report

```markdown
## Migration plan — SHARED HISTORY (fast-forward add): <source coords> → <target coords>
Target repo: EXISTS ✓  ·  Behind, no divergence  ·  Additive, non-destructive  ·  LFS: yes (M new objects) | no
Default branch: <name> (source) → <name> (target)

Branches: <total> total  ·  <n> new (created)  ·  <n> behind (fast-forwarded)  ·  <n> up-to-date (skipped)
Tags:     <total> total  ·  <n> new (created)  ·  <n> up-to-date (skipped)

Key branch tips (only main/master and develop, shown when they exist):
| Branch      | Source tip | Target tip | +commits to add | State      |
| ----------- | ---------- | ---------- | --------------- | ---------- |
| main/master | abc1234    | 111aaaa    | 5               | behind     |
| develop     | def5678    | def5678    | 0               | up-to-date |

Push is NON-FORCE (append only). A rejected push here means hidden divergence →
stop and switch to the new-or-divergent reference. Nothing on the target is replaced.

Cannot migrate (hidden refs, skipped): refs/pull/*, refs/merge-requests/*, refs/notes/*
```

## Execute (confirm each write)

- Perform a non-force push of every in-scope branch and tag to the target so that the branches that are behind fast-forward up to the source tip, the branches and tags that do not yet exist are created, and the refs that are already up-to-date are left completely untouched.
- Rely on the omission of force as the safety net, since any ref that is not a clean fast-forward is rejected outright rather than silently overwriting whatever is already on the target.
- Fetch the LFS content for the newly added commits from the source before pushing, because mirror clones only download the actual blob content lazily, and then push all of that LFS content across to the target so the large files resolve correctly.
- Set or confirm the target's default branch explicitly, because pushing refs does not by itself update which branch the target treats as its default.
- If the push reports a non-fast-forward rejection for any ref, treat that branch as actually divergent, abort this additive flow, and re-route to [target-new-or-divergent.md](target-new-or-divergent.md), where a force-replace is handled safely.

## Verify

- `git ls-remote target` — confirm each behind branch tip now equals the source tip and each new branch/tag was created.
- Confirm the previously up-to-date branches were **not** rewritten (their tips are unchanged, source == target).
- Confirm the number of added commits matches the dry-run `+commits to add`.
- Confirm the target default branch equals `<name>` and LFS objects resolved.
