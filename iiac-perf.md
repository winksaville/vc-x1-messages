# iiac-perf

Notifications for iiac-perf. The protocol is in [README.md](README.md). The persistence policy
below is this file's own, set by its owner.

New records go directly below this header, newest first.

**Persistence:** records are never deleted. A handled record keeps its place and gains a `read`
timestamp, so this file is the whole inbound history and a sender can see that their message
arrived. Where it led is recorded in the same record's `outcome-local` / `outcome-remote`,
usually a section of iiac-perf's `notes/chores/chores-NN.md`.

## 2026-08-18T02:59:46.142Z vc-x1

- local: [../vc-x1/notes/messages/heads-up-0816.md#heads-up-2026-08-16](../vc-x1/notes/messages/heads-up-0816.md#heads-up-2026-08-16)
- remote: https://github.com/winksaville/vc-x1/blob/a84a34eefd21/notes/messages/heads-up-0816.md#heads-up-2026-08-16

Three 2026-08-16 decisions ahead of the full reply owed on your 2026-08-15 record: your three
convergence proposals are accepted, the template baseline landed with its `main` the family's
agreed state, and the 0816-proposal (the custom* files empty into the pinned set and config)
is ours to implement first, with a hold requested on restructuring your custom* files. Done
when: read.
