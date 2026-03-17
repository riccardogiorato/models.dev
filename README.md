<p align="center">
  <a href="https://models.dev">
    <picture>
      <source srcset="./logo-dark.svg" media="(prefers-color-scheme: dark)">
      <source srcset="./logo-light.svg" media="(prefers-color-scheme: light)">
      <img src="./logo-light.svg" alt="Logo di Models.dev">
    </picture>
  </a>
</p>

---

[Models.dev](https://models.dev) è un database open-source completo di specifiche, prezzi e capacità dei modelli AI.

Non esiste un database unico con informazioni su tutti i modelli AI disponibili. Abbiamo avviato Models.dev come un progetto contribuito dalla comunità per affrontare questa necessità. Lo utilizziamo anche internamente in [opencode](https://opencode.ai).

## API

Puoi accedere a questi dati attraverso un'API.

```bash
curl https://models.dev/api.json
```

Usa il campo **Model ID** per fare una ricerca su qualsiasi modello; è l'identificatore usato da [AI SDK](https://ai-sdk.dev/).

### Logos

I loghi dei provider sono disponibili come file SVG:

```bash
curl https://models.dev/logos/{provider}.svg
```

Sostituisci `{provider}` con l'**Provider ID** (es. `anthropic`, `openai`, `google`). Se non abbiamo un logo del provider, viene mostrato un logo predefitato.

## Contributing

I dati sono stored nel repo come file TOML; organizzati per provider e modello. Il logo è stored come SVG. Questo viene usato per generare questa pagina e fornire l'API.

Abbiamo bisogno del tuo aiuto per mantenere i dati aggiornati.

### Aggiungere un Nuovo Modello

Per aggiungere un nuovo modello, inizia verificando se il provider esiste già nella directory `providers/`. Se non esiste, allora:

#### 1. Creare un Provider

Se il provider non è già in `providers/`:

1. Crea una nuova folder in `providers/` con l'ID del provider. Per esempio, `providers/newprovider/`.
2. Aggiungi un `provider.toml` con i dettagli del provider:

   ```toml
   name = "Provider Name"
   npm = "@ai-sdk/provider" # Nome del package AI SDK
   env = ["PROVIDER_API_KEY"] # Chiavi delle variabili ambientali usate per l'autenticazione
   doc = "https://example.com/docs/models" # Link alla documentazione del provider
   ```

   Se il provider non pubblica un package npm ma espone un endpoint compatibile con OpenAI, imposta il campo npm adeguatamente e includi il base URL:

   ```toml
   npm = "@ai-sdk/openai-compatible" # Usa SDK compatibile con OpenAI
   api = "https://api.example.com/v1" # Required con openai-compatible
   ```

#### 2. Aggiungere un Logo (opzionale)

Per aggiungere un logo per il provider:

1. Aggiungi un file `logo.svg` alla directory del provider (es. `providers/newprovider/logo.svg`)
2. Usa il formato SVG senza dimensione o colori fissi - usa `currentColor` per fills/strokes

Esempio di struttura SVG:

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor">
  <!-- Logo paths here -->
</svg>
```

#### 3. Aggiungere una Definizione di Modello

Crea un nuovo file TOML nella directory `models/` del provider dove il filename è l'ID del modello.

Se l'ID del modello contiene `/`, usa subfolders. Per esempio, per l'ID del modello `openai/gpt-5`, crea una folder `openai/` e inserisci un file chiamato `gpt-5.toml` dentro.

```toml
name = "Model Display Name"
attachment = true           # o false - supporta allegati di file
reasoning = false           # o true - supporta reasoning / chain-of-thought
tool_call = true            # o false - supporta tool calling
structured_output = true    # o false - supporta una funzionalità dedicata di output strutturato
temperature = true          # o false - supporta controllo della temperatura
knowledge = "2024-04"       # Data di cutoff della conoscenza
release_date = "2025-02-19" # Data del primo rilascio pubblico
last_updated = "2025-02-19" # Data dell'update più recente
open_weights = true         # o false  - i weights trained del modello sono pubblicamente disponibili

[cost]
input = 3.00                # Costo per milione di token input (USD)
output = 15.00              # Costo per milione di token output (USD)
reasoning = 15.00           # Costo per milione di token reasoning (USD)
cache_read = 0.30           # Costo per milione di token read cached (USD)
cache_write = 3.75          # Costo per milione di token write cached (USD)
input_audio = 1.00          # Costo per milione di token input audio (USD)
output_audio = 10.00        # Costo per milione di token output audio (USD)

[limit]
context = 400_000           # Maximum context window (tokens)
input = 272_000             # Maximum input tokens
output = 8_192              # Maximum output tokens

[modalities]
input = ["text", "image"]   # Supported input modalities
output = ["text"]           # Supported output modalities

[interleaved]
field = "reasoning_content" # Nome del campo interleaved "reasoning_content" o "reasoning_details"
```

#### 4. Submit un Pull Request

1. Fork questo repo
2. Crea un nuovo branch con le tue modifiche
3. Aggiungi i tuoi file provider e/o modello
4. Apri un PR con una descrizione chiara

### Validation

C'è una GitHub Action che automaticamente valida la tua submission contro il nostro schema per garantire:

- Tutti i campi required sono presenti
- I tipi di dati sono corretti
- I valori sono dentro i range accettable
- La sintassi TOML è valida

### Schema Reference

I modelli devono conformarsi allo schema seguente, come definito in `app/schemas.ts`.

**Provider Schema:**

- `name`: String - Display name del provider
- `npm`: String - Nome del package AI SDK
- `env`: String[] - Chiavi delle variabili ambientali usate per l'autenticazione
- `doc`: String - Link alla documentazione del provider
- `api` _(optional)_: String - Endpoint API compatibile con OpenAI. Required solo quando si usa `@ai-sdk/openai-compatible` come package npm

**Model Schema:**

- `name`: String — Display name del modello
- `attachment`: Boolean — Supporta allegati di file
- `reasoning`: Boolean — Supporta reasoning / chain-of-thought
- `tool_call`: Boolean - Supporta tool calling
- `structured_output` _(optional)_: Boolean — Supporta funzionalità di output strutturato
- `temperature` _(optional)_: Boolean — Supporta controllo della temperatura
- `knowledge` _(optional)_: String — Data di cutoff della conoscenza in formato `YYYY-MM` o `YYYY-MM-DD`
- `release_date`: String — Data del primo rilascio pubblico in formato `YYYY-MM` o `YYYY-MM-DD`
- `last_updated`: String — Data dell'update più recente in formato `YYYY-MM` o `YYYY-MM-DD`
- `open_weights`: Boolean - Indica che i weights trained del modello sono pubblicamente disponibili
- `interleaved` _(optional)_: Boolean o Object — Supporta reasoning interleaved. Usa `true` per supporto generale o un object con `field` per specificare il formato
- `interleaved.field`: String — Nome del campo interleaved (`"reasoning_content"` o `"reasoning_details"`)
- `cost.input`: Number — Costo per milione di token input (USD)
- `cost.output`: Number — Costo per milione di token output (USD)
- `cost.reasoning` _(optional)_: Number — Costo per milione di token reasoning (USD)
- `cost.cache_read` _(optional)_: Number — Costo per milione di token read cached (USD)
- `cost.cache_write` _(optional)_: Number — Costo per milione di token write cached (USD)
- `cost.input_audio` _(optional)_: Number — Costo per milione di token input audio, se billed separately (USD)
- `cost.output_audio` _(optional)_: Number — Costo per milione di token output audio, se billed separately (USD)
- `limit.context`: Number — Maximum context window (tokens)
- `limit.input`: Number — Maximum input tokens
- `limit.output`: Number — Maximum output tokens
- `modalities.input`: Array di strings — Supported input modalities (es. ["text", "image", "audio", "video", "pdf"])
- `modalities.output`: Array di strings — Supported output modalities (es. ["text"])
- `status` _(optional)_: String — Status supportato:
  - `alpha` - Indica che il modello è in testing alpha
  - `beta` - Indica che il modello è in testing beta
  - `deprecated` - Indica che il modello non è più served dall'API pubblica del provider

### Examples

Vedi i provider esistenti nella directory `providers/` per riferimento:

- `providers/anthropic/` - Modelli Anthropic Claude
- `providers/openai/` - Modelli OpenAI GPT
- `providers/google/` - Modelli Google Gemini

### Working on frontend

Assicurati di avere [Bun](https://bun.sh/) installato.

```bash
$ bun install
$ cd packages/web
$ bun run dev
```

E aprirà il frontend a http://localhost:3000

### Testing manuale con opencode

Puoi verificare manualmente le modifiche del provider con opencode:

```bash
$ bun install
$ cd packages/web
$ bun run build
$ OPENCODE_MODELS_PATH="dist/_api.json" opencode
```

### Questions?

Apri un issue se hai bisogno di aiuto o hai domande sul contributing.

---

Models.dev è creato dai maintainers di [SST](https://sst.dev).

**Join our community** [Discord](https://sst.dev/discord) | [YouTube](https://www.youtube.com/c/sst-dev) | [X.com](https://x.com/SST_dev)