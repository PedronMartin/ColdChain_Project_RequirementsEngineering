# ColdChain_Project_RequirementsEngineering
A formal Requirements Analysis for a fragile goods tracking platform. Showcases goal-oriented modeling and a structured IEEE-830 requirement document.

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
