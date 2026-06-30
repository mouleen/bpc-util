# BackupPC Retired Hosts

> Automate the lifecycle of retired BackupPC hosts using BackupPC's native retention policies while keeping the backup repository free of orphan directories.

---

## Overview

Managing retired hosts in BackupPC is often a manual and repetitive administrative task.

Although BackupPC provides an excellent backup retention mechanism, it does not manage the complete lifecycle of retired hosts. As BackupPC environments grow, administrators frequently encounter two common situations:

* Hosts that are no longer backed up but still retain historical backups.
* Backup directories that remain on disk after a host has been removed from the BackupPC configuration.

Both situations gradually increase storage usage and administrative overhead.

**BackupPC Retired Hosts** automates these maintenance tasks while remaining fully aligned with BackupPC's native behavior.

Instead of manually editing host configurations, changing retention policies, or deleting obsolete directories, administrators simply disable backups for a host through the BackupPC Web Interface. The script periodically detects retired hosts, applies a reduced retention policy and allows BackupPC to expire backups naturally during its normal housekeeping cycle.

Additionally, the script provides a maintenance mode capable of removing orphan backup directories that BackupPC does not automatically clean after hosts have been permanently removed from its configuration.

The result is a cleaner repository, lower storage consumption and a much simpler operational workflow.

---

# Why this project?

Large BackupPC installations tend to accumulate retired hosts over time.

Without an automated retirement process, administrators usually have to:

* Identify hosts that are no longer active.
* Modify retention settings manually.
* Wait for old backups to expire.
* Remove obsolete backup directories.
* Recover disk space manually.
* Periodically search for filesystem residuals.

These tasks are repetitive, time-consuming and easy to forget.

This project automates that operational workflow while continuing to rely on BackupPC's own retention engine.

Rather than replacing BackupPC's cleanup logic, it complements it by automating host retirement and removing filesystem residuals that BackupPC leaves behind after hosts are removed from its configuration.

---

# Features

* Automates the retirement lifecycle of BackupPC hosts.
* Uses BackupPC's native retention policies.
* No manual cleanup of retired hosts.
* Automatically prepares retired hosts for expiration.
* Removes orphan backup directories.
* Eliminates filesystem residuals left after host removal.
* Helps recover storage space.
* Reduces administrative effort.
* Supports unattended execution through cron.
* External configuration file.
* Designed to run as the **backuppc** user.

---

# How it works

```
                Host is no longer needed
                         │
                         ▼
      BackupDisable = 1 or BackupDisable = 2
                         │
                         ▼
         retired_hosts --run (weekly cron)
                         │
                         ▼
     Script applies retired host retention policy
                         │
                         ▼
      BackupPC housekeeping expires old backups
                         │
                         ▼
        Backup directories eventually disappear
                         │
                         ▼
   retired_hosts --cleanup-orphans (optional)
                         │
                         ▼
      Remove orphan directories left on disk
```

The script does **not** delete backups directly.

Instead, it prepares retired hosts so BackupPC can apply its own retention policy exactly as intended.

This approach preserves BackupPC's normal operation while eliminating repetitive administrative work.

---

# Requirements

* BackupPC
* Bash
* Standard BackupPC installation
* Execution as the **backuppc** user
* Permission to modify BackupPC host configuration files

---

# Installation

Copy the script into the BackupPC home directory.

```bash
cp retired_hosts /home/backuppc/
chmod +x /home/backuppc/retired_hosts
```

The script is intended to run as the **backuppc** user.

Example:

```bash
sudo -u backuppc /home/backuppc/retired_hosts --run
```

---

# Configuration

The script reads its settings from an external configuration file.

Example:

```bash
CONFDIR="/etc/BackupPC/pc"
HOSTSFILE="/etc/BackupPC/hosts"
PCDIR="/data0/backuppc/pc"
LOGDIR="/home/backuppc/log"
```

Keeping configuration outside the script allows the same version to be deployed across multiple BackupPC servers without modifying the source code.

A typical configuration includes:

| Variable    | Description                                    |
| ----------- | ---------------------------------------------- |
| `CONFDIR`   | BackupPC host configuration directory.         |
| `HOSTSFILE` | BackupPC hosts file.                           |
| `PCDIR`     | Backup repository containing host directories. |
| `LOGDIR`    | Directory where execution logs are written.    |

After creating the configuration file, verify that all paths match your BackupPC installation before running the script.

---

# Usage

The script provides two independent operating modes.

| Option              | Description                                                                         |
| ------------------- | ----------------------------------------------------------------------------------- |
| `--run`             | Applies the retired host retention policy.                                          |
| `--cleanup-orphans` | Removes orphan backup directories left after hosts have been deleted from BackupPC. |

