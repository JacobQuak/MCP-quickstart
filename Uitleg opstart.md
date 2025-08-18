## 🚧 Basis MCP-server opzetten in Cursor (stap-voor-stap)

In deze sectie maak je in **Cursor** een minimale MCP-server die via **stdio** draait. We gebruiken een simpele dispatcher en testen alles met **`mcp inspector`** uit `mcp[cli]`.

### 1) Project & venv (UV) in Terminal
```bash
# Nieuw project
mkdir my-mcp && cd my-mcp

# Virtual environment met UV
uv venv
source .venv/bin/activate

# MCP CLI installeren (incl. inspector)
uv add "mcp[cli]>=1.9.4"
```

> 💡 Je kunt dit ook binnen de ingebouwde terminal van Cursor doen.

### 2) Minimale projectstructuur aanmaken (in Cursor)

Maak onderstaande bestanden/mappen in Cursor:

## 📂 Projectstructuur

```text
my-mcp/
├─ .venv/                   # (aangemaakt door UV)
├─ src/
│  ├─ __init__.py
│  ├─ server.py             # entrypoint voor je MCP-server (stdio)
│  └─ tools/
│     ├─ __init__.py
│     └─ dev_hello.py       # voorbeeldtool
├─ pyproject.toml
└─ README.md (optioneel)
