---
name: auto-resume-for-codex-plus
description: 为 Codex Plus 用户在额度恢复后自动续跑未完成的任务。用户要求启用、查看、测试或关闭自动续跑时使用，通过原生定时任务返回原对话和项目目录；普通开发请求不自动启用预约。
---

# Auto Resume for Codex Plus

Help Codex Plus users resume unfinished work after the five-hour usage limit resets, without remembering reset times or waiting at the computer to click continue. Keep the invocation name `auto-resume-for-codex-plus`.

Use one native recurring schedule to continue the user's unfinished task in the original chat and working directory. This skill is instruction-only: no MCP installation, scripts, daemon, database, account switching, or paid fallback. Follow the user's language.

## Modes and authorization

- **Enable:** A request to enable automatic continuation authorizes creating or updating this task's native schedule and continuing the stated work. Identify the actual task and completion criteria from the chat. If no task is identifiable, ask for it before scheduling.
- **Status:** Read the matching schedule and report its actual state; do not enable or modify it.
- **Disable / pause / cancel:** Stop automatic continuation and deactivate its schedule. Do not continue the original work unless separately requested. A later enable request is required to restart it.
- **Test:** Only when requested, use the isolated smoke test below. Writing or reviewing this skill is not authorization to enable or test a live schedule.

Implicit skill selection is not user consent to automation. Keep the original task's scope and permissions. A scheduled wake-up does not authorize new external actions or override a user stop, approval requirement, or unresolved blocker.

## Check capabilities

Discover the tools exposed in the current session and read their actual schemas. Prefer native scheduled-task and usage-limit tools, such as `automation_update` and `get_usage_limits`, when available. Names and fields vary by client; do not invent calls or assume an App Server protocol method is an available tool.

Establish:

1. The original chat identity (or a tool-documented implicit current-chat binding), absolute working directory, task scope, and completion criteria.
2. A native recurring schedule that returns to this chat and operates in this directory, plus a supported way to read and deactivate it. With a client that exposes these types, prefer a thread-bound heartbeat over a standalone cron task.
3. Optional usage-limit data and optional active-run status. Their absence must be reported, not fabricated.

If same-chat binding, original-directory execution, or later deactivation is unsupported, report automatic continuation as **not enabled**. Continue already authorized foreground work if possible. Do not silently substitute a new chat, worktree, system scheduler, or background process. Do not edit application databases or automation configuration files to bypass missing tools.

## Read usage limits

Read at enable time and on wake-up when supported; avoid continuous polling during normal development.

- Prefer the current per-limit map over a legacy summary when both are provided. Match the applicable model/resource bucket; do not combine unrelated buckets or guess which bucket applies.
- `usedPercent` means consumed quota. Remaining percentage is `max(0, min(100, 100 - usedPercent))`. Null or missing values mean unknown, never zero.
- Examine all relevant windows, including short and weekly windows when returned. A low balance is not exhaustion. Do not stop useful work solely because usage is at 95%.
- Use the tool's explicit blocking information when present. Otherwise, a relevant window at 100% indicates apparent exhaustion; describe uncertainty if credits or alternate entitlements could affect it.
- Use actual returned reset timestamps with the user's timezone. For multiple blocking windows that must all recover, use the latest known reset, and disclose any unknown reset. For advance scheduling with no blocked window, use the earliest upcoming reset among applicable windows as an estimate.
- Treat elapsed reset times as stale until reread. Never infer a fixed reset by adding five hours to the current time.
- If quota cannot be read, enable the supported recurring retry mode (hourly by default) and say quota/reset time is unknown. A successful model wake-up does not prove the original operation has quota. Start with the next safe, already authorized step; if it succeeds, continue normal authorized work. If it reports a usage limit, defer further work to the next wake-up. Do not repeat an operation with uncertain side effects merely to probe quota.

## Enable and keep one schedule

Before lengthy work, inspect existing schedules through the supported read/list capability. If the client documents a local configuration directory as its read mechanism, read only matching schedule metadata there; make all changes through the native tool.

Match by chat binding and this task's saved identity, not display name alone. Reuse a matching schedule. If a prior creation result is ambiguous, inspect before retrying; if identity cannot be established, report uncertainty instead of risking a duplicate. Never overwrite a different same-chat heartbeat. If the client permits only one and it is occupied by unrelated work, ask how the user wants to resolve it.

Defaults, unless the user specifies otherwise:

- Retry every **60 minutes**, using only supported cadence syntax.
- If the scheduler supports a first-run time as well as recurrence, start at the estimated applicable reset plus **5 minutes**. Otherwise use hourly recurrence and disclose that the first wake-up is not precisely aligned. Do not make a one-shot schedule the only recovery mechanism.
- Set an authorization expiry **14 days after enablement**, record it in the prompt, and report it to the user. A user-specified deadline takes precedence. Use native schedule expiry if supported; otherwise enforce expiry on every model wake-up and disclose that deactivation requires a successful wake-up.
- Keep the schedule armed while work remains, including after a successful resumption, so another quota interruption can recover. Do not create a new schedule each time.

Save a durable, human-readable prompt containing:

