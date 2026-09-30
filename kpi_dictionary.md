# KPI Dictionary

| KPI | Definizione | Formula | Unità | Significato di business |
|---|---|---|---|---|
| **Cpk** | Process Capability Index | `min[(Media−LSL)/3σ ; (USL−Media)/3σ]` | adimensionale | Capacità del processo di rispettare le specifiche cliente tenendo conto anche del centraggio. Target cliente ≥1,33. Cpk<1 indica processo non capace. |
| **Cp** | Process Potential Index (non calcolato separatamente in questo progetto, coincide con Cpk quando il processo è centrato) | `(USL−LSL)/6σ` | adimensionale | Potenziale del processo, ignora il centraggio |
| **% fuori specifica** | Quota di confezioni sotto LSL o sopra USL | `(sotto+sopra)/totale × 100` | % | Quota di prodotto non conforme, impatto diretto su reclami/resi |
| **Deviazione standard di processo (σ)** | Variabilità del peso di confezionamento | `STDEV(weight_grams)` campionaria (ddof=1) | grammi | Misura diretta della stabilità del dosatore; obiettivo di progetto: ridurla da 1,83g a ≤1,25g |
| **UCL / LCL** | Limiti di controllo statistico | `Media ± 3σ` | grammi | Soglie operative per il monitoraggio in Control; un punto fuori questi limiti segnala una possibile causa speciale |
| **t (Welch)** | Statistica del t-test per il confronto di due medie a varianze diverse | — | — | Verifica se la differenza di peso medio tra due gruppi (es. macchine) è statisticamente significativa |
| **p-value** | Probabilità di osservare una differenza almeno pari a quella osservata, assumendo H0 vera | — | — | Soglia decisionale: p<α (0,05) → si rifiuta H0 |
| **Statistica di Levene** | Test di omogeneità delle varianze tra due o più gruppi | — | — | Verifica se la differenza di variabilità tra gruppi è statisticamente significativa (non robusta la sola osservazione visiva) |
| **ROI** | Return on Investment | `(Beneficio netto / Costo investimento) × 100` | % | Ritorno economico relativo dell'intervento |
| **Payback Period** | Tempo di recupero dell'investimento | `(Costo / Beneficio netto) × 365` | giorni | Tempo necessario perché il beneficio cumulato eguagli il costo dell'intervento — più intuitivo del ROI per un pubblico non tecnico |

## Limiti interpretativi da tenere presenti

- Il Cpk presuppone un processo **stabile e omogeneo**: calcolarlo su un mix di sotto-processi diversi (es. tre turni con medie/varianze diverse aggregati insieme) può mascherare o distorcere il risultato — per questo nel progetto è stato calcolato **per turno**, non solo in aggregato.
- Un p-value basso indica evidenza statistica **contro H0**, non una "dimostrazione" assoluta — va sempre accompagnato dalla direzione dell'effetto (segno della statistica) e da un giudizio di rilevanza pratica, non solo statistica.
- Il test di Levene conferma la **significatività** di una differenza di varianza, non ne indica la **direzione** — quella va letta dai valori descrittivi (deviazione standard) dei singoli gruppi.
