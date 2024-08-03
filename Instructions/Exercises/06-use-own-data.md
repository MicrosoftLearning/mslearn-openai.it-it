---
lab:
  title: Implementare RAG (Retrieval Augmented Generation) con il Servizio OpenAI di Azure
---

# Implementare RAG (Retrieval Augmented Generation) con il Servizio OpenAI di Azure

Il servizio OpenAI di Azure consente di usare i propri dati con l'intelligenza del modello linguistico di grandi dimensioni (LLM) sottostante. È possibile limitare il modello solo all'uso dei dati per argomenti pertinenti o combinarlo con i risultati del modello con training preliminare.

Nello scenario per questo esercizio si agirà nel ruolo di sviluppatore di software che lavora per l'agenzia di viaggi di Margie. Si esaminerà come usare l'IA generativa per semplificare e rendere più efficienti le attività di codifica. Le tecniche usate nell'esercizio possono essere applicate ad altri file di codice, linguaggi di programmazione e casi d'uso.

Questo esercizio richiederà circa **20** minuti.

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

OpenAI di Azure offre un portale basato sul Web denominato **Azure OpenAI Studio**, che è possibile usare per distribuire, gestire ed esplorare i modelli. Si inizierà l'esplorazione di OpenAI di Azure usando Azure OpenAI Studio per distribuire un modello.

1. Nella pagina **Panoramica** della risorsa OpenAI di Azure, usare il pulsante **Passa a Azure OpenAI Studio** per aprire Azure OpenAI Studio in una nuova scheda del browser.
2. Nella pagina **Distribuzioni** di Azure OpenAI Studio, visualizzare le distribuzioni di modelli esistenti. Se non è già disponibile, creare una nuova distribuzione del modello **gpt-35-turbo-16k** con le impostazioni seguenti:
    - **Nome distribuzione**: *nome univoco di propria scelta*
    - **Modello**: gpt-35-turbo-16k *(se il modello 16k non è disponibile, scegliere gpt-35-turbo)*
    - **Versione modello**: Aggiornamento automatico per impostazione predefinita
    - **Tipo di distribuzione**: Standard
    - **Limite di velocità dei token al minuto**: 5K\*
    - **Filtro contenuto**: Predefinito
    - **Abilitare la quota dinamica**: Abilitato

    > \* Un limite di 5.000 token al minuto è più che sufficiente per completare questo esercizio, lasciando capacità ad altre persone che usano la stessa sottoscrizione.

## Osservare il comportamento normale della chat senza aggiungere i propri dati

Prima di connettere OpenAI di Azure ai propri dati, osservare innanzitutto come il modello di base risponde alle query senza dati di base.

1. In **Azure OpenAI Studio** in `https://oai.azure.com`, nella sezione **Playground** selezionare la pagina **Chat**. La pagina Playground **Chat** è costituita da tre sezioni principali:
    - **Installazione**: usato per impostare il contesto per le risposte del modello.
    - **Sessione chat**: consente di inviare messaggi di chat e visualizzare le risposte.
    - **Configurazione**: usata per configurare le impostazioni per la distribuzione del modello.
2. Nella sezione **Configurazione** verificare che sia selezionata la distribuzione del modello.
3. Nell'area **Installazione**, selezionare il modello di messaggio di sistema predefinito per impostare il contesto per la sessione di chat. Il messaggio di sistema predefinito è *Si è un assistente di intelligenza artificiale che consente agli utenti di trovare informazioni*.
4. Nella **sessione di chat** inviare le query seguenti e verificare le risposte:

    ```prompt
    I'd like to take a trip to New York. Where should I stay?
    ```

    ```prompt
    What are some facts about New York?
    ```

    Provare a fare domande simili sul turismo e sui luoghi in cui soggiornare per altre località che verranno incluse nei dati di base, come Londra o San Francisco. È probabile che si ottengano risposte complete su aree o quartieri e alcuni fatti generali sulla città.

