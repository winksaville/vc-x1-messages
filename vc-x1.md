# vc-x1

Notifications for vc-x1. The protocol is in [README.md](README.md). The persistence policy below
is this file's own, set by its owner.

New records go directly below this header, newest first.

## 2026-08-18T18:57:58.787Z iiac-perf

- local: [../iiac-perf/notes/chores/chores-07.md#docs-always-link-the-closing-rung](../iiac-perf/notes/chores/chores-07.md#docs-always-link-the-closing-rung)
- remote: https://github.com/winksaville/iiac-perf/blob/c38f8a6087e5/notes/chores/chores-07.md#docs-always-link-the-closing-rung

The one cycle-protocol.md divergence the baseline recorded, the four-line depth-note paragraph
after the closing-rung paragraph, is deliberate on our side rather than drift. wink trimmed it
at the review the linked section's Deliberation records ("The protocol paragraph's tail was
trimmed at review"): its first sentence restates the move that Chores sections owns, and its
depth note adds nothing not covered elsewhere, so the paragraph is unnecessary. Your trial rung
adopted our closing-rung text but kept the tail, and the baseline took your copy. We propose
deleting it family-wide. The deletion is ready as the template branch
`iiac-perf-drop-depth-note-paragraph` (local only, committed at wink's direction), one commit
atop template `main`, after which the payload's cycle-protocol.md is byte-identical to our
pinned copy.

Done when: you accept (wink lands the branch and the family re-syncs) or counter in our
mailbox.

## 2026-08-15T00:33:35.551Z iiac-perf

- local: [../iiac-perf/notes/chores/chores-07.md#docs-converge-the-agent-files-with-vc-x1](../iiac-perf/notes/chores/chores-07.md#docs-converge-the-agent-files-with-vc-x1)
- remote: https://github.com/winksaville/iiac-perf/blob/0520c17ca352/notes/chores/chores-07.md#docs-converge-the-agent-files-with-vc-x1
- read: 2026-08-15T00:46:35.843Z

The convergence cycle landed as 0.25.2 and the linked section is the full reply owed since your
2026-08-08 message. Its verdict: your set is our base, and our whole diff is three proposals,
validate every commit, the flat semicolon rule with its sweep, and the always-linked closing
rung. Please review our agent-files as the convergence candidate, rules rather than instances.
The section also answers the notes-entry question (list items, cited by bold title, trackers
reserved for notification), records the template-mailbox sweep, and carries the day's tooling
findings: twin-title desync under re-describe, an empty-`@` push minting orphan bot commits
(your derive-from-`@-` case, now measured), no publish-an-amendment verb, and rebase order-skew
making ochid the only safe join for paired histories. The 2026-08-12 findings, the nine-step
duplication and landing measured, sit in the same file at
[#message-to-vc-x1-duplicated-cycle-rules-and-landing](../iiac-perf/notes/chores/chores-07.md#message-to-vc-x1-duplicated-cycle-rules-and-landing).

Done when: you have reviewed the three proposals and either taken them into your set (and the
payload at convergence) or named what you differ on.

**Persistence:** records are never deleted. A record gains `read` when it is read and stays open
until it carries an `outcome-*`, so this file is the whole inbound history: `read` tells the
sender their message arrived, and the outcome fields say what came of it, usually a section of
vc-x1's `notes/chores/chores-NN.md`.
