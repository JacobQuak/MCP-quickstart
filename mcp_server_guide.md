# MCP-server met UV + Cursor

## 🎯 Doel
In dit project initialiseren we een **MCP-server** (Model Context Protocol), werken we met **UV** (package & venv manager), en genereren we **Python tools** die je vanuit Cursor AI/Dev kunt bouwen en testen. Tot slot koppelen we de server via de desktop-app (“Cloud AI”/Claude Desktop) door de **Developer Config** te bewerken.

---

## 📦 Vereisten
- macOS of Linux (Windows kan via WSL2)
- Python 3.10+ geïnstalleerd
- **UV** geïnstalleerd (snelste Python package/venv manager)
- Cursor AI (of je editor naar keuze)
- (Optioneel) Claude/Cloud Desktop met MCP-client

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
# (Windows PowerShell: .venv\Scripts\Activate.ps1)

# 5) Basispakketten (pas gerust aan)
uv add uv-add-mcp-cli

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

**Voorbeeld: `src/server.py`**
```python
from typing import Any, Dict
from tools.dev_hello import run as hello_run

def handle_request(payload: Dict[str, Any]) -> Dict[str, Any]:
    tool = payload.get("tool")
    if tool == "hello":
        name = payload.get("name", "world")
        return {"ok": True, "result": hello_run(name)}
    return {"ok": False, "error": f"Unknown tool '{tool}'"}

def run_server() -> None:
    print("MCP server started. Waiting for JSON on stdin. Ctrl+C to stop.")
    import sys, json
    for line in sys.stdin:
        line = line.strip()
        if not line:
            continue
        try:
            payload = json.loads(line)
        except json.JSONDecodeError as e:
            print(json.dumps({"ok": False, "error": f"Invalid JSON: {e}"}))
            continue
        response = handle_request(payload)
        print(json.dumps(response), flush=True)

if __name__ == "__main__":
    run_server()
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
1. Zorg dat je server stdout/stderr logt.
2. Voeg in Cursor een **Tasks/Run Configuration** toe:
   - Command: `python -m src.server`
   - Working dir: projectroot
   - Env: `PYTHONUNBUFFERED=1`
3. Stuur JSON-requests:
   ```bash
   echo '{"tool":"hello"}' | python -m src.server
   ```
4. Gebruik debug-breakpoints in Cursor.

---

## 🖥️ Cloud/Claude Desktop Configuratie
**Voorbeeldconfig:**
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
