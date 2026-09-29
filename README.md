# Variabilità di Peso nel Confezionamento Caffè — Integrated Lean Six Sigma Case
- **Area**: Quality/ Continuous Improvement
- **Metodologia**: DMAIC (Define-Measure-Analyze-Improvement-Control)
- **Strumenti**: Python (pandas, scipy, numpy), analisi statistica (ANOVA, t-Welch, test di Levene, Cp/Cpk)
- **Dataset**: Simulato, generato per riprodurre in modo plausibile un caso di variabilità di processo in ambito confezionamento alimentare. Dichiarato esplicitamente come tale.
---
## 1. Business Problem
Un'azienda di torrefazione e confezionamento caffè riceve, nell'arco di due mesi, **tre reclami da clienti B2B** (bar, ristoranti) per confezioni di caffè con peso inferiore a quanto dichiarato in etichetta.
- **Target di peso**: 250 g
- **Specifiche cliente (CTQ)**: USL 255 g / LSL 245 g
- **Soglia minima di capability richiesta dal cliente**: Cpk ≥ 1,33
## 2. Obiettivo
Identificare dove e perchè si concentra la non conformità di peso, determinanrne la root cause e validare una contromisura e quantificarne l'impatto economico.

### 2.1 Goal Formale (Derivato dalla specifica cliente)
Ridurre la deviazione standard del peso da 1,83 g iniziali (baseline iniziale, dati grezzi) a ≤1,25 g portando il Cpk da 0,91 ad **almeno 1,33**.

## 3. Process Context
Il problema coinvolge le confezioni di caffè finite. Trattandosi di una problematica di peso, la parte coinvolta del processo è quella del confezionamento, divisibile nelle seguenti fasi:
```
Apertura sacchetto → Dosaggio caffè → Riempimento → Chiusura sacchetto → Pesatura
```
La linea opera su tra turni (Mattina, Pomeriggio, Notte). Il turno di notte lavora in regime più continuativo, con minore manutenzione. Il **Dosatore** (Fase di Dosaggio) è il punto del processo maggiormente collegato in modo diretto alla variabilità del peso finale — sensibile ad usura, taratura, variazioni di densità/umidità del prodotto.

### 3.1 SIPOC

| Categoria | Elementi |
|---|---|
| Supplier | Fornitore caffè tostato, Fornitore confezioni |
| Input | Caffè tostato macinato, Confezioni vuote |
| Process | Tostatura → Dosaggio → Riempimento → Chiusura → Pesatura |
| Output | Confezione da 250 g |
| Customer | Clienti B2B (bar, ristoranti) |
---
## 4. Dataset
- `pesature_clean.csv` — 420 osservazioni (14 giorni × 3 turni × 10 campioni), campionamento casuale distribuito durante l'intero turno (per evitare bias da Startup Rejects)
- `pilot_clean.csv` — 278 osservazioni, turno Notte, 2 macchine in parallelo (2 settimane): `Macchina_1_Contromisura` (con la contromisura) vs `Macchina_2_Riferimento` (gruppo di controllo, nessuna modifica)

Vedi `data_dictionary.md` per la struttura completa.

## 5. Data Quality
Il dataset grezzo conteneva, per costruzione, le seguenti imperfezioni, trattate esplicitamente prima dell'analisi:

| Problema | Trattamento |
|---|---|
| Missing values (~1,7%) | Imputazione con **mediana stratificata per turno/macchina** (non globale — per non attenuare la differenza tra gruppi, che è l'oggetto stesso dell'analisi) |
| Duplicati esatti | Identificati su tutte le colonne (incluso il peso) e rimossi |
| Valori impossibili (0 g, 2508 g) | Distinti dagli outlier statistici plausibili tramite un range di plausibilità fisica (200–300 g); il valore 2508 trattato come probabile errore di virgola (÷10) |
| Outlier statistici (IQR) entro range fisico plausibile | **Non trattati come errori** — riconosciuti come variabilità naturale, specialmente nel turno Notte, per non distruggere il segnale che l'analisi doveva rilevare |
| Formati data non uniformi | Riconosciuti e convertiti con parsing a due formati (`%Y-%m-%d` e `%d/%m/%Y`) |
## 6. KPI

