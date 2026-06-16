---
name: checkupformazione81
description: >
  Genera il CHECKUP FORMAZIONE 81 per un cliente di Overall Group Srl: un'analisi
  sintetica di 2-3 pagine dello stato di adempimento agli obblighi formativi sulla
  sicurezza sul lavoro (D.Lgs. 81/08 + Accordo Stato-Regioni 17/04/2025), prodotto
  come file .docx brandizzato Overall Group. Usa questa skill ogni volta che
  l'utente menziona "checkup formazione", "piano formativo sicurezza", "checkup 81",
  "verifica obblighi formativi", "CHECKUP FORMAZIONE 81", "executive summary
  formazione", oppure vuole produrre un documento sintetico di analisi della
  formazione sicurezza per un cliente nuovo o già in scadenzario. Include la
  gestione dei clienti NUOVI (partendo da zero con ATECO, sede, addetti oppure da
  visura camerale; con eventuale analisi attestati già in possesso) e VECCHI
  (matching con scadenzario allegato).
---

# CHECKUP FORMAZIONE 81

Skill di Overall Group Srl per la produzione del **CHECKUP FORMAZIONE 81** in formato **Executive Summary**: un documento sintetico di 2-3 pagine che fotografa lo stato di adempimento dell'azienda agli obblighi formativi in materia di salute e sicurezza sul lavoro ai sensi del **D.Lgs. 81/08** e dell'**Accordo Stato-Regioni 17/04/2025**.

**Il documento di output è pensato per essere consegnato al cliente**: sintetico, visivo, orientato all'azione commerciale. Non è un documento tecnico esteso.

---

## RISORSE DISPONIBILI IN `references/` e `assets/`

### Logo aziendale (cartella `assets/`)

- **`assets/logo_overall.jpg`** — logo Overall Group da inserire in testa al documento (frontespizio del summary) e, in versione ridotta, nell'header di ogni pagina.

### File di sintesi operativa (4 file `.md` in `references/`)

Questa skill dispone di una biblioteca normativa locale. **Consulta sempre i file in `references/` prima di ricorrere a ricerche web**, perché contengono i testi autentici delle norme di riferimento e le tabelle operative già predisposte.

- **`references/ateco-rischio.md`** — mappatura codice ATECO → classe di rischio formativo (BASSO/MEDIO/ALTO); include casi borderline frequenti (pulizie, gelaterie, officine, asili).
- **`references/percorsi-formativi.md`** — durate, aggiornamenti, propedeuticità di tutti i percorsi formativi (lavoratori, preposti, dirigenti, DL-RSPP, RSPP/ASPP, antincendio, primo soccorso, HACCP, attrezzature, RLS, CSE/CSP, formatori). Include il regime transitorio dell'ASR 17/04/2025.
- **`references/template-output.md`** — struttura standard dell'Executive Summary DOCX di output, con specifiche grafiche Overall Group e regole redazionali.
- **`references/accordi-storici.md`** — indice normativo di primo livello. **Da consultare per prima cosa** quando si analizza un attestato: contiene la stratificazione temporale (quale norma si applica a un attestato emesso in data X) e i riferimenti puntuali ai file di dettaglio.

### Testi normativi autentici (7 file `.txt` in `references/`)

Da consultare quando serve verificare requisiti formali di un attestato o dettagli di un percorso formativo:

- **`ASR_221_2011_punto7_ATTESTATI.txt`** — elementi minimi obbligatori dell'attestato sotto il regime precedente (attestati fino al 23/05/2026)
- **`ASR_59_2025_punto6_ATTESTAZIONI.txt`** — elementi minimi sotto il nuovo Accordo (attestati dal 24/05/2025)
- **`DI_06_03_2013_REQUISITI_FORMATORI.txt`** — qualificazione docenti SSL (6 criteri + aggiornamento triennale)
- **`DM_02_09_2021_ANTINCENDIO.txt`** — regime antincendio vigente (Livelli 1/2/3, docenti, aggiornamento 5 anni)
- **`DM_10_03_1998_ANTINCENDIO_ABROGATO.txt`** — regime antincendio storico (per attestati pre-04/10/2022)
- **`DM_388_2003_PRIMO_SOCCORSO.txt`** — primo soccorso (gruppi A/B/C, docente medico, aggiornamento triennale)
- **`REG_CE_852_2004_HACCP.txt`** — HACCP/alimentaristi

