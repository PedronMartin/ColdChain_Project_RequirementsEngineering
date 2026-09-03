# REPORT DI PROGETTO: VISTA STRUTTURALE (DIAGRAMMA DELLE CLASSI CONCETTUALI)

## 1. Introduzione Metodologica alla Vista Strutturale

La **Vista Strutturale** del progetto "V2-ProgettoING" è modellata attraverso un **Diagramma delle Classi UML** declinato secondo i canoni metodologici della **Goal-Oriented Requirements Engineering (GORE)** [10.1]. 

A differenza di un tradizionale diagramma delle classi utilizzato nella progettazione del software (Object-Oriented Design), questo modello rappresenta esclusivamente gli **oggetti concettuali del mondo reale (dominio d'applicazione)** e le loro relazioni, descrivendo il cosiddetto **sistema "To-Be"** (il mondo reale modificato a seguito dell'introduzione del software) [1, 10.1]. 

Per questa ragione, si applicano rigorosamente le seguenti euristiche metodologiche:
*   **Assenza di Operazioni/Metodi**: Le classi concettuali rappresentano entità passive o attive dell'ambiente e non contengono alcuna segnatura di metodi o funzioni software [10.1.2]. Esse modellano proprietà strutturali e stati logici/fisici del dominio, non implementazioni software.
*   **Focus sui Fenomeni Condivisi ed Ambientali**: Il modello descrive sia grandezze fisiche dell'ambiente (es. i valori misurati dai sensori) sia costrutti informativi condivisi tra l'ambiente e la macchina software (es. lo stato di una spedizione o il buffer locale dell'applicazione gateway) [12.2].
*   **Annotazioni Semantiche (Definizioni)**: Ogni classe, associazione ed enumerazione è corredata da note semantiche esplicite per eliminare qualsiasi ambiguità interpretativa e garantire una definizione formale univoca [10.4.1, 10.9].

---

## 2. Dizionario delle Classi Concettuali (Entità e Associazioni)

Il diagramma si compone di **9 classi concettuali** principali, classificate tra entità passive, agenti strutturali ed associazioni, corredate dai rispettivi attributi tipizzati.

### 2.1. Shipment (Entità Concettuale)
Rappresenta l'astrazione logica del viaggio e del trasporto sicuro di un lotto di merci fragili o sensibili (es. farmaci, opere d'arte) dal mittente al destinatario [60, 61].
*   **`shipmentID : String`**: Identificativo alfanumerico univoco della spedizione nel sistema Cloud SaaS.
*   **`logicalState : ShipmentState`**: Stato logico-temporale del ciclo di vita del trasporto. È un'enumerazione composta da:
    *   `CREATED`: Spedizione creata a livello amministrativo, sensori non ancora attivi o in fase di associazione.
    *   `IN_TRANSIT`: Merci in viaggio, sensori attivi che registrano e trasmettono dati.
    *   `PAUSED`: Sosta logistica o ispezione programmata; la registrazione può essere temporaneamente sospesa o marcata come sosta.
    *   `CANCELLED`: Spedizione annullata o viaggio non terminato con successo.
    *   `COMPLETED`: Viaggio terminato con successo, merci consegnate, tracciamento concluso.

### 2.2. IoTDevice (Agente Strutturale/Fisico)
Modella il dispositivo hardware sensore low-cost posizionato fisicamente all'interno o in prossimità del collo di spedizione [62].
*   **`hardwareID : String`**: Identificatore fisico univoco del microcontrollore (es. MAC Address o ID cablato nel silicio).
*   **`BLEConnectionStatus : Boolean`**: Stato della connessione Bluetooth Low Energy con l'applicazione gateway dell'autista (`True` se connesso, `False` altrimenti) [62, 68].
*   **`batteryLevel : Integer`**: Percentuale di carica residua della batteria del sensore (fondamentale per il monitoraggio della manutenzione, es. allarme se < 15%) [62, 69].
*   **`publicKey : String`**: Chiave crittografica pubblica utilizzata dal Cloud per verificare la firma digitale dei pacchetti dati generati dal sensore [60, 66].
*   **`BufferMemory : String`**: Stato o capacità della memoria flash non volatile a bordo del sensore, utilizzata per bufferizzare temporaneamente i dati in assenza di connessione BLE [62].

