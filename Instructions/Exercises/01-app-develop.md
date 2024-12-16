---
lab:
  title: "Sviluppo di applicazioni con Servizio OpenAI\_di Azure"
---

# Sviluppo di applicazioni con Servizio OpenAI di Azure

Con il Servizio OpenAI di Azure, gli sviluppatori possono sviluppare chatbot e altre applicazioni altamente performanti nella comprensione del linguaggio naturale, usando API REST o SDK specifici per il linguaggio di programmazione scelto. Quando si usano modelli linguistici, il modo in cui gli sviluppatori strutturano le richieste influisce significativamente sulla qualità e sulla pertinenza delle risposte generate dall'intelligenza artificiale. I modelli OpenAI di Azure sono in grado di personalizzare e formattare il contenuto, se richiesto in modo chiaro e conciso. In questo esercizio verrà illustrato come collegare un'app al servizio Azure OpenAI e come diverse richieste, basate su contenuti simili, possano essere ottimizzate per modellare la risposta del modello di intelligenza artificiale, al fine di soddisfare al meglio le esigenze dell'utente.

In questo esercizio, si assumerà il ruolo di uno sviluppatore software impegnato nella progettazione di una campagna di marketing dedicata alla fauna selvatica. Si sta esplorando come usare l'intelligenza artificiale generativa per migliorare i messaggi di posta elettronica pubblicitari e classificare gli articoli che potrebbero essere applicati al team. Le tecniche di progettazione richieste usate nell'esercizio possono essere applicate in modo analogo per un'ampia gamma di casi d'uso.

Questo esercizio richiederà circa **30** minuti.

## Effettuare il provisioning di una risorsa OpenAI di Azure

Se non è già disponibile, effettuare il provisioning di una risorsa OpenAI di Azure nella sottoscrizione di Azure.

1. Accedere al **portale di Azure** all'indirizzo `https://portal.azure.com`.

1. Creare una risorsa **OpenAI di Azure** con le impostazioni seguenti:
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

3. Attendere il completamento della distribuzione. Passare quindi alla risorsa OpenAI di Azure distribuita nel portale di Azure.

## Distribuire un modello

Successivamente, verrà distribuita una risorsa modello di Azure OpenAI dall'interfaccia della riga di comando. Fare riferimento a questo esempio e sostituire le variabili seguenti con i valori indicati in precedenza:

```dotnetcli
az cognitiveservices account deployment create \
   -g *Your resource group* \
   -n *Name of your OpenAI service* \
   --deployment-name gpt-35-turbo \
   --model-name gpt-35-turbo \
   --model-version 0125  \
   --model-format OpenAI \
   --sku-name "Standard" \
   --sku-capacity 5
```

    > \* Sku-capacity is measured in thousands of tokens per minute. A rate limit of 5,000 tokens per minute is more than adequate to complete this exercise while leaving capacity for other people using the same subscription.

> [!NOTE]
> Se durante l'esercizio vengono visualizzati avvisi relativi al mancato supporto del framework net7.0, è possibile ignorarli senza compromettere il completamento dell'attività.

## Configurare l'applicazione

Sono state fornite applicazioni sia per C# sia per Python e entrambe le app hanno la stessa funzionalità. Prima di tutto, verranno completate alcune parti chiave dell'applicazione per consentire l'uso della risorsa OpenAI di Azure con chiamate API asincrone.

1. Nel riquadro **Explorer** di Visual Studio Code, passare alla cartella **Labfiles/01-app-develop** ed espandere la cartella **CSharp** o **Python** a seconda del linguaggio scelto. Ogni cartella contiene i file specifici del linguaggio per un’app in cui si intende integrare la funzionalità OpenAI di Azure.
2. Fare clic con il pulsante destro del mouse sulla cartella **CSharp** o **Python** contenente i file di codice, quindi aprire un terminale integrato. Installare quindi il pacchetto SDK di OpenAI eseguendo il comando appropriato per il linguaggio scelto:

    **C#:**

    ```
    dotnet add package Azure.AI.OpenAI --version 2.0.0
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
        ChatCompletion completion = chatClient.CompleteChat(
        [
        new SystemChatMessage(systemMessage),
        new UserChatMessage(userMessage),
        ]);
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
    \n Include a list of the current animals we have at our rescue after the signature, in the form of a table. These animals include elephants, zebras, gorillas, lizards, and jackrabbits.
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
    \n Include a list of the current animals we have at our rescue after the signature, in the form of a table. These animals include elephants, zebras, gorillas, lizards, and jackrabbits.
    ```

1. Osservare l'output. Questa volta probabilmente si vedrà il messaggio di posta elettronica in un formato simile, ma con un tono molto più informale. Probabilmente si vedranno anche scherzi inclusi!
1. Per l'iterazione finale, si sta deviando dalla generazione della posta elettronica ed esaminando *contesto di base*. Qui viene fornito un semplice messaggio di sistema e si modifica l'app per fornire il contesto di base come inizio della richiesta dell'utente. L'app aggiungerà quindi l'input dell'utente ed estrae le informazioni dal contesto di base per rispondere alla richiesta dell'utente.
1. Aprire il file `grounding.txt` e leggere brevemente il contesto di base che verrà inserito.
1. Nell'app subito dopo il commento ***Formatta e invia la richiesta al modello*** e prima di qualsiasi codice esistente, aggiungere il frammento di codice seguente per leggere il testo da `grounding.txt` per integrare il prompt dell'utente con il contesto di riferimento.

    **C#**: Program.cs

    ```csharp
    // Format and send the request to the model
    Console.WriteLine("\nAdding grounding context from grounding.txt");
    string groundingText = System.IO.File.ReadAllText("grounding.txt");
    userMessage = groundingText + userMessage;
    ```

    **Python**: application.py

    ```python
    # Format and send the request to the model
    print("\nAdding grounding context from grounding.txt")
    grounding_text = open(file="grounding.txt", encoding="utf8").read().strip()
    user_message = grounding_text + user_message
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

> **Suggerimento**: Se si vuole visualizzare la risposta completa da Azure OpenAI, è possibile impostare la variabile **printFullResponse** su `True`ed eseguire nuovamente l'app.

## Eseguire la pulizia

Terminato l'utilizzo della risorsa OpenAI di Azure, ricordarsi di eliminare la distribuzione o l'intera risorsa nel **portale di Azure** all'indirizzo `https://portal.azure.com`.
