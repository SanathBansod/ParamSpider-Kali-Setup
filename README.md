# 🔎 ParamSpider — Kali Linux Setup & Troubleshooting

<p align="center">
  <b>Practical cybersecurity lab notes for installing, troubleshooting, and running ParamSpider on Kali Linux.</b>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/OS-Kali%20Linux-557C94?style=for-the-badge&logo=kalilinux&logoColor=white" alt="Kali Linux">
  <img src="https://img.shields.io/badge/Python-3.13.x-3776AB?style=for-the-badge&logo=python&logoColor=white" alt="Python">
  <img src="https://img.shields.io/badge/Environment-venv-2E8B57?style=for-the-badge" alt="venv">
  <img src="https://img.shields.io/badge/Status-Working-2EA44F?style=for-the-badge" alt="Working">
</p>

---

## 📌 About This Project

This repository documents a real-world setup and troubleshooting journey while configuring **ParamSpider** on Kali Linux.

The goal is not only to show the final command, but to preserve the problems encountered along the way, the evidence used to diagnose them, and the fixes that made the tool work.

> **Why document the failures?**  
> Because troubleshooting knowledge is often more valuable than a copy-paste installation command.

---

## 🧭 Quick Navigation

- [Project Overview](#-project-overview)
- [Environment](#-environment)
- [Repository Structure](#-repository-structure)
- [Installation](#-installation)
- [Problems Encountered](#-problems-encountered)
- [Root Cause Analysis](#-root-cause-analysis)
- [Final Working Configuration](#-final-working-configuration)
- [Usage](#-usage)
- [Output Workflow](#-output-workflow)
- [Troubleshooting](#-troubleshooting-cheat-sheet)
- [Lessons Learned](#-lessons-learned)
- [Responsible Use](#️-responsible-use)

---

## 🎯 Project Overview

**ParamSpider** is a parameter-discovery/reconnaissance tool designed to find URLs and parameters from web archives.

This project focuses on:

```text
Installation
     ↓
Dependency management
     ↓
Python environment isolation
     ↓
Error diagnosis
     ↓
Compatibility fix
     ↓
Verification
     ↓
Recon output workflow
```

---

## 🧰 Environment

| Component | Configuration |
|---|---|
| Operating System | Kali Linux |
| Python | 3.13.x |
| Environment | Python `venv` |
| ParamSpider Location | `/home/kali/ReconTools/ParamSpider` |
| Working Output Location | `/home/kali/Hunting` |

---

## 📁 Repository Structure

```text
ParamSpider-Kali-Setup/
│
├── README.md
│
├── docs/
│   ├── SETUP.md
│   ├── TROUBLESHOOTING.md
│   ├── USAGE.md
│   └── screenshots/
│       ├── 01-pip-requirements-error.png
│       ├── 02-venv-created-activated.png
│       ├── 03-old-urllib3-error.png
│       ├── 04-dependency-inspection.png
│       ├── 05-urllib3-packages-check.png
│       ├── 06-import-error-confirmed.png
│       └── 07-dependencies-upgraded.png
│
├── examples/
│   └── README.md
│
└── .gitignore
```

> The actual ParamSpider source repository is kept separate from this documentation repository. This keeps the portfolio/research notes clean and avoids committing the Python virtual environment.

---

# 🚀 Installation

## 1. Clone ParamSpider

```bash
cd /home/kali/ReconTools
git clone https://github.com/0xKayala/ParamSpider.git
cd ParamSpider
```

Verify the repository:

```bash
ls
```

Expected:

```text
core
gf_profiles
static
LICENSE
README.md
paramspider.py
requirements.txt
```

---

## 2. Create a Virtual Environment

Kali's system Python is externally managed, so use an isolated environment:

```bash
python3 -m venv venv
```

Activate:

```bash
source venv/bin/activate
```

Verify:

```bash
which python
```

Expected:

```text
/home/kali/ReconTools/ParamSpider/venv/bin/python
```

---

## 3. Install Dependencies

```bash
pip install -r requirements.txt
```

The repository specifies older dependency versions. On a modern Python environment this can lead to compatibility issues.

---

# 🧨 Problems Encountered

## Problem 1 — `externally-managed-environment`

Initial command:

```bash
pip3 install -r requirements.txt
```

Kali returned:

```text
error: externally-managed-environment
```

### Diagnosis

Kali/Debian protects the system Python environment from arbitrary `pip` modifications.

### Solution

Use:

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### Evidence

![Kali pip error](docs/screenshots/01-pip-requirements-error.png)

---

## Problem 2 — Virtual environment did not exist

Attempting:

```bash
source venv/bin/activate
```

returned:

```text
source: no such file or directory: venv/bin/activate
```

### Solution

Create it first:

```bash
python3 -m venv venv
```

Then:

```bash
source venv/bin/activate
```

### Evidence

![Virtual environment created](docs/screenshots/02-venv-created-activated.png)

---

## Problem 3 — `urllib3.packages.six.moves`

After installing the repository requirements, running:

```bash
python3 paramspider.py --help
```

produced:

```text
ModuleNotFoundError: No module named 'urllib3.packages.six.moves'
```

### Initial dependency state

```text
requests = 2.23.0
urllib3  = 1.25.8
Python   = 3.13.x
```

### Evidence

![Initial urllib3 error](docs/screenshots/03-old-urllib3-error.png)

---

# 🔬 Root Cause Analysis

The traceback showed that ParamSpider was importing the old `urllib3` package from the project's Python 3.13 virtual environment.

The installed package layout was inspected:

```bash
ls -la venv/lib/python3.13/site-packages/urllib3/packages/
```

` six.py ` was present, so the problem was not simply a missing file.

A direct import test reproduced the failure:

```bash
python3 -c "import urllib3.packages.six; print('six OK'); import urllib3.packages.six.moves; print('moves OK')"
```

This confirmed that the old dependency stack was incompatible with the current Python environment.

### Evidence

![Dependency inspection](docs/screenshots/04-dependency-inspection.png)

![urllib3 package inspection](docs/screenshots/05-urllib3-packages-check.png)

![Import failure confirmed](docs/screenshots/06-import-error-confirmed.png)

---

# 🔧 Compatibility Fix

Instead of modifying Kali's system Python, the dependencies inside the project's venv were upgraded:

```bash
pip install --upgrade "requests==2.31.0" "urllib3==1.26.18"
```

The resulting versions:

```text
requests = 2.31.0
urllib3  = 1.26.18
```

### Evidence

![Dependencies upgraded](docs/screenshots/07-dependencies-upgraded.png)

Verify:

```bash
python3 -c "import requests, urllib3; print('requests:', requests.__version__); print('urllib3:', urllib3.__version__)"
```

Expected:

```text
requests: 2.31.0
urllib3: 1.26.18
```

---

# ✅ Final Verification

Run:

```bash
python3 paramspider.py --help
```

A working ParamSpider help menu confirms the environment is operational.

The repository also emitted an old-code `SyntaxWarning` during startup. It was non-fatal because execution continued normally.

---

# ⚡ Final Working Configuration

```text
Kali Linux
    │
    └── ReconTools/
          │
          └── ParamSpider/
                │
                ├── paramspider.py
                ├── requirements.txt
                └── venv/
                      │
                      ├── Python 3.13.x
                      ├── requests 2.31.0
                      └── urllib3 1.26.18
```

Start a session:

```bash
cd /home/kali/ReconTools/ParamSpider
source venv/bin/activate
```

Then:

```bash
python3 paramspider.py --help
```

---

# 🧪 Usage

For a domain that you are explicitly authorized to assess:

```bash
python3 paramspider.py -d example.com
```

Save output:

```bash
python3 paramspider.py -d example.com -o output.txt
```

### Common options

| Option | Purpose |
|---|---|
| `-h` | Show help |
| `-d` | Target domain |
| `-s` | Specify subdomain input |
| `-l` | Set recursion/level |
| `-e` | Exclude extensions |
| `-o` | Save output |
| `-p` | Set parameter placeholder |
| `-q` | Quiet output |
| `-r` | Number of retries |

Always confirm the current repository help output before relying on an option because older tools can change behavior between versions.

---

# 🗂️ Output Workflow

A clean reconnaissance workspace separates tools from generated data:

```text
/home/kali/
│
├── ReconTools/
│   ├── ParamSpider/
│   └── SecLists/
│
└── Hunting/
    ├── urls.txt
    ├── active.txt
    ├── filtered.txt
    └── screenshots/
```

This prevents generated reconnaissance data from cluttering the tool repository.

---

# 🛠️ Troubleshooting Cheat Sheet

### Kali blocks pip

```text
error: externally-managed-environment
```

Use:

```bash
python3 -m venv venv
source venv/bin/activate
```

---

### `venv/bin/activate` does not exist

```bash
python3 -m venv venv
source venv/bin/activate
```

---

### Python cannot find `paramspider.py`

If you are in another directory:

```bash
python3 /home/kali/ReconTools/ParamSpider/paramspider.py --help
```

Or return to the project:

```bash
cd /home/kali/ReconTools/ParamSpider
```

---

### `urllib3.packages.six.moves` error

Check:

```bash
pip show requests urllib3
```

For this documented setup:

```text
requests 2.31.0
urllib3 1.26.18
```

If necessary, reinstall those versions inside the venv:

```bash
pip install --force-reinstall "requests==2.31.0" "urllib3==1.26.18"
```

---

# 🧠 Lessons Learned

### 1. Don't fight Kali's system Python

Use project-specific virtual environments.

### 2. Read the traceback

The traceback revealed exactly which Python environment and dependency were being loaded.

### 3. Old tools can have old dependencies

A successful `pip install` does not automatically mean an old security tool is compatible with a modern Python release.

### 4. Verify assumptions with small tests

Instead of repeatedly reinstalling everything, package inspection and a direct import test isolated the actual problem.

### 5. Keep source and results separate

```text
ReconTools → tools
Hunting    → results
```

This makes a security lab easier to maintain and document.

---

# 🖥️ Evidence / Troubleshooting Timeline

| Stage | Result |
|---|---|
| Clone repository | ✅ Successful |
| System `pip` installation | ❌ Blocked by Kali |
| Create `venv` | ✅ Successful |
| Install requirements | ✅ Successful |
| First ParamSpider execution | ❌ `urllib3` import error |
| Inspect package | 🔎 `six.py` present |
| Upgrade dependencies | ✅ Successful |
| Verify versions | ✅ Successful |
| ParamSpider `--help` | ✅ Working |

---

# ⚠️ Responsible Use

ParamSpider is a reconnaissance tool.

Use it only against:

- Systems you own
- Your own lab environments
- Authorized penetration-testing targets
- Bug-bounty assets explicitly listed as in scope

For bug-bounty work, follow the program's scope, rate limits, automation rules, and disclosure requirements.

---

## 📚 Original Project

ParamSpider source:

`https://github.com/0xKayala/ParamSpider`

This repository is **documentation and learning notes**, not a replacement for the original ParamSpider project.

---

## ⭐ Project Status

**Setup:** ✅ Working  
**Virtual Environment:** ✅ Configured  
**Dependency Issue:** ✅ Resolved  
**ParamSpider Help Test:** ✅ Passed

---

<p align="center">
  <b>Document the failure. Understand the cause. Fix it. Verify it.</b>
</p>
