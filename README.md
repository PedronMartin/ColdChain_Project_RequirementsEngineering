# ColdChain_Project_RequirementsEngineering --- ITA Version
Un'analisi formale dei requisiti per una piattaforma di tracciamento di merci fragili. con modellazione orientata agli obiettivi e un documento dei requisiti strutturato secondo lo standard IEEE-830.

## Contesto accademico e assegnazione del progetto
Questo repository ospita il progetto formale realizzato per il corso magistrale di ingegneria dei requisiti presso l'Università degli Studi di Verona. Il lavoro nasce dalla specifica richiesta di analizzare e modellare un sistema software avanzato per la gestione logistica, il monitoraggio del trasporto e l'immagazzinamento di merci fragili o sensibili, come prodotti farmaceutici, alimenti surgelati, fiori e opere d'arte. L'obiettivo primario consiste nel certificare la qualità e l'integrità dell'intera catena di approvvigionamento tramite dispositivi fisici IoT a basso costo e alimentati a batteria, inseriti direttamente all'interno dei colli. Questi sensori devono raccogliere costantemente misurazioni ambientali, tra cui temperatura, umidità e vibrazioni, trasmettendole in modo sicuro, certificato da firma digitale e a prova di manomissione, per garantire che nessun attore della filiera possa alterare i dati storici del prodotto.

## Panoramica del sistema e architettura ipotizzata
L'infrastruttura architetturale combina i dispositivi fisici IoT operanti sul campo con una piattaforma cloud in modalità SaaS, sfruttando applicazioni mobili su smartphone come ponte di comunicazione. Il sistema mette a disposizione dashboard di visualizzazione per il monitoraggio in tempo reale e interfacce API web per l'interoperabilità dei dati, offrendo una trasparenza totale sui processi logistici. Questa struttura permette, ad esempio, a un consumatore finale di verificare l'autenticità e le condizioni di viaggio del prodotto inquadrando un semplice codice QR. Avvisiamo i visitatori che questo repository si concentra esclusivamente sull'ingegneria dei requisiti, in quanto non contiene codice sorgente implementativo, ma racchiude un'esplorazione logica rigorosa, la valutazione dei rischi e la stesura delle specifiche formali del sistema da realizzare.

## Metodologia di analisi e strumenti adottati
L'intera analisi è stata condotta seguendo il paradigma di ingegneria dei requisiti orientata agli obiettivi, avvalendoci dello strumento di modellazione Objectiver per la derivazione e la verifica degli schemi grafici. La documentazione di specifica formale è stata redatta rispettando rigorosamente le linee guida e la struttura organizzativa dettate dallo standard internazionale IEEE 830-1998. Analizziamo ogni requisito applicando criteri di misurazione oggettivi per garantire che ogni vincolo di qualità sia pienamente verificabile nelle successive fasi di collaudo.

## Elaborati e documentazione inclusa nel repository
Il progetto esplora il problema logistico affrontando le dimensioni intenzionali, strutturali, funzionali e comportamentali attraverso tre documenti analitici principali. All'interno dell'archivio troviamo innanzitutto la relazione descrittiva del progetto, che offre una visione d'insieme del lavoro svolto e delle scelte architetturali. Accanto alla relazione è presente il documento dei requisiti formale secondo lo standard IEEE 830-1998, completato dai criteri di misurazione oggettivi per i vincoli di qualità e prestazione. Il terzo pilastro è costituito dai diagrammi orientati agli obiettivi realizzati e commentati con Objectiver, i quali si articolano su cinque viste complementari. La vista intenzionale presenta l'albero generale dei goal, integrando l'analisi dei rischi tramite i diagrammi degli ostacoli e le relative contromisure di sicurezza. La vista strutturale fotografa i concetti del dominio attraverso il diagramma delle classi, mentre la vista delle responsabilità mappa le deleghe e le dipendenze strategiche tra gli agenti software e ambientali. Infine, la vista funzionale definisce le firme e le regole di operazionalizzazione dei singoli servizi, lasciando alla vista comportamentale il compito di illustrare le dinamiche temporali tramite macchine a stati e diagrammi di sequenza per gli scenari normali e di allarme.

---

### Dettaglio delle viste di modellazione in Objectiver

