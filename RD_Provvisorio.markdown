# Software Requirements Specification
<!-- Tra i titoli a due ## uso 3 /n per separare, tra i sottotitoli a tre # uso 2 /n, e via dicendo... -->


## Table of Contents
<!-- INDICE -->
* [1. Introduction](#1-introduction)
    * [1.1 Document Purpose](#11-document-purpose)
    * [1.2 Product Scope](#12-product-scope)
    * [1.3 Definitions, Acronyms, and Abbreviations](#13-definitions-acronyms-and-abbreviations)
    * [1.4 References](#14-references)
    * [1.5 Document Overview](#15-document-overview)
* [2. Product Overview](#2-product-overview)
    * [2.1 Product Perspective](#21-product-perspective)
    * [2.2 Product Functions](#22-product-functions)
    * [2.3 Product Constraints](#23-product-constraints)
    * [2.4 User Characteristics](#24-user-characteristics)
    * [2.5 Assumptions and Dependencies](#25-assumptions-and-dependencies)
    * [2.6 Apportioning of Requirements](#26-apportioning-of-requirements)
* [3. Requirements](#3-requirements)
    * [3.1 External Interfaces](#31-external-interfaces)
    * [3.2 Functional](#32-functional)
    * [3.3 Quality of Service](#33-quality-of-service)
    * [3.4 Compliance](#34-compliance)
    * [3.5 Design and Implementation](#35-design-and-implementation)
    * [3.6 AI/ML](#36-aiml)
* [4. Verification](#4-verification)
* [5. Appendixes](#5-appendixes)
<!-- INDICE -->



## 1. Introduction


### 1.1 Document Purpose
Il presente documento descrive in modo formale e completo le specifiche dei requisiti software per la piattaforma di tracciamento e monitoraggio logistico di merci fragili e sensibili (quali farmaci, alimenti surgelati, fiori e opere d'arte). Lo scopo principale è fornire una base rigorosa e non ambigua per gli sviluppatori, i verificatori e gli stakeholder, definendo le funzionalità del sistema, i vincoli architetturali e i requisiti di qualità necessari a certificare l'integrità della catena di approvvigionamento.


### 1.2 Product Scope
Il prodotto software, denominato <<NOME>> (Versione 1.0), nasce con l'obiettivo di garantire la trasparenza, la tracciabilità e la certificazione della qualità lungo l'intera catena di approvvigionamento di merci fragili e sensibili. Le sue capacità chiave comprendono la raccolta sicura e firmata digitalmente dei dati telemetrici ambientali tramite dispositivi IoT, l'esposizione di dashboard in tempo reale e l'interoperabilità dei dati tramite API web. Il sistema mira a eliminare i problemi di fiducia tra gli attori della filiera e a consentire ai consumatori finali di verificare l'autenticità dei prodotti. Questo documento copre interamente la piattaforma SaaS cloud e la sua integrazione con i sensori di campo, escludendo tuttavia la produzione fisica dell'hardware e lo sviluppo nativo delle applicazioni per smartphone dei corrieri, che fungono unicamente da componenti di interfaccia esterna.

Il seguente diagramma logico definisce il confine esatto tra il software da sviluppare (Macchina) e l'ambiente esterno (Mondo), evidenziando il flusso delle variabili monitorate:

```
+---------------------------------------------------------------------------------+
|                                   MONDO (Ambiente)                              |
|                                                                                 |
|  [Sensori Fisici IoT] --(Bluetooth LE)--> ( Smartphone Autista / Magazziniere ) |
|       (Esclusi)                                |                                |
|                                                |                                |
+------------------------------------------------|--------------------------------+
                                                 | (Connessione 4G/5G)
+------------------------------------------------v--------------------------------+
|                             MACCHINA (Software-to-be)                           |
|                                                                                 |
|  +--------------------+       +---------------------+       +----------------+  |
|  |  App Mobile        | ----> |  Backend Cloud      | ----> | API Pubbliche  |  |
|  |  Gateway Autista   |       |  SaaS (Dashboard)   |       | (Interoperab.) |  |
|  +--------------------+       +---------------------+       +----------------+  |
|                                                                                 |
+---------------------------------------------------------------------------------+
```


### 1.3 Definitions
La seguente tabella definisce in modo non ambiguo tutti i termini specialistici, gli acronimi, le abbreviazioni e i concetti metodologici utilizzati all'interno di questo documento dei requisiti (RD). I termini sono presentati in ordine alfabetico per facilitarne la consultazione e garantire l'assoluta coerenza semantica nell'intero set documentale.

| Termine | Acronimo | Definizione |
| :--- | :--- | :--- |
| **API** | Application Programming Interface | Insieme di protocolli e definizioni software che espongono in modo sicuro e interoperabile i dati di tracciamento memorizzati nel cloud a sistemi esterni, quali applicativi di terze parti o sistemi informatici degli enti di certificazione.
| **Assunzione (Assumption / Expectation)** | | Enunciato prescrittivo rivolto a un componente o attore dell'ambiente esterno alla macchina software-to-be (es. il comportamento dell'autista o la durata della batteria dell'hardware), che il software non può forzare direttamente ma di cui presuppone il corretto funzionamento per garantire il soddisfacimento dei goal globali.
| **BLE** | Bluetooth Low Energy | Protocollo di trasmissione wireless a corto raggio e a bassissimo consumo energetico, utilizzato dai dispositivi IoT fisici per trasmettere localmente le letture ambientali al gateway mobile dell'autista. |
| **Dato Certificato** | | Misurazione di una grandezza fisica ambientale (temperatura, umidità, vibrazioni) firmata digitalmente all'origine dal dispositivo IoT tramite chiave privata, garantendone l'integrità e rendendola matematicamente a prova di manomissione lungo la filiera.
| **Dashboard Cloud** | |Interfaccia web centralizzata, erogata in modalità SaaS, che consente ai manager della logistica e alle autorità di certificazione di gestire le spedizioni, monitorare i dati in tempo reale e ricevere notifiche visive in caso di anomalie.
| **Dispositivo IoT (Sensore)** | | Unità hardware economica, riutilizzabile e alimentata a batteria, inserita all'interno dei box di trasporto o dei veicoli, deputata alla misurazione periodica delle grandezze fisiche del trasporto e sprovvista di connessione internet diretta.
| **Fit Criterion** | | Criterio di misurabilità quantitativo associato a ciascun requisito per renderlo oggettivamente verificabile in fase di Quality Assurance (QA), espresso esclusivamente tramite grandezze fisiche misurabili e protocolli di test (es. soglie numeriche di tempo o tolleranze fisiche).
| **Gateway Mobile (App Autista)** | | Applicazione installata sul dispositivo mobile (smartphone o tablet) del conducente che funge da ripetitore logico: riceve i dati firmati dai sensori tramite BLE e li inoltra al cloud di <<NOME>> sfruttando la connettività di rete cellulare (4G/5G).
| **Proprietà del Dominio (Domain Property)** | | Enunciato descrittivo che fotografa leggi naturali, vincoli fisici o regole inalterabili del mondo reale (es. "un farmaco termosensibile esposto a temperature superiori a 8°C perde efficacia") che rimangono veri a prescindere dall'esistenza o dall'azione del software.
| **Spedizione (Shipment)** | |Entità logica che identifica il processo di trasporto di un lotto di merci sensibili da un punto di partenza a un punto di arrivo, caratterizzato da uno stato operativo (es. Creata, Avviata, In Pausa, Conclusa) e associato a uno o più dispositivi IoT.
| **RD** | Document Requirements) | Il presente documento di specifica formale che descrive in modo rigoroso cosa il sistema software <<NOME>> deve fare (requisiti funzionali) e come deve operare (requisiti non funzionali e vincoli) per soddisfare i bisogni di business.
| **UI** | User Interface | Componente visiva e interattiva delle applicazioni (App Gateway e Dashboard Cloud) attraverso la quale i differenti attori umani (Autista, Manager, Consumatore) interagiscono con il sistema.


### 1.4 References
I seguenti documenti costituiscono i riferimenti normativi (vincolanti per lo sviluppo) o informativi (linee guida di supporto) utilizzati per la redazione del presente Documento dei Requisiti (RD):

* **IEEE Std 830-1998**
  * **Titolo**: IEEE Recommended Practice for Software Requirements Specifications
  * **Autore/Editore**: IEEE Computer Society (Software Engineering Standards Committee)
  * **Data**: 20 Ottobre 1998
  * **Tipologia**: Normativa (definisce la struttura, lo standard e i criteri di conformità di questo documento)
  * **Collocazione**: Standard ufficiale IEEE ([https://standards.ieee.org/](https://standards.ieee.org/))

* **Specifiche di Progetto "Logistics on fragile/sensitive items"**
  * **Titolo**: Course Project: Logistics on fragile/sensitive items (Slide di Laboratorio 2)
  * **Autore**: Prof. Mariano Ceccato, Università degli Studi di Verona[cite: 1]
  * **Versione/Data**: A.A. 2025/2026
  * **Tipologia**: Normativa (definisce i requisiti funzionali, non funzionali e i vincoli di dominio imposti dal committente)
  * **Collocazione**: Repository didattica Moodle
  

Aggiungi template???
 
 
 ### 1.5 Document Overview
Questo documento è strutturato in tre sezioni principali: dopo l'attuale capitolo introduttivo, il Capitolo 2 fornisce una descrizione generale ad alto livello delle funzioni, dei vincoli fisici e delle assunzioni sul dominio, mentre il Capitolo 3 formalizza i requisiti specifici (funzionali e non funzionali) completi dei relativi parametri di misurabilità (Fit criteria). All'interno del testo viene applicata la convenzione standard per cui l'ausiliare "shall" identifica requisiti obbligatori e vincolanti, mentre "should" indica caratteristiche desiderabili ma opzionali.



## 2. Product Overview


### 2.1 Product Perspective

Il sistema <<NOME>> (v1.0) è un prodotto software completamente nuovo progettato per operare come una piattaforma indipendente di monitoraggio, notarizzazione e certificazione logistica. Il sistema non sostituisce alcun software preesistente, ma si inserisce in un ecosistema distribuito interagendo con componenti hardware e sistemi informativi esterni.
Il posizionamento del prodotto all'interno dell'ecosistema logistico è definito dai seguenti confini di integrazione e responsabilità:

 - Sistemi a monte (Upstream): il software-to-be riceve i dati telemetrici ambientali da dispositivi hardware fisici (sensori IoT) posizionati nei colli. Questi sensori sono di proprietà dell'azienda logistica ma prodotti da terze parti; la macchina software interagisce con essi esclusivamente tramite protocollo di input Bluetooth Low Energy (BLE), assumendo che l'hardware esegua la firma crittografica dei dati all'origine;
   
 - Sistemi a valle (Downstream): il sistema espone interfacce di rete API verso l'esterno per consentire l'interoperabilità con i sistemi gestionali (ERP) (va messo negli acromini?) dei clienti e con i portali web degli Enti di Certificazione terzi, che agiscono come ispettori indipendenti della qualità della filiera;

 - Proprietà e Hosting: la piattaforma Cloud (backend e database) è di proprietà del fornitore del servizio logistico ed è ospitata su un'infrastruttura Cloud pubblica in modalità SaaS (Software-as-a-Service). Le applicazioni client (Dashboard Web e App Mobile Gateway) sono gestite e aggiornate centralmente dal fornitore.


### 2.2 Product Functions

Il sistema <<NOME>> organizza le proprie capacità operative in macro-aree funzionali, progettate per supportare in modo sinergico le attività degli attori umani e dei dispositivi fisici sul campo. Di seguito viene fornita una panoramica concisa delle principali funzionalità fornite dalla macchina software, rimandando la specifica dei singoli flussi di dettaglio e dei casi d'uso d'eccezione alla Sezione 3:

1. Gestione del ciclo di vita delle spedizioni (Shipments Management): il sistema consente ai Manager Logistici (aggiungere ad abbreviazioni del cap1 oppure nel 2.3 basta?) di creare, ispezionare, modificare o cancellare logicamente i lotti di spedizione associati alle merci sensibili. La macchina software permette di controllare lo stato operativo di ogni viaggio attraverso transizioni esplicite di avvio (Start), sospensione temporanea (Pause), ripresa (Resume) e completamento (Stop);

2. Gestione e associazione dei dispositivi IoT (Devices Management): il software fornisce gli strumenti per registrare nuovi sensori fisici nel sistema (provisioning), dismetterli permanentemente (decommissioning), consultarne lo stato operativo in una lista centralizzata (listing) e associarli o dissociarli liberamente a una determinata spedizione attiva per tracciarne il carico;

3. Configurazione remota dei sensori (Sensor Configuration): consente di personalizzare a livello cloud il comportamento operativo di ogni sensore IoT associato a una spedizione. Il manager può definire parametri chiave quali la frequenza di campionamento delle grandezze fisiche, la frequenza di sincronizzazione dei dati (upload) e selezionare quali metriche ambientali specifiche raccogliere (es. temperatura, umidità, vibrazioni o radiazioni UV) a seconda della tipologia di merce trasportata;

4. Sincronizzazione e notarizzazione delle letture (Tracking & Notarization): il sistema raccoglie via BLE i dati telemetrici ambientali accumulati localmente dai sensori fisici e li inoltra automaticamente alla piattaforma Cloud sfruttando la connettività di rete del Gateway mobile. Il backend Cloud (già messo nel cap 1?) esegue la verifica della firma digitale asimmetrica inserita alla sorgente dal sensore per ciascuna lettura, garantendone la provenienza e memorizzandola in modo inalterabile per prevenire tentativi di contraffazione da parte degli operatori;

5. Dashboard di monitoraggio e allarmi Real-Time (Live Dashboard & Alarms): offre ai manager e agli autisti una vista unificata e interattiva sullo stato delle spedizioni in corso. Il sistema analizza costantemente i flussi di dati in input e attiva segnalazioni grafiche immediate (condizioni di allarme "out of range") nel caso in cui le grandezze fisiche monitorate violino le soglie di tolleranza stabilite per quel determinato carico, garantendo l'evidenza immediata delle anomalie;

6. Monitoraggio proattivo dello stato hardware (Device Battery Alert): la piattaforma estrae le informazioni relative allo stato di salute interno dei dispositivi IoT (es. il livello di carica della batteria fisica) e genera notifiche automatiche di manutenzione sulla dashboard del manager non appena il livello scende sotto una soglia critica di sicurezza, impedendo buchi di tracciamento dovuti allo spegnimento improvviso dei sensori;

7. Esposizione API per l'interoperabilità esterna (Web API Gateway): la piattaforma espone un'interfaccia web programmabile (API REST sicure) (mi pare di aver definito solo API e non REST) per consentire lo scambio automatico dei dati di tracciamento notarizzati con sistemi informativi esterni, quali gli ERP gestionali dei clienti o le piattaforme tecnologiche degli Enti di Certificazione terzi;

8. Verifica trasparente della catena di custodia (End-User Validation): fornisce ai consumatori finali e agli auditor di qualità un portale pubblico di consultazione (accessibile in modo rapido, ad esempio inquadrando un codice QR univoco stampato sul collo del prodotto) che mostra in modo trasparente e intuitivo l'intero storico notarizzato del trasporto, certificando la conformità del processo di filiera. (ha senso fare esempio per il qrcode o dovrebbe essere una cosa da scegliere punto e basta?).


## 2.3 User Characteristics

In questa sezione vengono identificate e caratterizzate le diverse classi di utenti che interagiscono con il software <<NOME>>. Ciascuna classe è definita in base al comportamento operativo, alle competenze tecniche, al livello di autorizzazione e agli obiettivi di dominio, includendo considerazioni specifiche in merito a usabilità, localizzazione e accessibilità della UI.

1. Manager Logistico (Logistics Manager)

    Ruolo e comportamento: è l'amministratore operativo del sistema logistico. Configura i parametri dei sensori IoT (frequenze, metriche, soglie), inserisce ed elenca i dispositivi, pianifica le spedizioni a livello logico e interviene tempestivamente in caso di allarmi.
    
    Competenza tecnica: medio-alta. Ha familiarità con l'uso di computer desktop, browser web, applicativi gestionali aziendali (ERP) e piattaforme cloud in modalità SaaS.
    
    Livello di accesso: accesso completo in lettura e scrittura alla Dashboard Cloud per la gestione globale di flotte, spedizioni, configurazioni hardware e visualizzazione dei log storici.
    
    Frequenza d'uso: giornaliera e continuativa (interagisce con la dashboard desktop più volte al giorno per monitorare lo stato della flotta e delle spedizioni attive).
    
    Obiettivo principale: garantire che tutti i viaggi avvengano nel rispetto dei parametri fisici stabiliti, minimizzare le perdite di carico dovute a deterioramento termico o meccanico e monitorare lo stato di manutenzione dei sensori fisici.

2. Vettore / Autista (Driver)

    Ruolo e comportamento: è l'operatore sul campo responsabile del trasporto fisico delle merci sensibili. Interagisce direttamente con l'applicazione Gateway installata sul proprio dispositivo mobile di lavoro per comunicare l'avanzamento logico del viaggio (avvio, sosta, completamento) e ricevere allarmi in tempo reale.
    
    Competenza tecnica: medio-bassa. Utilizza abitualmente lo smartphone per scopi personali o per la navigazione stradale GPS, ma necessita di procedure semplificate che non interferiscano con la concentrazione durante la guida.
    
    Livello di accesso: accesso limitato tramite credenziali personali sul Gateway mobile: può visualizzare solo le spedizioni a lui assegnate, azionare i comandi di stato della tratta (Start/Stop/Pause/Resume) e ricevere avvisi. Non può configurare i sensori o modificare i dati storici memorizzati.
    
    Frequenza d'uso: intermittente (esclusivamente all'inizio del viaggio, durante le fermate intermedie per le pause e al momento dell'arrivo a destinazione).
    
    Obiettivo principale: completare la consegna nei tempi stabiliti ed essere avvisato istantaneamente se le condizioni fisiche del vano refrigerato escono dalle soglie di tolleranza, così da poter applicare procedure di emergenza.

3. Ente di Certificazione (Certification Authority):

    Ruolo e comportamento: è un auditor esterno e indipendente appartenente a un ente terzo deputato al rilascio di bollini di qualità o di sostenibilità ecologica per le merci trasportate (es. rispetto continuo della catena del freddo o verifica dei chilometri percorsi).
    
    Competenza tecnica: media. Ha familiarità con l'analisi di report di conformità, fogli di calcolo e l'utilizzo di portali web dedicati.
    Livello di accesso: accesso in sola lettura tramite Dashboard Web o credenziali API dedicate per ispezionare lo storico notarizzato e firmato digitalmente delle spedizioni concluse.
    
    Frequenza d'uso: occasionale (in corrispondenza dei cicli ispettivi di verifica o a fronte della richiesta di una nuova certificazione per una determinata azienda).
    
    Obiettivo principale: verificare con assoluta certezza matematica, tramite validazione delle firme digitali all'origine dei dati, che il processo logistico sia avvenuto in totale conformità con le specifiche normative di conservazione.

4. Consumatore finale (End Consumer):

    Ruolo e comportamento:è l'acquirente ultimo del prodotto fragile o deperibile (es. un paziente che ritira un vaccino o un farmaco termosensibile in farmacia). Desidera sincerarsi della qualità del processo logistico che ha portato il prodotto fino a lui.
    
    Competenza tecnica: estremamente variabile (da minima ad alta). Rappresenta l'utente generico della popolazione che utilizza il proprio smartphone personale (se, come detto, impostiamo il QRCODE...da verificare!).
    
    Livello di accesso: accesso pubblico, anonimo e in sola lettura a una pagina web di consultazione semplificata, attivabile inquadrando un codice identificativo univoco (es. QR Code o Tag NFC) stampato sulla confezione fisica del prodotto.
    
    Frequenza d'uso: saltuaria (esclusivamente al momento dell'acquisto o del ritiro della merce).
    
    Obiettivo principale: verificare in modo trasparente e immediato che la merce non sia stata esposta a condizioni ambientali dannose durante tutta la filiera produttiva e distributiva.


## 2.4 General Constraints

Questa sezione definisce l'insieme dei vincoli tecnologici, fisici, di sicurezza e normativi che limitano lo spazio di progettazione e implementazione del sistema <<NOME>>. Tali vincoli rappresentano requisiti rigidi non negoziabili imposti dal dominio d'applicazione o da decisioni strategiche di business. I requisiti specifici descritti nella Sezione 3 avranno il compito di operazionalizzare e verificare il rispetto di questi limiti.

1. Vincoli hardware e di alimentazione (Hardware & Power Constraints):
  
   CON-HW-01 [Mandatory - External]. I dispositivi di tracciamento IoT sul campo devono operare esclusivamente tramite alimentazione autonoma a batteria. Al fine di garantire la sostenibilità economica della filiera, i dispositivi devono essere di tipo economico (commodity hardware) e rigorosamente riutilizzabili per spedizioni, colli e veicoli differenti;
   
    CON-HW-02 [Mandatory - External]. I dispositivi IoT non devono possedere interfacce radio WAN dirette (quali antenne cellulari 4G/5G o moduli Wi-Fi) per la connessione diretta a Internet. Tutta la comunicazione e lo scaricamento dei dati telemetrici a corto raggio deve avvenire esclusivamente tramite interfaccia Bluetooth Low Energy (BLE);
    
    CON-HW-03 [Mandatory - External]. L'hardware del sensore IoT deve disporre di memoria flash locale non volatile per bufferizzare le letture telemetriche in caso di assenza temporanea di connessione BLE con l'applicazione Gateway dell'autista.

2. Vincoli di sicurezza, trust e crittografia (Security & Trust Constraints):

    CON-SEC-01 [Mandatory - External]. Tutti i dati fisici raccolti dai sensori ambientali devono essere firmati digitalmente alla sorgente direttamente dall'hardware del dispositivo IoT tramite crittografia asimmetrica (usando la chiave privata memorizzata nel sensore). Il sistema software Cloud deve poter verificare l'integrità del dato utilizzando esclusivamente la chiave pubblica associata;
    
    CON-SEC-02 [Mandatory - External]. I dati storici di tracciamento, una volta notarizzati ed inviati al Cloud, devono essere matematicamente e logicamente immodificabili (tamper-proof). Il sistema deve garantire l'impossibilità di alterazione o cancellazione logica dei dati da parte di qualsiasi attore umano della filiera, inclusi amministratori di sistema, manager della logistica, vettori o clienti finali;
    
    CON-SEC-03 [Preferred - Internal]. La memorizzazione delle credenziali e lo scambio dei token di autenticazione per le dashboard web e le applicazioni Gateway mobili devono avvenire nel rispetto degli standard industriali di sicurezza (es. hashing forte delle password e protocolli di cifratura TLS 1.3 per il traffico di rete).

3. Vincoli architetturali e di deployment (Architectural & Deployment Constraints):

    CON-ARC-01 [Mandatory - Internal]. Il backend (da definire?) del sistema e le dashboard di monitoraggio devono essere ospitati su un'infrastruttura Cloud erogata in modalità Software-as-a-Service (SaaS). L'architettura deve essere nativamente multi-tenant (???) per consentire la gestione logica separata di clienti differenti sulla medesima istanza fisica di esecuzione, massimizzando la scalabilità economica e prestazionale;
    
    CON-ARC-02 [Mandatory - External]. L'esposizione dei dati verso gli Enti di Certificazione esterni e i sistemi informativi dei clienti (ERP) deve avvenire tramite un API Gateway pubblico conforme agli standard REST con payload formattati rigorosamente in formato JSON (ha senso nel RD parlare così tecnico?), garantendo l'interoperabilità e la trasparenza dei processi;

4. Vincoli di conformità e regolamentazione (Regulatory & Compliance Constraints):

    CON-REG-01 [Mandatory - External]. Nello scenario di tracciamento della catena del freddo per prodotti farmaceutici e biologici (es. vaccini), i meccanismi di storicizzazione, i tempi di campionamento e le procedure di calibrazione dei sensori devono essere pienamente conformi alle linee guida internazionali GDP (Good Distribution Practice) e alle normative sanitarie di riferimento per la validazione dei sistemi di conservazione delle merci termosensibili.


## 2.5 Assumptions and Dependencies

Il corretto funzionamento e il soddisfacimento dei requisiti globali di <<NOME> dipendono da una serie di assunzioni relative al comportamento dell'ambiente esterno e da dipendenze tecnologiche verso servizi terzi. Di seguito vengono dettagliati tali fattori, evidenziando per ciascuno l'impatto potenziale sul sistema in caso di mancato rispetto e la relativa contromisura di mitigazione progettuale.

Assunzioni e ipotesi sull'Ambiente (Environmental Assumptions - ASM):

1. ASM-CONN-01 (Connettività Mobile del Gateway)
   
   Assunzione: si assume che il dispositivo mobile dell'autista (Gateway) disponga di connettività internet cellulare attiva (4G/5G) per la maggior parte (definire in modo più rigoroso?) della durata del viaggio logistico.
   
   Impatto in caso di fallimento: i dati telemetrici rimangono bloccati sullo smartphone (definire smartphone?) dell'autista, impedendo il tracciamento in tempo reale sulla dashboard web del manager logistico e ritardando la notifica tempestiva di eventuali allarmi critici.
   
   Mitigazione: l'applicazione Gateway mobile deve implementare un meccanismo di memorizzazione locale offline (es. database SQLite locale) (troppo tecnico?) per bufferizzare (si può dire in un RD una parola italianizzata? E' troppo tecnico?) i dati firmati dai sensori e inviarli in blocco non appena la connettività di rete viene ripristinata.
   
2. ASM-USER-01 (Correttezza dell'Associazione Logistica)
   
   Assunzione: si assume che l'operatore umano (manager logistico o magazziniere) esegua in modo accurato l'associazione fisica e logica tra il sensore IoT e la spedizione tramite dashboard prima che il veicolo parta dall'hub di origine.
   
   Impatto in caso di fallimento: i dati telemetrici inviati dal sensore non possono essere ricondotti ad alcuna spedizione attiva, rendendo nullo il monitoraggio e impedendo la notarizzazione dello storico della filiera per quel lotto.
   
   Mitigazione: l'applicazione software impedisce fisicamente l'avvio della spedizione se l'utente non ha associato ad essa almeno un sensore IoT precedentemente censito e attivo.
   
3. ASM-BAT-01 (Gestione della Batteria all'Origine)

   Assunzione: ci si aspetta che i sensori IoT vengano posizionati all'interno dei box di spedizione solo se provvisti di un livello di carica della batteria sufficiente a coprire l'intera durata stimata del viaggio.
   
   Impatto in caso di fallimento: lo spegnimento improvviso del sensore a metà tratta causa un'interruzione irreversibile nella raccolta dei dati ambientali, invalidando l'emissione del certificato finale di integrità della filiera.
   
   Mitigazione: il software cloud contrassegna automaticamente come "Non idonei alla spedizione" i sensori il cui livello di batteria comunicato nell'ultimo campionamento risulta inferiore al 20%.
   
Dipendenze da Sistemi Esterni (External Dependencies - DEP)

1. DEP-GPS-01 (Servizio di Geolocalizzazione dello Smartphone)

   Dipendenza: il sistema dipende dall'attivazione e dalla precisione del modulo hardware GPS (definisci?) e dei relativi servizi di localizzazione nativi del sistema operativo dello smartphone del conducente (iOS/Android).
   
   Impatto in caso di fallimento: i dati telemetrici inviati al Cloud non conterranno le coordinate geografiche di viaggio, impedendo il tracciamento del percorso e la verifica dei chilometri totali da parte dell'Ente di Certificazione.
   
   Mitigazione: l'applicazione Gateway mobile controlla i permessi di localizzazione all'avvio del viaggio, notificando l'autista in modo bloccante in caso di disattivazione, e memorizza temporaneamente l'ultima coordinata valida nota in caso di temporanea perdita di segnale GPS (es. gallerie).
   
2. DEP-CLOUD-01 (Disponibilità del Cloud Provider Terzo)

   Dipendenza: l'intera infrastruttura di backend, l'API Gateway e il database sicuro del sistema dipendono dalla continuità di servizio e dall'uptime garantito dal cloud provider selezionato (es. AWS, Microsoft Azure, Google Cloud).
   
   Impatto in caso di fallimento: interruzione totale dei servizi di dashboard web, impossibilità di ricevere pacchetti dati dai vettori e inaccessibilità dei dati storici per gli ispettori e i clienti finali.
   
   Mitigazione: ???
   
3. DEP-TIME-01 (Sincronizzazione Temporale Globale - NTP)

   Dipendenza: il sistema dipende dalla sincronizzazione oraria tramite server NTP (definisci?) per garantire che i clock interni del sensore IoT, dello smartphone e del cloud siano allineati entro una tolleranza massima di 60 secondi (rivedi parametro?).
   
   Impatto in caso di fallimento: sfasamento temporale che può causare il fallimento della validazione delle firme digitali temporali o un disallineamento cronologico delle misurazioni nel database Cloud.
   
   Mitigazione: l'applicazione Gateway mobile e il Cloud verificano lo scostamento temporale del pacchetto dati ricevuto dal sensore rispetto all'orario del server e applicano algoritmi di compensazione software o rigettano i pacchetti gravemente corrotti.
   
   
ALTRI? (vedi file su Drive)


## 2.6 Apportioning of Requirements

NB: questa parte non mi convince molto, non ho ben capito cosa bisogna fare e se abbia senso farla per questo progetto.

Questa sezione definisce l'allocazione dei requisiti del sistema attraverso i diversi agenti (sottosistemi) che compongono il System-to-be. 

Trattandosi di un progetto accademico con un unico traguardo di consegna, tutti i requisiti qui elencati sono considerati mandatori e vengono analizzati, modellati e specificati all'interno del presente documento con il medesimo livello di dettaglio. La ripartizione non avviene su base temporale (release successive), ma si focalizza sull'allocazione delle responsabilità fisiche e logiche tra i tre sottosistemi principali identificati nei confini del sistema.

La seguente tabella mappa ciascun requisito del sistema descritto nel documento rispetto al sottosistema (Agente) responsabile del suo soddisfacimento. Tale allocazione costituisce il vincolo formale per la successiva stesura dell'Agent Diagram all'interno dei modelli Objectiver.

| ID Requisito | Descrizione del Servizio / Vincolo | Sottosistema Responsabile (Agente) | Tipologia di Requisito
| :--- | :--- | :--- | :--- |
| **REQ-LOG-01** | Controllo stati del viaggio (*Start, Pause, Resume, Stop*) | App Gateway Mobile / Cloud SaaS | Funzionale (Core)
| **REQ-LOG-02** | Censimento e associazione logica dei sensori ai lotti | Backend Cloud SaaS | Funzionale (Core)
| **REQ-TRK-01** | Campionamento periodico dei parametri ambientali | Sensore IoT | Funzionale (Core)
| **REQ-TRK-02** | Trasmissione dati locali via protocollo BLE | Sensore IoT / App Gateway | Funzionale (Core)
| **REQ-SEC-01** | Firma crittografica asimmetrica delle letture | Sensore IoT | Sicurezza (NFR)
| **REQ-SEC-02** | Verifica della firma e storicizzazione non modificabile | Backend Cloud SaaS | Sicurezza (NFR)
| **REQ-ALM-01** | Rilevamento violazioni e allarmi grafici in Dashboard | Backend Cloud SaaS | Funzionale (Core)
| **REQ-ALM-02** | Segnalazioni acustiche e visive immediate per il vettore | App Gateway Mobile | Usabilità (NFR)
| **REQ-HW-01** | Monitoraggio stato batteria e alert di manutenzione | Sensore IoT / Cloud SaaS | Affidabilità (NFR)

IDK, non mi convince molto. In particolare gli agenti devono essere solo 1 per requisito se non sbaglio (dalla teoria), quindi cmq va revisionato.

