---
lab:
  title: Introduzione al Servizio OpenAI di Azure
---

# Introduzione al Servizio OpenAI di Azure

Il Servizio OpenAI di Azure offre i modelli di intelligenza artificiale generativi sviluppati da OpenAI alla piattaforma Azure, consentendo di sviluppare potenti soluzioni di intelligenza artificiale che traggono vantaggio dalla sicurezza, dalla scalabilità e dall'integrazione di servizi forniti dalla piattaforma cloud di Azure. In questo esercizio verrà illustrato come iniziare a usare Azure OpenAI effettuando il provisioning del servizio come risorsa di Azure e usando Fonderia Azure AI per distribuire ed esplorare i modelli di IA generativa.

Nello scenario per questo esercizio si assumerà il ruolo di sviluppatore di software che è stato incaricato di implementare un agente di intelligenza artificiale che può usare l'intelligenza artificiale generativa per aiutare un'organizzazione di marketing a migliorare l'efficacia nel raggiungere i clienti e pubblicizzare nuovi prodotti. Le tecniche usate nell'esercizio possono essere applicate a qualsiasi scenario in cui un'organizzazione vuole usare modelli di intelligenza artificiale generativa per aiutare i dipendenti a essere più efficaci e produttivi.

Questo esercizio richiede circa **30** minuti.

## Effettuare il provisioning di una risorsa OpenAI di Azure

Se non è già disponibile, effettuare il provisioning di una risorsa OpenAI di Azure nella sottoscrizione di Azure.

1. Accedere al **portale di Azure** all'indirizzo `https://portal.azure.com`.
2. Creare una risorsa **OpenAI di Azure** con le impostazioni seguenti:
    - **Sottoscrizione**: *Selezionare una sottoscrizione di Azure approvata per l'accesso al servizio OpenAI di Azure*
    - **Gruppo di risorse**: *Scegliere o creare un gruppo di risorse*
    - **Area**: *Effettuare una scelta **casuale** da una delle aree seguenti*\*
        - Australia orientale
        - Canada orientale
        - Stati Uniti orientali
        - Stati Uniti orientali 2
        - Francia centrale
        - Giappone orientale
        - Stati Uniti centro-settentrionali
        - Svezia centrale
        - Svizzera settentrionale
        - Regno Unito meridionale
    - **Nome**: *nome univoco di propria scelta*
    - **Piano tariffario**: Standard S0.

    > \* Le risorse OpenAI di Azure sono vincolate dalle quote a livello di area. Le aree elencate includono la quota predefinita per i tipi di modello usati in questo esercizio. La scelta casuale di un'area riduce il rischio che una singola area raggiunga il limite di quota negli scenari in cui si condivide una sottoscrizione con altri utenti. In caso di raggiungimento di un limite di quota più avanti nell'esercizio, potrebbe essere necessario creare un'altra risorsa in un'area diversa.

3. Attendere il completamento della distribuzione. Passare quindi alla risorsa OpenAI di Azure distribuita nel portale di Azure.

## Distribuire un modello

Azure offre un portale basato sul Web denominato **portale di Azure AI Foundry**, che è possibile usare per distribuire, gestire ed esplorare i modelli. Verrà iniziata l'esplorazione di OpenAI di Azure usando il portale di Azure AI Foundry per distribuire un modello.

> **Nota**: quando si usa il portale di Azure AI Foundry, è possibile visualizzare le finestre di messaggio che suggeriscono le attività da eseguire. È possibile chiuderle e seguire i passaggi di questo esercizio.

