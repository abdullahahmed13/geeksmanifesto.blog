---
title: "Automate Repetitive Tasks with Cron Jobs on Mac and Linux"
date: 2025-10-04
draft: false
---


In this article, You will learn how to automate repetitive tasks on **macOS** and **Linux** using **Cron Jobs**. We will cover the **crontab** syntax, real-world scenarios in the context of Cyber Security and how to easily edit schedules with [crontab.guru](https://crontab.guru).

## What is Crontab
**Crontab** is a powerful utility used in Task Scheduling and Automation in Unix-Like Operating Systems such as macOs and Linux. It allows users to run Unix commands, Bash Scripts or certain tasks at predefined specific time intervals.
It's really powerful for system maintenance tasks, cache management, updating databases and even automating Cyber Security or Asset discovery tasks.

## Crontab Syntax Explained
A **crontab** entry defines when and how often a certain command should run. Each line in the **crontab** file has *five time fields* followed by the executed command.

```bash
* * * * * command-to-run
| | | | |
| | | | └── Day of week (0 - 6, Sunday = 0 or 7)
| | | └──── Month (1 - 12)
| | └────── Day of month (1 - 31)
| └──────── Hour (0 - 23)
└────────── Minute (0 - 59)
```

### Special Characters in Crontab Syntax

| `*`   | Match all possible values (wildcard). | `* * * * *` = every minute.                  |
| ----- | ------------------------------------- | -------------------------------------------- |
| `*/N` | **Runs every N units.**               | **`*/5 * * * *` = every 5 minutes.**         |
| `,`   | **List multiple values.**             | **`0 8,16 * * *` = at 8 AM and 4 PM.**       |
| `-`   | **Specifies a range**                 | **`0 9-17 * * *` = every hour 9 AM - 5 PM.** |
### Predefined Crontab Macros:
**Crontab** supports some predefined Macros:
- `@reboot` → Run once, at system startup
- `@yearly` or `@annually` → `0 0 1 1 *`
- `@monthly` → `0 0 1 * *`
- `@weekly` → `0 0 * * 0`
- `@daily` → `0 0 * * *`
- `@hourly` → `0 * * * *`

Example:
```bash
@daily echo "Daily Welcome"
```

### Common Examples
- **Every Minute:** 
	```bash
	* * * * * command
	```
- **Every 10 Minutes:** 
	```bash
	*/10 * * * * command
	```
- **Every Day at 6 AM:**

	```bash
	0 6 * * * command
	```
- **Every Sunday at Midnight:**

	```bash
	0 0 * * 0 command
	```
- **On the first day of every month at 3 AM:**

	```bash
	0 3 1 * * command
	```

### Using crontab.guru for ease
[crontab.guru](https://crontab.guru) Offers easy interface for choosing commands execution timings.


![crontab.guru](img/crontabguru.png)

## Using Crontab on macOs and Linux

### Opening Crontab:
**Cron jobs** are stored per-user in a **crontab** file. To edit it:
```bash
crontab -e
```
This opens your crontab in the default editor (often `vim` or `nano`).


![crontabVim](img/crontabVim.png)

### View Existing Jobs
To see all jobs for your user:
```bash
crontab -l
```

![crontabList](img/crontabList.png)
### Removing all jobs
If ever needed to clear all jobs:
```bash
crontab -r
```

## Real  Example: Automating Asset Discovery Tasks
**Crontab** is especially useful in **cybersecurity and bug bounty hunting**, where you often need to run the same reconnaissance tasks repeatedly. Instead of doing it manually, you can schedule scripts to run automatically.

**Lets say you want to create pipeline does the following:**
1. Take a list of subdomains (`hosts.txt` file).
2. Run **Subfinder** (or any custom tool) on them.
3. Pipe the results into **httpx** to check for live domains.
4. Save only **newly-appeared** assets into a log.

### Create `minimal-recon-mac.sh`

```bash
#!/usr/bin/env bash

# minimal-recon-mac.sh - minimal subfinder -> httpx pipeline for macOS

# Place one domain per line in $HOME/recon/targets.txt

TARGETS="$HOME/recon/targets.txt"
WORKDIR="$HOME/recon"
EXPANDED="$WORKDIR/expanded.tmp"
CURRENT="$WORKDIR/live.txt"
PREVIOUS="$WORKDIR/previous.txt"
NEW="$WORKDIR/new.txt"
LOG="$WORKDIR/scan.log"

mkdir -p "$WORKDIR"

# macOS Homebrew (Apple Silicon) + common locations
export PATH="/opt/homebrew/bin:/usr/local/bin:$HOME/go/bin:/usr/bin:/bin:$PATH"

echo "[$(date)] start" >> "$LOG"

# expand with subfinder, skipping blanks/comments, then unique-sort
: > "$EXPANDED"
while IFS= read -r domain || [ -n "$domain" ]; do
  [ -z "$domain" ] && continue
  case "$domain" in \#*) continue ;; esac
  subfinder -d "$domain" -silent 2>/dev/null || true
done < "$TARGETS" | sort -u > "$EXPANDED"

# probe with httpx, write unique live hosts
httpx -silent < "$EXPANDED" | sort -u > "$CURRENT" || true

# get newly seen hosts (items in CURRENT not in PREVIOUS)
if [ -f "$PREVIOUS" ]; then
  comm -13 "$PREVIOUS" "$CURRENT" > "$NEW" || true
else
  cp -f "$CURRENT" "$NEW" || true
fi

# rotate for next run
cp -f "$CURRENT" "$PREVIOUS"

echo "[$(date)] done: $(wc -l < "$CURRENT" 2>/dev/null || echo 0) live, $(wc -l < "$NEW" 2>/dev/null || echo 0) new" >> "$LOG"
```

### Crontab task example:
```bash
0 9 * * 0 /Users/username/minimal-recon-mac.sh >> /Users/username/recon/cron.log 2>&1
```

## Conclusion
**Crontab** is one of those tools that feels almost invisible — it just works quietly in the background — but it’s incredibly powerful once you start using it. On **macOS** or **Linux**, you can schedule anything from simple housekeeping tasks to full security pipelines like automated subdomain discovery and monitoring.

>Whether you’re managing servers, blogging with Hugo, or running continuous security scans, cron makes it possible to turn repetitive tasks into one-time setups.