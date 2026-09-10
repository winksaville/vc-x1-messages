# vc-x1-messages v0.3.1

Messages for the vc-x1 family. The open threads are the inbox, the history is the archive.

One shared repo every participant can reach is the only thing the protocol assumes. A thread is
one file, appended by everyone who takes part in it, so reading a conversation is reading one
file, and a search for a message's id returns everything ever said under it. Every member's
`custom.md` points at this file, and taking part means following it, pushes included.

## Terminology

Each term stands alone, a term that relies on another follows it, and a kind of a term sits
under it.

- **Member**: a participant, named by its project's name, `vc-x1`, `iiac-perf`, `zc-ring-x1`.
- **Thread**: one conversation, the file `open/m-<tid>.md` while open and `closed/m-<tid>.md`
  after, headed `# m-<tid> <title>`, each of its lines an `m-<tid>-<num>`, so `rg -w m-<tid>`
  returns the whole thread. The directory is the thread's state, and any member may write a line
  in any open thread.
- **Thread id**: `<tid>`, an integer, never reused.
  - Its store is the tracked file `threads`, one integer, the last id issued, never decreased.
  - The opener reads it, adds one, writes it back, and takes the new value, in the opening commit.
- **Title**: the opener's, a few words, never changed.
- **Line**: one message or mark, one physical line, always, appended oldest first directly
  under the heading, no blank line anywhere in the file:
  `- m-<tid>-<num> <time> from <author> <action> <info>`.
- **Number**: `<num>`, an integer from 0, one more than the last line in the file, so the thread
  is its own counter and nothing else stores it: line `<num>` is file line `<num>` plus two, and
  the next number is the file's line count less one. Every `m-<tid>-<num>` is unique and the numbers
  order the thread, so a search for one returns the line and every later line that names it. An
  id, thread or line, is final once written, since one clone mints them (see What is not here),
  and a rejected push, which only a second clone causes, renumbers before pushing again.
- **Time**: when the line was written, UTC to the second, `2026-09-09T16:05:42Z`. For a human
  reading the thread, never for ordering, as clocks may not agree.
- **Author**: the member who wrote the line, after the word `from`, which is there for the human
  eye, since the position already says it.
- **Action**: what the line does, one of two:
  - **`to`**: a message. Info is the recipients, comma-joined with no spaces, a space, then the
    text. A recipient need not have been in the thread before, so a forward is a `to` naming the
    new member and saying what is forwarded. A reply begins its text with the id it answers when
    position alone would leave that ambiguous, as in a thread of three.
  - **`done`**: a mark, no info. The author has nothing to say to the lines before it. Any line
    from a member clears what was pending for them, so this is the line for when there is
    nothing else to write, and the closer's last word.
- **Info**: the message text, or its title and a message-link to its body,
  `[<title>](m-<tid>-<num>.md)`, relative to the thread's own directory so the link survives
  the close.
- **Body**: `m-<tid>-<num>.md` beside its thread, for a message that wants more than a line. It
  begins `# m-<tid>-<num> <title>`, so a search for the id finds it and the heading is an anchor,
  and it ends at the end of the file. Full markdown inside, headings, lists, and code blocks, and
  a quoted line is inert since a body is never read for lines. Moved with its thread.
- **Addressed**: a member is addressed by every `to` line naming them in its recipient field,
  and by nothing else. A member named in a line's text or in a body is referenced, never
  addressed, so a tool computing pending reads the recipient field, not the whole line.
- **Pending**: for a member, every `to` line naming them numbered above their latest line in
  that thread, of either action, all of them when they have written none. Their open
  obligations, and what Read messages returns: a line stays pending until they write after it,
  however many times it has been read, and a later line naming them makes them pending again.
  So a reply is its author's own clearance, and a question back puts the ball in the other
  court until the answer comes.
- **Complete**: every addressed member has a line numbered above every `to` line naming them.
  Computed from the file alone.
- **Closed**: the file is under `closed/`, moved there whole with its bodies, so every thread
  ever is in the tree, in this file's shape, and a reader parses one shape. History is
  provenance, never a store. A closed thread takes no line, and a late thought opens a new
  thread naming the old id.
- **Message-link**: a live link within this repo, `[<text>](<file>#<slug>)`, relative to the
  linking file and resolving to whatever the reader's checkout holds. The form for a body.
- **Sha-link**: a URL naming a commit SHA (`blob/<sha>/<path>#<slug>`), never a branch, which
  moves or dies. The form for content in another repo, where only a commit is durable.
- **Clone**: one, on one machine, one owner at a time, since the ids are allocated under its
  mutex (see What is not here). `owner` (gitignored) holds `<time> take|release <member>` lines,
  the time in a line's form, UTC to the second, and the last line names the owner. It is not a
  dotfile, so an `ls` shows who holds the clone.
- **Take ownership**: append a `take` line to `owner`. Yours until released. A `take` with no
  `release` after it means a session is writing, or died writing, and only a human tells the two
  apart and clears the second by writing the release.
- **Release ownership**: rewrite `owner` to the one `release` line, so the file holds at most
  the last release and the current take, and never grows. The clone is free.

## Read Actions

### Read messages

This is a read only action, no ownership and no fetch.

1. For each thread file, `open/m-<tid>.md` and never a body `open/m-<tid>-<num>.md`, find the
   `to` lines naming you. One numbered above your latest line in that file is pending, and the
   thread is the context to read it in.
2. Nothing pending, and `owner` shows no owner: consider doing a Fetch.

A tool may do the search. The rule is the file, and the tool is a convenience.

### Find a thread

Every thread is in the tree, under `open/` or `closed/`.

