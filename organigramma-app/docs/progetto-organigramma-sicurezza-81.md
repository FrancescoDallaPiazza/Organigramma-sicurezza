# Progetto — Organigramma Sicurezza 81

**Stato:** bozza di lavoro · **Versione:** 0.5 · **Data:** 24/06/2026
**Ambito:** Overall Group S.r.l. — strumento per mappare l'organigramma della sicurezza di un'azienda cliente dalla nomina alla formazione, e mantenerlo nel tempo (scadenze e aggiornamenti) ai sensi del D.Lgs. 81/08 e ASR 17/04/2025.

Questo documento è la **fonte unica della logica**. I dati dei corsi NON vivono qui: vivono nel file `ASR26-TabellaCorsi.xlsx`, che resta la fonte autorevole del catalogo. Il doc descrive *come* l'app ragiona; l'xlsx descrive *cosa* contiene il catalogo.

---

## 1. Principio guida

**Il motore è guidato dai corsi, non dai ruoli.**

Un ruolo non *è* una scadenza: un ruolo *genera un insieme di corsi obbligatori*. Ogni corso porta con sé periodicità, propedeuticità, modalità ammesse e scadenza transitoria, prese dal catalogo ASR. Per questo la stessa persona può avere più scadenze indipendenti (es. un lavoratore che guida il carrello: specifica 5 anni **e** carrello 5 anni, due scadenze separate).

---

## 2. Architettura dati (concettuale)

### 2.1 Catalogo corsi (sorgente: `ASR26-TabellaCorsi.xlsx`)

Catalogo a livello di **corso**. Ogni corso "Completo/iniziale" e il suo "Aggiornamento" sono **record separati**. Colonne e significato:

