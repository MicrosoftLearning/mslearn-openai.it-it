---
lab:
  title: "Sviluppo di applicazioni con Servizio OpenAI\_di Azure"
  status: new
---

# Sviluppo di applicazioni con Servizio OpenAI di Azure

Con il Servizio OpenAI di Azure, gli sviluppatori possono sviluppare chatbot e altre applicazioni altamente performanti nella comprensione del linguaggio naturale, usando API REST o SDK specifici per il linguaggio di programmazione scelto. Quando si usano modelli linguistici, il modo in cui gli sviluppatori strutturano le richieste influisce significativamente sulla qualità e sulla pertinenza delle risposte generate dall'intelligenza artificiale. I modelli OpenAI di Azure sono in grado di personalizzare e formattare il contenuto, se richiesto in modo chiaro e conciso. In questo esercizio verrà illustrato come collegare un'app al servizio Azure OpenAI e come diverse richieste, basate su contenuti simili, possano essere ottimizzate per modellare la risposta del modello di intelligenza artificiale, al fine di soddisfare al meglio le esigenze dell'utente.

In questo esercizio, si assumerà il ruolo di uno sviluppatore software impegnato nella progettazione di una campagna di marketing dedicata alla fauna selvatica. Si sta esplorando come usare l'intelligenza artificiale generativa per migliorare i messaggi di posta elettronica pubblicitari e classificare gli articoli che potrebbero essere applicati al team. Le tecniche di progettazione richieste usate nell'esercizio possono essere applicate in modo analogo per un'ampia gamma di casi d'uso.

Questo esercizio richiederà circa **30** minuti.

## Clonare il repository per questo corso

Se non è già stato fatto, è necessario clonare il repository di codice per questo corso:

1. Avviare Visual Studio Code.
2. Aprire il riquadro comandi (MAIUSC+CTRL+P o **Visualizza** > **Riquadro comandi...**) ed eseguire un comando **Git: Clone** per clonare il repository `https://github.com/MicrosoftLearning/mslearn-openai` in una cartella locale (non è importante usare una cartella specifica).
3. Dopo la clonazione del repository, aprire la cartella in Visual Studio Code.
4. Attendere il completamento dell'installazione di file aggiuntivi per supportare i progetti in codice C# nel repository.

    > **Nota**: se viene richiesto di aggiungere gli asset necessari per la compilazione e il debug, selezionare **Non adesso**.

## Effettuare il provisioning di una risorsa OpenAI di Azure

Se non è già disponibile, effettuare il provisioning di una risorsa OpenAI di Azure nella sottoscrizione di Azure.

1. Accedere al **portale di Azure** all'indirizzo `https://portal.azure.com`.

1. Creare una risorsa **OpenAI di Azure** con le impostazioni seguenti:
    - **Sottoscrizione**: *Selezionare una sottoscrizione di Azure approvata per l'accesso al servizio OpenAI di Azure*
    - **Gruppo di risorse**: *Scegliere o creare un gruppo di risorse*
    - **Area**: *Effettuare una scelta **casuale** da una delle aree seguenti*\*
        - Stati Uniti orientali
        - Stati Uniti orientali 2
        - Stati Uniti centro-settentrionali
        - Stati Uniti centro-meridionali
        - Svezia centrale
        - Stati Uniti occidentali
        - Stati Uniti occidentali 3
    - **Nome**: *nome univoco di propria scelta*
    - **Piano tariffario**: Standard S0.

    > \* Le risorse OpenAI di Azure sono vincolate dalle quote a livello di area. Le aree elencate includono la quota predefinita per i tipi di modello usati in questo esercizio. La scelta casuale di un'area riduce il rischio che una singola area raggiunga il limite di quota negli scenari in cui si condivide una sottoscrizione con altri utenti. In caso di raggiungimento di un limite di quota più avanti nell'esercizio, potrebbe essere necessario creare un'altra risorsa in un'area diversa.

1. Attendere il completamento della distribuzione. Passare quindi alla risorsa OpenAI di Azure distribuita nel portale di Azure.

## Distribuire un modello

Successivamente, verrà distribuita una risorsa modello di Azure OpenAI da Cloud Shell.

1. Usare il pulsante **[\>_]** a destra della barra di ricerca, nella parte superiore della pagina, per aprire una nuova sessione di Cloud Shell nel portale di Azure selezionando un ambiente ***Bash***. Cloud Shell fornisce un'interfaccia della riga di comando in un riquadro nella parte inferiore del portale di Azure.

    > **Nota**: se in precedenza è stata creata una sessione Cloud Shell che usa un ambiente *PowerShell*, passare a ***Bash***.

