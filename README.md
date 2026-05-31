# updatetools

One command to update **everything updatable** on your Mac — Homebrew, language
package managers, global CLIs, editor extensions, and (optionally) macOS itself —
plus an animated full-screen dashboard to watch while it runs.

```
┌────────────────────────────────────────────┐
│                                            │
│   updatetools  ·  keeping your Mac fresh   │
│                                            │
└────────────────────────────────────────────┘

  [██████████░░░░░░░░░░░░░░░░░░]  11/31   35%     ⏱  01:24

   ✓ Homebrew · update index
   ✓ Homebrew · upgrade formulae
   ⠹ Homebrew · upgrade casks
   ○ npm · global packages
   – deno · upgrade
   ↳ ==> Upgrading visual-studio-code...
```

## What it does

`updatetools` walks through every package manager and tool it can find and
upgrades it. Every step is **guarded** — if a tool isn't installed, the step is
skipped, so the same script is safe to run on any Mac.

Covered today:

| Area | Tools |
|------|-------|
| Homebrew | `brew update`, formula & cask upgrades (greedy), autoremove, cleanup, `brew doctor` |
| Node ecosystem | `npm` globals, `pnpm`, `yarn`, `bun`, `deno` |
| Python | `pipx`, `uv` (self + tools), `conda` |
| Rust | `rustup`, `cargo install-update` |
| Ruby / PHP | system-`gem` aware, `composer` self + global |
| Version managers | `mise`, `asdf`, `pyenv`, `rbenv`, `nodenv`, `fnm`, `volta`, `tfenv` |
| Cloud / k8s | `gcloud`, `az`, `aws`, `helm`, `kubectl krew`, `flutter`, `dotnet` |
| CLIs | Codex (custom npm prefix aware), Supabase, Vercel, `gh` extensions |
| Editor / docs | VS Code extensions, `tldr` cache |
| App Store / OS | `mas`, macOS software updates (opt-in) |

Verbose output streams to a per-run log file; the dashboard only shows status.

## Install

```bash
git clone https://github.com/mytler/updatetools.git
cd updatetools
chmod +x updatetools updatetools-tui

# put them on your PATH (example)
mkdir -p ~/.local/bin
cp updatetools updatetools-tui ~/.local/bin/
# make sure ~/.local/bin is on your PATH in ~/.zshrc:
#   export PATH="$HOME/.local/bin:$PATH"
```

> `updatetools-tui` must live next to `updatetools` (the `--pretty` flag execs it
> by relative path).

## Usage

```bash
updatetools                 # update all software (no OS reboot)
updatetools --pretty        # same, with the animated dashboard
updatetools --macos         # ALSO install macOS + App Store updates (may reboot!)
updatetools --pretty --macos
updatetools --no-greedy     # don't force-upgrade self-managing casks
updatetools --help
```

### Flags

| Flag | Effect |
|------|--------|
| `--pretty`, `--tui` | Run the animated full-screen dashboard (`updatetools-tui`). |
| `--macos`, `--all` | Install macOS **and** Mac App Store updates. Off by default. |
| `--no-greedy` | Skip `--greedy` so casks that self-update are left alone. |

### Environment toggles

| Var | Same as |
|-----|---------|
| `MACOS_UPDATES=1` | `--macos` |
| `GREEDY=0` | `--no-greedy` |

## Why macOS updates are opt-in

`sudo softwareupdate -ia` can **reboot your machine mid-work** without warning, so
OS and App Store updates are never installed by default. A normal run only *lists*
available macOS updates; pass `--macos` (or `MACOS_UPDATES=1`) to actually install
them.

## The dashboard (`updatetools-tui`)

- Spinner on the active step, live tail of its output.
- Progress bar, percentage, and elapsed timer.
- Status icons: `✓` done · `–` skipped · `✗` failed · `○` pending.
- Uses the alternate screen buffer (like `htop`/`vim`) and restores the terminal
  cleanly on exit or `Ctrl-C`.
- Falls back to plain `updatetools` automatically when not attached to a TTY
  (pipes, cron, CI).

On finish it prints a summary and the path to the full log:

```
⟳ updatetools finished in 00:57
  ✓ 11 updated   – 19 skipped   ✗ 1 failed
Full log: /var/folders/.../updatetools-39499.log
```

## Notes & gotchas

- **Codex CLI** is detected by resolving its binary symlink, so it updates
  correctly even when installed into a non-default npm prefix (e.g. an iCloud
  `~/.local`). The `codex-app` desktop cask is upgraded separately.
- **System Ruby** (`/usr/bin/gem`) is SIP-protected and intentionally skipped —
  use a Homebrew/`rbenv` Ruby if you want gems managed.
- **Homebrew Python** is externally managed, so global `pip` upgrades are skipped
  on purpose; use `pipx`/`uv` for tools.
- A UTF-8 locale is forced internally so the dashboard box always aligns.

## Requirements

- macOS (Apple Silicon or Intel)
- `bash` (the system `/bin/bash` is fine)
- Homebrew recommended (most steps build on it)

## License

MIT — see [LICENSE](LICENSE).
