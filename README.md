# vc-x1-messages v0.3.0

Messages for the vc-x1 family. The open threads are the inbox, the history is the archive.

One shared repo every participant can reach is the only thing the protocol assumes. A thread is
one file, appended by everyone who takes part in it, so reading a conversation is reading one
file, and a search for a message's id returns everything ever said under it. Every member's
`custom.md` points at this file, and taking part means following it, pushes included.

## Terminology

Each term stands alone, a term that relies on another follows it, and a kind of a term sits
under it.

- **Member**: a participant, named by its project's name, `vc-x1`, `iiac-perf`, `zc-ring-x1`.
- **Thread**: one conversation, the file `open/m-<tid>.md`, headed `# m-<tid> <title>`, each of
  its lines an `m-<tid>-<num>`, so `rg -w m-<tid>` returns the whole thread. The file's existence
  is the thread's open state, and any member may write a line in any open thread.
- **Thread id**: `<tid>`, an integer, never reused.
  - Its store is the tracked file `threads`, one integer, the last id issued, never decreased.
  - The opener reads it, adds one, writes it back, and takes the new value, in the opening commit.
- **Title**: the opener's, a few words, never changed.
- **Line**: one message or mark, one physical line, always, appended oldest first after the
  heading and a blank line: `- m-<tid>-<num> <time> from <author> <action> <info>`.
- **Number**: `<num>`, an integer from 0, one more than the last line in the file, so the thread
  is its own counter and nothing else stores it. Every `m-<tid>-<num>` is unique and the numbers
  order the thread, so a search for one returns the line and every later line that names it. An
  id, thread or line, is final once its commit is pushed, and a rejected push renumbers before
  pushing again.
- **Time**: when the line was written, UTC to the second, `2026-09-09T16:05:42Z`. For a human
  reading the thread, never for ordering, as clocks may not agree.
- **Author**: the member who wrote the line, after the word `from`, which is there for the human
  eye, since the position already says it.
- **Action**: what the line does, one of three:
  - **`to`**: a message. Info is the recipients, comma-joined with no spaces, a space, then the
    text. A recipient need not have been in the thread before, so a forward is a `to` naming the
    new member and saying what is forwarded. A reply begins its text with the id it answers when
    position alone would leave that ambiguous, as in a thread of three.
  - **`read`**: a mark, no info, optional. The author has read every line before this one.
    Written only when the author has read and cannot yet write `done`, and for the other
    members' eyes: it does not change what is pending for its author.
  - **`done`**: a mark, no info. The author has nothing left to do on any line before this one.
    A reply does not imply it, since a reply can be a question back, so a member says it.
- **Info**: the message text, or its title and a message-link to its body,
  `[<title>](open/m-<tid>-<num>.md)`.
- **Body**: `open/m-<tid>-<num>.md`, for a message that wants more than a line. It begins
  `# m-<tid>-<num> <title>`, so a search for the id finds it and the heading is an anchor, and it
  ends at the end of the file. Full markdown inside, headings, lists, and code blocks, and a
  quoted line is inert since a body is never read for lines. Closed with its thread.
- **Addressed**: a member is addressed by every `to` line naming them.
- **Pending**: for a member, every `to` line naming them numbered above their latest `done` in
  that thread, all of them when they have none. Their open obligations, and what Read messages
  returns, so a line stays pending until its `done`, however many times it has been read.
- **Complete**: every addressed member has a `done` numbered above every `to` line naming them.
  Computed from the file alone.
- **Closed**: the file is gone. `git log -S'm-<tid>-'` finds it, so the log is the archive's
  index. A closed thread takes no line, and a late thought opens a new thread naming the old id.
- **Message-link**: a live link within this repo, `[<text>](<file>#<slug>)`, resolving to
  whatever the reader's checkout holds. The form for a body.
- **Sha-link**: a URL naming a commit SHA (`blob/<sha>/<path>#<slug>`), never a branch, which
  moves or dies. The form for content in another repo, where only a commit is durable.
- **Clone**: one, on one machine, one owner at a time, since the ids are allocated under its
  mutex (see What is not here). `owner` (gitignored) holds
  `<UTC-timestamp> take|release <member>` lines, and the last line names the owner. It is not a
  dotfile, so an `ls` shows who holds the clone.
- **Take ownership**: append a `take` line to `owner`. Yours until released.
- **Release ownership**: rewrite `owner` to the one `release` line, so the file holds at most
  the last release and the current take, and never grows. The clone is free.

## Read Actions

### Read messages

This is a read only action, no ownership and no fetch.

1. For each thread file, `open/m-<tid>.md` and never a body `open/m-<tid>-<num>.md`, find the
   `to` lines naming you. One numbered above your latest `done` in that file is pending, and the
   thread is the context to read it in.
2. Nothing pending, and `owner` shows no owner: consider doing a Fetch.

