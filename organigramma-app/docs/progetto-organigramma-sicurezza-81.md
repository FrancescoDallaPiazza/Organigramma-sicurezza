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

`ragioneSociale`, `piva`, `cf`, `ateco`, `livelloRischio` (Basso/Medio/Alto), `nAddetti`, `sede`. La **P.IVA** è la chiave prioritaria per il match azienda in import (poi il CF, poi la ragione sociale). Il livello di rischio e l'ATECO/addetti **risolvono le varianti** dei corsi (vedi §7).

### 2.3 Persona

**Anagrafica normalizzata (v2.10).** Le persone vivono in un registry per azienda `ST.az.persone[] = {id, cognome, nome, cf, nato:{data,sesso,comune,prov,stato,estero}, flagRivedere}`. Gli incarichi referenziano `personaId`; `inc.persona` resta come **label denormalizzata** (`COGNOME Nome`) mantenuta sincronizzata, così l'aggregazione/scadenzario per persona non cambia. **Chiave identità:** codice fiscale se presente, altrimenti `cognome+nome` normalizzati (de-dup di chi ricopre più ruoli).

**Cognome e Nome obbligatori, sempre forzati e memorizzati in STAMPATELLO; CF facoltativo.** Inserito il CF, l'app **deriva** data e luogo di nascita (comune+prov o stato estero, via tabella Belfiore incorporata ~10.000 codici, soppressi inclusi) e sesso; gestisce **omocodia** (cifre L–V) e valida il **carattere di controllo**. Esegue un **cross-check** dei primi 6 caratteri contro cognome/nome: in caso di discordanza mostra un **avviso non bloccante** (non impedisce il salvataggio). Gestione anagrafica nella scheda **Persone**.

Due insiemi di assegnazioni (vedi §3): **ruoli/figure** e **abilitazioni per attività/rischio**. Per ogni corso derivato si registrano le **evidenze**: data iniziale, data ultimo aggiornamento, eventuale n° attestato, flag **esonero iniziale** (vedi §6).

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

Gli esoneri/crediti sono di **prima classe**. Si dividono in **due famiglie**, da tenere distinte nel motore (oggi sono mischiate nel testo libero della colonna *Formazione pregressa/esoneri* del catalogo):

- **Famiglia 1 — pregresso normativo.** Un corso del vecchio Accordo credita il corrispondente corso ASR. È testo del catalogo (`record.esonero`), già reso come fonti nell'helper. Esempi: Lavoratori 2011→Lav; Dirigenti 2011→Dirigente (non Cantieri); Preposto 2011→Preposto (se >2 anni → agg. ≤24/05/2026); attrezzature Acc. 22/02/2012→attrezzature; DL ASR-conforme ≤24/05/2025→DL; DDL-RSPP basso 16h→Comune; DDL-RSPP medio 32h/alto 48h **stesso ATECO**→integrativo.
- **Famiglia 2 — credito tra ruoli (Allegato III ASR 17/04/2025).** Possedere il ruolo X credita verso il ruolo Y. È la matrice di §6.1, **ora codificata** (prima solo poche derivazioni verso DL).

> **Gerarchia di visualizzazione (v2.15).** Sulle figure coperte dalla matrice (Datore, Dirigente, Preposto, RLS, Lavoratore-Generale/Specifica) la casella *"Possibili fonti di esonero"* mostra **solo i crediti di ruolo (Famiglia 2)**; il pregresso normativo (Famiglia 1) **non** è più elencato lì — il corso pregresso conforme si registra inserendo la **data** del corso (la persona è formata). **Famiglia 1 resta** invece per le figure **fuori matrice** (abilitazioni attrezzature → credito Acc. 22/02/2012) e per i **moduli DL-RSPP** (DDL-RSPP basso→Comune, medio/alto stesso ATECO→integrativo), che non hanno equivalente nella matrice ruolo-ruolo.

