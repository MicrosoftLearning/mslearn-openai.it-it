---
lab:
  title: Usare gli SDK OpenAI di Azure nell'app
---

# Usare le API OpenAI di Azure nell'app

Con il Servizio OpenAI di Azure, gli sviluppatori possono creare chatbot, modelli linguistici e altre applicazioni che eccellono nel comprendere il linguaggio umano naturale. OpenAI di Azure fornisce l'accesso ai modelli di intelligenza artificiale con training preliminare, oltre a una suite di API e strumenti per personalizzare e ottimizzare questi modelli per soddisfare i requisiti specifici dell'applicazione. In questo esercizio si apprenderà come distribuire un modello in OpenAI di Azure e usarlo nella propria applicazione.

Nello scenario per questo esercizio si eseguirà il ruolo di uno sviluppatore di software a cui è stato richiesto di implementare un'app in grado di usare l'intelligenza artificiale generativa per fornire consigli per le escursioni. Le tecniche usate nell'esercizio possono essere applicate a qualsiasi app che vuole usare le API OpenAI di Azure.

Questo esercizio richiederà circa **30** minuti.

## Effettuare il provisioning di una risorsa OpenAI di Azure

Se non è già disponibile, effettuare il provisioning di una risorsa OpenAI di Azure nella sottoscrizione di Azure.

1. Accedere al **portale di Azure** all'indirizzo `https://portal.azure.com`.
2. Creare una risorsa **OpenAI di Azure** con le impostazioni seguenti:
    - **Sottoscrizione**: *Selezionare una sottoscrizione di Azure approvata per l'accesso al Servizio OpenAI di Azure*
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
    - **Modello**: gpt-35-turbo-16k *(se il modello 16k non è disponibile, scegliere gpt-35-turbo)*
    - **Versione modello**: Aggiornamento automatico per impostazione predefinita
    - **Nome distribuzione**: *Nome univoco a propria scelta. Questo nome verrà utilizzato più avanti nel lab.*
    - **Opzioni avanzate**
        - **Filtro contenuto**: Predefinito
        - **Tipo di distribuzione**: Standard
        - **Limite di velocità dei token al minuto**: 5K\*
        - **Abilitare la quota dinamica**: Abilitato

    > \* Un limite di 5.000 token al minuto è più che sufficiente per completare questo esercizio, lasciando capacità ad altre persone che usano la stessa sottoscrizione.

## Prepararsi allo sviluppo di un'app in Visual Studio Code

Si svilupperà l'app OpenAI di Azure usando Visual Studio Code. I file di codice per l'app sono stati forniti in un repository GitHub.

> **Suggerimento**: Se il repository **mslearn-openai** è già stato clonato, aprirlo in Visual Studio Code. In caso contrario, eseguire i passaggi seguenti per clonarlo nell'ambiente di sviluppo.

1. Avviare Visual Studio Code.
2. Aprire il riquadro comandi (MAIUSC+CTRL+P) ed eseguire un comando **Git: Clone** per clonare il repository `https://github.com/MicrosoftLearning/mslearn-openai` in una cartella locale. Non è importante usare una cartella specifica.
3. Dopo la clonazione del repository, aprire la cartella in Visual Studio Code.

    > **Nota**: Se Visual Studio Code visualizza un messaggio popup per chiedere se si considera attendibile il codice che si apre, fare clic sull'opzione **Sì, considero attendibili gli autori** nel popup.

4. Attendere il completamento dell'installazione di file aggiuntivi per supportare i progetti in codice C# nel repository.

    > **Nota**: se viene richiesto di aggiungere gli asset necessari per la compilazione e il debug, selezionare **Non adesso**.

## Configurare l'applicazione

Sono state fornite applicazioni sia per C# che per Python. Entrambe le app presentano la stessa funzionalità. Prima di tutto, verranno completate alcune parti chiave dell'applicazione per consentire l'uso della risorsa OpenAI di Azure.

1. Nel riquadro **Esplora risorse** di Visual Studio Code, passare alla cartella **Labfiles/02-azure-openai-api** ed espandere la cartella **CSharp** o **Python** a seconda del linguaggio scelto. Ogni cartella contiene i file specifici del linguaggio per un’app in cui si intende integrare la funzionalità OpenAI di Azure.
2. Fare clic con il pulsante destro del mouse sulla cartella **CSharp** o **Python** contenente i file di codice, quindi aprire un terminale integrato. Installare quindi il pacchetto SDK di OpenAI eseguendo il comando appropriato per il linguaggio scelto:

    **C#:**

    ```
    dotnet add package Azure.AI.OpenAI --version 1.0.0-beta.14
    ```

    **Python**:

    ```
    pip install openai==1.13.3
    ```

3. Nel riquadro **Esplora risorse**, nella cartella **CSharp** o **Python**, aprire il file di configurazione per la lingua preferita

    - **C#**: appsettings.json
    - **Python**: .env
    
