# Luca Ginolfi

**Forward Deployed Engineer · Full-stack · livello Mid**

Porto il software dove si lavora davvero: sale, banconi, magazzini.

Lavoro nel digitale dal 2020, ma la passione per computer e tecnologia c'è da sempre. Studio informatica e, nei miei progetti, vado sul posto, capisco come lavora un ristorante o un negozio e torno con un gestionale, una pipeline che lo rilascia e, quando serve, un Raspberry Pi che stampa le comande.

Socio AIPIA, Associazione Italiana Professionisti Intelligenza Artificiale.

Contatti: luca.ginolfi@gmail.com

---

## In sintesi

| Area | Cosa faccio | Dove si vede |
|------|-------------|--------------|
| Lavoro sul campo | Analisi del processo sul posto, proposta tecnica e preventivo, formazione del personale, coordinamento dei fornitori | Tutti i progetti |
| CI/CD e rilasci | Pipeline GitHub Actions con gate di qualità e sicurezza, staging separato, approvazione manuale per la produzione, smoke test dopo il deploy | QUI, Studio arMonia |
| Sicurezza applicativa | RLS su Postgres, CSP e security header, gitleaks, Semgrep, CodeQL, dependency review, action pinnate per SHA | QUI, Studio arMonia, Piccoli Dettagli |
| Privacy e GDPR | Retention automatica dei dati personali, IP hashati, dati in region UE, analytics solo dopo consenso | QUI, Studio arMonia, Portale dentistico |
| Infrastruttura | Vercel, Cloudflare Workers e Pages, Supabase, VPS Linux con Docker Compose, PM2, UFW, Traefik | Tutti i progetti |
| Hardware locale | Print server su Raspberry Pi per stampanti termiche ESC/POS, etichette EAN-13, lettori barcode | Gestionale ristorante, Piccoli Dettagli |
| Costi | Free tier dove basta, self-hosting dove conviene, nessun abbonamento SaaS se esiste un'alternativa open source | Notifiche WhatsApp, print server |

---

## Livello

Mi considero Mid, con un profilo non uniforme. Lo scrivo per esteso perché chi legge sappia cosa aspettarsi.

| Area | Livello | Su cosa si basa |
|------|---------|-----------------|
| Analisi e gestione del cliente | Senior | Dalla richiesta generica alla consegna in autonomia: analisi, soluzione, stima, preventivo. Stime quasi sempre rispettate. Ho rifiutato lavori che non ritenevo corretti sul piano tecnico, economico o etico. |
| Responsabilità sulla produzione | Mid | Mantengo tra 4 e 8 sistemi attivi e gestisco in prima persona gli incidenti, compresi blocchi del servizio. |
| Pratiche di ingegneria | Mid | CI con gate di qualità e sicurezza, test unitari ed E2E, rilasci con staging. Decisioni documentate nei progetti più grandi. |
| Scala | Junior-Mid | Fino a 60 ordini al giorno su sistemi miei, fino a 200 su piattaforme di terzi che ho gestito. |
| Lavoro in team con code review | Junior | Finora solo progetti di gruppo all'università. Nei progetti individuali sono l'unico sviluppatore. |

**Dove sto crescendo**

| Obiettivo | Come |
|-----------|------|
| Lavoro in team | Entrare in un team con code review strutturata |
| Padronanza dello stack principale | Saper scrivere e spiegare ogni parte di SvelteKit, Next.js e Postgres anche senza assistenti AI |
| Osservabilità | Monitoraggio con alert e post-mortem scritti sui sistemi in produzione |
| Scala | Lavorare su sistemi con volumi e criticità maggiori |
| Inglese | Da colloquiale a fluente nel parlato. Scritto e lettura sono già a livello di lavoro. |

---

## Progetti principali

