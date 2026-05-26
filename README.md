# Lab: Concatenating Files — `cat`, `tac`, `nl`

**Series:** linux-ops-mastery — RHCSA Text File Management
**Subjects covered:** `cat` for concatenation and stdin capture, `tac` for reverse-order concatenation, `nl` for line-numbering, heredocs `<<EOF` for inline file creation, the "useless use of cat" anti-pattern, `cat -A` for showing invisible characters, `cat -n` for numbered output, `cat -s` for collapsing blank lines, how `cat` differs from `cp` semantically, atomic file assembly via `cat … > out.txt`, and the production reflex of "join with `cat`; reverse with `tac`; number with `nl`"
**Career arcs covered:** RHCSA (assemble fragments into one file; "save output of multiple commands to a single artifact"), RHCE (Ansible `assemble:` module is `cat` semantics for fragments), SRE (combining log fragments after rotation), DevOps (combining multi-fragment config files like `cloud-init.d`, `sudoers.d`), AI/MLOps (concatenating training shard logs)
**Prerequisite:** Labs 01, 03
**Time Estimate:** 25 to 35 minutes
**Difficulty arc:** Task 1 foundation (`cat FILE`) · 2 multi-file concatenation · 3 heredocs for file creation · 4 `tac` and `nl` siblings · 5 `cat -A`/`-n`/`-s` modifiers · 6 RHCSA exam-realistic capstone

---

## Objective

Use `cat` to assemble files, write multi-line content quickly, and read invisible characters. By the end you can compose pipeline-friendly file builders, reverse-line streams with `tac`, and reach for `nl` for proper line numbering — and you understand why **useless use of cat** is the meme it is.

The capstone is an exam-realistic prompt: *"Assemble `/etc/issue.d/01-banner.txt`, `/etc/issue.d/02-notice.txt`, and `/etc/issue.d/03-contact.txt` into a single `/etc/issue` using `cat`. Verify the resulting `/etc/issue` matches the concatenation byte-for-byte and contains the expected three section headers."*

> **Lab safety note:** All operations in this lab live in `/tmp/cat-lab` and (briefly) `/etc/issue.d` and `/etc/issue`. The capstone backs up `/etc/issue` first.

---

## Concept: `cat` Sends Files to stdout

`cat FILE` is the simplest possible reader — it opens each argument in order, copies bytes to stdout, and exits. With one argument it prints. With many, it concatenates. With redirection, the output lands in a file.

```
   $ cat file1 file2 file3 > combined.txt
       │   └──────┬──────┘  │
       │          │         └─ Send concatenated stream to file
       │          └─ Read each in argument order
       └─ The reader
```

> **Why this matters:** Every Linux config fragment system (`sudoers.d`, `cron.d`, `cloud-init`, systemd drop-ins) ultimately gets assembled by `cat`-like concatenation. Knowing the primitive is half of understanding the systems.

---

## 📜 Why `cat` Is Named "cat" — The Story

From the very first edition of Unix (1971), `cat` (short for **con**cate**nate**) was the canonical "send these files to stdout" command. Single-file `cat` (just print) is the degenerate case. The original intent was joining files.

The famous **"Useless Use of Cat" award** — coined by Randal Schwartz in the 1990s — pokes fun at `cat file | grep PAT` when `grep PAT file` works just as well. The cat process is wasted; one process is enough. The right tool for "print a file" is usually the command that's about to read it, not `cat`.

> **The point of the story:** `cat` shines for **joining** multiple inputs (or inputs + heredocs). For "just print," let the receiving tool open the file directly.

---

## 👪 The cat Family

| Tool | Purpose |
|---|---|
| `cat F1 F2 ...` | Concatenate to stdout |
| `tac F` | Reverse line order |
| `nl F` | Number non-blank lines |
| `head F` / `tail F` | First/last N lines |
| `cat -n F` | Number all lines (including blanks) |
| `cat -s F` | Squeeze consecutive blank lines |
| `cat -A F` | Show non-printables (tabs as `^I`, EOL as `$`) |
| `cat -E F` | Show only line endings |
| `cat -T F` | Show only tabs |
| `cat > FILE` then Ctrl-D | Capture typed input |
| `cat <<EOF > FILE` | Heredoc — multi-line write |
| `cat <<'EOF' > FILE` | Heredoc, no variable expansion |
| `zcat F.gz` | Cat a gzip-compressed file |

