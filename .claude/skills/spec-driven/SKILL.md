---
name: spec-driven
description: Runs the spec-driven development loop for this repo, from the constitution through plan, implement, validate, and replan. Use it when the user asks to write or update the constitution, plan the next feature, implement or validate a feature, or replan after a merge.
argument-hint: "<mode> [feature]"
disable-model-invocation: true
---

# Spec-driven workflow

This skill runs the spec-driven development loop. The user calls it as `/spec-driven <mode> [feature]`. The five modes are listed below in the order they are used.

- The `constitution` mode runs once at the start. It writes specs/mission.md, specs/tech-stack.md, and specs/roadmap.md. It also writes CLAUDE.md, or adds the missing settings to a CLAUDE.md that exists.
- The `plan` mode starts each feature. It makes a feature branch and writes plan.md, requirements.md, and validation.md for that feature.
- The `implement` mode runs after the user approves the plan. It writes code and tests on the feature branch.
- The `validate` mode runs after implement. It reports a pass or a fail for each check in validation.md.
- The `replan` mode runs after the user merges a feature. It updates the roadmap, the other constitution files if needed, and the changelog.

If the user gives no mode, look at which files exist in specs/ and at the git state, say which mode comes next, and ask before you start. If specs/roadmap.md does not exist, the next mode is constitution.

## Rules for every mode

1. Read CLAUDE.md and the files in specs/ before you do anything else, if they exist.
2. Stop if neither the brief nor the spec files answer a question the task depends on. Tell the user the gap, and wait.
3. Do not add scope beyond the brief and the spec files. If something seems useful and nothing asks for it, ask the user first.
4. Do not write code that CLAUDE.md says the user writes by hand. Write tests or empty stubs for that code only when plan.md says so.
5. Do not commit. The user commits and merges. When a mode ends, list the files you changed and tell the user to review and commit them. If the user asks you to commit, follow the commit rule in CLAUDE.md.
6. Run one mode per session. When a mode ends, tell the user to clear the session before the next mode, so each step starts with fresh context.
7. Do not write in any folder that CLAUDE.md says the user owns.
8. CLAUDE.md holds three settings. They are the code the user writes by hand, the folders the user owns, and the commit rule. If a rule needs a setting that CLAUDE.md does not have, ask the user. Then offer to add the answer to CLAUDE.md.
9. Write every file in plain English and keep each file short. Follow the writing rules in CLAUDE.md if it has any.
10. To find the default branch, run `git symbolic-ref --short refs/remotes/origin/HEAD` and drop the `origin/` prefix from the result. If the command fails, use `main` or `master` if exactly one of them is a local branch. If you still cannot tell, ask the user.

## Mode: constitution

1. Ask the user for the path to the project brief and to any other requirement documents, unless CLAUDE.md names them. Read each one in full, including any pictures in it.
2. Read any files that already exist in specs/. If the constitution files exist, update them and keep what is still true.
3. Ask the user whether you draft the files or write an outline of headings and questions for the user to fill in.
4. Ask the user, in one message, about each thing the files need that the brief does not answer. The usual gaps are why the project exists and who uses it, the tool and library versions, the test tools, and the order of features.
5. Write specs/mission.md. It says in a few sentences why the project exists, who uses it, and what the system does.
6. Write specs/tech-stack.md. It lists the languages, frameworks, versions, test tools, and where the system runs.
7. Write specs/roadmap.md. It is a numbered list of features. Each feature has a one-sentence goal, the brief requirements it covers, and a status of planned, in progress, or done.
8. Check CLAUDE.md for the three settings in rule 8. Ask the user about each setting that is missing. If CLAUDE.md does not exist, write a short one that points at specs/, names the brief, and records the three answers. If CLAUDE.md exists, add only the missing settings and change nothing else in it.
9. Check that every requirement in the brief maps to at least one feature. List any that has no feature.
10. Show the files to the user. Tell the user to commit them on the default branch before the first plan, because plan mode needs a clean working tree. Then stop.

## Mode: plan

1. Read the constitution files. Use the feature the user names. If the user names none, use the first feature with status planned, and confirm it with the user.
2. Check that the working tree is clean. If it is not clean, stop and tell the user which files need a commit.
3. Make a branch named `feature/NN-slug` from the local default branch, where NN is the two-digit roadmap number.
4. Make the folder specs/features/NN-slug/ and write three files in it.
   - requirements.md lists what this feature must do, with a reference to the brief requirement for each item.
   - plan.md lists the steps in order. Each step names the files it creates or changes and says whether the agent or the user writes it, based on CLAUDE.md.
   - validation.md lists the checks that prove the feature works. Each automated check lists its test cases with input and expected result, and gives the command that runs it. Each manual check gives exact steps and the expected result. Every worked example in the brief that this feature covers becomes a test case.
5. Set the feature status to in progress in specs/roadmap.md.
6. Show the files to the user and stop. Do not start implement until the user approves the plan.

## Mode: implement

1. Check that the checkout is on the branch for this feature. If it is on another branch, stop and tell the user.
2. Read the feature folder. Ask the user to confirm the plan is approved. If it is not, stop.
3. Do the plan steps in order. Change only the files the plan names. Skip the steps the user owns and list them in your report.
4. Write the automated tests from validation.md with the cases as listed. Do not add, remove, or change a case without asking.
5. Run the tests and the linters. Fix failures in code you wrote. If a failure comes from code the user wrote, report it and do not fix it.
6. Report the steps you did, the steps left for the user, and the test results. Then stop.

## Mode: validate

1. Check that the checkout is on the branch for this feature. If it is on another branch, stop and tell the user.
2. Run each automated check in validation.md and record a pass or a fail with a short summary of the output.
3. For each manual check, give the user the exact steps and ask for the result.
4. Check that each item in requirements.md has at least one passing check. List any item that has none.
5. Add a Results section with today's date at the end of validation.md. Keep the Results sections from earlier runs above it.
6. The feature is ready only if every check passes and every requirement has at least one passing check. If it is ready, tell the user the feature is ready for review, commit, and merge. If it is not ready, list what is missing. Do not commit.

## Mode: replan

1. Check that the checkout is on the default branch. If it is on another branch, stop and tell the user.
2. Check that the feature is merged into the default branch. Run `git branch --merged` first. If git does not list the feature branch, the user may have used a squash merge or deleted the branch. In that case, ask the user to confirm the merge and to name the commit that holds the feature.
3. Set the feature status to done in specs/roadmap.md.
4. Run the changelog skill for the merged feature. If the changelog skill is not installed, tell the user and skip this step.
5. Compare the merged code and the feature folder with the constitution files. Where a constitution file no longer matches what was built, make a small change to that file now. Add a large change to the roadmap as a new feature.
6. Ask the user whether a prompt they repeat should become a skill. Do not make the skill without approval.
7. Show the changes and stop.