## Connettere i dati nel playground della chat

A questo punto si aggiungeranno alcuni dati per una società fittizia di agenti di viaggio denominata *Margie's Travel*. Si vedrà quindi come risponde il modello OpenAI di Azure quando si usano le brochure di Margie's Travel come dati di base.

1. In una nuova scheda del browser scaricare un archivio di dati della brochure da `https://aka.ms/own-data-brochures`. Estrarre le brochure in una cartella sul PC.
1. Nel playground **Chat** di Azure OpenAI Studio, nella sezione **Installazione** selezionare **Aggiungi i dati**.
1. Selezionare **Aggiungi un'origine dati** e scegliere **Carica file**.
1. Sarà necessario creare un account di archiviazione e una risorsa di Azure AI Search. Nell'elenco a discesa per la risorsa di archiviazione selezionare **Crea una nuova risorsa di archiviazione BLOB di Azure** e creare un account di archiviazione con le impostazioni seguenti. Per tutto ciò che non è specificato viene lasciata l'impostazione predefinita.

    - **Sottoscrizione**: *la sottoscrizione di Azure usata*
    - **Gruppo di risorse**: *Selezionare lo stesso gruppo di risorse della risorsa di OpenAI di Azure*
    - **Nome account di archiviazione**: *immettere un nome univoco*
    - **Area**: *Selezionare la stessa area della risorsa di OpenAI di Azure*
    - **Ridondanza**: Archiviazione con ridondanza locale

1. Durante la creazione della risorsa dell'account di archiviazione, tornare ad Azure OpenAI Studio e selezionare **Crea una nuova risorsa di Azure AI Search** con le impostazioni seguenti. Per tutto ciò che non è specificato viene lasciata l'impostazione predefinita.

    - **Sottoscrizione**: *la sottoscrizione di Azure usata*
    - **Gruppo di risorse**: *Selezionare lo stesso gruppo di risorse della risorsa di OpenAI di Azure*
    - **Nome del servizio**: *immettere un nome univoco*
    - **Località**: *Selezionare la stessa località della risorsa di OpenAI di Azure*
    - **Piano tariffario**: Basic.

1. Attendere che la risorsa di ricerca venga distribuita, quindi tornare ad Azure AI Studio.
1. In **Aggiungi dati** immettere i valori seguenti per l'origine dati, quindi selezionare **Avanti**.

    - **Seleziona origine dati**: Caricare i file
    - **Sottoscrizione**: sottoscrizione di Azure.
    - **Selezionare la risorsa di archiviazione BLOB di Azure**: *Usare il pulsante **Aggiorna** per ripopolare l'elenco e quindi scegliere la risorsa di archiviazione creata*
        - Attivare CORS quando richiesto
    - **Selezionare la risorsa di Azure AI Search**: *Usare il pulsante **Aggiorna** per ripopolare l'elenco e quindi scegliere la risorsa di ricerca creata*
    - **Immettere il nome dell'indice**: `margiestravel`
    - **Aggiungere la ricerca vettoriale a questa risorsa di ricerca**: deselezionato
    - **Sono consapevole che la connessione a un account Azure AI Search comporterà l'utilizzo del mio account**: selezionato

1. Nella pagina **Carica file** caricare i PDF scaricati e quindi selezionare **Avanti**.
1. Nella pagina **Gestione dati** selezionare **Parola chiave** nell'elenco a discesa come tipo di ricerca e quindi selezionare **Avanti**.
1. Nella pagina **Connessione dati** selezionare **chiave API**.
1. Nella pagina **Rivedere e completare** selezionare **Salva e chiudi** per aggiungere i dati. Questa operazione potrebbe richiedere alcuni minuti, durante la quale è necessario lasciare aperta la finestra. Al termine, verrà visualizzata l'origine dati, la risorsa di ricerca e l'indice specificati nella sezione **Installazione**.

    > **Suggerimento**: In alcuni casi la connessione tra il nuovo indice di ricerca e Azure OpenAI Studio richiede troppo tempo. Se si è atteso qualche minuto e la connessione non è ancora avvenuta, controllare le risorse di AI Search nel portale Azure. Se viene visualizzato l'indice completato, è possibile scollegare la connessione dati in Azure OpenAI Studio e aggiungerla nuovamente specificando un'origine dati di Azure AI Search e selezionando il nuovo indice.

