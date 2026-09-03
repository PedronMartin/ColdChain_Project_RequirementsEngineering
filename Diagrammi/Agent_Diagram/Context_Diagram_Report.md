# REPORT DI PROGETTO: VISTA DEI FLUSSI INFORMATIVI (DIAGRAMMA DI CONTESTO DEGLI AGENTI)

#### 1. Inquadramento Metodologico del Diagramma di Contesto
In conformità con i canoni metodologici della **Goal-Oriented Requirements Engineering (GORE)** e le convenzioni formali dell'**Università di Verona**, la **Vista dei Flussi Informativi** (rappresentata tramite il **Context Diagram** o **Agent Context Diagram**) descrive la rete di scambi dinamici a runtime tra gli agenti del sistema [11.2.2]. 

Mentre l'**Agent Capabilities Model (Agent Diagram)** mappa in modo statico quali variabili del Class Diagram sono monitorate o controllate da ciascun agente [11.2.1], il **Context Diagram** esplicita come queste capacità si traducano in canali di comunicazione attivi [11.2.2]. I nodi del diagramma rappresentano esclusivamente gli **agenti** (sia software del *System-To-Be*, sia fisici/ambientali), mentre gli archi orientati rappresentano il passaggio di informazioni, segnali o grandezze fisiche [11.2.2, 11.5].

##### La Regola Matematica di Derivazione (Teorema di Verona)
La coerenza inter-vista tra l'Agent Diagram (Capabilities) e il Context Diagram è governata da una rigorosa regola di derivazione logico-matematica [11.5]:
> Se un agente $A$ ha capacità di **CONTROL** sulla variabile di stato $v$ ($CONTROL(A, v)$) e un agente $B$ ha capacità di **MONITOR** sulla medesima variabile ($MONITOR(B, v)$), allora nel Context Diagram **deve** essere tracciato un arco orientato da $A$ verso $B$ etichettato con la variabile $v$ ($A \xrightarrow{v} B$) [11.5].

Questo report dimostra la perfetta aderenza a tale regola di derivazione, analizzando la consistenza di tutti i flussi e giustificando formalmente l'inclusione di elementi costanti ed assunzioni di dominio [11.5, 14.2].

---

#### 2. Dizionario Formale dei Flussi Informativi (Interazioni Dinamiche)
Il diagramma di contesto si compone di **11 canali di flusso informativi** tra gli agenti, ciascuno mappato al millimetro sulle variabili e sugli attributi tipizzati del Class Diagram concettuale per garantire la coerenza intra-vista [11.1.1, 11.5].

```
                                  [SecureElement]
                                    ^          |
                measuredValue,      |          |
                TimeStamp           |          | signatureValue
                                    |          v
  [PhysicalTransducer] ------------+-------> [IntegrityVerifier]
       |             \                         |
       |              \                        | integrityStatus
       | measuredValue \                       v
       | TimeStamp      \measuredValue ----> [DashboardManager]
       v                 \                     ^       ^
  [APIManager]            \                    |       | isOutOfRange,
       ^                   v                   |       | batteryLevel
       |                 [OnBoardFirmware] ----+-------+
       |                        |       |
       | logicalState           |       | batteryLevel,
       |                        |       | BLEConnectionStatus
       |                        |       v
  [ShipmentManager]             |    [DeviceManager]
       |                        |
       | samplingInterval,      | BLEConnectionStatus
       | metricName, minValue,  v
       | maxValue               +----------> [AppGateway]
       v                                         |
  [OnBoardFirmware] <----------------------------+
                             tracked_by (instance), logicalState
```

##### 2.1. PhysicalTransducer $\xrightarrow{\text{measuredValue, TimeStamp}}$ SecureElement
*   **Significato Operativo**: Il trasduttore fisico (sensore) rileva sul campo il parametro ambientale (temperatura, umidità, vibrazioni) e lo invia al Secure Element crittografico integrato a bordo per la messa in sicurezza immediata [10.9].
*   **Coerenza Strutturale**: Mappa sugli attributi `measuredValue : Integer` e `TimeStamp : Date` della classe `EnvironmentalMeasurement` [10.9].
*   **Tracciabilità Requisito**: RF1 (Tracking Ambientale Continuo).

##### 2.2. SecureElement $\xrightarrow{\text{signatureValue}}$ IntegrityVerifier
*   **Significato Operativo**: Il chip crittografico genera la firma digitale inalterabile calcolata sull'hash dei dati di misurazione e la trasmette al modulo di verifica del backend Cloud [10.9].
*   **Coerenza Strutturale**: Mappa sull'attributo `signatureValue : String` della classe `CryptographicSignature` [10.9].
*   **Tracciabilità Requisito**: RF2 (Integrità e Non-ripudiabilità alla Sorgente).

