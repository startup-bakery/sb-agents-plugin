# SB Agents — portable plugin

Formato: [Agent Plugins Standard 1.0](https://agent-plugins.org/).

Questo è il pacchetto portabile di Startup Bakery per client compatibili con
Agent Plugins Standard 1.0. Contiene il bootstrap generico, la connessione al
gateway MCP Render e le regole minime per consegnare il flusso al catalogo
live; non contiene guide di dominio, runtime, migrazioni, credenziali, cookie
o configurazioni dei tenant.

## Struttura

```text
sb-agents/
├── plugin.json       # manifest Agent Plugins Standard
├── .mcp.json         # connessione MCP Streamable HTTP
├── skills/router/    # autenticazione, selezione e caricamento guide live
└── assets/           # icona e loghi Startup Bakery Agents
```

Il gateway configurato è il servizio condiviso SB Agents:
`https://sb-agents-gateway.onrender.com/mcp`. L’endpoint è intenzionalmente
unico: lo standard non definisce fallback automatici tra server o ambienti.
Il server è dichiarato con l'identificatore `sb-agents-production` e il plugin
usa questo endpoint senza fallback automatici.

## Onboarding e utilizzo

Alla prima connessione la pagina OAuth verifica soltanto la persona e i
permessi richiesti. La chat mostra poi gli agenti abilitati per l’account e
richiede la scelta dell’agente. Se esiste un solo tenant compatibile il gateway
lo associa automaticamente; con più tenant, la chat chiede quale usare. Per
compatibilità con i client MCP che memorizzano il primo
`tools/list`, il catalogo operativo viene pubblicizzato dopo l’autenticazione;
le chiamate restano comunque bloccate dal gateway finché agente e tenant non
sono stati selezionati. Le istruzioni specifiche di ciascun agente vengono
caricate dal gateway con `sb_agents_get_agent_guide` e
`sb_agents_get_agent_guide_module`. Quote, provider, permessi, conferme e job
asincroni sono controllati dal gateway, non da questo bundle. Per il Sourcing
Agent, il catalogo live può pubblicizzare un percorso diretto
People → HubSpot (`contact_enrichment_hubspot` e
`contact_enrichment_hubspot_status`) oppure il percorso legacy quote-first.
Quando è presente il percorso diretto, il client deve seguire la guida live,
chiamare `hubspot_import_requirements` per gli owner attivi, ottenere
approvazione per l'azione che può consumare crediti e scrivere nel CRM, quindi
usare la stessa `operation_id` per riprendere un'operazione pending. Non deve
combinare i due percorsi né scegliere il provider. Un aggiornamento della guida
diventa disponibile alle installazioni esistenti alla connessione successiva,
senza rigenerare lo ZIP.

Dopo l’installazione, un client compatibile scopre il router generico e legge
`.mcp.json`. L’autenticazione non è memorizzata nel plugin: viene gestita dal
client e dal flusso OAuth del gateway.

## Compatibilità vendor

Il manifest nella radice è la fonte portabile. Il file
`.codex-plugin/plugin.json` è mantenuto soltanto come adapter Codex; i manifest
vendor-specifici non devono cambiare la semantica delle skill o del gateway.

Per pubblicare questo contenuto nella repository pubblica `sb-agents-plugin`,
copiare il directory `plugins/sb-agents/` senza aggiungere codice runtime o
file `.env`. Ogni release deve aggiornare versione, checksum e origine MCP in
modo atomico.

## Installazione dal marketplace

Codex:

```sh
codex plugin marketplace add startup-bakery/sb-agents-plugin
codex plugin add sb-agents@sb-agents-plugin
```

Claude Code:

```text
/plugin marketplace add startup-bakery/sb-agents-plugin
/plugin install sb-agents@sb-agents-plugin
```

Aprire una nuova conversazione dopo l’installazione e completare l’accesso OAuth.