## Chattare con un modello basato sui propri dati

Dopo aver aggiunto i dati, porre le stesse domande fatte in precedenza e osservare come differisce la risposta.

```prompt
I'd like to take a trip to New York. Where should I stay?
```

```prompt
What are some facts about New York?
```

Si noterà una risposta molto diversa questa volta, con specifiche su alcuni hotel e una menzione di Margie's Travel, oltre a riferimenti alla provenienza delle informazioni fornite. Se si apre il riferimento PDF elencato nella risposta, verranno visualizzati gli stessi hotel del modello fornito.

Provare a chiedere informazioni su altre città incluse nei dati di base, ovvero Dubai, Las Vegas, Londra e San Francisco.

> **Nota**: **Aggiungi i dati** è ancora in anteprima e potrebbe non comportarsi sempre come previsto per questa funzionalità, ad esempio fornendo il riferimento non corretto per una città non inclusa nei dati di base.

## Connettere l'app ai propri dati

Si esaminerà ora come connettere l'app per usare i propri dati.

### Prepararsi allo sviluppo di un'app in Visual Studio Code

Si esaminerà ora l'uso dei propri dati in un'app che usa l'SDK del servizio OpenAI di Azure. L'app verrà sviluppata usando Visual Studio Code. I file di codice per l'app sono stati forniti in un repository GitHub.

> **Suggerimento**: Se il repository **mslearn-openai** è già stato clonato, aprirlo in Visual Studio Code. In caso contrario, eseguire i passaggi seguenti per clonarlo nell'ambiente di sviluppo.

1. Avviare Visual Studio Code.
2. Aprire il riquadro comandi (MAIUSC+CTRL+P) ed eseguire un comando **Git: Clone** per clonare il repository `https://github.com/MicrosoftLearning/mslearn-openai` in una cartella locale. Non è importante usare una cartella specifica.
3. Dopo la clonazione del repository, aprire la cartella in Visual Studio Code.

    > **Nota**: Se Visual Studio Code visualizza un messaggio popup per chiedere se si considera attendibile il codice che si apre, fare clic sull'opzione **Sì, considero attendibili gli autori** nel popup.

4. Attendere il completamento dell'installazione di file aggiuntivi per supportare i progetti in codice C# nel repository.

    > **Nota**: se viene richiesto di aggiungere gli asset necessari per la compilazione e il debug, selezionare **Non adesso**.

## Configurare l'applicazione

Sono state fornite applicazioni sia per C# sia per Python ed entrambe le app presentano la stessa funzionalità. Prima di tutto, verranno completate alcune parti chiave dell'applicazione per consentire l'uso della risorsa OpenAI di Azure.

1. Nel riquadro **Explorer** di Visual Studio Code, passare alla cartella **Labfiles/06-use-own-data** ed espandere la cartella **CSharp** o **Python** a seconda del linguaggio scelto. Ogni cartella contiene i file specifici del linguaggio per un'app in cui si intende integrare la funzionalità OpenAI di Azure.
2. Fare clic con il pulsante destro del mouse sulla cartella **CSharp** o **Python** contenente i file di codice, quindi aprire un terminale integrato. Installare quindi il pacchetto SDK di OpenAI eseguendo il comando appropriato per il linguaggio scelto:

    **C#:**

    ```
    dotnet add package Azure.AI.OpenAI --version 1.0.0-beta.14
    ```

    **Python**:

    ```
    pip install openai==1.13.3
    ```

