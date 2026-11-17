---
name: spec-driven
description: Runs a spec-driven development loop in any repo. It writes a constitution once, then plans, implements, reviews, validates, and replans each feature. Use it when the user asks to write or update the constitution, plan the next feature, implement, review, or validate a feature, or replan after a merge.
argument-hint: "<mode> [feature]"
---

# Spec-driven workflow

This skill runs the spec-driven development loop. The user calls it as `/spec-driven <mode> [feature]`. The six modes are listed below in the order they are used.

- The `constitution` mode runs once at the start. It writes specs/mission.md, specs/tech-stack.md, and specs/roadmap.md, and CLAUDE.md if the repo has no agent instructions file.
- The `plan` mode starts each feature. It makes a feature branch and writes plan.md, requirements.md, and validation.md for that feature.
- The `implement` mode runs after the user approves the plan. It writes code and tests on the feature branch.
- The `review` mode runs after implement and before validate. One reviewer looks for faults in the feature's code, and five voters check whether each finding is true.
- The `validate` mode runs after review. It reports a pass or a fail for each check in validation.md.
- The `replan` mode runs after the user merges a feature. It updates the roadmap, the other constitution files if needed, and the changelog.

If the user gives no mode, look at which files exist in specs/ and at the git state, say which mode comes next, and ask before you start. If specs/roadmap.md does not exist, the next mode is constitution.

## Rules for every mode

1. Read CLAUDE.md or AGENTS.md and the files in specs/ before you do anything else, if they exist. Follow any project rules they state, such as code the user writes by hand or folders the agent must leave alone.
2. Treat the files in specs/ as the source of truth for what to build.
3. Stop if the specs do not answer a question the task depends on. Tell the user the gap, and wait.
4. Do not add scope beyond the project documents and the files in specs/. If something seems useful and nothing asks for it, ask the user first.
5. Do not commit. The user reviews, commits, and merges. When a mode ends, list the files you changed and tell the user to review and commit them. If the user asks you to commit, do not add trailers, co-author lines, or any other mark of AI use.
6. Run one mode per session. When a mode ends, tell the user to clear the session before the next mode, so each step starts with fresh context.
7. Write every file in plain English with full sentences, including in lists. Do not use em dashes or en dashes. Keep each file short.
8. Ask one question at a time and wait for the answer before you ask the next one.
9. Keep replies short and plain. Do not repeat file contents the user can open, do not restate the user's answers, and do not summarise what you are about to do.
10. Say what an item is each time you refer to it by a label or a number. This covers test cases, checks, findings, requirements, plan steps, and features. The reader does not have the other file open, so a label alone tells them nothing. For example, write "case E4, which checks that one switch press picks the highlighted column" in place of "case E4". This holds in replies and in every file, even when another file defines the label.
11. To find the default branch, run `git symbolic-ref --short refs/remotes/origin/HEAD` and drop the `origin/` prefix from the result. If the command fails, use `main` or `master` if exactly one of them is a local branch. If you still cannot tell, ask the user.

## Mode: constitution

1. Ask the user for the documents that describe the project, such as a brief, a requirements list, or a README. Read each one in full, including any pictures in it. If the repo already has code, read enough of it to learn what the project does and how it is built.
2. Read any files that already exist in specs/. If the constitution files exist, update them and keep what is still true.
3. Ask the user whether you draft the files or write an outline of headings and questions for the user to fill in. If the user picks the outline, do steps 5 to 8 as outlines and skip step 4.
4. Ask the user about each thing the files need that the documents and the code do not answer, one question at a time. The usual gaps are why the project exists and who uses it, the tool and library versions, the test tools, and the order of features.
5. Write specs/mission.md. It says in a few sentences why the project exists, who uses it, and what the system does.
6. Write specs/tech-stack.md. It lists the languages, frameworks, versions, test tools, and where the system runs.
7. Write specs/roadmap.md. It is a numbered list of features. Each feature has a one-sentence goal, the requirements it covers, and a status of planned, in progress, or done.
8. If the repo has neither CLAUDE.md nor AGENTS.md, write a short CLAUDE.md. It points at specs/ and at the project documents, and it says that the user commits and merges.
9. Check that every requirement in the project documents maps to at least one feature. List any that has no feature.
10. Show the files to the user. Tell the user to commit them on the default branch before the first plan, because plan mode needs a clean working tree. Then stop.

## Mode: plan

1. Read the constitution files. Use the feature the user names. If the user names none, use the first feature with status planned, and confirm it with the user.
2. Check that the working tree is clean. If it is not clean, stop and tell the user which files need a commit.
3. Make a branch named `feature/NN-slug` from the local default branch, where NN is the two-digit roadmap number.
4. Make the folder specs/features/NN-slug/ and write three files in it.
   - requirements.md lists what this feature must do, with a reference to the requirement in the project documents or the constitution for each item. It then lists anything the feature needs that neither source covers, such as file names. Mark each such item and ask the user about it.
   - plan.md lists the steps in order. Each step names the files it creates or changes.
   - validation.md lists the checks that prove the feature works. Each automated check lists its test cases with input and expected result, and gives the command that runs it. Each manual check gives exact steps and the expected result. Every worked example in the project documents that this feature covers becomes a test case.