#### 1. Vista intenzionale e gestione dei rischi
Analizziamo il sistema partendo dalle motivazioni di base, posizionando alla radice il trasporto sicuro e certificato delle merci. Raffiniamo i macro-obiettivi tramite costrutti AND e OR fino a individuare requisiti delegabili a singoli agenti. Esaminiamo contemporaneamente gli ostacoli che minacciano gli obiettivi, tra cui guasti ai sensori, esaurimento della batteria o manomissione logica dei dati. Introduciamo per ogni minaccia precise contromisure, come firme crittografiche, notifiche di manutenzione preventiva e protocolli di comunicazione a basso consumo come il bluetooth low energy.

#### 2. Vista strutturale e concettuale
Fotografiamo le entità del dominio logistico tramite un diagramma delle classi concettuale, evitando scelte premature di design informatico. Strutturiamo il modello definendo entità come la spedizione, il collo e il sensore fisico, legandoli con associazioni che specificano quali dispositivi tracciano i singoli carichi in un dato istante. Definiamo rigorosamente gli attributi di stato dinamico, tra cui le soglie di temperatura ammesse, le vibrazioni massime e la frequenza di campionamento, accompagnando ogni elemento con definizioni esplicite e invarianti di dominio.

#### 3. Vista delle responsabilità e degli agenti
Allochiamo formalmente gli obiettivi di basso livello agli attori del sistema. Distinguiamo gli agenti software, come il controller cloud e l'interfaccia dashboard, dagli agenti ambientali, tra cui operatori logistici, corrieri e autorità di certificazione. Definiamo le interfacce esplicitando le variabili di stato monitorate e controllate da ciascun attore, applicando la regola strutturale secondo cui una variabile può essere modificata da un solo agente alla volta per impedire conflitti operativi.

#### 4. Vista funzionale e operazionalizzazione
Definiamo le transizioni di stato del sistema associando le operazioni ai rispettivi obiettivi foglia. Specifiamo la firma esatta di servizi chiave, come l'attivazione del tracciamento o l'apertura di un allarme di violazione termica, indicando le variabili lette in input e prodotte in output. Esplicitiamo le precondizioni e post-condizioni di dominio, aggiungendo le regole prescrittive di permesso per autorizzare l'esecuzione e le condizioni di innesco per rendere obbligatoria e istantanea l'attivazione dell'operazione.

#### 5. Vista comportamentale e scenari
Rappresentiamo le dinamiche temporali attraverso diagrammi di sequenza e macchine a stati. Gli scenari coprono sia le normali interazioni di consultazione dei dati sulla dashboard pubblica, sia i flussi di eccezione in cui valori fuori intervallo attivano allarmi di sicurezza. Le macchine a stati descrivono il ciclo di vita dei dispositivi e delle spedizioni, tracciando transizioni rigorosamente governate da guardie logiche e operazioni esecutive.

---

### Struttura del documento dei requisiti IEEE 830-1998
Il documento dei requisiti è organizzato per garantire completezza, chiarezza e totale assenza di ambiguità:
* introduzione: scopo del documento, glossario dei concetti logistici e riferimenti alle fonti.
* descrizione generale: prospettiva del sistema SaaS e IoT, caratteristiche degli stakeholder e assunzioni ambientali.
* requisiti specifici: categorizzazione puntuale di requisiti funzionali, interfacce esterne web API, prestazioni, sicurezza e interoperabilità, vincolati a protocolli di misurazione quantitativi per i colli di verifica.



# ColdChain_Project_RequirementsEngineering ---EN. Version
A formal Requirements Analysis for a fragile goods tracking platform. Showcases goal-oriented modeling and a structured IEEE-830 requirement document.

## Academic context and project assignment
This repository hosts the formal project developed for the master's degree course in requirements engineering at the University of Verona. The work arises from the specific request to analyze and model an advanced software system for logistics management, transport monitoring, and storage of fragile or sensitive goods, such as pharmaceuticals, frozen foods, flowers, and artworks. The primary objective is to certify the quality and integrity of the entire supply chain using low-cost, battery-powered physical IoT devices inserted directly into the packages. These sensors must continuously collect environmental measurements, including temperature, humidity, and vibrations, transmitting them in a secure, digitally signed, and tamper-proof manner to ensure that no actor in the supply chain can alter the historical product data.

