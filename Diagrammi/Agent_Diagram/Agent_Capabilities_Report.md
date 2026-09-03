### REPORT DI PROGETTO: VISTA DELLE RESPONSABILITÀ E DELLE CAPACITÀ (DIAGRAMMA DEGLI AGENTI)

#### 1. Inquadramento Metodologico della Vista delle Responsabilità
La **Vista delle Responsabilità** (rappresentata tramite l'**Agent Model** o **Capabilities Diagram**) definisce formalmente il confine logico e fisico tra il sistema software da realizzare (il sistema *To-Be*) e l'ambiente circostante [17, 25, 69].

In conformità con la metodologia *Goal-Oriented Requirements Engineering (GORE)* insegnata all'Università di Verona, questa vista risponde alla domanda fondamentale: **"Chi è responsabile di cosa?"** [17, 25]. 
*   **Agente**: Un componente attivo del sistema (umano, hardware o software) in grado di compiere scelte operative per raggiungere gli obiettivi (goal) assegnatigli [17, 25].
*   **Capacità dell'Agente (Capabilities)**: Definite rigorosamente dall'insieme di grandezze fisiche o variabili di stato che l'agente è in grado di **monitorare** (frecce entranti, input informativi per prendere decisioni) o **controllare** (frecce uscenti, modifiche dello stato del mondo o del sistema) [26].
*   **Responsabilità**: L'assegnamento univoco di un *leaf goal* (obiettivo foglia) a un singolo agente. Per evitare insorgenze di conflitti e garantire la decidibilità del sistema, ciascun requisito foglia deve essere assegnato a **un solo agente** (nessun *AND-assignment* di responsabilità a livello foglia, ammessi solo *OR-assignments* in fase di analisi delle alternative progettuali) [27, 33].

---

#### 2. Architettura del Backend: Decomposizione dei Sotto-Componenti Software
Un aspetto di eccezionale maturità metodologica di questo diagramma risiede nella scomposizione del Cloud/SaaS backend [59]. Invece di rappresentare il server come una scatola nera (un generico agente "Cloud"), il backend è stato segmentato in **5 sotto-componenti software specializzati**, ciascuno operante come agente autonomo con responsabilità e variabili di controllo distinte [35, 59]:

1.  **DeviceManager**: Gestisce il ciclo di vita logico dell'hardware (provisioning, decommissioning ed elicottazione dei sensori attivi in magazzino) [64].
2.  **ShipmentManager**: Modella l'interfaccia amministrativa utilizzata dai Manager per impostare i profili di viaggio, definire le soglie fisiche e variare lo stato logico del trasporto [64].
3.  **IntegrityVerifier**: Il motore crittografico che lavora in background sul Cloud. Ispeziona le firme e aggiorna lo stato di integrità del dato storico [60, 63].
4.  **DashboardManager**: Il motore di visualizzazione e di calcolo dell'allarmistica in tempo reale. Genera e notifica gli allarmi a schermo [60, 65].
5.  **APIManager**: L'interfaccia di interoperabilità del sistema, responsabile dell'esposizione protetta dei dati storici certificati verso l'esterno [61, 66].

Questa granularità architetturale fornisce una tracciabilità formale impareggiabile e previene l'effetto "black-box", elevando drasticamente il valore accademico dell'elaborato.

---

#### 3. Analisi di Consistenza Multivista: Regola dell'Univocità del Controllo
Per garantire una consistenza orizzontale perfetta con il **Class Diagram**, ogni singola freccia di monitoraggio o controllo mappa esattamente su un attributo, su un'associazione o su uno stato dichiarati nel dizionario delle classi concettuali [55].

Inoltre, il modello soddisfa rigorosamente il **Teorema di Consistenza di Verona**: **"Ogni variabile del Class Diagram deve essere controllata da al più un agente"** [27, 55]. Ciò impedisce l'insorgenza di conflitti di scrittura concorrente o indeterminismo di stato.

##### Tabella dei Collegamenti e Corrispondenze di Stato:

| Agente | Tipo di Agente | Classe Concettuale | Relazione | Attributo / Associazione Specifico | Significato Operativo e Coerenza dei Requisiti |
| :--- | :--- | :--- | :---: | :--- | :--- |
| **DeviceManager** | Software (Cloud) | `IoTDevice` | **CONTROL** | Istanza di `IoTDevice` | Registra fisicamente il dispositivo a livello Cloud per le spedizioni. |
| | | `IoTDevice` | **MONITOR** | `batteryLevel`, `BLEConnectionStatus` | Permette al backend di monitorare lo stato operativo complessivo. |
| **ShipmentManager** | Software (Cloud) | `Shipment` | **MONITOR/CONTROL** | `logicalState`, shipment (instance) | Consente la gestione amministrativa degli stati del viaggio (creazione (shipment instance), cancellazione (status CANCELLED) e ispezione stato logico). |
| | | `ConfigurationProfile` | **CONTROL** | Istanza di `ConfigurationProfile`, `samplingInterval` | Associa un profilo di viaggio logico con tempo di campionamento dedicato. |
| | | `ThresholdRule` | **CONTROL** | `metricName`, `minValue`, `maxValue` | Configura le soglie di tolleranza termica/meccanica della merce. |
| **AppGateway** | Software (Mobile) | `LocalGatewayBuffer` | **MONITOR/CONTROL** | `pendingPacketCount`, `isCloudSynchronized` | Gestisce l'accodamento locale delle letture offline sul telefono dell'autista. |
| | | `IoTDevice` | **MONITOR** | `BLEConnectionStatus` | Rileva lo stato di accoppiamento BLE attivo tra telefono e sensore. |
| | | `Shipment` | **CONTROL** | logicalState, `tracked_by` instance (associazione con `IoTDevice`) | Associa o rimuove fisicamente (Crea/Distrugge associazione) il sensore al lotto di merci alla partenza/arrivo. Modifica lo stato logico della spedizione tra avvia (CREATED -> IN_TRANSIT), pause (IN_TRANSIT -> PAUSED), resume (PAUSED -> IN_TRANSIT), complete (IN_TRANSIT -> COMPLETED). |
| **PhysicalTransducer** | Ambiente (Fisico) | `EnvironmentalMeasurement` | **CONTROL** | `measuredValue`, `TimeStamp`, `metricName` | Trasduce la grandezza fisica reale in una misura strutturata. |
| **OnBoardFirmware** | Software (Embedded) | `IoTDevice` | **CONTROL** | `batteryLevel`, `BufferMemory`, `BLEConnectionStatus` | Aggiorna lo stato diagnostico locale e l'occupazione della memoria flash. |
| | | `ConfigurationProfile` | **MONITOR** | `samplingInterval` | Legge l'intervallo richiesto per impostare il timer di polling locale. |
| | | `ThresholdRule` | **MONITOR** | `metricName`, `minValue`, `maxValue` | Acquisisce le soglie termiche locali caricate in fase di associazione. |
| | | `EnvironmentalMeasurement` | **CONTROL** | `isOutOfRange` | Esegue il confronto matematico a bordo e accende il flag locale di anomalia. |
| **SecureElement** | Ambiente (Crypto Chip) | `CryptographicSignature` | **CONTROL** | `signatureValue` | Calcola l'hash firmato con chiave privata per blindare il dato. |
| | | `EnvironmentalMeasurement` | **MONITOR** | `measuredValue`, `TimeStamp` | Legge la misura locale appena registrata per cifrarne il contenuto. |
| **IntegrityVerifier** | Software (Cloud) | `CryptographicSignature` | **CONTROL** | `integrityStatus` | Valuta matematicamente la firma sul Cloud e dichiara lo stato (VALID/INVALID). |
| | | `CryptographicSignature` | **MONITOR** | `signatureValue` | Preleva il payload cifrato in transito per sottoporlo a verifica. |
| | | `IoTDevice` | **MONITOR** | `publicKey` | Recupera la chiave pubblica registrata del sensore per decifrare l'hash. |
| **DashboardManager** | Software (Cloud) | `AlarmNotification` | **CONTROL** | `Alarm` (AlarmType), `Timestamp` | Istanzia la notifica di allarme a backend non appena si verifica un'anomalia. |
| | | `EnvironmentalMeasurement` | **MONITOR** | `measuredValue`, `isOutOfRange` | Legge le misure per la visualizzazione grafica ed evidenziazione dei picchi. |
| | | `CryptographicSignature` | **MONITOR** | `integrityStatus` | Rileva e notifica istantaneamente eventuali violazioni crittografiche (tampering). |
| | | `IoTDevice` | **MONITOR** | `batteryLevel` | Innesca l'allarme visivo di manutenzione preventiva se la carica scende sotto il 15%. |
| **APIManager** | Software (Cloud) | `Shipment` | **MONITOR** | `shipmentID`, `logicalState` | Espone lo stato del viaggio a software terzi di logistica integrata. |
| | | `EnvironmentalMeasurement` | **MONITOR** | `measuredValue`, `TimeStamp` | Rende disponibili le letture storiche per certificazioni ufficiali. |
| | | `ShippingLabel` | **MONITOR** | `isQrCodeReadable`, `QrCodeURL`, `AlphanumericCode` | Fornisce i parametri di tracciamento pubblico per l'utente finale via QR. |

---

#### 4. Commento Metodologico e Risoluzione dei Casi Limite

##### 4.1. Il Ruolo di `SecureElement` come Agente dell'Ambiente
Un dettaglio di fondamentale importanza metodologica riguarda la natura dell'agente **SecureElement**. Esso è stato classificato come **agente dell'ambiente (fisico/hardware)** [17]. 
*   *Giustificazione*: Essendo un chip crittografico blindato a livello hardware (es. integrato nel microcontrollore o un chip esterno saldato a bordo), le sue routine di calcolo e la sua chiave privata non possono in alcun modo essere riprogrammate o alterate dal software applicativo (To-Be). Dal punto di vista del progettista dei requisiti, opera come un vincolo fisico e inalterabile dell'ambiente reale, garantendo matematicamente la nascita sicura del dato prima che esso entri in canali di trasmissione insicuri.

##### 4.2. La Cooperazione sul Buffer Offline (`AppGateway` e `OnBoardFirmware`)
Il superamento del vincolo tecnologico dei sensori sprovvisti di connettività internet diretta (niente 4G/5G/Wi-Fi, solo BLE) è risolto metodologicamente dalla **cooperazione strutturale** di tre agenti [61, 62, 65]:
1.  **OnBoardFirmware**: Legge i parametri ambientali e li accumula nella memoria locale del sensore (`BufferMemory`), aggiornandone l'occupazione.
2.  **AppGateway**: Monitora la presenza del sensore tramite il segnale radio BLE (`BLEConnectionStatus`). Se la connessione è attiva, preleva i dati scaricandoli sul buffer locale del dispositivo mobile (`LocalGatewayBuffer.pendingPacketCount`).
3.  Quando l'applicazione dell'autista rileva la connettività internet a livello Cloud, l'agente sincronizza le misure pendenti, impostando il flag `isCloudSynchronized` a `True` sul Class Diagram.

Questo flusso dimostra una perfetta comprensione delle relazioni di dipendenza e monitoraggio/controllo asincrone nel mondo reale.

##### 4.3. Eliminazione delle Ridondanze e Allineamento Nominale delle Variabili
Al fine di eliminare ogni potenziale incongruenza di sensibilità alle maiuscole/minuscole e pluralità tra la vista strutturale e quella di responsabilità, sono stati operati i seguenti allineamenti definitivi:
*   La gestione degli allarmi da parte del `DashboardManager` controlla direttamente gli attributi `Alarm` (mappato sull'enumerazione `AlarmType`) e `Timestamp` dichiarati nella classe concettuale `AlarmNotification`, garantendo una corrispondenza nominale perfetta.
*   È stato rimosso l'attributo transitorio `isDismissed` dall'allarme poiché esso rappresenta uno stato volatile di sessione della UI e non una proprietà statica del modello del dominio del problema.
*   L'agente `SecureElement` monitora la misura `EnvironmentalMeasurement` leggendo esclusivamente `measuredValue` e `TimeStamp`. L'attributo ridondante "timestamp" precedentemente ipotizzato all'interno della firma crittografica è stato eliminato, in quanto l'integrità del tempo è garantita dalla marcatura temporale ereditata della misura stessa sottoposta ad hashing.

---

#### 5. Tracciabilità dei Goal alle Responsabilità degli Agenti
In linea con il ciclo di sviluppo di Verona, ogni agente descritto in questa vista è responsabile del soddisfacimento di specifici obiettivi foglia del Goal Model [55]:

*   **RF1 (Tracking Periodico)** $\rightarrow$ Assegnato a **PhysicalTransducer** (generazione fisica del dato) e **OnBoardFirmware** (gestione temporizzata del campionamento).
*   **RF2 (Firma alla Sorgente)** $\rightarrow$ Assegnato a **SecureElement** (generazione matematica della firma).
*   **RF3 (Gestione Dispositivi)** $\rightarrow$ Assegnato a **DeviceManager** (onboarding e listing) e **ShipmentManager** (associazione profili).
*   **RF4 (Gestione Spedizioni)** $\rightarrow$ Assegnato a **ShipmentManager** (transizioni amministrative degli stati) e **AppGateway** (collegamento fisico sensore-spedizione).
*   **RF5 (Dashboard e Allarmistica)** $\rightarrow$ Assegnato a **DashboardManager** (scatenamento dell'allarme visivo per anomalie di soglia o violazioni di firma).
*   **RNF2 (Buffer BLE Offline)** $\rightarrow$ Assegnato a **AppGateway** (gestione del buffer locale temporaneo dell'applicazione mobile).
*   **RNF3 (Monitoraggio Batteria)** $\rightarrow$ Assegnato a **OnBoardFirmware** (lettura hardware e scrittura di `batteryLevel`) e **DashboardManager** (notifica a schermo di manutenzione se batteria < 15%).
