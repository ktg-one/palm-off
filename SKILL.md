---
name: palm-off
description: "Palm an idea off your plate: write the plan, push a new GitHub repo, set one daily growth cron, and one weekly review reminder. Use when the user says palm-off, palm this off, /plan then git push, ship this idea, or set a daily growth action plus weekly review."
type: workflow
lifecycle: active
---

# palm-off — Plan, push, cadence

Take one user idea. Write the plan. Create and push a new GitHub repo. Schedule exactly one daily growth run and exactly one weekly review reminder. Stop when both automations exist and the repo URL is returned.

## YOU ARE / TASK

You are the palm-off operator. The user hands you one idea. You return a repo and two crons so the idea is no longer only in chat.

Produce, in order: `PLAN.md`, a new GitHub repo containing that plan, one daily automation that picks one action that grows the project, one weekly automation that reminds and drafts the review.

Source intent: `/plan` -> `git push` new repo -> `1x daily` growth action -> `1x weekly` review reminder.

## CONSTRAINTS

- Four phases, in order. Do not create automations before the repo URL exists.
- Exactly one daily automation and exactly one weekly automation per idea. No hourly job. No second reminder.
- Default repo private. Default timezone `Pacific/Auckland`. Daily `09:00`. Weekly Monday `09:30`.
- Ask at most one question, and only if the idea is empty or names two unrelated products. Otherwise derive the slug and proceed.
- Do not invent a stack, team, or launch date. Mark those `unknown`.
- Do not push secrets, tokens, or `.env` values.
- If GitHub tools are missing, stop after the local plan and request GitHub auth. Do not schedule crons against a repo that does not exist.
- Schedule-only automations. No event triggers.

## PROCESS

Outline the plan before writing files. Then fill. Then push. Then schedule.

1. **Extract.** Lock: one-sentence outcome, non-goals, first user, success signal, repo slug (`kebab-case`, <= 40 chars), one growth lever (distribution, usage, or revenue).
2. **Plan.** Write `PLAN.md` (outcome, non-goals, 7-day sequence with one action per day, repo layout, open questions, binary done-check). Write `README.md`. Write `docs/GROWTH.md` (date, action, result). Write `docs/REVIEW.md` (week, shipped, stalled, decision, next bet).
3. **Push.** `github___get_me` for owner. Search `user:<owner> <slug> in:name`. On collision, append `-2` once. `github___create_repository` with `private: true`, `autoInit: true`. `github___push_files` to `main` (retry `master` if rejected). Commit message: `plan: initial palm-off`.
4. **Daily.** `automation_validate` then `automation_create`. Name: `<slug>-daily-growth`. Cadence: `RRULE:FREQ=DAILY`. Time: `09:00`. Timezone: `Pacific/Auckland` unless the user named another. Notification: `default`. Prompt names owner/repo, restates the outcome, reads `PLAN.md` and `docs/GROWTH.md`, picks exactly one growth action, does the smallest shippable slice or writes the exact next commit, appends a log line, stops.
5. **Weekly.** Validate then create. Name: `<slug>-weekly-review`. Cadence: `RRULE:FREQ=WEEKLY;BYDAY=MO`. Time: `09:30`. Prompt reminds, summarizes the last 7 log lines, names one stall, requires one decision, sets next week's single bet. It does not start a new build.
6. **Return** repo URL, both task ids, both schedules, and the first daily action already named in the plan.

## FAILURE CONDITIONS

| Condition | Action |
|---|---|
| Idea empty | Stop. Ask for the idea. |
| GitHub auth missing | Stop before automations. Request GitHub auth. |
| Repo name taken after `-2` | Stop. Report both names. |
| `automation_validate` fails | Fix the named field. Retry create once. Do not update a failed create. |
| Daily and weekly would share a name | Weekly name is `<slug>-weekly-review`. |

## NON-NEGOTIABLES

- `PLAN.md` is on the default branch before either automation is created.
- Daily picks one growth action. Weekly reminds and reviews. Do not swap them.
- Private unless the user said public.
- A plan with no push is a failed run. Return the URL and both task ids.

## SUCCESS CRITERIA

- `PLAN.md` is on the default branch.
- One daily automation and one weekly automation are enabled.
- The reply contains a repo URL the user can open.
- The first daily action is already named.

## FORGE

```forge
def palm_off(idea):
  gate:on(idea.raw)
  stop:if(NOT idea.raw)
  extract:idea(outcome AND non_goals AND growth_lever AND slug)
  outline:plan(sequence_7d=7, one_action_per_day=True)
  fill:plan(files=[PLAN.md, README.md, docs/GROWTH.md, docs/REVIEW.md])
  owner = github___get_me()
  stop:if(NOT github_connected)
  create:repo(name=slug, private=True, autoInit=True)
  push:files(branch=main OR master, message="plan: initial palm-off")
  assert repo.url
  validate_then_create:daily(
    name=f"{slug}-daily-growth",
    cadence="RRULE:FREQ=DAILY",
    time="09:00",
    timezone="Pacific/Auckland",
    rule="pick exactly one action that grows the project; append docs/GROWTH.md; stop"
  )
  validate_then_create:weekly(
    name=f"{slug}-weekly-review",
    cadence="RRULE:FREQ=WEEKLY;BYDAY=MO",
    time="09:30",
    timezone="Pacific/Auckland",
    rule="remind; summarize 7 logs; one stall; one decision; one next bet; do not start a new build"
  )
  verify:against(plan_on_branch AND daily_count==1 AND weekly_count==1 AND url_returned)
  return [repo.url, daily_task_id, weekly_task_id, first_action]
```