### 2.3. EnvironmentalMeasurement (Entità Concettuale)
Rappresenta il singolo record ambientale campionato dai sensori del dispositivo IoT in un preciso istante temporale [61].
*   **`metricName : String`**: Tipo di grandezza fisica campionata (es. `TEMPERATURE`, `HUMIDITY`, `VIBRATION`, `UV_RADIATION`).
*   **`TimeStamp : Date`**: Marca temporale esatta in cui il trasduttore fisico ha effettuato il campionamento.
*   **`measuredValue : Integer`**: Valore intero digitalizzato della misura (espressa in unità standard o scalata secondo il datasheet del sensore).
*   **`isOutOfRange : Boolean`**: Indicatore booleano che segnala se il valore misurato viola le soglie di sicurezza definite per quella specifica spedizione.

### 2.4. CryptographicSignature (Entità Concettuale)
Garantisce la non-ripudiabilità e l'immutabilità del dato alla sorgente. È legata con una relazione di composizione 1-to-1 a ciascuna misurazione [60, 66].
*   **`signatureValue : String`**: Stringa esadecimale o Base64 contenente la firma crittografica generata dal Secure Element del dispositivo IoT applicando la chiave privata sui dati della misura (`metricName + TimeStamp + measuredValue`).
*   **`integrityStatus : ValidationState`**: Stato di validità della firma calcolato dal Cloud. È un'enumerazione composta da:
    *   `VALID`: La firma corrisponde matematicamente alla chiave pubblica del dispositivo; il dato è integro.
    *   `INVALID`: La firma non corrisponde; il dato è stato alterato (manomesso) durante la trasmissione o nel database [60, 66].

### 2.5. LocalGatewayBuffer (Entità Informativa dell'Ambiente)
Rappresenta lo stato del buffer di sincronizzazione temporaneo presente sull'applicazione mobile del conducente (che funge da gateway BLE-Internet) [62, 68].
*   **`pendingPacketCount : Integer`**: Numero di misurazioni scaricate dal sensore via BLE ma non ancora caricate sul Cloud a causa di assenza temporanea di connettività internet (es. galleria, zone remote) [62].
*   **`pendingPacket : String`**: Payload serializzato contenente i pacchetti dati crittografati in attesa di upload.
*   **`isCloudSynchronized : Boolean`**: Stato di allineamento del buffer locale con il database Cloud SaaS (`True` se tutti i dati locali sono stati sincronizzati, `False` se vi sono dati pendenti).

### 2.6. ShippingLabel (Entità Concettuale)
Modella l'etichetta cartacea applicata sul pacco, contenente i riferimenti visuali e digitali per l'ispezione rapida.
*   **`AlphanumericCode : String`**: Codice leggibile dall'occhio umano per l'identificazione manuale.
*   **`QrCodeURL : String`**: URL codificato nel QR Code che punta direttamente alla dashboard pubblica di quella spedizione, consentendo l'ispezione immediata a consumatori ed autorità [61, 68].
*   **`isQrCodeReadable : Boolean`**: Stato di usura fisica dell'etichetta (modella l'integrità del canale visivo).

### 2.7. AlarmNotification (Entità Concettuale)
Rappresenta le notifiche asincrone di anomalia generate dal sistema e mostrate sulla Live-Dashboard [61, 68].
*   **`Timestamp : Date`**: Ora e data di rilevamento dell'anomalia.
*   **`Alarm : AlarmType`**: Tipologia di allarme riscontrato. È un'enumerazione composta da:
    *   `LOW_BATTERY`: Batteria del sensore scesa sotto la soglia critica del 15% [62, 69].
    *   `INTEGRITY_VIOLATION`: Rilevata firma crittografica non valida lato Cloud (tentativo di tampering dati) [60, 66].
    *   `OUT_OF_RANGE`: Rilevamento di un parametro ambientale che supera le soglie del profilo di spedizione [61, 68].

### 2.8. ConfigurationProfile (Entità Concettuale)
Rappresenta il profilo di configurazione logica applicabile a un sensore in base alla natura della merce trasportata [61].
*   **`ProfileID : String`**: Identificatore univoco del profilo (es. "PROTOCOLLO_FARMACI_A_FREDDO").
*   **`samplingInterval : Integer`**: Frequenza di campionamento dei sensori espressa in secondi (es. misura ogni 60 secondi).

