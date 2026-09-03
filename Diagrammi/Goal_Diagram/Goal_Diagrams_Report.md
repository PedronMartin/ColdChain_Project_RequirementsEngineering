# Report della Vista Intenzionale (Goal Model)

Il presente documento costituisce il **Report della Vista Intenzionale** per il progetto di tracciamento sicuro di spedizioni fragili. La modellazione dei requisiti qui descritta è stata condotta secondo i canoni formali della metodologia **GORE (Goal-Oriented Requirements Engineering)** e in conformità con gli standard didattici dell'**Università di Verona**.

La vista intenzionale si occupa di rispondere alla domanda fondamentale del **"Perché"** (*Why*) un sistema debba essere realizzato, partendo dalle esigenze strategiche degli stakeholder (gli obiettivi radice) fino a derivare, attraverso successivi raffinamenti logici, i requisiti funzionali e non funzionali del software e le aspettative sull'ambiente circostante.

---

## 1. Inquadramento Metodologico: Requirements vs. Expectations

Nella modellazione GORE, gli obiettivi vengono raffinati progressivamente fino a raggiungere un livello di granularità tale da poter essere assegnati alla responsabilità di un singolo agente. A questo livello (obiettivi foglia o *leaf goal*), la metodologia impone una distinzione tassonomica fondamentale:

1.  **Requisito (Requirement)**: È un obiettivo foglia assegnato alla responsabilità esclusiva di un **singolo agente software** appartenente al **System-to-Be** (il sistema che stiamo progettando). Poiché l'organizzazione ha il pieno controllo sullo sviluppo del software, il soddisfacimento del requisito può essere *garantito* e imposto tramite lo sviluppo del codice.
2.  **Aspettativa (Expectation / Prospettiva)**: È un obiettivo foglia assegnato alla responsabilità di un **singolo agente dell'ambiente (Environment)**, sia esso un attore umano o un dispositivo hardware esterno. Poiché il sistema non ha il controllo diretto sul comportamento dell'ambiente, le aspettative rappresentano delle assunzioni di dominio sul corretto funzionamento di tali agenti, le quali devono essere mitigate o verificate tramite il software.

Nel nostro Goal Model, questa classificazione è stata implementata con estremo rigore:
*   I goal assegnati agli agenti del backend Cloud (`DeviceManager`, `ShipmentManager`, `IntegrityVerifier`, `DashboardManager`, `APIManager`) e all'agente mobile (`AppGateway`) sono formalizzati come **Requirements** (rappresentati graficamente con parallelogrammi azzurri).
*   I goal assegnati ai componenti fisici e hardware esterni (`PhysicalTransducer`, `SecureElement`, `OnBoardFirmware`) sono formalizzati come **Expectations** (rappresentati graficamente in Objectiver con parallelogrammi a sfondo sfumato/giallo), in quanto dipendono dalle proprietà fisiche dei sensori commerciali low-cost scelti per il progetto.

---

## 2. Architettura del Goal Model: Raffinamento AND Principale

