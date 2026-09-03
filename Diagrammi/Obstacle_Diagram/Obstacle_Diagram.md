## Ostacoli da trasporre nei diagrammi
Questi ostacoli descrivono anomalie di comportamento del software, guasti hardware dell'ambiente, disconnessioni fisiche o incidenti procedurali che minacciano direttamente i sotto-goal foglia del sistema.

### Ramo A: gestione spedizioni e dispositivi (configurazione)
Questo ramo analizza i rischi legati alla fase di inserimento dati nel Cloud e all'accoppiamento iniziale dei dispositivi fisici ai colli prima della partenza.

1. **Ostacolo 1:** `DeviceNotProvisionedOrDecommissioned`
   * **Goal Ostruito:** `Achieve [DeviceProvisionedAndDecommissioned]` (Requisito software assegnato al software `DeviceManager`).
   * **Definizione:** *Il gestore o l'operatore del magazzino non riesce a registrare (provision) o a dismettere (decommission) un dispositivo IoT a causa di un errore accidentale di inserimento dati o di formattazione.*
   * **Categoria:** `Human Error` / `Procedural Hazard`
   * **Risoluzione Software:** **Nessuna**. Trattandosi di un rischio procedurale esterno legato all'input manuale dell'utente, l'errore viene gestito in modo elettrico o procedurale tramite controlli sintattici e validazioni sul client (es. espressioni regolari per l'ID del sensore) o richiedendo all'operatore di ripetere manualmente l'operazione.

2. **Ostacolo 2:** `DeviceNotListed`
   * **Goal Ostruito:** `Achieve [DeviceListed]` (Requisito software assegnato al software `DeviceManager`).
   * **Definizione:** *Il backend Cloud non è in grado di mostrare l'elenco dei sensori attivi e registrati nel sistema a causa di un timeout di rete o di una congestione temporanea del database.*
   * **Categoria:** `System Timeout` / `Infrastructure Fault`
   * **Risoluzione Software:** **Nessuna**. La continuità del servizio e la tolleranza ai guasti infrastrutturali vengono delegate alle policy di affidabilità del Cloud Host SaaS (es. bilanciamento del carico, caching locale delle query o repliche attive-attive del database).

3. **Ostacolo 3:** `DeviceNotConfigured`
   * **Goal Ostruito:** `Achieve [DeviceConfigured]` (Requisito software assegnato al software `DeviceManager`).
   * **Definizione:** *Il sensore fisico IoT non riceve o non applica correttamente a bordo i parametri operativi (come frequenza di campionamento e intervalli di tolleranza) impostati dal manager per la spedizione.*
   * **Categoria:** `Communication Failure`
   * **Risoluzione Software:** **Nessuna**. Questo rischio viene gestito ed eliminato in modo nativo a basso livello dai tentativi di ritrasmissione automatici propri del protocollo Bluetooth Low Energy (BLE) e dalle procedure di handshake del firmware di bordo.

### Ramo B: Tracciamento ambientale e batteria (transito)
Questo ramo tratta i rischi fisici ed elettronici legati al campionamento continuo dei parametri e alla sicurezza crittografica dei dati telemetrici durante il transito.

