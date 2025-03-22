---
lab:
  title: Generare e migliorare il codice con il Servizio OpenAI di Azure
  status: stale
---

# Generare e migliorare il codice con il Servizio OpenAI di Azure

I modelli del Servizio OpenAI di Azure possono generare codice per l'utente usando richieste in linguaggio naturale, correggendo i bug nel codice completato e fornendo commenti al codice. Questi modelli possono anche spiegare e semplificare il codice esistente per comprendere cosa fa e come migliorarlo.

Nello scenario per questo esercizio si svolgerà il ruolo di sviluppatore di software che esplora come usare l'intelligenza artificiale generativa per semplificare e rendere più efficienti le attività di codifica. Le tecniche usate nell'esercizio possono essere applicate ad altri file di codice, linguaggi di programmazione e casi d'uso.

Questo esercizio richiederà circa **25** minuti.

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

## Generare codice nel playground della chat

Prima di usarlo nella propria app, esaminare il modo in cui OpenAI di Azure può generare e spiegare il codice nel playground della chat.

1. Nella sezione **Playground** selezionare la pagina **Chat**. La pagina Playground **Chat** è costituita da una riga di pulsanti e da due pannelli principali (che possono essere disposti da destra a sinistra orizzontalmente o dall'alto verso il basso verticalmente a seconda della risoluzione dello schermo):
    - **Configurazione**: utilizzata per selezionare la distribuzione, definire il messaggio di sistema e impostare i parametri per interagire con la distribuzione.
    - **Sessione chat**: consente di inviare messaggi di chat e visualizzare le risposte.
1. In **Distribuzioni** assicurarsi che sia selezionata la distribuzione del modello.
1. Nell'area **Messaggio di sistema** impostare il messaggio di sistema su `You are a programming assistant helping write code` e applicare le modifiche.
1. Nella **sessione di chat** inviare la query seguente:

    ```
    Write a function in python that takes a character and a string as input, and returns how many times the character appears in the string
    ```

    Il modello risponderà probabilmente con una funzione, con una spiegazione su cosa fa la funzione e come chiamarla.

1. Inviare quindi la richiesta `Do the same thing, but this time write it in C#`.

    Il modello ha probabilmente risposto in modo molto simile alla prima volta, ma questa con il codice scritto in C#. È possibile riformulare la richiesta per un linguaggio diverso a propria scelta o per una funzione per completare un'attività diversa, ad esempio il ripristino della stringa di input.

1. Si esaminerà ora l'uso dell'intelligenza artificiale per comprendere il codice. Inviare la richiesta seguente come messaggio utente.

    ```
    What does the following function do?  
    ---  
    def multiply(a, b):  
        result = 0  
        negative = False  
        if a < 0 and b > 0:  
            a = -a  
            negative = True  
        elif a > 0 and b < 0:  
            b = -b  
            negative = True  
        elif a < 0 and b < 0:  
            a = -a  
            b = -b  
        while b > 0:  
            result += a  
            b -= 1      
        if negative:  
            return -result  
        else:  
            return result  
    ```

    Il modello deve descrivere cosa fa la funzione, ovvero moltiplicare due numeri insieme usando un ciclo.

7. Inviare la richiesta `Can you simplify the function?`.

    Il modello deve scrivere una versione più semplice della funzione.

8. Inviare la richiesta `Add some comments to the function.`

    Il modello aggiunge commenti al codice.

## Prepararsi allo sviluppo di un'app in Visual Studio Code

Si esaminerà ora come creare un'app personalizzata che usa il servizio OpenAI di Azure per generare codice. L'app verrà sviluppata usando Visual Studio Code. I file di codice per l'app sono stati forniti in un repository GitHub.

> **Suggerimento**: Se il repository **mslearn-openai** è già stato clonato, aprirlo in Visual Studio Code. In caso contrario, eseguire i passaggi seguenti per clonarlo nell'ambiente di sviluppo.

1. Avviare Visual Studio Code.
2. Aprire il riquadro comandi (MAIUSC+CTRL+P) ed eseguire un comando **Git: Clone** per clonare il repository `https://github.com/MicrosoftLearning/mslearn-openai` in una cartella locale. Non è importante usare una cartella specifica.
3. Dopo la clonazione del repository, aprire la cartella in Visual Studio Code.

    > **Nota**: Se Visual Studio Code visualizza un messaggio popup per chiedere se si considera attendibile il codice che si apre, fare clic sull'opzione **Sì, considero attendibili gli autori** nel popup.

4. Attendere il completamento dell'installazione di file aggiuntivi per supportare i progetti in codice C# nel repository.

    > **Nota**: se viene richiesto di aggiungere gli asset necessari per la compilazione e il debug, selezionare **Non adesso**.

## Configurare l'applicazione

Sono state fornite applicazioni sia per C# che per Python, oltre a un file di testo di esempio che verrà usato per testare il riepilogo. Entrambe le app presentano la stessa funzionalità. Prima di tutto, verranno completate alcune parti chiave dell'applicazione per consentire l'uso della risorsa OpenAI di Azure.

1. Nel riquadro **Esplora risorse** di Visual Studio Code, passare alla cartella **Labfiles/04-code-generation** ed espandere la cartella **CSharp** o **Python** a seconda del linguaggio scelto. Ogni cartella contiene i file specifici del linguaggio per un’app in cui si intende integrare la funzionalità OpenAI di Azure.
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

## Aggiungere codice per usare il modello del servizio OpenAI di Azure

A questo punto, è possibile usare l'SDK di OpenAI di Azure per usare il modello distribuito.

1. Nel riquadro **Esplora risorse**, nella cartella **CSharp** o **Python**, aprire il file di codice per il linguaggio preferito. Nella funzione che chiama il modello OpenAI di Azure, sotto il commento ***Formatta e invia la richiesta al modello***, aggiungere il codice per formattare e inviare la richiesta al modello.

    **C#**: Program.cs

    ```csharp
    // Format and send the request to the model
    var chatCompletionsOptions = new ChatCompletionsOptions()
    {
        Messages =
        {
            new ChatRequestSystemMessage(systemPrompt),
            new ChatRequestUserMessage(userPrompt)
        },
        Temperature = 0.7f,
        MaxTokens = 1000,
        DeploymentName = oaiDeploymentName
    };

    // Get response from Azure OpenAI
    Response<ChatCompletions> response = await client.GetChatCompletionsAsync(chatCompletionsOptions);

    ChatCompletions completions = response.Value;
    string completion = completions.Choices[0].Message.Content;
    ```

    **Python**: code-generation.py

    ```python
    # Format and send the request to the model
    messages =[
        {"role": "system", "content": system_message},
        {"role": "user", "content": user_message},
    ]
    
    # Call the Azure OpenAI model
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0.7,
        max_tokens=1000
    )
    ```

4. Salvare le modifiche apportate al file di codice.

## Eseguire l'applicazione

Ora che l'app è stata configurata, eseguirla per provare a generare codice per ogni caso d'uso. Il caso d'uso è numerato nell'app e può essere eseguito in qualsiasi ordine.

> **Nota**: Alcuni utenti potrebbero riscontrare una limitazione della velocità se il modello viene chiamato troppo frequentemente. Se si verifica un errore relativo a un limite di velocità del token, attendere un minuto e riprovare.

1. Nel riquadro **Explorer** espandere la cartella **Labfiles/04-code-generation/sample-code** ed esaminare la funzione e l'app *go-fish* per la propria lingua. Questi file verranno usati per le attività nell'app.
2. Nel riquadro del terminale interattivo, accertarsi che il contesto della cartella sia la cartella per la lingua dell’interfaccia utente. Immettere, quindi, il comando seguente per eseguire l'applicazione.

    - **C#**: `dotnet run`
    - **Python**: `python code-generation.py`

    > **Suggerimento**: È possibile usare l'icona **Ingrandisci dimensioni pannello** (**^**) nella barra degli strumenti del terminale per visualizzare altro testo della console.

3. Scegliere l'opzione **1** per aggiungere commenti al codice e immettere la richiesta seguente. Si noti che la risposta potrebbe richiedere alcuni secondi per ognuna di queste attività.

    ```prompt
    Add comments to the following function. Return only the commented code.\n---\n
    ```

    I risultati verranno inseriti in **result/app.txt**. Aprire il file e confrontarlo con il file di funzione in **sample-code**.

4. Scegliere quindi l'opzione **2** per scrivere unit test per la stessa funzione e immettere la richiesta seguente.

    ```prompt
    Write four unit tests for the following function.\n---\n
    ```

    I risultati sostituiranno quanto contenuto in **result/app.txt** e descriveranno in dettaglio quattro unit test per tale funzione.

5. Scegliere quindi l'opzione **3** per correggere i bug in un'app per la riproduzione di Go Fish. Immettere la richiesta seguente.

    ```prompt
    Fix the code below for an app to play Go Fish with the user. Return only the corrected code.\n---\n
    ```

    I risultati sostituiranno quanto contenuto in **result/app.txt** e avranno un codice molto simile con alcune correzioni.

    - **C#**: Le correzioni vengono effettuate alla riga 30 e 59
    - **Python**: Le correzioni vengono effettuate alla riga 18 e 31

    L'app per Go Fish in **sample-code** può essere eseguita se si sostituiscono le righe che contengono i bug con la risposta di OpenAI di Azure. Se la si esegue senza le correzioni, non funzionerà correttamente.
    
    > **Nota**: È importante notare che anche se il codice di questa app Go Fish è stato corretto per alcune parti della sintassi, non è una rappresentazione rigorosamente accurata del gioco. Se si osserva attentamente, ci sono problemi legati al fatto che non si controlla se il mazzo è vuoto quando si pescano le carte, non si rimuovono le coppie dalla mano dei giocatori quando ottengono una coppia e alcuni altri bug che richiedono una conoscenza dei giochi di carte per essere realizzati. Questo è un ottimo esempio di come i modelli di intelligenza artificiale generativa utili possano essere utili per facilitare la generazione del codice, ma non possono essere considerati corretti e devono essere verificati dallo sviluppatore.

    Se si vuole visualizzare la risposta completa da OpenAI di Azure, è possibile impostare la variabile **printFullResponse** su `True` ed eseguire nuovamente l'app.

## Eseguire la pulizia

Terminato l'utilizzo della risorsa OpenAI di Azure, ricordarsi di eliminare la distribuzione o l'intera risorsa nel **portale di Azure** all'indirizzo `https://portal.azure.com`.