## System overview and proposed architecture
The architectural infrastructure combines physical IoT devices operating in the field with a SaaS cloud platform, utilizing mobile applications on smartphones as a communication bridge. The system provides display dashboards for real-time monitoring and web API interfaces for data interoperability, offering total transparency over logistics processes. This structure allows, for example, an end consumer to verify the authenticity and travel conditions of the product by scanning a simple QR code. Visitors are advised that this repository focuses exclusively on requirements engineering, as it contains no implementation source code, but encompasses a rigorous logical exploration, risk assessment, and the drafting of formal system specifications.

## Analysis methodology and tools
The entire analysis was conducted following the goal-oriented requirements engineering paradigm, utilizing the Objectiver modeling tool for deriving and verifying graphical diagrams. The formal specification documentation was drafted strictly adhering to the guidelines and organizational structure dictated by the international IEEE 830-1998 standard. Analyzing each requirement involves applying objective measurement criteria to ensure that every quality constraint is fully verifiable during subsequent testing phases.

## Repository contents and included artifacts
The project explores the logistics problem by addressing the intentional, structural, functional, and behavioral dimensions through three main analytical documents. Inside the archive, visitors can find first of all the descriptive project report, which provides an overview of the work done and architectural choices. Alongside the report is the formal requirements document according to the IEEE 830-1998 standard, completed by objective measurement criteria for quality and performance constraints. The third pillar consists of the goal-oriented diagrams created and commented using Objectiver, which are structured into five complementary views. The intentional view presents the general tree of goals, integrating risk analysis through obstacle diagrams and relative security countermeasures. The structural view captures domain concepts through the class diagram, while the responsibility view maps strategic delegations and dependencies between software and environmental agents. Finally, the functional view defines signatures and operationalization rules for individual services, leaving to the behavioral view the task of illustrating temporal dynamics via state machines and sequence diagrams for normal and alarm scenarios.

---

### Objectiver modeling views in detail

#### 1. Intentional view and risk management
Analyzing the system starts from fundamental motivations, placing secure and certified transport of goods at the root. Refining macro-goals via AND and OR constructs allows identifying requirements that can be delegated to individual agents. Examining obstacles that threaten goals simultaneously reveals risks such as sensor failures, battery depletion, or logical data tampering. Introducing precise countermeasures for each threat involves cryptographic signatures, preventive maintenance notifications, and low-power communication protocols such as bluetooth low energy.

#### 2. Structural and conceptual view
Capturing logistics domain entities via a conceptual class diagram avoids premature software design choices. Structuring the model defines entities such as shipment, package, and physical sensor, linking them with associations that specify which devices track individual loads at any given moment. Defining dynamic state attributes rigorously includes allowed temperature thresholds, maximum vibrations, and sampling frequency, accompanying each element with explicit definitions and domain invariants.

#### 3. Responsibility and agent view
Allocating low-level goals formally to system actors distinguishes software agents, such as cloud controllers and dashboard interfaces, from environmental agents, including logistics operators, couriers, and certification authorities. Defining interfaces clarifies monitored and controlled state variables for each actor, applying the structural rule that a variable can be modified by only one agent at a time to prevent operational conflicts.

#### 4. Functional view and operationalization
Defining system state transitions links operations to their respective leaf goals. Specifying exact signatures for key services, such as tracking activation or triggering a thermal violation alarm, indicates input read variables and output produced variables. Making domain pre-conditions and post-conditions explicit integrates prescriptive permission rules to authorize execution and trigger conditions to make operation activation mandatory and instantaneous.

#### 5. Behavioral view and scenarios
Representing temporal dynamics involves sequence diagrams and state machines. Scenarios cover both normal interactions of viewing data on the public dashboard and exception flows where out-of-range values trigger security alarms. State machines describe the lifecycle of devices and shipments, tracking transitions strictly governed by logical guards and executive operations.

---

### IEEE 830-1998 requirements document structure
The requirements document is organized to ensure completeness, clarity, and total absence of ambiguity:
* introduction: document purpose, glossary of logistics concepts, and references to sources.
* general description: SaaS and IoT system perspective, stakeholder characteristics, and environmental assumptions.
* specific requirements: detailed categorization of functional requirements, external web API interfaces, performance, security, and interoperability, bound to quantitative measurement protocols for testing benchmarks.