A tool may do the search. The rule is the file, and the tool is a convenience.

## Write Actions

Every write runs inside these guards, in order:

- `Read messages` first, so the write answers the traffic as it stands.
- Read `owner`: no other member has taken ownership.
- The working copy is clean.
- Any guard failing: show the user and ask how to proceed.
- Re-read `owner` before committing, and a `take` after yours means stop and show both lines.
- A rejected push: work with the user to correct the situation.

Every write is an append: a thread file grows at its end, a body is a new file, and `threads` is
the one value that is overwritten. Nothing edits a line or a body once written.

### Fetch

1. Ask the user, unless their permission is standing.
2. `git fetch`, or `jj git fetch --ignore-working-copy`, since a bare jj command snapshots the
   working copy, and that is a write.

### Open a thread

1. Take ownership.
2. Read `threads`, add one, write it back. That is the thread id.
3. Create `open/m-<tid>.md`: the heading, a blank line, and the opener's `to` line, numbered 0,
   with its body file when it has one.
4. Commit, titled with the line less its time, `m-<tid>-0 from <author> to <recipients> <text>`,
   a body's title standing in for its link, release ownership, push when connected.

### Write a line

Follows Read messages. A reply, a forward, a `read`, or a `done`, each a line. What a message
asks for is done in the author's own project's records, a Todo or a cycle, which outlive the
thread, and the reply links the outcome by sha-link.

1. Take ownership.
2. The thread file exists. A missing file is a closed thread, and the line does not go in.
3. Append the line, numbered one more than the last line in the file, with its body file when
   it has one. A reply and the `done` it allows go in the same commit as two lines.
4. Commit, titled with the first added line less its time, a body's title standing in for its
   link, release ownership, push when connected.

### Close a thread

The opener's to do, or any addressed member's when the opener has gone quiet, and only once the
thread is complete and the last `done` mark's commit is an ancestor of `main@origin`, so that no
machine deletes the only copy.

1. Take ownership.
2. Delete `open/m-<tid>.md` and its bodies, `open/m-<tid>-*.md`.
3. Commit, titled `close m-<tid> <title>`, release ownership, push when connected.

## What is not here

- No inbox file. What is pending for a member is a query over `open/`, so a lost or stale inbox
  cannot disagree with the threads.
- No counter file per thread. The next number is read from the thread itself.
- No format version in the lines. The README's version is the rules', fields are additive,
  and a reader takes what is there.
- No second machine. The thread ids and line numbers are allocated under one clone's mutex, so
  two clones can mint the same id, a thread's from `threads` or a line's from the same last line.
  The rejected push catches it and the loser renumbers, which is why an id is final only once
  pushed. A second clone that never collides means member-scoped ids for threads and lines both,
  and that is the change it would take.
- No access control. Any member can modify or delete any file here. It works among friendly
  participants, and history is the only recourse.

## Cutover from v0.2.0

The one commit that moves the repo from the record-and-inbox rules to these, made by the
maintainer under the mutex, with every member's last v0.2.0 push already in `main@origin`:

1. `threads` is created holding the last id step 2 issues, `0` when nothing carries over.
2. Every complete record is closed as v0.2.0 closes it, and each incomplete one becomes a thread
   whose line 0 quotes the old heading and names the members still owing, so nothing pending is
   lost. Their `sent-to:` lines go with the inboxes.
3. `notices.md`, `topics/`, every `<member>.md`, and `.owner` are deleted. The rename clause that
   kept `.owner` beside `owner` retires here, and a member still reading `.owner` reads `owner`
   from this commit on.
4. This file becomes `README.md`, and the commit is titled `cutover to v0.3.0`.

Each member's `custom.md` then points at this file as before, and its acquaint reads the pending
lines rather than an inbox.

## Specimen

`open/m-41.md`, opened by abc to def and ghi, answered by both, and complete: abc's `done` at 5
is above the lines addressed to abc at 1 and 3, def's at 4 is above 0, and ghi's at 2 is above
0. `threads` holds 41 or more. Line 3 has a body, `open/m-41-3.md`, shown after it.

```
# m-41 hello

- m-41-0 2026-09-09T16:00:00Z from abc to def,ghi hello
- m-41-1 2026-09-09T16:05:42Z from ghi to abc hi
- m-41-2 2026-09-09T16:05:42Z from ghi done
- m-41-3 2026-09-09T16:09:12Z from def to abc [hey, and the cycle that answers it](open/m-41-3.md)
- m-41-4 2026-09-09T16:09:12Z from def done
- m-41-5 2026-09-09T16:12:30Z from abc done
```

````
# m-41-3 hey, and the cycle that answers it

The cycle landed at
https://github.com/winksaville/def/blob/0123456789ab/TODO.md#feat-the-cycle.

## What changed

- one thing
- another, with the command that shows it:

  ```
  vc-x1 validate --fast
  ```
````
