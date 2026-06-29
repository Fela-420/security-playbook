# Kali Linux Workstation Cleanup, Optimization & Storage Architecture

**Author:** felix
**Operating System:** Kali Linux (Rolling)
**Date:** 29 June 2026

---

# Executive Summary

This project focused on performing a complete maintenance and optimization of a Kali Linux penetration testing workstation.

After months of installing security tools, Docker containers, Go utilities, Python environments, browsers, and testing applications, the workstation had accumulated a significant amount of duplicate data, cached files, obsolete software, and poorly organized development resources. This resulted in unnecessary storage consumption and made long-term maintenance more difficult.

The objectives of this maintenance session were to:

* Recover wasted disk space.
* Remove obsolete software and duplicate files.
* Organize large development resources.
* Separate operating system files from project data.
* Improve future scalability for penetration testing, bug bounty hunting, and security research.

By the end of the project, the workstation was reorganized into a clean and scalable environment while recovering **more than 60 GB** of storage.

---

# Initial Assessment

The first step was to inspect the filesystem rather than deleting files blindly.

The following commands were used:

```bash
df -h
du -xh /var/lib --max-depth=1 | sort -hr
du -xh ~ --max-depth=1 | sort -hr
```

These commands identified which directories were consuming the largest amount of storage.

Major storage consumers included:

| Directory                 | Approximate Size | Description                                 |
| ------------------------- | ---------------: | ------------------------------------------- |
| Docker                    |            15 GB | Images, containers, build cache and volumes |
| Go Workspace              |             9 GB | Go modules and compiled binaries            |
| OWASP ZAP                 |           4.5 GB | Scan sessions and browser data              |
| Burp Suite                |           3.5 GB | Embedded browsers and extensions            |
| Cache                     |           5.7 GB | Temporary application files                 |
| Pipx Virtual Environments |           7.2 GB | Python tool environments                    |
| Duplicate SecLists        |            ~8 GB | Multiple copies of the same wordlists       |

---

# Docker Removal

Docker was found to occupy approximately **15 GB** inside:

```
/var/lib/docker
```

Although Docker is an essential tool for many penetration testing tasks, it was not required for current work and would later be reinstalled using a better storage configuration.

Docker packages, runtime files, and stored images were removed.

## Why?

Docker continuously accumulates:

* Images
* Containers
* Volumes
* Networks
* Build cache

These files grow rapidly over time. Moving Docker to a dedicated storage partition prevents the operating system partition from filling up in the future.

---

# Cache Cleanup

The entire user cache was removed.

```bash
rm -rf ~/.cache/*
```

## Why?

Application caches contain temporary files that improve loading times but are recreated automatically whenever needed.

Examples include:

* Browser cache
* Package cache
* Icon cache
* Temporary downloads
* Application thumbnails

Removing them is generally safe and can recover several gigabytes of storage.

---

# Duplicate Go Workspace

Two complete Go workspaces existed:

```
~/go
~/Desktop/go
```

Both contained downloaded modules and compiled tools.

The duplicate Desktop workspace was deleted.

## Why?

Maintaining multiple Go workspaces causes:

* Duplicate downloads
* Wasted storage
* Version inconsistencies
* Confusion during development

Keeping a single workspace simplifies maintenance.

---

# Relocating the Go Workspace

Instead of storing Go data inside the home directory, the workspace was relocated to the dedicated storage partition.

Commands used:

```bash
rm -rf ~/go
mkdir -p ~/storage/Go
ln -s ~/storage/Go ~/go
```

## Why?

Go expects its default workspace to be:

```
~/go
```

Creating a symbolic link allows Go to continue functioning normally while physically storing all downloaded modules and binaries on the storage partition.

Advantages include:

* Unlimited growth without filling the operating system partition.
* No configuration changes required.
* Fully compatible with Go tooling.

A test installation of Assetfinder confirmed the new workspace functioned correctly.

---

# Duplicate SecLists

Multiple copies of SecLists were discovered.

Locations included:

* Desktop
* Storage partition
* Wordlists directory

Only one master copy was retained.

## Why?

SecLists is a large collection of payloads and wordlists used by penetration testers.

Keeping multiple copies wastes several gigabytes and complicates updates.

Maintaining one centralized copy reduces storage requirements and simplifies management.

---

# OWASP ZAP Cleanup

The directory:

