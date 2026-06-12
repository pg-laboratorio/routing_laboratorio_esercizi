# Routing Basics (Inoltro dei Pacchetti)


## Sezione 1: Tabelle di Routing

In questa sezione viene analizzato il comportamento logico di un router durante l'esame della propria tabella di routing. L'obiettivo è determinare l'interfaccia di uscita o il prossimo salto (Next-Hop) di un pacchetto basandosi sulla regola del **Longest Prefix Match** (la maschera più specifica vince sempre) e sull'utilizzo della rotta di default.


### Esercizio 1.1: Analisi della Tabella di Routing

Dato un router con la seguente tabella di routing attiva all'interno della memoria RAM:

| Prefisso di Rete / Subnet | Next-Hop / Interfaccia | Tipo di Rotta | Descrizione |
| :--- | :--- | :--- | :--- |
| `10.0.0.0/8` | FastEthernet 0/0 | Connessa direttamente | Rete locale attestata sul router |
| `172.16.0.0/16` | 10.1.1.2 | Statica | Configurata manualmente dall'amministratore |
| `172.16.40.0/24` | 10.1.1.5 | OSPF | Appresa tramite protocollo di routing dinamico |
| `0.0.0.0/0` | 10.200.1.1 | Rotta di Default | Gateway di default per destinazioni sconosciute |

---

#### Determiniamo il percorso esatto per tre diversi pacchetti IP in transito sul router.

#### Caso 1: Pacchetto destinato a `172.16.40.50`

* **Rotte Corrispondenti:** L'indirizzo IP `172.16.40.50` corrisponde contemporaneamente a tre rotte presenti in tabella:
  1. `172.16.0.0/16` (perché inizia con 172.16)
  2. `172.16.40.0/24` (perché inizia con 172.16.40)
  3. `0.0.0.0/0` (la rotta universale)
* **Applicazione della Regola:** Il router applica il principio del *Longest Prefix Match*. Tra i prefissi corrispondenti (`/16`, `/24` e `/0`), il prefisso più lungo e specifico è il `/24`.
  > **Nota sul calcolo:** Il router esegue questa operazione convertendo gli indirizzi in formato binario e contando i bit coincidenti da sinistra verso destra. Il prefisso `/24` indica che i primi 24 bit dell'indirizzo di destinazione corrispondono esattamente alla rotta indicata.
* **Decisione di Inoltro:** Il pacchetto viene inoltrato al Next-Hop **`10.1.1.5`** (tramite la rotta appresa via OSPF).

#### Caso 2: Pacchetto destinato a `172.16.20.11`

* **Rotte Corrispondenti:** L'indirizzo IP `172.16.20.11` corrisponde a due rotte in tabella:
  1. `172.16.0.0/16` (corrispondenza sui primi due ottetti)
  2. `0.0.0.0/0` (rotta di default)
  *(Nota: non corrisponde a `172.16.40.0/24` poiché il terzo ottetto è 20 e non 40).*
* **Applicazione della Regola:** Tra i prefissi utilizzabili (`/16` e `/0`), il prefisso `/16` è il più lungo e specifico per questa destinazione.
* **Decisione di Inoltro:** Il pacchetto viene inoltrato al Next-Hop **`10.1.1.2`** (tramite la rotta statica).

#### Caso 3: Pacchetto destinato a `8.8.8.8`

* **Rotte Corrispondenti:** L'indirizzo IP `8.8.8.8` non corrisponde a nessuna delle reti specifiche presenti in tabella (`10.0.0.0/8`, `172.16.0.0/16` o `172.16.40.0/24`).
* **Applicazione della Regola:** L'unica rotta che accetta questo pacchetto è la rotta `0.0.0.0/0`. Questo prefisso speciale indica "qualsiasi indirizzo" nelle configurazioni di routing e serve proprio a definire il gateway di default.
* **Decisione di Inoltro:** Il pacchetto viene inoltrato all'indirizzo del gateway di default **`10.200.1.1`** per essere spedito verso Internet.

> **Nota Bene:** La rotta di default (`0.0.0.0/0`) è considerata la "rotta di ultima istanza". Viene utilizzata dal router *solo ed esclusivamente* se l'indirizzo di destinazione non trova alcuna corrispondenza in altri prefissi di rete più specifici.

---

**Riepilogo delle Decisioni del Router**

