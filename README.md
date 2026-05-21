# Lab 19: Concatenating Files with `cat`

**Series:** File Operations & Shell Fundamentals · **Lab 19 of the Novice → RHCA path**  
**Certifications covered:** RHCSA EX200 (config inspection, scripted output), RHCE EX294 (Ansible task output, debug), CKA (manifest review, kubeconfig reads), RHCA building blocks (RH342 troubleshooting, RH358 service configs, RH236 brick logs)  
**Prerequisite:** Labs 05–18 (filesystem navigation, listing, links, deletion, redirection)  
**Time Estimate:** 30–45 minutes  
**Difficulty arc:** Tasks 1–6 foundation · 7–12 practical · 13–17 advanced · 18–20 exam-realistic

---

## 🎯 Objective

By the end of this lab you will use `cat` like a senior engineer — not as a "print the file" toy, but as the universal glue between files, pipes, redirection, and heredocs. You'll know when `cat` is the right tool, when it's a textbook misuse (the famous "useless use of cat"), and how to read what `cat` shows you down to the invisible characters that break configs.

---

## 🧠 Concept: `cat` Means **Con**catenate

`cat` was named for **con**catenate — its original job is to stitch multiple files into one output stream. Printing a single file to the terminal is just the degenerate case where you give it one input.

```
cat file1 file2 file3
   │
   └── reads each file in order → writes their bytes to STDOUT
        └── STDOUT defaults to your terminal but can be redirected
```

### The three faces of `cat`

| Mode | Example | What it does |
|---|---|---|
| **Read** | `cat /etc/hostname` | Stream a file to your terminal |
| **Concatenate** | `cat part1 part2 > full.iso` | Glue multiple files into one |
| **Heredoc / stdin** | `cat <<'EOF' > file` | Capture multi-line input into a file |

> **Rule of thumb:** Use `cat` only when you need to combine files or show short text. For anything longer than a screen, switch to `less` (Lab 20). For pattern searches, `grep` (Lab 22) — never `cat file \| grep`.

---

## 📚 Command Reference

| Command / flag | Purpose | When to reach for it |
|---|---|---|
| `cat FILE` | Print file to STDOUT | Short config files, quick peeks |
| `cat F1 F2 F3` | Concatenate in order | Joining log fragments, multipart downloads |
| `cat -n` | Number **every** output line | Auditing line numbers for `sed`/`vi` jumps |
| `cat -b` | Number only **non-blank** lines | Cleaner numbering for code |
| `cat -A` | Show **A**ll non-printable chars (`-vET` combined) | Detecting CRLF, tabs, trailing spaces |
| `cat -E` | Show line **E**ndings as `$` | Spotting trailing whitespace |
| `cat -T` | Show **T**abs as `^I` | Makefile / YAML indentation debugging |
| `cat -v` | Show non-printing chars (caret/M- notation) | Detecting binary garbage in "text" |
| `cat -s` | **S**queeze multiple blank lines into one | Cleaning verbose log output |
| `cat <<EOF` | Heredoc — read until delimiter | Multi-line scripted file creation |
| `cat <<-EOF` | Heredoc, strip leading tabs | Indented heredocs inside functions |
| `cat <<'EOF'` | Heredoc with **quoted** delimiter | Disable `$variable` expansion |
| `cat -` or `cat` (no args) | Read from STDIN | Pipes: `command \| cat -n` |
| `tac FILE` | `cat` spelled backward — reverse line order | Reading log files newest-first |

---

## 🛣️ RHCA Pathway Sidebar

| Cert level | Why this lab matters |
|---|---|
| **Foundation** | First reflex after `ls -l` is `cat` to read the file |
| **RHCSA EX200** | Every "verify the contents of /etc/X" sub-task uses `cat` |
| **RHCE EX294** | `cat` short Ansible facts files, kickstart files, vault templates |
| **CKA** | `cat ~/.kube/config`, `cat /etc/kubernetes/manifests/*.yaml` |
| **RHCA — RH342 (Troubleshooting)** | `cat -A` exposes invisible characters that broke configs |
| **RHCA — RH358 (Services)** | Heredoc creates reproducible service unit files in scripts |
| **RHCA — RH236 (Storage)** | Concatenate split brick logs for forensic timelines |

---

## 🔧 The 20 Tasks

> Each task ends with three short callouts: **Switches** (every flag explained), **Output decoded** (every line/column explained), and **Troubleshoot** (what to do if it goes wrong).

---

### Task 1 — Read a short file with plain `cat`

**Purpose:** The everyday job — get the contents of a short text file onto your screen. `cat /etc/hostname` is a reflex.

