# File-Permissions-On-Server

**3 master `find` commands** that cover **100% of all FAIL/PASS cases**.

***

# **MASTER COMMAND #1 — World‑Readable Sensitive Files (FAIL Condition)**

> **Meaning:** These files are readable by *everyone* → secrets can leak.

```bash
find / -perm -004 -type f 2>/dev/null
```
### Meaning
This finds all files on the system that have the permission “readable by others”.
-perm -004 means:

/ → scan entire filesystem

0 → “others”

04 → read permission

-perm -004 → find files readable by “others”

-type f → only files

2>/dev/null → hide permission errors

### ✔ PASS

*   Output is empty **OR**
*   Only non‑sensitive files appear (e.g., public docs)

### ❌ FAIL

If you see **ANY** of these in the output:

*   `.env`
*   `config.php`
*   `settings.py`
*   `application.properties`
*   `/etc/mysql/my.cnf`
*   `/etc/nginx/nginx.conf`
*   `/etc/httpd/conf/*`
*   `.pem` or `.key` files
*   backup config files

**FAIL reason:** These files must NOT be readable by “others.”

***

# ✅ **MASTER COMMAND #2 — World‑Writable Files (CRITICAL FAIL)**

> **Meaning:** Anyone on system can modify these → RCE or privilege escalation.

```bash
find / -perm -002 -type f 2>/dev/null
```
### Meaning
This finds all files that are writable by “others.”
-perm -002 means:

0 → “others”

02 → write permission

### ✔ PASS

Output is empty

### ❌ FAIL

If *any* files appear, especially:

*   `/etc/*`
*   `/var/www/*`
*   `/opt/*`
*   `.sh` scripts
*   `.php` files
*   `.py` files
*   configs, logs, binaries

These should **never** be world‑writable.

***

# **MASTER COMMAND #3 — World‑Writable Directories (CRITICAL FAIL)**

> **Meaning:** Attackers can upload/replace files.

```bash
find / -type d -perm -007 2>/dev/null
```
### Meaning
This finds directories where “others” have:
-perm -007:

0 → “others”

07 → rwx (read, write, execute)

### ✔ PASS

Only directories allowed:

*   `/tmp`
*   `/var/tmp`
*   `/dev/shm`

### ❌ FAIL

If you see 777‑permission directories such as:

*   `/var/www/html`
*   `/etc/`
*   `/opt/app/`
*   `/home/app/`
*   Anything except `/tmp`

These allow attackers to add malicious files.

***

# **BACKUP COMMANDS — Recommended but not mandatory**

### 🔍 Find backup/original/dev files (potentially sensitive)

```bash
find / -name "*.bak" -o -name "*.old" -o -name "*.orig" -o -name "*.backup" 2>/dev/null
```

**FAIL IF:**  
These files have weak permissions (644 or worse).

***

# 🟢 **Now here is the FULL PASS/FAIL MAP**

| Test                               | Command                     | PASS                        | FAIL                     |
| ---------------------------------- | --------------------------- | --------------------------- | ------------------------ |
| **World‑readable sensitive files** | `find / -perm -004 -type f` | No sensitive files found    | Sensitive configs appear |
| **World‑writable files**           | `find / -perm -002 -type f` | No files                    | ANY files                |
| **World‑writable directories**     | `find / -type d -perm -007` | Only `/tmp`                 | Any other directory      |
| **Backup files**                   | `find / -name "*.bak" ...`  | Not sensitive or restricted | Sensitive + weak perms   |
| **Config file perms**              | `ls -l /etc/configfiles`    | 600/640/400                 | 644/666                  |

***

# **ALL COMMANDS IN ONE BUNDLE (Copy & Run)**

```bash
# 1. World-readable files
find / -perm -004 -type f 2>/dev/null

# 2. World-writable files
find / -perm -002 -type f 2>/dev/null

# 3. World-writable directories
find / -type d -perm -007 2>/dev/null

# 4. Backup/dev/original files
find / -name "*.bak" -o -name "*.old" -o -name "*.orig" -o -name "*.backup" 2>/dev/null

# 5. Check specific sensitive configs
ls -l /etc/mysql/my.cnf
ls -l /etc/nginx/nginx.conf
ls -l /etc/httpd/conf/httpd.conf
ls -l /etc/ssh/sshd_config
```

# ✅ **1. Linux File Permissions: The Two Formats**

Linux permissions come in **two formats**:

### **A) Numeric (octal) format**

Examples:

    600
    640
    400
    644
    666
    777

### **B) Symbolic (rwx) format**

Examples:

    -rw-------
    -rw-r-----
    -r--------
    -rw-r--r--
    -rw-rw-rw-
    -rwxrwxrwx

