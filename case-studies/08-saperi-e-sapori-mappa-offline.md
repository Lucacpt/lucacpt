# Saperi e Sapori: mappa offline per una sagra

| | |
|---|---|
| Tipo | Mappa interattiva degli stand per una sagra di tre giorni a Gesualdo (AV) |
| Stack | HTML, CSS e JavaScript senza framework, Leaflet 1.9, Service Worker |
| Hosting | Cloudflare Pages, deploy automatico da `main` |
| Periodo | Luglio 2026 |
| Stato | Evento concluso, progetto archiviato |
| Link | [saperiesapori.pages.dev](https://saperiesapori.pages.dev) |

---

## Problema

La sagra si svolge nei vicoli di un centro storico, con 47 stand distribuiti su più strade. Durante l'evento la rete mobile è satura: un'app che dipende dalla connessione smette di funzionare proprio quando serve.

## Soluzione

Una PWA statica che funziona anche senza rete dopo il primo caricamento. Venti QR code affissi nei varchi dicono all'app dove si trova il visitatore, senza bisogno del GPS.

---

## Funzioni

- Ricerca per nome, prodotto, categoria e parole chiave in più lingue
- QR nei varchi: il link `?spot=ID` posiziona il marker "Sei qui"
- Scheda dello stand a comparsa dal basso
- "Naviga": usa il GPS e apre Google Maps; se il GPS non risponde disegna un percorso tratteggiato offline
- Fogli A4 stampabili con i QR dei 20 varchi

---

## Cache

| Risorsa | Cache | Strategia |
|---------|-------|-----------|
| App shell e dati | `saperi-sapori-v6` | Network-first con fallback alla cache |
| Tile della mappa | `saperi-tiles-v1` | Network-first con fallback alla cache |
| Librerie da CDN | App shell | Cache-first, con Subresource Integrity |

All'attivazione il Service Worker elimina le cache delle versioni precedenti.

---

## Scelte principali

| Decisione | Scelta | Motivo |
|-----------|--------|--------|
| Backend | Nessuno, dati in un file JS | Tre giorni di evento, 69 punti fissi: un database non serviva |
| Framework | Nessuno | Meno peso da scaricare su rete lenta |
| Posizione | QR prima del GPS | Nei vicoli il GPS è impreciso, il QR è esatto |
| Hosting | Cloudflare Pages | Statico, gratuito, deploy a ogni push |

---

## Evoluzioni previste

Per una prossima edizione dell'evento:

| Evoluzione | Beneficio |
|------------|-----------|
| Build con minificazione in CI | Nessun file generato nel repository |
| Versione della cache derivata dall'hash della build | Aggiornamenti dei visitatori senza interventi manuali |
| Dati degli stand in un file JSON editabile dagli organizzatori | Riutilizzo per altre sagre senza modificare il codice |
