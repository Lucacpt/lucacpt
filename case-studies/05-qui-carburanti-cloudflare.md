# QUI: disponibilità carburante su Cloudflare Workers

| | |
|---|---|
| Tipo | Web app pubblica e anonima basata su segnalazioni degli utenti |
| Stack | Cloudflare Workers, Durable Objects con SQLite, TypeScript, esbuild, Vitest, Biome |
| Periodo | Ottobre 2026, 35 commit, release da v0.1.0 a v0.1.8 |
| Stato | Beta, provincia di Avellino |
| Link | [quicarburanti.it](https://quicarburanti.it) · [codice](https://github.com/Lucacpt/qui_carburanti) (licenza EUPL 1.2) |

---

## Problema

Durante una carenza di carburante la domanda è una sola: quale distributore vicino ha ancora carburante adesso. I dati ufficiali (anagrafica MIMIT) dicono dove sono gli impianti, non se sono riforniti. L'unica fonte in tempo reale sono le persone che ci sono appena passate.

Un servizio del genere è un bersaglio facile per segnalazioni false e abusi, e tratta posizioni e abitudini delle persone.

## Soluzione

Una web app senza account. Chi è al distributore segnala se il carburante c'è o no. Un algoritmo combina le segnalazioni recenti e assegna a ogni impianto uno di quattro stati. Il server non riceve mai la posizione GPS e non conserva l'indirizzo IP.

---

## Architettura

```
Browser (TS senza framework, ~18 KB compressi al primo accesso)
   │  posizione calcolata sul dispositivo
   │  proof-of-work in un Web Worker prima di ogni segnalazione
   ▼
┌────────────────────────────────────────────────────────┐
│ Cloudflare Worker  (solo /api/*, il resto è asset)      │
│   /api/challenge   challenge PoW firmata HMAC           │
│   /api/report      segnalazione, rate limit per rete    │
│   /api/s/{prov}    snapshot in cache edge con ETag      │
│   /api/admin/*     dietro Cloudflare Access (JWT)       │
└──────────────┬───────────────────────────┬─────────────┘
               ▼                           ▼
     ProvinceDO (SQLite)             KeyDO (SQLite)
     un Durable Object               chiave HMAC giornaliera,
     per provincia                   cancellata a mezzanotte
     giurisdizione UE                giurisdizione UE
```

---

## Algoritmo di stato

Funzione pura in `src/domain/state.ts`, descritta in un ADR:

| Stato | Condizione |
|-------|------------|
| Disponibile | Almeno 3 conferme sul posto negli ultimi 45 minuti (una segnalazione da remoto vale un quarto) |
| Da verificare | Segnalazioni in conflitto o non ancora sufficienti |
| Segnalato esaurito | Più segnalazioni di assenza, almeno una sul posto |
| Nessuna informazione | Nessuna segnalazione nelle ultime 8 ore |

Per ogni segnalatore conta solo l'ultima segnalazione. Il peso per prefisso di rete ha un tetto e decade nel tempo.

---

## Privacy come vincolo tecnico

| Dato | Trattamento |
|------|-------------|
| Posizione GPS | Resta sul dispositivo, il calcolo della distanza è lato client |
| Indirizzo IP | Ridotto a prefisso e hashato con una chiave HMAC che ruota ogni giorno |
| Segnalazioni | Cancellate dopo 8 ore |
| Localizzazione dati | Durable Objects con giurisdizione UE |
| Cookie e tracciamento | Nessuno |

Questi vincoli sono verificati in CI, non solo dichiarati:
- uno script `privacy-guard` fallisce se il codice introduce cookie, fingerprinting, chiamate a terze parti o invio di coordinate al server
- un test controlla che `wrangler.jsonc` mantenga la giurisdizione UE
- un test confronta i tempi di conservazione scritti nelle pagine legali con quelli nel codice

---

## Pipeline

**CI su ogni PR e su `main`**

```
npm ci --ignore-scripts → lint (Biome) → typecheck (3 tsconfig) → test → build
   → privacy-guard → budget dimensione → controllo testi legali → smoke con wrangler dev
```

**Rilascio**

| Evento | Destinazione | Controlli |
|--------|--------------|-----------|
| Push su `main` | Staging (Worker e Durable Objects separati) | Smoke completo |
| Tag `v*` | Produzione | Approvazione manuale nell'environment GitHub, tag uguale alla versione in `package.json`, controllo legale bloccante, smoke in sola lettura, GitHub Release automatica |

**Supply chain e repository**
- Action pinnate per SHA, `harden-runner`, `permissions: {}` di default
- CodeQL con query `security-extended`, settimanale e su ogni PR
- Dependency review: blocca vulnerabilità alte e licenze non compatibili con EUPL
- Dependabot settimanale con 7 giorni di cooldown
- Ruleset su `main`: history lineare, solo squash, check `ci` obbligatorio, nessun bypass. CODEOWNERS su Worker, algoritmo e workflow.
- Nessuna dipendenza a runtime

---

## Difese contro l'abuso

| Minaccia | Contromisura |
|----------|--------------|
| Segnalazioni automatiche in massa | Proof-of-work con challenge firmata, rate limit di 30 segnalazioni al minuto per rete |
| Una persona che finge di essere molte | Tetto al peso per prefisso di rete, conta solo l'ultima segnalazione |
| Danno al gestore | Regole di tutela descritte in un ADR e in una pagina pubblica, contestazioni limitate |
| XSS | CSP `default-src 'none'` con Trusted Types, DOM costruito con API sicure |
| Information leak | Errori restituiti come 404 generico, log disattivati in produzione |

---

## Numeri

| Voce | Valore |
|------|--------|
| Test | circa 190 in 28 file |
| Peso al primo accesso | circa 18 KB compressi, verificato in CI |
| Dipendenze runtime | 0 |
| Release | 9 (v0.1.0 - v0.1.8) |

---

## Evoluzioni previste

Già in sviluppo su branch dedicati o pianificate dopo la beta:

| Evoluzione | Beneficio |
|------------|-----------|
| Strumenti di moderazione delle segnalazioni | Risposta più rapida a contestazioni e abusi |
| Nuova interfaccia | Lettura dello stato più immediata da smartphone |
| Estensione ad altre province | Un Durable Object per provincia: si scala aggiungendo istanze, senza cambiare architettura |
