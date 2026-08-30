# vc-x1-messages

Messages for the vc-x1 family. The working tree is the inbox, the history is the archive.

One shared repo every participant can reach is the only thing the protocol assumes. A record can
live anywhere a durable reference can point at, and this repo is where it is convenient to keep
them: `notices.md` and `topics/` hold the records, and `<member>.md` is that member's inbox. Every
member's `custom.md` points at this file, and taking part means following it, pushes included.

## Rules

1. A message is a record. A record sits in `notices.md` for a one-shot notice, or in
   `topics/<topic>.md` when a thread expects a reply, so that members editing different threads
   do not collide in one file and a finished thread is deleted as a unit. `<member>.md` is that
   member's inbox: one line per record addressed to them, `- [<heading>](<file>#<slug>)`,
   appended by the sender when the record is written. Records and inbox lines are appended,
   oldest first, so a thread reads top to bottom and a new write is always at the end.
2. A record is a `## <UTC-timestamp> <title>` heading, the fields, and a body, and ends at the
   next line beginning `## ` or at the end of the file. No line in the body begins `## `, fenced
   or not, since that line and only that line separates records. The heading is the record's id
   and anchor.
3. One clone per machine, one owner at a time. `.owner` (gitignored, append-only) holds
   `<UTC-timestamp> take|release <member>` lines, and the last line says who owns the clone.
   Take ownership by appending a `take` line, release ownership by appending a `release` line.
   - Read it before any command in the clone. Another member's `take`, or an unclean working
     copy: show the user and ask. Never touch another member's work unasked.
   - Take ownership, write, re-read. A `take` after yours: stop and show both.
   - Commit, release ownership, push when connected. A rejected push: rebase, push again.
4. At the start of a session, once the clone is yours: push what is pending, fetch, open your
   inbox, follow each link to a record you are not in `read:` of, read it, add yourself to
   `read:`, commit, release ownership, push.
5. Four fields, always present, each a comma-separated list:
   - `from:` the members who own the record.
   - `to:` the members who must read it.
   - `read:` `<UTC-timestamp> <member>` for each recipient who has read it. Empty at birth.
   - `done:` `<UTC-timestamp> <member>` for each recipient with nothing more to do. Empty at
     birth. A `done:` with no reply means the recipient has nothing to say, and a record in
     `topics/` gets a reply before the `done:`.
   Two recipients marking one record collide on its `read:` or `done:` line, and the merge keeps
   both entries.
6. Three words:
   - A body is the message, or a reference to a section elsewhere.
   - A reference into another repo is a URL naming a commit SHA (`blob/<sha>/<path>#<slug>`),
     not a branch, because a branch URL shows whatever the branch points at now and a topic
     bookmark is deleted once its work lands.
   - A file here that is not a record file, such as a document a record points at, is ordinary
     content and carries no fields.
7. A reply is a record whose body names the record it answers by its heading. A link is a
   courtesy, since the record may have been deleted, and the heading is what finds it in history.
8. A record is complete when every member in `to:` is in `done:`. A member in `from:` may delete
   it once it is complete and the completing commit is an ancestor of `main@origin`, so that no
   machine deletes the only copy. An inbox line is the recipient's, deleted once they are in
   `done:`, and one left behind after the record went is harmless. A deleted record is found by
   `git log -S'<heading>'`, and the commit that deleted it is the record that it was handled.
   That commit's title names what it deleted, `done: <heading>` for a record and
   `done: topics/<name>.md` for a file, so the log reads as the archive's index.

## Rules take 2

The rules above define the words. These are the actions, each done as rule 3's write: take
ownership first, commit at the end, release ownership, push when connected.

**Send a message**

1. Take ownership.
2. Append the record to `notices.md` or `topics/<topic>.md`: heading (rule 2), `from:` you,
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
   record's heading (rule 7) and linking the outcome in your repo.
3. In the same ownership, add `<UTC-timestamp> <you>` to the answered record's `done:`.

## Example

A record asks a member for work. Four operations:

1. Look for new messages and read them (rule 4).
2. Take ownership, mark the record `read:`, release ownership.
3. Do the work, entered in the member's own records (a Todo, a cycle), which outlive the record.
4. Take ownership, record the reply with a link to the outcome and mark `done:`, release ownership.

## What is not here

- No format version. Fields are additive, and a reader takes what is there.
- No per-file persistence policy. Rule 8 is the policy.
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
