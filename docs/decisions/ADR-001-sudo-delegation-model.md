# ADR-001 — Sudo delegation model for user1

**Status:** Accepted
**Date:** 2026-08-20

## Context

The assignment requires `user1` to create users (`user4`, `user5`) and groups
(`app`, `database`). Creating a user writes to `/etc/passwd`, `/etc/shadow`, and
`/etc/group`, all owned by `root`. `user1` is an ordinary account: uid 1001, no
membership in any group trusted by `/etc/sudoers`.

As written, the step cannot succeed. A decision is required on whether — and how —
to grant elevation.

## Options considered

**A — No elevation.** The step fails. Diagnose and document.

**B — Full sudo.** `user1 ALL=(ALL) NOPASSWD:ALL`. Grants root on demand.

**C — Binary allowlist.** `user1 ALL=(root) /usr/sbin/useradd, /usr/sbin/groupadd`.
Appears to be least privilege.

**D — Group membership.** Add `user1` to the `sudo` group. Equivalent to B.

## Decision

Two-phase.

**Phase 1 — Option A.** Attempt without elevation. Observe and document the failure.

**Phase 2 — Option C, argument-constrained.** Permit only the specific invocations
the assignment requires, not the bare binaries.

## Rationale

Option C in its unconstrained form is not least privilege. `useradd` accepts flags
that reach root by three separate paths:

- `useradd -u 0 -o backdoor` — creates an account with uid 0. The kernel resolves
  identity by integer, so this account *is* root under another name.
- `useradd -G sudo newguy` — places the new account directly in the sudo group.
- `useradd -d /etc -m victim` — creates and chowns `/etc` to the new user, who then
  owns the directory containing `/etc/passwd` and `/etc/shadow`.

Restricting *which binary* runs as root is not the same as restricting *what that
binary can do*. The unit of least privilege is the capability, not the executable
name. The same applies to `sudo vi` (`:!sh`), `sudo find` (`-exec`), and
`sudo tar` (`--to-command`).

## Consequences

- Phase 1 blocks progress by design; this is the intended lesson.
- Phase 2 relies on sudoers argument matching, which is a weak boundary and known
  to be bypassable. Acceptable in a disposable lab; **not** acceptable in production.
- Production equivalent: declare the user set in configuration management
  (Ansible, a directory service, or cloud IAM). No human holds ad-hoc create rights.

**Development/Learning Configuration** — this delegation exists to make the
assignment progress, not because it is a defensible production control.