1. Nel portale di Azure, nella pagina **Panoramica** per la risorsa OpenAI di Azure, scorrere verso il basso alla sezione **Attività iniziali** e selezionare il pulsante per passare al **portale di AI Foundry** (precedentemente Studio AI).
1. Nel portale di Azure AI Foundry, nel riquadro a sinistra, selezionare la pagina **Distribuzioni** e visualizzare le distribuzioni di modelli esistenti. Se non è già disponibile, creare una nuova distribuzione del modello **gpt-35-turbo-16k** con le impostazioni seguenti:
    - **Nome distribuzione**: *nome univoco di propria scelta*
    - **Modello**: gpt-35-turbo-16k *(se il modello 16k non è disponibile, scegliere gpt-35-turbo)*
    - **Versione del modello**: *Usare la versione predefinita*
    - **Tipo di distribuzione**: Standard
    - **Limite di velocità dei token al minuto**: 5K\*
    - **Filtro contenuto**: Predefinito
    - **Abilitare la quota dinamica**: disabilitato

    > \* Un limite di 5.000 token al minuto è più che sufficiente per completare questo esercizio, lasciando capacità ad altre persone che usano la stessa sottoscrizione.

## Usare il playground Chat

Ora che è stato distribuito un modello, è possibile usarlo per generare risposte in base ai prompt del linguaggio naturale. Il playground di *Chat* nel Portale Fonderia Azure AI offre un'interfaccia chatbot per i modelli GPT 3.5 e versioni successive.

> **Nota:** Il playground *Chat* usa l'API* ChatCompletions* anziché l'API *Completamenti* meno recente usata dal playground *Completamenti*. Il playground Completamenti viene fornito per la compatibilità con i modelli meno recenti.

