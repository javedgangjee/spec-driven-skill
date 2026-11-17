---
name: changelog
description: Drafts CHANGELOG.md entries from merged features and git history. Use it when the user asks to update the changelog, after a feature is merged, or when the user closes a release.
argument-hint: "[feature or release name]"
---

# Changelog

This skill keeps CHANGELOG.md at the repo root. The user calls it as `/changelog [feature or release name]`. Each entry says what changed for a person who reads or runs the project. The file has an Unreleased section at the top and one section for each closed release below it.

## Rules

1. Write only what the git history and the files in specs/ show. Do not guess at a change or at its reason.
2. Do not write in any folder that CLAUDE.md says the user owns. Do not commit. If the user asks you to commit, do not add trailers, co-author lines, or any other mark of AI use.
3. Write each entry as one full sentence in plain English. Follow the writing rules in CLAUDE.md if it has any.
4. Refer to a feature by its roadmap number.
5. Show new entries to the user and wait for approval before you save them.

## Add entries for a merged feature

1. Read CHANGELOG.md. If it does not exist, create it with a title, one sentence that says what the file records, and an empty Unreleased section.
2. Use the feature the user names. If the user names none, use the feature that was merged most recently, and confirm it with the user. Read the feature's folder in specs/features/, including the latest Results section of validation.md.
3. Find the commits on the default branch that belong to the feature. The way to find them depends on how the user merged.
   - A merge commit holds the feature in the range from the merge base to the merge commit.
   - A squash merge holds the feature in one commit.
   - A fast-forward merge holds the feature in the run of commits that came from the feature branch.
   If you cannot tell which commits belong to the feature, ask the user for the first and the last commit.
4. Read those commits with `git log` and the changed files with `git diff --stat`.
5. Put each change under one heading in Unreleased. The headings are Added, Changed, Fixed, and Removed. Leave out a heading that has no entries.
6. If a commit does not belong to any feature spec, show it to the user and ask whether it goes in the changelog.
7. Show the entries, save them after approval, and stop.

## Close a release

1. Ask the user for the release name or version and the date.
2. Move the entries under Unreleased into a new section headed with the release name and the date, for example `## Release 1 (2026-09-28)`. Leave an empty Unreleased section above it.
3. Ask the user for one sentence that says what the release delivered, and put it under the release heading.
4. Show the result, save it after approval, and stop.
