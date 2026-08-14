<h1 align="center">FixIt-Wiki-Helper</h1>

## Project Description

`FixIt-Wiki-Helper` is a modular technical documentation center (knowledge base & runbook repository). This repository is dedicated to recording troubleshooting solutions, system configurations, and operational development guides in a structured, concise, and easily accessible manner.
Each document follows the standard format **Problem /Context → Root Cause → Resolution /Steps**to ensure consistency and efficiency during the debugging and onboarding process.

## Navigation Structure

Documentation is grouped by technical domain. Each category has its own modular folder.


### 🖥️ Virtualization
Configuration and troubleshooting guide for virtualization platforms (VirtualBox, VMware, etc.).\
-[VirtualBox /Ubuntu Desktop](./VirtualBox/ubuntu-desktop/)
\
-[Shared Clipboard Not Working](./VirtualBox/ubuntu-desktop/shared-clipboard.md)\

### 🐧 Linux Systems & Tools
System configuration, package management, permissions, and command-line utilities in a Linux environment.\
-[File & Folder Permission Errors](./Linux-Systems/file-permission-errors.md)

### 🔀 Git & Version Control
Guide to conflict resolution, repository configuration, and workflow branching.\
-[Git General Cheat Sheet](./git/git-general.md)\
-[Resolving Merge Conflicts](./git/resolving-merge-conflicts.md)\
### 🐳 Containerization & Orchestration
Troubleshooting Docker, Docker Compose, and other container orchestrators.\
-[Common Containerization & Orchestration Issues](./Containerization/common-issues.md)\
-[Enabling HTTPS for n8n via Cloudflare Tunnel](./Containerization/n8n-https-cloudflare-tunnel.md)

### 🌐 Networking & Connectivity
Diagnose network problems, DNS, proxy and firewall configuration.\
-[DNS Resolution Issues](./Networking/common-issues.md)

### ⚙️ Dev Environment & Tooling
IDE configuration, shell, environment variables, and dependency management.\
--[Environment Variables & .env Not Loading](./Dev-Environment/env-variables-not-loading.md)

## Contribution Convention

1. **Document Structure:**Every technical `.md` file must contain three core sections:
   -**Problem /Context**
   -**Root Cause**
   -**Resolution /Steps**
2. **File & Folder Naming:**Use `lowercase-kebab-case` (example: `shared-clipboard.md`, `ubuntu-desktop/`).
3. **Markdown Format:**Use tiered headings (`##`, `###`), code blocks for commands, and bullet points for easy-to-scan points.

## Status

The repository is in the initialization stage. Other category folder structures will be added gradually as new documentation requires.

## 👤 Author & Credits
Created with ❤️ by Rinwikun
Distributed under the [MIT License](https://github.com/Rinwikun/FixIt-Wiki-Helper/blob/main/LICENSE).

## ☕ Support the Project
[![Trakteer](https://img.shields.io/badge/Trakteer-Traktir%20Coffee-red?style=for-the-badge&logo=coffee&logoColor=white)](https://trakteer.id/erwin%20wijaya)
[![Saweria](https://img.shields.io/badge/Saweria-Support%20Me-orange?style=for-the-badge&logo=heart&logoColor=white)](https://saweria.co/Rinwikun23)
[![Paypal](https://img.shields.io/badge/Paypal-Support%20Me-orange?style=for-the-badge&logo=heart&logoColor=white)](https://paypal.me/Rinwikun)