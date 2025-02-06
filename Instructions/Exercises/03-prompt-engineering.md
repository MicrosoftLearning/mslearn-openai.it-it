---
lab:
  title: Usare la progettazione delle richieste nell'app
---

# Usare la progettazione delle richieste nell'app

Quando si lavora con il servizio Azure OpenAI, il modo in cui gli sviluppatori modellano la richiesta influiscono notevolmente sul modo in cui risponderà il modello di intelligenza artificiale generativa. I modelli OpenAI di Azure sono in grado di personalizzare e formattare il contenuto, se richiesto in modo chiaro e conciso. In questo esercizio si apprenderà in che modo diverse richieste di contenuto simile consentono di modellare la risposta del modello di intelligenza artificiale per soddisfare meglio i requisiti.

Nello scenario per questo esercizio si agirà nel ruolo di sviluppatore di software che lavora su una campagna di marketing sulla fauna selvatica. Si sta esplorando come usare l'intelligenza artificiale generativa per migliorare i messaggi di posta elettronica pubblicitari e classificare gli articoli che potrebbero essere applicati al team. Le tecniche di progettazione richieste usate nell'esercizio possono essere applicate in modo analogo per un'ampia gamma di casi d'uso.

Questo esercizio richiederà circa **30** minuti.

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

## Esplorare le tecniche di progettazione rapida

Per iniziare, si esamineranno alcune tecniche di progettazione richieste nel playground Chat.

