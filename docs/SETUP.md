# Setup Guide

This guide contains the clean installation procedure documented in the project README.

## Prerequisites

- Kali Linux
- Python 3
- Git

## Install

```bash
cd /home/kali/ReconTools
git clone https://github.com/0xKayala/ParamSpider.git
cd ParamSpider
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

If the old dependency combination produces an `urllib3.packages.six.moves` import error on modern Python, use the documented compatibility fix:

```bash
pip install --upgrade "requests==2.31.0" "urllib3==1.26.18"
```

Verify:

```bash
python3 paramspider.py --help
```