Both operations are independent and may be scheduled separately depending on your maintenance policy.

---

# Retired Host Management (`--run`)

The primary purpose of this project is to automate the retirement of BackupPC hosts.

When a server, workstation or virtual machine is decommissioned, there is usually no need to continue creating new backups. However, historical backups often need to remain available for a limited period before being removed.

Instead of manually modifying retention parameters for every retired host, administrators simply disable backups using the BackupPC Web Interface.

Set one of the following values:

```text
BackupDisable = 1
```

or

```text
BackupDisable = 2
```

Once backups are disabled, the host enters the retirement process.

During the next scheduled execution, `retired_hosts --run` detects the retired host and automatically updates its BackupPC retention settings to the reduced retention policy defined by the script.

From that point forward, BackupPC's normal housekeeping process takes over and removes old backups according to its own retention mechanism.

No backups are deleted directly by this script.

Instead, the script prepares the host so BackupPC can perform the cleanup naturally and safely.

This approach has several advantages:

* Uses BackupPC's own expiration logic.
* Preserves BackupPC's normal operation.
* Avoids manual retention changes.
* Eliminates repetitive administrative work.
* Makes host retirement completely predictable.

---

# Typical Retirement Workflow

The recommended operational workflow is intentionally simple.

### Step 1

A host is no longer required.

---

### Step 2

Open the BackupPC Web Interface.

Edit the host configuration.

Set:

```text
BackupDisable = 1
```

or

```text
BackupDisable = 2
```

Save the configuration.

Nothing else needs to be modified.

---

### Step 3

The weekly scheduled execution runs:

```bash
/home/backuppc/retired_hosts --run
```

The script detects that the host has been retired and automatically applies the reduced retention configuration.

---

### Step 4

BackupPC housekeeping continues operating normally.

As backups become older than the configured retention period, BackupPC expires and removes them automatically.

No manual intervention is required.

---

# Cleaning Orphan Backup Directories (`--cleanup-orphans`)

Besides managing retired hosts, **BackupPC Retired Hosts** also provides a maintenance mode for cleaning orphan backup directories.

## What is an orphan backup directory?

An orphan backup directory is a host directory that still exists inside the BackupPC backup repository even though its host has already been removed from the BackupPC configuration.

This situation commonly occurs when a host has been permanently removed from the BackupPC configuration (either by editing the `hosts` file or by deleting it from the BackupPC Web Interface).

Once the host has been removed, BackupPC no longer manages it.

However, BackupPC does **not** automatically remove the corresponding backup directory from the filesystem.

As a result, obsolete backup directories can remain indefinitely, consuming storage space and leaving unnecessary residual data inside the repository.

The purpose of `--cleanup-orphans` is to eliminate these filesystem leftovers safely.

---

## How orphan detection works

The script compares:

* The hosts currently configured in BackupPC.
* The directories present inside the BackupPC repository.

Any directory that does not belong to a configured host is considered orphaned.

Only those orphan directories are removed.

Directories belonging to active BackupPC hosts are never touched.

---

## Example

Current BackupPC hosts:

```text
host01
host02
host03
```

Repository contents:

```text
host01/
host02/
host03/
old-server/
test-vm/
```

`host01`, `host02` and `host03` are valid BackupPC hosts.

`old-server` and `test-vm` are no longer configured.

Running:

```bash
/home/backuppc/retired_hosts --cleanup-orphans
```

will safely remove:

```text
old-server/
test-vm/
```

while leaving the valid BackupPC hosts untouched.

---

## Benefits of orphan cleanup

Regular orphan cleanup helps to:

* Recover disk space.
* Eliminate filesystem residuals.
* Keep the repository clean and organized.
* Simplify BackupPC administration.
* Prevent obsolete host directories from accumulating over time.

Unlike `--run`, which should execute regularly, `--cleanup-orphans` is usually executed only occasionally as part of periodic maintenance.

---

# Cron Example

The recommended schedule is to execute the retirement process once per week.

Example: **Every Tuesday at 12:02**

```cron
2 12 * * 2 /home/backuppc/retired_hosts --run
```

Install the cron entry as the **backuppc** user:

```bash
crontab -u backuppc -e
```

Then add:

```cron
2 12 * * 2 /home/backuppc/retired_hosts --run
```

This schedule ensures that retired hosts are processed automatically while BackupPC continues managing backup expiration using its native retention policies.
---

# Command Reference

## Apply the retired host retention policy

```bash
/home/backuppc/retired_hosts --run
```

