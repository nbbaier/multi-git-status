# mgitstatus

`mgitstatus` shows uncommitted, untracked, and unpushed changes across multiple Git repositories at once. It scans a directory tree concurrently and prints a status summary for each repo found.

## Installation

```bash
go install github.com/nbbaier/mgitstatus/cmd/mgitstatus@latest
```

Or build from source:

```bash
git clone https://github.com/nbbaier/mgitstatus
cd mgitstatus
go build -o mgitstatus ./cmd/mgitstatus
```

## Usage

```
mgitstatus [--version] [-w] [-e] [-f] [--throttle SEC] [-c] [-d/--depth=2] [--flatten] [--json] [--no-X] [DIR [DIR]...]
```

By default, scans the current directory two levels deep. Pass one or more directories to scan those instead.

```bash
# Scan current directory
mgitstatus

# Scan specific directories
mgitstatus ~/Code ~/Work

# Scan infinitely deep
mgitstatus -d 0 ~/Code
```

## Options

| Flag | Description |
|------|-------------|
| `-b` | Show currently checked out branch |
| `-c` | Force color output (useful when piping) |
| `-d`, `--depth=2` | Scan this many directories deep (0 = infinite) |
| `-e`, `--no-ok` | Exclude repos that are clean |
| `-f` | Fetch from remote before checking (slow for many repos) |
| `-w` | Warn about directories that are not Git repositories |
| `--flatten` | Print one status indicator per line instead of grouping |
| `--json` | Output each repo as a JSON object (JSONL format) |
| `--no-depth` | Do not recurse into subdirectories |
| `--no-stream` | Buffer all output and print after all repos are checked |
| `--throttle SEC` | Wait SEC seconds between fetches (used with `-f`) |
| `--version` | Print version and exit |

### Suppressing specific checks

| Flag | Suppresses |
|------|-----------|
| `--no-push` | "Needs push" |
| `--no-pull` | "Needs pull" |
| `--no-upstream` | "Needs upstream" |
| `--no-uncommitted` | "Uncommitted changes" |
| `--no-untracked` | "Untracked files" |
| `--no-stashes` | Stash count |

## Output

Each repo is printed on one line with color-coded status indicators:

```
~/Code/project-a: ok
~/Code/project-b: Needs push (main) Uncommitted changes
~/Code/project-c: Needs pull (main) Untracked files
~/Code/project-d: Needs upstream (feature-branch)
~/Code/project-e: 2 stashes
```

### JSON output

With `--json`, each repo is emitted as a JSON object (one per line):

```json
{"path":"~/Code/project-b","branch":"main","ok":false,"needs_push":["main"],"needs_pull":[],"needs_upstream":[],"uncommitted":true,"untracked":false,"stashes":0}
```

## Ignoring a repository

To exclude a specific repo from all scans, set the `mgitstatus.ignore` config option in that repo:

```bash
git config mgitstatus.ignore true
```

## Environment variables

| Variable | Description |
|----------|-------------|
| `MG_DEBUG=1` | Enable debug output to stderr |

## How it works

- Scans directories concurrently, bounded by the number of CPU cores
- Results stream to stdout as each repo finishes (use `--no-stream` to wait for all)
- Checks each branch for push/pull status against its upstream
- Detects uncommitted changes, untracked files, stashes, unsafe ownership, and index locks
