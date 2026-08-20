# Project Plan — linux-permissions-lab

Reference: NotHarshhaa/Fun-with-Linux-for-Cloud-DevOps-Engineers
This repository is an informed implementation, not a copy. See docs/decisions/ for
where we deviate and why.

## Environment

| Item | Value |
|---|---|
| Region | us-east-1 |
| Instance | t3.micro, Ubuntu 24.04 LTS |
| Network | Custom VPC, 10.0.0.0/16 |
| Access | SSH restricted to a single source IP (/32) |
| Cost control | AWS Budget alert, $5 monthly |

## Backlog

### EPIC 0 — Environment & Cost Control — DONE
- [x] T0.1 Decide region, instance type, AMI, access method
- [x] T0.2 Budget alert
- [x] T0.3 Launch EC2, restrictive SG, tags
- [x] T0.4 Baseline capture (whoami, id, uname -a, lsblk, df -h)

### EPIC 1 — Identity Foundation — DONE
- [x] T1.1 Permission/sudo model decided — ADR-001
- [x] T1.2 Create user1, user2, user3 (uid 1001-1003)
- [x] T1.3 Create groups devops (1004), aws (1005)
- [x] T1.4 Primary group of user2, user3 -> devops
- [x] T1.5 aws as secondary group for user1
- [x] T1.6 Verify with id / getent

### EPIC 2 — Filesystem Structure — DONE
- [x] T2.1 Reconstruct tree from diagram
- [x] T2.2 Create structure as root
- [x] T2.3 chgrp devops on /dir1, /dir7/dir10, /f2
- [x] T2.4 chown user1 on the same three paths
- [x] T2.5 Verify with ls -ld

### EPIC 3 — Delegated Administration — IN PROGRESS
- [ ] T3.1 Predict outcome of user1 running useradd
- [ ] T3.2 Attempt, observe failure, diagnose
- [ ] T3.3 Apply constrained sudoers grant per ADR-001 phase 2
- [ ] T3.4 Create user4, user5; groups app, database

### EPIC 4 — Permission Boundaries
- [ ] T4.1 Steps 3.x as user4 — predict, attempt, diagnose
- [ ] T4.2 Steps 4.x as user1 — incl. relative-path exercise
- [ ] T4.3 Steps 5.x as user2 — sed, vi, delete
- [ ] T4.4 Log every denial with root cause

### EPIC 5 — Search & Inspection (root)
- [ ] T5.1 find / -name f3
- [ ] T5.2 Count files under /
- [ ] T5.3 tail -1 /etc/passwd

### EPIC 6 — Block Storage
- [ ] T6.1 Create 5 GiB EBS in the instance AZ
- [ ] T6.2 Attach; identify device (expect nvme1n1, not sdf)
- [ ] T6.3 Create filesystem — ADR-003
- [ ] T6.4 Mount on /data, verify with df -h
- [ ] T6.5 fstab persistence decision — ADR-004
- [ ] T6.6 Create /data/f1

### EPIC 7 — Teardown
- [ ] T7.1 Steps 9.x as user5 — predict which fail
- [ ] T7.2 Delete users, groups, home directories
- [ ] T7.3 Unmount /data, remove mount point
- [ ] T7.4 Detach AND delete EBS volume
- [ ] T7.5 Terminate instance
- [ ] T7.6 Verify zero instances and zero volumes in console

### EPIC 8 — Consolidation
- [ ] T8.1 Permission-denied runbook
- [ ] T8.2 Repeat full assignment from memory on a fresh instance
- [ ] T8.3 Write README.md

## Known findings so far

- Group ownership without group write is a no-op — chgrp alone grants nothing
- getent group devops shows an empty member list despite user2/user3 holding it
  as primary; primary membership lives in /etc/passwd, not /etc/group
- Nitro instances present EBS as nvme*, not the sd*/xvd* name given at attach time