```bash
cat /etc/hostname
```

**Expected output:**

```
ip-172-31-45-12.ec2.internal
```

**Switches**

| Token | Meaning |
|---|---|
| `cat` | The command — short for **con**catenate |
| `/etc/hostname` | Absolute path to a one-line text file holding the system's hostname |

**Output decoded**

| Line | What it tells you |
|---|---|
| `ip-172-31-45-12.ec2.internal` | The system's static hostname (set by cloud-init on AWS RHEL images) |

**Troubleshoot**

| Symptom | Fix |
|---|---|
| `cat: /etc/hostname: No such file or directory` | Some minimal containers omit it — try `hostname` instead |
| Output is blank | The file exists but is empty — confirm with `ls -l /etc/hostname` |
| `Permission denied` | Rare on `/etc/hostname` but real on `/etc/shadow` — use `sudo cat` |

---

### Task 2 — Inspect a multi-line config file

**Purpose:** Real configs span dozens of lines. `cat` shows everything in one shot — but only do this when you trust the file is short.

```bash
cat /etc/os-release
```

**Expected output (truncated):**

```
NAME="Red Hat Enterprise Linux"
VERSION="9.4 (Plow)"
ID="rhel"
ID_LIKE="fedora"
VERSION_ID="9.4"
PLATFORM_ID="platform:el9"
PRETTY_NAME="Red Hat Enterprise Linux 9.4 (Plow)"
ANSI_COLOR="0;31"
HOME_URL="https://www.redhat.com/"
BUG_REPORT_URL="https://bugzilla.redhat.com/"
```

**Switches**

| Token | Meaning |
|---|---|
| `cat` | Print to STDOUT |
| `/etc/os-release` | Standard release-info file on all systemd-based distros |

**Output decoded**

| Line | What it tells you |
|---|---|
| `NAME=` | Vendor's marketing name |
| `VERSION=` | Major.minor + codename |
| `ID=` | Short identifier scripts key off (`rhel`, `centos`, `ubuntu`) |
| `ID_LIKE=` | Parent family — useful for "if-rhel-like" branches |
| `PLATFORM_ID=` | Modulemd platform tag |

**Why a sysadmin needs this:** Ansible's `ansible_facts['distribution']` and shell scripts both rely on `/etc/os-release` — read it directly when Ansible is unavailable.

**Troubleshoot**

| Symptom | Fix |
|---|---|
| Output looks like one giant line | Terminal not wrapping — your file is fine, your terminal isn't |
| File doesn't exist | Very old or non-systemd distro — try `/etc/redhat-release` |

---

### Task 3 — Concatenate two files into a third

**Purpose:** This is `cat`'s namesake job. Build two halves, then glue them.

```bash
cd ~
echo "Line A" > part1.txt
echo "Line B" > part2.txt
cat part1.txt part2.txt > combined.txt
cat combined.txt
```

**Expected output:**

```
Line A
Line B
```

**Switches**

| Token | Meaning |
|---|---|
| `cat part1.txt part2.txt` | Read both files, in argument order, write to STDOUT |
| `> combined.txt` | Redirect STDOUT to a new file (truncate first if it exists) |

**Output decoded**

| Line | What it tells you |
|---|---|
| `Line A` | First file's content, in file-argument order |
| `Line B` | Second file's content appended right after |

**Why a sysadmin needs this:** Split-then-recombine patterns appear in: multipart `tar` archives, signed package fragments, ISO images downloaded in pieces, and SOS report bundles.

**Troubleshoot**

| Symptom | Fix |
|---|---|
| Output runs together (no newline between files) | The first file had no trailing newline — fix with `echo >> part1.txt` |
| `combined.txt` is empty | You used `>` on a file that's also being read — never `cat a > a` |

---

### Task 4 — Number every line with `-n`

**Purpose:** Line numbers matter for editor jumps (`vi +42 file`), `sed` ranges, and `diff` output.

```bash
cat -n /etc/hosts
```

**Expected output:**

```
     1	127.0.0.1   localhost localhost.localdomain
     2	::1         localhost localhost.localdomain
     3	
     4	172.31.45.12 ip-172-31-45-12.ec2.internal
```

**Switches**

| Flag | Meaning |
|---|---|
| `-n` | **N**umber all output lines (including blank ones) |

**Output decoded**

| Column | Meaning |
|---|---|
| Right-aligned number | The line number `cat` assigned |
| Tab | Default separator between number and content |
| Content | The actual line from the file |
| Blank-line numbering | `-n` numbers blank lines too — use `-b` to skip them |