1. Fare riferimento a questo esempio e sostituire le variabili seguenti con i valori indicati in precedenza:

    ```dotnetcli
    az cognitiveservices account deployment create \
       -g <your_resource_group> \
       -n <your_OpenAI_service> \
       --deployment-name gpt-4o \
       --model-name gpt-4o \
       --model-version 2024-05-13 \
       --model-format OpenAI \
       --sku-name "Standard" \
       --sku-capacity 5
    ```

> **Nota**: la capacità SKU viene misurata in migliaia di token al minuto. Un limite di 5.000 token al minuto è più che sufficiente per completare questo esercizio, lasciando capacità ad altre persone che usano la stessa sottoscrizione.

## Configurare l'applicazione

Sono state fornite applicazioni sia per C# sia per Python e entrambe le app hanno la stessa funzionalità. Prima di tutto, verranno completate alcune parti chiave dell'applicazione per consentire l'uso della risorsa OpenAI di Azure con chiamate API asincrone.

1. Nel riquadro **Explorer** di Visual Studio Code, passare alla cartella **Labfiles/01-app-develop** ed espandere la cartella **CSharp** o **Python** a seconda del linguaggio scelto. Ogni cartella contiene i file specifici del linguaggio per un’app in cui si intende integrare la funzionalità OpenAI di Azure.
2. Fare clic con il pulsante destro del mouse sulla cartella **CSharp** o **Python** contenente i file di codice, quindi aprire un terminale integrato. Installare quindi il pacchetto SDK di OpenAI eseguendo il comando appropriato per il linguaggio scelto:

    **C#:**

    ```powershell
    dotnet add package Azure.AI.OpenAI --version 2.1.0
    ```

    **Python**:

    ```powershell
    pip install openai==1.65.2
    ```

3. Nel riquadro **Esplora risorse**, nella cartella **CSharp** o **Python**, aprire il file di configurazione per il linguaggio preferita

    - **C#**: appsettings.json
    - **Python**: .env

4. Aggiornare i valori di configurazione in modo da includere:
    - L'**endpoint** e una **chiave** della risorsa OpenAI di Azure creata (disponibile nella pagina **Chiavi ed endpoint** per la risorsa OpenAI di Azure nel portale di Azure)
    - Il **nome della distribuzione** specificato per la distribuzione del modello.
5. Salvare il file di configurazione.

## Aggiungere codice per usare il servizio OpenAI di Azure

A questo punto, è possibile usare l'SDK di OpenAI di Azure per usare il modello distribuito.

1. Nel riquadro **Esplora risorse**, nella cartella **CSharp** o **Python**, aprire il file di codice per il linguaggio preferito e sostituire il commento ***Aggiungi pacchetto OpenAI di Azure*** con il codice per aggiungere la libreria SDK di OpenAI di Azure:

    **C#**: Program.cs

    ```csharp
    // Add Azure OpenAI packages
    using Azure.AI.OpenAI;
    using OpenAI.Chat;
    ```

    **Python**: application.py

    ```python
    # Add Azure OpenAI package
    from openai import AsyncAzureOpenAI
    ```

2. Nel file di codice trovare il commento ***Configurare il client OpenAI di Azure***e aggiungere codice per configurare il client OpenAI di Azure:

    **C#**: Program.cs

    ```csharp
    // Configure the Azure OpenAI client
    AzureOpenAIClient azureClient = new (new Uri(oaiEndpoint), new ApiKeyCredential(oaiKey));
    ChatClient chatClient = azureClient.GetChatClient(oaiDeploymentName);
    ```

    **Python**: application.py

    ```python
    # Configure the Azure OpenAI client
    client = AsyncAzureOpenAI(
        azure_endpoint = azure_oai_endpoint, 
        api_key=azure_oai_key,  
        api_version="2024-02-15-preview"
    )
    ```

3. Nella funzione che chiama il modello Azure OpenAI, sotto il commento ***Ottieni una risposta da OpenAI di Azure***, aggiungere il codice per formattare e inviare la richiesta al modello.

    **C#**: Program.cs

    ```csharp
    // Get response from Azure OpenAI
    ChatCompletionOptions chatCompletionOptions = new ChatCompletionOptions()
    {
        Temperature = 0.7f,
        MaxOutputTokenCount = 800
    };

    ChatCompletion completion = chatClient.CompleteChat(
        [
            new SystemChatMessage(systemMessage),
            new UserChatMessage(userMessage)
        ],
        chatCompletionOptions
    );

    Console.WriteLine($"{completion.Role}: {completion.Content[0].Text}");
    ```

    **Python**: application.py

    ```python
    # Get response from Azure OpenAI
    messages =[
        {"role": "system", "content": system_message},
        {"role": "user", "content": user_message},
    ]
    
    print("\nSending request to Azure OpenAI model...\n")

    # Call the Azure OpenAI model
    response = await client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.7,
        max_tokens=800
    )
    ```