Vedi `kpi_dictionary.md` per definizioni complete. Principali: Cpk, deviazione standard di processo, % confezioni sotto LSL.

---
## 7. Analysis
### 7.1 Segmentazione per operatore
Entrambi gli operatori del turno Notte (OP-01: 6,78% fuori specifica; OP-06: 8,64%, deviazione standard più alta) mostrano tassi elevati di non conformità — **il problema non è imputabile a un singolo operatore**, ma è caratteristico del turno stesso, coerente con un'ipotesi legata alla macchina/manutenzione piuttosto che al fattore umano.

### 7.2 Segmentazione per turno
| Turno | n | Media (g) | Dev. Std (g) | Cpk | % fuori specifica |
|---|---:|---:|---:|---:|---:|
| Mattina | 140 | 250,28 | 1,56 | 1,009 | 0,71% |
| **Notte** | 140 | 248,94 | 2,49 | **0,526** | 7,86% |
| Pomeriggio | 140 | 250,04 | 1,24 | 1,334 | 0,00% |

**Chiave di lettura**: Dalle primestatistiche estrapolate dal set di dati emerge una distribuzione delle non conformità di peso verso il turno di notte (% fuori specifica). Per quanto riguarda invece la capability di processo (Cpk), si può notare come il turno del pomeriggio sia in target con quanto definito dal cliente. Il turno Mattina invece mostra una capability inferiore al target ma non drammaticamente. Il turno di notte raggiunge una capability di 0,52, meno della metà del target. 

#### 7.2.1 Verifica Statistica ANOVA (differenza tra turni)
Per validare le prime osservazioni si procede con un analisi statistica per valutare se è presente variabilità tra i gruppi presenti (Mattina. Pomeriggio, Notte). Come test statistico di considera la presenza di più di due gruppi che vengono analizzati su una singoa variabile, per questo si effettua un test ANOVA ad una via sulla colonna dati `weight_grams`, raggruppata per `shift`, utilizzando il dataset pulito (`pesature_clean-csv`, n = 420):

```
F = 21,03
p = 1,98 × 10⁻⁹
```
Il p-value è ampiamente sotto $\alpha$ = 0,05, questo evidenzia come è molto forte la variabilità e quindi almeno un turno abbia una media di peso realmente diversa dagli altri — indicando coerenza con quanto osservato nei primi dati Cpk e % fuori specifica (per entrambi il turno di Notte è outlier). L'ANOVA da sola non indica quale turno differisca: per questo utilizziamo un confronto descrittivo (Tabella) e dalla segmentazione per operatore, non da un test post-hoc formale (limite dichiarato, vedi sezione Limitations).
## 8. Root Cause Analysis
### 8.1 Ishikawa (6M)
applicato all'ipotesi "perché il turno Notte differisce":

| Categoria | Ipotesi |
|---|---|
| Manpower | Operatori più affaticati (esclusa: entrambi gli operatori Notte sono ugualmente colpiti) |
| **Machine** | **Usura/deriva di taratura del dosatore dopo funzionamento prolungato (24h continuo)** |
| Method | Assenza di controllo/serraggio durante il funzionamento continuo |
| Material | Differenza di densità del caffè caldo vs raffreddato |
| Measurement | Perdita di precisione del dosatore nel tempo |
| Environment | Variazioni di umidità ambientale tra turni |

### 8.2 5 Why

```
1. Perché il pacchetto pesa meno?          → Il dosatore sbaglia le quantità
2. Perché le sbaglia?                       → Perde di taratura
3. Perché perde di taratura?                → Vibrazioni della macchina alterano il parametro
4. Perché la macchina vibra?                → Una vite è allentata
5. Perché la vite è allentata?               → Non si controlla il serraggio (macchina in funzione 24h)
6. Perché non si controlla?                 → Non è previsto un piano di manutenzione periodica
```
### 8.3 Root Cause
L'assenza di un piano di manutenzione periodica per il controllo del serraggio delle viti critiche del dosatore, combinata con il funzionamento continuo a ciclo 24h che impedisce controlli di routine.