### 2.9. ThresholdRule (Entità Concettuale)
Definisce la regola di soglia per una singola metrica ambientale all'interno di un profilo di configurazione [61].
*   **`metricName : String`**: Nome della metrica a cui applicare la regola (es. `TEMPERATURE`).
*   **`minValue : Integer`**: Valore minimo consentito prima dell'attivazione dell'allarme.
*   **`maxValue : Integer`**: Valore massimo consentito prima dell'attivazione dell'allarme.

---

## 3. Analisi di Consistenza e Scelte Metodologiche di Pregio

### 3.1. Separazione tra Profilo di Configurazione e Regole di Soglia
Una delle scelte di modellazione più significative per soddisfare il requisito non funzionale di **basso costo e riusabilità dei sensori** [62] è la scomposizione in due classi: `ConfigurationProfile` e `ThresholdRule`.
*   **Motivazione metodologica**: Se avessimo cablato le soglie di allarme direttamente all'interno della classe `IoTDevice` o della classe `Shipment`, avremmo violato il principio di riusabilità [10.9]. Un singolo dispositivo IoT fisico deve poter essere rimosso da una scatola di medicinali (che richiede una temperatura tra 2°C e 8°C) e riutilizzato il giorno dopo su un veicolo che trasporta un'opera d'arte (che richiede limiti di umidità e vibrazioni, ma non refrigerazione).
*   **Soluzione**: Associando `Shipment` a `ConfigurationProfile` (relazione `defined_by` 1-to-1) e quest'ultimo a un set di `ThresholdRule` (relazione `contains` 1-to-many), separiamo la configurazione logica dall'hardware fisico. Il dispositivo IoT riceve semplicemente il `samplingInterval` e la lista di metriche da abilitare, mentre l'intelligenza di allarmistica risiede sul Cloud che incrocia le misure con le regole del profilo attivo.

### 3.2. Blindatura alla Sorgente e Non-Ripudiabilità
Il requisito rigido di **integrità dei dati** [60, 66] è stato tradotto strutturalmente tramite la relazione 1-to-1 di composizione tra `EnvironmentalMeasurement` e `CryptographicSignature` (relazione `secured_by`).
*   **Motivazione metodologica**: Nel modello concettuale, l'esistenza stessa di una misura è inscindibilmente legata alla sua firma crittografica. Poiché la firma viene generata direttamente a bordo del chip hardware del sensore prima di qualsiasi trasmissione, questa relazione strutturale impedisce la modellazione di "misure orfane di firma" o "firme condivise". Questo garantisce che nessun attore umano (nemmeno l'amministratore del database Cloud) possa falsificare o alterare una misura senza invalidare lo stato strutturale `integrityStatus` della firma associata [60, 66].

### 3.3. Gestione dei Vincoli Fisici tramite `LocalGatewayBuffer`
Il sistema presenta un pesante vincolo tecnologico: i sensori sono low-cost, funzionano solo a batteria e comunicano esclusivamente tramite **Bluetooth Low Energy (BLE)**, escludendo moduli 4G/5G o Wi-Fi [62, 68]. Non hanno quindi accesso diretto a Internet.
*   **Risoluzione nel Class Diagram**: L'introduzione della classe `LocalGatewayBuffer` risolve concettualmente questo limite fisico. Essa rappresenta la memoria temporanea gestita dall'App mobile dell'autista (l'agente gateway) [68].
*   La relazione `buffered in` collega `LocalGatewayBuffer` [0..1] a `EnvironmentalMeasurement` [0..*], modellando formalmente lo scenario in cui i dati vengono scaricati dal sensore ma rimangono "bloccati" sullo smartphone in assenza di rete cellulare, per poi essere sincronizzati asincronamente impostando il flag `isCloudSynchronized` a `True` non appena ripristinata la connessione [62, 68].

### 3.4. Analisi delle Cardinalità e Molteplicità Critiche