| IP Destinazione | Rotta Selezionata | Next-Hop Finale | Motivazione Tecnica |
| :--- | :--- | :--- | :--- |
| `172.16.40.50` | `172.16.40.0/24` | **10.1.1.5** | Vince il prefisso `/24` (più specifico di `/16` e `/0`). |
| `172.16.20.11` | `172.16.0.0/16` | **10.1.1.2** | Non rientra nel `/24`. Il `/16` è più specifico della rotta di default. |
| `8.8.8.8` | `0.0.0.0/0` | **10.200.1.1** | Nessuna rotta specifica trovata; inoltrato al gateway di default. |

---

### Esercizio 1.2: Selezione del Percorso e Longest Prefix Match

In questo secondo scenario, analizziamo il comportamento di un router periferico (Branch Router) che deve gestire il traffico tra la propria rete locale, la sede centrale dell'azienda (suddivisa in più sotto-rettorati) e la rete Internet esterna.

Dato un router con la seguente tabella di routing attiva:

| Prefisso di Rete / Subnet | Next-Hop / Interfaccia | Tipo di Rotta | Descrizione |
| :--- | :--- | :--- | :--- |
| `192.168.1.0/24` | GigabitEthernet 0/0 | Connessa direttamente | Rete LAN locale degli uffici |
| `10.0.0.0/16` | 192.168.1.254 | Statica | Rete aziendale generica della Sede Centrale |
| `10.5.0.0/20` | 192.168.1.10 | BGP | Sottorete specifica del Data Center aziendale |
| `0.0.0.0/0` | 192.168.1.1 | Rotta di Default | Uscita verso il modem/router internet (ISP) |

---

#### Determiniamo il percorso esatto per quattro diversi pacchetti IP in transito sul router

#### Caso 1: Pacchetto destinato a `10.5.2.100`

* **Rotte Corrispondenti:** L'indirizzo IP `10.5.2.100` (che in binario inizia con il blocco di rete del Data Center) si sovrappone a tre righe della tabella:
  1. `10.0.0.0/16` (perché fa parte del macro-blocco `10.0.x.x`)
  2. `10.5.0.0/20` (perché rientra nel range specifico da `10.5.0.0` a `10.5.15.255`)
  3. `0.0.0.0/0` (la rotta di default)
* **Applicazione della Regola:** Il router confronta la lunghezza dei prefissi di rete utilizzabili: `/16`, `/20` e `/0`. Il prefisso `/20` rappresenta la maschera più lunga e restrittiva.
* **Decisione di Inoltro:** Il pacchetto viene inoltrato al Next-Hop **`192.168.1.10`** (indirizzo del router del Data Center).

#### Caso 2: Pacchetto destinato a `10.20.1.5`

