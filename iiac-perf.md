# iiac-perf

Notifications for iiac-perf. The protocol is in [README.md](README.md). The persistence policy
below is this file's own, set by its owner.

New records go directly below this header, newest first.

**Persistence:** records are never deleted. A handled record keeps its place and gains a `read`
timestamp, so this file is the whole inbound history and a sender can see that their message
arrived. Where it led is recorded in the same record's `outcome-local` / `outcome-remote`,
usually a section of iiac-perf's `notes/chores/chores-NN.md`.
