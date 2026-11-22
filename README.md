# BurpBambdasGen

> **Natural language in, working Burp Suite Bambdas out.** Describe the filter,
> column, scan check or match-and-replace rule you need, and get Java that
> compiles in Burp's Bambda editor — grounded in 94 real-world examples rather
> than generated from memory.

---

## Overview

**What it does.** BurpBambdasGen turns a plain-language description into a
functional Bambda script for Burp Suite. Bambdas are the Java snippets that
drive Burp's custom filters, table columns, Repeater actions, scan checks and
proxy rewrite rules; writing them means knowing the Montoya API, its null-safety
rules and the return contract of each Bambda type. This tool removes that
barrier.

**How it works.** Every request runs through a retrieval-augmented pipeline.
The tool first infers the Bambda type from the request, then ranks a local
corpus of 94 production Bambdas by keyword overlap and type match, and injects
the three closest examples — full source included — into the prompt alongside
the relevant Montoya API reference. Claude writes the script against those
examples, not against recollection. The output is then checked for the return
contract of its type, balanced delimiters, missing `hasResponse()` guards and
null-safety gaps before it reaches you. Everything runs locally against a JSON
index; no vector database, no external service beyond the Claude API.

**Why it matters for bug hunters.** Custom tooling is what separates a
systematic hunt from a manual one. A filter that surfaces only the requests
carrying a JWT in a cookie, a column that extracts the signing algorithm of
every token in the proxy history, a passive check that flags a specific
misconfiguration across an entire scope — each takes minutes of your attention
instead of an afternoon of API documentation. The cost of asking "could I see
this traffic differently?" drops low enough that you actually ask it.

**Why it matters for organisations.** Security teams encode their own detection
logic — internal header conventions, in-house authentication schemes, the
specific anti-patterns their developers repeat — as reusable Bambdas that every
tester loads on day one. Review knowledge stops living in one engineer's head
and becomes tooling the whole team shares.

**Results.** A described filter or column typically comes back ready to paste.
Generated scripts arrive with Javadoc, null-safe access patterns and a
validation report naming any unmet requirement, and export to `.bambda` (YAML)
for direct import into Burp. The knowledge base is extensible: point the ingest
script at your own `.bambda` collection and the generator starts producing code
in your team's style.

<details>
<summary><strong>Descripción general (español)</strong></summary>

**Qué hace.** BurpBambdasGen convierte una descripción en lenguaje natural en un
script Bambda funcional para Burp Suite. Los Bambdas son los fragmentos de Java
que gobiernan los filtros personalizados, las columnas de tabla, las acciones de
Repeater, los chequeos del scanner y las reglas de reescritura del proxy;
escribirlos exige conocer la API Montoya, sus reglas de null-safety y el
contrato de retorno de cada tipo de Bambda. Esta herramienta elimina esa
barrera.

**Cómo funciona.** Cada pedido atraviesa un pipeline de generación aumentada por
recuperación (RAG). La herramienta infiere primero el tipo de Bambda, luego
ordena un corpus local de 94 Bambdas reales por coincidencia de palabras clave y
de tipo, e inyecta los tres ejemplos más cercanos —con su código completo— en el
prompt junto con la referencia pertinente de la API Montoya. Claude escribe el
script apoyado en esos ejemplos, no en su memoria. La salida se valida después
contra el contrato de retorno de su tipo, el balanceo de delimitadores, la
ausencia de `hasResponse()` y los accesos sin protección de nulos antes de
llegar a tus manos. Todo corre localmente sobre un índice JSON: sin base de
datos vectorial y sin más servicio externo que la API de Claude.

**Para qué sirve a un bug hunter.** El tooling propio es lo que separa una
cacería sistemática de una manual. Un filtro que aísla solo las requests con un
JWT en una cookie, una columna que extrae el algoritmo de firma de cada token
del historial, un chequeo pasivo que marca una mala configuración puntual en
todo el scope: cada uno pasa a costar minutos de atención en lugar de una tarde
de documentación. El costo de preguntarse "¿y si miro este tráfico de otra
forma?" baja lo suficiente como para que efectivamente te lo preguntes.