4. Salvare le modifiche apportate al file di codice.

## Eseguire l'applicazione

Ora che l'app è stata configurata, eseguirla per inviare la richiesta al modello e osservare la risposta. Si noterà che l'unica differenza tra le diverse opzioni è il contenuto del prompt, tutti gli altri parametri (ad esempio il numero di token e la temperatura) rimangono invariati per ogni richiesta.

1. Nella cartella della lingua dell'interfaccia utente, aprire `system.txt` in Visual Studio Code. Per ognuna delle interazioni, immettere il **messaggio di sistema** in questo file e salvarlo. Ogni iterazione verrà sospesa per prima cosa per modificare il messaggio di sistema.
1. Nel riquadro del terminale interattivo, accertarsi che il contesto della cartella sia la cartella per la lingua dell’interfaccia utente. Immettere, quindi, il comando seguente per eseguire l'applicazione.

    - **C#**: `dotnet run`
    - **Python**: `python application.py`

    > **Suggerimento**: È possibile usare l'icona **Ingrandisci dimensioni pannello** (**^**) nella barra degli strumenti del terminale per visualizzare altro testo della console.

1. Per la prima iterazione, immettere le istruzioni seguenti:

    **Messaggio di sistema**

    ```prompt
    You are an AI assistant
    ```

    **Messaggio utente:**

    ```prompt
    Write an intro for a new wildlife Rescue
    ```

1. Osservare l'output. Il modello di intelligenza artificiale produrrà probabilmente una buona introduzione generica a un salvataggio della fauna selvatica.
1. Immettere quindi i prompt seguenti che specificano un formato per la risposta:

    **Messaggio di sistema**

    ```prompt
    You are an AI assistant helping to write emails
    ```

    **Messaggio utente:**

    ```prompt
    Write a promotional email for a new wildlife rescue, including the following: 
    - Rescue name is Contoso 
    - It specializes in elephants 
    - Call for donations to be given at our website
    ```

    > **Suggerimento**: è possibile che la digitazione automatica nella macchina virtuale non funzioni correttamente con i prompt su più righe. In questo caso, copiare l'intero prompt e incollarlo in Visual Studio Code.

1. Osservare l'output. Questa volta, probabilmente si vedrà il formato di un messaggio di posta elettronica con gli animali specifici inclusi, nonché la chiamata per le donazioni.
1. Immettere quindi le istruzioni seguenti che specificano anche il contenuto:

    **Messaggio di sistema**

    ```prompt
    You are an AI assistant helping to write emails
    ```

    **Messaggio utente:**

    ```prompt
    Write a promotional email for a new wildlife rescue, including the following: 
    - Rescue name is Contoso 
    - It specializes in elephants, as well as zebras and giraffes 
    - Call for donations to be given at our website 
    Include a list of the current animals we have at our rescue after the signature, in the form of a table. These animals include elephants, zebras, gorillas, lizards, and jackrabbits.
    ```

1. Osservare l'output e vedere come il messaggio di posta elettronica è cambiato in base alle istruzioni chiare.
1. Immettere quindi le istruzioni seguenti in cui si aggiungono dettagli sul tono al messaggio di sistema:

    **Messaggio di sistema**

    ```prompt
    You are an AI assistant that helps write promotional emails to generate interest in a new business. Your tone is light, chit-chat oriented and you always include at least two jokes.
    ```

    **Messaggio utente:**

    ```prompt
    Write a promotional email for a new wildlife rescue, including the following: 
    - Rescue name is Contoso 
    - It specializes in elephants, as well as zebras and giraffes 
    - Call for donations to be given at our website 
    Include a list of the current animals we have at our rescue after the signature, in the form of a table. These animals include elephants, zebras, gorillas, lizards, and jackrabbits.
    ```

1. Osservare l'output. Questa volta probabilmente si vedrà il messaggio di posta elettronica in un formato simile, ma con un tono molto più informale. Probabilmente si vedranno anche scherzi inclusi!

## Usare il contesto di base e mantenere la cronologia delle chat

