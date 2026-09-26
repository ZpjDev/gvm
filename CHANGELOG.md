# Changelog

This fork's changes, relative to upstream [moovweb/gvm](https://github.com/moovweb/gvm)
1.0.22 (2016). Upstream history before that is in its own repository; nothing
here changes the interface, only how it works and what it gets right.

## 1.1.0 - 2026-09-26

First release of the fork. Version 1.1.0 rather than 1.0.23 because the release
list, the installer and the test suite were replaced, not patched.

### Release data

- The release list is fetched from `https://go.dev/dl/?mode=json&include=all`
  instead of being a file in the repository, so `gvm install 1.24` finds patch
  releases published after this repository was forked. `GVM_INDEX_FILE` points
  at a parsed index for air-gapped machines, `GVM_OFFLINE=1` reuses the cache
  and does not refresh it, and `GVM_NO_CACHE=1` forces a refresh.
- Platform artifacts are chosen from the index, so an OS/architecture without a
  published binary says so instead of downloading something that will not run.
- Every download is verified against the SHA256 in the index; a mismatch is
  refused and the file is deleted. `GVM_NO_VERIFY=1` is the only way past it.
- Version sorting is numeric per component, prereleases are excluded from
  partial requests such as `1.24`, and aliases may chain - with a cycle check.
- Source builds pick a bootstrap toolchain: 1.5-1.19 use 1.4, 1.20-1.21 use
  1.17.13, 1.22-1.23 use 1.20.6, 1.24-1.25 use 1.22.6, 1.26-1.27 use 1.24.6, and
  anything newer uses the previous even-numbered minor.

### Behaviour

- `GOTOOLCHAIN=local` is exported whenever a gvm-managed version is selected, so
  a `go.mod` asking for a newer Go cannot make the selected toolchain download
  and use that one behind your back. `gvm use system` unsets it again.
- `gvm install <version> --force` keeps the version's package set. It used to
  go through `uninstall`, which ran `go clean -modcache` and deleted the pkgset,
  so recovering from a corrupted GOROOT also threw away the module cache.
- Reinstalling also refreshes `environments/default` when it pointed at that
  version, so a new shell does not keep settings from an older gvm.
- `gvm implode` refuses a `GVM_ROOT` that is `$HOME` or lacks `scripts/` and
  `VERSION`, and only prompts when there is a terminal, so a script gets an
  error instead of a hang.
- `gvm uninstall` refuses to remove the version currently in use unless forced,
  repairs `environments/default` when it pointed at what was removed, and drops
  aliases that pointed there.

### Commands

- New: `gvm doctor`, `gvm ls-remote`, `gvm applymod`, `gvm which`, `gvm delete`.
- Rewritten: `gvm install`, `gvm use`, `gvm pkgset`, `gvm diff`, `gvm help`,
  `gvm linkthis`, `gvm pkgenv`, `gvm completion`, `gvm display`.
- `gvm get`, `gvm update` and `gvm cross` are gone. `cross` was a wrapper for
  build tooling that no longer exists; the other two were dead branches that
  shadowed `gvm install`.

### Portability and installation

- zsh is supported and covered by the suite, not just bash. Option changes
  (`KSH_ARRAYS`, `BASH_REMATCH`, `BASH_SOURCE`, `posix`) are saved and restored,
  and array handling works with and without `KSH_ARRAYS`.
- `GVM_ROOT` and other paths may contain spaces.
- `install.sh` replaces the autotools build: it installs from a checkout without
  cloning upstream over it, stages a self-install so it cannot delete its own
  source, keeps the git history as `git.bak`, and honours `--no-profile` on
  `--uninstall` as well as on install. A leftover `*.gvm-backup` is no longer
  restored over your shell profile, only mentioned.
- A Go inside `$GVM_ROOT` is no longer recorded as the "system" Go when
  installing over an existing gvm.
- Boolean environment variables accept `1/true/yes/on` and `0/false/no/off`, so
  `GVM_NO_VERIFY=0` verifies. `GVM_OFFLINE`, `GVM_NO_VERIFY`, `GVM_QUIET`,
  `GVM_DEBUG` and `GVM_NO_COLORS` all go through one helper.

### Documentation and tests

- `README.md` rewritten, and a Chinese translation added.
- The suite is 660 assertions of plain bash with no external framework, one file
  per area, each building a throwaway `GVM_ROOT`. It fails if a test writes to
  the developer's real `~/.bashrc` or `~/.zshrc`, and it runs under
  `LC_ALL=C` so a non-English locale cannot hide a failure.
- GitHub Actions runs lint and tests on Linux and on macOS, whose `bash` is 3.2.
