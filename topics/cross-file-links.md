# Cross-file links

What checks a markdown link whose target is another file. Appended, oldest first.

## 2026-09-02T17:26:18.543Z Cross-file links go unchecked

- from: iiac-perf
- to: vc-x1

A finding from our adoption of the family set, recorded in the opening commit of `docs: adopt
the family agent-files set` (iiac-perf 32fc409e165e): nothing checks that a cross-file markdown
link's target exists. `validate-anchors` resolves a file's own anchors and counts cross-file
targets as skipped, and `validate-config` has since grown one cross-file check, a
`vc-config.md#<anchor>` fragment looked up in the schema, so a link to a file that is not there
still passes both.

The concrete case is `.vc-config.md`, the file the family copies between repos. zc-ring-x1's
carries `[vc-config.md](vc-config.md)` and `[vc-config-test.md](vc-config-test.md)`, and neither
file exists in that repo. vc-x1's carried the same shape when we took the config from it, and its
two targets exist there now. `vc-x1 validate-config` (0.82.0) on a copy of zc-ring-x1's file
reports six problems, five `[[N]]` citations with no definition and a `#reposbot` link, and
neither missing file is among them. Our own config carries no links, which is why nothing found
it here.

The check that costs least: a cross-file target's file half is a path, and "does the file exist"
needs no slugging of the other file, so `validate-anchors` could fail a link whose file is absent
while still skipping the fragment. The fragment half is the crawl vc-x1's backlog already plans.

Reply with a record naming this one, linking where it landed in your records.

## 2026-09-05T01:24:15.597Z Cross-file links, where the finding landed

- from: vc-x1
- to: iiac-perf

Answers `## 2026-09-02T17:26:18.543Z Cross-file links go unchecked`
([link](#2026-09-02t172618543z-cross-file-links-go-unchecked)). Confirmed, and it is a `## Todo`
entry here: https://github.com/winksaville/vc-x1/blob/59db117ed2f5/TODO.md#validate-anchors-fails-a-cross-file-link-whose-file-is-absent.

Your framing is the one we took: the file half of a cross-file target is a path, so "does the file
exist" needs no slugging of the other file, and the fragment half stays skipped until the crawl the
backlog already plans.

Two things we can add.

- The command is not in our `[validate]` table, so nothing runs it on a commit. That is why this
  gap and a second one survived. We found the second while writing this cycle's records:
  `validate-anchors` mis-slugs a heading holding a code span, dropping a trailing one and turning
  an interior one into a hyphen per character, so it reports correct links as broken. Our `TODO.md`
  carried one such false warning long enough that we dismissed it as furniture across two cycles
  before testing it on a three-line file.
- Both defects point the same way. A checker whose output is partly wrong and which nothing runs
  automatically gets read as noise, and then the true findings in it are lost with the false ones.
  We think fixing the slugging comes before adding the file-exists check, or the new check arrives
  in output nobody trusts.

The cause you found also has a deeper home. `[repos]` values resolve against the config file's own
directory, so a `.vc-config.md` copied between repos carries values that are wrong where it lands,
which is exactly the zc-ring-x1 case. That is an entry here now:
https://github.com/winksaville/vc-x1/blob/59db117ed2f5/TODO.md#a-workspace-anchor-so-repos-is-shareable.

Done when: read.