### Testi integrali degli accordi (5 file `RAW.txt` in `references/accordi/`)

Da consultare solo quando la sintesi nei file sopra non basta e serve il testo integrale:

- `accordi/ASR_221_2011_lavoratori_RAW.txt`
- `accordi/ASR_223_2011_DL_RSPP_RAW.txt`
- `accordi/ASR_53_2012_attrezzature_RAW.txt`
- `accordi/ASR_128_2016_RSPP_ASPP_RAW.txt`
- `accordi/ASR_153_2012_linee_applicative_RAW.txt`

### Regola operativa sulla ricerca web

Per qualsiasi dubbio normativo, segui questa gerarchia:

1. **Consulta prima `references/accordi-storici.md`** per individuare quale norma si applica.
2. **Poi consulta il file `.txt` specifico della norma** in `references/`.
3. **Se serve il testo integrale**, consulta il file `RAW.txt` corrispondente in `references/accordi/`.
4. **Solo se il dato non è presente nei file locali** (es. norma uscita dopo la stesura della skill, interpello recente), effettua una ricerca web mirata e segnala sempre con trasparenza quando un dato è stato verificato online.

---

## FASE 1 — RACCOLTA DATI

All'attivazione della skill, chiedi subito:

> **Il cliente è NUOVO o VECCHIO?**
> - **NUOVO** = non ha mai fatto formazione con noi, partiamo da zero
> - **VECCHIO** = già cliente, ha uno scadenzario formativo esistente

---

### Se NUOVO

#### Step 1.1 — Dati aziendali (obbligatorio)

Chiedi i dati aziendali proponendo **sempre due modalità alternative**:

> "Per procedere ho bisogno dei dati dell'azienda. Puoi fornirmeli in **due modi alternativi**:
>
> **Opzione A — Allegare la visura camerale** (più rapido)
> Carica il PDF della visura: estrarrò automaticamente ragione sociale, sede, ATECO, numero addetti e soci.
>
> **Opzione B — Compilare i dati manualmente**
> ```
> - Ragione sociale:
> - Forma giuridica (Srl, Spa, ditta individuale... — opzionale):
> - Sede (indirizzo completo):
> - Codice ATECO (es. 46.49.9):
> - Numero addetti (totale, specificando dipendenti + titolare/soci):
> ```"

Se l'utente carica la visura, estrai i dati e mostrali in chat per conferma prima di procedere. Se estrai dalla visura, controlla in particolare: ragione sociale, forma giuridica, sede legale, codice ATECO (sia ATECO 2025 sia ATECORI 2007-2022 se presenti), numero addetti (dipendenti + indipendenti), nominativi soci/titolari di cariche.

#### Step 1.2 — Attestati di formazione (opzionale ma determinante)

Subito dopo aver confermato i dati aziendali, chiedi:

> "Hai attestati di formazione sulla sicurezza già in possesso dell'azienda?
> - **SÌ** → Allega gli attestati (PDF, immagini, scansioni). Analizzerò la loro congruenza rispetto alla normativa vigente al momento dell'emissione e a quella attuale (ASR 17/04/2025).
> - **NO** → Procedo ipotizzando un organigramma della sicurezza sulla base di ATECO, dimensione aziendale e settore, poi costruisco il piano formativo da zero."

**In base alla risposta:**
- Se **NO attestati** → vai alla **FASE 2A — Ramo A**
- Se **SÌ attestati** → vai alla **FASE 2A — Ramo B**

---

### Se VECCHIO

Chiedi:

```
DATI AZIENDA
- Ragione sociale:
- Sede:
```

Poi di':
> "Per procedere ho bisogno di due file (il secondo è opzionale ma migliora l'analisi):
> 1. **Scadenzario formativo** (obbligatorio) — file Excel o PDF estratto dal gestionale
> 2. **Elenco personale aggiornato** (opzionale) — lista dei dipendenti con ruoli/reparti produttivi, utile per derivare il livello di rischio per reparto e identificare nuove assunzioni o dimissioni rispetto allo scadenzario"

Una volta ricevuti i file:

