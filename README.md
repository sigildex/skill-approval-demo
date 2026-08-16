# Skill approval demo

A minimal repository showing how a Sigildex approval record is enforced by
**repository settings**, not by the tool. It holds one agent skill, one approval
record, the copy-paste CI check from the Sigildex repository, and the code-owner
and branch-protection configuration that turns "the record is consistent" into
"the record was reviewed."

| Path | What it is |
|---|---|
| `.claude/skills/log-summarizer/` | A synthetic example skill (written for the Sigildex repository as example content; it describes no real published skill). |
| `.sigildex/approvals/log-summarizer.lock.json` | Its approval record, written by `sigildex lock`. |
| `.github/workflows/approval-check.yml` | The Sigildex `approval-check` workflow: fails a pull request whose skill and record disagree. |
| `.github/CODEOWNERS` | Puts the approvals directory, the workflow directory, and this file itself under a code-owner team. |
| `vendor/` | Pre-publication only — the packed CLI, digest-pinned in the workflow. Goes away once the package is on npm. |

## What the repository settings enforce

Branch protection on `main`:

- pull requests required; **one approving review from a code owner** required;
  stale reviews dismissed on new pushes;
- the `approval-check` status check required, and the branch must be up to date;
- **enforced for administrators** — an admin who opens a pull request is held to
  the same rule as anyone else;
- no force pushes, no branch deletion.

Together with `CODEOWNERS`, that means a pull request that changes the skill
and regenerates its record *consistently* — which passes `approval-check` —
still cannot merge until a member of the code-owner team who did not author it
approves. Consistency is checked by the tool; approval is a human decision, and
the repository refuses to merge without one.

## What it does not enforce

- Nothing here judges whether a change is *safe*. The check proves the skill and
  the record agree; the reviewer decides whether they should.
- Administrator bypass is a setting. This repository enables "enforce for
  administrators"; a repository that leaves it off lets admins merge without the
  required review. If someone with admin rights can edit the branch-protection
  rule itself, they can also remove it — that is the platform's trust model, and
  no CLI can change it. Audit the settings, not just the workflow.
- The check watches the one skill/record pair named in the workflow's `env`.
  It does not audit the approvals directory for duplicate ids, duplicate
  artifact paths, or orphaned records.

## Trying it

1. Edit `.claude/skills/log-summarizer/SKILL.md` on a branch and open a pull
   request without touching the record → `approval-check` fails: the skill no
   longer matches its approval.
2. On another branch, edit the skill *and* re-run
   `sigildex lock .claude/skills/log-summarizer --out .sigildex/approvals/log-summarizer.lock.json`
   → the check passes, and the pull request stays blocked on a code-owner review.
3. Have a code owner who is not the author approve → mergeable.

Sigildex records byte identity only. It does not attest safety, provenance, or
future content. See the Sigildex repository for the identity specification, the
safe-adoption guide, and the threat model.