They represent the **exact same thing**, just in two different notations.

***

# ✅ **2. How Numeric Permissions Work**

Each permission digit is calculated using:

| Permission  | Value |
| ----------- | ----- |
| Read (r)    | 4     |
| Write (w)   | 2     |
| Execute (x) | 1     |

You add these values to get the permission number.

Examples:

*   `6 = 4 + 2 = rw-`
*   `7 = 4 + 2 + 1 = rwx`
*   `4 = r--`
*   `0 = ---`

***

# 🔥 **3. The Three Permission Groups**

Each file has **3 sets of permissions**:

1.  **Owner**
2.  **Group**
3.  **Others (world)**

Each of the three digits in `600`, `644`, `777`, etc. represents one set:

Example:

    644  
    | | |
    | | └── others  
    | └──── group  
    └────── owner

***

# 🔥 **4. How Numeric Maps to Symbolic**

Let’s break each one down clearly.

***

# 🟢 **600 → `-rw-------`** (SAFE)

600 means:

| Owner   | Group   | Others  |
| ------- | ------- | ------- |
| 6 = rw- | 0 = --- | 0 = --- |

Symbolic:

    -rw-------

✔ Owner can read/write  
✔ Group gets nothing  
✔ Others get nothing

**Ideal for:**  
Secrets, configs, private keys, `.env`, DB credentials.

***

# 🟢 **640 → `-rw-r-----`** (SAFE)

640 means:

| Owner   | Group   | Others  |
| ------- | ------- | ------- |
| 6 = rw- | 4 = r-- | 0 = --- |

Symbolic:

    -rw-r-----

✔ Owner can read/write  
✔ Group can read  
✔ Others get nothing

**Typically safe** for config files that webserver group must read.

***

# 🟢 **400 → `-r--------`** (SAFE)

400 means:

| Owner   | Group   | Others  |
| ------- | ------- | ------- |
| 4 = r-- | 0 = --- | 0 = --- |

Symbolic:

    -r--------

✔ Owner can read  
✔ Nobody else can read/write

**Good for sensitive read-only files**.

***

# 🔴 **644 → `-rw-r--r--`** (FAIL → World‑readable)\*\*

644 means:

| Owner   | Group   | Others  |
| ------- | ------- | ------- |
| 6 = rw- | 4 = r-- | 4 = r-- |

Symbolic:

    -rw-r--r--

❌ Others can read  
❌ World-readable

**BAD for sensitive files:**

*   config.php
*   database.ini
*   .env
*   private keys
*   passwords
*   logs with sensitive info

This is where most audits **fail**.

***

# 🔴 **666 → `-rw-rw-rw-`** (FAIL → World‑writable)\*\*

666 means:

| Owner   | Group   | Others  |
| ------- | ------- | ------- |
| 6 = rw- | 6 = rw- | 6 = rw- |

Symbolic:

    -rw-rw-rw-

❌ Anyone can modify the file  
❌ Serious security risk  
❌ Allows backdoors and privilege escalation

***

# 🔴 **777 → `-rwxrwxrwx`** (FAIL → World everything)\*\*

777 means:

| Owner   | Group   | Others  |
| ------- | ------- | ------- |
| 7 = rwx | 7 = rwx | 7 = rwx |

Symbolic:

    -rwxrwxrwx

❌ Anyone can read  
❌ Anyone can write  
❌ Anyone can execute

**Only acceptable for:**

*   `/tmp`
*   `/var/tmp`
*   `/dev/shm`

Everything else = **CRITICAL FAIL**.

***

# 🟩 **Quick Visual Cheat-Sheet**

| Numeric | Symbolic     | Meaning           | PASS/FAIL |
| ------- | ------------ | ----------------- | --------- |
| **600** | `-rw-------` | owner rw          | 🟢 PASS   |
| **640** | `-rw-r-----` | owner rw, group r | 🟢 PASS   |
| **400** | `-r--------` | owner r only      | 🟢 PASS   |
| **644** | `-rw-r--r--` | world-readable    | 🔴 FAIL   |
| **666** | `-rw-rw-rw-` | world-writable    | 🔴 FAIL   |
| **777** | `-rwxrwxrwx` | world full access | 🔴 FAIL   |

***

# 🧪 **How to check any file's permissions**

Use:

```bash
ls -l /path/to/file
```

Example output:

    -rw-r--r-- 1 root root 1200 Jan  5 12:01 config.php

Break it down:

    -rw-r--r--
     |  |  |
     |  |  └── others = r--  (4)
     |  └────── group = r--  (4)
     └────────── owner = rw- (6)

So this is **644 → FAIL**.

***
