# Auto Resume for Codex Plus

[简体中文](README.zh-CN.md)

Let Codex continue unfinished work after usage limits reset.

## Why this skill

Built for Codex Plus users who hit the five-hour usage limit and have to remember the reset time, stay at their computer, and manually click “continue.” Enable it ahead of time so Codex can pick up unfinished work on a scheduled wake-up after quota becomes available again.

## How it works

The skill sets up a native recurring schedule for the current chat. Each wake-up checks task status and available quota, then continues in the original chat and project directory. The schedule is deactivated when the task finishes or the user cancels it.

The default retry interval is 60 minutes. Where supported, the first wake-up is scheduled for five minutes after the estimated quota reset. If quota data is unavailable, it attempts continuation on the recurring schedule.

## Features

- **One file**: All continuation rules live in `SKILL.md`. Copy it to install.
- **No extra dependencies**: Uses Codex's own capabilities without an additional MCP server, daemon, or other tools.
- **Continue the original task**: Keeps the chat context and works in the original project directory.

## Usage

### Install with AI

Send this directly to Codex to read the repository and install the skill:

```text
Read this repository: https://github.com/feilaochub/Auto-Resume-for-Codex-Plus
Follow its instructions to install the auto-resume-for-codex-plus skill in my personal skills directory.
Preserve any existing skill with the same name, then tell me how to enable it.
```

### Manual installation

Copy the `auto-resume-for-codex-plus` folder from this repository to one of these locations:

| Scope | Destination file |
| --- | --- |
| Current project | `<project>/.agents/skills/auto-resume-for-codex-plus/SKILL.md` |
| Personal, macOS | `~/.agents/skills/auto-resume-for-codex-plus/SKILL.md` |
| Personal, Windows | `%USERPROFILE%\.agents\skills\auto-resume-for-codex-plus\SKILL.md` |

### Enable and disable

After installation, define the task in your project chat in the Codex desktop app, then simply say:

```text
Enable automatic continuation
```

Check status:

```text
$auto-resume-for-codex-plus status
```

Disable:

```text
$auto-resume-for-codex-plus disable automatic continuation
```

## Notes

- This project is for the Codex desktop app, primarily for Plus users. It is not intended for other AI assistants.
- Keep the computer on, Codex running, and the project directory accessible.
- The default retry interval is 60 minutes, so continuation may not happen immediately after reset. Authorization expires after 14 days unless you specify another deadline. Without native expiry, expiry is checked and the schedule deactivated on the next successful wake-up.
- Continuation modifies the original project directory. Disable it before manually continuing work to reduce the risk of overlapping edits.