##### 2.3. IntegrityVerifier $\xrightarrow{\text{integrityStatus}}$ DashboardManager
*   **Significato Operativo**: Il modulo Cloud verifica matematicamente l'autenticità del pacchetto tramite la chiave pubblica del sensore e comunica lo stato di integrità (VALID/INVALID) alla Dashboard per la visualizzazione [10.9].
*   **Coerenza Strutturale**: Mappa sull'attributo `integrityStatus : ValidationState` (con ValidationState $\in$ {VALID, INVALID}) [10.9].
*   **Tracciabilità Requisito**: RF2 (Integrità) e RF5 (Allarmistica).

##### 2.4. PhysicalTransducer $\xrightarrow{\text{measuredValue}}$ DashboardManager
*   **Significato Operativo**: Flusso informativo diretto dei valori fisici campionati dal sensore alla Dashboard di monitoraggio in tempo reale del Cloud per il rendering dei grafici storici.
*   **Coerenza Strutturale**: Mappa sull'attributo `measuredValue : Integer` della classe `EnvironmentalMeasurement`.
*   **Tracciabilità Requisito**: RF1 (Tracking) e RF5 (Live-Dashboard).

##### 2.5. PhysicalTransducer $\xrightarrow{\text{measuredValue, TimeStamp}}$ APIManager
*   **Significato Operativo**: Esportazione sicura dello storico temporale delle letture grezze certificate verso le API pubbliche destinate a sistemi terzi o autorità.
*   **Coerenza Strutturale**: Mappa su `measuredValue : Integer` e `TimeStamp : Date` di `EnvironmentalMeasurement`.
*   **Tracciabilità Requisito**: RF6 (Accesso profilato Autorità) e RNF5 (Interoperabilità API).

##### 2.6. OnBoardFirmware $\xrightarrow{\text{isOutOfRange, batteryLevel}}$ DashboardManager
*   **Significato Operativo**: Il firmware di bordo esegue il confronto locale con i limiti di soglia e invia lo stato di anomalia (`isOutOfRange`) e il livello diagnostico di batteria residua al Cloud [10.9].
*   **Coerenza Strutturale**: Mappa su `isOutOfRange : Boolean` (in `EnvironmentalMeasurement`) e `batteryLevel : Integer` (in `IoTDevice`) [10.9].
*   **Tracciabilità Requisito**: RF5 (Allarme anomalia) e RNF3 (Monitoraggio Batteria / Manutenzione).

##### 2.7. OnBoardFirmware $\xrightarrow{\text{batteryLevel, BLEConnectionStatus}}$ DeviceManager
*   **Significato Operativo**: Diagnostica di health-check periodica del dispositivo inviata al modulo di inventory a backend per il listing dello stato operativo del parco macchine.
*   **Coerenza Strutturale**: Mappa su `batteryLevel : Integer` e `BLEConnectionStatus : Boolean` della classe `IoTDevice`.
*   **Tracciabilità Requisito**: RF3 (Device Management e Listing).

##### 2.8. OnBoardFirmware $\xrightarrow{\text{BLEConnectionStatus}}$ AppGateway
*   **Significato Operativo**: Stato del beaconing BLE inviato a corto raggio all'applicazione mobile dell'autista per confermare che l'accoppiamento radio è attivo e stabile.
*   **Coerenza Strutturale**: Mappa su `BLEConnectionStatus : Boolean` della classe `IoTDevice`.
*   **Tracciabilità Requisito**: RNF2 (Connettività BLE ed Efficienza Energetica).