4. Aggiornare i valori di configurazione in modo da includere:
    - L'**endpoint** e una **chiave** della risorsa OpenAI di Azure creata (disponibile nella pagina **Chiavi ed endpoint** per la risorsa OpenAI di Azure nel portale di Azure)
    - **Nome della distribuzione** specificato per la distribuzione del modello (disponibile nella pagina **Distribuzioni** in Azure OpenAI Studio).
5. Salvare il file di configurazione.

## Aggiungere codice per usare il Servizio OpenAI di Azure

A questo punto, è possibile usare l'SDK di OpenAI di Azure per usare il modello distribuito.

1. Nel riquadro **Esplora risorse**, nella cartella **CSharp** o **Python**, aprire il file di codice per il linguaggio preferito e sostituire il commento ***Aggiungi pacchetto OpenAI di Azure*** con il codice per aggiungere la libreria SDK di OpenAI di Azure:

    **C#**: Program.cs

    ```csharp
    // Add Azure OpenAI package
    using Azure.AI.OpenAI;
    ```
    
    **Python**: test-openai-model.py
    
    ```python
    # Add Azure OpenAI package
    from openai import AzureOpenAI
    ```

1. Nel codice dell'applicazione per la lingua sostituire il commento ***Inizializzare il client OpenAI di Azure*** con il codice seguente per inizializzare il client e definire il messaggio di sistema.

    **C#**: Program.cs

    ```csharp
    // Initialize the Azure OpenAI client
    OpenAIClient client = new OpenAIClient(new Uri(oaiEndpoint), new AzureKeyCredential(oaiKey));
    
    // System message to provide context to the model
    string systemMessage = "I am a hiking enthusiast named Forest who helps people discover hikes in their area. If no area is specified, I will default to near Rainier National Park. I will then provide three suggestions for nearby hikes that vary in length. I will also share an interesting fact about the local nature on the hikes when making a recommendation.";
    ```

    **Python**: test-openai-model.py

    ```python
    # Initialize the Azure OpenAI client
    client = AzureOpenAI(
            azure_endpoint = azure_oai_endpoint, 
            api_key=azure_oai_key,  
            api_version="2024-02-15-preview"
            )
    
    # Create a system message
    system_message = """I am a hiking enthusiast named Forest who helps people discover hikes in their area. 
        If no area is specified, I will default to near Rainier National Park. 
        I will then provide three suggestions for nearby hikes that vary in length. 
        I will also share an interesting fact about the local nature on the hikes when making a recommendation.
        """
    ```

1. Sostituire il commento ***Aggiungi codice per inviare la richiesta*** con il codice necessario per la compilazione della richiesta; specificando i vari parametri per il modello, ad esempio `messages` e `temperature`.

    **C#**: Program.cs

    ```csharp
    // Add code to send request...
    // Build completion options object
    ChatCompletionsOptions chatCompletionsOptions = new ChatCompletionsOptions()
    {
        Messages =
        {
            new ChatRequestSystemMessage(systemMessage),
            new ChatRequestUserMessage(inputText),
        },
        MaxTokens = 400,
        Temperature = 0.7f,
        DeploymentName = oaiDeploymentName
    };

    // Send request to Azure OpenAI model
    ChatCompletions response = client.GetChatCompletions(chatCompletionsOptions);

    // Print the response
    string completion = response.Choices[0].Message.Content;
    Console.WriteLine("Response: " + completion + "\n");
    ```

    **Python**: test-openai-model.py

    ```python
    # Add code to send request...
    # Send request to Azure OpenAI model
    response = client.chat.completions.create(
        model=azure_oai_deployment,
        temperature=0.7,
        max_tokens=400,
        messages=[
            {"role": "system", "content": system_message},
            {"role": "user", "content": input_text}
        ]
    )
    generated_text = response.choices[0].message.content

    # Print the response
    print("Response: " + generated_text + "\n")
    ```

1. Salvare le modifiche apportate al file di codice.

## Testare l'applicazione

Ora che l'app è stata configurata, eseguirla per inviare la richiesta al modello e osservare la risposta.

1. Nel riquadro del terminale interattivo, accertarsi che il contesto della cartella sia la cartella per la lingua dell’interfaccia utente. Immettere, quindi, il comando seguente per eseguire l'applicazione.

    - **C#**: `dotnet run`
    - **Python**: `python test-openai-model.py`

    > **Suggerimento**: È possibile usare l'icona **Ingrandisci dimensioni pannello** (**^**) nella barra degli strumenti del terminale per visualizzare altro testo della console.

1. Quando richiesto, immettere il testo `What hike should I do near Rainier?`.
1. Osservare l'output, tenendo presente che la risposta segue le linee guida fornite nel messaggio di sistema aggiunto alla matrice di *messaggi*.
1. Specificare la richiesta `Where should I hike near Boise? I'm looking for something of easy difficulty, between 2 to 3 miles, with moderate elevation gain.` e osservare l'output.
1. Nel file di codice per la lingua preferita, modificare il valore del parametro *temperatura* nella richiesta in **1.0** e salvare il file.
1. Eseguire di nuovo l'applicazione usando le richieste precedenti e osservare l'output.