1. Nella sezione **Playground** selezionare la pagina **Chat**. La pagina Playground **Chat** è costituita da una riga di pulsanti e da due pannelli principali (che possono essere disposti da destra a sinistra orizzontalmente o dall'alto verso il basso verticalmente a seconda della risoluzione dello schermo):
    - **Configurazione**: utilizzata per selezionare la distribuzione, definire il messaggio di sistema e impostare i parametri per interagire con la distribuzione.
    - **Sessione chat**: consente di inviare messaggi di chat e visualizzare le risposte.
1. In **Distribuzioni** assicurarsi che sia selezionata la distribuzione del modello gpt-35-turbo-16k.
1. Esaminare il **Messaggio di sistema** predefinito, che deve essere *Questo è un assistente di intelligenza artificiale che aiuta gli utenti a trovare informazioni.* Il messaggio di sistema è incluso nelle richieste inviate al modello e fornisce il contesto per le risposte del modello; impostando le aspettative su come un agente di intelligenza artificiale basato sul modello deve interagire con l'utente.
1. Nel pannello **Sessione chat** immettere la query utente `How can I use generative AI to help me market a new product?`

    > **Nota**: È possibile ricevere una risposta che la distribuzione dell'API non è ancora pronta. In tal caso, attendere alcuni minuti e riprovare.

1. Esaminare la risposta, notando che il modello ha generato una risposta in linguaggio naturale coesivo pertinente alla query con cui è stata richiesta.
1. Immettere la query utente `What skills do I need if I want to develop a solution to accomplish this?`.
1. Esaminare la risposta, notando che la sessione chat ha mantenuto il contesto di conversazione (quindi "questo" viene interpretato come una soluzione di intelligenza artificiale generativa per il marketing). Questa contestualizzazione viene ottenuta includendo la cronologia di conversazione recente in ogni invio di richiesta successiva, quindi la richiesta inviata al modello per la seconda query includeva la query e la risposta originali, nonché il nuovo input dell'utente.
1. Nella barra degli strumenti **Sessione chat** selezionare **Cancella chat** e verificare di voler riavviare la sessione di chat.
1. Immettere la query `Can you help me find resources to learn those skills?` ed esaminare la risposta, che deve essere una risposta in linguaggio naturale valida, ma poiché la cronologia delle chat precedente è stata persa, è probabile che la risposta sia relativa alla ricerca di risorse di competenza generica anziché essere correlate alle competenze specifiche necessarie per creare una soluzione di marketing di intelligenza artificiale generativa.

## Sperimentare messaggi di sistema, richieste ed esempi di poche inquadrature

Finora è stata eseguita una conversazione di chat con il modello in base al messaggio di sistema predefinito. È possibile personalizzare la configurazione del sistema in modo da avere un maggiore controllo sui tipi di risposte generate dal modello.

1. Nella barra degli strumenti principale selezionare gli **esempi di prompt** e usare il modello di prompt **Assistente scrittura marketing**.
1. Esaminare il nuovo messaggio di sistema, che descrive come un agente di intelligenza artificiale deve usare il modello per rispondere.
1. Nel pannello **Sessione chat** immettere la query utente `Create an advertisement for a new scrubbing brush`.
1. Esaminare la risposta, che deve includere la copia pubblicitaria per una spazzola. La copia può essere abbastanza estesa e creativa.

    In uno scenario reale, un professionista di marketing probabilmente conosce già il nome del prodotto spazzola e ha alcune idee sulle caratteristiche chiave che devono essere evidenziate in un annuncio. Per ottenere i risultati più utili da un modello di intelligenza artificiale generativa, gli utenti devono progettare le proprie richieste per includere le informazioni più pertinenti possibile.

1. Immettere il prompt `Revise the advertisement for a scrubbing brush named "Scrubadub 2000", which is made of carbon fiber and reduces cleaning times by half compared to ordinary scrubbing brushes`.
1. Esaminare la risposta, che deve prendere in considerazione le informazioni aggiuntive fornite sul prodotto spazzola.

    La risposta dovrebbe ora essere più utile, ma per avere ancora più controllo sull'output del modello, è possibile fornire uno o più *esempi di inquadrature* su cui devono essere basate le risposte.

1. Nella casella di testo **Messaggio di sistema** espandere l'elenco a discesa **Aggiungi sezione** e selezionare **Esempi**. Digitare quindi il messaggio e la risposta seguente nelle caselle designate:

    **Utente**:
    
    ```prompt
    Write an advertisement for the lightweight "Ultramop" mop, which uses patented absorbent materials to clean floors.
    ```
    
    **Assistente**:
    
    ```prompt
    Welcome to the future of cleaning!
    
    The Ultramop makes light work of even the dirtiest of floors. Thanks to its patented absorbent materials, it ensures a brilliant shine. Just look at these features:
    - Lightweight construction, making it easy to use.
    - High absorbency, enabling you to apply lots of clean soapy water to the floor.
    - Great low price.
    
    Check out this and other products on our website at www.contoso.com.
    ```

1. Usare il pulsante **Applica modifiche ** per salvare gli esempi e avviare una nuova sessione.
1. Nella sezione **Sessione chat** immettere la query utente `Create an advertisement for the Scrubadub 2000 - a new scrubbing brush made of carbon fiber that reduces cleaning time by half`.
1. Esaminare la risposta, che deve essere una nuova inserzione per "Scrubadub 2000" modellata sull'esempio "Ultramop" fornito nella configurazione del sistema.

## Sperimentare con i parametri

Si è appreso come il messaggio di sistema, gli esempi e le richieste possono aiutare a perfezionare le risposte restituite dal modello. È anche possibile usare i parametri per controllare il comportamento del modello.

1. Nel pannello **Configurazione** selezionare la scheda **Parametri** e impostare i valori dei parametri seguenti:
    - **Risposta massima**: 1.000
    - **Temperatura**: 1

1. Nella sezione **Sessione chat** usare il pulsante **Cancella chat** per reimpostare la sessione di chat. Immettere quindi la query utente `Create an advertisement for a cleaning sponge` ed esaminare la risposta. La copia dell'annuncio risultante deve includere un massimo di 1000 token di testo e includere alcuni elementi creativi, ad esempio il modello potrebbe aver inventato un nome di prodotto per la spugna e ha fatto alcune affermazioni sulle sue caratteristiche.
1. Usare il pulsante **Cancella chat** per reimpostare di nuovo la sessione di chat e quindi immettere nuovamente la stessa query di prima (`Create an advertisement for a cleaning sponge`) ed esaminare la risposta. La risposta può essere diversa dalla risposta precedente.
1. Nel pannello **Configurazione**, nella scheda **Parametri** modificare il valore del parametro **Temperatura** su 0.
1. Nella sezione **Sessione chat**, usare il pulsante **Cancella chat** per reimpostare la sessione di chat e quindi immettere nuovamente la stessa query di prima (`Create an advertisement for a cleaning sponge`) ed esaminare la risposta. Questa volta, la risposta potrebbe non essere così creativa.
1. Usare il pulsante **Cancella chat** per reimpostare la sessione di chat ancora una volta e quindi immettere nuovamente la stessa query di prima (`Create an advertisement for a cleaning sponge`) ed esaminare la risposta; che deve essere molto simile (se non identica) alla risposta precedente.

    Il parametro **Temperatura** controlla il grado di creatività del modello nella generazione di una risposta. Un valore basso restituisce una risposta coerente con una piccola variazione casuale, mentre un valore elevato incoraggia il modello ad aggiungere elementi creativi al suo output; operazione che può influire sull'accuratezza e il realismo della risposta.

## Distribuire il modello in un'app Web

Ora che sono state esaminate alcune delle funzionalità di un modello di IA generativa nel playground di Fonderia Azure AI, è possibile distribuire un'app Web di Azure per fornire un'interfaccia dell'agente di intelligenza artificiale di base tramite cui gli utenti possono chattare con il modello.

> **Nota**: per alcuni utenti, la distribuzione nell'app Web non può essere eseguita a causa di un bug nel modello in Studio. In tal caso, ignorare questa sezione.

1. In alto a destra nella pagina del Playground **Chat**, nel menu **Distribuisci in**, selezionare **Nuova app Web**.
1. Nella finestra di dialogo **Distribuisci in un'app Web** creare una nuova app Web con le impostazioni seguenti:
    - **Nome**: *un nome univoco*
    - **Sottoscrizione**: *la sottoscrizione di Azure usata*
    - **Gruppo di risorse**: *Il gruppo di risorse in cui è stato effettuato il provisioning della risorsa OpenAI di Azure*
    - **Posizioni**: *L'area in cui è stato effettuato il provisioning della risorsa OpenAI di Azure*
    - **Piano tariffario**: Gratuito (F1): *Se non è disponibile, selezionare Basic (B1)*
    - **Abilitare la cronologia delle chat nell'app Web**: <u>non</u> selezionata
    - **Riconosco che le app Web comportano l'utilizzo dell'account**: selezionato
1. Distribuire la nuova app Web e attendere il completamento della distribuzione (che potrebbe richiedere almeno 10 minuti)
1. Dopo che l'app Web è stata distribuita correttamente, usare il pulsante in alto a destra nella pagina Playground **Chat** per avviare l'app Web. L'attivazione potrebbe richiedere alcuni minuti. Se richiesto, accettare la richiesta di autorizzazioni.
1. Nell'app Web immettere il messaggio di chat seguente:

    ```prompt
    Write an advertisement for the new "WonderWipe" cloth that attracts dust particulates and can be used to clean any household surface.
    ```

1. Rivedere la risposta.

    > **Nota**: Il *modello* è stato distribuito in un'app Web, ma questa distribuzione non include le impostazioni di sistema e i parametri impostati nel playground; pertanto la risposta potrebbe non riflettere gli esempi specificati nel playground. In uno scenario reale, aggiungere la logica all'applicazione per modificare il prompt in modo che includa i dati contestuali appropriati per i tipi di risposta che si desidera generare. Questo tipo di personalizzazione esula dall'ambito di questo esercizio introduttivo, ma è possibile ottenere informazioni sulle tecniche di progettazione richieste e sulle API OpenAI di Azure in altri esercizi e documentazione del prodotto.

1. Al termine dell'esperimento con il modello nell'app Web, chiudere la scheda dell'app Web nel browser per tornare al Portale Fonderia Azure AI.

## Eseguire la pulizia

Terminato l'utilizzo della risorsa OpenAI di Azure, ricordarsi di eliminare la distribuzione o l'intera risorsa nel **portale di Azure** all'indirizzo `https://portal.azure.com`.
