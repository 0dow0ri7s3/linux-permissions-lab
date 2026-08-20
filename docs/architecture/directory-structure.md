# Directory Structure

Reconstructed from the assignment diagram. Built entirely under `/` as root.

## Tree

```text
/
├── home/                          (pre-existing — not created)
├── dir1/f1
├── dir2/dir1/dir2/{dir10, f3}
├── dir3/dir11
├── dir4/dir12/{f4, f5}
├── dir5/dir13
├── dir6/
├── dir7/{dir10, f3}
├── dir8/dir9
├── opt/dir14/{dir10, f3}          (opt pre-existing — dir14 created inside)
├── f1
└── f2
```

12 directories, 8 files.

## Ownership after T2.3 / T2.4

| Path | Owner | Group |
|---|---|---|
| /dir1 | user1 | devops |
| /dir7/dir10 | user1 | devops |
| /f2 | user1 | devops |
| everything else | root | root |

## Development/Learning Configuration

Creating directories directly under `/` violates the Filesystem Hierarchy Standard
and would be rejected in code review at any organisation.

It is used here deliberately. `/` is `drwxr-xr-x root:root` — ordinary users fall
through to the *other* class, which has no `w` bit. That is precisely what generates
the permission failures this exercise teaches. Relocating the tree to a
user-writable directory would remove every intentional failure and reduce the
assignment to typing practice.

**Production equivalent:** application data under `/var/lib/<app>`, configuration
under `/etc/<app>`, scratch under `/tmp` or `/var/tmp`.

## Notes on the structure

Duplicate names across different paths (`dir10` ×3, `f3` ×3, `dir1`/`dir2` nested
inside each other at `/dir2/dir1/dir2`) are deliberate. They force path-based rather
than name-based thinking — the kernel has no concept of "the dir10 directory", only
inodes reached by traversal.

The three pre-existing `f3` files matter at assignment step 6.1 (`find / -name f3`),
where the correct result depends on which deletions have already been performed.

## Verification finding

Initial build omitted `/dir5/dir13`. Caught by `tree` verification against the
diagram, not by any command error — `mkdir` succeeds silently on the directories you
do specify. Confirms that "the command ran without error" is not verification.