```
Symptom:  aumento reclami cliente per peso sotto specifica
Cause:    errore di dosaggio nel turno Notte, maggiore variabilità
Root Cause: assenza di piano di manutenzione periodica, impossibilità di controllo per funzionamento 24h
```
## 9. Improve
- **Contromisura:** fermata programmata di 15 minuti a inizio turno Notte per controllo e serraggio delle viti critiche del dosatore, con chiave dinamometrica, checklist documentata.
- **Disegno del pilot test:** 2 settimane, turno Notte, due macchine in parallelo — `Macchina_1_Contromisura` vs `Macchina_2_Riferimento` (gruppo di controllo, per isolare l'effetto della contromisura da fattori esterni: lotto caffè, temperatura ambientale, ecc.).

In seguito ad un primo controllo statistico per visualizzare i dati dei due gruppi si procede all'esecuzione di test statistici per determinare la variabilità tra i due gruppi e per determinare se tra i due gruppi è presente una differenza significativa

### 9.1 Risultati dei test statistici
#### 9.1.1 Test di Levene (Omogeneità delle varianze)
Questo test è stato effettuato per determinare una differenza significativa tra le varianze dei gruppi analizzati. 

- $H_0$: le varianze dei due gruppi sono uguali 
- $H_1$: le varianze dei due gruppi hanno una differenza significativa 
- $\alpha$: Intervallo di significatività (0,05)
```
statistica = 49,81
p = 1,36 × 10⁻¹¹  →  p < α → le varianze sono significativamente diverse
```
(la direzione — quale macchina ha varianza minore — è confermata dai descrittivi: Macchina_1 std=1,358 vs Macchina_2 std=2,782, non dal test di Levene in sé, che conferma solo la significatività della differenza)

---

#### 9.1.2 t-test di Welch a una coda
Il test statistico utilizzato determina se ci sono differenze significative tra le medie dei due gruppi, utilizzando il test ad una coda possiamo effettivamente valutare se il peso medio è superiore o inferiore rispetto al riferimento

- $H_0$: il peso medio è minore o uguale al riferimento (μ₁≤μ₂)
- $H_1$: il peso medio è maggiore al al riferimento (μ₁>μ₂)
- $\alpha$: 0,05
```
t = 3,621 (positivo, coerente con H1)
p (una coda) = 0,0001855  →  p < α → si rifiuta H0
```
Supporta l'ipotesi di un peso medio superiore rispetto al riferimento

---

#### 9.1.3 Cpk sul pilot test
Per verificare se le modifiche apportate hanno migliorato la capability della macchina, determiniamo il Cpk delle due macchine per visualizzare le differenze 

| Macchina | n | Media (g) | Dev. Std (g) | Cpk |
|---|---:|---:|---:|---:|
| Macchina_2 (Riferimento) | 140 | 248,72 | 2,782 | 0,446 |
| **Macchina_1 (Contromisura)** | 139 | 249,67 | 1,358 | **1,147** |

**Conclusione Improve:** la contromisura ha prodotto un miglioramento **statisticamente significativo** sia sulla media che sulla varianza (confermato da Welch e Levene), quasi triplicando il Cpk (da 0,446 a 1,147). Il miglioramento **non è ancora sufficiente** a raggiungere il target di processo (Cpk ≥ 1,33). Il margine tra il nuovo LCL (245,60g) e la specifica cliente (LSL 245g) resta sottile (~0,6g), a conferma che il processo, pur migliorato, opera ancora vicino al limite.

**Ipotesi aggiuntive da investigare** (richiedono dati non disponibili nel dataset attuale — priorità del prossimo ciclo di miglioramento):
- Introduzione di controlli/tarature periodici aggiuntivi sul dosatore
- Verifica del contenuto di umidità/temperatura del caffè al momento del riempimento
- Verifica di eventuali perdite di materiale durante riempimento/saldatura del sacchetto

---

## 10. Control Plan
**Nuovi limiti di controllo (Macchina_1, post-contromisura):**
```
UCL = 249,67 + 3×1,358 = 253,75 g
LCL = 249,67 - 3×1,358 = 245,60 g
```
*(limiti di Macchina_2 conservati come baseline storica di confronto, non come standard operativo futuro)*

**Ownership:**
- Esecuzione della checklist di controllo/serraggio: operatore di linea addetto alla dosatrice, giornalmente a inizio turno Notte
- Supervisione: responsabile di turno — verifica che i controlli siano effettivamente eseguiti, con controllo statistico settimanale

**Piano di reazione in caso di recidiva (pesate oltre UCL/LCL):**
```
1. Verificare se la checklist di controllo/serraggio è stata eseguita quel giorno
2. Se NO → causa nota: ripristinare l'esecuzione, monitorare il turno successivo
3. Se SÌ → causa NON nota: Gemba Walk, verifica del lavoro di linea, raccolta di
   nuovi dati, nuova RCA se necessario
```

---

## 11. Business Case
**Assunzioni economiche** (dichiarate esplicitamente come ipotetiche):

| Voce | Valore |
|---|---:|
| Margine di contribuzione / confezione | €2,50 |
| Produzione turno Notte | 3.000 confezioni/notte |
| Giorni operativi/anno | ~240 (dichiarare valore finale scelto) |
| Costo per reclamo gestito | €150 |
| Costo orario intervento | €25/h |
| Durata checklist | 15 min/notte |
| Tasso di conversione difetto→reclamo | ancorato al dato storico: ~90.000 confezioni sotto LSL/anno (stimate) ÷ ~18 reclami/anno storici ≈ **1 reclamo ogni 5.000 confezioni sotto peso** |

**Nota metodologica sul tasso di difettosità post-contromisura:** con 0 confezioni sotto LSL osservate su un campione di 139, il tasso reale non è stimabile come "0%" (campione troppo piccolo). È stata applicata la regola prudenziale del "successo zero" (~3/n), stimando un tasso massimo plausibile di **2,16%** invece di 0%.

**Risultato (analisi di sensitività su 3 scenari del tasso di conversione reclami):**

| Scenario | Tasso (1 reclamo ogni N) | Beneficio netto/anno | ROI | Payback |
|---|---:|---:|---:|---:|
| Conservativo | 8.000 | €140.710 | ~9.381% | ~3,9 giorni |
| Base (ancorato allo storico) | 5.000 | €141.345 | ~9.423% | ~3,9 giorni |
| Ottimistico | 3.000 | €142.474 | ~9.498% | ~3,8 giorni |

**Interpretazione:** il beneficio economico è dominato dal recupero di margine su produzione, non dai reclami evitati — la conclusione economica è quindi robusta rispetto all'incertezza sul tasso di conversione reclami (variazione <1,5% tra scenari). Il ROI elevato è coerente con la natura dell'intervento: costo bassissimo (~€1.500/anno) su un processo ad alto volume con margine reale per unità — il **payback period (~4 giorni)** è la cifra più immediatamente comunicabile a un management non tecnico.

--

## 12. Tools

Python (pandas, numpy, scipy.stats), Excel (esplorazione iniziale), Power BI (misure DAX proposte per dashboard KPI).

## 13. Limitations

- Dataset simulato: i pattern (differenza tra turni, effetto della contromisura) sono plausibili ma generati, non osservati in un impianto reale
- Tasso di conversione difetto→reclamo stimato da un solo evento storico (3 reclami/2 mesi) — campione molto piccolo
- Ipotesi aggiuntive di root cause (materiale, saldatura) non validate per mancanza di dati

## 14. Conclusion

Il progetto dimostra un ciclo DMAIC completo su un problema di variabilità di processo: dalla diagnosi (segmentazione per turno/operatore, Ishikawa, 5 Why) alla validazione statistica di una contromisura (Welch, Levene, Cpk) fino alla quantificazione economica dell'impatto (business case con analisi di sensitività). La contromisura testata produce un miglioramento significativo ma parziale rispetto al target di capability — la raccomandazione operativa è di **estendere la contromisura a tutte le macchine mantenendo aperta un'indagine su cause concorrenti** (materiale, saldatura), non di considerare il problema chiuso.