| Campo (xlsx) | Significato | Uso nel motore |
|---|---|---|
| Figura (col. A) | ruolo/figura dell'organigramma a cui il corso appartiene | raggruppamento ruolo-first |
| Obbligatorio ci sia? (col. B) | `SI`/`NO`: se la figura è di presenza obbligatoria in azienda | informativo / priorità organigramma |
| ASR S/N (col. C) | `S` = il regime ASR 17/04/2025 **si applica** al corso (e quindi al ruolo); `N` = disciplinato da altra norma (DM 388/2003, DM 02/09/2021, BLSD/IRC) | **gate di applicabilità ASR**; abilita il segnale transitoria |
| Corso (col. D) | nome del corso | identità del corso (l'id = slug del nome) |
| Durata min (ore) | durata, con scomposizione `T/P` (teoria/pratica) | informativo |
| Periodicità (anni) | vuota = iniziale (non si ripete); valorizzata = aggiornamento ogni N **anni** (uniformata: il BLSD ex-"24 mesi" ora `2`) | **calcolo scadenza** |
| CorsoPropedeutico | prerequisito → definisce la catena | flag "propedeutico mancante" (vedi §5) |
| Moduli Aggiuntivi | es. Cantieri 6h (art. 97 c.3-ter) impresa affidataria | informativo / sotto-scadenza dedicata |
| Aula / VideoC sin / Elearning | modalità ammesse (SI/NO) | vincoli ASR sull'erogazione |
| % min pres | presenza minima (90) | informativo |
| N° max part | max partecipanti (30; pratici con rapporto istruttore, es. 1:6) | informativo |
| Cons Mod Conv | (significato non rilevante per ora — escluso) | — |
| Scadenza ASR 25 (col. O) | termine transitorio **solo dove ASR lo prescrive** (il resto resta vuoto, non è un buco): **24/05/2026** (carroponte, ambienti confinati) · **24/05/2027** (Datore di lavoro già in carica; modulo Cantieri dei Dirigenti). La col. C dice *se* il regime ASR si applica; la col. O *quando* scade | flag "scadenza transitoria" (vedi §5) |
| Formazione pregressa/esoneri | crediti/esoneri (testo libero, a volte vincolato all'ATECO sull'attestato) | base degli esoneri (vedi §6) |

Pulizia già applicata al file (catalogo uniformato, 24/06/2026):
- **periodicità** uniformata in **anni** numerici (BLSD ex-"24 mesi" → `2`); il motore la converte in mesi all'import (anni × 12);
- **Scadenza ASR 25** ripulita a sole **date** (la riga Dirigenti, ex testo "24/05/2027 (per modulo Cantieri)", è ora data + qualificatore Cantieri già presente in *Moduli Aggiuntivi*);
- **Figura** riempita ovunque mancasse (BLSD → `Addetto BLSD`); restano senza figura solo le righe-sezione;
- **header** aggiunto sulla col. D (`Corso`); newline rimossi da *Durata*/*N° max part*; freeze a `A2`.

Le **righe-sezione** (`Attrezzature`, `Rischi particolari`) sono separatori e si scartano **per nome** (la col. B è vuota, non le marca).

Volumi reali del file: 76 righe = 1 header + 75 righe dato = **73 corsi** (60 `S` + 13 `N`) + 2 righe-sezione. *(Le cifre precedenti — md "76 corsi", handoff "74" — erano disallineate; questa è la verità del file.)*

### 2.2 Azienda

`ragioneSociale`, `ateco`, `livelloRischio` (Basso/Medio/Alto), `nAddetti`, `sede`. Il livello di rischio e l'ATECO/addetti **risolvono le varianti** dei corsi (vedi §7).

### 2.3 Persona

`nominativo` + due insiemi di assegnazioni (vedi §3):
- **ruoli/figure** (espandono nei corsi obbligatori della figura);
- **abilitazioni per attività/rischio** (corsi legati a cosa fa, non al ruolo).

Per ogni corso obbligatorio derivato, sulla persona si registrano le **evidenze**: data corso iniziale, data ultimo aggiornamento, eventuale n° attestato, ed eventuale flag **esonero iniziale** (vedi §6).

---

## 3. Due livelli di assegnazione

1. **Ruoli/figure** → si espandono automaticamente nei corsi obbligatori della figura.
   Es.: Preposto ⟶ {Preposti 12h + Aggiornamento 6h/2a}; Lavoratore ⟶ {Generale 4h + Specifica B/M/A + Aggiornamento 6h/5a}; Addetto antincendio ⟶ {L1/L2/L3 + Aggiornamento 5a}.
2. **Abilitazioni per attività/rischio** → corsi che NON dipendono dal ruolo ma dall'attività svolta: carrello, PLE, gru, lavori in quota, PES/PAV/PEI, ambienti confinati. Si pescano dal catalogo e si attaccano alla persona.
   Senza questo livello, gran parte del file ASR non avrebbe dove vivere.

---

## 4. Anello di composizione

L'anello è ancorato alla **persona**; la formazione è **derivata** dal catalogo, non impostata a mano:

```
Persona
  → assegno RUOLI (+ ABILITAZIONI)
    → il sistema DERIVA i corsi obbligatori dal catalogo
      → per ogni corso inserisco le EVIDENZE (date, eventuale esonero)
        → il motore calcola lo STATO
          → SCADENZARIO aggregato (per persona / per azienda / portafoglio)
```

**Ingresso ruolo-first (deciso).** Il *flusso di lavoro* parte dalle figure/posizioni dell'organigramma: si crea/seleziona il ruolo → l'app deriva i corsi → si associa la/le persone → si inseriscono le evidenze. I *dati* restano però agganciati alla **persona** sotto il cofano: chi ricopre più ruoli (es. DL + RSPP + addetto antincendio) compare in più caselle dell'organigramma ma è una sola anagrafica, e le evidenze sono chiave persona+corso (un corso condiviso, es. generale lavoratori, non si duplica). Lo scadenzario aggrega comunque per persona, per azienda e per portafoglio.

Differenza rispetto al flusso "Ruolo → formazione → persona → evidenze → scadenzario": è lo stesso loop, con l'ingresso ruolo-first sopra descritto. Le sole vere aggiunte sono il livello *abilitazioni per attività* e i due flag di §5.

---

## 5. Logica di stato (per ciascun corso)

Stato base dall'evidenza + periodicità del catalogo:

- **nessuna evidenza** → `Mancante`
- **evidenza presente** → scadenza = (ultima data: aggiornamento se c'è, altrimenti iniziale) + periodicità → `In regola` / `In scadenza` (entro soglia preavviso) / `Scaduto`
- periodicità assente (corso una tantum) → `In regola (senza scadenza)`

Due **segnali aggiuntivi**, indipendenti dallo stato base:

- **Propedeutico mancante** — solo nei casi previsti da §6 (non scatta se la persona è esonerata dall'iniziale). Il prerequisito è soddisfatto se il corso propedeutico è **realmente svolto OPPURE coperto da credito** (es. generale lavoratori 4h = credito permanente → soddisfa per sempre la propedeuticità del preposto).
- **Scadenza transitoria ASR 25** — promemoria sul termine di messa a regime (24/05/2026 standard; 24/05/2027 Cantieri / prima formazione DL già in carica).

---

## 6. Esoneri / crediti (oggetto di prima classe)

Definiti dalla colonna *Formazione pregressa/esoneri*. Regola corretta:

- **La persona ha esonero sull'iniziale** →
  - iniziale = `Assolto per credito/esonero` (non `Mancante`);
  - **prerequisito NON richiesto** → nessun flag "propedeutico mancante";
  - resta **solo lo scadenzario dell'aggiornamento** (il credito copre l'iniziale, **non** azzera la periodicità).
- **La persona NON ha esonero** →
  - l'iniziale richiede l'evidenza (data), altrimenti `Mancante`;
  - **qui** scatta il controllo del prerequisito: se manca (corso non svolto e non creditato) → flag `Propedeutico mancante`.

> Correzione registrata: il prerequisito (propedeuticità) è richiesto **solo a chi non ha esonero dalla formazione iniziale**. Es.: "preposto formato ma manca formazione lavoratore" genera l'allarme **solo se** quel preposto non è esonerato/creditato sull'iniziale lavoratore.

Trattamento degli esoneri — **scelta A (manuale guidato)**, con poche derivazioni automatiche ovvie. Operativamente:

- sul singolo incarico c'è una spunta **"esonero formazione iniziale"** + campo nota della base;
- attivando la spunta, l'app mostra a video le **possibili fonti di esonero per quella figura** (vedi §6.1), così la scelta è guidata e non a memoria;
- alcune derivazioni **role-internal** sono suggerite in automatico (l'utente conferma):
  - generale lavoratori svolta/creditata → soddisfa la propedeuticità di preposto e degli altri corsi che la richiedono;
  - ruolo Dirigente presente sulla persona → suggerisce esonero sul corso Datore di lavoro;
  - ruolo RSPP / ASPP / DL-RSPP / CSP-CSE presente → suggerisce esonero sul corso Datore di lavoro.
- evoluzione futura: opzione B (derivazione piena codificando tutta la colonna esoneri).

### 6.1 Fonti di esonero per figura (helper guidato)

Sintesi della colonna *Formazione pregressa/esoneri* del catalogo, raggruppata per figura. È il contenuto che l'app propone quando si attiva la spunta esonero.

| Figura / corso iniziale | Iniziale esonerata se… | Testo-fonte nel catalogo |
|---|---|---|
| **Datore di lavoro** | già DDL-RSPP, o RSPP (Mod. A+B+C), o ASPP (Mod. A+B), o CSP/CSE; **oppure** corso DL conforme ASR 2025 entro 24/05/2025; **oppure** corso Dirigenti Acc. 2011 | "Credito totale se il DL ha: …" |
| **DL-RSPP — modulo Comune** | già DDL-RSPP rischio basso 16h | "Credito totale se DDL RSPP r. basso 16 ore" |
| **DL-RSPP — integrativo di settore** | già DDL-RSPP medio 32h / alto 48h, **con lo stesso ATECO riportato sull'attestato** | "…Attestato deve riportare ATECO …" |
| **Dirigente** | corso Dirigente Acc. 21/12/2011 (**non** copre il modulo Cantieri) | "Credito totale se Corso Acc 21/12/2011, non per Cantieri" |
| **Preposto** | corso Preposto Acc. 21/12/2011 (se >2 anni → aggiornarsi entro 24/05/2026) | "Credito totale se Corso Acc 21/12/2011" |
| **Lavoratore (generale + specifica B/M/A)** | corso Lavoratori Acc. 21/12/2011 | "Credito totale se Corso Acc 21/12/2011" |
| **RLS (iniziale 32h)** | qualifica CSP/CSE, RSPP o ASPP | "Credito totale se CSP/CSE - RSPP - ASPP" |
| **Abilitazioni attrezzature** (carrello, PLE, gru, trattori, escavatori) | abilitazione pregressa conforme Acc. 22/02/2012 | "Credito totale se Credito tot. 22/02/2012" |
| **Antincendio · Primo soccorso · BLSD · RSPP/ASPP prof. · ambienti confinati · lavori in quota · PES/PAV/PEI · carroponte** | nessun esonero codificato nel catalogo (cella vuota) | — |

> Attenzione: l'esonero sull'**aggiornamento** RLS ("se Dirigente/Preposto/Lavoratore") riguarda l'aggiornamento, non l'iniziale → va trattato come nota distinta, non come esonero-iniziale.

---

## 7. Risoluzione automatica delle varianti

Dove il catalogo lascia una scelta, l'app la risolve dall'anagrafica (sempre forzabile a mano):

| Variante | Risolta da |
|---|---|
| Lavoratore – specifica Basso/Medio/Alto | livello di rischio (anagrafica) |
| DL-RSPP – modulo integrativo di settore | settore / ATECO |
| RLS – aggiornamento 4h (≤50) / 8h (>50) | n° addetti |

---

## 8. Date chiave ASR 17/04/2025

- **24/05/2025** — entrata in vigore.
- **24/05/2026** — fine periodo transitorio (corsi vecchio formato); termine per portare a regime es. il preposto biennale se ultimo corso/aggiornamento ante 24/05/2023.
- **24/05/2027** — prima formazione obbligatoria dei datori di lavoro già in carica; modulo Cantieri.

---

## 9. Decisioni aperte (DA DECIDERE)

**A/B — Trattamento esoneri**
- **A (manuale):** spunta "esonero formazione iniziale" sul singolo incarico + nota della base. Rapido, robusto, decisione caso per caso.
- **B (derivato):** l'app deduce gli esoneri codificando le regole della colonna (es. ha la generale → preposto creditato; è dirigente → DL creditato), con forzatura manuale.
- *Proposta:* partire da **A**, con poche derivazioni ovvie automatiche (generale lavoratori → prerequisito preposto/altri); B come evoluzione. **→ scelta: A (manuale guidato) — vedi §6 e §6.1. CHIUSA.**

**1 — Punto d'ingresso**
- persona-first (come l'app attuale) oppure anche modalità "requisito → assegna persona" (comodo per coprire un buco: "chi mi fa l'antincendio?").
- *Proposta:* persona-first ora, l'altra dopo. **→ scelta: RUOLO-FIRST (ingresso dalle figure dell'organigramma; dati agganciati alla persona — vedi §4). CHIUSA.**

**2 — Ampiezza del catalogo**
- caricare *tutto* il file ASR oppure, per ora, solo le figure dell'organigramma + le abilitazioni più frequenti.
- **→ scelta: TUTTO il file ASR (76 corsi, incluse attrezzature e rischi particolari; escluse solo le righe-sezione). CHIUSA.**

---

## 10. Stato realizzativo e note tecniche

- **v1 (fatto):** app figura-level con un solo periodo di aggiornamento per ruolo. Due forme: artifact React (`window.storage`) e file HTML standalone offline (`localStorage`). Catalogo periodi allineato a `checkupformazione81/references/percorsi-formativi.md`.
- **v2 (costruita):** rifondazione su **catalogo corsi** (questo documento). File `organigramma-sicurezza-81-v2.html` (standalone, localStorage). Include Fascicolo documentale e le logiche di §11.
- **v2.1 (costruita):** **motore agganciato al catalogo** (decisione "opzione 2"). Periodicità, modalità, scadenze transitorie ed esoneri sono **letti dalle righe del catalogo** tramite una mappatura tipo→corso (per id). Ordine di risoluzione di ogni periodo: *override manuale (tabella Regole) → valore da catalogo → default curato*. I tipi fuori catalogo (RSPP/ASPP professionista, Medico Competente) restano "curati". **Aggiornamento del catalogo dall'app:** pulsante "Aggiorna catalogo da Excel" (SheetJS incorporato, offline) che ricarica un `ASR26-TabellaCorsi.xlsx` aggiornato e ne fa seguire il motore; "Ripristina predefinito" per tornare al catalogo incorporato. Vincolo: gli id corso derivano dal nome — se nel file cambia il *nome* di un corso, quel tipo si scollega e ricade sul default (l'app lo segnala). Sostituisce la scheda REGOLE figura-level con il catalogo, e in Organigramma si assegnano *ruoli + abilitazioni* da cui derivano i corsi.
- **v2.2 (costruita):** **app allineata al catalogo uniformato** (24/06/2026). `buildCatalogo` leggeva ancora il layout vecchio (ASR S/N in col. A): corretti gli indici al layout a 16 colonne reale (Figura A, Obbligatorio B, ASR C, Corso D …), così l'import "Aggiorna catalogo da Excel" è di nuovo idempotente. Il record di catalogo ora porta anche `figura`/`obbligatorio`. `CATALOGO_DEFAULT` rigenerato dal file uniformato → **73 corsi** (prima 74: il corso *Segnaletica stradale per cantieri*, assente dal file, è stato **escluso su decisione del committente** — vedi v2.5). La scheda **Catalogo & Regole** mostra ora anche le colonne **Figura** e **Transitoria** (col. O). Il testo della colonna **esoneri/formazione pregressa** (es. crediti RSPP DL coi vincoli ATECO per modulo) è preservato con gli **a-capo** e reso a **punti elenco** nell'helper di esonero (prima veniva appiattito su una riga sola). Nota d'uso: chi ha già usato l'app vede il catalogo salvato in `localStorage` finché non preme "Ripristina predefinito" o "Aggiorna catalogo da Excel".
- **v2.3 (costruita):** tre interventi su Organigramma/Cruscotto. (a) **Date in testo `gg/mm/aaaa`** (niente più rotella anno): tutti i campi data — nomina, formazione iniziale, ultimo aggiornamento, fascicolo — sono input testo con parsing tollerante (`parseHuman`) e bordo rosso se non valido; internamente restano ISO. (b) **Formazione mancante = Non Conformità**: continua a confluire nel cruscotto (KPI "Formazioni mancanti" + lista "Da gestire subito"), ora con etichetta **NC** esplicita. (c) **Evidenza attestato**: per ogni formazione presente (stato ≠ mancante) la card mostra la spunta *"Attestato agli atti"* + rif.; se manca, badge *"COSA DA FARE: acquisire/allegare attestato"* e la voce confluisce nel cruscotto (KPI "Attestati da acquisire", incluso fra le cose da gestire). L'evidenza è ancorata a persona+corso (coerente con §3).
- **v2.4 (costruita):** rischio ATECO, integrativo DL-RSPP, attestato come allegato. (a) **Allegato IV ASR 17/04/2025 incorporato** (88 divisioni ATECO 2007, fonte: libreria `formazione-81-utils`): `rischioAteco(codice)` → livello BASSO/MEDIO/ALTO + ore. **ATECO ora obbligatorio** in anagrafica, scelto da una **combobox filtrabile** (`datalist`, 88 divisioni — ricerca per codice o per attività); alla compilazione il livello di rischio è **calcolato in automatico** (override manuale ammesso per il DVR, Interpelli MLPS 11/2013 e 1/2025). (b) **Pill rischio** colorata e cliccabile (dettaglio + fonte) in **header, anagrafica e cruscotto**. (c) **DL-RSPP art. 34**: la formazione iniziale è ora **Comune (8h) + modulo integrativo di settore** scelto via selettore (Mod 1 A 01-02 · Mod 2 A 03 · Mod 3 F · Mod 4 C 19-20), **pre-selezionato da ATECO** (`modIntegrativoAteco`) e modificabile; `resolve` compone "8h Comune + Nh ModX". Il legame è **per divisione ATECO 2007** (01-02→Mod1, 03→Mod2, 41-43→Mod3, 19-20→Mod4) ed è **dinamico**: al cambio ATECO il modulo si ri-propone (salvo override manuale, flag `varManuale`); sulla card un badge mostra «ATECO NN → Mod X». ⚠️ Solo questi 4 settori hanno un modulo dedicato nel catalogo; per gli altri ATECO la proposta è «solo Comune».
- **v2.5 (costruita):** decisioni chiuse dal committente. (a) **Modello integrativo DL-RSPP confermato completo**: i 4 moduli (A 01-02, A 03, F, C 19-20) sono l'insieme voluto; per ogni altro ATECO vale il **solo modulo Comune** (8h) — comportamento corretto, non una lacuna. Il legame ATECO→modulo è ora anche **dinamico** (badge «ATECO NN → Mod X» sulla card; ri-proposta al cambio ATECO salvo override). (b) **Segnaletica stradale rimossa**: la figura *Segnaletica stradale (cantieri)* è stata tolta da `TIPI` e `MAP` (non utilizzata e assente dal catalogo). Risultato: la **mappatura tipo→corso non ha più voci scollegate** (0 orfani). Aggiunta una guardia: incarichi di tipo sconosciuto non mandano in errore la card.
- **v2.6 (costruita):** (a) **pill rischio nell'intestazione di ogni gruppo-figura** dell'organigramma (banda `tipohead`), utile a valutare gli esoneri legati al livello di rischio pregresso. (b) **DL-RSPP: rimosso il menù a tendina** del modulo integrativo — non è una scelta dell'utente ma è **determinato dall'ATECO**: la card propone direttamente i due corsi (badge «ATECO NN → Mod X» + "Iniziale: 8h Comune + Nh ModX"), aggiornati automaticamente al cambio ATECO. Le altre figure con scelta legittima (lavoratore B/M/A, antincendio L1/L2/L3, primo soccorso A/B-C, BLSD, RLS) mantengono il selettore.
- **v2.7 (costruita):** DL-RSPP — la "Formazione iniziale" diventa un blocco **"Formazione iniziale — moduli"** con due righe distinte (Modulo Comune 8h + Modulo integrativo di settore), ciascuna con **data propria** (gg/mm/aaaa) e **attestato proprio** da allegare. `inc.moduli={comune,integr}`; `syncRsppIniziale()` calcola `dataIniziale` = data più recente quando entrambi i moduli richiesti sono datati (solo Comune se l'ATECO non ha integrativo), così l'aggiornamento parte dal completamento reale. Il cruscotto elenca i due attestati separatamente ("Attestato Comune" / "Attestato modulo integrativo"). Migrazione automatica: per incarichi esistenti la vecchia data/attestato iniziale confluisce nel Modulo Comune. (d) **Attestato = allegato file**: sulla card si carica il PDF/immagine (<2 MB); finché non è allegato resta **COSA DA FARE** e confluisce nel cruscotto ("Attestati da acquisire").
- **Branding:** palette Overall (blu `#1F3864`/`#2E5496`, semaforo `#C6E0B4`/`#FFE699`/`#F8CBAD`), font Calibri/Carlito.
- **Persistenza:** localStorage (HTML standalone) o window.storage (artifact). Per uso multi-postazione servirà più avanti un archivio condiviso.

## 11. Logiche di compilazione aggiuntive

**Individuazione del Datore di lavoro.** All'assegnazione del ruolo DL si seleziona la *base di individuazione*: **Visura camerale** (rappresentante legale) oppure **Delega esplicita (art. 16)**. Il documento citato è **creato in automatico nel Fascicolo documentale** dell'azienda con stato `Da acquisire (COSA DA FARE)` finché non è marcato presente / allegato. Logica estendibile alle deleghe dei dirigenti.

**Fascicolo documentale (nuovo).** Ogni azienda ha un fascicolo: ogni voce = {nome, riferimento, stato (presente/da fare), data, nota, file allegato opzionale, collegamento all'incarico}. Le COSE DA FARE documentali risalgono nel Cruscotto (KPI + elenco). Allegati in localStorage (file piccoli).

**Esclusività RSPP.** Il ruolo RSPP può essere assunto dal DL (art. 34) o da un professionista interno/esterno. Regola: se è presente **Datore di lavoro–RSPP (art. 34)**, i ruoli **RSPP professionista** e **ASPP** non sono selezionabili (e viceversa). RSPP e ASPP restano compatibili tra loro.

---

## 12. Fonti

- `ASR26-TabellaCorsi.xlsx` — catalogo corsi (fonte autorevole dei dati).
- `analisi-dvr-81/references/base_normativa_tecnica.md` (sez. 7) — disposizioni regionali.
- `checkupformazione81/references/percorsi-formativi.md` — quadro figure/percorsi.
- D.Lgs. 81/08; ASR 17/04/2025; DM 02/09/2021; DM 388/2003; DPR 177/2011; ASR 53/2012.
