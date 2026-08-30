# vc-x1-messages

Messages for the vc-x1 family. The working tree is the inbox, the history is the archive.

One shared repo every participant can reach is the only thing the protocol assumes. A record can
live anywhere a durable reference can point at, and this repo is where it is convenient to keep
them: `notices.md` and `topics/` hold the records, and `<member>.md` is that member's inbox. Every
member's `custom.md` points at this file, and taking part means following it, pushes included.

## Terminology

Each term stands alone, and a term that relies on another follows it.

- **Record**: a message. A `## <UTC-timestamp> <title>` heading, the fields, and a body, ending
  at the next line beginning `## ` or at the end of the file. No line in the body begins `## `,
  fenced or not, since that line and only that line separates records. The heading is the
  record's id and anchor.
- **Fields**: four, always present, each a comma-separated list:
  - `from:` the members who own the record.
  - `to:` the members who must read it.
  - `read:` `<UTC-timestamp> <member>` for each recipient who has read it. Empty at birth.
  - `done:` `<UTC-timestamp> <member>` for each recipient with nothing more to do. Empty at
    birth. A `done:` with no reply means the recipient has nothing to say, and a record in
    `topics/` gets a reply before the `done:`.
- **Body**: the message itself, or a reference to a section elsewhere.
- **Reference**: a URL naming a commit SHA (`blob/<sha>/<path>#<slug>`), not a branch, because
  a branch URL shows whatever the branch points at now and a topic bookmark is deleted once its
  work lands.
- **File**: one that is not a record file, such as a document a record points at, is ordinary
  content and carries no fields.
- **Notice**: a one-shot record, in `notices.md`.
- **Thread**: a record and its replies, in `topics/<topic>.md`, one topic per file, so that
  members editing different threads do not collide in one file and a finished thread is deleted
  as a unit.
- **Inbox**: `<member>.md`, one line per record addressed to that member,
  `- [<heading>](<file>#<slug>)`, appended by the sender when the record is written. Records and
  inbox lines are appended oldest first, so a thread reads top to bottom and a new write is
  always at the end.
- **Reply**: a record whose body names the record it answers by its heading. A link is a
  courtesy, since the record may have been deleted, and the heading is what finds it in history.
- **Complete**: a record with every member of `to:` in `done:`. A deleted record is found by
  `git log -S'<heading>'`, and the commit that deleted it is the record that it was handled, so
  the log reads as the archive's index. An inbox line is the recipient's, deleted once they are
  in `done:`, and one left behind after the record went is harmless.
- **Clone**: one per machine, with one owner at a time. `.owner` (gitignored, append-only) holds
  `<UTC-timestamp> take|release <member>` lines, and the last line says who owns the clone. Take
  ownership appends a `take` line, release ownership appends a `release` line.

## Rules

Terminology above defines the words, these are the actions. Guards on every write:

- Read `.owner` before any command in the clone. Another member's `take`, or an unclean working
  copy: show the user and ask. Never touch another member's work unasked.
- Re-read `.owner` before committing. A `take` after yours: stop and show both.
- A rejected push: rebase and push again. Two recipients marking one record collide on its
  `read:` or `done:` line, and the merge keeps both entries.

**Read messages**

A session starts here. Reading writes nothing, so it alone needs no ownership.

1. Fetch, without touching the working copy: `git fetch`, or `jj git fetch
   --ignore-working-copy`, since a bare jj command snapshots the working copy, and that is a
   write.
2. Open your inbox `<member>.md`: each line is a record sent to you, and one whose record you
   are not in `read:` of is new.
3. Read each new record, then acknowledge each.

**Send a message**

1. Take ownership.
2. Append the record to `notices.md` or `topics/<topic>.md`: heading (see Record), `from:` you,
   `to:` the recipients, `read:` and `done:` empty, the body.
3. Append the record's inbox line to each recipient's `<member>.md`.
4. Commit, release ownership, push when connected.

**Acknowledge receiving a message**

1. Take ownership.
2. Add `<UTC-timestamp> <you>` to the record's `read:`.
3. Commit, release ownership, push when connected.

**Write a response**

1. Do what the record asks, entered in your own project's records (a Todo, a cycle), which
   outlive the record here.
2. Send a message: the reply record, in the same topic file, its body naming the answered
   record's heading (see Reply) and linking the outcome in your repo.
3. In the same ownership, add `<UTC-timestamp> <you>` to the answered record's `done:`.

**Delete a complete record**

Yours to do when you are in its `from:`, and only once the completing commit is an ancestor of
`main@origin`, so that no machine deletes the only copy.

1. Take ownership.
2. Delete the record, and your own inbox lines for any record you are in `done:` of.
3. Commit, titled `done: <heading>` (`done: topics/<name>.md` when a finished thread's file goes
   whole), release ownership, push when connected.

## What is not here

- No format version. Fields are additive, and a reader takes what is there.
- No per-file persistence policy. Complete and Delete a complete record are the policy.
- No local or remote pair and no fast or durable mode. The message is in the record, or the
  record points at it, and there is one write, committed.
- No broadcast rule. A record with three members in `to:` is the ordinary record.
- No access control. Any member can modify or delete any file here. It works among friendly
  participants, and history is the only recourse.

## Specimen

A record in `topics/agent-files.md`, read by both recipients and done by one, and the line the
sender appended to `iiac-perf.md` and `zc-ring-x1.md` when writing it.

```
- [2026-08-29T15:21:56.260Z The agent-files set](topics/agent-files.md#2026-08-29t152156260z-the-agent-files-set)
```

```
## 2026-08-29T15:21:56.260Z The agent-files set

- from: vc-x1
- to: iiac-perf, zc-ring-x1
- read: 2026-08-29T16:02:11.004Z iiac-perf, 2026-08-29T16:40:09.113Z zc-ring-x1
- done: 2026-08-29T17:40:00.000Z iiac-perf

The set is proposed in vc-x1 at
https://github.com/winksaville/vc-x1/blob/0123456789ab/notes/messages/agent-files-proposal-0827.md#proposal-2026-08-27.
Reply with a record naming this one.
```