```
~/.ZAP
```

occupied approximately **4.5 GB**.

The majority consisted of saved scan sessions.

Since ZAP was no longer actively used, these files were removed.

## Why?

Large scan sessions can consume several gigabytes.

The application recreates its configuration automatically when needed.

---

# Burp Suite Review

Burp Suite occupied approximately **3.5 GB**.

Most of the storage consisted of:

* Embedded Chromium browser versions
* Browser profiles
* Extensions
* Temporary browser data

Burp Suite was retained.

## Why?

Burp Suite is the primary web application testing platform used for:

* API security testing
* Web application penetration testing
* PortSwigger Web Security Academy labs
* Bug bounty research

Retaining Burp avoids unnecessary reinstallation while preserving a configured working environment.

---

# Python Virtual Environment Investigation

The Pipx virtual environment for **subwiz** occupied approximately **6.8 GB**.

Further inspection showed that it contained:

* PyTorch
* CUDA runtime libraries
* NVIDIA libraries
* Triton compiler
* Transformer libraries

## Why?

These machine learning dependencies had been installed automatically even though they were unnecessary for normal reconnaissance tasks.

This explained why one Python tool consumed more storage than dozens of others combined.

---

# Software Cleanup

Unused applications were removed, including:

* Maltego

Additional obsolete packages and temporary files were also removed.

## Why?

Removing software that is no longer required:

* Frees disk space
* Reduces update times
* Simplifies system maintenance
* Lowers system complexity

---

# Storage Architecture Redesign

The workstation contains two partitions.

## Operating System Partition

Purpose:

* Kali Linux
* Installed packages
* Burp Suite
* Browsers
* Development tools
* System configuration

This partition should remain relatively small and stable.

---

## Storage Partition

A second partition with approximately **94 GB** available was designated for all large and frequently changing files.

Directory structure:

```
storage/
├── Backups
├── Docker
├── Go
├── HTB
├── Projects
├── Recon
├── VMs
└── Wordlists
```

## Why?

Separating operating system files from project data provides several benefits:

* Easier operating system reinstalls.
* Faster backups.
* Cleaner organization.
* Reduced risk of filling the root filesystem.
* Better scalability for future projects.

---

# Benefits for Penetration Testing

This architecture is particularly useful for cybersecurity work because:

* Docker images can grow without affecting the operating system.
* Virtual machines remain isolated from system files.
* Large wordlists have a permanent storage location.
* Go modules no longer consume root partition space.
* Reconnaissance output remains organized.
* Bug bounty projects become easier to archive and manage.

---

# Results

## Storage Recovered

More than **60 GB** of storage was recovered through:

* Docker removal.
* Duplicate workspace removal.
* Cache cleanup.
* Duplicate SecLists removal.
* OWASP ZAP cleanup.
* Software removal.
* Python environment cleanup.
* Miscellaneous obsolete files.

## Root Partition

Before maintenance:

```
Used: 65 GB
Available: 60 GB
```

After maintenance:

```
Used: 49 GB
Available: 76 GB
```

The operating system partition is now significantly healthier and has ample free space for updates and future software installations.

---

# Lessons Learned

This maintenance session reinforced several important principles:

1. Always inspect disk usage before deleting files.
2. Duplicate development environments waste significant storage.
3. Docker should be stored outside the operating system partition whenever possible.
4. Application caches should be cleaned periodically.
5. A single, centralized copy of large datasets is easier to maintain.
6. Symbolic links provide a simple method for relocating development workspaces.
7. Separating operating system files from project data creates a cleaner, more scalable workstation.
8. Regular maintenance prevents storage-related issues and improves long-term system reliability.

---

# Conclusion

The Kali Linux workstation was successfully transformed into a clean, scalable, and professional penetration testing environment.

By recovering more than **60 GB** of storage, consolidating duplicate resources, and redesigning the storage architecture, the system is now better suited for long-term use in web application security, API testing, bug bounty hunting, and offensive security research.

The new architecture ensures that future growth—including Docker images, Go modules, virtual machines, wordlists, and reconnaissance data—will be stored on the dedicated storage partition, preserving the stability and performance of the operating system.

This maintenance project demonstrates the importance of disciplined system administration in cybersecurity. A well-organized workstation not only conserves storage but also improves productivity, simplifies backups, and provides a reliable foundation for professional penetration testing activities.
