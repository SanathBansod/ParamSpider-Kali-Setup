# Troubleshooting

## `externally-managed-environment`

Cause: Kali/Debian protects system Python packages.

Solution:

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

## `venv/bin/activate` not found

Create the environment:

```bash
python3 -m venv venv
```

## `paramspider.py` not found

Check the current directory:

```bash
pwd
ls
```

Or use the full path:

```bash
python3 /home/kali/ReconTools/ParamSpider/paramspider.py --help
```

## `urllib3.packages.six.moves`

Inspect:

```bash
pip show requests urllib3
```

Documented working versions:

```text
requests 2.31.0
urllib3 1.26.18
```

Fix inside the venv:

```bash
pip install --upgrade "requests==2.31.0" "urllib3==1.26.18"
```
