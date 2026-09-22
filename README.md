# asdf-ccrotate

[asdf](https://asdf-vm.com) plugin for [ccrotate](https://github.com/CruGlobal/ccrotate) —
rotates between Claude Enterprise seats before any one of them exhausts its
five-hour usage window, so running Claude Code sessions and their sub-agents
never hit a rate limit.

Supports macOS and Linux. On macOS ccrotate keeps credentials in the Keychain
and runs its daemon under launchd. On Linux it keeps them in the libsecret
Secret Service and runs its daemon as a systemd user service.

## Install

```sh
asdf plugin add ccrotate https://github.com/CruGlobal/asdf-ccrotate.git
asdf install ccrotate latest
asdf set -u ccrotate latest
```

### Requirements

The [GitHub CLI](https://cli.github.com), authenticated:

```sh
brew install gh && gh auth login
```

On Linux, `libsecret` (the `secret-tool` command) is recommended so credentials
land in your desktop keyring. Without a reachable Secret Service, ccrotate falls
back to `0600` files under `~/.local/state/ccrotate/secrets`.

`CruGlobal/ccrotate` is private, so its release assets cannot be downloaded
anonymously. This plugin shells out to `gh release download`, which reuses the
credentials you already have — no personal access token to create, and no Go
toolchain, since the binaries are prebuilt.

This plugin repository is public precisely so that `asdf plugin add` works
without credentials: since asdf 0.16 the Go rewrite clones plugins with go-git
rather than the system `git`, so private *plugin* repos fail to clone
([asdf#1882](https://github.com/asdf-vm/asdf/issues/1882)). Keeping the plugin
public and the binaries private avoids that entirely.

## Setup

```sh
ccrotate init
ccrotate account add <name> --email <you@cru.org>   # once per seat, opens a browser
ccrotate doctor
ccrotate install
```

`ccrotate install` loads the daemon as a service (a LaunchAgent on macOS, a
systemd user service on Linux) and points Claude Code at the local proxy by
setting `ANTHROPIC_BASE_URL` in `~/.claude/settings.json` (backed up first).
Restart any running `claude` sessions afterwards. Remote Control and `/schedule`
are disabled while traffic goes through the proxy. `ccrotate uninstall` reverses
both changes.

On Linux you can check the daemon with `systemctl --user status ccrotate.service`,
and its logs live in `~/.local/state/ccrotate/logs/`.

## Upgrading

```sh
asdf install ccrotate <version>
asdf set -u ccrotate <version>
ccrotate install      # re-point the service at the new binary
```

That last step is not optional. The service records an absolute path to the
binary, and asdf installs each version to its own directory, so the daemon keeps
running the previous release — which still exists, so nothing errors.
`ccrotate doctor` warns when the agent and your `PATH` disagree.
