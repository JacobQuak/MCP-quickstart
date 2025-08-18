# MCP-server met UV + Cursor

## 🎯 Doel
In dit project initialiseren we een **MCP-server** (Model Context Protocol), werken we met **UV** (package & venv manager), en genereren we **Python tools** die je vanuit Cursor AI/Dev kunt bouwen en testen. Tot slot koppelen we de server via de desktop-app (Claude Desktop) door de **Developer Config** te bewerken.

---

## 📦 Vereisten
- macOS of Linux (Windows kan via WSL2)
- Python 3.10+ geïnstalleerd
- **UV** geïnstalleerd (snelste Python package/venv manager)
- Cursor AI (of je editor naar keuze)
- (Optioneel) Claude Desktop met MCP-client

### UV installeren
```bash
# macOS/Linux (officieel installer-script)
curl -LsSf https://astral.sh/uv/install.sh | sh

# check
uv --version
```

---

## 🚀 Snelstart

```bash
# 1) Nieuw project
mkdir mcp-server-demo && cd mcp-server-demo

# 2) Git initialiseren
git init
git branch -m main

# 3) UV venv (in ./.venv)
uv venv

# 4) venv activeren
# macOS/Linux:
source .venv/bin/activate


# 5) Basispakketten (pas gerust aan)
uv add 'mcp[cli]'

# (optioneel)
# uv add mcp mcp-server

# 6) Repo-structuur klaarzetten
mkdir -p src/tools
touch src/__init__.py src/server.py src/tools/__init__.py

# 7) Cursor openen (of je editor)
open .
```

---

## 🧱 Projectstructuur

```
mcp-server-demo/
├─ .venv/                 # UV virtual environment
├─ src/
│  ├─ server.py           # entrypoint MCP-server
│  └─ tools/
│     ├─ __init__.py
│     └─ dev_<toolnaam>.py
├─ README.md
├─ pyproject.toml
└─ .gitignore
```

**.gitignore (voorbeeld):**
```
.venv/
__pycache__/
*.pyc
.idea/
.vscode/
.DS_Store
```

---

## 🛠️ Tools genereren in Cursor (dev workflow)
In Cursor kun je een nieuwe tool aanmaken met je eigen “dev”-commando. Plaats de code van elke tool in `src/tools/dev_<toolnaam>.py`.

**Voorbeeldtool: `src/tools/dev_hello.py`**
```python
def run(name: str = "world") -> str:
    # Eenvoudige voorbeeldtool
    return f"Hello, {name}! 👋"
```

---

## 🧠 MCP-server entrypoint

**Voorbeeld: `cbs.py`**
```python
from __future__ import annotations

from typing import Any, Dict, Optional

import requests
from mcp.server.fastmcp import FastMCP
import mcp
mcp = FastMCP("cbs")

CBS_ODATA_BASE = "https://opendata.cbs.nl/ODataApi/odata"

@mcp.tool()
def get_cpi(period: str) -> str:
    """Convenience wrapper for CPI (inflation) figures.

    Parameters
    ----------
    period:
        Period label understood by dataset ``84649NED``.
        Accepts either a StatLine *Period* code (e.g. ``2024M06``)
        **or** Dutch full‑month description such as ``\"december 2024\"``.

    Returns
    -------
    str
        The CPI index figure for the requested period or an error message.

    Notes
    -----
    * **Dataset:** ``84649NED`` – *Consumer price index (2015 = 100)*.
    * Period codes are documented by CBS and follow formats like
      ``YYYYMmm`` (monthly) or ``YYYY`` (annual).
    """

    dataset = "84649NED"
    # Map Dutch month names to month numbers for user convenience.
    NL_MONTHS = {
        "januari": "01",
        "februari": "02",
        "maart": "03",
        "april": "04",
        "mei": "05",
        "juni": "06",
        "juli": "07",
        "augustus": "08",
        "september": "09",
        "oktober": "10",
        "november": "11",
        "december": "12",
    }

    period_code = period.strip()

    # Convert \"december 2024\" → \"2024M12\" etc.
    if " " in period_code and not period_code.upper().startswith("20"):

        try:
            month_name, year = period_code.lower().split()
            month = NL_MONTHS[month_name]
            period_code = f"{year}M{month}"
        except (ValueError, KeyError):
            # Keep original if conversion fails – will be handled by API.
            pass

    params = {
        "$format": "JSON",
        "$select": "Period,CPI_1",
        "$filter": f"Period eq '{period_code}'",
    }
    url = _build_odata_url(dataset, "TypedDataSet", params)
    payload = _http_get(url)
    rows = payload.get("value", [])

    if not rows:
        return f"Geen CPI‑waarde gevonden voor '{period}'. (Period code: {period_code})"

    cpi = rows[0].get("CPI_1")
    return f"CPI voor {period} ({period_code}): **{cpi}** (2015 = 100)"


if __name__ == "__main__":
    mcp.run()
```

