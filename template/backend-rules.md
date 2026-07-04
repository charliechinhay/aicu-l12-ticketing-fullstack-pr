# Backend Rules - L11

## Valori Validi

### Priority

- alta
- normale
- bassa

### Source Channel

- email
- telefono
- chat

## Mapping

| Priority | Source Channel | Urgency Label     |
| -------- | -------------- | ----------------- |
| alta     | telefono       | intervento rapido |
| alta     | chat           | prioritario       |
| alta     | email          | prioritario       |
| normale  | telefono       | da gestire        |
| normale  | chat           | da gestire        |
| normale  | email          | standard          |
| bassa    | telefono       | monitorare        |
| bassa    | chat           | monitorare        |
| bassa    | email          | monitorare        |

## Dati Decisi Dal Server

- `urgencyLabel`: calcolato da `computeUrgencyLabel(priority, sourceChannel)` secondo la tabella di mapping sopra; non deve mai arrivare dal client.
- `createdByOperatorId`: non applicabile in questo progetto (non esiste autenticazione/operatore nel modello Ticket).
- eventuali default: `id` generato dal server (formato `TCK-#####`), `createdAt` (timestamp server), `status` fisso a `"aperto"`.

## Dati Inviati Dal Client

- `title`:
- `description`:
- `priority`:
- `sourceChannel`:
- `customer`: (campo presente nel modello reale di questo progetto, non previsto dal template originale)

## Fuori Scope

- `responseDueAt`
- duplicate ticket detection
- notifiche
- assegnazioni
- filtri avanzati
- dashboard analytics
- autenticazione reale
- suite di test completa

## Regola In Forma Di Funzione

```txt
computeUrgencyLabel(priority, sourceChannel) -> urgencyLabel
```

Non duplicare questa regola nel client.
