# REPORT DI PROGETTO: ANALISI DEI RISCHI E ROBUSTEZZA (DIAGRAMMA DEGLI OSTACOLI)

#### 1. Inquadramento Metodologico dell'Analisi dei Rischi (GORE)
In conformità con i principi del **Goal-Oriented Requirements Engineering (GORE)** e con le direttive accademiche dell'**Università di Verona**, la **Vista dei Rischi** (rappresentata tramite l'**Obstacle Diagram**) costituisce l'estensione difensiva del modello intenzionale [9.1]. 

Mentre il Goal Diagram definisce il comportamento ideale del sistema (il *mondo felice*), l'Obstacle Diagram modella gli ostacoli fisici, umani, tecnologici o procedurali che minacciano attivamente il soddisfacimento degli obiettivi foglia (*leaf goal*), compromettendo di riflesso l'obiettivo radice `ProductIntegrityPreserved` [9.1, 9.3.1].

##### 1.1. La Formualizzazione Semantica dell'Ostruzione
Un ostacolo $O$ è un'anomalia di comportamento o uno stato del mondo che ostruisce un goal $G$ se e solo se la presenza contemporanea dell'ostacolo e delle assunzioni di dominio $Dom$ rende logicamente impossibile il soddisfacimento del goal stesso [19]:
$$\{O, Dom\} \models \neg G$$

L'obiettivo ingegneristico dell'analisi consiste nell'identificare in modo sistematico tali condizioni di guasto a partire dai goal di alto livello (tramite negazione logica e raffinamenti AND/OR) e nel progettare **contromisure software o procedurali** (rappresentate come goal risolutori collegati tramite link di risoluzione) che neutralizzino o mitighino la minaccia, incrementando la robustezza complessiva del sistema [19, 21, 24].

---

#### 2. Analisi Sistematica dei Rami di Ostacolo (Dizionario dei Rischi)

