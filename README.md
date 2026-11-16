# spec-driven

This repo holds a small spec-driven workflow for Claude Code. It is made for projects where you write some of the code yourself and the agent writes the rest. Take-home projects, interview prep, and learning a new codebase are good fits.

The workflow is two skills. Each skill is one short Markdown file, and you install them by copying two folders.

## What it does

![Diagram of the spec-driven workflow. The context goes into the constitution, each feature runs through a loop of plan, implement, review, and validate, and a replan step leads back to the constitution.](docs/workflow.svg)

The context goes into the constitution. Each feature then runs through the loop and adds to the code. After each merge, the replan step updates the constitution before the next feature starts. The green boxes are your steps. The skill has no mode for them.

## How you use it

You run one mode at a time with `/spec-driven <mode>`. Clear the session before each mode, so that each step starts with fresh context.

1. Run `/spec-driven constitution`. The agent reads your project brief and writes a mission, a tech stack, and a numbered roadmap into `specs/`. It also writes a short `CLAUDE.md` if you have none. Review the files and commit them.
2. Run `/spec-driven plan`. The agent makes a feature branch and writes the requirements, the plan, and the validation checks for one feature. Review the three files, and commit them when you approve the plan.
3. Run `/spec-driven implement`. The agent writes the code and the tests that the plan gives to it. It skips the steps that you own. Write your part and review all the code.
4. Run `/spec-driven validate`. The agent runs every check and records a pass or a fail. Commit and merge the feature when it is ready.
5. Run `/spec-driven replan` on the default branch. The agent marks the feature done, updates the changelog, and fixes any spec file that no longer matches the code. Commit the changes.
6. Repeat steps 2 to 5 for each feature on the roadmap.

If you run `/spec-driven` with no mode, the skill looks at `specs/` and the git state and tells you which mode comes next.

## Rules the workflow follows

- You tell `CLAUDE.md` which code you write by hand, and the agent does not write that code.
- Each mode runs in its own session.
- The agent lists every requirement in the brief that no roadmap feature covers.
- A feature is ready only when every check passes and every requirement has at least one passing check.
- Every worked example in the brief becomes a test case.
- The agent does not commit or merge unless you ask it to.

GitHub Spec Kit and similar tools cover more ground and come with more tooling. This workflow is smaller.

## Install

You need git and Claude Code.

Run these commands to add the skills to one project. Replace `your-project` with the path to your project.

```sh
git clone https://github.com/javedgangjee/spec-driven-skill.git
mkdir -p your-project/.claude/skills
cp -R spec-driven-skill/.claude/skills/. your-project/.claude/skills/
```

Your project then has `.claude/skills/spec-driven/` and `.claude/skills/changelog/`. The commands leave any other skills in that folder as they are.

To use the skills in all your projects, copy them into your home folder instead.

```sh
mkdir -p ~/.claude/skills
cp -R spec-driven-skill/.claude/skills/. ~/.claude/skills/
```

The `spec-driven` skill runs only when you type `/spec-driven`. It does not start by itself, because the plan mode makes a branch and writes files.

The `replan` mode calls the `changelog` skill. Keep both skills together. If the changelog skill is missing, `replan` skips that step and tells you.

## Settings in CLAUDE.md

The `constitution` mode asks you three questions and records the answers in `CLAUDE.md`. If you already have a `CLAUDE.md`, it adds only the answers that are missing. You can also edit them by hand.

- You list the code that you write by hand.
- You list the folders that you own, such as `docs/`. The agent does not write in them.
- You state how commits must be marked. Some teams and schools require you to disclose AI help, and others do not. The agent follows what you write here.

You can also add your own writing rules to `CLAUDE.md`, such as a rule against dashes. The agent follows them in every file it writes.

## Files the workflow creates

```text
CLAUDE.md
CHANGELOG.md
specs/
  mission.md
  tech-stack.md
  roadmap.md
  features/
    01-example/
      requirements.md
      plan.md
      validation.md
```

## Credit

This workflow is a personal take on the method from the DeepLearning.AI short course [Spec-Driven Development with Coding Agents](https://www.deeplearning.ai/short-courses/spec-driven-development-with-coding-agents/), which was built with JetBrains and is taught by Paul Everitt. The constitution and the feature loop follow that course.

## License

This project uses the MIT License. See `LICENSE`.
