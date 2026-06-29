# Voice-only accessibility setup

**Goal:** have Claude speak everything aloud regardless of focus — so a user who relies on audio cues (visually impaired developer, multitasker keeping eyes elsewhere) gets the same information stream as a sighted user gets from the banner.

## Use case

By default `claude-callout` suppresses voice when the terminal/IDE is the foreground app, on the assumption that a user *at the keyboard* will see the banner. For accessibility, you may need the spoken cue regardless of focus.

## Commands

User-wide, one-time setup:

```
/voice-on --global                 # ensure voice is enabled everywhere (already default)
/voice-when-focused-on --global    # speak even at the keyboard
/notify-on --global                # keep banners on too (don't lose them)
```

## What changes

| Event | Before | After |
|---|---|---|
| Terminal focused, Claude finishes | banner only (voice suppressed) | banner **and** spoken "Claude ready — Bash×4, Edit×2" |
| Terminal focused, Claude needs permission | banner only | banner **and** spoken "Claude waiting" |
| Terminal not focused (you walked away) | banner + voice | banner + voice (unchanged) |

The voice message stays terse ("Claude ready" / "Claude waiting") — only ~1 second of speech. The detail is in the banner, which a screen reader can pick up if it's set as the foreground OS event.

## Variations

| Need | Command |
|---|---|
| Voice everywhere on this project only | `/voice-when-focused-on` (drop `--global`) |
| Use a faster voice (macOS) | `defaults write com.apple.speech.voice.prefs SelectedVoiceCreator -int 1953069538` (system setting, not plugin-specific) |
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
