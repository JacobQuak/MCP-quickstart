# README – Werken met MCP & Zelf een MCP‑server bouwen

## Wat is MCP?
**Model Context Protocol (MCP)** is een open protocol dat AI‑clients (zoals Claude Desktop) in staat stelt om op een veilige en gestandaardiseerde manier te communiceren met externe systemen. Denk hierbij aan databases, API’s, interne tools of bestanden. 

Een **MCP‑server** stelt functionaliteit beschikbaar aan AI‑clients via drie bouwstenen:
- **Tools**: functies die de client kan aanroepen met gestructureerde parameters en resultaten.
- **Resources**: leesbare bronnen zoals bestanden, API‑responses of datasets.
- **Prompts**: herbruikbare prompt‑sjablonen die de client kan gebruiken.

## Wanneer gebruik je MCP?
MCP is vooral handig als je:
- AI‑assistenten toegang wilt geven tot **live bedrijfsdata** zonder de volledige data in prompts te zetten.
- **Acties** of **workflows** gecontroleerd en veilig wilt laten uitvoeren door een AI‑agent.
- **Herbruikbare integraties** wilt maken die in meerdere AI‑omgevingen bruikbaar zijn.

Voor kleine, eenmalige scripts is MCP vaak te zwaar; kies dan voor een simpel script, notebook of AI tool.

## Architectuur in het kort
- **Transport**: communicatie verloopt meestal via `stdio` (de server draait als subprocess) of via HTTP/WebSocket.
- **Beschrijving**: een MCP‑server beschrijft zichzelf en geeft door welke tools, resources en prompts beschikbaar zijn.
- **Interacties**: de client roept tools aan of leest resources, waarna de server gestructureerde resultaten terugstuurt.

## Hoe maak je een MCP‑server?
Een MCP‑server kan in elke taal gebouwd worden (bijvoorbeeld Python, Node.js of Go). Het belangrijkste is dat de server:
1. Een **beschrijving** van zichzelf teruggeeft (naam, versie, tools, resources, prompts).
2. **Tools** aanbiedt met duidelijke input‑ en outputschema’s.
3. **Resources** beschikbaar stelt via endpoints of vaste URIs.
4. **Prompts** optioneel meelevert als sjablonen.

## Aan de slag met Cursor en uv
Binnen Qando bouwen we MCP‑servers vaak in **Cursor** (IDE) met gebruik van **uv** (package manager en runner). Dit is de aanbevolen workflow:

### Stappenplan opzet
1. **Nieuwe projectmap** aanmaken in Cursor.
2. **Virtuele omgeving beheren met uv**:
   - `uv init` om een nieuw project te initialiseren.
   - `uv add` om dependencies toe te voegen (bijv. FastAPI, Pydantic, mcp[cli]).
3. **Structuur bepalen**:
   - Een centrale `server.py` of `main.py` voor de FastAPI-app of stdio‑server.
   - Submappen voor tools, resources en schemas.
4. **Basisconfiguratie schrijven**:
   - Beschrijf de server (naam, versie, tools, resources, prompts).
   - Voeg minimaal één tool toe (bijv. een echo‑functie).
5. **Testen vanuit Cursor**:
   - Start de server via `uv run`.
   - Controleer of de beschrijving en tools correct worden weergegeven.
6. **Integratie met client**:
   - Voeg de server toe in de config van Claude Desktop of een andere MCP‑client. Zie hiervoor `uitleg opstart.md`
   - Zorg dat de juiste startcommandos (met `uv run ...`) in de clientconfig staan.
7. **Itereren**:
   - Breid de server uit met meer tools en resources.
   - Voeg validatie, logging en beveiliging toe.

### Waarom Cursor + uv?
- **Cursor** geeft AI‑ondersteuning bij het schrijven van code en documentatie.
- **uv** is lichtgewicht, snel en vervangt traditionele package managers zoals pip/poetry.
- Het maakt afhankelijkheden duidelijk en herhaalbaar, wat belangrijk is voor teamsamenwerking.

## Opzet met Cursor & uv (algemene methode)
Met deze methode kun je generiek elke MCP‑server opzetten in Cursor met `uv`. Het is niet gebonden aan een specifiek project of domein, maar geldt voor alle toekomstige servers die je wilt bouwen.

### 1) Projectinitialisatie
- Maak in Cursor een nieuwe repository of map voor je MCP‑server.
- Initialiseer het project met `uv`, zodat je afhankelijkheden en scripts centraal beheert.
- Leg de projectmetadata en dependencies vast in `pyproject.toml` (bijvoorbeeld `mcp[cli]` en extra libraries voor jouw use case).

### 2) Structuur van de server
- **`main.py`**: bevat de opstart en registratie van je MCP‑server.
- **Modules**: plaats per domein of thema losse bestanden of mappen waarin je tools, resources of prompts definieert.
- Houd serverkern en domeinspecifieke logica gescheiden voor overzicht en herbruikbaarheid.

### 3) Dependencies installeren en draaien
- Gebruik `uv sync` om afhankelijkheden uit `pyproject.toml` te installeren.
- Start de server lokaal met `uv run`, gevolgd door je startcommando (bijvoorbeeld `python main.py`).
- Op deze manier draai je de server direct binnen Cursor zonder aparte virtualenv‑handelingen.

### 4) Aanbieden van MCP‑functionaliteit
- Zorg dat je server zichzelf beschrijft met **capabilities**:
  - **Tools**: acties met duidelijke input en output.
  - **Resources**: leesbronnen zoals datasets of API‑resultaten.
  - **Prompts**: optioneel, vaste prompt‑sjablonen.
- Houd de beschrijvingen generiek, zodat een client zonder voorkennis weet wat er mogelijk is.

### 5) Testen in Cursor
- Start de server in de geïntegreerde terminal van Cursor en controleer dat deze zonder fouten draait.
- Gebruik de MCP CLI (`mcp`) of eenvoudige requests om te verifiëren dat tools/resources correct worden weergegeven.
- Dit helpt om schema’s en responses te valideren voordat je een AI‑client aansluit.

### 6) Koppelen aan een MCP‑client
- Voeg in de configuratie van je AI‑client (zoals Claude Desktop) je server toe met naam, commando en argumenten.
- Omdat je `uv` gebruikt, zal dit vaak het commando `uv run python main.py` zijn.
- Kies transport (stdio of HTTP/WebSocket) passend bij de client en leg dit vast in de README.

### 7) Uitbreiden en onderhouden
- Voeg nieuwe functionaliteit toe door extra modules of mappen aan te maken (bijvoorbeeld voor nieuwe tools of resources).
- Houd dependencies actueel en documenteer wijzigingen in de README.
- Gebruik semantische versies in `pyproject.toml` en een changelog voor transparantie richting clients.

## Best practices
- **Kleine, gerichte tools** werken beter dan grote, complexe functies.
- **Valideer input** strikt en beschrijf altijd een duidelijk schema.
- **Houd resources idempotent** (zonder bijwerkingen).
- **Versiebeheer**: documenteer wijzigingen zodat clients stabiel blijven.
- **Beveiliging**: gebruik API‑keys, logging en toegangslimieten waar nodig.

## Troubleshooting
- **Client ziet geen tools** → controleer of de server een geldige beschrijving teruggeeft.
- **Inputfouten** → zorg dat de aangeleverde data overeenkomt met het input‑schema.
- **Timeouts** → optimaliseer je server of stel een hogere timeout in.
- **Mismatch in transport** → let op of de client `stdio` of HTTP verwacht.
