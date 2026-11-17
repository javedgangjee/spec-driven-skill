# spec-driven

This repo holds a small spec-driven workflow for Claude Code. It works in any repo. The agent writes the specs, the code, and the tests one feature at a time, and you review and commit each step. You can also keep part of the code for yourself with a rule in `CLAUDE.md`.

The workflow is two skills. Each skill is one short Markdown file, and you install them by copying two folders.

## What it does

![Diagram of the spec-driven workflow. The context goes into the constitution, each feature runs through a loop of plan, implement, review, and validate, and a replan step leads back to the constitution.](docs/workflow.svg)

The context goes into the constitution. Each feature then runs through the loop, and the implement step adds to the code. The review step can send the feature back to implement for fixes. After each merge, the replan step updates the constitution before the next feature starts. The green boxes are your steps. The skill has no mode for them.

## How you use it

You run one mode at a time with `/spec-driven <mode>`. Clear the session before each mode, so that each step starts with fresh context.

1. Run `/spec-driven constitution`. The agent reads your project documents and writes a mission, a tech stack, and a numbered roadmap into `specs/`. It also writes a short `CLAUDE.md` if the repo has neither `CLAUDE.md` nor `AGENTS.md`. Review the files and commit them.
2. Run `/spec-driven plan`. The agent makes a feature branch and writes the requirements, the plan, and the validation checks for one feature. Review the three files, and commit them when you approve the plan.
3. Run `/spec-driven implement`. The agent writes the code and the tests in the plan. Review the code.
4. Run `/spec-driven review`. The agent looks for faults in the feature's code and writes them to `review.md`. Five smaller agents then vote on whether each finding is true. You pick the findings to fix, and you run implement again if you pick any.
5. Run `/spec-driven validate`. The agent runs every check and records a pass or a fail. Commit and merge the feature when it is ready.
6. Run `/spec-driven replan` on the default branch. The agent marks the feature done, updates the changelog, and fixes any spec file that no longer matches the code. Commit the changes.
7. Repeat steps 2 to 6 for each feature on the roadmap.

If you run `/spec-driven` with no mode, the skill looks at `specs/` and the git state and tells you which mode comes next.

## Rules the workflow follows

- The agent follows the project rules in `CLAUDE.md` or `AGENTS.md`.
- Each mode runs in its own session.
- The agent does not add scope beyond your project documents and the files in `specs/`. It asks first.
- The agent lists every requirement in your project documents that no roadmap feature covers.
- A feature is ready only when every check passes and every requirement has at least one passing check.
- Every worked example in your project documents becomes a test case.
- The agent says what a test case, a finding, or a step is each time it names one by its label.
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

You can type `/spec-driven`, or ask for a step in plain words, such as "plan the next feature".

The `replan` mode calls the `changelog` skill. Keep both skills together. If the changelog skill is missing, `replan` skips that step and tells you.

## Project rules in CLAUDE.md

The agent reads `CLAUDE.md` or `AGENTS.md` at the start of every mode and follows the rules it finds there. Two rules are useful with this workflow.

- You can list the code that you write by hand. The agent leaves that code to you.
- You can list the folders that you own, such as `docs/`. The agent does not write in them.

If you ask the agent to commit, it adds no trailers, no co-author lines, and no other mark of AI use.

You can also add your own writing rules to `CLAUDE.md`. The agent follows them in every file it writes.

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
      review.md
      validation.md
```

## Credit

This workflow is a personal take on the method from the DeepLearning.AI short course [Spec-Driven Development with Coding Agents](https://www.deeplearning.ai/short-courses/spec-driven-development-with-coding-agents/), which was built with JetBrains and is taught by Paul Everitt. The constitution and the feature loop follow that course.

## License

This project uses the MIT License. See `LICENSE`.
