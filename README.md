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
| E-commerce e logistica | Marketplace, Amazon FBA, corrieri, apertura di nuovi canali di vendita | [Sezione dedicata](#e-commerce-e-logistica) |
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

## E-commerce e logistica

Il commercio online è il campo in cui ho iniziato a lavorare, e da lì vengono molti dei problemi che risolvo nei progetti: codici EAN, etichette termiche, lotti di magazzino, spedizioni.

**Marketplace** <br>
![Amazon Seller Central](https://img.shields.io/badge/Amazon_Seller_Central-FF9900?style=for-the-badge) ![Amazon FBA](https://img.shields.io/badge/Amazon_FBA-232F3E?style=for-the-badge) ![eBay](https://img.shields.io/badge/eBay-E53238?style=for-the-badge&logo=ebay&logoColor=white) ![TikTok Shop](https://img.shields.io/badge/TikTok_Shop-000000?style=for-the-badge&logo=tiktok&logoColor=white)

**Corrieri e piattaforme di spedizione** <br>
![GLS Sell&Send](https://img.shields.io/badge/GLS-Sell%26Send-061AB1?style=for-the-badge&labelColor=FFD100) ![GLS WebLabelling](https://img.shields.io/badge/GLS-WebLabelling-061AB1?style=for-the-badge&labelColor=FFD100) ![DHL](https://img.shields.io/badge/DHL-FFCC00?style=for-the-badge&logo=dhl&logoColor=D40511) ![UPS](https://img.shields.io/badge/UPS-351C15?style=for-the-badge&logo=ups&logoColor=FFB500)

| Ambito | Attività |
|--------|----------|
| Marketplace | Account venditore, catalogo, listini, ordini e resi su Amazon Seller Central, eBay e TikTok Shop |
| Amazon FBA | Preparazione delle spedizioni verso i centri logistici Amazon e controllo dello stock |
| Nuovi canali | Apertura di un canale di vendita su marketplace partendo da zero |
| Spedizioni | Etichette, ritiri e tracciamento con GLS Sell&Send, GLS WebLabelling, DHL e UPS |
| Automazioni | Script e import/export per catalogo, listini e ordini |
| Sistemi | Postazioni, rete, stampanti e account a supporto delle vendite |
| Marketing | SEO, Google Ads e analisi del traffico |

Dove ritorna nei progetti: [Piccoli Dettagli](case-studies/07-piccoli-dettagli-pos-magazzino.md) (lotti, EAN-13, etichette 60x40 mm), [gestionale ristorante](case-studies/01-gestionale-ristorante-pwa.md) (stampa ESC/POS, magazzino), [santosuossotrasporti.it](https://santosuossotrasporti.it/) (sito per un'azienda di trasporti).

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

## Stack e strumenti

**Linguaggi** <br>
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white) ![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black) ![SQL](https://img.shields.io/badge/SQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white) ![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white) ![Dart](https://img.shields.io/badge/Dart-0175C2?style=for-the-badge&logo=dart&logoColor=white)

**Frontend** <br>
![SvelteKit](https://img.shields.io/badge/SvelteKit-FF3E00?style=for-the-badge&logo=svelte&logoColor=white) ![React](https://img.shields.io/badge/React-20232A?style=for-the-badge&logo=react&logoColor=61DAFB) ![Next.js](https://img.shields.io/badge/Next.js-000000?style=for-the-badge&logo=nextdotjs&logoColor=white) ![Astro](https://img.shields.io/badge/Astro-BC52EE?style=for-the-badge&logo=astro&logoColor=white) ![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-06B6D4?style=for-the-badge&logo=tailwindcss&logoColor=white) ![Vite](https://img.shields.io/badge/Vite-646CFF?style=for-the-badge&logo=vite&logoColor=white) ![Leaflet](https://img.shields.io/badge/Leaflet-199900?style=for-the-badge&logo=leaflet&logoColor=white) ![Flutter](https://img.shields.io/badge/Flutter-02569B?style=for-the-badge&logo=flutter&logoColor=white)

**Backend e dati** <br>
![Node.js](https://img.shields.io/badge/Node.js-339933?style=for-the-badge&logo=nodedotjs&logoColor=white) ![Supabase](https://img.shields.io/badge/Supabase-3FCF8E?style=for-the-badge&logo=supabase&logoColor=white) ![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white) ![Redis](https://img.shields.io/badge/Redis-DC382D?style=for-the-badge&logo=redis&logoColor=white) ![Durable Objects](https://img.shields.io/badge/Durable_Objects-F38020?style=for-the-badge&logo=cloudflare&logoColor=white)

**Hosting e infrastruttura** <br>
![Vercel](https://img.shields.io/badge/Vercel-000000?style=for-the-badge&logo=vercel&logoColor=white) ![Cloudflare Workers](https://img.shields.io/badge/Cloudflare_Workers-F38020?style=for-the-badge&logo=cloudflareworkers&logoColor=white) ![Cloudflare Pages](https://img.shields.io/badge/Cloudflare_Pages-F38020?style=for-the-badge&logo=cloudflarepages&logoColor=white) ![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white) ![Proxmox](https://img.shields.io/badge/Proxmox-E57000?style=for-the-badge&logo=proxmox&logoColor=white) ![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white) ![PM2](https://img.shields.io/badge/PM2-2B037A?style=for-the-badge&logo=pm2&logoColor=white) ![Traefik](https://img.shields.io/badge/Traefik-24A1C1?style=for-the-badge&logo=traefikproxy&logoColor=white) ![Let's Encrypt](https://img.shields.io/badge/Let's_Encrypt-003A70?style=for-the-badge&logo=letsencrypt&logoColor=white)

**CI/CD, sicurezza e test** <br>
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=for-the-badge&logo=githubactions&logoColor=white) ![Dependabot](https://img.shields.io/badge/Dependabot-025E8C?style=for-the-badge&logo=dependabot&logoColor=white) ![CodeQL](https://img.shields.io/badge/CodeQL-181717?style=for-the-badge&logo=github&logoColor=white) ![Semgrep](https://img.shields.io/badge/Semgrep-25A6A6?style=for-the-badge) ![gitleaks](https://img.shields.io/badge/gitleaks-F05032?style=for-the-badge&logo=git&logoColor=white) ![Vitest](https://img.shields.io/badge/Vitest-6E9F18?style=for-the-badge&logo=vitest&logoColor=white) ![Playwright](https://img.shields.io/badge/Playwright-2EAD33?style=for-the-badge) ![Biome](https://img.shields.io/badge/Biome-60A5FA?style=for-the-badge&logo=biome&logoColor=white)

**Automazione e AI** <br>
![n8n](https://img.shields.io/badge/n8n-EA4B71?style=for-the-badge&logo=n8n&logoColor=white) ![Claude](https://img.shields.io/badge/Claude-D97757?style=for-the-badge&logo=anthropic&logoColor=white) ![Gemini](https://img.shields.io/badge/Gemini-8E75B2?style=for-the-badge&logo=googlegemini&logoColor=white) ![Ollama](https://img.shields.io/badge/Ollama-000000?style=for-the-badge&logo=ollama&logoColor=white) ![Cursor](https://img.shields.io/badge/Cursor-000000?style=for-the-badge&logo=cursor&logoColor=white) ![Antigravity](https://img.shields.io/badge/Antigravity-4285F4?style=for-the-badge&logo=google&logoColor=white) ![VS Code](https://img.shields.io/badge/VS_Code-0078D4?style=for-the-badge) ![GitHub Copilot](https://img.shields.io/badge/GitHub_Copilot-121011?style=for-the-badge&logo=githubcopilot&logoColor=white)

**Hardware** <br>
![Raspberry Pi](https://img.shields.io/badge/Raspberry_Pi-A22846?style=for-the-badge&logo=raspberrypi&logoColor=white) ![Stampanti termiche](https://img.shields.io/badge/Stampanti_termiche-ESC%2FPOS-555555?style=for-the-badge) ![Barcode](https://img.shields.io/badge/Barcode-EAN--13-555555?style=for-the-badge)

**CMS, ERP e GIS** <br>
![WordPress](https://img.shields.io/badge/WordPress-21759B?style=for-the-badge&logo=wordpress&logoColor=white) ![Odoo](https://img.shields.io/badge/Odoo-714B67?style=for-the-badge&logo=odoo&logoColor=white) ![QGIS](https://img.shields.io/badge/QGIS-589632?style=for-the-badge&logo=qgis&logoColor=white)

**SEO e marketing** <br>
![SEOZoom](https://img.shields.io/badge/SEOZoom-FF6600?style=for-the-badge) ![Google Ads](https://img.shields.io/badge/Google_Ads-4285F4?style=for-the-badge&logo=googleads&logoColor=white) ![Google Tag Manager](https://img.shields.io/badge/Google_Tag_Manager-246FDB?style=for-the-badge&logo=googletagmanager&logoColor=white) ![Google Analytics](https://img.shields.io/badge/Google_Analytics-E37400?style=for-the-badge&logo=googleanalytics&logoColor=white) ![Looker Studio](https://img.shields.io/badge/Looker_Studio-4285F4?style=for-the-badge&logo=looker&logoColor=white)
