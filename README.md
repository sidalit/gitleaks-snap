<div align="center">

<img src="snap/gui/gitleaks.png" alt="Gitleaks logo" width="128" />

# Gitleaks

**Find secrets in Git repositories, files and standard input.**

[![gitleaks](https://snapcraft.io/gitleaks/badge.svg)](https://snapcraft.io/gitleaks)
[![Base: core26](https://img.shields.io/badge/Base-core26-2C8C3C?logo=canonical&logoColor=white)](https://snapcraft.io/core26)
![Confinement: strict](https://img.shields.io/badge/Confinement-strict-2C8C3C)
[![Maintainer: sidaliteir](https://img.shields.io/badge/Maintainer-sidaliteir-2C8C3C?logo=snapcraft&logoColor=white)](https://snapcraft.io/publisher/sidaliteir)
![Architectures: amd64 and arm64](https://img.shields.io/badge/Architectures-amd64%20%7C%20arm64-2C8C3C?logo=linux&logoColor=white)

[Getting Started](#getting-started) · [Configure](#configure) · [Limitations](#limitations) · [Build](#build) · [Support](#support) · [License](#license)

Unofficial snap packaging for [Gitleaks](https://github.com/gitleaks/gitleaks), a scanner that detects passwords, API keys and tokens in Git history, files and piped text.

</div>

---

## Getting Started

<a href="https://snapcraft.io/gitleaks"><img alt="Get it from the Snap Store" src="https://snapcraft.io/en/dark/install.svg" width="180"></a>

```bash
sudo snap install gitleaks
cd ~/my-project
gitleaks git -v
```

Replace `~/my-project` with a Git repository. `gitleaks git` scans its history, `gitleaks dir` scans files as they are on disk, and `gitleaks stdin` scans piped text:

```bash
gitleaks dir -v ~/my-project
cat notes.txt | gitleaks stdin -v
gitleaks git --report-path gitleaks-report.json
```

Bash completion is included.

To check staged changes before each commit, save this as `.git/hooks/pre-commit` in the repository and make it executable:

```sh
#!/bin/sh
exec /snap/bin/gitleaks git --pre-commit --staged --redact --verbose
```

With the [pre-commit](https://pre-commit.com) framework, add a local hook to `.pre-commit-config.yaml`:

```yaml
repos:
  - repo: local
    hooks:
      - id: gitleaks
        name: Detect hardcoded secrets
        entry: gitleaks git --pre-commit --staged --redact --verbose
        language: system
        pass_filenames: false
```

Upstream's `gitleaks-system` hook also runs this snap, but only scans correctly with `pass_filenames: false` added. The `gitleaks` and `gitleaks-docker` hooks build or pull their own copy.

## Configure

This snap has no `snap set` options, and Gitleaks keeps no settings or data of its own. It takes its rules from the first of these sources that is present, and from its built-in rules otherwise:

| Source | Value |
| --- | --- |
| `--config` | Path of a TOML file |
| `GITLEAKS_CONFIG` | Path of a TOML file |
| `GITLEAKS_CONFIG_TOML` | Content of a TOML file |
| `.gitleaks.toml` | File in the scanned folder |

A `.gitleaksignore` file in the scanned folder lists findings to skip, and `--baseline-path` takes an earlier JSON report. `--report-path` writes a report whose format follows the file extension (`.json`, `.csv`, `.sarif`) or `--report-format`.

The included Git reads `~/snap/gitleaks/current/.gitconfig` and the folder `~/snap/gitleaks/current/.config/git`, also when `XDG_CONFIG_HOME` is set in your session. It does not use your own `~/.gitconfig`, `~/.config/git` or `/etc/gitconfig`.

Optional access to repositories and files under `/media` or `/mnt`:

```bash
sudo snap connect gitleaks:removable-media
```

## Limitations

- Repositories, files, configurations and reports must be in a normal folder under your home directory or on connected removable storage. Hidden folders directly under your home directory and system paths such as `/opt` are not accessible. Hidden files inside a project, such as `.env`, are scanned.
- `/tmp` is private to the snap: files in the host's `/tmp` are not found, and a report written there is not visible outside.
- Read the log, not only the exit code. A folder the snap cannot read, or an error from Git, can end with a `permission denied` warning or an `ERR [git]` line followed by `no leaks found` and exit code 0.
- Git runs inside the snap, without the programs a repository's Git configuration may call, such as a `sops` textconv driver or the Git LFS filter. For textconv drivers, scan with `--log-opts="--all --no-textconv"`.
- Run Gitleaks as the owner of the files. With `sudo` it cannot read other users' home folders.
- The snap has no network access, so `--diagnostics http` cannot start its profiling server. `--diagnostics cpu,mem,trace` with `--diagnostics-dir` works.

## Build

```bash
git clone https://github.com/sidalit/gitleaks-snap.git
cd gitleaks-snap
sudo snap install snapcraft --classic
snapcraft pack
```

## Support

- **Installation or snap issues:** [open a packaging issue](https://github.com/sidalit/gitleaks-snap/issues).
- **Application bugs:** [report them upstream](https://github.com/gitleaks/gitleaks/issues).

## License

Snap packaging: [MIT](https://github.com/sidalit/gitleaks-snap/blob/main/LICENSE).