**Para qué sirve a una empresa.** Los equipos de seguridad codifican su propia
lógica de detección —convenciones internas de headers, esquemas de autenticación
propios, los antipatrones que sus desarrolladores repiten— como Bambdas
reutilizables que cada tester carga desde el primer día. El conocimiento de
revisión deja de vivir en la cabeza de una persona y se convierte en herramienta
compartida.

**Resultados.** Un filtro o una columna descritos suelen volver listos para
pegar. Los scripts generados llegan con Javadoc, patrones de acceso seguros ante
nulos y un informe de validación que nombra cualquier requisito incumplido, y se
exportan a `.bambda` (YAML) para importarlos directo en Burp. La base de
conocimiento es extensible: apuntá el script de ingesta a tu propia colección de
`.bambda` y el generador empieza a producir código con el estilo de tu equipo.

</details>

---

## Pipeline

```
┌─────────────────────────────────────────────────────────────────┐
│  1. YOUR PROMPT                                                 │
│     "filter showing only 403 responses"                         │
└─────────────────────┬───────────────────────────────────────────┘
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│  2. RAG RETRIEVAL (data/bambdas.json)                           │
│     - Ranks stored Bambdas by keyword overlap                   │
│     - Picks the 3 closest examples from the corpus              │
│     - Pulls the matching API documentation                      │
└─────────────────────┬───────────────────────────────────────────┘
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│  3. PROMPT ASSEMBLY                                             │
│                                                                 │
│     SYSTEM: Bambda types, Montoya API, return contracts         │
│                                                                 │
│     USER:                                                       │
│       - Your description                                        │
│       - Detected type (VIEW_FILTER, CUSTOM_ACTION, ...)         │
│       - Relevant documentation                                  │
│       - 3 similar Bambdas, full source included                 │
└─────────────────────┬───────────────────────────────────────────┘
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│  4. CLAUDE API                                                  │
│     - Receives the RAG-enriched prompt                          │
│     - Writes Java grounded in the retrieved examples            │
└─────────────────────┬───────────────────────────────────────────┘
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│  5. OUTPUT                                                      │
│     - Validates the return contract and syntax                  │
│     - Prints the generated code                                 │
│     - Optionally exports to .bambda                             │
└─────────────────────────────────────────────────────────────────┘
```

## Installation

```bash
git clone https://github.com/OliveXSecurity/BurpBambdasGen.git
cd BurpBambdasGen

# Create a virtual environment
python -m venv venv
source venv/bin/activate  # Linux/macOS
# venv\Scripts\activate   # Windows

# Install dependencies
pip install -r requirements.txt

# Configure the API key
echo "ANTHROPIC_API_KEY=your-api-key" > .env
```

## Usage

### 1. Generate Bambdas

The 94 example Bambdas ship with the repository in `data/bambdas.json`, so you
can generate straight away. PortSwigger's documentation is **not** redistributed
here (see [`data/README.md`](data/README.md)); without it the generator still
works, it simply retrieves less context.

**Direct command:**
```bash
python bambda-gen.py generate "filter showing only 403 responses" -t filter
```

**Interactive mode:**
```bash
python bambda-gen.py interactive
```

### 2. Extend the knowledge base (optional)

```bash
python scripts/ingest.py /path/to/knowledge-base
```

Indexes additional `.bambda` files and, if you hold them locally, the Bambda
documentation and the Montoya API reference.

## CLI commands

```bash
python bambda-gen.py --help

Commands:
  generate     Generate a Bambda from a description
  ingest       Ingest documents into the knowledge base
  list         List the stored Bambda examples
  types        Show the available Bambda types
  interactive  Interactive mode
  status       Knowledge base status
```

### Generate options

```bash
python bambda-gen.py generate "description" [OPTIONS]

Options:
  -t, --type       Type: filter, column, action, scan-active, scan-passive
  -o, --output     Write to a file
  -c, --copy       Copy to the clipboard
  -e, --examples   Number of RAG examples to use (default: 3)
```

## Bambda types

