## 🚧 Basis MCP-server opzetten in Cursor (stap-voor-stap)

In deze sectie maak je in **Cursor** een minimale MCP-server die via **stdio** draait. We gebruiken een simpele dispatcher en testen alles met 'mcp inspector`** uit `mcp[cli]`.

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
```
### 3) Tools 
Je kunt nu onder 'dev_hello.py' tools toevoegen. Cursor is hierbij handig aangezien het ondersteuning biedt bij het genereren van juiste code die de tools kunnnen executeren. 

Hierbij is het aan te raden om in markdown een richting per tool aan te geven. Een voorbeeld uit een eerder gemaakte mcp tool:
```text
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
```

## 🔍 MCP Inspector starten

Omdat we `mcp[cli]` gebruiken, open je de Inspector met:

```bash
source .venv/bin/activate
mcp dev
```



