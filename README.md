# File-Permissions-On-Server

…but **not in one single command**, because each “fail condition” is a *different type of permission problem*.  
So you use **3 master `find` commands** that cover **100% of all FAIL/PASS cases**.

Below I give you:

1.  **The exact commands**
2.  **Why each command is used**
3.  **What output = FAIL**
4.  **What output = PASS**

This is the final cheat‑sheet.

***

# ✅ **MASTER COMMAND #1 — World‑Readable Sensitive Files (FAIL Condition)**

> **Meaning:** These files are readable by *everyone* → secrets can leak.

```bash
find / -perm -004 -type f 2>/dev/null
```
### Meaning
This finds all files on the system that have the permission “readable by others”.
-perm -004 means:

0 → “others”
04 → read permission

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

# ✅ **MASTER COMMAND #3 — World‑Writable Directories (CRITICAL FAIL)**

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

# 🔥 **BONUS COMMANDS — Recommended but not mandatory**

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

# 🎯 **FINAL: ALL COMMANDS IN ONE BUNDLE (Copy & Run)**

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

***

# 🟦 If you want:

Paste your outputs from the 3 master commands and I will:

✔ Identify dangerous files  
✔ Classify severity (Low/Medium/High/Critical)  
✔ Tell you PASS/FAIL  
✔ Give you the audit wording exactly in your corporate format