L'aumento della temperatura spesso causa la variazione della risposta, anche quando viene fornito lo stesso testo, perché aumenta la casualità. È possibile eseguirlo più volte per vedere come l'output può cambiare. Provare a usare valori diversi per la temperatura con lo stesso input.

## Mantenere la cronologia delle conversazioni

Nella maggior parte delle applicazioni reali, la possibilità di fare riferimento alle parti precedenti della conversazione consente un'interazione più realistica con un agente di intelligenza artificiale. L'API OpenAI di Azure è senza stato per impostazione predefinita, ma fornendo una cronologia della conversazione nella richiesta, si consente al modello di intelligenza artificiale di fare riferimento ai messaggi precedenti.

1. Eseguire di nuovo l'app e specificare la richiesta `Where is a good hike near Boise?`.
1. Osservare l'output e quindi richiedere `How difficult is the second hike you suggested?`.
1. La risposta del modello indicherà probabilmente che non è possibile comprendere l'escursione a cui si fa riferimento. Per risolvere questo problema, è possibile consentire al modello di fare riferimento ai messaggi di conversazione precedenti.
1. Nell'applicazione è necessario aggiungere la richiesta precedente e la risposta alla richiesta futura che verrà inviato. Sotto la definizione del **messaggio di sistema**, aggiungere il codice seguente.

    **C#**: Program.cs

    ```csharp
    // Initialize messages list
    var messagesList = new List<ChatRequestMessage>()
    {
        new ChatRequestSystemMessage(systemMessage),
    };
    ```

    **Python**: test-openai-model.py

    ```python
    # Initialize messages array
    messages_array = [{"role": "system", "content": system_message}]
    ```

1. Sotto il commento ***Aggiungi codice per inviare la richiesta***, sostituire tutto il codice dal commento alla fine del **ciclo while** con il codice seguente e quindi salvare il file. Il codice è principalmente lo stesso, ma ora usa la matrice di messaggi per archiviare la cronologia delle conversazioni.

    **C#**: Program.cs

    ```csharp
    // Add code to send request...
    // Build completion options object
    messagesList.Add(new ChatRequestUserMessage(inputText));

    ChatCompletionsOptions chatCompletionsOptions = new ChatCompletionsOptions()
    {
        MaxTokens = 1200,
        Temperature = 0.7f,
        DeploymentName = oaiDeploymentName
    };

    // Add messages to the completion options
    foreach (ChatRequestMessage chatMessage in messagesList)
    {
        chatCompletionsOptions.Messages.Add(chatMessage);
    }

    // Send request to Azure OpenAI model
    ChatCompletions response = client.GetChatCompletions(chatCompletionsOptions);

    // Return the response
    string completion = response.Choices[0].Message.Content;

    // Add generated text to messages list
    messagesList.Add(new ChatRequestAssistantMessage(completion));

    Console.WriteLine("Response: " + completion + "\n");
    ```

    **Python**: test-openai-model.py

    ```python
    # Add code to send request...
    # Send request to Azure OpenAI model
    messages_array.append({"role": "user", "content": input_text})

    response = client.chat.completions.create(
        model=azure_oai_deployment,
        temperature=0.7,
        max_tokens=1200,
        messages=messages_array
    )
    generated_text = response.choices[0].message.content
    # Add generated text to messages array
    messages_array.append({"role": "assistant", "content": generated_text})

    # Print generated text
    print("Summary: " + generated_text + "\n")
    ```

1. Salvare il file. Nel codice aggiunto, si noti che ora si aggiunge l'input e la risposta precedenti alla matrice di richieste che consente al modello di comprendere la cronologia della conversazione.
1. Nel riquadro del terminale, immettere il comando seguente per eseguire l'applicazione.

    - **C#**: `dotnet run`
    - **Python**: `python test-openai-model.py`

1. Eseguire di nuovo l'app e specificare la richiesta `Where is a good hike near Boise?`.
1. Osservare l'output e quindi richiedere `How difficult is the second hike you suggested?`.
1. È probabile che si ottenga una risposta sulla seconda escursione suggerita dal modello, fornendo una conversazione molto più realistica. È possibile porre ulteriori domande che fanno riferimento alle risposte precedenti e ogni volta la cronologia fornisce il contesto per consentire al modello di rispondere.

    > **Suggerimento**: Il numero di token è impostato solo su 1200, quindi se la conversazione continua troppo a lungo, l'applicazione esaurisce i token disponibili, generando una richiesta incompleta. Negli usi di produzione, limitare la lunghezza della cronologia agli input e alle risposte più recenti, consentirà di controllare il numero di token necessari.

## Eseguire la pulizia

Terminato l’utilizzo della risorsa OpenAI di Azure, ricordarsi di eliminare la distribuzione o l'intera risorsa nel **portale di Azure** all'indirizzo `https://portal.azure.com`.