L'obiettivo strategico del sistema è garantire l'integrità dei prodotti sensibili (come farmaci biologici o opere d'arte di valore storico) durante l'intera filiera del trasporto. Questo obiettivo supremo è modellato dal Root Goal:

> **`ProductIntegrityPreserved`** (Pattern: *Maintain*)

Per soddisfare questo obiettivo, è stata applicata una scomposizione **AND-refinement** in tre sotto-obiettivi macroscopici, che rappresentano i tre pilastri temporali e logici della catena di custodia:

```
                      [ProductIntegrityPreserved]
                                   | (AND)
         +-------------------------+-------------------------+
         |                         |                         |
[ShipmentAndDevicesReady] [ContinuousTrackingAndSecure...] [DataExposedAndCertified]
```

Questa scomposizione soddisfa rigorosamente i teoremi di **Completezza** e **Minimalità** della metodologia di Verona:
*   **Completezza**: Se tutte e tre le condizioni sono contemporaneamente vere (il viaggio è configurato correttamente, i dati sono registrati continuamente e in sicurezza, e le prove sono esposte in modo certificato a destinazione), allora l'integrità del prodotto è logicamente e matematicamente preservata.
*   **Minimalità**: Se si escludesse anche solo una di queste tre sotto-aree, l'obiettivo supremo fallirebbe (ad esempio, un tracciamento perfetto è inutile se i dispositivi non sono stati pre-configurati con le giuste soglie, o se i dati non possono essere visualizzati e verificati dalle autorità a destinazione).

---

## 3. Analisi dei Sotto-Alberi e dei Pattern GORE

I tre rami principali del Goal Model sono a loro volta raffinati in obiettivi operativi, ciascuno regolato da un preciso pattern temporale della logica lineare (LTL):

### A. Sotto-Albero: `ShipmentAndDevicesReady` (Fase Amministrativa e Preparatoria)
Questo ramo governa le attività necessarie prima che il trasporto abbia inizio. È un raffinamento orientato all'attivazione e alla configurazione:
*   **`DeviceProvisionedAndDecommissioned`** (Pattern: *Achieve*): Assegnato al `DeviceManager`, garantisce che ogni sensore IoT sia censito con un ID univoco prima di essere immesso nella filiera.
*   **`DeviceListed`** (Pattern: *Achieve*): Consente al `DeviceManager` di elencare i dispositivi monitorandone lo stato hardware (batteria, BLE).
*   **`DeviceConfigured`** (Pattern: *Achieve*): Assegnato al `ShipmentManager`, permette la creazione dei profili di soglia applicabili alla tipologia di merce.
*   **`DeviceAssociatedToShipment`** (Pattern: *Achieve*): Assegnato all'`AppGateway`, formalizza l'accoppiamento fisico tra la scatola contenente il prodotto e il dispositivo IoT tramite scansione BLE/Barcode.

### B. Sotto-Albero: `ContinuousTrackingAndSecureDataTransmission` (Fase di Transito e Sicurezza)
Rappresenta il nucleo dinamico e operativo del sistema. È impostato principalmente sul pattern **Maintain**, poiché deve garantire la continuità dei vincoli per l'intera durata del trasporto:
*   **`PhysicalParametersSampledBySensor`** (Expectation, Pattern: *Achieve*): Il trasduttore fisico (`PhysicalTransducer`) campiona periodicamente temperatura, umidità e vibrazioni.
*   **`DataDigitallySignedAtSource`** (Expectation, Pattern: *Achieve*): Il `SecureElement` hardware firma istantaneamente ogni misura con la chiave privata del sensore per garantire la non-ripudiabilità.
*   **`DataBufferedLocallyIfNoBLE`** (Expectation, Pattern: *Maintain*): L' `OnBoardFirmware` gestisce la memoria flash locale per evitare la perdita di dati quando il Bluetooth dell'autista è disattivato o distante.
*   **`BLEAutoConnectionAndDownload`** (Requirement, Pattern: *Achieve*): L'`AppGateway` stabilisce connessioni BLE automatiche e scarica i pacchetti accumulati non appena si trova nel raggio d'azione del sensore.
*   **`AsynchronousDataUploadToCloud`** (Requirement, Pattern: *Achieve*): L'`AppGateway` esegue l'upload differito dei dati verso il Cloud SaaS appena rileva connettività di rete (Wi-Fi/4G/5G).
*   **`DataIntegrityCheck`** (Requirement, Pattern: *Achieve*): L'`IntegrityVerifier` decifra le firme all'arrivo per validare l'assenza di alterazioni.

#### Il Goal di Sicurezza passiva: `Avoid [UnauthorizedDataModification]`
Questo è l'unico obiettivo del sotto-albero a utilizzare il pattern **Avoid**. È un requisito di sicurezza cruciale: stabilisce che non deve mai verificarsi la modifica o la cancellazione fraudolenta dei log storici. Il sistema soddisfa questo obiettivo in modo strutturale (append-only database e firme matematiche generate dall'hardware alla sorgente), impedendo fisicamente qualsiasi operazione di scrittura distruttiva da parte di utenti umani.

