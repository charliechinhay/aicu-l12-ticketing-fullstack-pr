# HAND-OFF

## ✅ Handoff: implementazione urgencyLabel e validazione sourceChannel

### 1) Descrizione del problema

L'endpoint `POST /api/tickets` accettava anche `sourceChannel` non validi (es. `"fax"`, `"whatsapp"`) senza restituire errore, e il campo `urgencyLabel` restituito nel ticket creato era sempre `null`.

### 2) Root cause

In `server/index.js` c'erano due gap lasciati come TODO (L12):

- `computeUrgencyLabel(priority, sourceChannel)` era uno stub che ritornava sempre `null`, invece di applicare la regola `priority + sourceChannel -> urgencyLabel` descritta in `template/backend-rules.md`.
- `validateTicketInput` validava `title`, `customer` e `priority`, ma non controllava `sourceChannel` contro `validSourceChannels`, quindi valori non ammessi passavano la validazione.

### 3) Soluzione implementata

- Aggiunta la mappa `urgencyLabelMap` (priority → sourceChannel → label) coerente con la tabella in `template/backend-rules.md`, e `computeUrgencyLabel` ora la consulta (`server/index.js:84-104`).
- Aggiunto il controllo `if (!validSourceChannels.includes(input.sourceChannel)) fieldErrors.sourceChannel = "Canale richiesta non valido."` in `validateTicketInput` (`server/index.js:121-123`).

### 4) Come testare la fix

Con il server avviato su `http://127.0.0.1:4173`:

```bash
# Caso valido: priority=alta, sourceChannel=telefono -> 201, urgencyLabel="intervento rapido"
curl -X POST http://127.0.0.1:4173/api/tickets \
  -H "Content-Type: application/json" \
  -d '{"title":"Test","customer":"ACME","description":"desc","priority":"alta","sourceChannel":"telefono"}'

# Caso invalido: sourceChannel=fax -> 400, fieldErrors.sourceChannel
curl -X POST http://127.0.0.1:4173/api/tickets \
  -H "Content-Type: application/json" \
  -d '{"title":"Test","customer":"ACME","description":"desc","priority":"alta","sourceChannel":"fax"}'
```

Verificato manualmente: il primo caso risponde `201` con `urgencyLabel: "intervento rapido"`; il secondo risponde `400` con `fieldErrors.sourceChannel: "Canale richiesta non valido."`.

### 5) Note aggiuntive

- File modificati: `server/index.js` (uniche modifiche funzionali).
- Nessun impatto su altre parti del sistema: `urgencyLabel` resta calcolato solo lato server (mai accettato dal client, come richiesto in `template/backend-rules.md`), e non ci sono migrazioni di schema (la colonna `urgency_label` esisteva già).
- Riferimenti: `template/backend-rules.md`, `template/payload-cases.md`, `template/request-trace.md` (analisi originale dei due gap).

---
