# Contributing to claude-callout

Thanks for the interest. This is a small, focused plugin — no build system, no test framework, just Bash + per-OS shell-outs. Contributions should match that scope: keep changes minimal, predictable, and aligned with the hook lifecycle.

## Quick start

Clone, then point Claude Code at your working copy directly (no marketplace round-trip needed):

```bash
git clone https://github.com/amIT-nitb/claude-callout
cd claude-callout
claude --plugin-dir .
```

Inside Claude Code, try `/voice-test` to verify end-to-end (sound + voice + notification).

## Project layout

| Path | What lives here |
|---|---|
| `.claude-plugin/plugin.json` | Plugin manifest (name, version, license) |
| `.claude-plugin/marketplace.json` | Single-plugin marketplace listing |
| `hooks/hooks.json` | Wires `Notification` / `Stop` / `UserPromptSubmit` / `SessionStart` to scripts |
| `scripts/on-*.sh` | One script per Claude Code event |
| `scripts/lib/common.sh` | All shared logic — OS detection, gating, dispatch, mute, transcript parsing |
| `bin/claude-callout` | Side CLI mirroring slash commands for shell-driven use |
| `commands/*.md` | Slash commands (one file per command) |
| `examples/` | Real-world recipes |

## Conventions

- **Hooks must `exit 0`**, even on internal failure. A non-zero exit aborts Claude's turn handling.
- **All shell-outs must background with `&` and redirect to `/dev/null`.** A hook must never block a turn, and a missing binary must silently no-op.
- **Voice line is terse on purpose** ("Claude ready" / "Claude waiting"). Don't add detail to voice — add it to the notification body.
- **Notification body leads with the same phrase as voice** so visual + audible cues match.
- **Bump `version` in both `plugin.json` and `marketplace.json` together.** They must agree.

## Manual test

There's no automated test suite. To exercise end-to-end:

```bash
scripts/test-announce.sh
```

That fires both "ready" and "waiting" announcements, bypassing all gates (voice/notify on, no quiet hours, no focus check).

### Per-hook synthetic payloads

To exercise `on-notification.sh` directly:

```bash
echo '{"cwd":"/tmp/foo","session_id":"abc12345","message":"needs permission"}' \
  | scripts/on-notification.sh
```

To also exercise the transcript-snippet code path (`last_assistant_text`), create a minimal transcript first:

```bash
cat > /tmp/transcript.jsonl <<'EOF'
{"role":"user","content":[{"type":"text","text":"run tests"}]}
{"role":"assistant","content":[{"type":"text","text":"Let me run the test suite to verify"}]}
EOF
echo '{"cwd":"/tmp/foo","session_id":"abc12345","message":"needs permission","transcript_path":"/tmp/transcript.jsonl"}' \
  | scripts/on-notification.sh
```

To exercise `on-stop.sh` with a shorter debounce (default is 10s):

```bash
echo '{"cwd":"/tmp/foo","session_id":"abc12345"}' \
  | CLAUDE_VOICE_DEBOUNCE=2 scripts/on-stop.sh
# Wait ~2 seconds for the watcher to fire from a detached subshell.
```

### Validate the manifest

Also worth running before submitting a PR:

```bash
claude plugin validate .
```

Should pass clean. Treat warnings as blockers.

## Changes that need extra care

- **`scripts/on-stop.sh`** — the debounce works by passing a unique token through `stop-pending`. Don't add an announce path that doesn't re-check the token, or you'll fire duplicate "ready" announcements.
- **`scripts/lib/common.sh` → `notify_os`** — preserve two invariants: never pass a `--group` / `-group` flag (each event must stack as a separate banner), and every shell-out must background. Alerter and terminal-notifier are blocking binaries that would freeze the hook.
- **Gating precedence** in `voice_enabled` / `notify_enabled` is intentional: **mute > project > user > env > default**. New flags should follow the same chain.

## Filing issues

Bug reports and feature requests go in [GitHub Issues](https://github.com/amIT-nitb/claude-callout/issues). Include:

- macOS / Linux / Windows version
- Claude Code version (`claude --version`)
- Plugin version (`claude plugin list | grep claude-callout`)
- What you ran, what you expected, what you got
- For notification issues: whether `alerter` / `terminal-notifier` is installed (`command -v alerter terminal-notifier`)

## Pull requests

PRs welcome. Small focused changes get merged faster. For anything non-trivial, open an issue first so we agree on the approach before you spend time writing code.

Pre-PR checklist:

- [ ] `claude plugin validate .` passes
- [ ] `scripts/test-announce.sh` still fires two banners + voice
- [ ] Version bump if user-visible behavior changes (semver: minor for new features, patch for fixes)
- [ ] CHANGELOG.md entry under `[Unreleased]`
- [ ] Docs updated if a flag, command, or default changed

## License

By contributing, you agree your contributions will be licensed under the project's [MIT License](LICENSE).