**Se solo scadenzario**: leggilo per estrarre:
- Elenco dei lavoratori e relative figure di sicurezza
- Corsi già effettuati con date di completamento
- Scadenze imminenti (entro 6-12 mesi)
- Corsi mancanti o mai effettuati

**Se scadenzario + elenco personale**: esegui anche il **match tra le due fonti**:
- Identifica i dipendenti presenti nell'elenco personale ma assenti dallo scadenzario → **nuove assunzioni** (formazione da zero)
- Identifica i dipendenti presenti nello scadenzario ma assenti dall'elenco personale → **possibili dimissioni/trasferimenti** (segnalare per verifica)
- Usa i ruoli/reparti dell'elenco personale per assegnare il corretto livello di rischio (BASSO/MEDIO/ALTO) a ciascun lavoratore, consultando `references/ateco-rischio.md`

**Poi vai alla FASE 2B.**

---

## FASE 2A — ELABORAZIONE CLIENTE NUOVO

### Passo comune: Determinare il livello di rischio dal codice ATECO

Leggi il file `references/ateco-rischio.md` per mappare il codice ATECO al livello di rischio (BASSO / MEDIO / ALTO).

Il livello di rischio determina le ore di **formazione specifica** per i lavoratori:
- **BASSO**: 4 ore specifica → totale 8 ore (4 gen + 4 spec)
- **MEDIO**: 8 ore specifica → totale 12 ore (4 gen + 8 spec)
- **ALTO**: 12 ore specifica → totale 16 ore (4 gen + 12 spec)

---

### Ramo A — Cliente NUOVO SENZA attestati

#### Passo A1: Proporre 2-3 scenari di organigramma ipotetico

Sulla base di ATECO, livello di rischio e numero addetti, proponi all'utente **2-3 scenari alternativi di organigramma della sicurezza**, descritti in modo chiaro con pro/contro di ciascuno. Esempi di scenari tipici:

- **Scenario 1 — DL=RSPP (art. 34)**: il datore di lavoro svolge direttamente la funzione di RSPP. Possibile solo nei casi ammessi dall'art. 34 D.Lgs. 81/08 e Allegato II (aziende con numero di lavoratori e settore compatibili).
- **Scenario 2 — RSPP interno dipendente**: figura aziendale dedicata, con percorso modulare A+B+C.
- **Scenario 3 — RSPP esterno in consulenza**: professionista incaricato dall'esterno (tipico per microimprese non rientranti nell'art. 34 o che preferiscono esternalizzare).

Per ciascuno scenario indica le figure di contorno obbligatorie (Preposto, RLS o RLST, Addetti PS, Addetti AI) e quelle eventuali in base alle attrezzature/attività (carrellisti, PLE, carroponte).

Chiedi all'utente di scegliere lo scenario o di confermarne uno con modifiche.

#### Passo A2: Costruire il piano formativo sintetico

Una volta scelto lo scenario, per ogni figura presente usa la tabella in `references/percorsi-formativi.md` e determina il piano formativo di alto livello (quanti corsi, per chi, con che priorità).

**Non serve il dettaglio corso-per-corso**: il documento di output è un Executive Summary, quindi la sintesi può limitarsi a 5-8 righe di piano.

Regole generali:
- Per i **lavoratori**: la formazione deve avvenire **prima dell'avvio dell'attività** (eliminato il termine dei 60 giorni - novità ASR 17/04/2025)
- Per il **DL-RSPP art.34**: il percorso è DL base (16h) + Modulo Comune (8h) = 24h. Il Modulo Tecnico-Integrativo si applica solo ad agricoltura (16h), pesca (12h), costruzioni (16h), chimico-petrolchimico (16h). Per tutti gli altri settori NON è richiesto.
- Per il **Preposto**: accede al modulo integrativo (12h) solo dopo aver completato la formazione da lavoratore (gen + spec). Aggiornamento biennale di 6h.
- Per i **carrellisti / PLE / carroponte**: sono corsi abilitativi ex art.73 d.lgs.81/08. Leggi i dettagli in `references/percorsi-formativi.md`.
- **Scadenza transitoria**: entro il **24/05/2027** (24 mesi dall'entrata in vigore dell'Accordo) le aziende devono adeguarsi ai nuovi percorsi.

#### Passo A3: Generare l'Executive Summary DOCX