**Why a sysadmin needs this on RHCSA:** A task says "comment line 14 of `/etc/X`." `cat -n` is the fastest verification before opening `vi`.

**Troubleshoot**

| Symptom | Fix |
|---|---|
| Numbers reset between files | Expected — each `cat` invocation starts at 1 |
| Want continuous numbering across files | Use `cat F1 F2 \| cat -n` (single cat numbers the concatenated stream) |

---

### Task 5 — Number only non-blank lines with `-b`

**Purpose:** When auditing code or configs, blank lines are noise. `-b` numbers content only.

```bash
printf 'first\n\nsecond\n\n\nthird\n' > ~/sample.txt
cat -b ~/sample.txt
```

**Expected output:**

```
     1	first

     2	second


     3	third
```

**Switches**

| Flag | Meaning |
|---|---|
| `-b` | Number only non-blank lines (overrides `-n` if both given) |

**Output decoded**

| Token | Meaning |
|---|---|
| `1`, `2`, `3` | Sequential numbers for non-blank lines only |
| Blank rows (no number) | Preserved in output but not counted |

**Troubleshoot**

| Symptom | Fix |
|---|---|
| Both `-n` and `-b` used — wrong count | `-b` wins; pick one |
| Lines with only spaces still numbered | Whitespace-only is "non-blank" to `cat` — use `grep -cv '^[[:space:]]*$'` for true blanks |

---

### Task 6 — Reveal hidden characters with `-A`

**Purpose:** "It looks fine but doesn't work" is almost always **invisible** chars: CRLF (Windows line endings), trailing spaces, hard tabs where YAML wants spaces. `-A` is your X-ray.

```bash
printf 'good_line\nbad_line   \r\n\ttab_indented\n' > ~/bad.conf
cat -A ~/bad.conf
```

**Expected output:**

```
good_line$
bad_line   ^M$
^Itab_indented$
```

**Switches**

| Flag | Meaning |
|---|---|
| `-A` | Show **A**ll non-printables (equivalent to `-vET`) |
| `-E` | Show line endings as `$` |
| `-T` | Show tabs as `^I` |
| `-v` | Show other non-printing chars in caret (`^X`) or `M-` notation |

**Output decoded**

| Symbol | Meaning |
|---|---|
| `$` | Newline (`\n`) — every healthy line should end with one |
| `^M` | Carriage return (`\r`) — the Windows-style CRLF problem |
| `^I` | Hard tab — fatal in YAML, sometimes needed in Makefiles |
| `M-` | High-bit (8th bit) set — usually a UTF-8 byte or junk |

**Why a sysadmin needs this on RHCA RH342:** "My Ansible playbook fails with weird parse errors" → 80% of the time, someone edited it on Windows and saved with CRLF. `cat -A` proves it in one command.

**Troubleshoot**

| Symptom | Fix |
|---|---|
| Sea of `^M` markers | Convert with `dos2unix file` (or `sed -i 's/\r$//' file`) |
| `^I` in YAML | Replace tabs with spaces: `expand -t 2 file > file.new && mv file.new file` |

---

### Task 7 — Squeeze blank lines with `-s`

**Purpose:** Some logs dump 20 blank lines between events. `-s` collapses runs of blank lines into a single blank — much more readable.

```bash
printf 'event A\n\n\n\nevent B\n\n\n\n\nevent C\n' > ~/noisy.log
cat -s ~/noisy.log
```

**Expected output:**

```
event A

event B

event C
```

**Switches**

| Flag | Meaning |
|---|---|
| `-s` | **S**queeze consecutive blank lines down to one |

**Output decoded**

| Token | Meaning |
|---|---|
| Single blank between events | `-s` collapsed multiple blanks |
| Content lines | Unchanged |

**Why a sysadmin needs this:** When piping into `less` or `mail`, squeezed output is human-friendly.

**Troubleshoot**

| Symptom | Fix |
|---|---|
| Lines with only spaces still seen as separators | They're not blank to `-s` — pre-process with `sed 's/^[[:space:]]*$//'` |

---

### Task 8 — Reverse a file with `tac`

**Purpose:** `tac` is `cat` spelled backward — it prints the **last line first**, the first line last. Perfect for log files where newest entries are at the bottom.

```bash
seq 1 5 > ~/numbers.txt
cat ~/numbers.txt
echo "---"
tac ~/numbers.txt
```

**Expected output:**

```
1
2
3
4
5
---
5
4
3
2
1
```

**Switches**

| Command | Meaning |
|---|---|
| `tac FILE` | Print lines in reverse order |
| `tac -s SEP` | Use a custom record separator (not just `\n`) |

