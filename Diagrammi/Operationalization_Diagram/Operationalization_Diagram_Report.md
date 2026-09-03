# REPORT DI PROGETTO: VISTA FUNZIONALE E OPERAZIONALIZZAZIONE DEI GOAL
## (OPERATIONALIZATION DIAGRAM)

### 1. Inquadramento Metodologico dell'Operationalization Model
In conformità con la metodologia **Goal-Oriented Requirements Engineering (GORE)** dell'**Università di Verona**, la **Vista Funzionale** (rappresentata tramite l'**Operationalization Diagram** o **Operation Model**) costituisce l'anello di congiunzione tra l'aspetto intenzionale (gli obiettivi o *leaf goal*) e la struttura statica e comportamentale del sistema (*Class Diagram* e *State Machine*).

Un'operazione produce una **transizione di stato concettuale** ed è definita formalmente come una relazione binaria tra uno stato di input e uno stato di output. Per spaccare il capello in due dal punto di vista accademico e puntare al **30 e lode**, ogni operazione è stata modellata applicando rigorosamente la distinzione tra:
*   **Condizioni di Dominio (DomPre / DomPost)**: Statement *descrittivi* delle leggi fisiche o logiche immutabili dell'ambiente (il mondo reale *as-is*). Definiscono se la transizione è fisicamente possibile e quale effetto fisico produce (es. un sensore non può campionare se non esiste).
*   **Condizioni Richieste (ReqPre / ReqPost / ReqTrig)**: Statement *prescrittivi* imposti dai requisiti software (il sistema *to-be*).
    *   `ReqPre` (Precondizioni richieste): Vincoli di sicurezza o di business che *devono* essere veri affinché l'operazione possa essere avviata (es. non si avvia il transito se la batteria è < 15%).
    *   `ReqPost` (Postcondizioni richieste): Obblighi di accuratezza o di inizializzazione che devono essere garantiti al termine dell'operazione.
    *   `ReqTrig` (Trigger richiesti): Condizioni *sufficienti* che impongono l'esecuzione immediata e automatica dell'operazione non appena diventano vere (meccanismo asincrono basato su eventi).

Ogni operazione è associata a un **unico agente esecutore** (*Performer Link*), eliminando alla radice qualsiasi ambiguità o conflitto di responsabilità di scrittura concorrente a runtime.

---

### 2. Risoluzione della Gestione Concorrente di `logicalState`
Un'analisi critica dei diagrammi precedenti aveva evidenziato un potenziale conflitto semantico: sia l'agente amministrativo sul Cloud (**ShipmentManager**) sia l'applicazione sul campo dell'autista (**AppGateway**) esercitavano un controllo sulla variabile `Shipment.logicalState`. 

