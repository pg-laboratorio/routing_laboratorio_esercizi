# Esercizi di Routing e Inoltro dei Pacchetti

Questo modulo è focalizzato sui concetti fondamentali del routing, l'analisi delle tabelle d'instradamento e il ciclo di vita del pacchetto IP. L'obiettivo è fornire un metodo strutturato per comprendere le decisioni di inoltro di un router e i cambiamenti degli header di rete (Layer 2 e Layer 3) durante il transito end-to-end.

## Materiale

* [routing_esercizi_svolti.md](/routing_esercizi_svolti.md) — Documento completo con le spiegazioni teoriche, l'analisi logica delle tabelle di routing e gli esercizi svolti passo-passo sui cambiamenti degli header (Hop-by-Hop).

---

## Cosa impariamo a fare

In ogni esercizio applichiamo un ragionamento logico a step che ti aiuterà a capire esattamente come i dispositivi di rete gestiscono il traffico.

**Analisi delle Tabelle di Routing:**

* **Regola del Longest Prefix Match:** Confrontare destinazioni multiple e capire perché la maschera di sottorete più specifica vince sempre.
* **Gestione della Rotta di Default:** Comprendere quando e come viene utilizzata la rotta `0.0.0.0/0`.
* **Decisione di Inoltro:** Identificare con precisione il Next-Hop (indirizzo IP del prossimo router) o l'interfaccia fisica di uscita.

**Tracciamento del Pacchetto (Hop-by-Hop):**

* **Comportamento del Layer 3 (IP):** Analizzare come gli indirizzi IP sorgente e destinazione si mantengono da un capo all'altro della comunicazione.
* **Comportamento del Layer 2 (MAC):** Tracciare la riscrittura continua degli indirizzi MAC a ogni salto fisico (da router a router).
* **Gestione del TTL:** Calcolare il decremento del Time To Live a ogni attraversamento di un dispositivo Layer 3.

**Scenari Avanzati:**

* **Trasparenza del Layer 2:** Valutare il passaggio dei pacchetti attraverso gli Switch (che non intaccano IP, MAC originari o TTL).
* **Intervento del NAT:** Capire l'eccezione alla regola del Layer 3, tracciando le modifiche agli IP quando un router esegue la traduzione degli indirizzi di rete.
