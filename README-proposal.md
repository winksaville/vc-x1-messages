# vc-x1-messages

Messages for the vc-x1 family. The working tree is the inbox, the history is the archive.

One shared repo every participant can reach is the only thing the protocol assumes. A record can
live anywhere a durable reference can point at, and this repo is where it is convenient to keep
them: `notices.md` and `topics/` hold the records, and `<member>.md` is that member's inbox. Every
member's `custom.md` points at this file, and taking part means following it, pushes included.

## Terminology

Each term stands alone, and a term that relies on another follows it.

- **Record**: a message: a `## <UTC-timestamp> <title>` heading, the fields, and a body. Only a
  line beginning `## ` separates records, so no body line may begin one, fenced or not. The
  heading is the record's id and anchor.
- **Fields**: `from:`, the members who own the record, and `to:`, the members who must read it.
  Comma-separated lists, never changed after birth.
- **Body**: the message itself, or a reference to a section elsewhere.
- **Reference**: a URL naming a commit SHA (`blob/<sha>/<path>#<slug>`), never a branch, which
  moves or dies.
- **File**: a non-record file, such as a document a record points at: ordinary content, no
  fields.
- **Notice**: a one-shot record, in `notices.md`.
- **Thread**: a record and its replies, in `topics/<topic>.md`, one topic per file: threads
  never collide, and a finished one is deleted as a unit.
- **Inbox**: `<member>.md`, one line per record addressed to that member,
  `- [<heading>](<file>#<slug>)`, appended by the sender, then marked only by its member:
  `read <UTC-timestamp>` appended on reading, the line deleted when done (after the reply, for
  a thread), the deleting commit carrying who and when. Everything appends oldest first.
- **Reply**: a record whose body names the record it answers by its heading. A link is a
  courtesy, the heading is what finds it in history.
- **Complete**: no `to:` member's inbox still carries the record's line.
  `git log -S'<heading>'` finds what was deleted, so the log is the archive's index.
- **Clone**: one per machine, one owner at a time. `.owner` (gitignored, append-only) holds
  `<UTC-timestamp> take|release <member>` lines, and the last line names the owner.
- **Take ownership**: append a `take` line to `.owner`. Yours until released.
- **Release ownership**: append a `release` line to `.owner`. The clone is free.

## Read Actions

### Read messages

A session starts here. Read only: no ownership, and no fetch.

1. Open your inbox `<member>.md`: each line is a record sent to you, and one without a `read`
   mark is new.
2. Read each new record.
3. Nothing new, and `.owner` shows no owner: consider doing a Fetch.

## Write Actions

Prior to performing these actions, read `.owner` and verify no other member has taken
ownership and the working copy is clean. If not, show the user and ask how to proceed. Re-read
`.owner` before committing, and a `take` after yours means stop and show both lines. If a push
is rejected, work with the user to correct the situation.

### Fetch

1. Ask the user, unless their permission is standing.
2. `git fetch`, or `jj git fetch --ignore-working-copy`, since a bare jj command snapshots the
   working copy, and that is a write.

### Send a message

1. Take ownership.
2. Append the record to `notices.md` or `topics/<topic>.md`: heading (see Record), `from:` you,
   `to:` the recipients, the body.
3. Append the record's inbox line to each `to:` member's `<member>.md`, in the same commit,
   since Complete reads a missing line as done.
4. Commit, release ownership, push when connected.

### Acknowledge receiving a message

Follows Read messages, once per record read.

1. Take ownership.
2. Append `read <UTC-timestamp>` to your line in your own `<member>.md`. With nothing more to
   do, delete the line instead: done, and the commit title is `done: <heading>` (a thread
   record gets a reply first, see Write a response).
3. Commit, release ownership, push when connected.

### Write a response

1. Do what the record asks, entered in your own project's records (a Todo, a cycle), which
   outlive the record here.
2. Send a message: the reply record, in the same topic file, its body naming the answered
   record's heading (see Reply) and linking the outcome in your repo.
3. In the same commit, delete your inbox line for the answered record: done.

### Delete a complete record

Yours to do when you are in its `from:`, and only once it is complete and the last line's
deleting commit is an ancestor of `main@origin`, so that no machine deletes the only copy.

1. Take ownership.
2. Delete the record.
3. Commit, titled `close: <heading>` (`close: topics/<name>.md` when a finished thread's file
   goes whole), release ownership, push when connected.

## What is not here

- No format version. Fields are additive, and a reader takes what is there.
- No per-file persistence policy. Complete and Delete a complete record are the policy.
- No local or remote pair and no fast or durable mode. The message is in the record, or the
  record points at it, and there is one write, committed.
- No broadcast rule. A record with three members in `to:` is the ordinary record.
- No access control. Any member can modify or delete any file here. It works among friendly
  participants, and history is the only recourse.

## Specimen

A record in `topics/agent-files.md`, and iiac-perf's inbox line after acknowledging.
zc-ring-x1's line is already deleted: done, with who and when in the deleting commit.

```
- [2026-08-29T15:21:56.260Z The agent-files set](topics/agent-files.md#2026-08-29t152156260z-the-agent-files-set) read 2026-08-29T16:02:11.004Z
```

```
## 2026-08-29T15:21:56.260Z The agent-files set

- from: vc-x1
- to: iiac-perf, zc-ring-x1

The set is proposed in vc-x1 at
https://github.com/winksaville/vc-x1/blob/0123456789ab/notes/messages/agent-files-proposal-0827.md#proposal-2026-08-27.
Reply with a record naming this one.
```
