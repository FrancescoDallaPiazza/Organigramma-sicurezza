# Percorsi formativi — strumento di sintesi (D.Lgs. 81/08 + Accordi Stato-Regioni)

Strumento unico per costruire la sintesi formativa di un'azienda **sia nuova sia attiva
da anni**. Contiene tre livelli:
1. **Quadro a regime** (ASR 17/04/2025) — cosa serve oggi, per figura.
2. **Quadro storico** (accordi 2011/2012/2016 + DM emergenze) — cosa valeva prima,
   per giudicare attestati pregressi e calcolare i crediti.
3. **Transizione e crediti** — come si salda il vecchio col nuovo.

Tutte le durate sono verificate sui testi in `references/` e `references/accordi/`:
- ASR 2025 → `accordi/ASR_59_2025_nuovo_accordo_RAW.txt` (Parte II corsi, Parte III aggiornamenti)
- Lavoratori/Preposti/Dirigenti 2011 → `accordi/ASR_221_2011_lavoratori_RAW.txt`
- DL-RSPP 2011 → `accordi/ASR_223_2011_DL_RSPP_RAW.txt`
- Attrezzature 2012 → `accordi/ASR_53_2012_attrezzature_RAW.txt`
- RSPP/ASPP 2016 → `accordi/ASR_128_2016_RSPP_ASPP_RAW.pdf` (scansione; struttura confermata dall'ASR 2025)
- Antincendio → `DM_02_09_2021_ANTINCENDIO.txt`; Primo soccorso → `DM_388_2003_PRIMO_SOCCORSO.txt`

---

## GUIDA D'USO — ditta NUOVA vs ditta ATTIVA DA ANNI

**Ditta NUOVA (nessun attestato pregresso):** parti dal livello di rischio (vedi
`ateco-rischio.md`) e applica direttamente il **Quadro a regime** qui sotto. Costruisci il
piano figura per figura con le durate "oggi". Non serve il quadro storico.

**Ditta ATTIVA DA ANNI (con attestati pregressi):** per ogni attestato segui 4 passi:
1. **Data di emissione** → con `accordi-storici.md` individua quale accordo si applicava allora.
2. **Conformità storica** → confronta l'attestato col **Quadro storico** (durata/moduli corretti per
   l'epoca?) e con gli elementi minimi del relativo `.txt` attestazioni.
3. **Validità oggi** → calcola la scadenza (data + cadenza di aggiornamento) e verifica se il
   percorso è ancora valido o va aggiornato/rifatto secondo il **Quadro a regime**.
4. **Credito** → applica le regole di **Transizione e crediti**: un corso pregresso conforme
   spesso vale come credito verso il nuovo percorso (non si rifà, basta l'aggiornamento).

> Regola d'oro per la ditta storica: **un attestato non è "valido o scaduto" in assoluto**, ma
> "valido/scaduto rispetto alla norma di allora E adeguato/da adeguare rispetto a quella di oggi".

---

## 1. QUADRO A REGIME — ORGANIGRAMMA SICUREZZA (ASR 17/04/2025)

| Figura | Formazione | Tipo e durata | Aggiornamento | Esoneri / crediti | Note |
|---|---|---|---|---|---|
| **Datore di lavoro** (con RSPP esterno/interno) | Sì (novità 2025) | Corso DL **16h**; +6h modulo cantieri se ATECO edile | **6h / 5 anni** | Esonerato chi ha il corso Dirigente; credito totale a chi ha già corso DL 16h o DL-RSPP pregresso (FAQ n.7) | Prima attivazione entro **24/05/2027**. Presenza/VCS/e-learning |
| **Datore di lavoro-RSPP** (art. 34) | Sì | Corso DL 16h + Modulo Comune 8h = **24h** + integrativo settore (Agri 16h/Pesca 12h/Costruz. 16h/Chimico 16h) | **8h / 5 anni** dal modulo comune | Moduli pregressi a credito; integrativi solo se stesso ATECO sull'attestato (FAQ n.63) | Modulo comune solo presenza/VCS |
| **Dirigente** | Sì | **12h** (ex 16h) | **6h / 5 anni** | Esonera dal corso DL | Presenza/VCS/e-learning |
| **Preposto** | Sì | **12h** (ex 8h) | **6h / 2 anni** | Prerequisito: formazione lavoratore | Solo presenza/VCS. Transitorio: ante 24/05/2023 → entro 24/05/2026 |
| **Lavoratori** | Sì | Generale 4h + Specifica 4/8/12h (Basso/Medio/Alto) | **6h / 5 anni** | Generale = credito permanente | Prima dell'avvio attività |
| **RLS** | Sì | **32h** (12h rischi specifici) | **annuale** 4h (15-50) / 8h (>50) | — | <15 lav. rimesso al CCNL |
| **RSPP** | Sì | Mod. A 28h + Mod. B 48h (+SP) + Mod. C 24h | **40h / 5 anni** dal Mod. B | Esonero Mod. A/B per titoli (art. 32 c.5) | Struttura confermata dal 2016 |
| **ASPP** | Sì | Mod. A 28h + Mod. B 48h (+SP) | **20h / 5 anni** dal Mod. B | Come RSPP per i titoli | No Modulo C |
| **Addetto antincendio** | Sì | Liv.1 4h / Liv.2 8h / Liv.3 16h+esame VVF | **5 anni**: 2h/5h/8h | — | DM 02/09/2021 |
| **Addetto primo soccorso** | Sì | Gr. A 16h / Gr. B-C 12h | **3 anni** (prassi 6h A / 4h B-C) | — | DM 388/2003. Decreto fissa cadenza, non ore |
| **Addetto attrezzature** (art. 73 c.5) | Sì | Corso teorico-pratico per tipo (vedi §4) | **5 anni, min 4h** pratica | Abilitazioni pregresse valide entro 5 anni | DL non può più formarli direttamente |
| **Ambienti confinati** (DPR 177/2011) | Sì | Formazione-addestramento specifica | **5 anni, min 4h** pratica | — | Lavoratori/datori/autonomi |

> Fuori tabella: **CSE/CSP** (cantieri, 120h + 40h/5 anni) escluso su indicazione; **HACCP**
> non è ASR ma Reg. CE 852/2004 + legge regionale (vedi i rispettivi `.txt`).

---

## 2. QUADRO STORICO — durate/cadenze PRIMA dell'ASR 2025

Da usare per giudicare attestati di ditte attive da anni. Per ogni figura: cosa valeva, quale
norma, cosa è cambiato oggi.

| Figura | Regime pre-2025 (durata) | Aggiornamento pre-2025 | Norma | Cosa cambia oggi |
|---|---|---|---|---|
| **Datore di lavoro (solo)** | **Nessun obbligo** se non RSPP | — | — | Introdotto obbligo: 16h + 6h/5 anni |
| **Datore di lavoro-RSPP** | **16h (basso) / 32h (medio) / 48h (alto)** | quinquennale | ASR 223/2011 | Diventa 16h base + 8h modulo comune (24h) + integrativi; agg. 8h/5 anni |
| **Dirigente** | **16h** | 6h / 5 anni | ASR 221/2011 | Scende a 12h |
| **Preposto** | **8h** | 6h / 5 anni | ASR 221/2011 | Sale a 12h; aggiornamento **biennale** |
| **Lavoratori** | Generale 4h + Specifica 4/8/12h | 6h / 5 anni | ASR 221/2011 | **Invariato** (cambia: formazione prima dell'avvio, non più 60 gg) |
| **RLS** | 32h | annuale 4h/8h | art. 37 D.Lgs. 81/08 | Invariato |
| **RSPP / ASPP** | A 28h + B 48h + C 24h; agg. 40h/20h | quinquennale | ASR 128/2016 | Sostanzialmente invariato |
| **Antincendio** | 4h/8h/16h (basso/medio/elevato) | **nessuna periodicità obbligatoria** | DM 10/03/1998 (fino 03/10/2022) | DM 02/09/2021: stesse 4/8/16h + agg. quinquennale 2/5/8h |
| **Primo soccorso** | Gr. A 16h / B-C 12h | triennale (pratica) | DM 388/2003 | Invariato |
| **Attrezzature** | PLE 8/10/12h; gru autocarro 12h; gru torre 12/14/16h; carrelli 12/16/20h | quinquennale 4h | ASR 53/2012 | Durate invariate; DL non può più formare direttamente |

---

## 3. TRANSIZIONE E CREDITI (vecchio → nuovo)

**Date chiave ASR 2025:**
- 24/05/2025 entrata in vigore; 24/05/2026 fine periodo transitorio (corsi vecchio formato);
  24/05/2027 termine prima formazione obbligatoria dei datori di lavoro già in carica.

**Regole di credito (per la ditta storica):**
- **Formazione generale lavoratori (4h)** e **modulo preposto pregresso**: crediti permanenti
  (già previsto da ASR 221/2011).
- **Corso Dirigente** → esonera dal corso DL.
- **Corso DL 16h o DL-RSPP pregresso conforme** → credito totale verso il corso DL; resta solo
  l'aggiornamento (FAQ Commissione Salute n.7). Per il DL-RSPP i moduli integrativi sono a credito
  solo se l'attestato riporta lo **stesso settore ATECO** (FAQ n.63).
- **Preposto 8h pre-2025**: il corso resta valido; va però portato a regime con l'aggiornamento
  **biennale** di 6h (transitorio: se l'ultimo corso/aggiornamento è ante 24/05/2023 → aggiornarsi
  entro 24/05/2026).
- **RSPP/ASPP**: crediti A/B/C mantenuti; il credito abilitante non decade entro 10 anni anche con
  aggiornamento tardivo (ma la funzione non è esercitabile finché non lo si completa).
- **Antincendio pre-04/10/2022 (DM 1998)**: l'attestato resta valido ma scatta l'obbligo di
  aggiornamento quinquennale del DM 2021 (primo aggiornamento entro 12 mesi se erano già
  trascorsi >5 anni all'entrata in vigore).
- **Decadenza generale**: oltre 10 anni senza aggiornamento → credito decaduto, percorso da rifare.

---

## 4. Dettaglio attrezzature (art. 73 c.5) — durate corso (teoria+pratica)

| Attrezzatura | Durata corso |
|---|---|
| PLE (piattaforme elevabili) | 8h (con stab.) / 10h (senza) / 12h (entrambe) |
| Gru per autocarro | 12h |
| Gru a torre | 12h / 14h / 16h (rotazione basso/alto/entrambe) |
| Carrelli elevatori semoventi | 12h (industriali) / 16h (telescopici) / 20h (telesc. rotativi) |
| Trattori agricoli/forestali, macchine movimento terra, ecc. | vedi `accordi/ASR_53_2012_attrezzature_RAW.txt` |

Aggiornamento: **quinquennale, min 4h** (di cui ≥3h pratica) per tutte.

---

## 5. Cronoprogramma sintetico aggiornamenti (a regime)

| Figura | Cadenza | Ore minime |
|---|---|---|
| Lavoratori | 5 anni | 6 |
| Preposto | 2 anni | 6 |
| Dirigente | 5 anni | 6 |
| Datore di lavoro | 5 anni | 6 |
| Datore di lavoro-RSPP | 5 anni | 8 |
| RSPP | 5 anni | 40 |
| ASPP | 5 anni | 20 |
| RLS | 1 anno | 4 (15-50) / 8 (>50) |
| Antincendio | 5 anni | 2/5/8 (liv. 1/2/3) |
| Primo soccorso | 3 anni | 6 (A) / 4 (B-C) — prassi |
| Attrezzature | 5 anni | 4 (pratica) |
| Ambienti confinati | 5 anni | 4 (pratica) |

---

## 6. Riferimenti per l'analisi degli attestati
- Quale norma si applica a un attestato emesso in data X → `accordi-storici.md`
- Elementi minimi attestato: fino al regime precedente → `ASR_221_2011_punto7_ATTESTATI.txt`;
  dal 24/05/2025 → `ASR_59_2025_punto6_ATTESTAZIONI.txt`
- Requisiti del docente/formatore → `DI_06_03_2013_REQUISITI_FORMATORI.txt`
