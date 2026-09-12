# Usage Notes

Activate the project environment:

```bash
cd /home/kali/ReconTools/ParamSpider
source venv/bin/activate
```

Show help:

```bash
python3 paramspider.py --help
```

Run against an explicitly authorized domain:

```bash
python3 paramspider.py -d example.com
```

Save output:

```bash
python3 paramspider.py -d example.com -o output.txt
```

Deactivate when finished:

```bash
deactivate
```