**Output decoded**

| Block | Meaning |
|---|---|
| First block | Forward order from `cat` |
| Second block | Reverse order from `tac` |

**Why a sysadmin needs this on RHCA RH342:** `tac /var/log/messages \| less` shows the most recent event on screen one — no scrolling.

**Troubleshoot**

| Symptom | Fix |
|---|---|
| `tac: command not found` | Part of `coreutils` — install with `dnf install coreutils` |
| Reversed bytes garbled UTF-8 | Use `tac -s '\n'` explicitly if locale issues appear |

---

### Task 9 — Use `cat` with STDIN (read from a pipe)

**Purpose:** `cat` with no arguments — or with a literal `-` — reads STDIN. This is how you "tap" a pipeline mid-stream.

```bash
echo "hello pipe" | cat
ls /etc | cat -n | head -5
```

**Expected output:**

```
hello pipe
     1	adjtime
     2	alternatives
     3	audit
     4	bash_completion.d
     5	centos-release
```

**Switches**

| Token | Meaning |
|---|---|
| (no args) | `cat` reads STDIN |
| `-` | Literal "read from STDIN here" — useful when mixing files and pipes |
| `\| cat -n` | Number lines from the previous command's output |

**Output decoded**

| Line | Meaning |
|---|---|
| `hello pipe` | Echo's output passed through `cat` unchanged |
| Numbered list | `ls`'s output, numbered by `cat -n`, truncated by `head -5` |

**Why a sysadmin needs this:** Mid-pipeline numbering, mid-pipeline pause (`cat -` doubles as a placeholder during script debugging).

**Troubleshoot**

| Symptom | Fix |
|---|---|
| `cat` hangs forever | You ran it with no args and no pipe — it's waiting for keyboard input. Press `Ctrl-D` |
| `useless use of cat` linter warning | Many `cat F \| cmd` patterns can be replaced with `cmd < F` |

---

### Task 10 — Mix files and STDIN with `-`

**Purpose:** You want to bracket a file's contents with a header or footer from another source. `-` lets you splice STDIN into the middle of a `cat` argument list.

```bash
echo "=== START ===" > ~/header.txt
echo "=== END ===" > ~/footer.txt
echo "BODY FROM PIPE" | cat ~/header.txt - ~/footer.txt
```

**Expected output:**

```
=== START ===
BODY FROM PIPE
=== END ===
```

**Switches**

| Token | Meaning |
|---|---|
| `~/header.txt` | First file argument |
| `-` | Read STDIN at this position |
| `~/footer.txt` | Third argument, appended last |

**Output decoded**

| Line | Meaning |
|---|---|
| `=== START ===` | From `header.txt` |
| `BODY FROM PIPE` | Came in on STDIN via the `echo` pipe |
| `=== END ===` | From `footer.txt` |

**Why a sysadmin needs this:** Building wrapped reports — `cat banner.txt - signature.txt > final.txt` while another command pipes the body.

**Troubleshoot**

| Symptom | Fix |
|---|---|
| `cat` hangs | No pipe is providing STDIN — supply one or remove `-` |

---

### Task 11 — Append a file's contents to another with `>>`

**Purpose:** Unlike `cat A > B` (which **overwrites** B), `cat A >> B` appends. Build composite files safely.

```bash
echo "alpha" > ~/list.txt
echo "beta"  > ~/extra.txt
cat ~/extra.txt >> ~/list.txt
cat ~/list.txt
```

**Expected output:**

```
alpha
beta
```

**Switches**

| Token | Meaning |
|---|---|
| `>>` | Append (create if missing, otherwise add to the end) |
| `>` | Overwrite (create if missing, **truncate** if existing) |

**Output decoded**

| Line | Meaning |
|---|---|
| `alpha` | Original line in `list.txt` |
| `beta` | Appended from `extra.txt` |

**Why a sysadmin needs this:** Building `authorized_keys` from multiple admins' public keys; growing log archives.

**Troubleshoot**

| Symptom | Fix |
|---|---|
| Accidentally used `>` and lost data | If the file was open in another process, `lsof -p PID` may still recover from `/proc/PID/fd/N` |
| Permission denied on `>>` | The shell, not `cat`, opens the file — use `sudo tee -a file` instead |

---

### Task 12 — Create files with a heredoc

**Purpose:** Heredocs are the single most readable way to script multi-line files. Used for: kickstart, systemd units, nginx configs, kubeconfigs, `motd`.

```bash
cat > ~/welcome.txt <<EOF
Welcome to $(hostname)
Today is $(date +%F)
User: $USER
EOF
cat ~/welcome.txt
```