4. **Ostacolo 4:** `SensorPowerLoss`
   * **Goal Ostruito:** `Maintain [PhysicalParametersSampledBySensor]` (Aspettativa d'ambiente assegnata all'agente `PhysicalTransducer`).
   * **Definizione:** *Il sensore IoT smette improvvisamente di funzionare e di campionare i parametri fisici (temperatura, umidità, vibrazioni) a causa dell'esaurimento completo della batteria interna durante il viaggio.*
   * **Categoria:** `Hardware Failure`
   * **Risoluzione (Mitigazione Preventiva):** `Achieve [LowBatteryNotified]` (Requisito software assegnato all'agente `DeviceManager` nel Cloud).
   * **Meccanismo di Risoluzione:** Il Cloud intercetta costantemente i metadati relativi allo stato di carica trasmessi dal sensore in ogni pacchetto. Qualora la batteria scenda sotto la soglia critica del 15% (scelta progettuale), il sistema cloud genera immediatamente una notifica visiva o un alert sulla dashboard web del manager, consentendo una sostituzione preventiva del dispositivo prima dell'avvio o del transito della spedizione.

5. **Ostacolo 5:** `DataTamperedInTransit`
   * **Goal Ostruito:** `Avoid [UnauthorizedDataModification]` (Requisito software assegnato all'agente `ShipmentManager` a protezione dello storico a database).
   * **Definizione:** *Un utente malintenzionato o un attaccante esterno intercetta i pacchetti dati trasmessi via radio o accede abusivamente alle code di trasmissione per alterare i dati di temperatura e telemetria, tentando di nascondere sbalzi termici.*
   * **Categoria:** `Security Threat` / `Man-In-The-Middle`
   * **Risoluzione (Prevenzione Crittografica):** `Maintain [DataIntegrityVerified]` (Requisito software assegnato all'agente `IntegrityVerifier` nel Cloud).
   * **Meccanismo di Risoluzione:** Ogni pacchetto dati viene firmato crittograficamente alla sorgente dall'agente `SecureElement` del sensore usando la propria chiave privata. All'arrivo nel Cloud, l'agente `IntegrityVerifier` convalida matematicamente la firma asimmetrica tramite la chiave pubblica associata. Se i dati sono stati alterati in transito, la firma risulterà non valida: il Cloud scarterà immediatamente il pacchetto e genererà una segnalazione di violazione di integrità.

6. **Ostacolo 6:** `CryptographicSignatureFailure`
   * **Goal Ostruito:** `Maintain [DataDigitallySignedAtSource]` (Aspettativa d'ambiente assegnata all'agente `SecureElement`).
   * **Definizione:** *Il chip crittografico hardware gestito da SecureElement a bordo del sensore non riesce ad applicare la firma digitale asimmetrica ai dati per un guasto elettrico interno o un degrado hardware permanente.*
   * **Categoria:** `Hardware Failure`
   * **Risoluzione Software:** **Nessuna**. Trattandosi di un danno fisico irreversibile sul silicio del microcontrollore del sensore, la macchina software non ha alcuna possibilità di ripristinare il componente guasto. Il sistema cloud rileva l'anomalia (mancanza di firme conformi e fallimento sistematico del convalidatore), evidenzia lo stato di non conformità della merce e segnala la necessità di scartare il sensore guasto a fine viaggio.
   
12. **Ostacolo 12:**  UnrepresentativeSensorPlacement
    *   **Goal Ostruito:**  Maintain [PhysicalParametersSampledBySensor] (Aspettativa d'ambiente assegnata all'agente PhysicalTransducer).
    *   **Definizione:**   *Il sensore IoT viene posizionato accidentalmente in modo errato (es. all'esterno del contenitore refrigerato del prodotto di trasporto o in un punto del veicolo esposto a sbalzi termici non rappresentativi), registrando parametri fisici che non rispecchiano lo stato di reale conservazione della merce fragile.*
    *   **Categoria:**  Human Error/Procedural Error
    *   **Risoluzione (Mitigazione Passiva):**  Assunzione (Physical Placement Domain Assumption).
    *   **Meccanismo di Risoluzione:**  Trattandosi di un errore operativo sul campo da parte degli addetti alla logistica, il rischio non è risolvibile tramite logica software a runtime. La minaccia viene mitigata e gestita a livello procedurale tramite una **Domain Assumption (Physical Placement)**, che assume contrattualmente il rispetto di rigide linee guida di imballaggio (Standard Operating Procedures) che obbligano gli operatori a posizionare e ancorare stabilmente il sensore all'interno del collo prima dell'avvio della spedizione.

### Ramo C: Connessione e trasmissione (gateway)
Questo ramo analizza i rischi relativi alla trasmissione wireless a corto raggio (BLE) e all'inoltro WAN cellulare (4G/5G) verso la piattaforma SaaS.

7. **Ostacolo 7:** `BLEConnectionLost`
   * **Goal Ostruito:** `Maintain [BLEAutoConnectionAndDownload]` (Requisito software assegnato all'agente `AppGateway`).
   * **Definizione:** *La connessione Bluetooth Low Energy tra il sensore IoT all'interno del collo fragile e l'applicazione Gateway installata sullo smartphone dell'autista si interrompe improvvisamente durante il viaggio.*
   * **Categoria:** `Wireless Outage` / `Fault`
   * **Raffinamento dell'Ostacolo (Sotto-Ostacoli in OR):**
     * *Sotto-Ostacolo A:* `DriverTooFarFromCargo` (L'autista si allontana fisicamente dal vano di carico con lo smartphone oltre la portata massima di trasmissione del segnale BLE, pari a circa 10 metri).
     * *Sotto-Ostacolo B:* `PhoneBatteryDead` (Lo smartphone dell'autista si scarica completamente o viene spento, provocando l'arresto forzato dell'App Gateway mobile).
   * **Risoluzione (Mitigazione Forte):** `Maintain [DataBufferedLocallyIfNoBLE]` (Aspettativa d'ambiente assegnata all'agente `OnBoardFirmware`).
   * **Meccanismo di Risoluzione:** Nel caso in cui il collegamento Bluetooth Gateway-Sensore sia inattivo per una qualsiasi delle cause in OR, l'agente d'ambiente `OnBoardFirmware` memorizza temporaneamente tutti i campionamenti eseguiti nella flash memory locale del sensore. Non appena il segnale BLE viene ripristinato, l'App Gateway effettua la riconnessione automatica e scarica asincronamente l'intero buffer accumulato, impedendo qualsiasi perdita di dati di tracciamento.

8. **Ostacolo 8:** `GatewayUploadFailed`
   * **Goal Ostruito:** `Maintain [DataTransmissionSecureAndVerified]` (o genericamente `Achieve [AsynchronousDataUploadToCloud]` assegnato all'agente `AppGateway`).
   * **Definizione:** *L'applicazione mobile Gateway non riesce a trasmettere i dati telemetrici accumulati al backend Cloud SaaS a causa di interferenze radio temporanee o della totale assenza di connettività di rete cellulare (4G/5G o Wi-Fi) lungo la tratta di viaggio.*
   * **Categoria:** `Network Outage`
   * **Risoluzione (Mitigazione Intrinseca):** `Achieve [AsynchronousDataUploadToCloud]` (Requisito software assegnato all'agente `AppGateway`).
   * **Meccanismo di Risoluzione:** Il requisito stesso di caricamento asincrono funge da contromisura. L'App Gateway mobile memorizza i dati scaricati dal sensore in un buffer locale sicuro sullo smartphone dell'autista. Il software esegue tentativi di upload ciclici (retry logici) ad intervalli regolari: i dati vengono inoltrati al backend Cloud solo quando viene rilevato il ripristino di una connessione di rete stabile, tollerando lunghi periodi di blackout cellulare (es. gallerie, zone montane).

### Ramo D: Visualizzazione e integrazione (consegna)
Questo ramo copre i rischi di mancato accesso o visualizzazione errata dello storico dati da parte degli attori finali durante la verifica della spedizione.

9. **Ostacolo 9:** `QrCodeUnreadable`
   * **Goal Ostruito:** `Achieve [DashboardAccessibleToActors]` (Requisito software assegnato all'agente `DashboardManager`).
   * **Definizione:** *Il codice QR stampato sull'etichetta esterna del collo fragile o del prodotto è graffiato, bagnato, strappato o sporco, impedendo alla fotocamera dello smartphone del consumatore o del certificatore di effettuarne la scansione ottica per decodificare l'URL.*
   * **Categoria:** `Physical Damage` / `Usability Hazard`
   * **Risoluzione (Mitigazione Debole - Nuovo Requisito Software):** `Achieve [DashboardAccessibleViaCode]` (Requisito software assegnato all'agente `DashboardManager`).
   * **Meccanismo di Risoluzione:** Per aggirare l'illeggibilità fisica del QR Code, il sistema espone sulla pagina di benvenuto della dashboard web un campo form per l'inserimento manuale di un codice di backup. L'utente digita il codice alfanumerico identificativo univoco della spedizione, stampato in chiaro direttamente sotto il QR code sull'etichetta fisica, ottenendo immediato accesso alla visualizzazione dei dati certificati.

10. **Ostacolo 10:** `OutofRangeConditionsNotHighlighted`
    * **Goal Ostruito:** `Achieve [OutofRangeConditionsHighlighted]` (Requisito software assegnato all'agente `DashboardManager`).
    * **Definizione:** *Un bug software o un errore imprevisto nella logica di rendering della dashboard web impedisce la corretta evidenziazione grafica (es. marcatori rossi, grafici o alert) dei campionamenti fisici che hanno superato le soglie di temperatura consentite.*
    * **Categoria:** `Software Bug` / `Frontend Failure`
    * **Risoluzione Software:** **Nessuna**. Trattandosi di un puro difetto di implementazione logica del codice, questo rischio non si risolve aggiungendo un altro requisito di runtime, ma viene mitigato e prevenuto a monte tramite rigorose procedure di qualità del codice (es. unit testing delle funzioni di soglia e integration testing sul frontend grafico prima del rilascio in produzione).

11. **Ostacolo 11:** `APIAccessFailure`
    * **Goal Ostruito:** `Achieve [SecureDataExposedViaAPI]` (Requisito software assegnato all'agente `APIManager`).
    * **Definizione:** *I sistemi informatici esterni o i server degli enti di certificazione terzi non riescono a interrogare gli endpoint API del sistema a causa di errori temporanei del server o blackout della rete WAN.*
    * **Categoria:** `Interoperability Failure` / `API Connection Error`
    * **Risoluzione Software:** **Nessuna**. Il fallimento di connessione di rete dei sistemi esterni viene gestito in modo standard a livello architetturale e procedurale dalle policy di autenticazione e rate-limiting dell'API Gateway, e richiede l'intervento manuale dei sistemisti esterni per verificare la connettività di rete o rigenerare le API-key scadute.
