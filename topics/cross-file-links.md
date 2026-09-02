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
