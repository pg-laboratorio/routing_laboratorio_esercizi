# Esercizi Pratici di Routing e Networking: Casi Reali Aziendali

Questo documento contiene una raccolta di esercizi strutturati basati su scenari aziendali realistici. Gli esercizi coprono concetti fondamentali di networking come: **Longest Prefix Match (LPM)**, **forwarding**, **decremento del TTL**, **scelta del Gateway**, **tracciamento degli header Layer 2/Layer 3**, **Network Address Translation (NAT)** e **troubleshooting delle tabelle di routing**.


---

### Esercizio 1: Accesso a una Filiale Aziendale (Longest Prefix Match)

**Scenario:**
La sede centrale di un'azienda logistica deve instradare il traffico verso una filiale periferica appena ristrutturata. Il router di core della sede centrale possiede una tabella di routing complessa che include rotte riassuntive (summary routes), rotte specifiche per le sottoreti della filiale e una rotta di default verso Internet.

**Tabella di Routing del Router (R1):**

| Rete di Destinazione | Next-Hop / Interfaccia |
| :--- | :--- |
| `10.0.0.0/8` | Interfaccia `Fa0/0` |
| `10.20.0.0/16` | Gateway `192.168.1.2` |
| `10.20.5.0/24` | Gateway `192.168.1.1` |
| `0.0.0.0/0` | Gateway `203.0.113.1` (Internet) |

#### Domande
Determinare quale specifica rotta verrà utilizzata dal router R1 e quale sarà l'azione di inoltro (Next-Hop o Interfaccia) per i pacchetti IP diretti verso i seguenti indirizzi di destinazione:
1. `10.20.5.100` (Un server di database locale alla filiale)
2. `10.20.8.5` (Una postazione VoIP della filiale)
3. `8.8.8.8` (Un server DNS pubblico su Internet)

<details>
<summary>Clicca qui per visualizzare la soluzione</summary>

#### Soluzione Completa e Analisi

La scelta della rotta corretta si basa rigorosamente sulla regola del **Longest Prefix Match (LPM)**: il router confronta i bit dell'IP di destinazione con le maschere di rete disponibili e sceglie la rotta con la maschera più lunga (più specifica).

| Destinazione | Rotta Selezionata | Next-Hop / Interfaccia | Motivazione Tecnica |
| :--- | :--- | :--- | :--- |
| **10.20.5.100** | `10.20.5.0/24` | `192.168.1.1` | Corrisponde sia a `/8`, `/16` che `/24`. La subnet `/24` è la più specifica (24 bit corrispondenti), quindi prevale per LPM. |
| **10.20.8.5** | `10.20.0.0/16` | `192.168.1.2` | Corrisponde a `10.0.0.0/8` e `10.20.0.0/16`. Non corrisponde a `10.20.5.0/24` (il terzo ottetto è 8, non 5). Prevale `/16` rispetto a `/8`. |
| **8.8.8.8** | `0.0.0.0/0` | `203.0.113.1` | Nessuna rotta aziendale specifica corrisponde a questo IP. Il traffico viene quindi catturato dalla rotta di default (Default Gateway). |

</details>

---

### Esercizio 2: Data Center e Cloud (Instradamento Ibrido)

**Scenario:**
Un'azienda fintech gestisce un'infrastruttura ibrida: una parte dei servizi risiede in un Data Center locale (on-premises) e una parte è ospitata sul Cloud pubblico. Le reti sono interconnesse tramite collegamenti dedicati ad alta velocità. Il router aziendale deve gestire le rotte verso i server fisici e le istanze cloud.

**Tabella di Routing del Router di Frontiera:**

| Rete di Destinazione | Next-Hop |
| :--- | :--- |
| `172.16.0.0/16` | `10.1.1.1` (Data Center Core) |
| `172.16.10.0/24` | `10.1.1.10` (Cloud Gateway VGW) |
| `172.16.10.128/25` | `10.1.1.20` (Subnet Sandbox Sviluppo Cloud) |
| `0.0.0.0/0` | `192.168.0.254` (Firewall per Internet) |

