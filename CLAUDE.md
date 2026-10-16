# CLAUDE.md

Orientation for Claude Code working in this repo. What the tool is for is in
[README.md](./README.md); operator documentation is in [`docs/`](./docs/).

**`dev-notes/` is an internal design archive and is not published.** It names
real client sites and quotes their logs. If your checkout has it, start at
`dev-notes/00-project-tracker.md` and read the numbered design docs before
writing code. Source comments may cite `dev-notes/NN`; every such comment must
also state its substance inline.

## What this is

A read-only command-line health check for PHP-FPM pools. Given a pool config
file (or `--all`), it samples the pool's workers, listen socket, logs and
optional status page for a few seconds and reports findings with stable check
ids, severities and evidence.

**Current state: design.** No code yet. `dev-notes/00-project-tracker.md` is the
authority on what is decided and what is open.

## Rules that shape everything

- **Read-only, always.** Never reload or signal FPM, never edit a pool, never
  execute a site's code. The only network traffic is FastCGI to the pool's own
  socket, and only for the status page. It must be safe to run on a
  revenue-bearing production server at any moment, as root.
- **Tripwire, not filter.** Precision over recall. A check that cries wolf on
  busy-but-healthy sites gets ignored, and then it is worthless. Every threshold
  is tested against real fleet logs before it ships.
- **Never skip silently.** A check that cannot run (permission denied, status
  page not enabled, log not found) produces an explicit `skipped` finding with
  the reason. A clean report must mean "looked and found nothing", never
  "could not look".
- **The tool's PHP is not the pool's PHP.** A POSIX `sh` launcher finds a
  suitable PHP 8 CLI and execs the real code with it. Nothing assumes the
  default `php` binary, and nothing needs to match the pool's version. When a
  check needs version-specific facts it runs that pool's own `php-fpmX.Y`
  binary as a subprocess.

## Coding standards

- No Composer runtime dependencies. Dev-only tooling is fine.
- Command-line options and any environment variables are parsed in exactly one
  place into a typed config object, failing loudly and listing every problem at
  once.
- Single entry, single exit. Never `return` from inside a loop. A guard clause
  at the top of a function is fine.
- No silent `catch`. A caught error is rethrown or becomes a `skipped` or
  `error` finding.
- Descriptive names everywhere, including the launcher: `PHP_BINARY_PATH`, not
  `P`. Brace every shell expansion.
- Check ids, severities, exit codes and JSON keys are a contract with callers.
  Each is defined once as a named constant and referenced, never retyped.
- One-line docblock per function. Inline comments only where the mechanism is
  non-obvious; rationale belongs in docs.

## Writing

Public files (`README.md`, `docs/`) never name a client, a client site or a
Headwall server. British English, no em-dashes.
