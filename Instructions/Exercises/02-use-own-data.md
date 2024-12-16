---
lab:
  title: Implementare RAG (Retrieval Augmented Generation) con il Servizio OpenAI di Azure
---

# Implementare RAG (Retrieval Augmented Generation) con il Servizio OpenAI di Azure

Il servizio OpenAI di Azure consente di usare i propri dati con l'intelligenza del modello linguistico di grandi dimensioni (LLM) sottostante. È possibile limitare il modello solo all'uso dei dati per argomenti pertinenti o combinarlo con i risultati del modello con training preliminare.

Nello scenario per questo esercizio si agirà nel ruolo di sviluppatore di software che lavora per l'agenzia di viaggi di Margie. Si esaminerà l'uso di Azure AI Search per indicizzare i propri dati e usarli con Azure OpenAI per aumentare le richieste.

Questo esercizio richiederà circa **30** minuti.

## Effettuare il provisioning delle risorse di Azure

Per completare questo esercizio, sarà necessario disporre di:

- Risorsa Azure OpenAI.
- Una risorsa di Azure AI Search.
- Una risorsa Account di archiviazione di Azure.

1. Accedere al **portale di Azure** all'indirizzo `https://portal.azure.com`.
2. Creare una risorsa **OpenAI di Azure** con le impostazioni seguenti:
    - **Sottoscrizione**: *Selezionare una sottoscrizione di Azure approvata per l'accesso al servizio OpenAI di Azure*
    - **Gruppo di risorse**: *Scegliere o creare un gruppo di risorse*
    - **Area**: *Effettuare una scelta **casuale** da una delle aree seguenti*\*
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

3. Durante il provisioning della risorsa Azure OpenAI, creare una risorsa **Azure AI Search** con le impostazioni seguenti:
    - **Sottoscrizione**: *la sottoscrizione in cui è stato effettuato il provisioning della risorsa Azure OpenAI*
    - **Gruppo di risorse**: *Il gruppo di risorse in cui è stato effettuato il provisioning della risorsa OpenAI di Azure*
    - **Nome del servizio**: *nome univoco di propria scelta*
    - **Posizione**: *l'area in cui è stato effettuato il provisioning della risorsa Azure OpenAI*
    - **Piano tariffario**: Basic.
4. Durante il provisioning della risorsa di Azure AI Search, creare una risorsa **Account di archiviazione** con le impostazioni seguenti:
    - **Sottoscrizione**: *la sottoscrizione in cui è stato effettuato il provisioning della risorsa Azure OpenAI*
    - **Gruppo di risorse**: *Il gruppo di risorse in cui è stato effettuato il provisioning della risorsa OpenAI di Azure*
    - **Nome dell'account di archiviazione**: *nome univoco di propria scelta*
    - **Area**: *l'area in cui è stato effettuato il provisioning della risorsa Azure OpenAI*
    - **Servizio primario**: Archiviazione BLOB di Azure o Azure Data Lake Storage Gen 2
    - **Prestazioni**: standard
    - **Ridondanza**: archiviazione con ridondanza locale