5. Set the feature status to in progress in specs/roadmap.md.
6. Show the files to the user and stop. Do not start implement until the user approves the plan.

## Mode: implement

1. Check that the checkout is on the branch for this feature. If it is on another branch, stop and tell the user.
2. Read the feature folder. Ask the user to confirm the plan is approved. If it is not, stop.
3. Do the plan steps in order. Change only the files the plan names.
4. Write the automated tests from validation.md with the cases as listed. Do not add, remove, or change a case without asking.
5. Run the tests and the linters, and fix the failures.
6. Report the steps you did, the test results, and anything that differs from the plan. Tell the user the next mode is review. Then stop.

## Mode: review

1. Check that the checkout is on the branch for this feature. If it is on another branch, stop and tell the user.
2. Read the feature folder. If the feature branch has no code changes against the default branch, stop and tell the user to run implement first.
3. Read the diff of the feature branch against the default branch, and read each changed file in full. Read the code around the changes where you need it to follow a call.
4. Review the changes as an opponent of the code. Look for wrong behaviour, cases the tests miss, and weak code practice or quality. Report only what is worth a change. Leave out style points, and leave out anything the specs chose on purpose.
5. Write specs/features/NN-slug/review.md. Number the findings. Each finding has a one-sentence claim, the file and line, a concrete case that shows the fault, a suggested fix, and a level of high, medium, or low. Say at the top what you ran and what you only read.
6. Start five voter subagents in one message, so they run at the same time. Use a smaller model than your own, such as sonnet. Use fresh subagents, not forks, so that no voter carries your reasoning. Give each voter the same brief, which holds the items below.
   - The brief gives the repo path, one sentence on what the project is and who uses it, the path to the specs, and the path to review.md.
   - The brief says that the reviewer can be wrong, and that the voter must open the cited files and check each claim against the code itself.
   - The brief asks whether each level fits this project, and whether the specs already cover or rule out the point.
   - The brief says that the voter works alone, changes no files, and builds nothing.
   - The brief asks for one line per finding with a vote of valid, partly, or invalid, the level the voter would give of high, medium, low, or none, and a confidence of high, medium, or low. Two to four sentences of evidence with file and line follow each line. The evidence says which claims the voter confirmed, which are wrong or overstated, and which it could not check.
   - The brief asks for the voter's own order of all findings from most to least important, and for a list of anything the review states as fact that is false.
7. Wait for all five voters. Do not tally early. Tally each finding as follows.
   - Keep the finding if at least three voters voted valid or partly. Drop it if at least three voted invalid.
   - Give the finding the middle value of the five levels the voters gave. The reviewer's level does not count.
   - Order the kept findings by that level, and within a level by the voters' average rank.
8. Add a Votes section with today's date at the end of review.md. It lists the kept findings in order, each with its vote count, its level, and any fact the voters corrected. It then lists the dropped findings, each with the reason the voters gave. Do not change the findings above it.
9. Show the ordered list to the user and ask which findings to fix. For each one the user picks, add a step to plan.md and a check to validation.md. Then tell the user to run implement again, and validate after that. If the user picks none, tell the user the next mode is validate. Do not fix any code in this mode.

## Mode: validate

1. Check that the checkout is on the branch for this feature. If it is on another branch, stop and tell the user.
2. Run each automated check in validation.md and record a pass or a fail with a short summary of the output.
3. For each manual check, give the user the exact steps and ask for the result.
4. Check that each item in requirements.md has at least one passing check. List any item that has none.
5. Add a Results section with today's date at the end of validation.md. Keep the Results sections from earlier runs above it.
6. The feature is ready only if every check passes and every requirement has at least one passing check. If it is ready, tell the user the feature is ready for commit and merge. If it is not ready, list each failure and each requirement with no passing check, and tell the user to run implement again or change the plan. Do not commit.

## Mode: replan

1. Check that the checkout is on the default branch. If it is on another branch, stop and tell the user.
2. Check that the feature is merged into the default branch. Run `git branch --merged` first. If git does not list the feature branch, the user may have used a squash merge or deleted the branch. In that case, ask the user to confirm the merge and to name the commit that holds the feature.
3. Set the feature status to done in specs/roadmap.md.
4. Run the `changelog` skill for the merged feature. Save the entries without waiting for approval, even though the `changelog` skill says to wait. Carry on with the next step here, and show the saved entries with the other changes at the end. If the `changelog` skill is not installed, tell the user and skip this step.
5. Compare the merged code and the feature folder with the constitution files. List each place where a constitution file no longer matches what was built.
6. Ask the user what else changed during the feature, such as drift from the plan or new findings.
7. Make small changes to the constitution files now, for the mismatches you listed and for what the user reports. Add large changes to the roadmap as new features.
8. If a change goes against one of the project documents, stop and tell the user. The user decides.
9. Ask the user whether a prompt they repeat should become a skill. Do not make the skill without approval.
10. Show the changes and stop.
