# Kali Linux Shell Recovery Documentation

## Overview

While performing web security testing, the terminal suddenly stopped recognizing standard Linux commands such as:

* `curl`
* `ls`
* `head`
* `sort`
* `cat`
* `printenv`

Even though these programs were installed, Zsh reported:

```text
zsh: command not found
```

This indicated that the problem was not with the software itself but with the shell environment.

---

# Symptoms

The following commands failed:

```bash
curl
ls
head
sort
cat
```

Example:

```text
zsh: command not found: curl
zsh: command not found: ls
```

Running:

```bash
echo $PATH
```

returned:

```text
/auth
```

instead of the normal system PATH.

---

# Why This Happened

Linux uses the `PATH` environment variable to determine where executable programs are located.

A normal Kali Linux PATH looks similar to:

```text
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

When you type:

```bash
curl
```

the shell searches these directories until it finds:

```text
/usr/bin/curl
```

However, the shell's PATH had somehow become:

```text
/auth
```

Since `/auth` does not contain standard Linux utilities, Zsh could not locate commands such as:

* curl
* ls
* head
* sort
* cat

The commands themselves were never deleted—the shell simply stopped knowing where to find them.

---

# Investigation Process

## Step 1 — Verify PATH

```bash
echo $PATH
```

Output:

```text
/auth
```

This immediately indicated a PATH problem.

---

## Step 2 — Verify the programs still existed

Commands were executed using absolute paths.

Example:

```bash
/usr/bin/curl --version
```

The command executed successfully.

This proved that:

* curl still existed
* /usr/bin still existed
* the filesystem was healthy

Only PATH was incorrect.

---

## Step 3 — Inspect shell configuration

The following files were examined:

```
~/.zshrc
~/.bashrc
~/.profile
~/.zprofile
/etc/profile
/etc/environment
```

None of the system-wide configuration files were corrupt.

---

## Step 4 — Inspect .zshrc

The current `.zshrc` contained only:

```bash
export PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
export PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
```

Its size was only about 150 bytes.

A normal Kali `.zshrc` is several kilobytes long and contains:

* prompt configuration
* aliases
* completion
* colors
* shell initialization

This meant the original `.zshrc` had been overwritten.

---

## Step 5 — Check for a Backup

A backup file was found:

```text
~/.zshrc.backup
```

Its size was approximately:

```text
10 KB
```

indicating it was the original configuration.

---

# Temporary Issue During Troubleshooting

A clean Zsh shell was started using:

```bash
/usr/bin/env -i /usr/bin/zsh -f
```

This intentionally ignores:

* ~/.zshrc
* ~/.zprofile
* ~/.profile

The result was the minimal default prompt:

```text
kali%
```

This shell is useful for debugging because it starts with almost no configuration.

It was initially mistaken for another issue, but it was simply the expected behavior of a clean Zsh session.

---

# Permanent Fix

The original configuration was restored:

```bash
cp ~/.zshrc ~/.zshrc.broken
cp ~/.zshrc.backup ~/.zshrc
source ~/.zshrc
```

Immediately afterwards:

The prompt returned to:

```text
┌──(hollyspirit㉿kali)-[~]
└─$
```

PATH became:

```text
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

---

# Verification

The following commands all worked correctly:

```bash
which curl
which head
which sort

curl --version
head --version
sort --version
ls
```

Output confirmed:

```
/usr/bin/curl
/usr/bin/head
/usr/bin/sort
```

and all utilities executed normally.

---

# Lessons Learned

1. "Command not found" does not necessarily mean software is missing.
2. Always inspect `PATH` before reinstalling packages.
3. Using absolute paths (for example `/usr/bin/curl`) helps determine whether the problem is PATH or a missing executable.
4. Keep backups of important shell configuration files such as `.zshrc`.
5. The command:

```bash
/usr/bin/env -i /usr/bin/zsh -f
```

is an excellent diagnostic tool because it starts Zsh without user configuration.

---

# Preventive Measures

Create a backup after any significant shell customization:

```bash
cp ~/.zshrc ~/.zshrc.backup
```

If the configuration becomes corrupted:

```bash
cp ~/.zshrc.backup ~/.zshrc
source ~/.zshrc
```

will restore the working configuration.

---

# Final Status

* Shell restored successfully.
* Normal Kali prompt restored.
* Correct PATH restored.
* Standard Linux utilities functioning correctly.
* No operating system reinstallation required.
* Development and penetration testing environment fully operational.
