## Diagramma di Sequenza 1: StartShipmentSequenceDiagram

@startuml
actor "Manager" as Man
actor "Operatore Magazzino" as Op
participant "App Gateway" as App
participant "Sensore IoT" as Sensor
participant "SaaS Cloud" as Cloud

autonumber

== Inizializzazione Spedizione ==
Man -> Cloud : Crea spedizione (dati carico, profili soglia)
activate Cloud
Cloud -> Cloud : Registra viaggio\n[Stato: CREATED]
Cloud --> Man : Spedizione creata (shipmentID)
Cloud --> Op : Notifica spedizione da preparare
deactivate Cloud

== Associazione Fisica e Configurazione BLE ==
Op -> App : Scansiona etichetta o QR-Code Sensore
activate App
App -> Cloud : Associa Sensore a Spedizione (shipmentID, hardwareID)
activate Cloud
Cloud --> App : Conferma associazione e invia profilo soglie
deactivate Cloud

App -> Sensor : Connessione BLE & Invia configurazione (soglie, intervallo)
activate Sensor
Sensor -> Sensor : Salva profilo in memoria
Sensor --> App : Configurazione completata e sensore pronto
deactivate Sensor
deactivate App

Op -> Op : Posiziona il sensore nel pacco e sigilla il carico
@enduml


## Diagramma di Sequenza 2: TrackingDataSequenceDiagram


@startuml
autonumber
actor "Autista (Driver)" as Driver
box "Mobile Gateway" #LightCyan
    participant "App Gateway\n(Smartphone)" as App
end box
box "Hardware IoT" #LightYellow
    participant "IoT Device\n(OnBoardFirmware)" as Device
    participant "Secure Element\n(Crypto Chip)" as SE
    participant "Physical Transducer\n(Sensori)" as Trans
end box
box "Backend Cloud SaaS" #LightBlue
    participant "ShipmentManager\n(Gestione Spedizioni)" as SM
end box

== Fase 1: Avvio Fisico del Viaggio ==
Driver -> App : Seleziona "Avvia Spedizione" (StartShipment)
activate App
App -> SM : Comunica Avvio Viaggio (shipmentID)
activate SM
SM -> SM : Aggiorna Shipment\n(logicalState = IN_TRANSIT)
SM --> App : Stato Aggiornato
deactivate SM
App --> Driver : Interfaccia in modalità di Guida / Tracciamento attivo
deactivate App

== Fase 2: Ciclo di Campionamento e Firma Hardware (Ininterrotto) ==
loop Ogni 'samplingInterval' (es. 15 minuti)
    Device -> Trans : Richiesta Misura Ambientale (SamplePhysicalParameter)
    activate Trans
    Trans --> Device : measuredValue, TimeStamp
    deactivate Trans
    
    Device -> SE : Richiesta Firma Crittografica (SignMeasurement)
    activate SE
    note over SE : Calcola SHA256(measuredValue + TimeStamp)\ne firma con Chiave Privata Device
    SE --> Device : CryptographicSignature
    deactivate SE
    
    Device -> Device : Scrittura in Memoria Locale (BufferMemory)
end

== Fase 3: Sincronizzazione BLE ==
loop Rilevazione BLE Prossimità
    App -> Device : Rilevamento Presenza BLE
    activate App
    activate Device
    App -> Device : Richiesta Scaricamento Dati (DownloadSensorDataViaBLE)
    Device --> App : Invio Misure + Firme Crittografiche accumulate
    Device -> Device : Svuota Memoria Interna (BufferMemory = 0)
    deactivate Device
    
    App -> App : Scrittura in LocalGatewayBuffer (pendingPacketCount++)
    
    ref over App : Sincronizzazione Cloud e Verifica Integrità
    deactivate App
end
@enduml


Sotto-diagramma referenziato:

@startuml
autonumber
box "Mobile Gateway" #LightCyan
    participant "App Gateway\n(Smartphone)" as App
end box
box "Backend Cloud SaaS" #LightBlue
    participant "IntegrityVerifier" as IV
    participant "DashboardManager\n(Visualizzazione Dati)" as DM
end box

== Sincronizzazione Cloud e Verifica Integrità (UploadDataToCloud) ==

alt Caso A: Connettività Internet Cloud Presente (4G/5G)
    activate App
    App -> IV : Caricamento Pacchetti Pendenti (UploadDataToCloud)
    activate IV
    loop Per ogni Misura nel Pacchetto
        IV -> IV : Verifica Chiave Pubblica & Decrittografia (VerifyDataIntegrity)
        alt Firma Matematica Valida
            IV -> IV : Salva Misura (integrityStatus = VALID)
        else Firma non Corrispondente (Man-In-The-Middle / Dati Alterati)
            IV -> IV : Salva Misura (integrityStatus = INVALID)
            IV -> DM : Trigger Allarme (GenerateIntegrityViolationAlarm)
            activate DM
            DM -> DM : Istanzia AlarmNotification (Alarm = INTEGRITY_VIOLATION)
            deactivate DM
        end
    end
    IV --> App : Risposta di Avvenuta Sincronizzazione Cloud
    deactivate IV
    App -> App : Allinea Buffer Locale (pendingPacketCount = 0, isCloudSynchronized = True)
    deactivate App
    
else Caso B: Connettività Internet Assente (Galleria / Zone Remote)
    note over App : Conserva i dati nel buffer dello smartphone\n(isCloudSynchronized = False)
end
@enduml



## Diagramma di Sequenza 3: showDataSequenceDiagram



@startuml
autonumber
actor "Consumatore/Ente di Certificazione" as Customer
participant "Shipping Label\n(Supporto Cartaceo)" as Label
box "Backend Cloud SaaS" #LightBlue
    participant "APIManager" as AM
end box

== Fase 1: Scansione Ottica di Fallback ==
Customer -> Label : Inquadra QR Code (QrCodeURL) o Legge Codice Alfanumerico
activate Label
Label --> AM : Reindirizzamento al Portale Web pubblico
deactivate Label

== Fase 2: Interrogazione Portale Web ==
Customer -> AM : Richiesta Storico Spedizione tramite Codice (AccessDashboardViaAlphanumericCode)
activate AM
AM -> AM : Verifica esistenza AlphanumericCode associato a Shipment
AM -> AM : Recupera Spedizione, Misure e Firme associate

== Fase 3: Visualizzazione Dati e Allarmi Dashboard ==
AM -> AM : Recupero Storico Allarmi
Note over AM : Evidenzia con Marker Grafici le misure in 'isOutOfRange = True'\ne visualizza l'esito della validazione delle Firme ('VALID' / 'INVALID')
AM --> Customer : Fornisce i Dati di Telemetria Storica
AM --> Customer : Mostra Dashboard Allarmi
deactivate AM
@enduml