### cat flags

| Flag | Meaning |
|---|---|
| `-n` | Number all lines |
| `-b` | Number only non-blank lines |
| `-s` | Squeeze repeated blank lines |
| `-A` | Show all non-printables |
| `-E` | Show EOL as `$` |
| `-T` | Show tabs as `^I` |
| `-v` | Show non-printables (except tab/EOL) |
| `-u` | Unbuffered output (rare) |

> **The point of the family tree:** `cat` for joining; `tac` for reversing; `nl` for numbering. Use the right one for the job.

---

## 🔬 The Anatomy of `cat -A` — In One Diagram

```
$ printf "a\tb\nc\nd\n\n" > /tmp/x
$ cat -A /tmp/x
a^Ib$
c$
d$
$
└┬┘
 └─ Each line ends with `$` (the EOL marker)
^I = literal tab
$  on its own = blank line (just a newline)
```

> **Reading rule:** `cat -A` makes invisibles visible. Use it when "the file looks empty but `wc -l` returns 3" — usually a Windows CRLF or trailing whitespace issue.

---

## 📚 cat Reference Table

| Task | Command | Notes |
|---|---|---|
| Print one file | `cat FILE` | (Or just `grep ... FILE`) |
| Concatenate | `cat F1 F2 F3 > combined.txt` | Argument order is concatenation order |
| Append | `cat F1 F2 >> log.txt` | |
| Heredoc create | `cat > FILE <<'EOF' ... EOF` | Single-quoted EOF disables `$var` |
| Number all lines | `cat -n FILE` | |
| Number non-blank | `cat -b FILE` | |
| Show invisibles | `cat -A FILE` | Tabs as `^I`, EOL as `$` |
| Squeeze blanks | `cat -s FILE` | Multiple blank lines collapse to one |
| Reverse lines | `tac FILE` | |
| Number with format | `nl -ba FILE` | Numbering control |
| Cat gzip | `zcat FILE.gz` | Decompress + cat |
| Cat to file held open | `cat NEW > /dev/stderr` | Rare |

> **Rule one of cat:** Use `cat` when you have **multiple** inputs (files or heredocs) to join. Use the receiving tool to read **one** file directly when possible.

---

## 🎯 Career Pathway Sidebar

| Level | Why this lab matters |
|---|---|
| **RHCSA candidate** | "Combine these files into one" is a direct exam variant. Heredocs are also useful for creating quick config files. |
| **RHCE candidate** | Ansible's `assemble:` module is the playbook equivalent. |
| **SRE / Platform** | Reassemble rotated logs after an incident; `cat log.1 log` for chronological order. |
| **DevOps** | Build pipelines often `cat` fragments into one config (`sudoers.d` → `/etc/sudoers`-style). |
| **AI / MLOps** | Combine per-shard training logs into one experiment artifact. |

---

## 🔧 The 6 Tasks

> Six exam-realistic phases that build the **fragment → assemble → verify** habit.

---

### Task 1 — Print and concatenate

```bash
mkdir -p /tmp/cat-lab && cd /tmp/cat-lab
echo "alpha"   > a.txt
echo "bravo"   > b.txt
echo "charlie" > c.txt

cat a.txt
cat a.txt b.txt c.txt
cat a.txt b.txt c.txt > combined.txt
cat combined.txt
wc -l combined.txt
```

**Reading it left to right:** Single-file `cat` prints. Multi-file `cat` concatenates in argument order. `>` captures the combined stream.

**Expected output:**

```text
alpha
alpha
bravo
charlie
alpha
bravo
charlie
3 combined.txt
```

---

### Task 2 — Heredoc file creation

```bash
cd /tmp/cat-lab

cat > banner.txt <<'EOF'
==================================
  Welcome to the Linux Lab Host
==================================
EOF
cat banner.txt

# With variable expansion
HOST=$(hostname)
cat > greeting.txt <<EOF
Hello, you are on $HOST today.
EOF
cat greeting.txt
```

