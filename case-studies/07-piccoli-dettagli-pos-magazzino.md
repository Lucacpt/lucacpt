# Piccoli Dettagli: cassa e magazzino per un negozio

| | |
|---|---|
| Tipo | Gestionale interno per un negozio di bomboniere e oggettistica |
| Stack | SvelteKit 2, Svelte 5 (runes), TypeScript, Tailwind 4, Supabase, jsPDF, JsBarcode, html5-qrcode |
| Hosting | Vercel, Supabase |
| Periodo | Luglio - ottobre 2026, 27 commit |
| Stato | In produzione |
| Link | [piccolidettagli.shop](https://www.piccolidettagli.shop) (area gestionale riservata) |

---

## Problema

Il negozio gestiva carichi, prezzi e vendite su fogli Excel. Non sapeva quanto guadagnava davvero su ogni articolo, perché lo stesso prodotto arriva in lotti con costi diversi. Le etichette si scrivevano a mano.

## Soluzione

Un'applicazione unica per cassa, magazzino ed etichette, usata in negozio da tablet e PC con lettore barcode. Il costo viene tracciato per lotto, così il margine di ogni vendita è calcolato sul costo reale di quel pezzo.

---

## Moduli

**Cassa**
- Griglia touch e input da lettore barcode o fotocamera
- Sconti per riga e sul totale, calcolo del resto, carta e contanti
- Resi e articoli in prova

**Magazzino**
- Articoli con varianti (generatore taglia per colore)
- Carichi per lotto con barcode univoco del lotto
- Import da Excel tramite RPC, annullamento di un carico intero
- Sconti percentuali di massa che conservano il prezzo di listino

**Etichette**
PDF 60x40 mm per stampante termica, codice EAN-13.

**Buoni regalo**
Emissione, ricerca, utilizzo parziale, annullamento, storico movimenti, stampa con barcode.

**Analisi vendite**
Incasso, margine netto, ROI, articoli più venduti, filtri per periodo.

**Ricerca prodotto**
Dato un EAN interroga UPCItemDB e Open Food Facts. Se non trova nulla usa Gemini con ricerca web per proporre nome e descrizione.

---

## Modello dati

```
articoli ── varianti ── lotti_di_carico (costo, barcode lotto)
                              │
vendite ── righe_vendita ─────┘  prezzo, sconto e costo congelati alla vendita
buoni ── movimenti_buono
audit_logs
```

| Voce | Valore |
|------|--------|
| Migration | 17 |
| Tabelle | 13 |
| Policy RLS | 28 |
| RPC | `importa_carico` (con hash del file per evitare doppi import), `annulla_carico` |

Ogni riga di vendita salva prezzo, sconto e costo del lotto al momento della vendita. Le modifiche successive ai prezzi non alterano lo storico dei margini.

---

## Sicurezza

- Autenticazione SSR con cookie; tutte le route `/app` richiedono il ruolo admin lato server, altrimenti 403
- CSP, HSTS e security header impostati sia in `vercel.json` sia negli hook di SvelteKit
- `X-Robots-Tag: noindex` sull'area gestionale
- Migration di hardening che ha sostituito le policy `USING (true)` con controlli su `auth.uid()`

---

## Test e rilascio

| Voce | Dettaglio |
|------|-----------|
| CI | GitHub Actions su push e PR: `svelte-check`, Vitest, Playwright |
| Test unitari | circa 53 in 9 file |
| E2E | 16 in 7 suite (login, cassa, resi, buoni, health, header di sicurezza) |
| Deploy | `predeploy` esegue check, test unitari ed E2E prima di `vercel --prod` |

---

## Evoluzioni previste

Da un'analisi successiva al rilascio sono emerse queste implementazioni per la prossima versione:

| Evoluzione | Beneficio |
|------------|-----------|
| Checkout in una singola RPC transazionale | Vendita, scarico lotto e righe scritti in modo atomico |
| Deploy in produzione dalla CI | Rilasci tracciati e ripetibili, senza passaggi da locale |
| Cache e quota sulle ricerche prodotto via Gemini | Costi API prevedibili |
| PWA con cassa utilizzabile offline | Vendite registrate anche con connessione assente |