| Progetto | Cosa è | Stack | Stato | Link |
|----------|--------|-------|-------|------|
| [QUI](case-studies/05-qui-carburanti-cloudflare.md) | Web app anonima per segnalare la disponibilità di carburante durante una carenza | Cloudflare Workers, Durable Objects (SQLite), TypeScript | Beta, v0.1.8 | [sito](https://quicarburanti.it) · [codice](https://github.com/Lucacpt/qui_carburanti) |
| [Studio arMonia](case-studies/06-studio-armonia-nextjs-security.md) | Sito e piattaforma corsi per uno studio di neuropsicologia, migrato da WordPress | Next.js 15, Supabase, GitHub Actions | In produzione | [sito](https://www.studioarmonia.org) |
| [Piccoli Dettagli](case-studies/07-piccoli-dettagli-pos-magazzino.md) | Cassa e magazzino per un negozio di oggettistica | SvelteKit 2, Svelte 5, Supabase, Vercel | In produzione | [sito](https://www.piccolidettagli.shop) |
| [Gestionale ristorante](case-studies/01-gestionale-ristorante-pwa.md) | PWA per sala, cucina, prenotazioni, magazzino ed eventi | React, Supabase Realtime, Vercel, Raspberry Pi | In produzione | [sito](https://poke.cucinadasandro.it) |
| [Portale dentistico](case-studies/02-portale-dentistico-lead-generation.md) | Sito con raccolta contatti e blog gestito dallo studio | Next.js 15, Supabase, Tiptap | In produzione | [sito](https://www.studioodontoiatricofrancescobruno.it/) |
| [Saperi e Sapori](case-studies/08-saperi-e-sapori-mappa-offline.md) | Mappa offline per una sagra di tre giorni, con QR di posizionamento | HTML/JS vanilla, Leaflet, Service Worker, Cloudflare Pages | Evento concluso | [sito](https://saperiesapori.pages.dev) |
| [Notifiche WhatsApp self-hosted](case-studies/03-whatsapp-gratuito-vps-evolution-api.md) | Promemoria appuntamenti per un centro estetico senza costi per messaggio | Docker Compose, Evolution API, Redis, Node.js, PM2 | Prototipo | - |
| [SaaS multi-tenant per sagre](case-studies/04-saas-multi-tenant-sveltekit.md) | Preordini e cassa per più eventi sulla stessa piattaforma | SvelteKit, Supabase, Vercel Edge | Prototipo | - |

Sono progetti individuali realizzati per attività del territorio. Dove il codice non è pubblico, i case study descrivono architettura e scelte tecniche senza riportare codice proprietario. Ognuno chiude con le evoluzioni previste per la versione successiva.

---

## Come lavoro

**Pipeline**
- La CI è il gate per il merge: lint, typecheck, test unitari, build, E2E sull'artefatto buildato.
- Rilascio a due stadi dove il progetto lo giustifica: push su `main` va in staging, un tag `v*` va in produzione dopo approvazione.
- Dopo il deploy, uno smoke test controlla health endpoint, pagine principali e header di sicurezza.

**Sicurezza**
- Secret scanning sull'intera history (gitleaks), SAST (Semgrep, CodeQL), `npm audit` e allowlist delle licenze.
- Action GitHub pinnate per SHA, `permissions: {}` di default, Dependabot con cooldown.
- Uno script di CI blocca le migration SQL che creano tabelle senza RLS o policy di scrittura aperte ad `anon`.
- Security review periodiche sui progetti in produzione.

**Privacy**
- Dati personali con scadenza: job `pg_cron` o cron Vercel che cancellano lead e log oltre la retention dichiarata.
- Dove possibile il server non vede il dato: in QUI la posizione resta sul telefono e l'IP viene ridotto e hashato con una chiave che ruota ogni giorno.

**Intelligenza artificiale**
- Socio AIPIA (Associazione Italiana Professionisti Intelligenza Artificiale).
- Uso Claude Code, Cursor e Antigravity nello sviluppo quotidiano, e modelli locali con Ollama e LM Studio.
- Integro LLM nei prodotti dove portano valore concreto, ad esempio Gemini per riconoscere un prodotto dal codice EAN in Piccoli Dettagli.

---

## Altri lavori

**Siti web**

| Sito | Settore | Tecnologia |
|------|---------|------------|
| [ciroaquino.it](https://www.ciroaquino.it/) | Comunicazione politica, blog e rassegna stampa | Astro SSR, Supabase, Vercel (57 articoli migrati, CMS custom) |
| [cucinadasandro.it](https://cucinadasandro.it/) | Ristorazione | WordPress |
| [santosuossotrasporti.it](https://santosuossotrasporti.it/) | Trasporti e logistica | WordPress |
| [eclairolivierilab.it](https://www.eclairolivierilab.it/) | Pasticceria | WordPress |
| [grazianobuonanno.com](https://grazianobuonanno.com/) | Portfolio professionale | WordPress |

**Laboratorio e prototipi**

| Progetto | Contenuto |
|----------|-----------|
| Eventi Irpinia | Stack self-hosted su VPS: n8n, Postgres su rete interna, Traefik v3 con TLS Let's Encrypt. Base per un aggregatore di eventi con estrazione dati tramite LLM. Solo infrastruttura. |
| Odoo lab | Odoo 17 e Postgres 15 in Docker Compose per valutare l'ERP. |
| LLM locali | Ollama e LM Studio per provare modelli in locale. |

**Università**

| Progetto | Contenuto |
|----------|-----------|
| [WellYess](https://github.com/Lucacpt/WellYess) | Prototipo Flutter per anziani e caregiver: farmaci, parametri vitali, SOS. Corso di Interazione Uomo-Macchina. |
| [CyberDojo](https://github.com/Lucacpt/CyberDojo) | Piattaforma con gamification per avvicinare i ragazzi all'informatica. |
| Rischio idrogeologico, area vesuviana | Analisi in QGIS: elaborazione dati territoriali, mappatura delle aree a rischio, cartografia tematica. |

---

## Competenze tecniche

| Categoria | Strumenti |
|-----------|-----------|
| Linguaggi | TypeScript, JavaScript, SQL, Python, Dart |
| Frontend | SvelteKit, Svelte 5, React, Next.js (App Router), Astro, Tailwind CSS |
| Backend e dati | Node.js, Supabase (Postgres, Auth, Realtime, Storage, Edge Functions), Cloudflare Durable Objects |
| Hosting | Vercel, Cloudflare Workers e Pages, VPS Linux (Ubuntu) |
| Container e processi | Docker, Docker Compose, PM2, Traefik |
| CI/CD | GitHub Actions, Dependabot, rulesets e CODEOWNERS |
| Sicurezza | gitleaks, Semgrep, CodeQL, RLS, CSP e Trusted Types, UFW |
| Test | Vitest, Playwright, smoke test post-deploy |
| Automazione | n8n, cron (Vercel, pg_cron) |
| Hardware | Raspberry Pi, stampanti termiche ESC/POS, lettori barcode |
| CMS ed ERP | WordPress, Odoo |
| GIS | QGIS |
| SEO e advertising | SEOZoom, Google Ads, Google Tag Manager, Looker Studio |

---

## E-commerce e logistica

Esperienza operativa sulla vendita online e sulle spedizioni:

- **Marketplace:** Amazon Seller Central (compreso FBA), eBay, TikTok Shop
- **Spedizioni:** GLS Sell&Send, GLS WebLabelling, DHL, UPS