* **Rotte Corrispondenti:** L'indirizzo IP `10.20.1.5` viene valutato dal router rispetto alle regole attive:
  1. Corrisponde a `10.0.0.0/16` (perché l'IP inizia con il primo ottetto 10).
  2. Corrisponde a `0.0.0.0/0` (rotta di default).
  *(Nota: non corrisponde a `10.5.0.0/20` perché il secondo ottetto è 20 e non 5).*
* **Applicazione della Regola:** Tra i due prefissi validi (`/16` e `/0`), il router sceglie la rotta specifica `/16` rispetto alla rotta di default per la regola del prefisso più lungo.
* **Decisione di Inoltro:** Il pacchetto viene inoltrato al Next-Hop **`192.168.1.254`**.

#### Caso 3: Pacchetto destinato a `192.168.1.50`

* **Rotte Corrispondenti:** L'indirizzo IP `192.168.1.50` attiva due corrispondenze in tabella:
  1. `192.168.1.0/24` (corrispondenza esatta sui primi tre ottetti)
  2. `0.0.0.0/0` (rotta di default)
* **Applicazione della Regola:** Il prefisso `/24` è decisamente più lungo e specifico rispetto a `/0`.
* **Decisione di Inoltro:** Poiché la rotta associata a `/24` è di tipo "Connessa direttamente", il router non ha bisogno di inviare il pacchetto a un altro router (Next-Hop). Il pacchetto viene iniettato direttamente sul cavo attraverso l'interfaccia fisica **`GigabitEthernet 0/0`** per raggiungere lo switch locale.

#### Caso 4: Pacchetto destinato a `1.1.1.1`

* **Rotte Corrispondenti:** L'indirizzo IP `1.1.1.1` (DNS pubblico di Cloudflare) viene confrontato con le destinazioni note (`192.168.1.0`, `10.0.0.0`, `10.5.0.0`). Nessuna di esse è compatibile.
* **Applicazione della Regola:** Il router si affida all'ultima risorsa disponibile, la rotta jolly `0.0.0.0/0`.
* **Decisione di Inoltro:** Il pacchetto viene indirizzato al Next-Hop **`192.168.1.1`**, che rappresenta l'IP del modem per l'uscita su Internet.

---

**Riepilogo delle Decisioni del Router**

| IP Destinazione | Rotta Selezionata | Interfaccia / Next-Hop | Motivazione Tecnica |
| :--- | :--- | :--- | :--- |
| `10.5.2.100` | `10.5.0.0/20` | **10.5.0.0/20** via **192.168.1.10** | Selezionata la rotta `/20` (Longest Prefix Match rispetto a `/16`). |
| `10.20.1.5` | `10.0.0.0/16` | **10.0.0.0/16** via **192.168.1.254** | Non appartiene al blocco `/20`. Vince il `/16` rispetto a `/0`. |
| `192.168.1.50` | `192.168.1.0/24` | **GigabitEthernet 0/0** | Destinazione locale. Il pacchetto esce direttamente dall'interfaccia. |
| `1.1.1.1` | `0.0.0.0/0` | **0.0.0.0/0** via **192.168.1.1** | Nessuna corrispondenza specifica; instradato tramite gateway di default. |

---

## Sezione 2: Instradamento pacchetti

Questa sezione analizza il meccanismo di incapsulamento e instradamento dei pacchetti attraverso più nodi di rete (Layer 3) e collegamenti fisici (Layer 2).


### Esercizio 2.1: Tracciamento del Pacchetto e Modifiche dei Layer (Hop-by-Hop)

#### Scenario della Topologia di Rete

![Topologia Esercizio 2.1](/immagini/schema_2_1.png)

**Dati dei Dispositivi:**
* **PC A (Sorgente):** IP `10.1.1.5` | MAC `AA:AA:AA:AA:AA:AA`
* **Router 1 (R1):**
    * Interfaccia Fa0/0 (lato PC A): IP `10.1.1.1` | MAC `11:11:11:11:11:11`
    * Interfaccia Fa0/1 (lato R2): IP `192.168.12.1` | MAC `22:22:22:22:22:22`
* **Router 2 (R2):**
    * Interfaccia G0/0 (lato R1): IP `192.168.12.2` | MAC `33:33:33:33:33:33`
    * Interfaccia G0/1 (lato Server): IP `172.16.1.1` | MAC `44:44:44:44:44:44`
* **Server B (Destinazione):** IP `172.16.1.100` | MAC `BB:BB:BB:BB:BB:BB`

Il PC A genera un pacchetto IP destinato al Server B con un valore **TTL iniziale pari a 64**. 

---

#### Determina come cambiano i valori di **IP Sorgente, IP Destinazione, MAC Sorgente, MAC Destinazione e TTL** nei tre segmenti del viaggio.


#### Tratto 1: Dal PC A all'interfaccia Fa0/0 del Router 1
Il PC A deve inviare il pacchetto a una rete remota, quindi lo incapsula indirizzandolo al suo Gateway di Default (R1).
* **IP Sorgente:** `10.1.1.5` (Resta invariato)
* **IP Destinazione:** `172.16.1.100` (Resta invariato)
* **MAC Sorgente:** `AA:AA:AA:AA:AA:AA` (L'interfaccia del PC A)
* **MAC Destinazione:** `11:11:11:11:11:11` (L'interfaccia Fa0/0 di R1, trovata tramite ARP)
* **TTL:** `64` (Il valore di partenza non è ancora decrementato perché non ha superato alcun router)

#### Tratto 2: Dall'interfaccia Fa0/1 di R1 all'interfaccia G0/0 di R2
Il Router 1 riceve il frame Layer 2, lo scapsula, legge l'IP di destinazione, consulta la sua tabella di routing e decide di inoltrarlo a R2. Prima di farlo, **riscrive l'header Layer 2** e decrementa il TTL di 1.
* **IP Sorgente:** `10.1.1.5` (L'identità del mittente originale non cambia)
* **IP Destinazione:** `172.16.1.100` (L'obiettivo finale rimane lo stesso)
* **MAC Sorgente:** `22:22:22:22:22:22` (Diventa il MAC dell'interfaccia di uscita di R1)
* **MAC Destinazione:** `33:33:33:33:33:33` (Diventa il MAC dell'interfaccia di ricezione del prossimo router, R2)
* **TTL:** `63` (Diminuito di 1 perché ha attraversato il "salto" del Router 1)

#### Tratto 3: Dall'interfaccia G0/1 di R2 al Server B
Il Router 2 ripete l'operazione: scapsula il frame Layer 2, vede che la rete `172.16.1.0/24` è connessa direttamente alla sua interfaccia G0/1, riscrive l'header Layer 2 per l'ultimo balzo e decrementa nuovamente il TTL.
* **IP Sorgente:** `10.1.1.5` (Invariato)
* **IP Destinazione:** `172.16.1.100` (Invariato)
* **MAC Sorgente:** `44:44:44:44:44:44` (Il MAC dell'interfaccia di uscita di R2)
* **MAC Destinazione:** `BB:BB:BB:BB:BB:BB` (Il MAC finale della scheda di rete del Server B)
* **TTL:** `62` (Diminuito ancora di 1 dopo aver attraversato il Router 2)

---

**Tabella Riassuntiva del Ciclo di Vita del Pacchetto**

| Tratto del Percorso | IP Sorgente | IP Destinazione | MAC Sorgente | MAC Destinazione | TTL |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Segmento 1** (PC A -> R1) | `10.1.1.5` | `172.16.1.100` | `AA:AA:AA:AA:AA:AA` | `11:11:11:11:11:11` | **64** |
| **Segmento 2** (R1 -> R2) | `10.1.1.5` | `172.16.1.100` | `22:22:22:22:22:22` | `33:33:33:33:33:33` | **63** |
| **Segmento 3** (R2 -> Server B) | `10.1.1.5` | `172.16.1.100` | `44:44:44:44:44:44` | `BB:BB:BB:BB:BB:BB` | **62** |

>#### Concetti Chiave:
>1.  **Layer 3 (IP) End-to-End:** Gli indirizzi IP non cambiano mai durante l'instradamento ordinario (salvo presenza di NAT/PAT).
>2.  **Layer 2 (MAC) Hop-by-Hop:** Gli indirizzi di collegamento cambiano a ogni singola tratta fisica. Il MAC destinazione indica sempre il "prossimo passo", non la destinazione finale.
>3.  **Il ruolo del TTL:** Ogni router decrementa il valore del TTL di 1 prima di inoltrare il pacchetto. Se il TTL dovesse raggiungere lo zero (`0`), il router scarterebbe il pacchetto inviando un messaggio ICMP di errore (*Time Exceeded*), evitando che i pacchetti girino all'infinito nella rete in caso di loop di routing.

---

### Esercizio 2.2: Tracciamento con Switch e NAT

Questo esercizio introduce scenari avanzati di rete aziendale, analizzando l'impatto di uno Switch di transito e l'intervento di una regola di NAT (Network Address Translation) sugli header del pacchetto.

#### Scenario della Topologia di Rete

![Topologia Esercizio 2.2](/immagini/schema_2_2.png)

**Dati dei Dispositivi:**
* **PC A (Sorgente LAN):** IP `192.168.1.5` | MAC `AA:AA:AA:AA:AA:AA`
* **Switch S1 (LAN Switch):** Dispositivo puramente Layer 2 | MAC di management `SS:SS:SS:SS:SS:SS`
* **Router 1 (R1 - Border Router con NAT attivo):**
    * Interfaccia Fa0/0 (lato LAN): IP `192.168.1.1` | MAC `11:11:11:11:11:11`
    * Interfaccia Fa0/1 (lato WAN Pubblica): IP `203.0.113.50` | MAC `22:22:22:22:22:22`
* **Router 2 (R2 - Gateway ISP):**
    * Interfaccia G0/0 (lato R1): IP `203.0.113.1` | MAC `33:33:33:33:33:33`
    * Interfaccia G0/1 (lato Internet): IP `198.51.100.1` | MAC `44:44:44:44:44:44`
* **Server B (Destinazione Internet):** IP `8.8.8.8` | MAC `BB:BB:BB:BB:BB:BB`

Il PC A genera un pacchetto diretto a `8.8.8.8` con un **TTL iniziale pari a 128**. 

---

#### Determina la composizione esatta dei parametri di Layer 2 e Layer 3 nei quattro segmenti fisici del transito.


#### Tratto 1: Dal PC A allo Switch S1
Il PC A incapsula il pacchetto IP inserendo come MAC di destinazione quello del suo Gateway (Router 1), poiché l'IP `8.8.8.8` si trova fuori dalla sua rete locale.
* **IP Sorgente:** `192.168.1.5`
* **IP Destinazione:** `8.8.8.8`
* **MAC Sorgente:** `AA:AA:AA:AA:AA:AA` (PC A)
* **MAC Destinazione:** `11:11:11:11:11:11` (Interfaccia LAN di R1)
* **TTL:** `128`

#### Tratto 2: Dallo Switch S1 all'interfaccia Fa0/0 del Router 1
Lo Switch S1 è un dispositivo di Layer 2. Analizza il MAC di destinazione, consulta la sua tabella MAC e inoltra il frame verso la porta corretta. 
* **IP Sorgente / Destinazione:** `192.168.1.5` / `8.8.8.8` (Invariati)
* **MAC Sorgente / Destinazione:** `AA:AA:AA:AA:AA:AA` / `11:11:11:11:11:11` (Invariati)
* **TTL:** `128` 

> **Nota Bene:** Lo Switch S1 lavora al Layer 2. Non legge l'header IP e **non decrementa il TTL**. Inoltre, non sostituisce i MAC sorgente/destinazione con il proprio MAC di gestione. Il frame passa inalterato.

#### Tratta 3: Dall'interfaccia Fa0/1 di R1 all'interfaccia G0/0 di R2
Il Router 1 riceve il frame, lo scapsula e lo elabora. Trattandosi di un router di confine, applica il **NAT (Source NAT / PAT)** prima di inoltrarlo su Internet: sostituisce l'IP privato del PC con il suo IP pubblico della porta WAN. Decrementa inoltre il TTL.
* **IP Sorgente:** **`203.0.113.50`** (L'IP privato è stato tradotto nell'IP pubblico di R1)
* **IP Destinazione:** `8.8.8.8` (Invariato)
* **MAC Sorgente:** `22:22:22:22:22:22` (Il MAC della porta WAN di uscita di R1)
* **MAC Destinazione:** `33:33:33:33:33:33` (Il MAC della porta di ingresso di R2)
* **TTL:** **`127`** (Diminuito di 1 dall'elaborazione di R1)

#### Tratta 4: Dall'interfaccia G0/1 di R2 al Server B
Il Router 2 riceve il pacchetto con gli IP già pubblici. Effettua il normale instradamento IP verso il server di destinazione finale, riscrive l'header Layer 2 e scala nuovamente il TTL.
* **IP Sorgente:** `203.0.113.50` (Resta l'IP pubblico del NAT di R1)
* **IP Destinazione:** `8.8.8.8`
* **MAC Sorgente:** `44:44:44:44:44:44` (Il MAC della porta di uscita di R2)
* **MAC Destinazione:** `BB:BB:BB:BB:BB:BB` (Il MAC del Server B)
* **TTL:** **`126`** (Diminuito di 1 dall'elaborazione di R2)

---

**Tabella Riassuntiva degli Header** (Livello Intermedio)**

| Tratta del Percorso | IP Sorgente | IP Destinazione | MAC Sorgente | MAC Destinazione | TTL |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Tratta 1** (PC A -> Switch) | `192.168.1.5` | `8.8.8.8` | `AA:AA:AA:AA:AA:AA` | `11:11:11:11:11:11` | **128** |
| **Tratta 2** (Switch -> R1) | `192.168.1.5` | `8.8.8.8` | `AA:AA:AA:AA:AA:AA` | `11:11:11:11:11:11` | **128** *(Invariato)* |
| **Tratta 3** (R1 -> R2) | **`203.0.113.50`** | `8.8.8.8` | `22:22:22:22:22:22` | `33:33:33:33:33:33` | **127** *(Router 1)* |
| **Tratta 4** (R2 -> Server B) | `203.0.113.50` | `8.8.8.8` | `44:44:44:44:44:44` | `BB:BB:BB:BB:BB:BB` | **126** *(Router 2)* |

>#### Concetti Chiave da ricordare:
>1.  **I dispositivi Layer 2 sono "trasparenti":** Gli switch standard non modificano gli indirizzi logici o fisici del traffico in transito e non alterano il contatore TTL.
>2.  **Il NAT modifica il Layer 3:** La regola generale "gli IP non cambiano mai da sorgente a destinazione" NON si applica quando un router esegue una traduzione NAT. Questo è fondamentale per permettere a miliardi di IP privati interni di navigare sulla rete pubblica usando pochissimi indirizzi IP instradabili.