#### Domande
Identificare il percorso scelto (Next-Hop) e la rotta applicata per i pacchetti con le seguenti destinazioni:
1. `172.16.10.50`
2. `172.16.10.200`
3. `172.16.50.5`
4. `1.1.1.1`

<details>
<summary>Clicca qui per visualizzare la soluzione</summary>

#### Soluzione Completa e Analisi

Applicando l'algoritmo di corrispondenza binaria delle subnet (Longest Prefix Match):

| Destinazione | Rotta Selezionata | Next-Hop Corrispondente | Analisi di Corrispondenza |
| :--- | :--- | :--- | :--- |
| **172.16.10.50** | `172.16.10.0/24` | `10.1.1.10` | L'IP rientra nel range `172.16.10.0 - 172.16.10.255`. Non rientra nella `/25` (che parte da `.128`). La rotta `/24` è la più specifica. |
| **172.16.10.200** | `172.16.10.128/25` | `10.1.1.20` | L'IP rientra nel range della subnet `/25` (`172.16.10.128` - `172.16.10.255`), in quanto `200 >= 128`. Avendo una lunghezza di prefisso maggiore di `/24` e `/16`, viene scelta questa rotta. |
| **172.16.50.5** | `172.16.0.0/16` | `10.1.1.1` | Non appartiene alle reti `/24` e `/25` focalizzate sul terzo ottetto `.10`. Trova corrispondenza solo nella supernet aziendale `/16`. |
| **1.1.1.1** | `0.0.0.0/0` | `192.168.0.254` | Nessun prefisso specifico corrisponde. Il pacchetto viene instradato verso il firewall Internet tramite la rotta di default. |

</details>

---

### Esercizio 3: Analisi del Time-To-Live (TTL)

**Scenario:**
Durante una sessione di videoconferenza aziendale in tempo reale, un pacchetto video viene generato dalla postazione di un dipendente. Per raggiungere il server di streaming aziendale, il pacchetto deve attraversare la rete core dell'azienda superando diversi nodi di routing intermedio.

**Flusso del Pacchetto:**
`[PC Utente]` ➔ `[Router R1]` ➔ `[Router R2]` ➔ `[Router R3]` ➔ `[Server Video]`

*Nota tecnica:* Il sistema operativo del PC inizializza il campo TTL (Time-To-Live) nell'header IP a un valore standard di **64**.

#### Domande
1. Quale sarà il valore del campo TTL nell'header IP immediatamente all'uscita dal router R1 (sul link verso R2)?
2. Quale sarà il valore del campo TTL all'uscita dal router R2 (sul link verso R3)?
3. Quale sarà il valore del campo TTL quando il pacchetto viene finalmente consegnato al Server Video?

<details>
<summary>Clicca qui per visualizzare la soluzione</summary>

#### Soluzione Completa e Analisi

Il campo **TTL (Time-To-Live)** è un contatore a 8 bit inserito nell'header IP per prevenire i loop di routing infiniti. Ogni router Layer 3 che riceve e inoltra (effettua il forwarding di) un pacchetto ha l'obbligo di **decrementare il valore del TTL di 1 unità** prima di trasmetterlo sull'interfaccia di uscita. Se il TTL raggiunge lo zero, il router scarta il pacchetto e invia un messaggio ICMP *Time Exceeded* alla sorgente.

