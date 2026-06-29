# Voice-only accessibility setup

**Goal:** have Claude speak everything aloud regardless of focus — so a user who relies on audio cues (visually impaired developer, multitasker keeping eyes elsewhere) gets the same information stream as a sighted user gets from the banner.

## Use case

By default `claude-callout` suppresses voice when the terminal/IDE is the foreground app — **on macOS only**, where the plugin can detect focus via `osascript`. On Linux and Windows the focus check is a no-op, so voice always announces. For users who rely on audio cues on macOS, the `voice-when-focused` flag opts back in to speaking regardless of focus.

## Commands

User-wide, one-time setup:

```
/voice-on --global                 # ensure voice is enabled everywhere (already default)
/voice-when-focused-on --global    # speak even at the keyboard
/notify-on --global                # keep banners on too (don't lose them)
```

## What changes (macOS)

| Event | Before | After |
|---|---|---|
| Terminal focused, Claude finishes | banner only (voice suppressed) | banner **and** spoken "Claude ready" |
| Terminal focused, Claude needs permission | banner only | banner **and** spoken "Claude waiting" |
| Terminal not focused (you walked away) | banner + voice | banner + voice (unchanged) |

On Linux and Windows, voice already speaks regardless of focus (the focus check is macOS-only), so this flag is a no-op there.

The voice message stays terse ("Claude ready" / "Claude waiting"). The tool summary ("Bash×4, Edit×2") and the quoted snippet of what Claude was asking go to the banner body only — they're not spoken, by design, to keep voice brief.

## Variations

| Need | Command |
|---|---|
| Voice everywhere on this project only | `/voice-when-focused-on` (drop `--global`) |
| Change the macOS voice / speech rate | Open **System Settings → Accessibility → Spoken Content** → adjust **System voice** and **Speaking rate**. The plugin's `say_text` shells out to plain `say`, which respects this system setting. Preview voices with `say -v ?` in your shell. |
| Set quiet hours so voice doesn't fire overnight | `export CLAUDE_VOICE_QUIET="22-7"` in your shell rc |
| Pair with VoiceOver for full screen-reader stack | Enable in System Settings → Accessibility → VoiceOver — independent of this plugin |

## Verifying

After enabling, run:

```
/voice-test
```

You should hear two spoken phrases ("Claude ready", then "Claude waiting") regardless of which app is focused. If voice is silent, check:

- System volume isn't muted
- macOS has TTS access: `say "test"` from your shell should work
- `/voice-status` shows `Voice-when-focused: on (user)`
