## Premessa
Per prima cosa separiamo rigidamente ciò che va modellato nel Goal Diagram da ciò che va trattato come assunzione, ipotesi di dominio o vincolo progettuale esterno. Per questa divisione, applichiamo i tre criteri dell'ing. dei requisiti:

1. Prescrittivo vs descrittivo: i goal (e quindi requisiti ed aspettative) sono prescrittivi (dettano come il sistema deve comportarsi). Le proprietà e ipotesi di dominio sono descrittive (fatti fisici o assunzioni sullo stato attuale del mondo, indipendenti dal software). Le prime vanno nel diagramma, le seconde no;

2. Soddisfazione netta: solo i requisiti con un criterio di soddisfacimento binario (sì/no) diventano nodi del Goal Diagram di Objectiver. Le qualità vaghe o soggettive (Soft Goals) si spostano nel testo o si usano solo per valutare alternative;
3. Confine del sistema: i vincoli fisici dell hardware preesistente (es. il fatto che il sensore funzioni a batteria) non sono funzioni software, ma vincoli ambientali stabili (ipotesi).

Per chiudere, i nomi utilizzati per Goal, Agenti, Assunzioni, HP di dominio e leggi di dominio DEVONO essere coerenti in tutti i testi del progetto, per tanto seguirà la creazione di un file apposito per tenere traccia dei nomi formali utilizzati da entrambi nei vari diagrammi/testi. I seguenti sono perciò da considerarsi provvisori.

## Requisiti da trasporre nei diagrammi
Questi requisiti descrivono comportamenti desiderati del sistema complessivo che possono essere assegnati a un agente del software (requisiti) o dell ambiente (aspettative).

### Ramo A: gestione spedizioni e dispositivi (configurazione)

Gestione dispositivi (devices):

1. Sotto-Goal 1: Achieve [DeviceProvisionedAndDecommissioned] (requisito assegnato al software cloud, i dispositivi vengono regolarmente registrato o dismessi dal sistema);
2. Sotto-Goal 2: Achieve [DeviceListed] (requisito assegnato al software cloud, i dispositivi vengono tenuti sotto traccia dal sistema);
3. Sotto-Goal 3: Achieve [DeviceAssociatedToShipment] (requisito assegnato al software cloud,, i dispositivi vengono associati/dissociati alle spedizioni);
4. Sotto-Goal 4: Achieve [DeviceConfigured] (requisito assegnato al software cloud, i dispositivi vengono configurati di frequenze di campionamento e metriche/grandezze).

Gestione spedizioni (shipments):

1. Sotto-Goal 5: Achieve [ShipmentLifeCycleManaged] (requisito assegnato al software cloud, vengono gestiti creazione, modifica, cancellazione logica);
2. Sotto-Goal 6: Achieve [ShipmentStateTransitionExecuted] (requisito assegnato al software cloud, vengono gestiti stati di start, stop, pause, resume).

### Ramo B: Tracciamento ambientale e batteria (transito)

Tracking ambientale continuo:

1. Sotto-Goal 7: Maintain [ContinuousTracking] (goal multi-agente ad alto livello);
2. Sotto-Goal 8: Maintain [PhysicalParametersSampledBySensor] (aspettativa assegnata all agente fisico sensore IoT per il corretto campionamento dei dati);
3. Sotto-Goal 9: Maintain [DataBufferedLocallyIfNoBLE] (aspettativa/requisito assegnato al firmware del sensore IoT, gestisce la memoria flash offline in assenza di connessione BLE).