**Expected output:**

```
Welcome to ip-172-31-45-12.ec2.internal
Today is 2026-05-21
User: ec2-user
```

**Switches**

| Token | Meaning |
|---|---|
| `<<EOF` | Heredoc — read from here until a line containing exactly `EOF` |
| `EOF` | The chosen delimiter — any word works (must match start and end) |

**Output decoded**

| Line | Meaning |
|---|---|
| `Welcome to ip-172-…` | `$(hostname)` was expanded by the shell **before** writing |
| `Today is 2026-05-21` | `$(date +%F)` similarly expanded |
| `User: ec2-user` | `$USER` substituted |

**Troubleshoot**

| Symptom | Fix |
|---|---|
| File has literal `$(hostname)` | You quoted the delimiter (`<<'EOF'`) — expansion was disabled |
| Heredoc never ends | The closing `EOF` must be alone on its line, with **no trailing spaces** |

---

### Task 13 — Disable variable expansion with `<<'EOF'`

**Purpose:** When you're writing scripts or templates that **contain** `$variables`, you don't want the shell to expand them. Quoting the delimiter freezes the heredoc.

```bash
cat > ~/template.sh <<'EOF'
#!/bin/bash
echo "Running on $(hostname) as $USER"
EOF
cat ~/template.sh
```

**Expected output:**

```
#!/bin/bash
echo "Running on $(hostname) as $USER"
```

**Switches**

| Token | Meaning |
|---|---|
| `<<'EOF'` | Quoted delimiter — no expansion of `$var`, `$(cmd)`, or `` `cmd` `` |
| `<<EOF` | Unquoted — expand normally |
| `<<\EOF` | Backslash-escape — same effect as quotes |

**Output decoded**

| Line | Meaning |
|---|---|
| `$(hostname)` stays literal | Quoting preserved it for later execution |
| `$USER` stays literal | Will be resolved when the script *runs*, not when it's written |

**Why a sysadmin needs this on RHCE EX294:** Ansible `copy` modules and `template` modules behave the same way — know which expansion happens **now** vs. **later**.

**Troubleshoot**

| Symptom | Fix |
|---|---|
| Script contains literal `$(hostname)` but you wanted runtime expansion | That's actually correct — the script will expand at run time |
| Want some variables expanded, others not | Escape selectively: `\$USER` inside an unquoted heredoc |

---

### Task 14 — Strip leading tabs with `<<-EOF`

**Purpose:** Inside functions and `if` blocks, you want to indent your heredoc with the rest of the code — but those tabs would end up in the file. `<<-EOF` strips them.

```bash
cat > ~/indented.txt <<-EOF
	line one
	line two
		extra-indented
EOF
cat ~/indented.txt
```

**Expected output:**

```
line one
line two
extra-indented
```

**Switches**

| Token | Meaning |
|---|---|
| `<<-EOF` | Strip leading **tabs** (not spaces) from every line including the delimiter |

**Output decoded**

| Token | Meaning |
|---|---|
| `line one` | Tabs at the start removed by `<<-` |
| `extra-indented` | All leading tabs removed — including the deeper indent |

**Why a sysadmin needs this:** Clean shell scripts that build config files inside `case` statements or loops.

**Troubleshoot**

| Symptom | Fix |
|---|---|
| Tabs still in output | Your indents are **spaces**, not tabs — `<<-` only strips tabs |
| Editor converted tabs to spaces | Disable expand-tab in vi: `:set noexpandtab` before saving heredocs |

---

### Task 15 — Concatenate split files with shell globbing

**Purpose:** Tools like `split`, `tar --multi-volume`, and download accelerators produce numbered chunks. `cat F.part*` glues them back.

```bash
cd ~
seq 1 30 | split -l 10 - chunk_
ls chunk_*
cat chunk_* > rebuilt.txt
wc -l rebuilt.txt
```

**Expected output:**

```
chunk_aa  chunk_ab  chunk_ac
30 rebuilt.txt
```

**Switches**

| Token | Meaning |
|---|---|
| `split -l 10 - chunk_` | Split STDIN into 10-line files named `chunk_aa`, `chunk_ab`, … |
| `cat chunk_*` | Shell expands the glob in **alphabetical** order — critical for split files |

**Output decoded**

| Line | Meaning |
|---|---|
| `chunk_aa chunk_ab chunk_ac` | Three chunks, alphabetical = correct order |
| `30 rebuilt.txt` | All 30 lines reassembled |

**Why a sysadmin needs this on RHCA RH236:** GlusterFS brick log fragments must be concatenated in time order — alphabetical naming matters.