* **Valore Iniziale al PC:** `64`
* **Dopo il Router R1:** `63` (R1 riceve il pacchetto, consulta la tabella di routing, decrementa il TTL da 64 a 63 e lo invia a R2).
* **Dopo il Router R2:** `62` (R2 elabora il livello 3, decrementa il TTL da 63 a 62 e lo invia a R3).
* **Dopo il Router R3:** `61` (R3 elabora l'instradamento finalizzato alla consegna locale/diretta, decrementa da 62 a 61).
* **Arrivo al Server:** Il pacchetto viene ricevuto dalla scheda di rete del Server con un **TTL finale pari a 61**.

*Nota:* Gli switch Layer 2 attraversati all'interno delle stesse VLAN non modificano il valore del TTL, in quanto operano esclusivamente sull'header Ethernet (Layer 2).

</details>

---

### Esercizio 4: Individuazione del Gateway e Livello 2 vs Livello 3

**Scenario:**
Un amministratore di rete sta configurando una postazione di lavoro in un ufficio. Per comprendere la logica di inoltro dei pacchetti operata dallo stack TCP/IP dell'host, analizza i parametri IP locali assegnati staticamente.

**Configurazione IP del PC:**
* **Indirizzo IP:** `192.168.10.25`
* **Subnet Mask:** `255.255.255.0` (`/24`)
* **Default Gateway:** `192.168.10.1`

#### Domande
Per ciascuno dei seguenti indirizzi IP di destinazione, indicare se il PC invierà il frame Ethernet **direttamente all'host finale** (consegna locale tramite ARP) oppure se dovrà incapsulare il pacchetto indirizzandolo al **Default Gateway**:
1. `192.168.10.50`
2. `192.168.10.200`
3. `192.168.20.5`
4. `8.8.8.8`

<details>
<summary>Clicca qui per visualizzare la soluzione</summary>

#### Soluzione Completa e Analisi

L'host esegue un'operazione di AND logico bit a bit tra il proprio IP e la Subnet Mask per determinare l'indirizzo della propria rete locale (`192.168.10.0`). Qualsiasi destinazione compresa tra `192.168.10.1` e `192.168.10.254` fa parte dello stesso dominio di broadcast (consegna diretta). Qualsiasi IP esterno richiede l'intervento del router (Gateway).

1.  **192.168.10.50:** **Direttamente all'host finale.** L'indirizzo appartiene alla stessa subnet locale (`192.168.10.0/24`). Il PC utilizzerà il protocollo ARP per scoprire l'indirizzo MAC dell'host destinatario e invierà un frame Layer 2 diretto.
2.  **192.168.10.200:** **Direttamente all'host finale.** Come sopra, appartiene alla stessa subnet. Il pacchetto non attraversa il router.
3.  **192.168.20.5:** **Al Default Gateway.** Pur essendo un IP privato, appartiene a una subnet differente (`192.168.20.0/24`). Il PC invia il pacchetto inserendo l'IP di destinazione finale (`172.16.20.5`) nell'header di Layer 3, ma incapsula il frame inserendo il **MAC address del Gateway** (`192.168.10.1`) come destinazione di Layer 2.
4.  **8.8.8.8:** **Al Default Gateway.** L'IP appartiene a una rete remota pubblica su Internet. Il PC instrada l'operazione verso l'interfaccia interna del router aziendale.

</details>

---

### Esercizio 5: Tracciamento degli Header L2/L3 in un transito tra subnet

**Scenario:**
Un host aziendale della rete Amministrazione deve inviare un file a un server posizionato all'interno della rete di Produzione. Le due reti sono separate da un router che effettua l'instradamento e l'interconnessione dei due segmenti fisici.

**Topologia Logica:**
`[PC]` ➔ `(Interfaccia LAN)` `[Router R1]` `(Interfaccia WAN)` ➔ `[Server]`

![schema_rete](/immagini/schema_5.png)

**Dati Tecnici Completi:**

| Dispositivo / Interfaccia | Indirizzo IP | Indirizzo MAC |
| :--- | :--- | :--- |
| **PC Client** | `192.168.1.10` | `AA:AA:AA:AA:AA:AA` |
| **R1 LAN** (Gateway PC) | `192.168.1.1` | `BB:BB:BB:BB:BB:BB` |
| **R1 WAN** | `172.16.1.1` | `CC:CC:CC:CC:CC:CC` |
| **Server Target** | `172.16.1.100` | `DD:DD:DD:DD:DD:DD` |

*Nota:* Il pacchetto viene originato dal PC con un valore **TTL iniziale pari a 128**.

#### Domande
Completare la seguente tabella di tracciamento degli header per descrivere come cambiano i campi Layer 3 (IP e TTL) e i campi Layer 2 (MAC) durante il transito nei due distinti segmenti di rete:

| Tratto / Segmento | IP Sorgente | IP Destinazione | MAC Sorgente | MAC Destinazione | Valore TTL |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Tratto 1:** PC ➔ R1 LAN | *?* | *?* | *?* | *?* | *?* |
| **Tratto 2:** R1 WAN ➔ Server | *?* | *?* | *?* | *?* | *?* |

<details>
<summary>Clicca qui per visualizzare la soluzione</summary>

#### Soluzione Completa e Analisi

La tabella correttamente compilata evidenzia un principio cardine delle reti: **gli indirizzi IP logici (Layer 3) rimangono invariati end-to-end**, mentre **gli indirizzi MAC fisici (Layer 2) cambiano a ogni hop di routing**. Il TTL viene decrementato dal router.

| Tratto / Segmento | IP Sorgente | IP Destinazione | MAC Sorgente | MAC Destinazione | Valore TTL |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Tratto 1:** PC ➔ R1 LAN | `192.168.1.10` | `172.16.1.100` | `AA:AA:AA:AA:AA:AA` | `BB:BB:BB:BB:BB:BB` | `128` |
| **Tratto 2:** R1 WAN ➔ Server | `192.168.1.10` | `172.16.1.100` | `CC:CC:CC:CC:CC:CC` | `DD:DD:DD:DD:DD:DD` | `127` |

**Dettagli dell'elaborazione:**
1.  **Nel Tratto 1**, il PC vuole parlare con una rete esterna, quindi indirizza il frame Ethernet al MAC del proprio gateway (`BB:...:BB`), mantenendo come IP finale quello del server.
2.  **Il Router R1** riceve il frame, decapsula il Layer 2, analizza l'IP destinazione `172.16.1.100` e decrementa il TTL da 128 a 127.
3.  **Nel Tratto 2**, il router ricapsula il pacchetto IP in un nuovo frame Ethernet. Cambia il MAC sorgente immettendo quello della propria interfaccia di uscita WAN (`CC:...:CC`) e scopre tramite ARP il MAC del Server di destinazione (`DD:...:DD`). Gli IP rimangono identici.

</details>

---

### Esercizio 6: Comportamento del NAT Aziendale (PAT)

**Scenario:**
Un utente all'interno di una rete LAN aziendale sta navigando su Internet e interroga un server web pubblico (es. un server DNS di Google o un portale web esterno). Il router di frontiera aziendale implementa la funzionalità di **NAT (Network Address Translation)** di tipo *Many-to-One* (noto anche come PAT - Port Address Translation) per consentire a centinaia di indirizzi privati di condividere un unico IP pubblico valido.

**Parametri di Rete:**
* **IP Privato Host Interno:** `192.168.1.50`
* **IP Pubblico Interfaccia WAN Router NAT:** `203.0.113.100`
* **IP Server Pubblico su Internet:** `8.8.8.8`

#### Domande
Rispondere ai quesiti analizzando le modifiche apportate dal router NAT durante l'attraversamento del pacchetto dall'interno verso l'esterno:
1. Quale indirizzo IP sorgente vedrà il server pubblico (`8.8.8.8`) quando riceve la richiesta HTTP/DNS?
2. Quale indirizzo IP sorgente era originariamente presente nell'header del pacchetto prima che questo transitasse sul router NAT?
3. Il processo di NAT modifica di per sé il valore del campo TTL dell'header IP? Spiegare il perché.
4. Il processo di NAT modifica gli indirizzi MAC del frame?

<details>
<summary>Clicca qui per visualizzare la soluzione</summary>

#### Soluzione Completa e Analisi

1.  **IP Sorgente rilevato dal server esterno:** `203.0.113.100`. Il router NAT sostituisce l'indirizzo privato non instradabile su Internet con il proprio indirizzo pubblico allocato dal provider.
2.  **IP Sorgente originario:** `192.168.1.50`. Questo indirizzo è visibile solo all'interno del segmento LAN locale prima di toccare l'interfaccia interna del router.
3.  **Modifica del TTL da parte del NAT:** **No.** Il NAT, per definizione tecnica, si occupa unicamente della traduzione degli indirizzi di livello 3 (e delle porte di livello 4). Tuttavia, poiché il dispositivo che esegue il NAT è a tutti gli effetti un **router Layer 3**, esso eseguirà *anche* l'operazione standard di routing sul pacchetto, comportando il consueto **decremento del TTL di 1 unità**. La riduzione del TTL è causata dall'azione di routing/forwarding, non dal meccanismo di traduzione NAT.
4.  **Modifica del MAC:** **Sì.** Come visto nell'Esercizio 5, il pacchetto attraversa un confine di routing Layer 3. Il frame Ethernet originario della LAN viene rimosso dal router e ne viene generato uno completamente nuovo per il transito sulla rete WAN del provider, con nuovi indirizzi MAC sorgente e destinazione appropriati per quel collegamento specifico.

</details>

---

### Esercizio 7: Troubleshooting di Rete e Connettività Mancante

**Scenario:**
Un amministratore di rete riceve un ticket di guasto (troubleshooting) da un utente che lavora nel reparto sviluppo. L'utente dichiara che la sua applicazione non riesce a stabilire una connessione con il server applicativo remoto che ha indirizzo IP `172.16.10.100`. L'ingegnere di rete accede tramite SSH al router aziendale incaricato dell'instradamento per quell'area e lancia il comando di visualizzazione della tabella di routing.

**Tabella di Routing estratta dal Router:**

| Rete di Destinazione | Interfaccia / Next-Hop |
| :--- | :--- |
| `10.0.0.0/8` | Interfaccia `Fa0/0` |
| `172.16.0.0/16` | Gateway `10.1.1.2` |

*Nota di configurazione:* Si evince chiaramente dalla tabella che **non è presente alcuna rotta di default (`0.0.0.0/0`)**.

#### Domande
Analizzando lo stato logico della tabella di routing sopra esposta, rispondere alle seguenti domande di diagnostica:
1. Il router è teoricamente in grado di instradare correttamente un pacchetto indirizzato al server di sviluppo `172.16.10.100`? Fornire la motivazione tecnica.
2. Se lo stesso utente provasse a navigare su una risorsa web esterna o a contattare un server DNS pubblico con IP `8.8.8.8`, il router sarebbe in grado di completare l'operazione?
3. Quale specifica ed essenziale configurazione manca all'interno di questa tabella di routing per garantire la piena connettività aziendale verso l'esterno, e quale comando/rotta andrebbe inserita?

<details>
<summary>Clicca qui per visualizzare la soluzione</summary>

#### Soluzione Completa e Analisi

1.  **Raggiungibilità di 172.16.10.100:** **Sì, il router può instradare questo traffico.** Quando arriva il pacchetto, il router esegue un match con la rotta `172.16.0.0/16`. L'indirizzo `172.16.10.100` rientra perfettamente in questo intervallo di indirizzi (i primi due ottetti corrispondono a 172.16). Il router effettuerà il forwarding inviando il traffico al Next-Hop `10.1.1.2`.
2.  **Raggiungibilità di 8.8.8.8:** **No, il pacchetto verrà scartato.** Il router esamina la tabella e nota che `8.8.8.8` non corrisponde né alla rete `10.0.0.0/8` né alla rete `172.16.0.0/16`. Poiché non esiste una rotta di default (`0.0.0.0/0`) che funga da "ultima spiaggia", il router applica la regola del *no match* e scarta immediatamente il pacchetto (generando internamente un errore di *Destination Network Unreachable*).
3.  **Configurazione Mancante:** Manca la configurazione di una **Rotta statico-dinamica di Default (Default Route)**. 
    * **Identificativo di rete:** `0.0.0.0/0` con subnet mask `0.0.0.0`.
    * In ambiente di rete (ad esempio in sintassi Cisco IOS), l'amministratore dovrebbe applicare un comando analogo al seguente per sanare il problema (ipotizzando un IP del provider IP di frontiera pari a `10.1.1.254` o interfaccia di uscita specifica):
        `ip route 0.0.0.0 0.0.0.0 10.1.1.254`
    L'inserimento di questa riga garantisce che tutto il traffico non esplicitamente menzionato per le reti interne (`10.0.0.0/8` e `172.16.0.0/16`) venga delegato al gateway di frontiera per l'accesso a Internet.

</details>
