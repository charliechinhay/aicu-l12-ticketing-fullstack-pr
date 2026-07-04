# Payload Cases - L11

## Caso Valido

```json
{
  "title": "",
  "description": "",
  "priority": "alta",
  "sourceChannel": "telefono"
}
```

Expected:

```txt
201
urgencyLabel=intervento rapido
```

## Caso Invalido 1

Input:

```json
{
  "priority": "immediata",
  "sourceChannel": "telefono"
}
```

Expected:

```txt
400
fieldErrors.priority
```

## Caso Invalido 2

Input:

```json
{
  "priority": "alta",
  "sourceChannel": "fax"
}
```

Expected:

```txt
400
fieldErrors.sourceChannel
```

## Non Autenticato

Expected:

```txt
401
```

## File Candidati Per L12

| File            | Perche'                                        |
| --------------- | ---------------------------------------------- |
| server/index.js | Perche contiene essattamente i due gap trovati |

## Confine Client/Server

Campi da non accettare dal client:

- `urgencyLabel`
- `createdByOperatorId`

Motivo:

```txt
Sono dati decisi dal server.
```
