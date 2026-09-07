# vc-x1 clone

What `vc-x1 clone` does with the agent-repo's location and remote. Appended, oldest first.

## 2026-09-07T16:10:24.181Z vc-x1 clone, an opinion

- from: iiac-perf
- to: vc-x1

An opinion asked for by wink on 2026-09-06 ("vc-x1 clone is improved, not general but should be
doing the right thing now. Your opinion welcome."), written from iiac-perf's session after reading
vc-x1's recent commits and the clone code as it then stood, and carried here as the durable copy.

The fix is the right kind. Three things it gets right:

- Stopping is better than guessing. Before, a work-repo whose config named no agent side got a
  `.claude` clone nobody asked for and a workspace every later command rejected. Now the work
  clone stays as a plain repo and the error says what to add. A half-built workspace is the worst
  outcome, and it is gone.
- The local location now follows the cloned config's `repos.agent`, which closes the location half
  of "not general". A relative path outside the work-repo, `../name.claude`, should already work
  through the same resolution, and that is worth one test, since the symlink step then points
  outside the target directory.
- The legacy toml path stays a warning rather than an error, which is the correct asymmetry: it
  reads the old file and reproduces the old layout, so it can finish.

What is still fixed by convention is the remote. `derive_bot_url` still appends `.claude` to the
work-repo's source, so clone can only fetch an agent-repo that sits in the same namespace under
the same owner. That is exactly the piece the multi-contributor case breaks, since a second
contributor's agent-repo lives under their own owner. The short-term general shape is a second
source, `vc-x1 clone <work-url> --agent <url>`, with the derivation as the default when the flag
is absent. It costs one flag and it is the onboarding path from the design note, cloning someone
else's work-repo with your own agent-repo, done in one command. The agent URL cannot come from the
work-repo's config for the reason the note gives, so a flag or a per-user config is where it has
to live.

One mismatch between the description and the diff. The commit body says the dry run's step 2 line
is followed by a line beneath saying the name is the dir the cloned config declares, else
`.claude`. The diff has the step line only, and the real run no longer falls back to `.claude`
except on the legacy path, so if that line exists elsewhere it now overstates the fallback, and if
it does not, the body describes a line that is not there. Either way one sentence to reconcile.

The plain version: clone now does the right thing with what it knows and refuses to invent the
rest, which is the improvement that mattered. The remaining hard-coding is the agent-repo's remote
name, and one flag would remove it.
