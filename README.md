# sealed CLI

The command-line client for [Sealed](https://sealed.souravlayek.com), a secrets manager for developers. Load a project's environment variables into your terminal, or run a command with them injected, without storing values in dotfiles.

## Install

### Homebrew (macOS)

```bash
brew install --cask souravlayek/tap/sealed
```

On first install, macOS may block the binary because it isn't notarized yet. Remove the quarantine flag once:

```bash
xattr -d com.apple.quarantine "$(which sealed)"
```

Repeat this after each upgrade.

### Go toolchain (any platform)

```bash
go install github.com/souravlayek/sealed-cli@latest
```

Requires Go 1.22 or later. The binary goes to `$(go env GOPATH)/bin`, so make sure that directory is on your `PATH`.

### Prebuilt archives

Download the archive for your platform from the [Releases page](https://github.com/souravlayek/sealed-cli/releases), extract it, and move `sealed` onto your `PATH`.

Verify the install:

```bash
sealed version
```

## Quick start

```bash
# Sign in this machine (opens a browser to approve the device)
sealed login

# Load a project's dev secrets into the current shell
eval "$(sealed load payments-api dev)"

# Or run one command with the secrets injected
sealed run payments-api dev -- npm start
```

`load` prints `export KEY='value'` lines. `eval` runs them in your current shell, since a child process can't change its parent's environment. The variables last until you close the terminal or run `unset`.

## Commands

| Command | Description |
|---|---|
| `sealed login` | Authorize this machine through the device flow. The private key is stored in the OS keychain. |
| `sealed load <project> <env> [KEY...]` | Print `export` lines for the environment. |
| `sealed run <project> <env> -- <command>` | Run a command with the secrets in its environment. Values are never printed. |
| `sealed version` | Print the version, commit, and build date. |


## Shell notes

- **bash and zsh:** use `eval "$(sealed load ...)"` as shown.
- **fish:** `sealed load` prints POSIX syntax, so use `sealed run` instead until a fish output mode exists.

## Security

- No tokens or secrets are written to plaintext files.
- The device's Ed25519 private key lives in the OS keychain. Every request is signed with it.
- Revoke a lost machine from the dashboard, and rotate any secrets it already loaded at the provider.

## Configuration

| Variable | Purpose |
|---|---|
| `SEALED_API_URL` | Override the API server URL (for example, a self-hosted instance) |

## Development

```bash
go run ./cmd/sealed --help
go test ./...
```

Build a local snapshot with GoReleaser:

```bash
goreleaser release --snapshot --clean
```

## Release

Releases are built with [GoReleaser](https://goreleaser.com) from a semver tag:

```bash
git tag v0.1.0
git push origin v0.1.0
goreleaser release --clean
```

The macOS build is compiled with `CGO_ENABLED=1` so it can use the Keychain. The Linux and Windows builds use `CGO_ENABLED=0`.

## License

MIT