Vai alla **FASE 3** e genera il documento seguendo `references/template-output.md`.

Nel Ramo A il documento indicherà chiaramente che l'organigramma è **ipotetico** e soggetto a conferma da parte del cliente. Non ci sono schede attestati.

---

### Ramo B — Cliente NUOVO CON attestati

#### Passo B1: Acquisire gli attestati

Richiedi gli attestati come allegati. Per ciascun attestato estrai:
- Nominativo del lavoratore
- Figura/ruolo oggetto della formazione (es. Lavoratore, Preposto, RSPP, Addetto AI, ecc.)
- Data di emissione / data di completamento del corso
- Ente formatore e eventuale accreditamento
- Ore di formazione
- Normativa di riferimento citata nell'attestato
- Eventuale scadenza indicata

#### Passo B2: Analisi di congruenza normativa

Per ogni attestato, esegui una **doppia analisi** basata sui testi normativi autentici disponibili in `references/`.

**a) Congruenza alla normativa vigente al momento dell'emissione**

Per prima cosa, consulta **`references/accordi-storici.md`**: contiene la stratificazione temporale che ti dice, data per data, quale norma si applica a un attestato emesso in quel momento.

Tappe normative principali:
- **ASR 21/12/2011 rep. 221/CSR** — formazione lavoratori, preposti, dirigenti → dettaglio in `references/ASR_221_2011_punto7_ATTESTATI.txt`; testo integrale in `references/accordi/ASR_221_2011_lavoratori_RAW.txt`
- **ASR 21/12/2011 rep. 223/CSR** — formazione DL-RSPP ex art. 34 → testo integrale in `references/accordi/ASR_223_2011_DL_RSPP_RAW.txt`
- **ASR 22/02/2012 rep. 53/CSR** — formazione attrezzature ex art. 73 → testo integrale in `references/accordi/ASR_53_2012_attrezzature_RAW.txt`
- **ASR 25/07/2012 rep. 153/CSR** — linee applicative → testo in `references/accordi/ASR_153_2012_linee_applicative_RAW.txt`
- **ASR 07/07/2016 rep. 128/CSR** — formazione RSPP/ASPP → testo in `references/accordi/ASR_128_2016_RSPP_ASPP_RAW.txt`
- **D.M. 388/2003** — primo soccorso → dettaglio in `references/DM_388_2003_PRIMO_SOCCORSO.txt`
- **D.M. 10/03/1998** — antincendio (fino al 03/10/2022) → dettaglio in `references/DM_10_03_1998_ANTINCENDIO_ABROGATO.txt`
- **D.M. 02/09/2021** — antincendio (dal 04/10/2022) → dettaglio in `references/DM_02_09_2021_ANTINCENDIO.txt`
- **D.I. 06/03/2013** — requisiti formatori SSL → dettaglio in `references/DI_06_03_2013_REQUISITI_FORMATORI.txt`
- **Reg. CE 852/2004** — HACCP → dettaglio in `references/REG_CE_852_2004_HACCP.txt`
- **ASR 17/04/2025 rep. 59/CSR** — nuovo accordo unico → dettaglio in `references/ASR_59_2025_punto6_ATTESTAZIONI.txt`

Valuta:
- Sul piano **formale**: l'attestato riporta tutti gli elementi minimi obbligatori richiesti dalla norma vigente alla data di emissione? (Es. soggetto organizzatore, normativa di riferimento, dati anagrafici, codice fiscale se richiesto, durata, firma, ecc. — consulta il file `.txt` della norma specifica.)
- Sul piano **sostanziale**: l'attestato era conforme alla norma applicabile? Durata corretta? Moduli corretti? Ente formatore accreditato? Docente in possesso dei requisiti (consulta `DI_06_03_2013_REQUISITI_FORMATORI.txt` per i corsi ex artt. 34 e 37)?

**b) Congruenza alla normativa attuale (ASR 17/04/2025 e D.Lgs. 81/08 in vigore)**

Consulta `references/ASR_59_2025_punto6_ATTESTAZIONI.txt` per i nuovi elementi minimi (da applicare ai corsi dal 24/05/2025 in poi) e `references/percorsi-formativi.md` per le nuove durate/cadenze.