Il modello si articola su due rami principali che rispecchiano le fasi operative della catena logistica di prodotti fragili (es. farmaci biologici o opere d'arte).

##### 2.1. Ramo A: Gestione Spedizioni e Dispositivi (Fase Configurazione / Handshake)
Questo ramo analizza i rischi procedurali ed elettronici che minacciano la preparazione amministrativa del viaggio e l'inizializzazione dei sensori in magazzino [71].

*   **Ostacolo 1: `DeviceNotProvisionedOrDecommissioned`**
    *   **Goal Ostruito**: `Achieve [DeviceProvisionedAndDecommissioned]` (Requisito assegnato a `DeviceManager`) [71].
    *   **Definizione**: *L'operatore logistico commette un errore manuale inserendo caratteri non conformi o ID errati durante le operazioni di onboarding o decommissioning di un sensore IoT, bloccandone l'operatività.*
    *   **Categoria**: Human Error / Procedural Hazard [71].
    *   **Risoluzione Software**: **Nessuna (Mitigazione all'Interfaccia Client)**. Essendo un rischio procedurale legato ad azioni umane non controllabili dal software a runtime, l'errore viene mitigato impedendo inserimenti sporchi tramite controlli sintattici sul client (es. espressioni regolari per la validazione dell'ID hardware) [71].
*   **Ostacolo 2: `DeviceNotListed`**
    *   **Goal Ostruito**: `Achieve [DeviceListed]` (Requisito assegnato a `DeviceManager`) [71].
    *   **Definizione**: *La dashboard di monitoraggio fallisce l'interrogazione dell'elenco dei sensori attivi a causa di una congestione di rete o di un timeout temporaneo del database Cloud.*
    *   **Categoria**: Infrastructure Fault / System Timeout [71].
    *   **Risoluzione Software**: **Nessuna (Tolleranza ai Guasti infrastrutturale)**. La minaccia è delegata alle policy di affidabilità del Cloud Host SaaS (es. repliche attive-attive, bilanciamento del carico, caching locale delle query di backend) [71].
*   **Ostacolo 3: `DeviceNotConfigured`**
    *   **Goal Ostruito**: `Achieve [DeviceConfigured]` (Requisito assegnato a `ShipmentManager`) [71].
    *   **Definizione**: *Il dispositivo IoT non applica correttamente in memoria i parametri di samplingInterval e le ThresholdRule a causa di un'interruzione brusca dello handshake radio.*
    *   **Categoria**: Communication Failure / Wireless Loss [71].
    *   **Risoluzione Software**: **Nessuna (Gestione a Basso Livello)**. Il rischio è mitigato in modo nativo dai tentativi di ritrasmissione automatici propri dello stack di protocollo Bluetooth Low Energy (handshake BLE gestito a livello firmware) [71].

##### 2.2. Ramo B: Tracciamento Ambientale e Batteria (Fase di Transito)
Questo ramo tratta le minacce fisiche, ambientali e di sicurezza attiva che incombono sui dati telemetrici e sull'hardware durante il viaggio [72].

*   **Ostacolo 4: `SensorPowerLoss`**
    *   **Goal Ostruito**: `Maintain [PhysicalParametersSampledBySensor]` (Aspettativa assegnata a `PhysicalTransducer`) [72].
    *   **Definizione**: *Il sensore IoT smette improvvisamente di campionare i dati a causa dell'esaurimento completo della batteria interna a metà del viaggio.*
    *   **Categoria**: Hardware Failure [72].
    *   **Risoluzione Software (Mitigazione Preventiva)**: `Achieve [LowBatteryNotified]` (Requisito assegnato a `DashboardManager`) [72].
    *   **Meccanismo di Risoluzione**: Il Cloud traccia l'attributo `batteryLevel` allegato a ogni pacchetto. Se il valore scende sotto la soglia critica del 15%, viene generato istantaneamente un allarme visivo `AlarmNotification` di tipo `LOW_BATTERY` sulla Live-Dashboard, consentendo la sostituzione programmata del sensore prima della partenza o del degrado [72].
*   **Ostacolo 5: `DataTamperedInTransit`**
    *   **Goal Ostruito**: `Avoid [UnauthorizedDataModification]` (Safety Goal presidiato in modo passivo/strutturale dal backend) [72].
    *   **Definizione**: *Un utente malintenzionato intercetta le trasmissioni radio BLE o Cloud per alterare i valori storici di temperatura e nascondere sbalzi termici distruttivi per la merce.*
    *   **Categoria**: Security Threat / Man-In-The-Middle (MITM) [72].
    *   **Risoluzione Software (Prevenzione Crittografica)**: `Maintain [DataIntegrityVerified]` (implementato dal requisito `DataIntegrityCheck` in capo all'agente `IntegrityVerifier`) [72].
    *   **Meccanismo di Risoluzione**: Le misure sono firmate a bordo dal chip crittografico (`SecureElement`) generando la classe `CryptographicSignature`. All'upload, l'agente `IntegrityVerifier` valida la firma asimmetrica con la `publicKey` del dispositivo. Se i dati sono alterati, la firma si invalida (`integrityStatus = INVALID`), il Cloud rigetta il dato e genera l'allarme `INTEGRITY_VIOLATION` sulla dashboard [72].
*   **Ostacolo 6: `CryptographicSignatureFailure`**
    *   **Goal Ostruito**: `Maintain [DataDigitallySignedAtSource]` (Aspettativa assegnata a `SecureElement`) [72].
    *   **Definizione**: *Il SecureElement fallisce l'applicazione della firma per un danno fisico irreversibile sul silicio del microcontrollore.*
    *   **Categoria**: Hardware Failure [72].
    *   **Risoluzione Software**: **Nessuna**. Trattandosi di un guasto irreversibile sul silicio, il software non può ripristinare il chip. Il Cloud rileva la mancanza sistematica di firme corrette, solleva un'anomalia di non-conformità procedurale e contrassegna il viaggio come compromesso [72].
*   **Ostacolo 7: `BLEConnectionLost`**
    *   **Goal Ostruito**: `Maintain [BLEAutoConnectionAndDownload]` (Requisito assegnato a `AppGateway`).
    *   **Definizione**: *La connessione radio wireless BLE tra il sensore nel pacco e lo smartphone dell'autista si interrompe durante il viaggio.*
    *   **Categoria**: Wireless Interruption.
    *   **Raffinemento OR**: Questo ostacolo è scomposto in due sotto-cause in OR:
        1.  `DriverTooFarFromCargo`: L'autista si allontana temporaneamente dal veicolo con lo smartphone in tasca, uscendo dal raggio BLE.
        2.  `PhoneBatteryDead`: Lo smartphone dell'autista si scarica completamente, spegnendo l'App Gateway.
    *   **Risoluzione (Mitigazione Ambientale / Fallback Hardware)**: `Maintain [DataBufferedLocallyIfNoBLE]` (Aspettativa assegnata a `OnBoardFirmware`) [80].
    *   **Meccanismo di Risoluzione**: In assenza di connessione BLE (`BLEConnectionStatus = False`), l'operazione `BufferMeasurementLocally` interviene facendo accumulare le misure campionate all'interno della memoria flash non volatile del sensore (`BufferMemory`). Al ripristino della connettività con lo smartphone, l'AppGateway scaricherà asincronamente l'intero storico memorizzato, evitando qualsiasi perdita informativa [80].
*   **Ostacolo 8: `GatewayUploadFailed`**
    *   **Goal Ostruito**: `Maintain [DataTransmissionSecureAndVerified]` (Goal logico di trasmissione continua).
    *   **Definizione**: *L'App Gateway mobile dell'autista non riesce a comunicare con il server Cloud a causa dell'assenza di copertura cellulare (4G/5G) in tratti stradali remoti.*
    *   **Categoria**: Network Outage / Environment Failure.
    *   **Risoluzione Software**: `Achieve [AsynchronousDataUploadToCloud]` (Requisito assegnato a `AppGateway`) [66].
    *   **Meccanismo di Risoluzione**: L'App mobile accumula le misure scaricate via BLE nel proprio database locale (`LocalGatewayBuffer.pendingPacketCount` incrementato e `isCloudSynchronized = False`). L'operazione `UploadDataToCloud` viene ritentata continuamente a runtime in background: non appena lo smartphone rileva connettività Internet attiva, scarica l'intero buffer sul Cloud impostando `isCloudSynchronized = True` [66, 85].
*   **Ostacolo 9: `QrCodeUnreadable`**
    *   **Goal Ostruito**: `Achieve [DashboardAccessibleToActors]` (tramite scansione veloce della label sul pacco).
    *   **Definizione**: *Il codice QR stampato sulla lettera di vettura fisica del collo è strappato, sbiadito o sporco, impedendone la lettura ottica tramite la fotocamera.*
    *   **Categoria**: Physical Media Degradation.
    *   **Risoluzione Software**: `Achieve [DashboardAccessibleViaCode]` (Requisito assegnato a `APIManager`).
    *   **Meccanismo di Risoluzione**: In caso di fallimento di scansione del codice QR, l'agente `APIManager` consente l'inserimento manuale, tramite interfaccia web, del codice alfanumerico univoco (`AlphanumericCode`) stampato in chiaro sulla lettera di vettura, permettendo la decodifica dell'ID spedizione e il caricamento dei grafici conformi.
*   **Ostacolo 10: `UnrepresentativeSensorPlacement` (o `SensorePosizionatoMale`)**
    *   **Goal Ostruito**: `Maintain [PhysicalParametersSampledBySensor]` (Aspettativa assegnata a `PhysicalTransducer`) [72].
    *   **Definizione**: *Il sensore IoT viene posizionato per errore al di fuori del contenitore refrigerato contenente i prodotti termosensibili, determinando letture di temperatura sballate che non rispecchiano lo stato reale della merce.*
    *   **Categoria**: Human Error / Procedural Hazard [72].
    *   **Risoluzione (Mitigazione Passiva)**: `ASM2 (Physical Placement Domain Assumption)` [73].
    *   **Meccanismo di Risoluzione**: Trattandosi di una negligenza operativa sul campo, il software SaaS Cloud non ha modo di rilevare a runtime se il sensore si trovi fisicamente dentro o fuori la scatola. Il rischio viene perciò mitigato e gestito a livello contrattuale e procedurale tramite la **Domain Assumption ASM2**, la quale assume il rispetto rigoroso di Standard Operating Procedures (SOP) e linee guida di imballaggio che obbligano il personale di magazzino ad ancorare stabilmente il dispositivo all'interno del collo prima di sigillarlo e far partire la spedizione [73].

---

#### 3. Analisi di Consistenza Multivista ed Evidenze di Pregio (30L)

L'analisi degli ostacoli presenta tre elementi di eccezionale coerenza formale con il resto del sistema:

1.  **Risoluzione della Troncamento Grafico**: Viene documentato che l'etichetta grafica troncata `PhysicalParametersSampledBySen` presente nel diagramma di Objectiver corrisponde formalmente e semanticamente all'aspettativa di dominio **`PhysicalParametersSampledBySensor`** (G11) del Goal Model.
2.  **Risoluzione della Sicurezza Matematica dell'Avoid**: L'ostacolo `DataTamperedInTransit` che minaccia il goal di sicurezza passivo `Avoid [UnauthorizedDataModification]` è risolto in modo forte tramite il link di risoluzione che punta al requisito software attivo **`DataIntegrityCheck`** in capo a `IntegrityVerifier`. Ciò dimostra come un obiettivo di non-interferenza passivo sia tradotto operativamente in un modulo di convalida asimmetrica a backend.
3.  **Gestione dell'Offline Asincrono**: Gli ostacoli combinati `BLEConnectionLost` (mancanza di raggio d'azione BLE del sensore) e `GatewayUploadFailed` (mancanza di rete 4G dello smartphone dell'autista) illustrano graficamente la robustezza dell'architettura multi-buffer. Anche in condizioni di totale assenza di connettività, i dati fluiscono dal sensore al buffer locale (`BufferMemory`) e da quest'ultimo al buffer dell'app mobile (`pendingPacketCount`), per poi essere sincronizzati asincronamente solo a rete ripristinata, eliminando la perdita di telemetrie storiche.

---

#### 4. Matrice di Tracciabilità degli Ostacoli (Obstacle Coverage)

La tabella seguente costituisce l'evidenza formale di copertura dei rischi del sistema. Ogni minaccia identificata è mappata sul goal ostruito, sulla contromisura di mitigazione/risoluzione (sia essa un requisito o un'assunzione) e sulle variabili del Class Diagram coinvolte:

| Codice Ostacolo | Nome Ostacolo Rilevato | Goal Ostruito | Contromisura di Risoluzione / Mitigazione | Attributi e Classi del Class Diagram Coinvolti |
| :---: | :--- | :--- | :--- | :--- |
| **O1** | `DeviceNotProvisionedOrDecommissioned` | G1: `DeviceProvisionedAndDecommissioned` | *Validazioni Client Input* | `IoTDevice.hardwareID` |
| **O2** | `DeviceNotListed` | G2: `DeviceListed` | *Policy di Ridondanza SaaS Cloud* | `IoTDevice.batteryLevel`, `BufferMemory`, `BLEConnectionStatus` |
| **O3** | `DeviceNotConfigured` | G3: `DeviceConfigured` | *Handshake automatico a livello BLE* | `ConfigurationProfile.samplingInterval` |
| **O4** | `SensorPowerLoss` | G11: `PhysicalParametersSampledBySensor` | G8: `LowBatteryNotified` | `IoTDevice.batteryLevel`, `AlarmNotification` (valore `LOW_BATTERY`) |
| **O5** | `DataTamperedInTransit` | G15: `Avoid [UnauthorizedDataModification]` | G14: `DataIntegrityCheck` | `CryptographicSignature.integrityStatus` (valore `INVALID`) |
| **O6** | `CryptographicSignatureFailure` | G10: `DataDigitallySignedAtSource` | *Audit di non-conformità post-viaggio* | `CryptographicSignature.signatureValue` |
| **O7** | `BLEConnectionLost` | G12: `BLEAutoConnectionAndDownload` | G9: `DataBufferedLocallyIfNoBLE` | `IoTDevice.BufferMemory`, `IoTDevice.BLEConnectionStatus` |
| **O8** | `GatewayUploadFailed` | *DataTransmissionSecureAndVerified* | G13: `AsynchronousDataUploadToCloud` | `LocalGatewayBuffer.pendingPacketCount`, `isCloudSynchronized` |
| **O9** | `QrCodeUnreadable` | G17: `DashboardAccessibleToActors` | G19: `DashboardAccessibleViaCode` | `ShippingLabel.AlphanumericCode`, `ShippingLabel.isQrCodeReadable` |
| **O10** | `UnrepresentativeSensorPlacement` | G11: `PhysicalParametersSampledBySensor` | **ASM2 (Physical Placement Assumption)** | *Nessuno (Vincolo procedurale esterno di imballaggio)* |