1. Nel riquadro laterale sinistro, nella sezione **Playground** selezionare la pagina **Chat**. La pagina Playground **Chat** è costituita da una riga di pulsanti e da due pannelli principali (che possono essere disposti da destra a sinistra orizzontalmente o dall'alto verso il basso verticalmente a seconda della risoluzione dello schermo):
    - **Configurazione**: utilizzata per selezionare la distribuzione, definire il messaggio di sistema e impostare i parametri per interagire con la distribuzione.
    - **Cronologia chat**: consente di inviare messaggi di chat e visualizzare le risposte.
2. In **Distribuzione** assicurarsi che sia selezionata la distribuzione del modello gpt-35-turbo-16k.
1. Esaminare il messaggio di sistema predefinito contenuto nella casella di testo immediatamente sotto la distribuzione selezionata, che deve essere *Questo è un assistente di intelligenza artificiale che aiuta gli utenti a trovare informazioni.*
4. Nella **Cronologia chat** inviare la query seguente:

    ```prompt
    What kind of article is this?
    ---
    Severe drought likely in California
    
    Millions of California residents are bracing for less water and dry lawns as drought threatens to leave a large swath of the region with a growing water shortage.
    
    In a remarkable indication of drought severity, officials in Southern California have declared a first-of-its-kind action limiting outdoor water use to one day a week for nearly 8 million residents.
    
    Much remains to be determined about how daily life will change as people adjust to a drier normal. But officials are warning the situation is dire and could lead to even more severe limits later in the year.
    ```

    La risposta fornisce una descrizione dell'articolo. Si supponga tuttavia di voler usare un formato più specifico per la categorizzazione degli articoli.

5. Nella sezione **Installazione**, modificare il messaggio di sistema in `You are a news aggregator that categorizes news articles.`

6. Sotto il nuovo messaggio di sistema, selezionare il pulsante **Aggiungi sezione** e scegliere **Esempi**. Aggiungere quindi l’esempio seguente.

    **Utente**:
    
    ```prompt
    What kind of article is this?
    ---
    New York Baseballers Wins Big Against Chicago
    
    New York Baseballers mounted a big 5-0 shutout against the Chicago Cyclones last night, solidifying their win with a 3 run homerun late in the bottom of the 7th inning.
    
    Pitcher Mario Rogers threw 96 pitches with only two hits for New York, marking his best performance this year.
    
    The Chicago Cyclones' two hits came in the 2nd and the 5th innings but were unable to get the runner home to score.
    ```
    
    **Assistente**:
    
    ```prompt
    Sports
      ```

7. Aggiungere un altro esempio con il testo seguente.

    **Utente**:
    
    ```prompt
    Categorize this article:
    ---
    Joyous moments at the Oscars
    
    The Oscars this past week where quite something!
    
    Though a certain scandal might have stolen the show, this year's Academy Awards were full of moments that filled us with joy and even moved us to tears.
    These actors and actresses delivered some truly emotional performances, along with some great laughs, to get us through the winter.
    
    From Robin Kline's history-making win to a full performance by none other than Casey Jensen herself, don't miss tomorrows rerun of all the festivities.
    ```
    
    **Assistente**:
    
    ```prompt
    Entertainment
    ```

8. Usare il pulsante **Applica modifiche** sotto la casella di testo del messaggio di sistema nella sezione **Configurazione** per salvare le modifiche.

9. Nella sezione **Cronologia chat** inviare nuovamente il prompt seguente:

    ```prompt
    What kind of article is this?
    ---
    Severe drought likely in California
    
    Millions of California residents are bracing for less water and dry lawns as drought threatens to leave a large swath of the region with a growing water shortage.
    
    In a remarkable indication of drought severity, officials in Southern California have declared a first-of-its-kind action limiting outdoor water use to one day a week for nearly 8 million residents.
    
    Much remains to be determined about how daily life will change as people adjust to a drier normal. But officials are warning the situation is dire and could lead to even more severe limits later in the year.
    ```

    La combinazione di un messaggio di sistema più specifico e alcuni esempi di query e risposte previste generano un formato coerente per i risultati.

10. Modificare il messaggio di sistema riportandolo al modello predefinito, che deve essere `You are an AI assistant that helps people find information.` senza esempi. Applicare quindi le modifiche.

11. Nella sezione **Cronologia chat** inviare il prompt seguente:

    ```prompt
    # 1. Create a list of animals
    # 2. Create a list of whimsical names for those animals
    # 3. Combine them randomly into a list of 25 animal and name pairs
    ```

    Il modello risponderà probabilmente con una risposta per soddisfare il prompt, suddiviso in un elenco numerato. Questa è una risposta appropriata, ma supponiamo che quello che si vuole in realtà sia che il modello scriva un programma Python che esegua i compiti descritti?

12. Modificare il messaggio di sistema in `You are a coding assistant helping write python code.` e applicare le modifiche.
13. Inviare di nuovo il prompt seguente al modello:

    ```
    # 1. Create a list of animals
    # 2. Create a list of whimsical names for those animals
    # 3. Combine them randomly into a list of 25 animal and name pairs
    ```

    Il modello deve rispondere correttamente con il codice Python eseguendo le operazioni richieste dai commenti.

## Prepararsi allo sviluppo di un'app in Visual Studio Code

Si esaminerà ora l'uso della progettazione dei prompt in un'app che usa Azure OpenAI Service SDK. L'app verrà sviluppata usando Visual Studio Code. I file di codice per l'app sono stati forniti in un repository GitHub.

> **Suggerimento**: Se il repository **mslearn-openai** è già stato clonato, aprirlo in Visual Studio Code. In caso contrario, eseguire i passaggi seguenti per clonarlo nell'ambiente di sviluppo.

1. Avviare Visual Studio Code.
2. Aprire il riquadro comandi (MAIUSC+CTRL+P) ed eseguire un comando **Git: Clone** per clonare il repository `https://github.com/MicrosoftLearning/mslearn-openai` in una cartella locale. Non è importante usare una cartella specifica.
3. Dopo la clonazione del repository, aprire la cartella in Visual Studio Code.

    > **Nota**: Se Visual Studio Code visualizza un messaggio popup per chiedere se si considera attendibile il codice che si apre, fare clic sull'opzione **Sì, considero attendibili gli autori** nel popup.

4. Attendere il completamento dell'installazione di file aggiuntivi per supportare i progetti in codice C# nel repository.

    > **Nota**: se viene richiesto di aggiungere gli asset necessari per la compilazione e il debug, selezionare **Non adesso**.

## Configurare l'applicazione

Sono state fornite applicazioni sia per C# sia per Python e entrambe le app hanno la stessa funzionalità. Prima di tutto, verranno completate alcune parti chiave dell'applicazione per consentire l'uso della risorsa OpenAI di Azure con chiamate API asincrone.

1. Nel riquadro **Explorer** di Visual Studio Code, passare alla cartella **Labfiles/03-prompt-engineering** ed espandere la cartella **CSharp** o **Python** a seconda del linguaggio scelto. Ogni cartella contiene i file specifici del linguaggio per un’app in cui si intende integrare la funzionalità OpenAI di Azure.
2. Fare clic con il pulsante destro del mouse sulla cartella **CSharp** o **Python** contenente i file di codice, quindi aprire un terminale integrato. Installare quindi il pacchetto SDK di OpenAI eseguendo il comando appropriato per il linguaggio scelto:

    **C#:**

    ```
    dotnet add package Azure.AI.OpenAI --version 1.0.0-beta.14
    ```

    **Python**:

    ```
    pip install openai==1.55.3
    ```

3. Nel riquadro **Esplora risorse**, nella cartella **CSharp** o **Python**, aprire il file di configurazione per il linguaggio preferita

    - **C#**: appsettings.json
    - **Python**: .env
    
4. Aggiornare i valori di configurazione in modo da includere:
    - L'**endpoint** e una **chiave** della risorsa OpenAI di Azure creata (disponibile nella pagina **Chiavi ed endpoint** per la risorsa OpenAI di Azure nel portale di Azure)
    - Il **nome della distribuzione** specificato per la distribuzione del modello (disponibile nella pagina **Distribuzioni** nel portale di Azure AI Foundry).
5. Salvare il file di configurazione.

## Aggiungere codice per usare il servizio OpenAI di Azure

A questo punto, è possibile usare l'SDK di OpenAI di Azure per usare il modello distribuito.

1. Nel riquadro **Esplora risorse**, nella cartella **CSharp** o **Python**, aprire il file di codice per il linguaggio preferito e sostituire il commento ***Aggiungi pacchetto OpenAI di Azure*** con il codice per aggiungere la libreria SDK di OpenAI di Azure:

    **C#**: Program.cs

    ```csharp
    // Add Azure OpenAI package
    using Azure.AI.OpenAI;
    ```

    **Python**: prompt-engineering.py

    ```python
    # Add Azure OpenAI package
    from openai import AsyncAzureOpenAI
    ```

2. Nel file di codice trovare il commento ***Configurare il client OpenAI di Azure***e aggiungere codice per configurare il client OpenAI di Azure:

    **C#**: Program.cs

    ```csharp
    // Configure the Azure OpenAI client
    OpenAIClient client = new OpenAIClient(new Uri(oaiEndpoint), new AzureKeyCredential(oaiKey));
    ```

    **Python**: prompt-engineering.py

    ```python
    # Configure the Azure OpenAI client
    client = AsyncAzureOpenAI(
        azure_endpoint = azure_oai_endpoint, 
        api_key=azure_oai_key,  
        api_version="2024-02-15-preview"
        )
    ```

3. Nella funzione che chiama il modello Azure OpenAI, sotto il commento ***Formatta e invia la richiesta al modello***, aggiungere il codice per formattare e inviare la richiesta al modello.

    **C#**: Program.cs

    ```csharp
    // Format and send the request to the model
    var chatCompletionsOptions = new ChatCompletionsOptions()
    {
        Messages =
        {
            new ChatRequestSystemMessage(systemMessage),
            new ChatRequestUserMessage(userMessage)
        },
        Temperature = 0.7f,
        MaxTokens = 800,
        DeploymentName = oaiDeploymentName
    };
    
    // Get response from Azure OpenAI
    Response<ChatCompletions> response = await client.GetChatCompletionsAsync(chatCompletionsOptions);
    ```

    **Python**: prompt-engineering.py

    ```python
    # Format and send the request to the model
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
    - **Python**: `python prompt-engineering.py`

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

    **Python**: prompt-engineering.py

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
