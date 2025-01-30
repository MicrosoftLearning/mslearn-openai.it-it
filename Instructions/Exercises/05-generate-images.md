---
lab:
  title: Generare immagini con un modello DALL-E
---

# Generare immagini con un modello DALL-E

Il servizio Azure OpenAI include un modello di generazione di immagini denominato DALL-E. È possibile usare questo modello per inviare richieste di linguaggio naturale che descrivono un'immagine desiderata e il modello genererà un'immagine originale in base alla descrizione specificata.

In questo esercizio si userà un modello DALL-E versione 3 per generare immagini in base ai prompt del linguaggio naturale.

Questo esercizio richiederà circa **25** minuti.

## Effettuare il provisioning di una risorsa OpenAI di Azure

Prima di poter usare OpenAI di Azure per generare immagini, è necessario effettuare il provisioning di una risorsa OpenAI di Azure della sottoscrizione di Azure. La risorsa deve trovarsi in un'area in cui sono supportati i modelli DALL-E.

1. Accedere al **portale di Azure** all'indirizzo `https://portal.azure.com`.
1. Creare una risorsa **OpenAI di Azure** con le impostazioni seguenti:
    - **Sottoscrizione**: *Selezionare una sottoscrizione di Azure approvata per l'accesso al Servizio OpenAI, tra cui DALL-E*
    - **Gruppo di risorse**: *Scegliere o creare un gruppo di risorse*
    - **Area**: *scegliere **Stati Uniti orientali** o **Svezia centrale**\*
    - **Nome**: *nome univoco di propria scelta*
    - **Piano tariffario**: Standard S0.

    > \* Modelli DALL-E 3 sono disponibili solo nelle risorse del servizio Azure OpenAI nelle aree **Stati Uniti orientali** e **Svezia centrale**.

1. Attendere il completamento della distribuzione. Passare quindi alla risorsa OpenAI di Azure distribuita nel portale di Azure.
1. Nella pagina **Panoramica** per la risorsa Azure OpenAI, scorrere verso il basso fino alla sezione **Attività iniziali** e selezionare il pulsante per passare al **Portale Fonderia AI** (in precedenza Studio AI).
1. Nel portale di Azure AI Foundry, nel riquadro a sinistra, selezionare la pagina **Distribuzioni** e visualizzare le distribuzioni di modelli esistenti. Se non ce n'è già una disponibile per DALL-E 3, creare una nuova distribuzione del modello **dall-e-3** con le impostazioni seguenti:
    - **Nome della distribuzione**: dalle3
    - **Versione del modello**: *usare la versione predefinita*
    - **Tipo di distribuzione**: Standard
    - **Unità di capacità**: 1K
    - **Filtro contenuto**: Predefinito
    - **Abilitare la quota dinamica**: disabilitato
1. Dopo la distribuzione, tornare alla pagina **Immagini** nel riquadro a sinistra.

## Esplorare la generazione di immagini nel playground delle immagini

È possibile usare il playground delle immagini nel **Portale Fonderia Azure AI** per sperimentare la generazione di immagini.