3. Nel riquadro **Esplora risorse**, nella cartella **CSharp** o **Python**, aprire il file di configurazione per il linguaggio preferita

    - **C#**: appsettings.json
    - **Python**: .env
    
4. Aggiornare i valori di configurazione in modo da includere:
    - L'**endpoint** e una **chiave** della risorsa OpenAI di Azure creata (disponibile nella pagina **Chiavi ed endpoint** per la risorsa OpenAI di Azure nel portale di Azure)
    - Il **nome della distribuzione** specificato per la distribuzione del modello (disponibile nella pagina **Distribuzioni** in Azure OpenAI Studio).
    - L'endpoint per il servizio di ricerca (il valore **Url** nella pagina di panoramica per la risorsa di ricerca nel portale di Azure).
    - Una **chiave** per la risorsa di ricerca (disponibile nella pagina **Chiavi** per la risorsa di ricerca nel portale di Azure. È possibile usare una delle chiavi amministratore)
    - Nome dell'indice di ricerca (che deve essere `margiestravel`).
1. Salvare il file di configurazione.

### Aggiungere codice per usare il servizio OpenAI di Azure

A questo punto, è possibile usare l'SDK di OpenAI di Azure per usare il modello distribuito.

1. Nel riquadro **Esplora risorse**, nella cartella **CSharp** o **Python**, aprire il file di codice per il linguaggio preferito e sostituire il commento ***Configurare l'origine dati*** con il codice per aggiungere la libreria SDK di OpenAI di Azure:

    **C#**: ownData.cs

    ```csharp
    // Configure your data source
    AzureSearchChatExtensionConfiguration ownDataConfig = new()
    {
            SearchEndpoint = new Uri(azureSearchEndpoint),
            Authentication = new OnYourDataApiKeyAuthenticationOptions(azureSearchKey),
            IndexName = azureSearchIndex
    };
    ```

    **Python**: ownData.py

    ```python
    # Configure your data source
    extension_config = dict(dataSources = [  
            { 
                "type": "AzureCognitiveSearch", 
                "parameters": { 
                    "endpoint":azure_search_endpoint, 
                    "key": azure_search_key, 
                    "indexName": azure_search_index,
                }
            }]
        )
    ```

2. Esaminare il resto del codice, notando l'uso delle *estensioni* nel corpo della richiesta usato per fornire informazioni sulle impostazioni dell'origine dati.

3. Salvare le modifiche apportate al file di codice.

## Eseguire l'applicazione

Ora che l'app è stata configurata, eseguirla per inviare la richiesta al modello e osservare la risposta. Si noterà che l'unica differenza tra le diverse opzioni è il contenuto del prompt, tutti gli altri parametri (ad esempio il numero di token e la temperatura) rimangono invariati per ogni richiesta.

1. Nel riquadro del terminale interattivo, accertarsi che il contesto della cartella sia la cartella per la lingua dell’interfaccia utente. Immettere, quindi, il comando seguente per eseguire l'applicazione.

    - **C#**: `dotnet run`
    - **Python**: `python ownData.py`

    > **Suggerimento**: È possibile usare l'icona **Ingrandisci dimensioni pannello** (**^**) nella barra degli strumenti del terminale per visualizzare altro testo della console.

2. Esaminare la risposta alla richiesta `Tell me about London`, che deve includere una risposta e alcuni dettagli dei dati usati per creare la richiesta, ottenuta dal servizio di ricerca.

    > **Suggerimento**: Se si vuole visualizzare le citazioni dell'indice di ricerca, impostare la variabile ***mostra citazioni*** nella parte superiore del file di codice su **true**.

## Eseguire la pulizia

Terminato l'utilizzo della risorsa OpenAI di Azure, ricordarsi di eliminare la risorsa nel **portale di Azure** all'indirizzo `https://portal.azure.com`. Assicurarsi di includere anche l'account di archiviazione e la risorsa di ricerca, in quanto possono comportare un costo relativamente elevato.
