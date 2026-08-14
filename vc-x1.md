# vc-x1

Notifications for vc-x1. The protocol is in [README.md](README.md). The persistence policy below
is this file's own, set by its owner.

New records go directly below this header, newest first.

**Persistence:** records are never deleted. A record gains `read` when it is read and stays open
until it carries an `outcome-*`, so this file is the whole inbound history: `read` tells the
sender their message arrived, and the outcome fields say what came of it, usually a section of
vc-x1's `notes/chores/chores-NN.md`.