Valuta:
- L'attestato è ancora valido o scaduto? (Calcola dalla data + periodicità di aggiornamento — vedi `references/percorsi-formativi.md` sezione "Cronoprogramma sintetico aggiornamenti")
- Serve un adeguamento al nuovo ASR entro il **24/05/2027**?
- Sono necessari moduli aggiuntivi (es. modulo integrativo preposto, modulo comune DL-RSPP)?

Assegna a ogni attestato un semaforo:
- **VERDE** 🟢 — conforme e non in scadenza
- **GIALLO** 🟡 — in scadenza entro 12 mesi, criticità formale minore, o necessita di adeguamento ASR entro il 24/05/2027
- **ROSSO** 🔴 — scaduto, non conforme sostanzialmente, o figura mancante (formazione da integrare)

**Fallback ricerca web:** se emerge un dubbio su una norma non coperta dai file in `references/` (es. interpello MLPS recente, circolare specifica di settore), effettua una ricerca web mirata e segnala con trasparenza che il dato proviene da fonte esterna.

#### Passo B3: Identificare i gap di formazione

Confronta gli attestati analizzati con i requisiti per la figura/mansione e identifica:
- Corsi scaduti da rinnovare
- Corsi mai fatti per figure già nominate
- Figure obbligatorie non coperte (PS, AI, Preposto, RLS, ecc.)
- Necessità di aggiornamento normativo (ASR 2025)

#### Passo B4: Sintetizzare per l'Executive Summary

**Il documento finale non contiene schede dettagliate dei singoli attestati.** L'analisi approfondita che hai condotto (attestato per attestato, formale + sostanziale) viene **distillata** nelle seguenti informazioni di alto livello:

- **Contatore**: quanti attestati sono conformi (🟢), quanti con criticità minori (🟡), quanti con criticità sostanziali (🔴)
- **Tabella per lavoratore**: per ciascun addetto, una singola riga con un semaforo complessivo e una frase che spiega cosa manca o cosa va fatto
- **Top-5 azioni urgenti** da intraprendere nei prossimi 30-60 giorni
- **Riepilogo numerico** degli interventi per priorità (ALTA / MEDIA / BASSA)

Vai alla **FASE 3**.

---

## FASE 2B — ELABORAZIONE CLIENTE VECCHIO

### Passo 1: Analisi dello scadenzario

Dal file allegato, estrai per ogni persona:
- Figura di sicurezza ricoperta
- Corsi effettuati (tipologia, data, ore, soggetto formatore)
- Scadenza dell'aggiornamento (calcola dalla data del corso + periodicità — vedi `references/percorsi-formativi.md`)

### Passo 2 (opzionale): Match scadenzario ↔ elenco personale

Se l'utente ha fornito anche l'**elenco personale aggiornato**:

1. **Nuove assunzioni** — dipendenti nell'elenco personale non presenti nello scadenzario: trattali come nuovi lavoratori da formare da zero. Usa il reparto/ruolo per determinare il livello di rischio (consulta `references/ateco-rischio.md`).
2. **Possibili uscite** — dipendenti nello scadenzario non presenti nell'elenco personale: segnalali nel documento come "da verificare (possibile dimissione/trasferimento)" senza inserirli nel piano formativo.
3. **Livello di rischio per reparto** — usa i ruoli/reparti dell'elenco personale per affinare l'assegnazione del rischio (es. ufficio amministrativo → BASSO, produzione/magazzino → ALTO).

### Passo 3: Match con i requisiti normativi

Confronta quanto presente nello scadenzario (integrato con le nuove assunzioni se disponibili) con i requisiti di `references/percorsi-formativi.md`:
- **VERDE** 🟢: corso effettuato e non in scadenza
- **GIALLO** 🟡: corso in scadenza entro 12 mesi
- **ROSSO** 🔴: corso mancante o già scaduto

### Passo 4: Sintetizzare per l'Executive Summary

Stessi principi del Ramo B (vedi Passo B4 sopra): distilla l'analisi in contatori, tabella per lavoratore e top-azioni urgenti. Se hai fatto il match con l'elenco personale, nella tabella lavoratori evidenzia le nuove assunzioni (con dicitura "nuovo ingresso") e NON inserire chi risulta uscito.

Vai alla **FASE 3**.

---

## FASE 3 — PRODUZIONE DELL'EXECUTIVE SUMMARY DOCX