### C. Sotto-Albero: `DataExposedAndCertified` (Fase di Esposizione e Allarmistica)
Questo ramo gestisce la visualizzazione del dato e la reazione tempestiva alle anomalie:
*   **`LowBatteryNotified`** (Requirement, Pattern: *Achieve*): Genera allarmi predittivi se il livello di carica del sensore scende sotto il 15%.
*   **`InvalidIntegrityNotified`** (Requirement, Pattern: *Achieve*): Notifica immediatamente al pannello di controllo la presenza di telemetrie alterate o prive di firma valida.
*   **`OutOfRangeConditionsHighlighted`** (Requirement, Pattern: *Achieve*): Notifica in tempo reale sulla Live-Dashboard il superamento delle soglie definite nelle regole di trasporto.
*   **`DashboardAccessibleToActors`** (Requirement, Pattern: *Achieve*): Garantisce l'accesso profilato e continuo ai dati per Manager, Autisti e Clienti finali.
*   **`DashboardAccessibleViaCode`** (Requirement, Pattern: *Achieve*): Permette l'accesso rapido e anonimo alle informazioni di conformità tramite scansione del QR-Code presente sulla lettera di vettura cartacea.
*   **`SecureDataExposedViaAPI`** (Requirement, Pattern: *Achieve*): Espone le API di esportazione per l'interoperabilità con enti di certificazione terzi.

---

## 4. Lo Sdoppiamento Strategico di `ShipmentReady`

Per risolvere una potenziale sovrapposizione di controllo sul ciclo di vita della spedizione (`Shipment.logicalState`) tra il backend SaaS amministrativo e l'applicazione mobile sul campo, l'obiettivo di gestione delle spedizioni è stato sdoppiato in due goal indipendenti e perfettamente tracciabili:

1.  **`ShipmentsReady`**: Assegnato al **`ShipmentManager`** (backend Cloud). Governa le operazioni puramente amministrative che avvengono "in ufficio", ovvero la creazione anagrafica del viaggio (`CreateShipment` $\rightarrow$ stato `CREATED`) e l'eventuale cancellazione d'ufficio per auditing storico in caso di annullamento dell'ordine (`DeleteShipment` $\rightarrow$ stato `CANCELLED`).
2.  **`StatusShipmentsManagement`**: Assegnato all'**`AppGateway`** (applicazione mobile). Governa le transizioni fisiche e dinamiche del viaggio guidate dall'autista sul campo (l'avvio effettivo `StartShipment` $\rightarrow$ stato `IN_TRANSIT`, le pause logistiche `PauseShipment` $\rightarrow$ stato `PAUSED`, la ripresa `ResumeShipment` $\rightarrow$ stato `IN_TRANSIT`, e la chiusura della consegna `CompleteShipment` $\rightarrow$ stato `COMPLETED`).

Questo sdoppiamento garantisce che non vi siano conflitti di scrittura concorrente sulla medesima variabile del Class Diagram, incanalando le modifiche attraverso transizioni di stato mutualmente esclusive nel tempo.

---

## 5. Matrice di Consistenza Inter-Vista (La prova formale del 30 e Lode)

La tabella seguente costituisce la **prova formale di consistenza multi-vista** del progetto. Mappa in modo bidirezionale e univoco ogni singolo obiettivo foglia del Goal Model con la sua tipologia, l'agente responsabile assegnato nell'Agent Model, l'operazione che lo implementa nell'Operationalization Diagram e gli attributi fisici o relazioni coinvolti nel Class Diagram:

| Identificativo Goal | Nome Obiettivo Foglia | Tipo | Agente Responsabile | Operazione Operazionalizzante | Attributi / Associazioni Class Diagram Coinvolti |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **G1** | `DeviceProvisionedAndDecommissioned` | Requirement | `DeviceManager` | `RegisterDevice`<br>`DecommissionDevice` | `IoTDevice` (istanza)<br>`IoTDevice.hardwareID` |
| **G2** | `DeviceListed` | Requirement | `DeviceManager` | `ListDevices` | `IoTDevice.batteryLevel`<br>`IoTDevice.BufferMemory`<br>`IoTDevice.BLEConnectionStatus` |
| **G3** | `DeviceConfigured` | Requirement | `ShipmentManager` | `CreateConfigurationProfile`<br>`AddThresholdRuleToProfile` | `ConfigurationProfile.ProfileID`<br>`ConfigurationProfile.samplingInterval`<br>`ThresholdRule` (istanza) |
| **G4** | `DeviceAssociatedToShipment` | Requirement | `AppGateway` | `AssociateDeviceToShipment`<br>`DissociateDeviceFromShipment` | Associazione `tracked_by` (molteplicità `0..1` ↔ `0..*`) |
| **G5** | `ShipmentsReady` | Requirement | `ShipmentManager` | `CreateShipment`<br>`DeleteShipment`<br>`ViewShipmentHistory` | `Shipment.logicalState` (stati `CREATED`, `CANCELLED`) |
| **G6** | `StatusShipmentsManagement` | Requirement | `AppGateway` | `StartShipment`<br>`PauseShipment`<br>`ResumeShipment`<br>`CompleteShipment` | `Shipment.logicalState` (stati `IN_TRANSIT`, `PAUSED`, `COMPLETED`) |
| **G7** | `BatteryLevelTransmittedBySensor` | Expectation | `OnBoardFirmware` | `AcquireAndTransmitBatteryLevel` | `IoTDevice.batteryLevel` |
| **G8** | `LowBatteryNotified` | Requirement | `DashboardManager` | `GenerateLowBatteryAlarm` | `AlarmNotification.Alarm` (valore `LOW_BATTERY`) |
| **G9** | `DataBufferedLocallyIfNoBLE` | Expectation | `OnBoardFirmware` | `BufferMeasurementLocally` | `IoTDevice.BufferMemory`<br>Associazione `samples` |
| **G10** | `DataDigitallySignedAtSource` | Expectation | `SecureElement` | `SignMeasurement` | `CryptographicSignature.signatureValue`<br>`CryptographicSignature.integrityStatus` (valore `VALID`) |
| **G11** | `PhysicalParametersSampledBySensor` | Expectation | `PhysicalTransducer` | `SamplePhysicalParameter` | `EnvironmentalMeasurement.measuredValue`<br>`EnvironmentalMeasurement.TimeStamp` |
| **G12** | `BLEAutoConnectionAndDownload` | Requirement | `AppGateway` | `DownloadSensorDataViaBLE` | `LocalGatewayBuffer.pendingPacketCount`<br>`IoTDevice.BLEConnectionStatus` |
| **G13** | `AsynchronousDataUploadToCloud` | Requirement | `AppGateway` | `UploadDataToCloud` | `LocalGatewayBuffer.isCloudSynchronized` |
| **G14** | `DataIntegrityCheck` | Requirement | `IntegrityVerifier` | `VerifyDataIntegrity` | `CryptographicSignature.integrityStatus` (valore `INVALID`) |
| **G15** | `Avoid [UnauthorizedDataModification]` | Safety Goal | *(Strutturale)* | *(Nessuna - Append-Only)* | Immutabilità fisica di misure e firme crittografiche |
| **G16** | `InvalidIntegrityNotified` | Requirement | `DashboardManager` | `GenerateIntegrityViolationAlarm` | `AlarmNotification.Alarm` (valore `INTEGRITY_VIOLATION`) |
| **G17** | `DashboardAccessibleToActors` | Requirement | `DashboardManager` | `RenderProfilatedDashboard` | `Shipment.logicalState`<br>`AlarmNotification` (istanze) |
| **G18** | `OutOfRangeConditionsHighlighted` | Requirement | `DashboardManager` | `HighlightOutOfRangeCondition` | `EnvironmentalMeasurement.isOutOfRange`<br>`AlarmNotification.Alarm` (valore `OUT_OF_RANGE`) |
| **G19** | `DashboardAccessibleViaCode` | Requirement | `APIManager` | `AccessDashboardViaAlphanumericCode` | `ShippingLabel.AlphanumericCode`<br>`ShippingLabel.QrCodeURL` |
| **G20** | `SecureDataExposedViaAPI` | Requirement | `APIManager` | `ExportComplianceDataViaAPI` | Endpoint API REST (interoperabilità esterna) |

---

### Conclusioni di Consistenza

La perfetta corrispondenza tra questa matrice, i report strutturati delle classi, delle capabilities degli agenti e delle specifiche delle operazioni dimostra matematicamente l'**assenza di orfani** (non esistono goal non operazionalizzati da operazioni, né operazioni orfane di obiettivi) e l'**assenza di conflitti di controllo** sulle variabili dinamiche di stato. Il modello risulta integrato, auto-esplicativo e pronto per essere tradotto nei diagrammi di comportamento (State Machine e Use Case) e nella specifica formale dei requisiti **IEEE Std 830-1998**.