1. Per l'iterazione finale, invece di generare e-mail, verrà esplorato il *contesto di base* e il mantenimento della cronologia delle chat. Qui verrà fornito un semplice messaggio di sistema e l'app verrà modificata per includere il contesto di base all'inizio della cronologia della chat. L'app aggiungerà quindi l'input dell'utente ed estrae le informazioni dal contesto di base per rispondere alla richiesta dell'utente.
1. Aprire il file `grounding.txt` e leggere brevemente il contesto di base che verrà inserito.
1. Nell'app, subito dopo il commento ***Inizializza l'elenco dei messaggi*** e prima di qualsiasi codice esistente, aggiungere il seguente frammento di codice per leggere il testo da `grounding.txt` e inizializzare la cronologia della chat con il contesto di base.

    **C#**: Program.cs

    ```csharp
    // Initialize messages list
    Console.WriteLine("\nAdding grounding context from grounding.txt");
    string groundingText = System.IO.File.ReadAllText("grounding.txt");
    var messagesList = new List<ChatMessage>()
    {
        new UserChatMessage(groundingText),
    };
    ```

    **Python**: application.py

    ```python
    # Initialize messages array
    print("\nAdding grounding context from grounding.txt")
    grounding_text = open(file="grounding.txt", encoding="utf8").read().strip()
    messages_array = [{"role": "user", "content": grounding_text}]
    ```

1. Sotto il commento ***Formatta e invia la richiesta al modello***, sostituire il codice dal commento alla fine del ciclo **while** con il codice seguente. Il codice è sostanzialmente lo stesso, ma ora viene usato l'array dei messaggi per inviare la richiesta al modello.

    **C#**: Program.cs
   
    ```csharp
    // Format and send the request to the model
    messagesList.Add(new SystemChatMessage(systemMessage));
    messagesList.Add(new UserChatMessage(userMessage));
    GetResponseFromOpenAI(messagesList);
    ```

    **Python**: application.py

    ```python
    # Format and send the request to the model
    messages_array.append({"role": "system", "content": system_text})
    messages_array.append({"role": "user", "content": user_text})
    await call_openai_model(messages=messages_array, 
        model=azure_oai_deployment, 
        client=client
    )
    ```

1. Sotto il commento ***Definisci la funzione che otterrà la risposta dall'endpoint Azure OpenAI***, sostituire la dichiarazione di funzione con il codice seguente per usare l'elenco della cronologia delle chat quando viene chiamata la funzione `GetResponseFromOpenAI` per C# o `call_openai_model` per Python.

    **C#**: Program.cs
   
    ```csharp
    // Define the function that gets the response from Azure OpenAI endpoint
    private static void GetResponseFromOpenAI(List<ChatMessage> messagesList)
    ```

    **Python**: application.py

    ```python
    # Define the function that will get the response from Azure OpenAI endpoint
    async def call_openai_model(messages, model, client):
    ```
    
1. Infine, sostituire l'intero codice sotto ***Ottieni una risposta da OpenAI di Azure***. Il codice è principalmente lo stesso, ma ora usa la matrice di messaggi per archiviare la cronologia delle conversazioni.

    **C#**: Program.cs
   
    ```csharp
    // Get response from Azure OpenAI
    ChatCompletionOptions chatCompletionOptions = new ChatCompletionOptions()
    {
        Temperature = 0.7f,
        MaxOutputTokenCount = 800
    };

    ChatCompletion completion = chatClient.CompleteChat(
        messagesList,
        chatCompletionOptions
    );

    Console.WriteLine($"{completion.Role}: {completion.Content[0].Text}");
    messagesList.Add(new AssistantChatMessage(completion.Content[0].Text));
    ```

    **Python**: application.py

    ```python
    # Get response from Azure OpenAI
    print("\nSending request to Azure OpenAI model...\n")

    # Call the Azure OpenAI model
    response = await client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.7,
        max_tokens=800
    )   

    print("Response:\n" + response.choices[0].message.content + "\n")
    messages.append({"role": "assistant", "content": response.choices[0].message.content})
    ```
    
1. Salvare il file ed eseguire di nuovo l'app.
1. Immettere le istruzioni seguenti (con il **messaggio di sistema** ancora immesso e salvato in `system.txt`).

    **Messaggio di sistema**

    ```prompt
    You're an AI assistant who helps people find information. You'll provide answers from the text provided in the prompt, and respond concisely.
    ```

    **Messaggio utente:**

    ```prompt
    What animal is the favorite of children at Contoso?
    ```

   Notare che il modello sfrutta le informazioni del testo di base per rispondere alla domanda.

1. Senza modificare il messaggio di sistema, inserire il seguente prompt per il messaggio utente:

    **Messaggio utente:**

    ```prompt
    How can they interact with it at Contoso?
    ```

    Notare che il modello riconosce "they" (loro) come i bambini e "it" (egli) come il loro animale preferito, poiché ora ha accesso alla domanda precedente nella cronologia della chat.
   
## Eseguire la pulizia

Terminato l'utilizzo della risorsa OpenAI di Azure, ricordarsi di eliminare la distribuzione o l'intera risorsa nel **portale di Azure** all'indirizzo `https://portal.azure.com`.
