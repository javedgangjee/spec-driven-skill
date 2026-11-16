---
name: changelog
description: Drafts CHANGELOG.md entries from merged features and git history. Use it when the user asks to update the changelog, after a feature is merged, or when the user closes a build.
---

# Changelog

This skill keeps CHANGELOG.md at the repo root. The user calls it as `/changelog [feature or build name]`. Each entry says what changed for a person who reads or runs the project. The file has an Unreleased section at the top and one section for each closed build below it.

## Rules

1. Write only what the git history and the files in specs/ show. Do not guess at a change or at its reason.
2. Do not write in any folder that CLAUDE.md says the user owns. Do not commit. If the user asks you to commit, follow the commit rule in CLAUDE.md.
3. Write each entry as one full sentence in plain English. Do not use em dashes or en dashes.
4. Refer to a feature by its roadmap number. If CLAUDE.md names a spec document with numbered sections, refer to a rule by its section number and do not copy the rule text.
5. Show new entries to the user and wait for approval before you save them.

## Add entries for a merged feature

1. Read CHANGELOG.md. If it does not exist, create it with a title, one sentence that says what the file records, and an empty Unreleased section.
2. Use the feature the user names. If the user names none, use the most recent merge into the default branch. Read the feature's folder in specs/features/, including the Results section of validation.md.
3. Read the commits in the merge with `git log` and the changed files with `git diff --stat`, from the merge base to the merge commit.
4. Put each change under one heading in Unreleased. The headings are Added, Changed, Fixed, and Removed. Leave out a heading that has no entries.
5. If the spec document named in CLAUDE.md changed in the merge, add an entry under Changed that names the section. Do not write the reason for the change.
6. If a commit does not belong to any feature spec, show it to the user and ask whether it goes in the changelog.
7. Show the entries, save them after approval, and stop.

## Close a build

1. Ask the user for the build name and the date.
2. Move the entries under Unreleased into a new section headed with the build name and the date, for example `## Build 1 (2026-09-28)`. Leave an empty Unreleased section above it.
3. Ask the user for one sentence that says what the build delivered, and put it under the build heading.
4. Show the result, save it after approval, and stop.