**Troubleshoot**

| Symptom | Fix |
|---|---|
| Files out of order | `split` defaults are alphabetical; if you used custom names, sort numerically: `cat $(ls part_* \| sort -V)` |

---

### Task 16 — Use process substitution with `cat`

**Purpose:** When you need to feed multiple **command outputs** (not files) to a tool that only accepts files, `<(cmd)` makes a temporary file-like object.

```bash
cat <(ls /etc | head -3) <(ls /var | head -3)
```

**Expected output:**

```
adjtime
alternatives
audit
account
adm
cache
```

**Switches**

| Token | Meaning |
|---|---|
| `<(cmd)` | Process substitution — runs `cmd`, presents its output as a file descriptor like `/dev/fd/63` |
| `cat <(…) <(…)` | Concatenate two command outputs as if they were files |

**Output decoded**

| Lines | Meaning |
|---|---|
| First three | `ls /etc \| head -3` |
| Next three | `ls /var \| head -3` |

**Why a sysadmin needs this on RHCE EX294:** Comparing two `ansible-inventory` outputs side-by-side; `diff <(cmd-A) <(cmd-B)` (Lab 23).

**Troubleshoot**

| Symptom | Fix |
|---|---|
| `bash: syntax error near <(` | You're in `sh`, not `bash` — process substitution is bash/zsh/ksh only |

---

### Task 17 — Spot the "Useless Use of Cat" anti-pattern

**Purpose:** Every Linux veteran has seen `cat file \| grep foo`. It works but spawns an extra process and is harder to read. Learn the cleaner alternatives.

```bash
cat /etc/passwd | grep "^root"
grep "^root" /etc/passwd
grep "^root" < /etc/passwd
```

**Expected output:**

```
root:x:0:0:root:/root:/bin/bash
root:x:0:0:root:/root:/bin/bash
root:x:0:0:root:/root:/bin/bash
```

**Switches**

| Variant | Why it's preferred |
|---|---|
| `cat F \| cmd` | Avoid — spawns extra process |
| `cmd F` | Best — most tools accept a filename directly |
| `cmd < F` | Good — shell does the redirection, no extra process |

**Output decoded**

| Block | Meaning |
|---|---|
| All three identical | Same result — `cat` added nothing |

**When `cat F \| cmd` is actually justified**

| Case | Reason |
|---|---|
| Concatenating > 1 file | `cat A B C \| cmd` is fine — that's `cat`'s job |
| Tool refuses filenames | Rare, but `cat F \| cmd` is then a workaround |
| Readability in a teaching script | Sometimes left-to-right reads clearer for learners |

**Troubleshoot**

| Symptom | Fix |
|---|---|
| Linter (`shellcheck`) flags UUOC (SC2002) | Rewrite as `cmd F` or `cmd < F` |

---

### Task 18 — Build a multi-file report with `cat` + `awk` framing

**Purpose:** Wrap several files with headers showing where each section came from — a common ops-report pattern.

```bash
cd ~
for f in /etc/hostname /etc/os-release /etc/timezone; do
  test -f "$f" || continue
  echo "===== $f ====="
  cat "$f"
  echo
done > ~/system-report.txt
cat ~/system-report.txt
```

**Expected output (truncated):**

```
===== /etc/hostname =====
ip-172-31-45-12.ec2.internal

===== /etc/os-release =====
NAME="Red Hat Enterprise Linux"
VERSION="9.4 (Plow)"
…
```

**Switches**

| Token | Meaning |
|---|---|
| `for f in …` | Iterate over file paths |
| `test -f "$f" \|\| continue` | Skip files that don't exist (handles `/etc/timezone` missing on some systems) |
| `echo "===== $f ====="` | Section header |
| `cat "$f"` | File body |
| `> ~/system-report.txt` | Redirect the **whole loop's** STDOUT into one report |

**Output decoded**

| Block | Meaning |
|---|---|
| Header lines | Generated by `echo` inside the loop |
| File contents | From each `cat` |
| Blank line | Separator from the trailing `echo` |

**Why a sysadmin needs this on RHCA RH342:** Building "sos-style" snapshots when you don't have `sosreport` available.

**Troubleshoot**

| Symptom | Fix |
|---|---|
| Report is empty | The redirect must wrap the **whole loop**, not just one command |
| Missing files cause errors | The `test -f` guard skips them silently |

---

### Task 19 — Generate a self-contained installer script with a heredoc

**Purpose:** Heredocs let one script create *another* file with full content embedded. Critical for kickstart, cloud-init, and ad-hoc installers.