Regola di stato invariata (vedi §5): persona **con esonero sull'iniziale** → iniziale `Assolta per credito/esonero`, **prerequisito non richiesto**, resta solo lo scadenzario dell'aggiornamento (il credito copre l'iniziale, non azzera la periodicità).

---

### 6.1 Famiglia 2 — Matrice credito tra ruoli (Allegato III)

Legenda: **T** totale (iniziale assolta) · **T\*** totale **solo se il ruolo-fonte è svolto nella stessa azienda** (vedi D3) · **P** parziale (→ nota, vedi D2) · **F** frequenza (nessun credito) · **—** stesso ruolo / n.a.

#### Matrice madre (Allegato III, pag. 130) — *possiede ▼ / credito verso ▶*

| Possiede ▼ | RLS | DL (16h) | Lav. Gen | Lav. Spec | Dirigente | Preposto |
|---|---|---|---|---|---|---|
| RSPP (A+B+C)        | T | T | T | T\* | T | T\* |
| ASPP (A+B)          | T | T | T | T\* | T | T\* |
| CSP/CSE (coord.)    | T | T | T | T\* | T | T\* |
| DL-RSPP (art. 34)   | **F** | T | T | T\* | T | T\* |
| DL (art. 37)        | F | — | T | T\* | T | T\* |
| RLS                 | — | F | T | F | T | **T** |
| Lavoratore Generale | F | F | — | F | F | F |
| Lavoratore Specifica| F | F | — | — | F | F |
| Dirigente           | F | T | T | T\* | — | T\* |
| Preposto            | F | F | F | F | F | — |

Asimmetrie da non perdere: **DL-RSPP → RLS = F** (essere RSPP di sé stessi non credita l'RLS); **RLS → Preposto = T pieno** (non T\*).

#### Moduli Cantieri / casi senza credito (Allegato III, pag. 131)

| Possiede ▼ | DL mod. cantieri | Dirig. mod. cantieri | Lav. sospetto inquin. | Operatore attrezz. |
|---|---|---|---|---|
| RSPP base / ASPP base            | F | F | F | F |
| RSPP (A+B com.+**B-SP3**+C)      | T | T | F | F |
| ASPP (A+B com.+**B-SP3**)        | T | T | F | F |
| CSP/CSE                          | T | T | F | F |
| DL-RSPP **integrativo 3**        | T | T | F | F |
| DL cantiere ↔ Dirigente cantiere | T (incrociato) | T (incrociato) | F | F |

Il credito sui Cantieri arriva **solo da chi ha la specializzazione costruzioni** (B-SP3 / integrativo 3). *Sospetto inquinamento* e *operatore attrezzature*: nessun credito da ruoli (attrezzature → solo Famiglia 1, Acc. 22/02/2012). I moduli Cantieri restano *moduli aggiuntivi*, non figure autonome dell'organigramma → derivazione automatica **non** attiva su questi (solo nota), almeno in v2.13.

#### Percorso professionale PARZIALE (Allegato III, pag. 127-129) → **nota, non esonero** (D2)

Crediti **parziali** verso RSPP/ASPP/CSP-CSE/moduli DL-RSPP partendo da moduli A o A+B già svolti (es. *RSPP con solo Mod. A* → credito Modulo giuridico 28h; deve frequentare tecnico 52h + metodologico 16h + pratica 24h). Non rappresentabili con l'esonero binario: l'app li mostra come **nota informativa** ("credito parziale — frequenza dei moduli residui necessaria"), senza attivare l'iniziale-assolta.

---

### 6.2 Sintesi per figura — *iniziale esonerata (TOTALE) se la persona possiede…*

Combina Famiglia 2 (matrice, ruolo nella stessa azienda) + Famiglia 1 *(F1)*. `(*)` = T\*, soggetto a D3.

- **Datore di lavoro 16h** → RSPP · ASPP · CSP/CSE · DL-RSPP · Dirigente(*) · *(F1)* DL ASR-conforme ≤24/05/2025 · Dirigenti 2011
- **DL-RSPP — Comune 8h** → *(F1)* DDL-RSPP basso 16h · *(P)* RSPP A+B+C
- **DL-RSPP — integrativo settore** → *(F1)* DDL-RSPP medio/alto **stesso ATECO** · *(P)* CSP/CSE→integr. 3, RSPP→integrativi
- **Dirigente 12h** → RSPP · ASPP · CSP/CSE · DL-RSPP · DL · RLS(*) · *(F1)* Dirigenti 2011 (**non** Cantieri)
- **Preposto 12h** → **RLS (pieno)** · RSPP/ASPP/CSP-CSE/DL-RSPP/DL/Dirigente (*) · *(F1)* Preposto 2011 (se >2 anni → agg. ≤24/05/2026)
- **Lavoratore — Generale 4h** → RSPP · ASPP · CSP/CSE · DL-RSPP · DL · RLS · Dirigente · *(F1)* Lavoratori 2011
- **Lavoratore — Specifica B/M/A** → RSPP/ASPP/CSP-CSE/DL-RSPP/DL/Dirigente (*) · *(F1)* Lavoratori 2011 — *(RLS e Preposto = frequenza)*
- **RLS 32h** → **solo** RSPP · ASPP · CSP/CSE *(aggiornamento RLS: esonero distinto se Dirigente/Preposto/Lavoratore — riguarda l'agg., non l'iniziale; resta nota a sé)*
- **Antincendio · Primo soccorso · BLSD · ambienti confinati · lavori in quota · PES/PAV/PEI · carroponte** → Allegato III non li tratta (regimi DM/IRC dedicati): nessun esonero da ruoli.

---

### 6.3 Decisioni chiuse (D1-D4) e implicazioni dato/UI

**D1 — Split Lavoratore (sì).** L'iniziale lavoratore si scinde in **due righe**, come i due moduli DL-RSPP:
- *Generale 4h* → una tantum, **nessun aggiornamento = credito permanente**; è anche fonte di propedeuticità (soddisfa per sempre il prerequisito del preposto ecc.);
- *Specifica B/M/A* (4/8/12h per rischio) → con **aggiornamento 6h/5a**; la scadenza della figura nasce da qui.
- Esonero/credito **per riga** (non a livello figura). Box "Formazione iniziale — moduli" con le due righe data+attestato proprie. `iniziale assolta` = Generale assolta **e** Specifica assolta (datate+attestate o coperte da esonero).

Modello: `inc.moduli = { generale:{data, file, fileNota, esonero:{flag,nota,file}}, specifica:{variante, data, file, fileNota, esonero:{…}, aggiornamento} }` (parallelo a `datoreRspp`). La variante (basso/medio/alto) resta risolta da rischio azienda (§7), forzabile.
*Migrazione:* vecchio `inc.dataIniziale`/attestato lavoratore → confluisce in `moduli.specifica`; `moduli.generale` parte **da compilare** (todo). Se sul vecchio incarico c'era l'esonero figura-level con fonte Lavoratori 2011 → si applica a **entrambe** le righe (il credito 2011 copre generale+specifica), modificabile a mano.

**D2 — TOTALE vs PARZIALE (parziale = nota).** L'esonero binario rappresenta **solo il TOTALE**. I casi PARZIALE (§6.1 pag. 127-129) si mostrano come **nota informativa** sull'incarico, senza attivare iniziale-assolta né rimuovere il prerequisito.

**D3 — Clausola "stessa azienda" (T\*) → spunta default-no.** L'app è per-azienda: il ruolo-fonte è di norma svolto **in questa** azienda → T\* vale come T. Sull'esonero la cui fonte è un ruolo **T\*** compare la spunta **"qualifica conseguita in altra azienda"** (default **OFF**). Se attivata → il credito **decade** (serve frequenza): l'iniziale torna `Mancante`/da svolgere e l'app lo segnala. Le fonti **T pieno** (es. RLS→Preposto, ruoli→Lav. Generale) non mostrano la spunta.

**D4 — Opzione B: derivazione automatica dalla matrice (sì), guidata.** Quando una persona ha un ruolo-fonte (incarico nell'azienda) e si apre un incarico-bersaglio con cella **T** (o **T\*** non flaggata altra-azienda), l'app **propone** di attivare l'esonero sull'iniziale del bersaglio, citando la fonte; **l'utente conferma** (mai auto-attivazione silenziosa — coerente §4 import). Estende le derivazioni odierne (solo verso DL) a tutta la matrice (verso Preposto, Dirigente, Lavoratore Gen/Spec, RLS). Le celle **F**, **—**, **P** non generano proposta. Cantieri/attrezzature: nessuna proposta automatica (solo nota).

---

### 6.4 Struttura dati per l'implementazione (Famiglia 2)

Chiavi-ruolo = chiavi TIPI dell'app (allineare in fase di build: rspp/aspp professionista, cspcse, datoreRspp, datore, rls, lavoratore[generale|specifica], dirigente, preposto).

```js
// Allegato III pag.130 — possiede → { verso: stato }
// stati: 'T' (totale) | 'Tstar' (totale solo stessa azienda) | 'F' | '-' (na)
const CREDITI_RUOLI = {
  rspp:      {rls:'T',  dl:'T', lavGen:'T', lavSpec:'Tstar', dirigente:'T', preposto:'Tstar'},
  aspp:      {rls:'T',  dl:'T', lavGen:'T', lavSpec:'Tstar', dirigente:'T', preposto:'Tstar'},
  cspcse:    {rls:'T',  dl:'T', lavGen:'T', lavSpec:'Tstar', dirigente:'T', preposto:'Tstar'},
  dlRspp:    {rls:'F',  dl:'T', lavGen:'T', lavSpec:'Tstar', dirigente:'T', preposto:'Tstar'},
  dl:        {rls:'F',  dl:'-', lavGen:'T', lavSpec:'Tstar', dirigente:'T', preposto:'Tstar'},
  rls:       {rls:'-',  dl:'F', lavGen:'T', lavSpec:'F',     dirigente:'T', preposto:'T'},
  lavGen:    {rls:'F',  dl:'F', lavGen:'-', lavSpec:'F',     dirigente:'F', preposto:'F'},
  lavSpec:   {rls:'F',  dl:'F', lavGen:'-', lavSpec:'-',     dirigente:'F', preposto:'F'},
  dirigente: {rls:'F',  dl:'T', lavGen:'T', lavSpec:'Tstar', dirigente:'-', preposto:'Tstar'},
  preposto:  {rls:'F',  dl:'F', lavGen:'F', lavSpec:'F',     dirigente:'F', preposto:'-'}
};
// D4: propone esonero se cell==='T' || (cell==='Tstar' && !inc.esonero.altraAzienda)
```

---

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
- *Proposta originaria:* partire da **A** (manuale guidato), B come evoluzione. **→ Scelta definitiva: opzione B (derivazione piena) guidata — decisioni D1-D4 chiuse (vedi §6). D1 (split Lavoratore) implementata in v2.13; D2-D4 (motore di derivazione tra ruoli, clausola stessa-azienda, parziale=nota) implementate in v2.14. CHIUSA.**

**1 — Punto d'ingresso**
- persona-first (come l'app attuale) oppure anche modalità "requisito → assegna persona" (comodo per coprire un buco: "chi mi fa l'antincendio?").
- *Proposta:* persona-first ora, l'altra dopo. **→ scelta: RUOLO-FIRST (ingresso dalle figure dell'organigramma; dati agganciati alla persona — vedi §4). CHIUSA.**

**2 — Ampiezza del catalogo**
- caricare *tutto* il file ASR oppure, per ora, solo le figure dell'organigramma + le abilitazioni più frequenti.
- **→ scelta: TUTTO il file ASR (76 corsi, incluse attrezzature e rischi particolari; escluse solo le righe-sezione). CHIUSA.**

**3 — Anagrafica persona e codice fiscale (v2.10)**
- **→ scelta:** registry `persone[]` con `personaId` sugli incarichi; chiave identità **CF se presente, altrimenti cognome+nome** normalizzati. CHIUSA.
- **→ scelta:** derivazione dal CF **completa** (data + sesso + **comune/stato** via tabella Belfiore incorporata) + check del carattere di controllo + omocodia. CHIUSA.
- **→ scelta:** **cross-check** cognome/nome vs CF = **avviso non bloccante**. CHIUSA.
- **→ scelta:** migrazione **non distruttiva** delle vecchie stringhe `persona` → `{cognome:<stringa>, nome:""}` con flag **da rivedere** (nessuno split automatico). CHIUSA.

**4 — Condivisione dati in team (v2.10)**
- localStorage non è condivisibile. **→ scelta: Opzione A — file condiviso via File System Access API**, granularità **per azienda** (.json in cartella sincronizzata/di rete), con `schemaVersion`/`updatedAt`/`updatedBy` e **avviso di conflitto** sul salvataggio. Backend cloud (opzione C) rimandato a se servirà editing concorrente reale (con valutazione GDPR: CF + dati formativi = dati personali). CHIUSA.

---

## 10. Stato realizzativo e note tecniche

- **v2.10 (costruita):** **anagrafica persona normalizzata + CF + condivisione su file.** (a) Registry `ST.az.persone[]` con `personaId` sugli incarichi; `inc.persona` mantenuta come label denormalizzata sincronizzata (`migraPersone()` migra le vecchie stringhe in modo non distruttivo, le marca *da rivedere*, e risincronizza le label; idempotente). Identità per **CF** o, in mancanza, `cognome+nome` normalizzati. (b) Inserimento incarico con **Cognome\* / Nome\* + CF facoltativo** (oppure selezione di persona esistente); scheda **Persone** per l'anagrafica. (c) **Motore CF** incorporato (puro, testato): validità + carattere di controllo, data/sesso, **comune/stato** via tabella **Belfiore** incorporata (~10.000 codici, soppressi inclusi), **omocodia** (cifre L–V), **cross-check** cognome/nome → avviso **non bloccante**. (d) **Condivisione — Opzione A**: salvataggio/apertura diretta su file `.json` (File System Access API, Chrome/Edge) **per azienda** in cartella condivisa, busta `{schemaVersion, updatedAt, updatedBy, az}`, **avviso di conflitto** se il file è stato modificato da altri dopo l'ultimo caricamento; fallback Esporta/Importa JSON (ora include `persone`). **(e) DL-RSPP — aggiornamento per modulo:** ogni modulo (Comune, integrativo) ha la propria data di aggiornamento; la scadenza dell'aggiornamento della figura è calcolata dal **modulo con la data più vecchia** (`rsppBaseScad`), con riferimento per modulo = `aggiornamento || data iniziale`. Sparisce il campo aggiornamento unico di figura per il DL-RSPP (le altre figure lo mantengono). Footer → v2.10.
- **v2.11 (costruita) — Import libretti formativi (step 1).** Nuova scheda **📥 Importa**: carica un export Excel «Elenco Visite/Formazioni» (una riga = persona + evento). **Chiave comune dei corsi** (`IMPORTCORE.mapCorso`): ogni «Tipo» a testo libero è normalizzato e risolto in chiave stabile `figura:sottotipo:natura` (+ `corsoId` di catalogo quando esiste) via alias/euristiche + **mapping appreso** persistito (`org81v2:corsoMap`) per i non riconosciuti. Modello **`persona.formazioni[]`** (libretto): voci `{key,corsoId,figura,sottotipo,natura,data,tipoRaw,fonte}`. Import **a staging con report a 4 sezioni** (eventi riconosciuti · corsi da mappare · anomalie da confermare · proposte di incarico da confermare): **non scrive nulla** finché non confermi. Dedup persone per **CF** (+ cross-check nome/nascita); società del file ≠ azienda → **avviso unico** con conferma; eventi già in libretto marcati duplicati. Le proposte confermate creano l'incarico e ne impostano l'evidenza (iniziale/aggiornamento, data più recente). Validato sul file reale: 27 eventi / 11 persone, 14/14 «Tipo» mappati. **Step 2 (prossimo):** vista libretto stampabile + export .docx Overall. Footer → v2.11.
- **v2.12 (costruita) — Libretto formativo per persona (stampa/PDF + Word, step 2).** Dalla scheda **👥 Persone**, ogni persona ha due pulsanti: **📄 Libretto (stampa / PDF)** e **⬇️ Word (.doc)**. `librettoData(pid)` assembla: anagrafica persona (nome, CF, **nato a/il + sesso derivati dal CF**), **Posizioni e formazione obbligatoria** dagli incarichi (figura+variante, nomina/designazione con eventuale data, formazione iniziale, ultimo aggiornamento, scadenza e **stato** riusando `statoForm`/`STATI`; per il **DL-RSPP** iniziale = modulo con data più recente, aggiornamento = aggiornamento di modulo più recente), e il **Libretto eventi** da `persona.formazioni[]` (corso = `tipoRaw`, data, fonte; ordinati per data). Output **brandizzato Overall** (logo orizzontale embedded base64, palette #1F3864/#2E5496, testata #404040, righe alternate, semaforo stato verde/giallo/rosso) in formato **A4** con footer dati Overall (OVERALL GROUP S.R.L., P.IVA 04534450236). La **stampa** apre una finestra dedicata e lancia `window.print()` (Salva come PDF dal browser); l’**export Word** è un **HTML-Word `.doc`** (`application/msword`) apribile e ri-salvabile in `.docx` da Word — scelta leggera per non appesantire il file standalone con una libreria docx. Note: il libretto è un riepilogo gestionale, non sostituisce gli attestati originali del fascicolo. Footer → v2.12. 
- **v2.13 (costruita) — Split Lavoratore (D1) + rifondazione esoneri su Allegato III (mappa §6).** La formazione iniziale del **Lavoratore** passa a **due moduli** come il DL-RSPP: *Generale 4h* (una tantum, **credito permanente**, fonte di propedeuticità) + *Specifica B/M/A* (con **aggiornamento 6h/5 anni**; la **scadenza della figura nasce dalla Specifica**). **Esonero/credito per-riga** (non più a livello figura), evidenza per modulo, todo cruscotto distinti per Generale/Specifica. **Migrazione automatica**: il vecchio iniziale lavoratore confluisce nella *Specifica*; un esonero figura-level pregresso (Lavoratori 2011) viene applicato a **entrambe** le righe e l'esonero di figura ritirato. `prereqOk` reso **module-aware** (Generale svolta/creditata soddisfa la propedeuticità del preposto). Nuove funzioni `ensureLavModuli/lavInizialeAssolta/syncLavIniziale/lavModEso/moduliLavHTML` + dispatcher `syncModuli`. Validazione: `node --check` sul blocco app + harness **23/23** (stato, scadenza 2028, scaduto, verifica, migrazione, propedeuticità, idempotenza). Il **§6** è riscritto: esoneri in **due famiglie** (pregresso normativo / credito tra ruoli) con **matrice Allegato III** (`CREDITI_RUOLI`), clausola stessa-azienda (T*) e parziale=nota — **D1-D4 chiuse**. Footer → v2.13. **Prossimo (v2.14):** motore di **derivazione automatica tra ruoli** (D4) + spunta "qualifica conseguita in altra azienda" (D3) + nota PARZIALE (D2), da costruire e testare come unità unica. 
- **v2.14 (costruita) — Motore di derivazione tra ruoli (D2+D3+D4).** Codificata la **matrice Allegato III** (`CREDITI_RUOLI`) con mappe `ROLE_OF`/`COL_OF` sulle figure dell'app (CSP/CSE non presente come figura → omesso; RSPP/ASPP = designazione). **D4 — derivazione automatica guidata**: su Datore/Dirigente/Preposto/RLS e sui due moduli del Lavoratore, se la persona possiede un altro ruolo che dà credito **T/T\*** verso quell'iniziale, l'app mostra un **banner di proposta** con pulsante *Applica* (mai automatico); l'esonero applicato registra `derivT`/`derivFrom`. **D3 — clausola stessa-azienda**: sugli esoneri **T\*** compare la spunta *"qualifica conseguita in altra azienda"* (default off); se attivata il credito **decade** — nuovo `esoOk()` (usato in `lavInizialeAssolta`/`rsppInizialeAssolta`/`statoForm`/`prereqOk`) riporta l'iniziale a *mancante* con avviso. **D2 — PARZIALE**: solo i crediti **TOTALE** (T/T\*) generano proposta; i percorsi professionali parziali restano nota (nessuna proposta). Nuove funzioni `esoOk/ruoloAcquisito/fontiCredito/applyDeriv/derivBannerHTML/derivModHTML` + handler `applyDeriv`/`esoAltraAz`/`modEsoAltraAz`. Validazione: `node --check` + harness **22/22** (matrice, decadimento T\*, proposte, apply, gating per ruolo non acquisito, isolamento per-persona) e **regressione 23/23** sui test v2.13. Footer → v2.14. 
- **v2.15 (costruita) — Gerarchia dei crediti: sulle figure di matrice si usa solo Famiglia 2.** Sulle figure coperte da Allegato III (Datore, Dirigente, Preposto, RLS) e sui due moduli del Lavoratore, la casella *"Possibili fonti di esonero"* ora elenca i **crediti di ruolo** (`creditiRuoloInfo(col)` → righe "Credito totale se la persona è anche …" per i **T** e "Credito totale solo stessa azienda (T*) se anche …" per i **T\***), **al posto** del testo di catalogo Famiglia 1. `lavModEso` re-instradata su `lavGen`/`lavSpec`; helper figura via nuovo `figEsoSources(inc)` (matrice → Famiglia 2; **fuori matrice → Famiglia 1 invariata**, così le **attrezzature** mantengono il credito Acc. 22/02/2012 e i **moduli DL-RSPP** il pregresso DDL-RSPP). Asimmetria preservata: l'RLS credita la Generale ma **non** la Specifica. Il **banner di proposta** resta il modo per **applicare** i crediti con un click. Validazione: `node --check` + harness **15/15** (liste per colonna, asimmetria RLS, attrezzature Famiglia 1 intatte) e **regressione 22/22** sul motore v2.14. Footer → v2.15.

- **v1 (fatto):** app figura-level con un solo periodo di aggiornamento per ruolo. Due forme: artifact React (`window.storage`) e file HTML standalone offline (`localStorage`). Catalogo periodi allineato a `checkupformazione81/references/percorsi-formativi.md`.
- **v2 (costruita):** rifondazione su **catalogo corsi** (questo documento). File `organigramma-sicurezza-81-v2.html` (standalone, localStorage). Include Fascicolo documentale e le logiche di §11.
- **v2.1 (costruita):** **motore agganciato al catalogo** (decisione "opzione 2"). Periodicità, modalità, scadenze transitorie ed esoneri sono **letti dalle righe del catalogo** tramite una mappatura tipo→corso (per id). Ordine di risoluzione di ogni periodo: *override manuale (tabella Regole) → valore da catalogo → default curato*. I tipi fuori catalogo (RSPP/ASPP professionista, Medico Competente) restano "curati". **Aggiornamento del catalogo dall'app:** pulsante "Aggiorna catalogo da Excel" (SheetJS incorporato, offline) che ricarica un `ASR26-TabellaCorsi.xlsx` aggiornato e ne fa seguire il motore; "Ripristina predefinito" per tornare al catalogo incorporato. Vincolo: gli id corso derivano dal nome — se nel file cambia il *nome* di un corso, quel tipo si scollega e ricade sul default (l'app lo segnala). Sostituisce la scheda REGOLE figura-level con il catalogo, e in Organigramma si assegnano *ruoli + abilitazioni* da cui derivano i corsi.
- **v2.2 (costruita):** **app allineata al catalogo uniformato** (24/06/2026). `buildCatalogo` leggeva ancora il layout vecchio (ASR S/N in col. A): corretti gli indici al layout a 16 colonne reale (Figura A, Obbligatorio B, ASR C, Corso D …), così l'import "Aggiorna catalogo da Excel" è di nuovo idempotente. Il record di catalogo ora porta anche `figura`/`obbligatorio`. `CATALOGO_DEFAULT` rigenerato dal file uniformato → **73 corsi** (prima 74: il corso *Segnaletica stradale per cantieri*, assente dal file, è stato **escluso su decisione del committente** — vedi v2.5). La scheda **Catalogo & Regole** mostra ora anche le colonne **Figura** e **Transitoria** (col. O). Il testo della colonna **esoneri/formazione pregressa** (es. crediti RSPP DL coi vincoli ATECO per modulo) è preservato con gli **a-capo** e reso a **punti elenco** nell'helper di esonero (prima veniva appiattito su una riga sola). Nota d'uso: chi ha già usato l'app vede il catalogo salvato in `localStorage` finché non preme "Ripristina predefinito" o "Aggiorna catalogo da Excel".
- **v2.3 (costruita):** tre interventi su Organigramma/Cruscotto. (a) **Date in testo `gg/mm/aaaa`** (niente più rotella anno): tutti i campi data — nomina, formazione iniziale, ultimo aggiornamento, fascicolo — sono input testo con parsing tollerante (`parseHuman`) e bordo rosso se non valido; internamente restano ISO. (b) **Formazione mancante = Non Conformità**: continua a confluire nel cruscotto (KPI "Formazioni mancanti" + lista "Da gestire subito"), ora con etichetta **NC** esplicita. (c) **Evidenza attestato**: per ogni formazione presente (stato ≠ mancante) la card mostra la spunta *"Attestato agli atti"* + rif.; se manca, badge *"COSA DA FARE: acquisire/allegare attestato"* e la voce confluisce nel cruscotto (KPI "Attestati da acquisire", incluso fra le cose da gestire). L'evidenza è ancorata a persona+corso (coerente con §3).
- **v2.4 (costruita):** rischio ATECO, integrativo DL-RSPP, attestato come allegato. (a) **Allegato IV ASR 17/04/2025 incorporato** (88 divisioni ATECO 2007, fonte: libreria `formazione-81-utils`): `rischioAteco(codice)` → livello BASSO/MEDIO/ALTO + ore. **ATECO ora obbligatorio** in anagrafica, scelto da una **combobox filtrabile** (`datalist`, 88 divisioni — ricerca per codice o per attività); alla compilazione il livello di rischio è **calcolato in automatico** (override manuale ammesso per il DVR, Interpelli MLPS 11/2013 e 1/2025). (b) **Pill rischio** colorata e cliccabile (dettaglio + fonte) in **header, anagrafica e cruscotto**. (c) **DL-RSPP art. 34**: la formazione iniziale è ora **Comune (8h) + modulo integrativo di settore** scelto via selettore (Mod 1 A 01-02 · Mod 2 A 03 · Mod 3 F · Mod 4 C 19-20), **pre-selezionato da ATECO** (`modIntegrativoAteco`) e modificabile; `resolve` compone "8h Comune + Nh ModX". Il legame è **per divisione ATECO 2007** (01-02→Mod1, 03→Mod2, 41-43→Mod3, 19-20→Mod4) ed è **dinamico**: al cambio ATECO il modulo si ri-propone (salvo override manuale, flag `varManuale`); sulla card un badge mostra «ATECO NN → Mod X». ⚠️ Solo questi 4 settori hanno un modulo dedicato nel catalogo; per gli altri ATECO la proposta è «solo Comune».
- **v2.5 (costruita):** decisioni chiuse dal committente. (a) **Modello integrativo DL-RSPP confermato completo**: i 4 moduli (A 01-02, A 03, F, C 19-20) sono l'insieme voluto; per ogni altro ATECO vale il **solo modulo Comune** (8h) — comportamento corretto, non una lacuna. Il legame ATECO→modulo è ora anche **dinamico** (badge «ATECO NN → Mod X» sulla card; ri-proposta al cambio ATECO salvo override). (b) **Segnaletica stradale rimossa**: la figura *Segnaletica stradale (cantieri)* è stata tolta da `TIPI` e `MAP` (non utilizzata e assente dal catalogo). Risultato: la **mappatura tipo→corso non ha più voci scollegate** (0 orfani). Aggiunta una guardia: incarichi di tipo sconosciuto non mandano in errore la card.
- **v2.6 (costruita):** (a) **pill rischio nell'intestazione di ogni gruppo-figura** dell'organigramma (banda `tipohead`), utile a valutare gli esoneri legati al livello di rischio pregresso. (b) **DL-RSPP: rimosso il menù a tendina** del modulo integrativo — non è una scelta dell'utente ma è **determinato dall'ATECO**: la card propone direttamente i due corsi (badge «ATECO NN → Mod X» + "Iniziale: 8h Comune + Nh ModX"), aggiornati automaticamente al cambio ATECO. Le altre figure con scelta legittima (lavoratore B/M/A, antincendio L1/L2/L3, primo soccorso A/B-C, BLSD, RLS) mantengono il selettore.
- **v2.7 (costruita):** DL-RSPP — la "Formazione iniziale" diventa un blocco **"Formazione iniziale — moduli"** con due righe distinte (Modulo Comune 8h + Modulo integrativo di settore), ciascuna con **data propria** (gg/mm/aaaa) e **attestato proprio** da allegare. `inc.moduli={comune,integr}`; `syncRsppIniziale()` calcola `dataIniziale` = data più recente quando entrambi i moduli richiesti sono datati (solo Comune se l'ATECO non ha integrativo), così l'aggiornamento parte dal completamento reale. Il cruscotto elenca i due attestati separatamente ("Attestato Comune" / "Attestato modulo integrativo"). Migrazione automatica: per incarichi esistenti la vecchia data/attestato iniziale confluisce nel Modulo Comune.

- **v2.8 (costruita):** (a) **font del bottone "Scegli file" normalizzato** (`::file-selector-button` eredita font/dimensione dell'app). (b) **Esonero consolidato a livello di figura** — rimosso l'esonero per-modulo: un solo checkbox "Esonero / credito sulla formazione iniziale". Quando attivo, il box della formazione iniziale (i moduli per il DL-RSPP, il campo singolo per le altre figure) **scompare** e resta **solo il box Esonero** con la sua gestione completa: fonti dal catalogo, Nota e **evidenza dedicata** (allegato dell'attestato che dà il credito), oltre all'**aggiornamento** che continua a essere gestito. Le evidenze sono quindi separate per natura: formazione iniziale *prevista* → attestati dei corsi; *non prevista* (esonero) → attestato dell'esonero. Nel cruscotto, con esonero attivo, compare "Evidenza esonero/credito (da allegare)" al posto degli attestati dei moduli; il prerequisito non è richiesto.

- **v2.9 (costruita) — correzione di v2.8:** per le figure con **più corsi iniziali** (DL-RSPP) l'esonero/credito **non** è a livello di figura ma **per singolo modulo**: il credito può coprire il Comune e non l'integrativo (o viceversa). Ripristinato quindi l'esonero **per modulo**: ogni modulo (Comune, integrativo) ha il proprio checkbox "Esonero / credito su questo modulo"; se attivo, **sparisce la riga Data+attestato di quel modulo** e compare il suo box esonero con fonti dal catalogo, Nota e **evidenza dedicata** (attestato che dà il credito). Le figure a corso unico (lavoratore, antincendio, primo soccorso, ecc.) mantengono l'esonero **a livello di figura** (anch'esso con evidenza). Cruscotto: per ogni modulo esonerato chiede "Evidenza esonero [Comune/integrativo]", per ogni modulo non esonerato e datato chiede l'attestato.
- **v2.8 (costruita):** DL-RSPP — **esoneri/crediti gestiti per singolo modulo**, non piu in bundle. Ogni modulo (Comune, integrativo) ha la propria spunta "Esonero / credito su questo modulo", con la **propria nota** e le **fonti di esonero specifiche di quel corso** (lette dal catalogo: `MAP.datoreRspp.ini` per il Comune, `MAP.datoreRspp.integr[modX]` per il settore). Un modulo è **assolto** se datato+attestato **oppure** coperto da esonero. `rsppInizialeAssolta()` richiede che ogni modulo necessario sia assolto; `syncRsppIniziale()` pone `dataIniziale` = data del/i modulo/i effettivamente svolto/i (la piu recente), o "" se tutto a credito. `statoForm()` per datoreRspp usa l’assolvimento per-modulo: se assolta solo a credito e senza data aggiornamento → "Verifica agg."; se svolta, lo scadenzario parte dalla data reale. Nel cruscotto l’attestato è richiesto **solo per i moduli svolti** (i moduli a esonero non lo richiedono). Tolto l’esonero unico dal fondo della card per il DL-RSPP; le altre figure lo mantengono. Migrazione: vecchio esonero bundle → Modulo Comune. (d) **Attestato = allegato file**: sulla card si carica il PDF/immagine (<2 MB); finché non è allegato resta **COSA DA FARE** e confluisce nel cruscotto ("Attestati da acquisire").
- **Branding:** palette Overall (blu `#1F3864`/`#2E5496`, semaforo `#C6E0B4`/`#FFE699`/`#F8CBAD`), font Calibri/Carlito.
- **Persistenza:** localStorage (HTML standalone) come archivio di lavoro; **condivisione multi-postazione via File System Access** su file `.json` per azienda in cartella sincronizzata/di rete (v2.10, Opzione A). Backend cloud rimandato (opzione C).

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