| Type | Alias | Returns | Purpose |
|------|-------|---------|---------|
| VIEW_FILTER | filter | boolean | Filter the Proxy/Logger tables |
| CUSTOM_COLUMN | column | String | Custom table column |
| CUSTOM_ACTION | action | void | Repeater action |
| SCAN_CHECK_ACTIVE_* | scan-active | AuditResult | Active scan check |
| SCAN_CHECK_PASSIVE | scan-passive | AuditResult | Passive scan check |
| MATCH_AND_REPLACE_REQUEST | match-request | HttpRequest | Rewrite requests |
| MATCH_AND_REPLACE_RESPONSE | match-response | HttpResponse | Rewrite responses |

## Example prompts

**Filters:**
```bash
python bambda-gen.py generate "show only requests carrying a JWT in a cookie" -t filter
python bambda-gen.py generate "highlight 500 responses in red" -t filter
python bambda-gen.py generate "filter requests containing SQLi patterns" -t filter
```

**Columns:**
```bash
python bambda-gen.py generate "extract the Server header" -t column
python bambda-gen.py generate "show the signing algorithm of the JWT" -t column
python bambda-gen.py generate "show the response time in ms" -t column
```

**Actions:**
```bash
python bambda-gen.py generate "send the request 10 times for a race condition" -t action
python bambda-gen.py generate "decode Base64 in the response" -t action
python bambda-gen.py generate "extract every URL from the body" -t action
```

**Scan checks:**
```bash
python bambda-gen.py generate "detect a missing CSP header" -t scan-passive
python bambda-gen.py generate "test for CORS misconfiguration" -t scan-active
python bambda-gen.py generate "detect SSTI" -t scan-active
```

## Project layout

```
BurpBambdasGen/
├── bambda-gen.py          # CLI entry point
├── requirements.txt       # Dependencies
├── .env                   # API key (create manually)
│
├── src/
│   ├── config/
│   │   ├── settings.py    # Configuration
│   │   └── prompts.py     # System prompt and templates
│   │
│   ├── ingestion/
│   │   ├── loader.py      # Loads .bambda files and docs
│   │   └── chunker.py     # Splits documentation into chunks
│   │
│   ├── retrieval/
│   │   ├── vectorstore.py # JSON store and search
│   │   └── retriever.py   # Retrieval logic
│   │
│   ├── generation/
│   │   ├── generator.py   # Claude API integration
│   │   └── validator.py   # Generated-code validation
│   │
│   └── cli/
│       └── main.py        # CLI application (Typer)
│
├── data/
│   └── bambdas.json       # Example corpus (94 Bambdas)
│
└── scripts/
    └── ingest.py          # Ingestion script
```

## Environment variables

```bash
# .env
ANTHROPIC_API_KEY=sk-ant-...    # Required
MODEL_NAME=claude-opus-5-5      # Claude model
MAX_TOKENS=4096                 # Output token limit
N_EXAMPLES=3                    # Default number of RAG examples
DATA_DIR=data                   # Data directory
```

> Current Claude models reject the `temperature` parameter with HTTP 400, which
> is why the tool does not expose it.

## Why RAG?

Without retrieval, a model writes Bambdas from memory: syntax that almost
compiles, Montoya API calls that almost exist, and return types that do not
match the contract of the Bambda type.

With retrieval, the model sees **real, working Bambdas** of the same type, picks
up the **correct patterns** for each one, references the **official
documentation**, and produces code that holds up in Burp's editor.

## Dependencies

| Package | Purpose |
|---------|---------|
| `anthropic` | Claude API client |
| `typer` + `rich` | Interactive CLI |
| `python-dotenv` | Loads variables from `.env` |
| `pyyaml` | Exports Bambdas in YAML format |
| `pyperclip` | Clipboard support (optional) |

## Requirements

- Python 3.10+
- An Anthropic (Claude) API key

## Tests

```bash
pip install -e ".[dev]"
pytest
```

## Credits

The 94 example Bambdas come from
[PortSwigger's public repository](https://github.com/PortSwigger/bambdas) and
retain their original `@author` in `metadata.author`.

## License

MIT. See [LICENSE](LICENSE).