1. Nella sezione **Playground immagini** la distribuzione di DALL-E 3 dovrebbe essere selezionata automaticamente. Se così non fosse, selezionarla dall'elenco a discesa delle distribuzioni.
1. Nella casella **Prompt** immettere una descrizione di un'immagine da generare. Ad esempio, `An elephant on a skateboard` Selezionare quindi **Genera** e visualizzare l'immagine generata.

    ![Playground delle immagini nel Portale Fonderia Azure AI con un'immagine generata.](../media/images-playground.png)

1. Modificare il prompt per fornire una descrizione più specifica. Ad esempio, `An elephant on a skateboard in the style of Picasso`. Generare quindi la nuova immagine ed esaminare i risultati.

    ![Playground delle immagini nel Portale Fonderia Azure AI con due immagini generate.](../media/images-playground-new-style.png)

## Usare l'API REST per generare immagini

Il servizio Azure OpenAI fornisce un'API REST che è possibile usare per inviare richieste di generazione di contenuti, incluse le immagini generate da un modello DALL-E.

### Prepararsi allo sviluppo di un'app in Visual Studio Code

Si esaminerà ora come creare un'app personalizzata che usa il servizio Azure OpenAI per generare immagini. L'app verrà sviluppata usando Visual Studio Code. I file di codice per l'app sono stati forniti in un repository GitHub.

> **Suggerimento**: Se il repository **mslearn-openai** è già stato clonato, aprirlo in Visual Studio Code. In caso contrario, eseguire i passaggi seguenti per clonarlo nell'ambiente di sviluppo.

1. Avviare Visual Studio Code.
2. Aprire il riquadro comandi (MAIUSC+CTRL+P) ed eseguire un comando **Git: Clone** per clonare il repository `https://github.com/MicrosoftLearning/mslearn-openai` in una cartella locale. Non è importante usare una cartella specifica.
3. Dopo la clonazione del repository, aprire la cartella in Visual Studio Code.

    > **Nota**: Se Visual Studio Code visualizza un messaggio popup per chiedere se si considera attendibile il codice che si apre, fare clic sull'opzione **Sì, considero attendibili gli autori** nel popup.

4. Attendere il completamento dell'installazione di file aggiuntivi per supportare i progetti in codice C# nel repository.

    > **Nota**: se viene richiesto di aggiungere gli asset necessari per la compilazione e il debug, selezionare **Non adesso**.

### Configurare l'applicazione

Sono state fornite applicazioni sia per C# che per Python. Entrambe le app presentano la stessa funzionalità. Prima di tutto, si aggiungeranno l'endpoint e la chiave per la risorsa OpenAI di Azure al file di configurazione dell'app.

1. Nel riquadro **Esplora risorse** di Visual Studio Code, passare alla cartella **Labfiles/05-image-generation** ed espandere la cartella **CSharp** o **Python** a seconda del linguaggio scelto. Ogni cartella contiene i file specifici del linguaggio per un’app in cui si intende integrare la funzionalità OpenAI di Azure.
2. Nel riquadro **Esplora risorse**, nella cartella **CSharp** o **Python**, aprire il file di configurazione per la lingua preferita

    - **C#**: appsettings.json
    - **Python**: .env
    
3. Aggiornare i valori di configurazione per includere l'**endpoint** e la **chiave** dalla risorsa Azure OpenAl creata (disponibile nella pagina **Chiavi ed endpoint** per la risorsa Azure OpenAI nel portale di Azure).
4. Salvare il file di configurazione.

### Visualizzare il codice dell'applicazione

Ora si è pronti per esplorare il codice usato per chiamare l'API REST e generare un'immagine.

1. Nel riquadro ** Explorer** selezionare il file di codice principale per l'applicazione:

    - C#: `Program.cs`
    - Python: `generate-image.py`

2. Esaminare il codice che il file contiene, notando le funzionalità principali seguenti:
    - Il codice effettua una richiesta HTTPS all'endpoint per il servizio, inclusa la chiave per il servizio nell'intestazione. Entrambi questi valori vengono ottenuti dal file di configurazione.
    - La richiesta include alcuni parametri, tra cui la richiesta dei parametri sui quali l'immagine deve essere basata, il numero di immagini da generare e le dimensioni delle immagini generate.
    - La risposta include una richiesta rivista che il modello DALL-E ha estrapolato dal prompt fornito dall'utente per renderlo più descrittivo e l'URL per l'immagine generata.
    
    > **Importante**: se alla distribuzione è stato assegnato un nome diverso da quello consigliato, cioè *dalle3*, sarà necessario aggiornare il codice per usare il nome della distribuzione.

### Eseguire l'app

Dopo aver esaminato il codice, è possibile eseguirlo e generare alcune immagini.

1. Fare clic con il pulsante destro del mouse sulla cartella **CSharp** o **Python** contenente i file di codice, quindi aprire un terminale integrato. Immettere quindi il comando appropriato per eseguire l'applicazione:

   **C#**
   ```
   dotnet run
   ```
   
   **Python**
   ```
   pip install requests
   python generate-image.py
   ```

3. Quando richiesto, immettere una descrizione per un'immagine. Ad esempio, *Una giraffa che fa volare un aquilone*.

4. Attendere che l'immagine venga generata. Nel riquadro del terminale verrà visualizzato un collegamento ipertestuale. Selezionare quindi il collegamento ipertestuale per aprire una nuova scheda del browser ed esaminare l'immagine generata.

   > **SUGGERIMENTO**: Se l'app non restituisce una risposta, attendere un minuto e riprovare. La disponibilità delle risorse appena distribuite può richiedere fino a 5 minuti.

5. Chiudere la scheda del browser contenente l'immagine generata ed eseguire nuovamente l'app per generare una nuova immagine con un prompt diverso.

## Eseguire la pulizia

Terminato l’utilizzo della risorsa OpenAI di Azure, ricordarsi di eliminare la risorsa nel **portale di Azure** all'indirizzo `https://portal.azure.com`.
