# Request Trace - L11

## Feature Studenti

```txt
priority + sourceChannel -> urgencyLabel
```

## Trace

| Step         | Cosa entra                                                                                               | Cosa decide il server                                                                                            | Errore possibile                                                                                                            | Evidenza attesa                                        |
| ------------ | -------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------ |
| request      | POST `/api/tickets` con `{title, customer, description, priority, sourceChannel}` da `src/main.js:77-84` | —                                                                                                                | body non JSON valido                                                                                                        | 500 con `detail` (nessun try/catch dedicato al parse)  |
| validation   | `input` normalizzato da `normalizeTicketInput` (server/index.js:204)                                     | `validateTicketInput` (server/index.js:90) controlla title, customer, priority                                   | `priority` non in `validPriorities` -> `fieldErrors.priority`; `sourceChannel` NON validato (TODO riga 105-106, bug aperto) | 400 + `fieldErrors`                                    |
| auth/session | —                                                                                                        | nessuna autenticazione presente                                                                                  | —                                                                                                                           | fuori scope (confermato in backend-rules.md)           |
| rule         | `input.priority`, `input.sourceChannel`                                                                  | `computeUrgencyLabel(priority, sourceChannel)` (server/index.js:84) — attualmente STUB che ritorna sempre `null` | nessuna gestione errori nella regola                                                                                        | `urgencyLabel` sempre `null` finche' non implementata  |
| Prisma       | — (non c'e' Prisma)                                                                                      | `db.prepare(...).run(...)` inserisce riga in tabella `tickets` (sqlite, server/index.js:137-161)                 | vincoli NOT NULL sulla tabella potrebbero far lanciare eccezione -> 500                                                     | riga persistita con `urgency_label` colonna            |
| response     | —                                                                                                        | `sendJson(response, 201, { ticket: createTicket(input) })` (server/index.js:261)                                 | —                                                                                                                           | corpo JSON con ticket completo, incluso `urgencyLabel` |

## Caso Valido

```txt
priority=alta
sourceChannel=telefono
expected urgencyLabel=intervento rapido
```

Nota: attualmente il server restituirebbe `urgencyLabel=null` a causa dello stub non implementato in `computeUrgencyLabel` — gap gia' segnalato in `backend-rules.md` e `payload-cases.md`.

## Caso Invalido

```txt
field=sourceChannel
value=fax
expected error=fieldErrors.sourceChannel
```

Nota: attualmente NON produce errore, perche' la validazione di `sourceChannel` e' un TODO non implementato in `validateTicketInput` (server/index.js:105-106) — bug aperto.

## File Candidati

| Area          | File candidato                                   | Perche'                                                     |
| ------------- | ------------------------------------------------ | ----------------------------------------------------------- |
| UI payload    | `src/main.js:11-21` (`getFormPayload`)           | costruisce il payload inviato dal form                      |
| API route     | `server/index.js:248-263`                        | endpoint POST `/api/tickets`                                |
| validation    | `server/index.js:90-109` (`validateTicketInput`) | valida title/customer/priority; manca sourceChannel         |
| rule          | `server/index.js:84-88` (`computeUrgencyLabel`)  | stub da implementare secondo mapping in `backend-rules.md`  |
| Prisma/schema | `server/index.js:20-32` (CREATE TABLE sqlite)    | schema reale della tabella `tickets` (non Prisma)           |
| lista/card    | `src/main.js:42-64` (`renderTickets`)            | mostra `urgencyLabel` nella UI, con fallback "da calcolare" |

## Fuori Scope

- Autenticazione reale (nessun meccanismo presente)
- Prisma/ORM (il progetto usa `node:sqlite` con SQL raw)
- notifiche, assegnazioni, filtri avanzati, dashboard analytics (come da `backend-rules.md`)