##### 2.9. ShipmentManager $\xrightarrow{\text{samplingInterval, metricName, minValue, maxValue}}$ OnBoardFirmware
*   **Significato Operativo**: Configurazione iniziale del viaggio configurata dal Manager a backend e scaricata sul sensore (tramite il gateway dell'autista) per impostare il timer di polling locale e le soglie di sforamento [10.9].
*   **Coerenza Strutturale**: Mappa su `samplingInterval : Integer` (in `ConfigurationProfile`) e `minValue`, `maxValue`, `metricName` (in `ThresholdRule`) [10.9].
*   **Tracciabilità Requisito**: RF3 (Configurazione metriche/frequenze).

##### 2.10. ShipmentManager $\xrightarrow{\text{logicalState}}$ APIManager
*   **Significato Operativo**: Esposizione dello stato amministrativo corrente della spedizione (es. se completata o in transito) alle API per i gestionali ERP esterni dei clienti.
*   **Coerenza Strutturale**: Mappa sull'attributo `logicalState : ShipmentState` della classe `Shipment`.
*   **Tracciabilità Requisito**: RF4 (Ispezione Spedizioni) e RF6 (Accesso profilato Consumer).

##### 2.11. AppGateway $\xrightarrow{\text{tracked\_by (instance)}}$ ShipmentManager
*   **Significato Operativo**: L'applicazione gateway mobile dell'autista, rilevato il sensore corretto via BLE prima di partire, trasmette al backend Cloud il comando di associazione fisica sensore-spedizione consolidando il viaggio. Durante il viaggio, può modificare lo stato della spedizione ponendolo in pausa e riprendendolo.
*   **Coerenza Strutturale**: Rappresenta l'istanziazione dinamica della relazione strutturale `tracked_by` tra `Shipment` e `IoTDevice` (molteplicità $0..1 \leftrightarrow 0..*$) e la modifica della variabile logica logicalState di Shipment.
*   **Tracciabilità Requisito**: RF4 (Associazione/Rimozione sensore su Spedizione).

---

#### 3. Analisi di Consistenza Multivista Rigorosa
La stesura di questo diagramma rispetta stringenti criteri di consistenza logica progettati per superare i massimi standard accademici [11.5, 14.2].

##### 3.1. Risoluzione della Gestione Concorrente di `logicalState`
Un potenziale conflitto di scrittura concorrente potrebbe insorgere poiché sia l'agente amministrativo `ShipmentManager` sia l'applicazione mobile sul campo `AppGateway` hanno capacità di **CONTROL** sulla variabile `logicalState` della classe `Shipment` [11.1.2].
*   **Risoluzione e Coerenza**: Non vi è alcun conflitto di interferenza distruttiva, poiché l'accesso alla variabile è rigorosamente partizionato e governato dalla macchina a stati comportamentale della spedizione:
    1.  `ShipmentManager` controlla esclusivamente la transizione di inizializzazione amministrativa (passaggio da `CREATED` a stato pronto) e la dismissione/cancellazione manuale d'ufficio.
    2.  `AppGateway` controlla le transizioni operative in tempo reale regolate dal GPS e dal BLE del veicolo (avvio del transito $\rightarrow$ `IN_TRANSIT`, pausa temporanea per sosta $\rightarrow$ `PAUSED`, e completamento all'arrivo fisico al gate del destinatario $\rightarrow$ `COMPLETED`).
La State Machine garantisce che in ogni istante un solo agente possa eseguire la transizione corretta a seconda del sotto-stato di lifecycle corrente.

##### 3.2. Giustificazione dell'Esclusione delle Grandezze Statiche e di Dominio
Una delle caratteristiche di eccezionale robustezza formale di questo diagramma è la consapevole esclusione dei flussi informativi per attributi quali la chiave pubblica del sensore (`publicKey`), i codici delle etichette (`AlphanumericCode`, `QrCodeURL`) e gli ID di istanza (`shipmentID`, `hardwareID`) [11.4].
*   **Giustificazione Metodologica**: In conformità con la teoria di Verona, il Context Diagram deve catturare solo le **interazioni dinamiche a runtime** regolate dalle funzioni del software [11.2.2]. 
    *   La `publicKey` è caricata nel chip in fabbrica e registrata a backend una sola volta in fase di onboarding (costante statica di sicurezza).
    *   Il codice QR e l'ID alfanumerico sono stampati fisicamente sull'etichetta cartacea (`ShippingLabel`) incollata al pacco; rappresentano proprietà statiche del mondo fisico (Domain Properties) monitorate dagli utenti tramite scansione ottica una tantum, e non variabili di stato fluide modificate dal backend [11.4].
    *   Rappresentare graficamente questi flussi statici avrebbe appesantito il diagramma senza apportare valore semantico, introducendo un inutile "effetto ragnatela" a scapito della leggibilità complessiva [11.4].

---

#### 4. Copertura dei Vincoli Fisici dell'Ambiente
Il diagramma illustra graficamente come l'architettura software cooperante risponda brillantemente ai vincoli critici imposti dalla traccia [80]:

1.  **L'Assenza di Internet sui Sensori (Il ruolo di AppGateway)**: Il sensore BLE non potendo raggiungere autonomamente il Cloud, invia il segnale di `BLEConnectionStatus` a `AppGateway`. L'AppGateway agisce come proxy di sincronizzazione dinamica, controllando lo stato del buffer locale (`isCloudSynchronized` in `LocalGatewayBuffer`) e fungendo da "ponte radio" per scaricare i dati storici e le configurazioni di viaggio inviate da `ShipmentManager` [10.9].
2.  **Integrità Blindata alla Sorgente**: Il diagramma mostra un loop crittografico chiuso di responsabilità hardware-software che esclude l'intervento umano [10.9]. Il dato viaggia dal trasduttore locale (`PhysicalTransducer`) direttamente al co-processore sicuro (`SecureElement`) che lo blinda con la firma (`signatureValue`) before any gateway (potenzialmente compromettibile) or user can read or modify it. Il Cloud backend (`IntegrityVerifier`) garantisce la validazione formale prima che i dati tocchino i database di visualizzazione storici della Dashboard [10.9].