#### A. Relazione `tracked_by` (`Shipment [0..1] ------- [0..*] IoTDevice`)
*   **Giustificazione del raffinamento a `0..1` lato Shipment**: Inizialmente la molteplicità era impostata a `1` fisso lato Shipment, implicando che un dispositivo IoT debba essere costantemente associato a una spedizione. Metodologicamente questo era un errore di dominio. Un sensore fisico, prima di essere censito o dopo essere stato dismesso (decommissionato), o semplicemente mentre si trova in magazzino per la ricarica della batteria, esiste indipendentemente da qualsiasi viaggio [62, 67]. Impostando la cardinalità a `0..1` espremiamo correttamente che:
    1. Un `IoTDevice` può non essere associato ad alcuna `Shipment` (es. stato di riposo o manutenzione).
    2. Una `Shipment` appena creata (stato `CREATED`) può esistere a livello logico nel Cloud prima che l'operatore di magazzino le assegni fisicamente un sensore per la partenza.

#### B. Relazione `samples` (`IoTDevice [1] ------- [0..*] EnvironmentalMeasurement`)
*   **Giustificazione**: Una misurazione ambientale non può esistere nel mondo reale se non è stata generata da uno specifico sensore fisico (molteplicità `1` lato `IoTDevice`). Viceversa, un dispositivo IoT appena avviato o formattato ha raccolto esattamente `0` misure, ma nel corso del tempo ne collezionerà un numero arbitrario (`*`).

### 3.5. Giustificazione dell'Attributo di Convenienza `isOutOfRange`
Da un punto di vista puramente formale, lo stato "fuori soglia" di una misura è un'informazione derivata, calcolabile tramite la formula logica del primo ordine che confronta `measuredValue` con i campi `minValue` e `maxValue` della `ThresholdRule` corrispondente [10.9].
*   **Spiegazione Accademica per la Sospensione del Dubbio**: Nel Report viene esplicitato che *«l'attributo `isOutOfRange` all'interno della classe `EnvironmentalMeasurement` è modellato esclusivamente come attributo di convenienza strutturale ed ottimizzazione delle prestazioni. Sebbene ridondante sul piano della logica pura, la sua materializzazione nel record della misura consente all'infrastruttura Cloud SaaS di attivare istantaneamente i trigger di allarmistica asincrona sulla Dashboard ed inviare notifiche in tempo reale, evitando costose e continue operazioni di join relazionali su tabelle storiche contenenti milioni di campionamenti»*. Questa giustificazione dimostra una straordinaria sensibilità ingegneristica, bilanciando il rigore formale con i vincoli di efficienza del sistema reale [62].

---

## 4. Matrice di Tracciabilità dei Requisiti (Structural Coverage)

La struttura delle classi concettuali garantisce la copertura totale dei requisiti estratti dalla consegna del progetto [34]:

| Codice Requisito | Descrizione Requisito | Classi e Associazioni Coinvolte | Commento Strutturale |
| :--- | :--- | :--- | :--- |
| **RF1** [61, 66] | Tracking Ambientale Periodico | `EnvironmentalMeasurement`, `IoTDevice` (relazione `samples`) | Consente la memorizzazione delle letture fisiche associate al dispositivo hardware. |
| **RF2** [60, 66] | Firma Digitale alla Sorgente | `CryptographicSignature` (relazione `secured_by`) | Blindatura matematica irrevocabile 1-to-1 tra misura e firma. |
| **RF3** [61, 67] | Gestione e Configurazione Dispositivi | `IoTDevice`, `ConfigurationProfile`, `ThresholdRule` | Supporta il provisioning/decommissioning logico e l'assegnamento dei profili di soglia. |
| **RF4** [61, 67] | Gestione Spedizioni (Stati) | `Shipment` (attributo `logicalState`) | Modella le transizioni e lo stato del viaggio tramite l'enumerazione `ShipmentState`. |
| **RF5** [61, 68] | Live-Dashboard e Allarmistica | `AlarmNotification` (relazione `generates`) | Consente la memorizzazione e la notifica asincrona degli eventi di sforamento delle soglie o violazione d'integrità. |
| **RNF2** [62, 68] | Connettività BLE / No Direct Internet | `LocalGatewayBuffer` (relazione `buffered in`) | Risolve strutturalmente il vincolo dei periodi offline dei sensori tramite buffering mobile. |
| **RNF3** [62, 69] | Monitoraggio Batteria | `IoTDevice` (attributo `batteryLevel`) | Consente il tracciamento dello stato di carica per scatenare allarmi di manutenzione `LOW_BATTERY`. |