Processes all hosts marked as retired (`BackupDisable = 1` or `BackupDisable = 2`) and automatically applies the reduced retention policy configured for retired hosts.

The script **does not remove backups directly**. It prepares the host so BackupPC can remove expired backups during its normal housekeeping cycle.

---

## Clean orphan backup directories

```bash
/home/backuppc/retired_hosts --cleanup-orphans
```

Searches the BackupPC repository for backup directories that no longer belong to any configured BackupPC host and removes them safely.

This operation is independent of the retired host retention process.

---

# Recommended Maintenance

A typical maintenance schedule is:

| Task                              | Frequency                                   |
| --------------------------------- | ------------------------------------------- |
| `retired_hosts --run`             | Weekly                                      |
| `retired_hosts --cleanup-orphans` | Monthly or after permanently removing hosts |

Example:

Weekly:

```cron
2 12 * * 2 /home/backuppc/retired_hosts --run
```

Monthly (first Sunday at 02:00):

```cron
0 2 1-7 * 0 /home/backuppc/retired_hosts --cleanup-orphans
```

This schedule keeps retired hosts under BackupPC's retention policy while periodically removing orphan backup directories that would otherwise remain indefinitely.

---

# Best Practices

For the best operational experience:

* Retire hosts by setting `BackupDisable = 1` or `BackupDisable = 2` from the BackupPC Web Interface.
* Avoid manually deleting backup directories for retired hosts.
* Allow BackupPC to expire backups using its native retention policy.
* Execute `retired_hosts --run` regularly using cron.
* Execute `--cleanup-orphans` periodically to remove residual filesystem data.
* Review execution logs after the first scheduled runs.
* Test configuration changes before deploying them into production.

Following this workflow keeps the BackupPC repository consistent while minimizing manual intervention.

---

# Frequently Asked Questions

### Does this script delete backups?

No.

The script never removes backup files belonging to retired hosts directly.

Instead, it modifies the retired host configuration so BackupPC's own housekeeping process removes expired backups according to the configured retention policy.

---

### Why not delete retired hosts immediately?

Organizations often need to retain historical backups for days or weeks after a system has been decommissioned.

Using BackupPC's native retention mechanism preserves this history while avoiding unnecessary manual work.

---

### What is an orphan backup directory?

An orphan backup directory is a directory that still exists inside the BackupPC repository even though its host has already been removed from the BackupPC configuration.

Because BackupPC no longer manages that host, these directories can remain on disk indefinitely unless they are removed manually.

The `--cleanup-orphans` option automates this maintenance task.

---

### Is `--cleanup-orphans` safe?

Yes.

The script compares the BackupPC host configuration with the repository contents and only removes directories that are **not associated with any configured BackupPC host**.

Directories belonging to active hosts are preserved.

---

### Can both modes be scheduled?

Yes.

The two operating modes are completely independent.

A common recommendation is:

* Run `--run` weekly.
* Run `--cleanup-orphans` monthly.

---

# Operational Benefits

Deploying **BackupPC Retired Hosts** provides several long-term advantages:

* Reduces administrative effort.
* Standardizes the host retirement process.
* Keeps retention management consistent across all retired hosts.
* Leverages BackupPC's native housekeeping mechanism.
* Eliminates obsolete filesystem residuals.
* Helps recover disk space.
* Prevents orphan backup directories from accumulating.
* Simplifies long-term BackupPC administration.
* Enables fully unattended operation through cron.

Rather than introducing a new backup management model, the project complements BackupPC by automating repetitive maintenance tasks that are typically performed manually.

---

# Contributing

Contributions, bug reports and feature requests are welcome.

If you encounter an issue or have an idea for improvement, please open an Issue or submit a Pull Request.

---

# License

This project is distributed under the MIT License.

You are free to use, modify and distribute it according to the terms of the license.

Although the script has been designed to operate safely alongside BackupPC, always validate its behavior in a non-production environment before deploying it into production.

---

## Acknowledgements

This project was created to simplify the day-to-day administration of BackupPC environments by automating repetitive maintenance tasks while preserving BackupPC's native retention workflow.

It is intended to complement BackupPC—not replace it—by reducing administrative overhead, improving repository hygiene and helping administrators keep large BackupPC installations clean, consistent and easy to maintain over time.

---

# Author

Developed by **mouleen**.

Infrastructure Engineer with extensive experience in Linux, BackupPC, virtualization, networking and automation.

This project was created to simplify BackupPC administration in large environments by automating repetitive maintenance tasks while preserving BackupPC's native behavior.

Contributions, suggestions and pull requests are welcome.

GitHub: https://github.com/mouleen