```bash
cat > ~/deploy.sh <<'EOF'
#!/bin/bash
set -euo pipefail
echo "[deploy] starting on $(hostname) at $(date -Is)"
cat > /tmp/banner.txt <<INNER
This server is managed.
Contact: ops@example.com
INNER
echo "[deploy] banner installed at /tmp/banner.txt"
EOF
chmod +x ~/deploy.sh
~/deploy.sh
cat /tmp/banner.txt
```

**Expected output:**

```
[deploy] starting on ip-172-31-45-12.ec2.internal at 2026-05-21T14:39:00+00:00
[deploy] banner installed at /tmp/banner.txt
This server is managed.
Contact: ops@example.com
```

**Switches**

| Token | Meaning |
|---|---|
| `<<'EOF'` | Outer heredoc, **quoted** — preserve `$(hostname)` etc. for runtime |
| `<<INNER` | Inner heredoc inside the generated script — uses a different word to avoid collision |
| `set -euo pipefail` | Strict mode — exit on errors, undefined vars, or pipe failures |

**Output decoded**

| Line | Meaning |
|---|---|
| `[deploy] starting on …` | Shell expanded `$(hostname)` and `$(date)` at run-time, not write-time |
| Banner text | From the inner heredoc inside the generated script |

**Why a sysadmin needs this on RHCE EX294:** Ansible playbooks frequently embed full file templates via `copy: content:` — same mental model.

**Troubleshoot**

| Symptom | Fix |
|---|---|
| Inner heredoc collides with outer | Use distinct words: `EOF` outside, `INNER` inside |
| Generated script has weird `$` literals | Re-check whether your outer heredoc was quoted vs. unquoted |

---

### Task 20 — Exam-style scenario: produce an evidence report

**Task statement (RHCSA/RH342-style):** *"Capture an evidence snapshot of `/etc/passwd` (numbered lines), `/etc/hostname`, and the contents of `~/.ssh/authorized_keys` (if it exists) into `~/evidence.txt`. The report must have a header line containing the hostname and ISO date."*

```bash
{
  echo "=== Evidence from $(hostname) at $(date -Is) ==="
  echo
  echo "--- /etc/passwd (numbered) ---"
  cat -n /etc/passwd
  echo
  echo "--- /etc/hostname ---"
  cat /etc/hostname
  echo
  echo "--- ~/.ssh/authorized_keys ---"
  if [ -f ~/.ssh/authorized_keys ]; then
    cat ~/.ssh/authorized_keys
  else
    echo "(no authorized_keys file present)"
  fi
} > ~/evidence.txt

wc -l ~/evidence.txt
head -5 ~/evidence.txt
```

**Expected output:**

```
54 /home/ec2-user/evidence.txt
=== Evidence from ip-172-31-45-12.ec2.internal at 2026-05-21T14:39:00+00:00 ===

--- /etc/passwd (numbered) ---
     1	root:x:0:0:root:/root:/bin/bash
     2	bin:x:1:1:bin:/bin:/sbin/nologin
```

**Step-by-step rationale**

| Step | Why |
|---|---|
| `{ … } > ~/evidence.txt` | Group all commands so a single redirection captures everything |
| Header `=== … ===` | Self-documenting — tells future-you what and when |
| `cat -n /etc/passwd` | Numbered for cross-reference with `vipw`/audit |
| Plain `cat /etc/hostname` | One-liner — no numbering needed |
| `if [ -f … ]` guard | Avoid `cat: No such file` noise on systems without that key file |
| `wc -l` verification | Trust-but-verify; confirms report has content |
| `head -5` | Quick visual sanity check |

**Output decoded**

| Line | Meaning |
|---|---|
| `54 /home/.../evidence.txt` | The report has 54 lines — non-empty, real content |
| First five lines | Header, blank, section heading, first two passwd lines |

**Troubleshoot**

| Symptom | Fix |
|---|---|
| `evidence.txt` is empty | The `{ … }` block didn't capture — re-check the closing brace is on its own line or after `;` |
| `Permission denied` on `/etc/shadow` | You probably tried to include it — needs `sudo cat /etc/shadow` |
| Heredoc-style approach also valid | Yes — but a brace group is cleaner when mixing commands |

---

## 🔍 `cat` Decision Guide

```
Read a SHORT file                  → cat FILE
Read a LONG file                   → less FILE        (see Lab 20)
Find a pattern in a file           → grep PATTERN F   (see Lab 22) — NOT cat F | grep
Number lines                       → cat -n FILE
Reveal invisible chars             → cat -A FILE
Reverse order (newest log first)   → tac FILE
Concatenate files                  → cat A B C > combined
Append one file to another         → cat add.txt >> existing.txt
Build a multi-line file in a script→ cat > F <<'EOF' … EOF
Read from a pipe                   → cmd | cat -n   (or just cmd | next)
```