**Reading it left to right:** `cat > FILE <<'EOF' ... EOF` writes a heredoc into FILE. Single-quoted `EOF` disables `$var` expansion. Unquoted `EOF` allows expansion.

**Expected output:**

```text
==================================
  Welcome to the Linux Lab Host
==================================
Hello, you are on ip-10-0-0-12 today.
```

---

### Task 3 — `tac` for reverse, `nl` for numbering

```bash
seq 1 5 > nums.txt
cat nums.txt
tac nums.txt
nl nums.txt
nl -ba nums.txt           # number all, including blank lines
nl -nrn -w4 nums.txt      # right-align numbers, width 4
```

**Reading it left to right:** `tac` is `cat` reversed (line-by-line, not byte-by-byte). `nl` produces formatted line numbers; `-ba` numbers blank lines too; `-nrn -w4` is `right-justify-no-leading-zero, width 4`.

**Expected output:**

```text
1
2
3
4
5
5
4
3
2
1
     1  1
     2  2
     3  3
     4  4
     5  5
   1  1
   2  2
   3  3
   4  4
   5  5
```

---

### Task 4 — Show invisible characters with `cat -A`

```bash
printf "alpha\tbravo\r\ncharlie\n\n\ndelta\n" > mixed.txt
cat mixed.txt
cat -A mixed.txt
cat -s mixed.txt           # squeeze blanks
cat -A mixed.txt | head -n 6
```

**Reading it left to right:** `cat -A` exposes tabs (`^I`), Windows `\r` line endings (`^M`), and the EOL marker (`$`). `-s` collapses repeated blank lines into one.

**Expected output:**

```text
alpha	bravo
charlie



delta
alpha^Ibravo^M$
charlie$
$
$
$
delta$
alpha	bravo
charlie

delta
alpha^Ibravo^M$
charlie$
$
$
$
delta$
```

---

### Task 5 — Useless-use-of-cat & where `cat` does shine

```bash
cd /tmp/cat-lab

# Useless — one process wasted
cat combined.txt | grep alpha

# Better
grep alpha combined.txt

# Where `cat` IS necessary — multi-file pipeline
cat a.txt b.txt c.txt | sort > sorted-combined.txt
cat sorted-combined.txt

# Real fragment-assembly use case
mkdir -p fragments
echo "# section 1" > fragments/01.txt
echo "data 1"      >> fragments/01.txt
echo "# section 2" > fragments/02.txt
echo "data 2"      >> fragments/02.txt
echo "# section 3" > fragments/03.txt
echo "data 3"      >> fragments/03.txt

cat fragments/*.txt > assembled.txt
cat assembled.txt
```

**Reading it left to right:** `cat FILE | grep` wastes a process; `grep PAT FILE` is cleaner. But when you have **multiple inputs** to feed a pipeline, `cat` is the right tool. And for fragment assembly (`fragments/*.txt > assembled.txt`), `cat` is the correct primitive.

**Expected output:**

```text
alpha
alpha
alpha
bravo
charlie
# section 1
data 1
# section 2
data 2
# section 3
data 3
```

---

### Task 6 — Capstone: RHCSA-realistic fragment assembly into `/etc/issue`

```bash
sudo -i

mkdir -p /etc/issue.d
cp -a /etc/issue /root/issue.bak.$(date +%F-%H%M)

cat > /etc/issue.d/01-banner.txt <<'EOF'
=========================================
  RHEL 9 - Authorized Access Only
=========================================
EOF

cat > /etc/issue.d/02-notice.txt <<'EOF'
NOTICE: All connections to this system are monitored and logged.
EOF

cat > /etc/issue.d/03-contact.txt <<'EOF'
For support, contact ops@example.com
EOF

cat /etc/issue.d/01-banner.txt /etc/issue.d/02-notice.txt /etc/issue.d/03-contact.txt > /etc/issue

cat /etc/issue
wc -l /etc/issue
grep -c '^=' /etc/issue
grep -c '^NOTICE' /etc/issue

test -s /etc/issue && echo "VERIFY: /etc/issue is non-empty"
diff <(cat /etc/issue.d/0*.txt) /etc/issue && echo "VERIFY: concatenation byte-perfect"
```