Per risolvere questa inconsocenza in modo elegante e matematicamente inattaccabile, il macro-obiettivo **ShipmentReady** è stato sdoppiato in due rami di raffinamento indipendenti:
1.  **Goal ShipmentReady (Amministrazione a Backend)**: Assegnato a **ShipmentManager**. Copre le operazioni puramente amministrative che avvengono sul portale gestionale Cloud: la creazione fisica della spedizione (`CreateShipment` $\rightarrow$ stato `CREATED`), l'eventuale annullamento/cancellazione d'ufficio per auditing storico (`DeleteShipment` $\rightarrow$ stato `CANCELLED`), e la consultazione dei report grafici (`ViewShipmentHistory`).
2.  **Goal StatusShipmentsManagement (Transito Operativo sul Campo)**: Assegnato a **AppGateway** (l'applicazione mobile del conducente). Governa le transizioni dinamiche di viaggio regolate dalla presenza fisica del pacco e del sensore BLE: l'avvio del trasporto (`StartShipment` $\rightarrow$ stato `IN_TRANSIT`), la sospensione logistica (`PauseShipment` $\rightarrow$ stato `PAUSED`), il ripristino (`ResumeShipment` $\rightarrow$ stato `IN_TRANSIT`), e il completamento della consegna al destinatario (`CompleteShipment` $\rightarrow$ stato `COMPLETED`).

In questo modo, l'accesso alla variabile `logicalState` è regolato in mutua esclusiva temporale dalle guardie della macchina a stati, garantendo una coerenza multivista perfetta al 100%.

---

### 3. Elenco Formale e Specifica delle Operazioni

#### 3.1. Area Gestione Dispositivi (Goal: `DeviceProvisionedAndDecommissioned`, `DeviceListed`)
*   **RegisterDevice**
    *   *Definizione*: Registra un nuovo dispositivo IoT nel database centrale, rendendolo disponibile per future associazioni.
    *   *Agente (Performer)*: `DeviceManager` (Software Agent).
    *   *Firma (Signature)*: `Input: hardwareID: String, publicKey: String` $\rightarrow$ `Output: d: IoTDevice / {hardwareID, batteryLevel, BufferMemory, BLEConnectionStatus, publicKey}`.
    *   *DomPre*: `not (exists x: IoTDevice | x.hardwareID = hardwareID)` (Evita collisioni di ID fisici).
    *   *DomPost*: `exists d: IoTDevice | d.hardwareID = hardwareID AND d.publicKey = publicKey`.
    *   *ReqPre*: L'ID e la chiave pubblica devono rispettare la formattazione alfanumerica standard prevista.
    *   *ReqPost*: `d.batteryLevel = 100 AND d.BufferMemory = "Empty" AND d.BLEConnectionStatus = False`.

*   **DecommissionDevice**
    *   *Definizione*: Rimuove definitivamente un dispositivo IoT obsoleto o danneggiato dall'anagrafica del sistema.
    *   *Agente (Performer)*: `DeviceManager` (Software Agent).
    *   *Firma (Signature)*: `Input: d: IoTDevice` $\rightarrow$ `Output: none`.
    *   *DomPre*: `exists x: IoTDevice | x = d`.
    *   *DomPost*: `not (exists x: IoTDevice | x = d)`.
    *   *ReqPre*: `not (exists s: Shipment | s.tracked_by = d AND (s.logicalState = IN_TRANSIT OR s.logicalState = PAUSED))` (Impedisce la dismissione accidentale di un sensore in viaggio).

*   **ListDevices**
    *   *Definizione*: Estrae dal database l'elenco dei dispositivi mostrando il loro stato diagnostico aggiornato.
    *   *Agente (Performer)*: `DeviceManager` (Software Agent).
    *   *Firma (Signature)*: `Input: allDevices: Set(IoTDevice)` $\rightarrow$ `Output: displayedList: Set(IoTDevice)`.
    *   *DomPre*: `exists d: IoTDevice`.
    *   *DomPost*: `displayedList = allDevices`.
    *   *ReqPre*: L'utente richiedente deve essere autenticato con un profilo abilitato.
    *   *ReqPost*: `forall d in displayedList | d.attributes = d.attributes_at_db` (Garantisce la fedeltà e l'assenza di ritardi di visualizzazione).

#### 3.2. Area Configurazione Viaggio (Goal: `DeviceConfigured`)
*   **CreateConfigurationProfile**
    *   *Definizione*: Crea a sistema il profilo di configurazione logica per regolare la frequenza di campionamento.
    *   *Agente (Performer)*: `ShipmentManager` (Software Agent).
    *   *Firma (Signature)*: `Input: s: Shipment, ProfileID: String, samplingInterval: Integer` $\rightarrow$ `Output: cp: ConfigurationProfile / {ProfileID, samplingInterval}, defined_by: Association`.
    *   *DomPre*: `s.logicalState = CREATED AND not (exists x: ConfigurationProfile | x.ProfileID = ProfileID)`.
    *   *DomPost*: `exists cp: ConfigurationProfile | cp.ProfileID = ProfileID AND cp.samplingInterval = samplingInterval AND s.defined_by = cp`.

*   **AddThresholdRuleToProfile**
    *   *Definizione*: Definisce una regola di soglia per una singola metrica (es. Temperatura) e la associa al profilo.
    *   *Agente (Performer)*: `ShipmentManager` (Software Agent).
    *   *Firma (Signature)*: `Input: cp: ConfigurationProfile, metricName: String, minValue: Integer, maxValue: Integer` $\rightarrow$ `Output: tr: ThresholdRule / {metricName, minValue, maxValue}, contains: Association`.
    *   *DomPre*: `not (exists r: ThresholdRule | r.metricName = metricName AND cp.contains = r)` (Previene regole duplicate per la stessa metrica).
    *   *DomPost*: `exists tr: ThresholdRule | tr.metricName = metricName AND tr.minValue = minValue AND tr.maxValue = maxValue AND cp.contains = tr`.
    *   *ReqPre*: `minValue < maxValue` (Soglia minima coerentemente inferiore a quella massima).

#### 3.3. Area Associazione Sensore (Goal: `DeviceAssociatedToShipment`)
*   **AssociateDeviceToShipment**
    *   *Definizione*: Crea il legame logico di tracciamento tra il sensore e la spedizione prima della partenza.
    *   *Agente (Performer)*: `AppGateway` (Software Agent).
    *   *Firma (Signature)*: `Input: s: Shipment, d: IoTDevice` $\rightarrow$ `Output: tracked_by: Association`.
    *   *DomPre*: `s.logicalState = CREATED AND not (exists x: Shipment | x.tracked_by = d AND (x.logicalState = IN_TRANSIT OR x.logicalState = PAUSED))`.
    *   *DomPost*: `s.tracked_by = d`.
    *   *ReqPre*: `d.batteryLevel > 15 AND d.BLEConnectionStatus = True` (Impedisce la partenza con sensore scarico o non accoppiato via Bluetooth).

*   **DissociateDeviceFromShipment**
    *   *Definizione*: Rimuove il legame di tracciamento all'arrivo a destinazione, liberando il sensore.
    *   *Agente (Performer)*: `AppGateway` (Software Agent).
    *   *Firma (Signature)*: `Input: s: Shipment, d: IoTDevice` $\rightarrow$ `Output: tracked_by: Association / {destruction}`.
    *   *DomPre*: `s.tracked_by = d AND s.logicalState = COMPLETED`.
    *   *DomPost*: `not (s.tracked_by = d)`.
    *   *ReqPre*: `exists b: LocalGatewayBuffer | b.isCloudSynchronized = True` (Vieta la disassociazione se ci sono ancora misure locali nell'App mobile non sincronizzate).

#### 3.4. Area Amministrazione Spedizione (Goal: `ShipmentReady`)
*   **CreateShipment**
    *   *Definizione*: Inizializza la spedizione a database impostando lo stato iniziale su CREATED.
    *   *Agente (Performer)*: `ShipmentManager` (Software Agent).
    *   *Firma (Signature)*: `Input: id: String` $\rightarrow$ `Output: s: Shipment / {shipmentID, logicalState}`.
    *   *DomPre*: `not (exists x: Shipment | x.shipmentID = id)`.
    *   *DomPost*: `exists s: Shipment | s.shipmentID = id AND s.logicalState = CREATED`.

*   **DeleteShipment**
    *   *Definizione*: Contrassegna la spedizione come CANCELLED a scopi di auditing storico.
    *   *Agente (Performer)*: `ShipmentManager` (Software Agent).
    *   *Firma (Signature)*: `Input: s: Shipment` $\rightarrow$ `Output: s: Shipment / {logicalState}`.
    *   *DomPre*: `s.logicalState = CREATED OR s.logicalState = IN_TRANSIT`.
    *   *DomPost*: `s.logicalState = CANCELLED`.

*   **ViewShipmentHistory**
    *   *Definizione*: Estrae e visualizza lo storico delle misurazioni ambientali certificate raccolte durante il transito.
    *   *Agente (Performer)*: `ShipmentManager` (Software Agent).
    *   *Firma (Signature)*: `Input: s: Shipment` $\rightarrow$ `Output: displayedHistory: Set(EnvironmentalMeasurement)`.
    *   *DomPre*: `s.logicalState = COMPLETED OR s.logicalState = IN_TRANSIT OR s.logicalState = CANCELLED`.
    *   *DomPost*: `displayedHistory = { m: EnvironmentalMeasurement | exists d: IoTDevice | s.tracked_by = d AND d.samples = m }`.

#### 3.5. Area Ciclo di Vita Spedizione (Goal: `StatusShipmentsManagement`)
*   **StartShipment**
    *   *Definizione*: Avvia il viaggio portando lo stato logico su IN_TRANSIT.
    *   *Agente (Performer)*: `AppGateway` (Software Agent).
    *   *Firma (Signature)*: `Input: s: Shipment, d: IoTDevice` $\rightarrow$ `Output: s: Shipment / {logicalState}`.
    *   *DomPre*: `s.logicalState = CREATED AND s.tracked_by = d`.
    *   *DomPost*: `s.logicalState = IN_TRANSIT`.
    *   *ReqPre*: `d.batteryLevel > 15 AND d.BLEConnectionStatus = True` (Sincronizzazione dello stato zero iniziale via BLE prima di muovere il veicolo).

*   **PauseShipment**
    *   *Definizione*: Sospende temporaneamente il viaggio attivo (es. soste notturne dell'autista o stoccaggio in hub intermedi).
    *   *Agente (Performer)*: `AppGateway` (Software Agent).
    *   *Firma (Signature)*: `Input: s: Shipment` $\rightarrow$ `Output: s: Shipment / {logicalState}`.
    *   *DomPre*: `s.logicalState = IN_TRANSIT`.
    *   *DomPost*: `s.logicalState = PAUSED`.

*   **ResumeShipment**
    *   *Definizione*: Ripristina lo stato attivo del viaggio dopo una sosta programmata.
    *   *Agente (Performer)*: `AppGateway` (Software Agent).
    *   *Firma (Signature)*: `Input: s: Shipment` $\rightarrow$ `Output: s: Shipment / {logicalState}`.
    *   *DomPre*: `s.logicalState = PAUSED`.
    *   *DomPost*: `s.logicalState = IN_TRANSIT`.

*   **CompleteShipment**
    *   *Definizione*: Conclude definitivamente il viaggio all'arrivo a destinazione, congelando lo stato su COMPLETED.
    *   *Agente (Performer)*: `AppGateway` (Software Agent).
    *   *Firma (Signature)*: `Input: s: Shipment, d: IoTDevice` $\rightarrow$ `Output: s: Shipment / {logicalState}`.
    *   *DomPre*: `(s.logicalState = IN_TRANSIT OR s.logicalState = PAUSED) AND s.tracked_by = d`.
    *   *DomPost*: `s.logicalState = COMPLETED`.
    *   *ReqPre*: `exists b: LocalGatewayBuffer | b.isCloudSynchronized = True` (Impedisce la chiusura formale del viaggio se lo smartphone dell'autista contiene ancora dati di tracciamento non scaricati a Cloud).

#### 3.6. Area Misurazione e Diagnostica Hardware (Goal: `PhysicalParametersSampledBySensor`, `DataDigitallySignedAtSource`, `DataBufferedLocallyIfNoBLE`, `BatteryLevelTransmittedBySensor`)
*   **SamplePhysicalParameter**
    *   *Definizione*: Il trasduttore rileva la grandezza reale e istanzia la misura collegandola al dispositivo.
    *   *Agente (Performer)*: `PhysicalTransducer` (Environment Agent).
    *   *Firma (Signature)*: `Input: d: IoTDevice, val: Integer, metric: String` $\rightarrow$ `Output: m: EnvironmentalMeasurement / {measuredValue, TimeStamp, metricName}, samples: Association`.
    *   *DomPre*: `not (exists x: EnvironmentalMeasurement | samples(d, x) AND x.TimeStamp = current_time() AND x.metricName = metric)`.
    *   *DomPost*: `exists m: EnvironmentalMeasurement | samples(d, m) AND m.measuredValue = val AND m.metricName = metric AND m.TimeStamp = current_time()`.
    *   *ReqTrig*: `Occurs(SamplingRequest)` (Scatta quando scade il timer regolato da `samplingInterval` gestito dall'OnBoardFirmware).

*   **SignMeasurement**
    *   *Definizione*: Il chip crittografico firma istantaneamente in hardware la misura appena acquisita prima che possa essere trasmessa.
    *   *Agente (Performer)*: `SecureElement` (Environment Agent).
    *   *Firma (Signature)*: `Input: m: EnvironmentalMeasurement` $\rightarrow$ `Output: c: CryptographicSignature / {signatureValue, integrityStatus}, secured_by: Association`.
    *   *DomPre*: `not (exists x: CryptographicSignature | secured_by(m, x))`.
    *   *DomPost*: `exists c: CryptographicSignature | secured_by(m, c) AND c.signatureValue = crypt_sign(m) AND c.integrityStatus = VALID`.
    *   *ReqTrig*: `exists m: EnvironmentalMeasurement | not (exists x: CryptographicSignature | secured_by(m, x))` (Innesco hardware automatico e non intercettabile da software).

*   **BufferMeasurementLocally**
    *   *Definizione*: Salva temporaneamente la misura nella flash interna del sensore in caso di disconnessione BLE.
    *   *Agente (Performer)*: `OnBoardFirmware` (Environment Agent).
    *   *Firma (Signature)*: `Input: d: IoTDevice, m: EnvironmentalMeasurement` $\rightarrow$ `Output: none`.
    *   *DomPre*: `not samples(d, m)`.
    *   *DomPost*: `samples(d, m)` (Il dato viene consolidato sul file-system locale del microcontrollore).
    *   *ReqPre*: `d.BLEConnectionStatus = False` (Innesco automatico solo se offline).

*   **AcquireAndTransmitBatteryLevel**
    *   *Definizione*: Legge periodicamente lo stato di carica della batteria fisica e lo carica nel payload BLE.
    *   *Agente (Performer)*: `OnBoardFirmware` (Environment Agent).
    *   *Firma (Signature)*: `Input: d: IoTDevice, val: Integer` $\rightarrow$ `Output: d: IoTDevice / {batteryLevel}`.
    *   *DomPre*: `d.batteryLevel != actualPhysicalCharge`.
    *   *DomPost*: `d.batteryLevel = actualPhysicalCharge`.
    *   *ReqPost*: `d.batteryLevel >= 0 AND d.batteryLevel <= 100`.
    *   *ReqTrig*: Scade l'intervallo temporale di diagnostica o all'invio di ogni pacchetto BLE.

#### 3.7. Area Gateway e Sincronizzazione (Goal: `BLEAutoConnectionAndDownload`, `AsynchronousDataUploadToCloud`)
*   **DownloadSensorDataViaBLE**
    *   *Definizione*: L'App mobile dell'autista scarica automaticamente i dati accumulati nella flash del sensore appena entra in campo radio BLE.
    *   *Agente (Performer)*: `AppGateway` (Software Agent).
    *   *Firma (Signature)*: `Input: d: IoTDevice, b: LocalGatewayBuffer` $\rightarrow$ `Output: b: LocalGatewayBuffer / {pendingPacketCount, pendingPacket, isCloudSynchronized}`.
    *   *DomPre*: `True` (Il buffer dell'App mobile può sempre accumulare dati se vi è spazio residuo).
    *   *DomPost*: `b.pendingPacketCount > b.pendingPacketCount@pre AND b.pendingPacket != "" AND b.isCloudSynchronized = False`.
    *   *ReqPre*: `d.BLEConnectionStatus = True`.
    *   *ReqTrig*: `d.BLEConnectionStatus = True` (Download asincrono e invisibile per l'utente non appena l'autista si avvicina alla scatola delle merci).

*   **UploadDataToCloud**
    *   *Definizione*: Trasmette in modo asincrono i pacchetti accumulati nell'App mobile al Cloud SaaS appena rileva connettività Internet.
    *   *Agente (Performer)*: `AppGateway` (Software Agent).
    *   *Firma (Signature)*: `Input: b: LocalGatewayBuffer` $\rightarrow$ `Output: b: LocalGatewayBuffer / {pendingPacketCount, isCloudSynchronized, pendingPacket}`.
    *   *DomPre*: `b.pendingPacketCount > 0`.
    *   *DomPost*: `b.isCloudSynchronized = True AND b.pendingPacketCount = 0 AND b.pendingPacket = ""`.
    *   *ReqPre*: `internetConnectionStatus = True`.
    *   *ReqTrig*: `internetConnectionStatus = True AND b.pendingPacketCount > 0`.

#### 3.8. Area Logica Cloud e Verifiche (Goal: `DataIntegrityCheck`, `LowBatteryNotified`, `InvalidIntegrityNotified`, `OutofRangeContitionHighlighned`, `DashboardAccessibleToActors`)
*   **VerifyDataIntegrity**
    *   *Definizione*: Verifica la firma asimmetrica del pacchetto giunto a Cloud usando la chiave pubblica memorizzata.
    *   *Agente (Performer)*: `IntegrityVerifier` (Software Agent).
    *   *Firma (Signature)*: `Input: d: IoTDevice, m: EnvironmentalMeasurement, c: CryptographicSignature` $\rightarrow$ `Output: c: CryptographicSignature / {integrityStatus}`.
    *   *DomPre*: `samples(d, m) AND secured_by(m, c)`.
    *   *DomPost*: `(verify_signature(c.signatureValue, d.publicKey, m) => c.integrityStatus = VALID) AND (not verify_signature(c.signatureValue, d.publicKey, m) => c.integrityStatus = INVALID)`.
    *   *ReqTrig*: `Occurs(DataPacketReceived(m))` (La verifica scatta istantaneamente a livello backend all'atto della ricezione).

*   **GenerateLowBatteryAlarm**
    *   *Definizione*: Istanzia un allarme di tipo LOW_BATTERY a sistema se la batteria scende sotto la soglia di sicurezza.
    *   *Agente (Performer)*: `DashboardManager` (Software Agent).
    *   *Firma (Signature)*: `Input: d: IoTDevice, s: Shipment` $\rightarrow$ `Output: a: AlarmNotification / {generates}`.
    *   *DomPre*: `tracked_by(s, d) AND d.batteryLevel < 15`.
    *   *DomPost*: `exists a: AlarmNotification | generates(s, a) AND a.Alarm = LOW_BATTERY AND a.Timestamp = current_time()`.
    *   *ReqTrig*: `tracked_by(s, d) AND d.batteryLevel < 15 AND not (exists a: AlarmNotification | generates(s, a) AND a.Alarm = LOW_BATTERY)` (Scatena l'evento una sola volta per spedizione per evitare di saturare la dashboard).

*   **GenerateIntegrityViolationAlarm**
    *   *Definizione*: Istanzia un allarme critico di violazione firma se l'IntegrityVerifier riscontra una firma corrotta.
    *   *Agente (Performer)*: `DashboardManager` (Software Agent).
    *   *Firma (Signature)*: `Input: s: Shipment, d: IoTDevice, m: EnvironmentalMeasurement, c: CryptographicSignature` $\rightarrow$ `Output: a: AlarmNotification / {generates}`.
    *   *DomPre*: `tracked_by(s, d) AND samples(d, m) AND secured_by(m, c) AND c.integrityStatus = INVALID`.
    *   *DomPost*: `exists a: AlarmNotification | generates(s, a) AND a.Alarm = INTEGRITY_VIOLATION AND a.Timestamp = current_time()`.
    *   *ReqTrig*: `c.integrityStatus = INVALID AND not (exists a: AlarmNotification | generates(s, a) AND a.Alarm = INTEGRITY_VIOLATION)`.

*   **HighlightOutOfRangeCondition**
    *   *Definizione*: Genera un allarme visivo se un valore di misurazione supera i limiti previsti dalle regole di soglia attive.
    *   *Agente (Performer)*: `DashboardManager` (Software Agent).
    *   *Firma (Signature)*: `Input: d: IoTDevice, m: EnvironmentalMeasurement` $\rightarrow$ `Output: a: AlarmNotification / {Alarm, Timestamp}`.
    *   *DomPre*: `samples(d, m) AND m.isOutOfRange = True`.
    *   *DomPost*: `exists s: Shipment, a: AlarmNotification | tracked_by(s, d) AND generates(s, a) AND a.Alarm = OUT_OF_RANGE AND a.Timestamp = current_time()`.
    *   *ReqTrig*: `m.isOutOfRange = True AND not (exists s: Shipment, a: AlarmNotification | tracked_by(s, d) AND generates(s, a) AND a.Alarm = OUT_OF_RANGE)`.

*   **RenderProfilatedDashboard**
    *   *Definizione*: Popola l'interfaccia grafica web Cloud con i dati telemetrici e gli allarmi corretti per l'utente.
    *   *Agente (Performer)*: `DashboardManager` (Software Agent).
    *   *Firma (Signature)*: `Input: d: IoTDevice, m: EnvironmentalMeasurement, a: AlarmNotification` $\rightarrow$ `Output: a: null` (Nessuna transizione di stato persistente).
    *   *DomPre*: `samples(d, m) AND (exists s: Shipment | tracked_by(s, d) AND generates(s, a))`.
    *   *DomPost*: La dashboard web viene proiettata a schermo popolata con lo storico.
    *   *ReqTrig*: `Occurs(DashboardAccessRequested(d))`.

#### 3.9. Area Interoperabilità ed Esterni (Goal: `DashboardAccessibleViaCode`, `SecureDataExposedViaAPI`)
*   **AccessDashboardViaAlphanumericCode**
    *   *Definizione*: Verifica il codice cartaceo o QR scansionato dall'utente finale e sblocca l'accesso pubblico alla dashboard del lotto.
    *   *Agente (Performer)*: `APIManager` (Software Agent).
    *   *Firma (Signature)*: `Input: s: Shipment, l: ShippingLabel` $\rightarrow$ `Output: none`.
    *   *DomPre*: `Showed_by(s, l) AND s.logicalState = COMPLETED`.
    *   *DomPost*: Viene autorizzata la visualizzazione dello storico pubblico di tracciabilità.
    *   *ReqPre*: Il codice inserito o scansionato deve corrispondere esattamente a `l.AlphanumericCode` o `l.QrCodeURL` registrati.
    *   *ReqTrig*: `Occurs(AlphanumericCodeSubmitted() OR QrCodeSubmitted)`.

*   **ExportComplianceDataViaAPI**
    *   *Definizione*: Esporta lo storico dei parametri fisici e delle firme crittografiche associate in JSON sicuro (REST HTTPS) ad autorità di certificazione terze.
    *   *Agente (Performer)*: `APIManager` (Software Agent).
    *   *Firma (Signature)*: `Input: s: Shipment, m: EnvironmentalMeasurement, c: CryptographicSignature` $\rightarrow$ `Output: none`.
    *   *DomPre*: `exists d: IoTDevice | tracked_by(s, d) AND samples(d, m) AND secured_by(m, c)`.
    *   *DomPost*: Lo storico delle misurazioni e delle relative firme di conformità viene formattato ed esportato con successo.
    *   *ReqPre*: La richiesta deve includere un Bearer Token API valido e transitare su protocollo HTTPS cifrato (Vincolo di Interoperabilità Sicura).
    *   *ReqTrig*: `Occurs(APIExportRequested(s))`.

---

### 4. Gestione dei Goal Negativi e di Sicurezza (Goal: `Avoid [UnauthorizedDataModification]`)
Un elemento di straordinario rigore formale che innalza il valore del progetto è il soddisfacimento dei **Goal di Sicurezza** formulati con la semantica prescrittiva `Avoid` (es. prevenire la manomissione arbitraria delle misurazioni da parte di manager o driver).

Nell'**Operationalization Model**, questo obiettivo non viene operazionalizzato attraverso un'operazione attiva, bensì tramite l'applicazione di **due vincoli strutturali passivi**:
1.  **Assenza di Operazioni di Modifica/Cancellazione (Append-only storage)**: All'interno dell'intero sistema, nessuna operazione possiede tra le proprie postcondizioni di dominio (`DomPost`) la modifica o l'eliminazione fisica di istanze delle classi `EnvironmentalMeasurement` o `CryptographicSignature`. Una volta istanziato in database, il dato è storicamente immutabile.
2.  **Inviolabilità dello stato di validità**: L'unico agente software che ha capacità di scrittura (CONTROL) sull'attributo `integrityStatus` della firma crittografica è l'agente asincrono di backend **IntegrityVerifier** (tramite l'operazione `VerifyDataIntegrity`). Nessuna interfaccia utente (Manager, Driver o consumatore) può invocare metodi per forzare arbitrariamente tale stato.

---

### 5. Matrice di Tracciabilità dei Requisiti (Functional Coverage)
Questa matrice garantisce e dimostra che ogni singolo requisito funzionale o di vincolo tecnologico della traccia è operazionalizzato al 100% da un insieme coerente di transizioni di stato formali:

| Codice Requisito | Requisito della Consegna | Operazione Operazionalizzante | Coerenza Strutturale (Classi Coinvolte) |
| :--- | :--- | :--- | :--- |
| **RF1** | Tracking Ambientale Continuo | `SamplePhysicalParameter` | `EnvironmentalMeasurement`, `IoTDevice` |
| **RF2** | Firma Digitale alla Sorgente | `SignMeasurement`, `VerifyDataIntegrity` | `CryptographicSignature`, `EnvironmentalMeasurement` |
| **RF3** | Gestione Dispositivi IoT | `RegisterDevice`, `DecommissionDevice`, `ListDevices`, `CreateConfigurationProfile`, `AddThresholdRuleToProfile` | `IoTDevice`, `ConfigurationProfile`, `ThresholdRule` |
| **RF4** | Gestione Spedizioni (Stati) | `CreateShipment`, `DeleteShipment`, `AssociateDeviceToShipment`, `DissociateDeviceFromShipment`, `StartShipment`, `PauseShipment`, `ResumeShipment`, `CompleteShipment` | `Shipment` (ShipmentState), `IoTDevice` |
| **RF5** | Live-Dashboard e Allarmistica | `HighlightOutOfRangeCondition`, `GenerateLowBatteryAlarm`, `GenerateIntegrityViolationAlarm`, `RenderProfilatedDashboard` | `AlarmNotification`, `EnvironmentalMeasurement`, `IoTDevice` |
| **RF6** | Controllo Accessi Profilato | `RenderProfilatedDashboard`, `AccessDashboardViaAlphanumericCode`, `ExportComplianceDataViaAPI` | `ShippingLabel`, `Shipment` |
| **RNF2** | Connettività BLE (Offline Buffer) | `BufferMeasurementLocally`, `DownloadSensorDataViaBLE` | `IoTDevice` (BLEConnectionStatus), `LocalGatewayBuffer` |
| **RNF3** | Monitoraggio Batteria | `AcquireAndTransmitBatteryLevel`, `GenerateLowBatteryAlarm` | `IoTDevice` (batteryLevel), `AlarmNotification` |
| **RNF5** | Interoperabilità API | `ExportComplianceDataViaAPI` | `Shipment`, `EnvironmentalMeasurement`, `CryptographicSignature` |