Leggi `/mnt/skills/public/docx/SKILL.md` prima di scrivere qualsiasi codice.

**La struttura esatta è definita in `references/template-output.md`**: rispettala fedelmente. Il documento deve essere lungo **massimo 3 pagine** (tipicamente 2-3, a seconda del numero di lavoratori).

### Specifiche di stile (sintesi — dettagli in `template-output.md`)

- **Font**: Gill Sans MT per tutto il documento (fallback: Calibri)
- **Colore principale**: blu scuro `#003366` — per titoli, intestazioni tabelle, bordi principali
- **Semafori**: 🟢 🟡 🔴 come emoji (mai colori astratti)
- **Pagina**: A4, margini 2 cm top/bottom, 2,5 cm destra/sinistra

### Logo Overall Group (obbligatorio)

Il logo è in `assets/logo_overall.jpg` e **deve essere integrato** nel documento:

- **Nell'header di ogni pagina**: logo piccolo (altezza ≈ 40-50px / 1 cm) allineato a sinistra, con titolo "Executive Summary — [Ragione Sociale]" allineato a destra
- **Eventualmente nel frontespizio/intestazione**: se l'Executive Summary si apre con un titolo grande ("Sintesi per il cliente"), il logo può essere ripetuto sopra il titolo in dimensione maggiore

**Come inserirlo in docx-js**:

```javascript
const { ImageRun, Paragraph, Header } = require('docx');
const fs = require('fs');

// Leggi l'immagine dal percorso della skill
const logoBuffer = fs.readFileSync('/mnt/skills/user/checkupformazione81/assets/logo_overall.jpg');

// Dentro l'header
const header = new Header({
  children: [
    new Paragraph({
      children: [
        new ImageRun({
          data: logoBuffer,
          transformation: { width: 80, height: 32 }, // dimensioni in pixel
        }),
        new TextRun({ text: "\tExecutive Summary — [Nome Cliente]", ... }),
      ],
    }),
  ],
});
```

Se la riga di header diventa complessa, usa una tabella invisibile a 2 colonne (logo a sinistra, testo a destra) dentro l'header.

### Nome del file di output

Salva come:
`EXECUTIVE_SUMMARY_CHECKUP_81_[RAGIONE_SOCIALE]_[ANNO].docx`

Esempio: `EXECUTIVE_SUMMARY_CHECKUP_81_GASTRONOMIA_CISAMOLO_2026.docx`

### Procedura finale

1. Salva il file in `/mnt/user-data/outputs/`
2. Valida con `python /mnt/skills/public/docx/scripts/office/validate.py <file>`
3. Presenta con `present_files`
4. Scrivi in chat una breve sintesi (2-3 righe) con: stato complessivo, numero di criticità rilevate, una CTA per l'incontro.

### Testo iniziale consigliato da mostrare in chat dopo la generazione

> "L'Executive Summary del CHECKUP FORMAZIONE 81 è pronto. Stato complessivo: [🟢/🟡/🔴]. Sono state rilevate [N] criticità sostanziali e pianificate [N] azioni prioritarie. Il documento è scaricabile qui sopra ed è pronto per essere consegnato al cliente."

---

## NOTE IMPORTANTI

- **L'output è SEMPRE un Executive Summary di 2-3 pagine**, mai un documento integrale. Se l'utente chiede un documento più esteso (es. "voglio le schede complete di tutti gli attestati"), informalo che questa skill produce solo il summary sintetico, e suggerisci che — se serve — può richiedere separatamente un CHECKUP integrale come deliverable custom.
- Usa sempre "ipotetico" per l'organigramma del cliente NUOVO senza attestati.
- Segnala sempre le novità dell'ASR 17/04/2025 rispetto al precedente accordo, dove rilevanti.
- Non inventare dati: se mancano informazioni, segnalalo con "da definire" o chiedi conferma in chat.
- **Per l'analisi di attestati vecchi, consulta prima i file autentici in `references/`** (vedi sezione "Risorse disponibili"); ricorri alla ricerca web solo se il dato non è presente nei file locali e segnalalo in trasparenza.
- Il documento deve essere **pronto per essere consegnato al cliente finale**: tono commerciale-professionale, orientato all'azione, non autoreferenziale.
