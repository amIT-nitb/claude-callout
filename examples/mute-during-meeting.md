# Mute during a meeting

**Goal:** silence both voice and OS notifications for a fixed window — say, a 30-min standup — and have things auto-restore afterward, no manual cleanup needed.

## Use case

You're heading into a meeting and Claude Code is running a long task in another terminal. You don't want the banners popping up during your call, but you also don't want to forget to turn things back on after.

## Commands

From inside Claude Code:

```
/voice-mute 30m --global
```

That's it. Both voice and OS notifications go silent **user-wide** for 30 minutes — every Claude Code session on this machine stays quiet. After the window expires, the mute file is auto-cleaned the next time a hook fires, so there's no "did I forget to turn it back on?" failure mode.

## Variations

| Need | Command |
|---|---|
| Mute everywhere for 2 hours | `/voice-mute 2h --global` |
| Mute only the current project | `/voice-mute 30m` (project scope is the default when `--global` is omitted) |
| Cancel early (meeting ended sooner) | `/voice-unmute --global` (or `/voice-unmute` for project scope) |
| Mute via shell instead of slash command | `claude-callout mute 30m --global` (shell — requires symlink or use full path, see [README](../README.md#enable)) |

## Verifying

While the mute is active, `/voice-status` will show:

```
Voice:               off (muted (user))
Notifications:       off (muted (user))
…
Mute:                user scope, 28m 12s remaining
```

After the window expires, run `/voice-status` again — the Mute row no longer shows and Voice/Notifications report their normal values. The `mute-until` file itself is removed the next time a hook fires (Stop, Notification, etc.), not at the `/voice-status` check.

## How it works under the hood

A single-line file at `~/.claude/callout/mute-until` (or `<project>/.claude-callout/mute-until`) holds an epoch-seconds expiry. Every gating check compares `now` against that timestamp. Expired files are removed on read.
