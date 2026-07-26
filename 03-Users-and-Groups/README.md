# Users and Groups

## Overview

User and group management is one of the core responsibilities of a Linux System Administrator. This section of the homelab focuses on creating and managing user accounts, assigning group memberships, understanding Linux account databases, and configuring file permissions.

These labs were completed while studying for the Red Hat Certified System Administrator (RHCSA) certification and emphasize hands-on administration using the command line.

---

## Topics Covered

- User IDs (UID)
- Group IDs (GID)
- Primary and supplementary groups
- Creating and modifying users
- Creating and managing groups
- Password management
- Linux account database files
- File ownership
- File permissions
- Administrative commands for user management

---

## Key System Files

| File | Purpose |
|------|---------|
| `/etc/passwd` | Stores user account information |
| `/etc/shadow` | Stores encrypted passwords and password policies |
| `/etc/group` | Stores group information |
| `/etc/gshadow` | Stores secure group password information |

---

## Common Commands

- `useradd`
- `usermod`
- `userdel`
- `passwd`
- `groupadd`
- `groupmod`
- `groupdel`
- `id`
- `groups`
- `chown`
- `chmod`

---

## Labs

- Lab 01 – Auditing-User-Login-History.md
- Lab 02 – Verifying-User-and-Group-Identity.md
- Lab 03 – Creating-and-Managing-Local-Users.md
- Lab 04 – Configuring-Non-Interactive-Service-Accounts.md
- Lab 05 - Understanding-Linux-Authentication-Files.md