---

## ✅ Lab Checklist (20 Tasks)

- [ ] 01 Plain `cat` reads a short file
- [ ] 02 `cat` shows a multi-line config (`/etc/os-release`)
- [ ] 03 Concatenate two files with `cat A B > C`
- [ ] 04 `cat -n` numbers every line
- [ ] 05 `cat -b` numbers only non-blank lines
- [ ] 06 `cat -A` reveals CRLF, tabs, hidden chars
- [ ] 07 `cat -s` squeezes blank lines
- [ ] 08 `tac` reverses a file
- [ ] 09 `cat` reads STDIN from a pipe
- [ ] 10 `cat F1 - F2` splices STDIN between files
- [ ] 11 `>>` appends, `>` overwrites — know the difference
- [ ] 12 Heredoc `<<EOF` creates a multi-line file with expansion
- [ ] 13 Quoted heredoc `<<'EOF'` freezes literal `$vars`
- [ ] 14 `<<-EOF` strips leading tabs for indented heredocs
- [ ] 15 Concatenate split files with a glob in alphabetical order
- [ ] 16 Process substitution `<(cmd)` feeds `cat`
- [ ] 17 Recognize and refactor "useless use of cat"
- [ ] 18 Build a labelled multi-file report
- [ ] 19 Generate a self-contained installer with nested heredocs
- [ ] 20 Exam combo: evidence report into a single file

---

## ⚠️ Common Pitfalls

| Mistake | Symptom | Fix |
|---|---|---|
| `cat file > file` (same name) | File becomes empty | Use a different output filename, or `tac file > new && mv new file` |
| `cat` on a huge log | Terminal floods, hard to scroll | Use `less` (Lab 20) or pipe to `head`/`tail` |
| `cat` on a binary file | Garbage characters, terminal corruption | Run `reset` or `stty sane`; check with `file FILE` first |
| Closing heredoc has trailing space | Heredoc never ends | The terminator must be alone with no whitespace |
| Heredoc inside a function loses expansion | `$var` appears literally | Don't quote the delimiter unless you want literals |
| `>` truncated a working config | Lost data | Always preview with `cat -n` first; back up with `cp`; use `sudo install -b` |
| `cat F \| grep …` | Linter warns; extra process | Use `grep PATTERN F` |
| Used `cat -A` on UTF-8 file | Loads of `M-` markers | Normal — those are high-bit UTF-8 bytes, not corruption |

---

## 📌 Exam Strategy

**RHCSA EX200**
- Use `cat -n FILE` to verify "edit line N" tasks before opening `vi`.
- Use heredocs to create kickstart-style files reproducibly: `cat > /etc/X <<'EOF' … EOF`.
- Pair `cat /etc/passwd` and `cat /etc/group` when answering UID/GID questions.

**RHCE EX294 (Ansible)**
- `ansible.builtin.copy` with `content:` block ≈ heredoc to a remote file.
- `cat /var/log/ansible.log` after a play to verify the actual error.

**CKA**
- `cat ~/.kube/config` to confirm current context and clusters.
- `cat /etc/kubernetes/manifests/kube-apiserver.yaml` to read the static pod spec.
- `cat /var/log/pods/<pod>/<container>/0.log` when `kubectl logs` is unavailable.

**RHCA**
- RH342: `cat -A` is the single fastest invisible-character diagnostic.
- RH358: heredocs reproduce service unit files across rebuilds.
- RH236: concatenate brick logs `cat brick*.log > all.log` for forensic timelines.

---

## 🔗 Related Labs

| Lab | Connection |
|---|---|
| Lab 20 — `less`, `more` | What to use when `cat` would flood your terminal |
| Lab 21 — `tail -f` | Live monitoring; `cat` is the static counterpart |
| Lab 22 — `grep` | `grep PATTERN FILE` replaces the `cat \| grep` anti-pattern |
| Lab 23 — `diff` | Often paired with `cat -A` to spot invisible differences |
| Lab 24 — `sed` | Use `cat -n` to find line numbers `sed` should target |
| Lab 26 — `vi` | When `cat` shows you a file needs editing, `vi` makes the change |

---

## 👤 Author

**Kelvin R. Tobias**  
[kelvinintech.com](https://kelvinintech.com) · [GitHub](https://github.com/kelvintechnical) · [LinkedIn](https://www.linkedin.com/in/kelvin-r-tobias-211949219)