**Reading it left to right:** Become root. Build three fragments with heredocs. Concatenate them into `/etc/issue`. Verify size, section headers, and byte-perfect match against the original concatenation (using process substitution `<(...)`).

**Expected verification output:**

```text
=========================================
  RHEL 9 - Authorized Access Only
=========================================
NOTICE: All connections to this system are monitored and logged.
For support, contact ops@example.com
5
2
1
VERIFY: /etc/issue is non-empty
VERIFY: concatenation byte-perfect
```

**Cleanup**

```bash
cp -a /root/issue.bak.* /etc/issue
rm -rf /etc/issue.d
rm -rf /tmp/cat-lab
exit
```

---

## 🔍 cat Decision Guide

```
  ├── "Join several files into one"     → cat F1 F2 F3 > combined
  ├── "Write multi-line file in script" → cat > FILE <<'EOF' ... EOF
  ├── "Just print one file"             → cat FILE  (but consider letting the next tool read FILE directly)
  ├── "Number lines"                    → cat -n FILE  (or nl FILE)
  ├── "Show invisible characters"       → cat -A FILE
  ├── "Squeeze blank lines"             → cat -s FILE
  ├── "Reverse line order"              → tac FILE
  └── "Cat a gzip file"                 → zcat FILE.gz
```

---

## ✅ Lab Checklist (6 Tasks)

- [ ] 01 Print one file; concatenate three files with `>`
- [ ] 02 Heredoc-create files with `cat > FILE <<EOF ... EOF`
- [ ] 03 Reverse with `tac`; number with `nl` and `cat -n`
- [ ] 04 Show invisibles with `cat -A` and squeeze blanks with `cat -s`
- [ ] 05 Avoid useless `cat`; use `cat` for multi-input pipelines and fragment assembly
- [ ] 06 Execute the RHCSA capstone — fragment-assemble `/etc/issue`

---

## ⚠️ Common Pitfalls

| Mistake | Symptom | Fix |
|---|---|---|
| `cat F \| grep` | Wasted process | `grep PAT F` |
| Heredoc with unquoted EOF | `$var` expanded too early | Quote EOF: `<<'EOF'` |
| Files concatenated in wrong order | Argument order matters | Reorder |
| `cat` truncated existing file | Used `>` instead of `>>` | Append mode |
| `cat -A` not on RHEL | Use `cat --show-all` | |
| Windows CRLF in files | `^M$` shown by `cat -A` | `dos2unix` or `sed -i 's/\r$//'` |
| `tac` on huge file slow | Reads from end | Acceptable for normal sizes |
| `nl` numbered nothing | By default `nl` skips blank lines | Add `-ba` |
| `zcat` on plain text | Errors | Use plain `cat` |
| Lost the original by `cat ... > FILE` | Truncate-then-write trap | Back up first |

---

## 🎯 Career & Interview Strategy

**RHCSA candidate**
- Drill heredoc file creation and multi-file `cat ... > out`. Both appear constantly.

**RHCE candidate**
- Ansible `assemble:` module mirrors this; `concat: yes`.

**SRE / Platform interview**
- "Reassemble rotated logs in time order" → `cat $(ls -tr server.log.*) server.log > all.log`.

**DevOps**
- Fragment-assembly is the daily pattern: `cron.d/`, `sudoers.d/`, `nginx/conf.d/`.

**AI / MLOps**
- Combine per-shard training metrics into one CSV.

---

## 🔗 Related Labs

| Lab | Connection |
|---|---|
| Lab 01 — stdout Redirection | `>` and `>>` foundations |
| Lab 03 — Pipe Text Streams | Combine `cat` with pipelines |
| Lab 20 — Scrolling with `less` / `more` | After assembling, page through |
| Lab 21 — `tail -f` Live Logs | Watch as fragments grow |
| Lab 24 — `sed` Stream Editor | When `cat` is not enough |

---

## 👤 Author

**Kelvin R. Tobias**
[kelvinintech.com](https://kelvinintech.com) · [GitHub](https://github.com/kelvintechnical) · [LinkedIn](https://www.linkedin.com/in/kelvin-r-tobias-211949219)
