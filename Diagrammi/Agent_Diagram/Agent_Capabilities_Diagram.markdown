Per garantire la massima coerenza inter-vista, tutti i collegamenti di monitoraggio e controllo fanno riferimento esclusivamente alle entità e alle variabili di stato definite nel Class Diagram.

| Agente | Tipo | Entità Class Diagram | Tipo Relazione | Attributo / Associazione Specifico | Significato Operativo e Logica dei Requisiti |
| :--- | :--- | :--- | :---: | :--- | :--- |
| **`DeviceManager`** | Software | `IoTDevice` | **`CONTROL`** | Istanza di `IoTDevice` | Registra e attiva il dispositivo hardware nel sistema (onboarding). |
| | | `IoTDevice` | **`MONITOR`** | `batteryLevel`, `bleConnectionStatus` | Rileva lo stato di carica della batteria e la presenza del sensore a backend. |
| **`ShipmentManager`** | Software | `Shipment` | **`MONITOR`** | `logicalState` | Ispeziona lo stato corrente del viaggio (nessun controllo diretto per via della Domain Assumption di creazione). |
| | | `ConfigurationProfile`| **`CONTROL`** | Istanza di `ConfigurationProfile`, `samplingInterval` | Crea e configura il profilo di viaggio associato alla spedizione. |
| | | `ThresholdRule` | **`CONTROL`** | `metricName`, `minValue`, `maxValue` | Imposta e definisce le soglie di tolleranza fisiche impartite dal manager. |
| **`AppGateway`** | Software | `ShippingLabel` | **`MONITOR`** | `qrCodePayload`, `backupAlphanumericCode` | Esegue la scansione dell'etichetta fisica per identificare il viaggio (nessun controllo sul QR che è statico). |
| | | `LocalGatewayBuffer` | **`CONTROL`** | `pendingPacketsCount`, `isCloudSynchronized` | Gestisce l'accodamento locale dei dati sul telefono e la loro trasmissione asincrona. |
| | | `IoTDevice` | **`MONITOR`** | `bleConnectionStatus` | Rileva lo stato di connessione BLE attiva con il dispositivo sensore. |
| **`PhysicalTransducer`**| Ambiente | `EnvironmentalMeasurement`| **`CONTROL`** | `measuredValue`, `timestamp`, `metricName` | Rileva la grandezza fisica (temperatura, umidità, ecc.) sul campo e genera la misura. |
| **`OnBoardFirmware`** | Software | `IoTDevice` | **`CONTROL`** | `batteryLevel`, `availableFlashMemory`, `bleConnectionStatus` | Aggiorna lo stato di salute interna dell'hardware (diagnostica di bordo). |
| | | `ConfigurationProfile`| **`MONITOR`** | `samplingInterval` | Legge l'intervallo temporale configurato per determinare la frequenza di polling. |
| | | `ThresholdRule` | **`MONITOR`** | `metricName`, `minValue`, `maxValue` | Legge i limiti di soglia impostati per poter eseguire il confronto con i dati campionati. |
| | | `EnvironmentalMeasurement`| **`CONTROL`** | `isOutOfRange` | Esegue il confronto locale e imposta il flag se la misura evade i limiti. |
| **`SecureElement`** | Ambiente | `CryptographicSignature`| **`CONTROL`** | `signatureValue`, `timestamp` | Genera la firma crittografica inalterabile sui dati. |
| | | `EnvironmentalMeasurement`| **`MONITOR`** | `measuredValue` | Legge il dato grezzo campionato prima di firmarlo. |
| **`IntegrityVerifier`**| Software | `CryptographicSignature`| **`CONTROL`** | `integrityStatus` | Valuta l'attendibilità del dato e scrive l'esito della verifica (`VALID`/`TAMPERED`). |
| | | `CryptographicSignature`| **`MONITOR`** | `signatureValue` | Ispeziona la firma crittografica allegata ai pacchetti in transito nel Cloud. |
| | | `IoTDevice` | **`MONITOR`** | `publicKey` | Recupera la chiave pubblica memorizzata del sensore per decifrare l'hash. |
| **`DashboardManager`**| Software | `AlarmNotification` | **`CONTROL`** | `alarmType`, `creationTimestamp`, `isDismissed` | Genera l'allarme visivo a schermo e permette la presa visione (`isDismissed`). |
| | | `EnvironmentalMeasurement`| **`MONITOR`** | `measuredValue`, `isOutOfRange` | Legge lo storico delle misure e delle anomalie per renderizzarle in tempo reale. |
| **`APIManager`** | Software | `Shipment` | **`MONITOR`** | `shipmentID`, `logicalState` | Consente ai sistemi gestionali esterni (ERP) di interrogare lo stato. |
| | | `EnvironmentalMeasurement`| **`MONITOR`** | `measuredValue`, `timestamp` | Esporta lo storico dei dati certificati e conformi verso le API integrate. |

---






---
NB manca il collegamento OnBoardFirmwware -> threshold che deve monitorare i dati dei range impostati per poter eseguire il confronto con i dati campionati e eventualmente impostare le outOfRange. Eliminato control da parte di shipmentmanager su shipment entity. Manca anche controllo da parte di ShipmentManager rispetto a threshold (imposta le soglie impartite dal manager).

In sostanza, devicemanager monitora la batteria. IntegrityVerifier monitora lo stato di sicurezza e crittografia dei pacchetti/dati. DashboardManager monitora la flag isOutOfRange per potenziali fuori soglia. Questo in questo preciso diagramma. Poi però, è SOLO DashboardManager che può CONTROLLARE le variabili di allarme, perciò in uno dei diagrammi successivi collegheremo in qualche modo gli altri due controlli degli agenti al DashboardManager che quindi li gestisce tutti diciamo.





Note:
1. ha senso che shipmentManager monitori e controlli la stessa variabile dello stato logico della spedizione? Così a naso me par e no, se proprio bisogna rappresentare la creazione della consegna da parte di un cliente bisogna creare una terza variabile.
2. sul Qrcode ell'esecuzione che fa AppGateway, ha senso? Qrcode non dovrebbe essere (cosi come il codice alfanumerico) un qualcosa di statico (per il prodotto) per cui a parte crearlo (e non abbiamo un agente che lo fa) non viene fatto altro? La visualizzazione se proprio dovrebbe essere fatta dal customer che non abbiamo messo nel sistema.
3. l'integrity verifier monitora e controlla il crittografore; non so...è corretto? nei goal non lo avevamo messo per controllare i pacchetti lato cloud?
4. dashboard manager non dovrebbe monitorare anche batteria e integrità pacchetti per creare l'allarme? O facciamo fare i monitoraggi ad altri agenti come abbiamo fatto e poi nei prossimi diagrammi sugli agenti li mettiamo in relazione?


Risoluzione punto 1: controllare che sia presente la seguente assunzione:
"Assunzione: La spedizione viene inizializzata e inserita a database da sistemi gestionali terzi (fuori dal confine del nostro sistema). Il ShipmentManager (il nostro backend) ne rileva l'esistenza nello stato CREATED.".

Risoluzione punto 2: ho tolto il control da parte di appgateway sui codici di visualizzazione, deve solo vederli non modificarli;

Risoluzione punto 3: va bene così perchè IntegrityVerifier fa già parte del controllo lato cloud.