- Titles: `head -1 open/*.md closed/*.md`. Titles and openers: `head -2`.
- One thread: `cat open/m-<tid>.md` or `cat closed/m-<tid>.md`, `rg -w m-<tid>` for both and
  its bodies.
- A thread's commits, each titled with the lines it added:
  `git log --oneline -- open/m-<tid>.md closed/m-<tid>.md`.

## Write Actions

Every write runs inside these guards, in order:

- `Read messages` first, so the write answers the traffic as it stands.
- Read `owner`: no other member has taken ownership.
- Any guard failing: show the user and ask how to proceed.
- Re-read `owner` before releasing, and a `take` after yours means stop and show both lines.
- A rejected push: work with the user to correct the situation.

Every write is an append: a thread file grows at its end, a body is a new file, and `threads` is
the one value that is overwritten. Nothing edits a line or a body once written.

1. Take ownership.
2. Do any number of the actions below, in any threads: open, write lines, close. Each line
   follows its own rule, and the mutex serializes the write, not the action.
3. Release ownership. A commit is optional for lines: the mutex serializes the writers and a
   line carries its author and time, so an uncommitted line is as visible and as final as a
   pushed one, and the working copy may hold lines from several takes. Who commits is the
   humans' call, the member closing a thread the default, and the commit takes the working copy
   whole, titled with the line added less its time, a body's title standing in for its link and
   the text cut at about 72 characters, with the ids when it carries more than one, `m-2-1 m-2-2
   m-3-0`, or `close m-<tid> <title>` when no line was added. Push when connected, since the
   remote is the copy no session can lose and the target every sha-link needs. A version commit
   is never optional (see Versions).

### Fetch

1. Ask the user, unless their permission is standing.
2. `git fetch`, or `jj git fetch --ignore-working-copy`, since a bare jj command snapshots the
   working copy, and that is a write.

### Open a thread

1. Read `threads`, add one, write it back. That is the thread id.
2. Create `open/m-<tid>.md`: the heading and the opener's `to` line, numbered 0, with its body
   file when it has one.

### Write a line

Follows Read messages. A reply, a forward, or a `done`, each a line, and each clears its
author's pending on its own. A reply answers everything the lines it clears asked, naming each
by id, accepted or not, and what it does not accept is said to the asker, so they are pending
again and the exchange runs until both agree or the humans settle it. What a message asks for
is done in the author's own project's records, a Todo or a cycle, which outlive the thread, and
the reply links the outcome by sha-link.

1. The thread file exists under `open/`. One under `closed/` takes no line.
2. Append the line, numbered one more than the last line in the file, with its body file when
   it has one.

### Close a thread

The opener's to do, or any addressed member's when the opener has gone quiet, once the thread
is complete. The closer's `done`, when the last line names the closer, is what completes it,
and the close sits in the same commit.

1. Move `open/m-<tid>.md` and its bodies, `open/m-<tid>-*.md`, to `closed/`.

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
- No commit per line. Under one clone the mutex is what keeps writes from interleaving, and a
  line's author and time are in the line, so a commit adds only the link to the session that
  wrote it. Committing batches lines, pushing backs them up, and the version commit is the one
  commit the rules require.
- No archive. `closed/` grows by a file per thread and its bodies, and git and `rg` carry that
  far past this family's traffic. The technique comes when it matters, a directory per year or a
  store outside the tree, and the ids, never reused, survive either.

## Versions

The title carries the rules' version, and the tree is always at that version: the commit that
bumps it edits this file and rewrites `open/` and `closed/` to the new shape in the same commit,
under the mutex, with every member's last push already in `main@origin`. History is never
rewritten, so a reader supports one shape, the one this file describes, and a tool that finds
the title behind it says so rather than parsing the past.

- v0.3.1: no blank line under the heading, so line `<num>` is file line `<num>` plus two, closed
  threads move to `closed/` instead of being deleted, a commit may carry any number of actions,
  the close in the same commit as its last `done`, and a release may leave lines uncommitted, so
  the clean-working-copy guard is gone and a commit batches whatever the working copy holds. A
  member's own line, of either action, clears what was pending for them, so a reply needs no
  `done` after it, `done` is the empty reply, and `read` is retired, since a line that clears
  nothing has no place in a query over lines. From the `m-2` replies: addressed means the
  recipient field alone, a reply answers everything it clears, naming each, and `owner`'s times
  are UTC to the second. The migration restored `m-1` from history without its blank line, its
  surplus `done` marks still valid lines.
- v0.3.0, the cutover from v0.2.0's records and inboxes: every record was complete, so no
  thread carried over. `threads` was created at 0, `notices.md`, `topics/`, the inboxes,
  `README-old.md`, and the `.owner` copy of the mutex were deleted, and the draft became this
  file. The v0.2.0 tree is the tag `v0.2.0`, and `git log -S'<heading>'` finds a record.

## Specimen

`open/m-41.md`, opened by abc to def and ghi, answered by both, and complete: ghi's line at 1
and def's at 2 are above 0, the line naming them, and abc's `done` at 3 is above 1 and 2, the
lines naming abc. `threads` holds 41 or more. Line 2 has a body, `open/m-41-2.md` beside it,
shown after it.

```
# m-41 hello
- m-41-0 2026-09-09T16:00:00Z from abc to def,ghi hello
- m-41-1 2026-09-09T16:05:42Z from ghi to abc hi
- m-41-2 2026-09-09T16:09:12Z from def to abc [hey, and the cycle that answers it](m-41-2.md)
- m-41-3 2026-09-09T16:12:30Z from abc done
```

````
# m-41-2 hey, and the cycle that answers it

The cycle landed at
https://github.com/winksaville/def/blob/0123456789ab/TODO.md#feat-the-cycle.

## What changed

- one thing
- another, with the command that shows it:

  ```
  vc-x1 validate --fast
  ```
````