Firma digitale alla sorgente:
1. Sotto-Goal 10: Maintain [DataDigitallySignedAtSource] (aspettativa assegnata all agente sensore IoT: il chip crittografico deve firmare i dati appena campionati);
1.1 Sotto-Goal del Goal 10: Maintain [DataIntegrityVerified] (requisito assegnato all'IntegrityVerifier (sotto-agente software della macchina, incaricato specificamente di questa funzione di quality assurance), ogni pacchetto di dati ambientali ricevuto dal cloud deve essere sottoposto a verifica matematica della firma crittografica per confermare che non sia stato alterato o corrotto prima della memorizzazione definitiva;
2. Sotto-Goal 11: Avoid [UnauthorizedDataModification] (requisito assegnato al software cloud: impedisce la modifica dei dati storici a magazzinieri, autisti o manager);

Monitoraggio della batteria:
1. Sotto-Goal 12: Maintain [BatteryStatusMonitored] (goal multi-agente);
2. Sotto-Goal 13: Maintain [BatteryLevelTransmittedBySensor] (aspettativa assegnata al sensore IoT);
3. Sotto-Goal 14: Achieve [LowBatteryNotified] (requisito assegnato al software cloud, invio notifiche manutenzione per batteria sotto il 15%).

(NB: il 15% è a tutti gli effetti una scelta progettuale.)

### Ramo C: Connessione e trasmissione (gateway)

Sincronizzazione automatica:
1. Sotto-Goal 15: Achieve [BLEAutoConnectionAndDownload] (requisito assegnato al software gateway app, applicazione mobile Gateway deve tentare automaticamente la connessione Bluetooth Low Energy con il sensore non appena si trova entro il raggio di copertura radio (<10m) e scaricare i dati accumulati nella memoria flash);
2. Sotto-Goal 16: Achieve [AsynchronousDataUploadToCloud] (requisito assegnato al software gateway app, deve conservare localmente i dati scaricati dal sensore e inoltrarli in modo asincrono al database Cloud non appena rileva la presenza di connettività internet (Wi-Fi o rete cellulare 4G/5G).

### Ramo D: Visualizzazione e integrazione (consegna)

Dashboard e allarmi:
1. Sotto-Goal 17: Achieve [OutofRangeConditionsHighlighted] (requisito assegnato al software cloud, evidenziazione grafica dei parametri fuori soglia nella dashboard);
2. Sotto-Goal 18: Achieve [DashboardAccessibleToActors] (requisito assegnato al software cloud, accesso continuo profilato per manager, autisti, consumatori e enti di certificazione).

Interoperabilità via API:
1. Sotto-Goal 19: Achieve [SecureDataExposedViaAPI] (requisito assegnato al software cloud, esposizione delle API per lo scambio sicuro dei dati con enti terzi).

## Requisiti che si spostano come assunzioni (no diagrammi principali)
Questi punti non sono obiettivi di comportamento software (goal), ma vincoli fisici dell ambiente, decisioni tecnologiche a priori o procedure organizzative esterne. Inserirli nel Goal Diagram lo renderebbe ridondante e confusionario. Vengono invece inseriti nell IEEE-830 e usati come ipotesi di dominio (HP) per validare i requisiti. La lista è:

1. dispositivi low-cost e riusabili: l'essere economico è un soft goal qualitativo di sviluppo non formalizzabile matematicamente. La riutilizzabilità fisica è garantita a livello logico dal software tramite i requisiti di associazione/dissociazione (Goal 3). La spostiamo nelle assunzioni;

2. alimentazione e connettività dei sensori: il fatto che il sensore IoT funzioni solo a batteria e non abbia un modulo SIM (no 4G/5G) è una proprietà fisica dell ambiente reale (DOMINIO/HP), non un obiettivo del codice. È un vincolo tecnologico che scriviamo nel capitolo 2 dell IEEE-830;

Architettura SaaS cloud: è una scelta tecnologica ed architetturale pregressa. Non è un requisito utente erogato dal sistema, ma un vincolo sullo sviluppo. Va documentato nella sezione dei constraints dell IEEE-830;

Sensori di emergenza (guasto sensori): questa è una mitigazione organizzativa per la sicurezza (fault tolerance). Non fa parte dei flussi operativi standard del software. Questa assunzione non va nel Goal Diagram principale, ma la useremo nell Obstacle Resolution Diagram per risolvere l'ostacolo guasto del sensore in viaggio.

## Struttura diagramma disegnata
               [Maintain: FragileGoodsIntegrityPreserved]
                               (Radice)
                        /              |               \
                     AND              AND              AND
                      /                |                 \
                [Macro-Goal 1]        [Macro-Goal 2]    [Macro-Goal 3]                     
                (Configurazione)      (Tracciamento)    (Distribuzione)
                   /                   |                     \
          (Sotto-goal 1-6)         (Sotto-goal 7-16)         (Sotto-goal 17-19)
          

Questi nodi si posizionano esattamente sotto il Goal Radice (Maintain [FragileGoodsIntegrityPreserved]) tramite un raffinamento AND e raggruppano logicamente i sotto-goal foglia che abbiamo già definito. Abbiamo quindi:

1. Macro-Goal intermedio 1: configurazione. Questo nodo unifica tutta la fase iniziale di inserimento dati e accoppiamento dell'hardware prima della partenza.

    Nome Formale: Achieve [ShipmentAndDevicesReady]
    
    Definizione: la spedizione deve essere creata correttamente a sistema, con i relativi profili di soglia configurati e i sensori fisici attivati e associati prima che inizi il trasporto.
    
    Raffinamento: si scompone in AND nei seguenti sotto-goal foglia:
    
        Achieve [DeviceProvisionedAndDecommissioned] (Sotto-Goal 1)
        Achieve [DeviceListed] (Sotto-Goal 2)
        Achieve [DeviceAssociatedToShipment] (Sotto-Goal 3)
        Achieve [DeviceConfigured] (Sotto-Goal 4)
        Achieve [ShipmentLifeCycleManaged] (Sotto-Goal 5)
        Achieve [ShipmentStateTransitionExecuted] (Sotto-Goal 6)

2. Macro-Goal intermedio 2: tracciamento e trasmissione sicura
Questo è il cuore operativo del sistema in viaggio. Garantisce che i dati vengano raccolti, firmati alla sorgente, protetti in caso di disconnessione e inviati al cloud via gateway.

    Nome Formale: Maintain [ContinuousTrackingAndSecureDataTransmission]
    
    Definizione: i parametri ambientali e lo stato della batteria devono essere campionati con continuità, firmati digitalmente per garantirne l'integrità e trasmessi in modo asincrono al cloud tramite l'applicazione Gateway.
    
    Raffinamento: si scompone in AND nei seguenti sotto-goal foglia:
    
        Maintain [ContinuousTracking] (Sotto-Goal 7)
        Maintain [PhysicalParametersSampledBySensor] (Sotto-Goal 8)
        Maintain [DataBufferedLocallyIfNoBLE] (Sotto-Goal 9)
        Maintain [DataDigitallySignedAtSource] (Sotto-Goal 10)
        Avoid [UnauthorizedDataModification] (Sotto-Goal 11)
        Maintain [DataIntegrityVerified] (Sotto-Goal 11.1)
        Maintain [BatteryStatusMonitored] (Sotto-Goal 12)
        Maintain [BatteryLevelTransmittedBySensor] (Sotto-Goal 13)
        Achieve [LowBatteryNotified] (Sotto-Goal 14)
        Achieve [BLEAutoConnectionAndDownload] (Sotto-Goal 15)
        Achieve [AsynchronousDataUploadToCloud] (Sotto-Goal 16)

3. Macro-Goal intermedio 3: accessibilità e certificazione dati
Questo nodo rappresenta l'erogazione del servizio verso l'esterno, garantendo che i dati siano consultabili tramite dashboard e API, evidenziando eventuali anomalie.

    Nome Formale: Achieve [DataExposedAndCertified]
    
    Definizione: i dati storici consolidati devono essere messi a disposizione in modo profilato agli attori interessati tramite dashboard web (con evidenza degli allarmi) e tramite interfacce API sicure per gli enti terzi e i consumatori;
    
    Raffinamento: si scompone in AND nei seguenti sotto-goal foglia:
    
        Achieve [OutofRangeConditionsHighlighted] (Sotto-Goal 17)
        Achieve [DashboardAccessibleToActors] (Sotto-Goal 18)
        Achieve [SecureDataExposedViaAPI] (Sotto-Goal 19)