---

## ▶️ Lokaal testen (zonder client)
```bash
source .venv/bin/activate
python -m src.server

# in een tweede shell
echo '{"tool":"hello","name":"Alice"}' | python -m src.server
```

---

## 🔎 Inspector gebruiken
1. Wanneer je mcp[cli] hebt geïnstalleerd in je terminal; typ *mcp dev [NAAM .py bestand]* in je terminal. (Zorg dat alle 
2. Bij de vraag *ok to proceed [y]?* typ y
3. Als je bent geredirect naar de pagina van de inspector, navigeer je naar tools (zie screenshot) 
<img width="1468" height="59" alt="image" src="https://github.com/user-attachments/assets/1ca56b06-12cc-4539-90ff-21350a6aed40" />
4. Klik op list tools. Hier staan alle tools van je gecreëerde MCP server.
5. Selecteer één van je tools en je kunt hem testen in de inspector omgeving.  

---

## 🖥️ Cloud/Claude Desktop Configuratie
**Voorbeeldconfig template:**
```json
{
  "mcpServers": {
    "mcp-server-demo": {
      "command": "/absolute/pad/naar/je/project/.venv/bin/python",
      "args": ["-m", "src.server"],
      "env": {
        "PYTHONUNBUFFERED": "1"
      },
      "enabled": true,
      "autoStart": true,
      "workingDirectory": "/absolute/pad/naar/je/project"
    }
  }
}
```
**Voorbeeldconfig CBS:**
```json
{
  "mcpServers": {
    "cbsmcp": {
      "command": "/Users/jacobquak/CBSMCP/.venv/bin/python3",
      "args": ["/Users/jacobquak/CBSMCP/cbs.py", "--serve"],
      "cwd": "/Users/jacobquak/CBSMCP",
      "env": {
        "PYTHONPATH": "/Users/jacobquak/CBSMCP",
        "PYTHONUNBUFFERED": "1"
      }
    }
  }
}
```
---

## 🧩 Extra tool aanmaken
```bash
touch src/tools/dev_translate.py
```
```python
def run(text: str, to: str = "nl") -> str:
    return f"[{to}] {text}"
```

Koppel in `handle_request`:
```python
from tools.dev_translate import run as translate_run

if tool == "translate":
    return {"ok": True, "result": translate_run(payload.get("text", ""), payload.get("to", "nl"))}
```

---

## 🧪 Snelle testmatrix
```bash
echo '{"tool":"hello","name":"Bob"}' | python -m src.server
echo '{"tool":"translate","text":"Good morning","to":"nl"}' | python -m src.server
```

---

## 🗃️ Optioneel: pyproject.toml
```toml
[project]
name = "mcp-server-demo"
version = "0.1.0"
description = "MCP server demo met UV + Cursor"
requires-python = ">=3.10"

[tool.uv]

[project.scripts]
mcp-server = "src.server:run_server"
```

---

## 🔄 Git-commando’s
```bash
git add .
git commit -m "Init MCP server scaffold (UV + tools + server)"
git remote add origin <repo-url>
git push -u origin main
```

---

## 🧯 Troubleshooting
- **Server start niet:** check `command` pad en of `.venv` geactiveerd is.
- **Modules niet gevonden:** run `source .venv/bin/activate` en installeer opnieuw.
- **Geen tools zichtbaar:** verifieer registratie in MCP-protocol.
- **Logs ontbreken:** zet `PYTHONUNBUFFERED=1`.

---

## 📌 Aanpassingen voor jouw setup
- Vervang `run_server()` door `MCP.run(...)` van jouw MCP-framework.
- Vul je eigen tools in.
- Pas paden en env-variabelen in Desktop config aan.
- Voeg extra instructies toe voor `uv-add-mcp-cli` als nodig.
