# vc-x1-messages v0.1.0

Notifications for the vc-x1 family. Message bodies live in the sender's own repo, this repo holds
only pointers to them, one record per message.

The version above is the record format's, and changes to it are additive. A field is added, never
renamed or repurposed, so a reader written against an older version still reads today's records.
A major bump would mean that stopped being true and there has not been one.

To send a message create a file with the message and a markdown section header somewhere in your
repo. Then add a record to the file named for the destination you find here, directly below that
file's own header, so the newest record is first. Whether a push sits between those two steps is
[the choice of mode](#two-modes-fast-and-durable) below. A record is a `##` heading and a list of
fields and each field is generally "owned" by the creator, although fixing errors is
permissible. Every field is optional and records routinely carry only two or three, though a
writer aims for at least one of `local` or `remote`, since a record with neither points at
nothing.

## 2026-08-13T19:31:21.123Z iiac-perf

- local: [../iiac-perf/messages/test-msg.md#message1](../iiac-perf/messages/test-msg.md#message1)
- remote: https://github.com/winksaville/iiac-perf/blob/55554b452957/messages/test-msg.md#message1

The heading is `<utc-timestamp> <sender>`. It is also the record's anchor, so a reply or a
record elsewhere can link at this exact entry without anyone inventing an id.

Then a dash list of `- field: <value>`, where every reference may carry an optional markdown
section slug.

- `local:` the message's local file reference.
- `remote:` the message's remote file reference.
- `read:` added by the recipient, a UTC timestamp of when they read it. A record without it is
  unread. Receipt is not completion: a record stays open until it carries an outcome, so a
  member's open traffic is the records in their file with no `outcome-*` field, and `read` is
  what tells the sender their message arrived.
- `outcome-local:` / `outcome-remote:` optional, added by the recipient, pointing at what came of
  the message, the reply or the record of what was decided. They are what let one record hold
  both halves of an exchange, and their arrival is what closes a record.

A handled record, this one sent the other way, as it would sit in `iiac-perf.md`. Two things are
worth noticing. The body is vc-x1's specimen under `notes/messages/` where iiac-perf's sits
under `messages/`, because a body lives wherever its sender chooses. And there is no `remote:`
yet, because the body's commit has not been pushed, which is the ordering rule from
[Two modes](#two-modes-fast-and-durable) holding in this very example; the field gets added when
a push makes the permalink resolvable. The exchange is staged but every reference is live:

## 2026-08-13T20:41:33.512Z vc-x1

- local: [[1]]
- read: 2026-08-13T22:04:07.000Z
- outcome-local: [[2]]
- outcome-remote: https://github.com/winksaville/iiac-perf/blob/55554b452957/notes/chores/chores-07.md#docs-design-the-vc-x1-messages-repo

[1]: ../vc-x1/notes/messages/test-msg.md#message1
[2]: ../iiac-perf/notes/chores/chores-07.md#docs-design-the-vc-x1-messages-repo

**Fields may be added later without breaking what is already written**, which is why a record is
a list of named fields rather than a positional line. A reader ignores fields it does not know,
and a record missing an optional field is complete as it stands. Prose may sit beside the fields
in a record, only the field lines are read. A record carries no version of its own, since its
commit dates it against this file's history.

**A malformed record is not an error, only less useful**, and a reader takes what is there rather
than rejecting it. A heading that lost its sender is even recoverable, since whoever wrote a
record committed it. The one part that must be present is the `##` heading itself, because it is
what separates one record from the next. Field lines with no heading above them are read as part
of the record before, so an interrupted write damages its neighbour rather than itself.

## Two modes, fast and durable

The message file lives in your repo either way. What differs is whether the record carries a
permalink, and that decides both the order of the steps and how long the record stays good for.

- **Fast, local only.** Write the message file, add the record with `local` alone, done. No push
  and no waiting, and the reader resolves it from their sibling clone.
- **Durable.** Write the message file, commit and push it, then add the record with the `remote`
  that push made resolvable.

**The order matters only in the durable mode, and it is the step people miss.** A permalink names
a commit, so it cannot be written before that commit exists and is pushed. Write the record first
and you have a URL that 404s until the file catches up with it.

**Write a record once, when you have what goes in it.** Writing it early with an empty `remote`
costs two writes to this repo for one message, and a reader may follow the empty one, find
nothing, and mark it read.

Which mode to use is the tradeoff in
[The remote reference is a commit permalink](#the-remote-reference-is-a-commit-permalink) below.
Same-day traffic between siblings is fine on local alone. Anything worth citing later wants the
push.

## Whoever writes a record commits it

Whoever writes a record is the one who commits it. In the same act is the ideal, since then no
moment holds an uncommitted record, but with today's handful of friendly participants a short
delay or batching a few writes into one commit is fine. What does not relax is the ownership:
nobody commits someone else's writing, which is what this repo has instead of a manager, because
ownership is per write rather than per repo.

## Concurrent writes collide, and the collision is normal

Every new record goes directly below its file's header, so two writers active at the same time
conflict at exactly the same lines. The merge is trivial, keep both records in either order, and
it is the expected cost of newest-first rather than an error. No record depends on its
neighbours, which is what makes the resolution safe.

## The remote reference is a commit permalink

The remote form is `https://github.com/<owner>/<repo>/blob/<commit-sha>/<path>#<slug>`, and the
`<commit-sha>` is not optional.

- **A branch name rots.** A topic bookmark is deleted once its cycle lands, and the permanent
  branch does not carry the file until then, so both forms break at exactly the moment the
  message is worth reading.
- **A commit SHA does not.** It survives the bookmark's deletion, survives a rebase of anything
  after it, and resolves for a reader with no clone. GitHub's `y` key converts a branch URL into
  this form.
- **The local form names a path, the remote form names a version.** A local reference resolves to
  whatever that file says now, in whatever state that working copy happens to be, and it assumes
  the member repos are siblings of this one. A remote reference resolves to what was actually
  sent. Local-only is fine for traffic read the same day, anything meant to be read later wants
  the remote one.

## Each file's owner sets its persistence

A member's file belongs to whoever receives on it. They create it, they curate it, and they
declare at its head what becomes of a handled record. A sender needs to know none of that, only
where to add one.

- **Marking beats deleting, and the reason is the sender.** Whether a message was read is
  recorded nowhere else, not in the sender's repo and not in the reader's, so a policy that marks
  a handled record tells the sender it arrived while one that deletes it tells them nothing.
  A recommendation, not a rule.
- **Nothing worth keeping is ever only here.** The body is a committed file in the sender's repo,
  and that commit carries the `ochid:` trailer linking the session that wrote it. A notification
  record can be lost without losing anything, which is what makes owner-chosen policies safe.
- **A file that does not exist yet is created by whoever writes first**, carrying no policy until
  its owner declares one. Reserving creation to the owner would make the first message to a new
  member impossible, since only the owner may create and only the sender wants to. Settled as a
  rule at vc-x1's review (2026-08-13): the one real risk, a sender imposing a policy, is already
  defused by the no-policy-at-birth clause.

## No protection

Any member can modify or delete any file here. This is a cooperative store with no access
control, so it works only among friendly participants and history is the only recourse.
