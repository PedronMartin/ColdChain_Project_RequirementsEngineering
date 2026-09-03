Per garantire la massima coerenza inter-vista, tutti i collegamenti di monitoraggio e controllo fanno riferimento esclusivamente alle entità e alle variabili di stato definite nel Class Diagram.

| Agente | Tipo | Entità Class Diagram | Tipo Relazione | Attributo / Associazione Specifico | Significato Operativo e Logica dei Requisiti |
| :--- | :--- | :--- | :---: | :--- | :--- |
| **`DeviceManager`** | Software | `IoTDevice` | **`CONTROL`** | Istanza di `IoTDevice` | Registra e attiva il dispositivo hardware nel sistema (onboarding). |
| | | `IoTDevice` | **`MONITOR`** | `batteryLevel`, `BLEConnectionStatus` | Rileva lo stato di carica della batteria e la presenza del sensore a backend (per listing). |
| **`ShipmentManager`** | Software | `Shipment` | **`MONITOR/CONTROL`** | `logicalState` | Ispeziona lo stato corrente del viaggio (nessun controllo diretto sull'avvio per via della Domain Assumption di creazione ma può modificarne lo stato creazione -> in transito -> ecc.ecc.). Può sembrare un controsenso ma nella teoria delle macchine a stati, per cambiare uno stato deterministico bisogna sapere lo stato precedente. Crea a sistema la spedizione impostandola a CREATED (shipment instance); successivamente può mostrarla (anche per visionare lo stato corrente) e cancellarla (CANCELLED). |
| | | `ConfigurationProfile`| **`CONTROL`** | Istanza di `ConfigurationProfile`, `samplingInterval` | Crea e configura il profilo di viaggio associato alla spedizione. |
| | | `ThresholdRule` | **`CONTROL`** | `metricName`, `minValue`, `maxValue` | Imposta e definisce le soglie di tolleranza fisiche impartite dal manager. |
| **`AppGateway`** | Software | `LocalGatewayBuffer` | **`CONTROL/MONITOR`** | `pendingPacketCount`, `isCloudSynchronized` | Gestisce l'accodamento locale dei dati sul telefono e la loro trasmissione asincrona (neccesita monitoraggio). |
| | | `IoTDevice` | **`MONITOR`** | `BLEConnectionStatus` | Rileva lo stato di connessione BLE attiva con il dispositivo sensore. |
| | | Shipment | **`CONTROL`** | logicalState, `tracked_by` instance (association with IoTDevice) | Crea/Distrugge l'associazione spedizione-IoTDevice alla partenza/termine del viaggio e modifica lo stato logico di spedizione tra avvia (CREATED -> IN_TRANSIT), pause (IN_TRANSIT -> PAUSED), resume (PAUSED -> IN_TRANSIT), complete (IN_TRANSIT -> COMPLETED). |
| **`PhysicalTransducer`**| Ambiente | `EnvironmentalMeasurement`| **`CONTROL`** | `measuredValue`, `TimeStamp`, `metricName` | Rileva la grandezza fisica (temperatura, umidità, ecc.) sul campo e genera la misura. |
| **`OnBoardFirmware`** | Software | `IoTDevice` | **`CONTROL`** | `batteryLevel`, `BufferMemory`, `BLEConnectionStatus` | Aggiorna lo stato di salute interna dell'hardware (diagnostica di bordo). |
| | | `ConfigurationProfile`| **`MONITOR`** | `samplingInterval` | Legge l'intervallo temporale configurato per determinare la frequenza di polling. |
| | | `ThresholdRule` | **`MONITOR`** | `metricName`, `minValue`, `maxValue` | Legge i limiti di soglia impostati per poter eseguire il confronto con i dati campionati. |
| | | `EnvironmentalMeasurement`| **`CONTROL`** | `isOutOfRange` | Esegue il confronto locale e imposta il flag se la misura evade i limiti. |
| **`SecureElement`** | Ambiente | `CryptographicSignature`| **`CONTROL`** | `signatureValue` | Genera la firma crittografica inalterabile sui dati. |
| | | `EnvironmentalMeasurement`| **`MONITOR`** | `measuredValue`, `TimeStamp` | Legge il dato grezzo campionato e il suo timestamp prima di firmarlo. |
| **`IntegrityVerifier`**| Software | `CryptographicSignature`| **`CONTROL`** | `integrityStatus` | Valuta l'attendibilità del dato e scrive l'esito della verifica (`VALID`/`TAMPERED`). |
| | | `CryptographicSignature`| **`MONITOR`** | `signatureValue` | Ispeziona la firma crittografica allegata ai pacchetti in transito nel Cloud. |
| | | `IoTDevice` | **`MONITOR`** | `publicKey` | Recupera la chiave pubblica memorizzata del sensore per decifrare l'hash. |
| **`DashboardManager`**| Software | `AlarmNotification` | **`CONTROL`** | `alarmType`, `Timestamp` | Genera l'allarme visivo a schermo e permette la presa visione. |
| | | `EnvironmentalMeasurement`| **`MONITOR`** | `measuredValue`, `isOutOfRange` | Legge lo storico delle misure e delle anomalie per renderizzarle in tempo reale. |
| | | CryptographicSignature | **`MONITOR`** | `integrityStatus` | Legge l'andamento del reparto sicurezza dell'applicativo, in modo da segnalare eventuali manomissioni o compromissioni della cifratura. |
| | | IoTDevice | **`MONITOR`** | batteryLevel | Monitora lo stato di carica residua dei sensori associati alle spedizioni attive per innescare l'allarme visivo. |
| **`APIManager`** | Software | `Shipment` | **`MONITOR`** | `shipmentID`, `logicalState` | Consente ai sistemi gestionali esterni (ERP) di interrogare lo stato. |
| | | `EnvironmentalMeasurement`| **`MONITOR`** | `measuredValue`, `TimeStamp` | Esporta lo storico dei dati certificati e conformi verso le API integrate. |
| | | ShipmentLabel |  **`MONITOR`**  | isQrCodeReadable, QrCodeURL, AlphanumericCode | Confronta il codice ricevuto dalle richieste degli utenti in DashboardManager confrontando i codici (alphanumeric o QrCode) per accedere alla corretta porzione di dati richiesta |

---