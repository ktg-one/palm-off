# PLAN — palm-off

## Outcome

One idea in, four artifacts out: plan, private GitHub repo, one daily growth cron, one weekly review reminder.

## Non-goals

- Not a multi-agent build system.
- Not hourly checks.
- Not a public launch.
- Not a second reminder channel.

## First user

ktg-one, palming an idea out of chat.

## Growth lever

Usage. The project grows when a second real idea is run through palm-off and the repo URL comes back.

## 7-day sequence

| Day | One action |
|---|---|
| 1 | Confirm the skill file, repo URL, daily task id, and weekly task id are live. |
| 2 | Run palm-off on one real idea that is not this repo. |
| 3 | Append that run's result to `docs/GROWTH.md`. |
| 4 | Cut any step that asked a question the slug could have answered. |
| 5 | Add one failure row from the real run to the skill failure table. |
| 6 | Leave the weekly review prompt pointed at that second repo. |
| 7 | Stop. Do not add a third cron. |

## Repo layout

- `SKILL.md` — operator contract
- `PLAN.md` — this file
- `README.md` — what it is
- `docs/GROWTH.md` — daily log
- `docs/REVIEW.md` — weekly reminder log
- `evals/` — trigger and behavior checks

## Open questions

- Stack: unknown. Skill is markdown plus GitHub and Automations connectors.
- Launch date: unknown.
- Public vs private: private until the user says public.

## Done-check

Pass only if `PLAN.md` is on the default branch, exactly one daily automation is enabled, exactly one weekly automation is enabled, and the reply contains the repo URL.

## First daily action

Day 1: confirm skill file, repo URL, daily task id, and weekly task id are live. Do not pick a second action.
