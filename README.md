# php-fpm-doctor

A command-line health check for PHP-FPM pools. Point it at a pool's config file
and it tells you, in a few seconds, whether that pool is healthy, struggling or
stuck, and why.

> **Status: design stage.** Nothing here runs yet. This README describes the
> intended tool.

## Why

The usual ways to notice a sick PHP-FPM pool are indirect: an uptime monitor
that times out, a CPU graph that has been flat at the top for hours, or a client
who rings up. By then the pool has often been in trouble for a long time.

Most of the evidence is already sitting on the server: the worker processes in
`/proc`, the pool's listen socket, the access and error logs, and FPM's own
status page and slow log. `php-fpm-doctor` reads all of it in one pass and
produces a diagnosis rather than a dashboard.

## What it looks for

- **Loopback storms.** A site making requests to itself at a rate no normal
  site does: a background job, async queue runner or migration that keeps
  dispatching itself.
- **Saturation and over-provisioning.** Requests queueing on the listen socket,
  `pm.max_children` being hit, or a pool sized far beyond anything it has ever
  used.
- **Memory trouble.** PHP `memory_limit` exhaustion, workers killed by the
  kernel's OOM killer, and hosts whose pools together could promise more memory
  than the machine has.
- **Stuck and hot workers.** A worker pinned at a full CPU core, blocked on
  disk or network storage, crashing, or timing out.

## How it runs

```
php-fpm-doctor --pool /etc/php/8.4/fpm/pool.d/example.com.conf
php-fpm-doctor --all --json
```

Plain text for people, JSON for scripts, and Nagios-style exit codes
(`0` OK, `1` warning, `2` critical, `3` unknown) so a cron job or monitoring
agent can decide what to do with the result.

It is read-only. It never reloads FPM, never changes a pool and never runs a
site's code.

## Requirements

- Linux with PHP-FPM
- A PHP 8 CLI binary somewhere on the host. This is only for the tool itself:
  the pools it examines can run any PHP version, including legacy 5.6 sites.
- Run as root for the full picture. As an ordinary user it still runs, and says
  which checks it could not perform.

## Licence

To be decided.
