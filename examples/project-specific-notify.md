# Project-specific notifications only

**Goal:** keep things quiet across most of your work, but get **loud, persistent** alerts for one specific project — the one running long migrations or flaky tests where you really do want to be pinged.

## Use case

You have a dozen repos on your machine. Most don't need notifications — you're at the keyboard, you'll see the next prompt. But one repo is doing 20-minute test runs and you keep missing when Claude finishes. You want notifications **only** for that one repo.

## Commands

One-time setup. From inside Claude Code:

```
# Step 1: disable everything user-wide (quiet across all projects)
/voice-off --global
/notify-off --global
```

Then `cd` into the *one* repo you care about (shell):

```bash
cd ~/code/the-noisy-project
```

And inside Claude Code there:

```
/voice-on            # this project only
/notify-on           # this project only
```

Done. Every other CC session stays silent; this one repo speaks + banners. The project-specific state lives in `<project>/.claude-callout/` — see the "Sharing across collaborators" section below for what to do with that directory.

## What `/voice-status` shows in each scope

In a quiet project:

```
Voice:               off (user)
Notifications:       off (user)
…
Resolution (highest precedence wins):
  Voice         project=-    user=off  env=unset  → off
  Notify        project=-    user=off  env=unset  → off
```

In the noisy project:

```
Voice:               on (project)
Notifications:       on (project)
…
Resolution (highest precedence wins):
  Voice         project=on   user=off  env=unset  → on
  Notify        project=on   user=off  env=unset  → on
```

Same machine, same user, opposite behavior — driven by `<project>/.claude-callout/voice-enabled`.

## Variations

| Need | Command |
|---|---|
| Reset back to default-on for everything | `/voice-on --global` and `/notify-on --global` |
| Add a third state: quiet by default, voice in two repos | Run `/voice-on` in each of the two repos |
| Temporarily mute the noisy project for a meeting | `/voice-mute 30m` (project scope) |

## Sharing across collaborators

The `.claude-callout/` directory in your project is **per-developer** — it's gitignored by default in this plugin's own repo. If your team agrees that a specific repo should always be loud, commit the sentinel files explicitly:

```bash
git add -f .claude-callout/voice-enabled .claude-callout/notify-enabled
git commit -m "Always notify in this repo (long test runs)"
```

The `-f` force-add is needed if your repo's `.gitignore` excludes `.claude-callout/` (recommended — see this plugin's own `.gitignore`). Once committed, everyone who clones the repo inherits the same notification behavior.