5. Dopo che tutte e tre le risorse sono state distribuite correttamente nella sottoscrizione di Azure, esaminarle nel portale di Azure e raccogliere le informazioni seguenti (che saranno necessarie più avanti nell'esercizio):
    - L'**endpoint** e una **chiave** della risorsa OpenAI di Azure creata (disponibile nella pagina **Chiavi ed endpoint** per la risorsa OpenAI di Azure nel portale di Azure)
    - L'endpoint per il servizio Azure AI Search (il valore **URL** nella pagina di panoramica per la risorsa di Azure AI Search nel portale di Azure).
    - Una **chiave di amministrazione primaria** per la risorsa Azure AI Search (disponibile nella pagina **Chiavi** della risorsa Azure AI Search nel portale di Azure).

## Caricare i dati

Verranno radicati i prompt usati con un modello di intelligenza artificiale generativa usando i propri dati. In questo esercizio i dati sono costituiti da una raccolta di brochure di viaggio della società fittizia *Margie's Travel*.

1. In una nuova scheda del browser scaricare un archivio di dati della brochure da `https://aka.ms/own-data-brochures`. Estrarre le brochure in una cartella sul PC.
1. Accedere al portale di Azure, passare all'account di archiviazione e visualizzare la pagina **Browser archiviazione**.
1. Selezionare **Contenitori BLOB** e quindi aggiungere un nuovo contenitore denominato `margies-travel`.
1. Selezionare il contenitore **margies-travel** e quindi caricare le brochure in formato PDF estratte in precedenza nella cartella radice del contenitore BLOB.

## Distribuire modelli di intelligenza artificiale

In questo esercizio si useranno due modelli di intelligenza artificiale:

- Un modello di incorporamento di testo per *vettorizzare* il testo nelle brochure, così da poterlo indicizzare in modo efficiente per l'uso nel radicamento dei prompt.
- Un modello GPT che l'applicazione può usare per generare risposte ai prompt basate sui propri dati.


## Distribuire un modello

Successivamente, verrà distribuita una risorsa modello di Azure OpenAI dall'interfaccia della riga di comando. Nell portale di Azure, selezionare **l'icona Cloud Shell** nella barra dei menu in alto e assicurarsi che il terminale sia impostato su **Bash**. Fare riferimento a questo esempio e sostituire le seguenti variabili con i valori:

```dotnetcli
az cognitiveservices account deployment create \
   -g *your resource group* \
   -n *your Open AI resource* \
   --deployment-name text-embedding-ada-002 \
   --model-name text-embedding-ada-002 \
   --model-version "2"  \
   --model-format OpenAI \
   --sku-name "Standard" \
   --sku-capacity 5
```

    > \* Sku-capacity is measured in thousands of tokens per minute. A rate limit of 5,000 tokens per minute is more than adequate to complete this exercise while leaving capacity for other people using the same subscription.


Dopo aver distribuito il modello di incorporamento del testo, creare una nuova distribuzione del modello **gpt-35-turbo-16k** con le seguenti impostazioni:

```dotnetcli
az cognitiveservices account deployment create \
   -g *your resource group* \
   -n *your Open AI resource* \
   --deployment-name gpt-35-turbo-16k \
   --model-name gpt-35-turbo-16k \
   --model-version "0125"  \
   --model-format OpenAI \
   --sku-name "Standard" \
   --sku-capacity 5
```

    > \* Sku-capacity is measured in thousands of tokens per minute. A rate limit of 5,000 tokens per minute is more than adequate to complete this exercise while leaving capacity for other people using the same subscription.

## Creare un indice

Per semplificare l'uso dei propri dati in una richiesta, questi verranno indicizzati usando Azure AI Search. Il modello di incorporamento del testo verrà usato per *vettorizzare* i dati testuali, ossia per rappresentare ogni token di testo presente nell'indice attraverso vettori numerici. Questo approccio garantisce la compatibilità con il formato di rappresentazione del testo adottato dai modelli di intelligenza artificiale generativa.

1. Nel portale di Azure passare alla risorsa Azure AI Search.
1. Nella pagina **Panoramica** selezionare **Importa e vettorizza dati**.
1. Nella pagina **Configura la connessione dati** selezionare **Archiviazione BLOB di Azure** e configurare l'origine dati con le impostazioni seguenti:
    - **Sottoscrizione**: sottoscrizione di Azure in cui è stato effettuato il provisioning dell'account di archiviazione.
    - **Account di archiviazione BLOB**: account di archiviazione creato in precedenza.
    - **Contenitore BLOB**: margies-travel
    - **Cartella BLOB**: *lasciare vuoto*
    - **Abilitare il rilevamento delle eliminazioni**: non selezionato
    - **Autenticazione con identità gestita**: non selezionato
1. Nella pagina **Vettorizzare il testo** selezionare le impostazioni seguenti:
    - **Tipologia**: Azure OpenAI
    - **Sottoscrizione**: sottoscrizione di Azure in cui è stato effettuato il provisioning del Servizio OpenAI di Azure.
    - **Servizio OpenAI di Azure**: risorsa del Servizio OpenAI di Azure
    - **Distribuzione del modello**: text-embedding-ada-002
    - **Tipo di autenticazione**: chiave API
    - **Sono consapevole che la connessione a un Servizio OpenAI di Azure comporterà costi aggiuntivi per il mio account**: selezionato
1. Nella pagina successiva <u>non</u> selezionare l'opzione per vettorizzare le immagini o estrarre i dati con competenze di intelligenza artificiale.
1. Nella pagina successiva abilitare la classificazione semantica e pianificare l'esecuzione dell'indicizzatore una sola volta.
1. Nella pagina finale impostare il **prefisso nome oggetti** su `margies-index` e quindi creare l'indice.

## Prepararsi allo sviluppo di un'app in Visual Studio Code

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

1. Nel riquadro **Explorer** di Visual Studio Code, passare alla cartella **Labfiles/02-use-own-data** ed espandere la cartella **CSharp** o **Python** a seconda del linguaggio scelto. Ogni cartella contiene i file specifici del linguaggio per un'app in cui si intende integrare la funzionalità OpenAI di Azure.
2. Fare clic con il pulsante destro del mouse sulla cartella **CSharp** o **Python** contenente i file di codice, quindi aprire un terminale integrato. Installare quindi il pacchetto SDK di OpenAI eseguendo il comando appropriato per il linguaggio scelto:

    **C#:**

    ```
    dotnet add package Azure.AI.OpenAI --version 1.0.0-beta.17
    ```

    **Python**:

    ```
    pip install openai==1.54.3
    ```

3. Nel riquadro **Esplora risorse**, nella cartella **CSharp** o **Python**, aprire il file di configurazione per il linguaggio preferita

    - **C#**: appsettings.json
    - **Python**: .env
    
4. Aggiornare i valori di configurazione in modo da includere:
    - L'**endpoint** e una **chiave** della risorsa OpenAI di Azure creata (disponibile nella pagina **Chiavi ed endpoint** per la risorsa OpenAI di Azure nel portale di Azure)
    - Il **nome della distribuzione** specificato per la distribuzione del modello gpt-35-turbo (dovrebbe essere `gpt-35-turbo-16k`.
    - L'endpoint per il servizio di ricerca (il valore **Url** nella pagina di panoramica per la risorsa di ricerca nel portale di Azure).
    - Una **chiave** per la risorsa di ricerca (disponibile nella pagina **Chiavi** per la risorsa di ricerca nel portale di Azure. È possibile usare una delle chiavi amministratore)
    - Nome dell'indice di ricerca (che deve essere `margies-index`).
5. Salvare il file di configurazione.

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
text = input('\nEnter a question:\n')

completion = client.chat.completions.create(
    model=deployment,
    messages=[
        {
            "role": "user",
            "content": text,
        },
    ],
    extra_body={
        "data_sources":[
            {
                "type": "azure_search",
                "parameters": {
                    "endpoint": os.environ["AZURE_SEARCH_ENDPOINT"],
                    "index_name": os.environ["AZURE_SEARCH_INDEX"],
                    "authentication": {
                        "type": "api_key",
                        "key": os.environ["AZURE_SEARCH_KEY"],
                    }
                }
            }
        ],
    }
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

Terminato l'utilizzo della risorsa Azure OpenAI, ricordarsi di eliminare le risorse nel **portale di Azure** all'indirizzo `https://portal.azure.com`. Assicurarsi di includere anche l'account di archiviazione e la risorsa di ricerca, in quanto possono comportare un costo relativamente elevato.