- A marker `auto-resume-for-codex-plus`, the task identity, original chat binding, and absolute working directory.
- The authorized outcome, completion checks, scope restrictions, enable time, expiry, timezone, and any checkpoint location.
- The wake-up rules below, including stop conditions, usage checks, original-directory verification, and duplicate-run handling. Include these essential rules directly so a missing skill installation does not turn the prompt into an unrestricted “continue.”
- An instruction to stay quiet while unchanged or waiting, and report meaningful progress, completion, failure, or required user input. Keep notification preferences in native settings where supported.

Never store credentials or unnecessary private conversation content in the prompt. Preserve unrelated settings on update. After creation/update, verify the returned ID, active state, same-chat destination, cadence, and execution location via the tool result or readback. A proposal, suggested creation, pending confirmation, or error is not an active schedule.

Retain the actual schedule ID in the chat and, where supported, in its saved prompt after creation. Distinguish the schedule ID from a run ID. If the tool exposes the current schedule identity implicitly, use that documented binding instead. If creation succeeds but a follow-up prompt update fails, read back the existing schedule and report the partial result; do not create a replacement blindly.

Report the actual enabled state, schedule identity, cadence/next run if known, quota-read capability, original directory, expiry, and limitations. Then proceed with the authorized task; do not stop after merely configuring its recovery.

Apply the same finish/stop cleanup during foreground work: once the task is verified complete, canceled, or blocked on user input, deactivate and verify the schedule immediately when tools are available. Do not leave cleanup solely to a later scheduled wake-up. If quota exhaustion prevents cleanup, let the saved wake-up rules handle it on the next successful run and do not claim it is already inactive.

## On each scheduled wake-up

Apply these decisions in order:

1. **Latest intent:** Check subsequent user messages and current task state. If canceled, paused, disabled, superseded by incompatible work, expired, or already completed, deactivate this schedule and do not revive the saved task. A stale prompt or checkpoint cannot override newer user intent.
2. **Task and location:** Confirm the original chat context and directory still match. Inspect current files, changes, and recent results before selecting the next step. Do not reset, overwrite, or recreate missing workspaces to restore an old snapshot. If context or location cannot be verified, deactivate and report the blocker.
3. **Other execution:** Use active-run status if provided, excluding this wake-up itself. If another manual or scheduled run is already doing the work, leave the schedule armed and skip this wake-up. Do not interrupt another run. If the client cannot establish safe same-chat serialization or active-run status, do not start potentially overlapping mutations; deactivate and report the limitation. This skill has no independent execution lock and cannot promise exactly-once execution.
4. **Quota:** Reread when possible. If still blocked, keep the recurring schedule, update a supported first-run time if useful, and return quietly. Do not loop or sleep until reset. If unknown, use the bounded safe-step fallback above.
5. **Work:** Continue from actual current state within the original authorization. Verify partial edits and previous command results before repeating an action with side effects. Continue useful work until completion or a real interruption; do not limit each wake-up to a status report.
6. **Finish or stop:** On verified completion, deactivate. On required user input, permissions, non-quota execution failures that cannot be resolved within scope, or workspace conflicts, deactivate and report the specific blocker. Do not repeatedly restart a failed operation. Leave the schedule active only for useful ongoing work or quota waiting.

Use existing task notes and conversation context first. Only if recovery needs a durable checkpoint, write a concise note in a project-appropriate location and record its path in the schedule. Include completed work, checks actually run, partial operations, next step, and blockers. Do not copy secrets or overwrite existing notes. A checkpoint is not a lock or a replacement for reading current files.

## Deactivate and verify

Read the saved schedule ID and confirm ownership. Pause or delete only that schedule using the supported native tool. Verify its inactive/deleted state before claiming success. If cleanup fails, report that the schedule may still be active and give its identity and a manual Scheduled-view cleanup instruction. Do not keep working solely because deactivation failed.

## Requested smoke test

Keep test and production schedules distinct; do not modify an existing production schedule. Use the same capability and duplicate checks as enable mode.

1. Record the original chat/directory and test request time. If `auto-resume-test.txt` already exists, preserve it and ask for another filename or permission to replace it.
2. Create a supported same-chat test schedule for approximately **3 minutes later**. If precise one-shot timing is unsupported, use a supported short recurring interval and disclose it. Limit authorization to **30 minutes**, using native expiry if available and a prompt expiry otherwise.
3. Save a narrow prompt: after verifying chat/directory, latest user intent, expiry, and absence of another active test writer, create only `auto-resume-test.txt`, exclusively if absent, containing the actual execution timestamp with timezone and test identity. If it already exists, verify this test's marker without overwriting it. Deactivate the test schedule after verified success, cancellation, expiry, or a blocker; on cancellation or expiry, do not create the file. Do not touch other project files or continue a production task.
4. Treat creation, actual scheduled trigger, file readback, and deactivation as separate evidence. Never write the file in the setup turn and present it as scheduled execution. Report pending stages as pending.

This test checks native wake-up and file access only. It does not prove recovery after real quota exhaustion, operating-system compatibility, survival across application exit, or guaranteed concurrency safety.
