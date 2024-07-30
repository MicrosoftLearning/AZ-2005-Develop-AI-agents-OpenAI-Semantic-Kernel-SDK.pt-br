---
lab:
  title: 'Laboratório: Desenvolver agentes de IA usando o OpenAI do Azure e o SDK do Semantic Kernel'
  module: 'Module 01: Build your kernel'
---

# Laboratório: Concluir um agente de viagens com IA
# Manual de laboratório do aluno

Neste laboratório, você concluirá um agente de viagens de IA usando o SDK do Semantic Kernel. Você criará um ponto de extremidade para o serviço de modelo de linguagem grande (LLM), criará funções do Semantic Kernel e usará o recurso de chamada automática de funções do SDK do Semantic Kernel para encaminhar a intenção do usuário para os plug-ins apropriados, incluindo alguns plug-ins predefinidos que foram fornecidos. Você também fornecerá contexto ao LLM usando o histórico de conversas e permitirá que o usuário continue a conversa.

## Cenário do laboratório

Você é desenvolvedor em uma agência de viagens especializada em criar experiências de viagem personalizadas para seus clientes. Você recebeu a tarefa de criar um agente de viagens com IA que possa ajudar os clientes a saber mais sobre destinos de viagem e planejar atividades para suas viagens. O agente de viagens com IA deve ser capaz de converter valores monetários, sugerir destinos e atividades, fornecer frases úteis em diferentes idiomas e traduzir frases. O agente de viagens com IA também deve ser capaz de fornecer respostas contextualmente relevantes às solicitações do usuário usando o histórico de conversas.

## Objetivos

Ao concluir este laboratório, você fará o seguinte:

* Criar um ponto de extremidade para o serviço de modelo de linguagem grande (LLM)
* Criar um objeto Semantic Kernel
* Executar prompts usando o SDK do Semantic Kernel
* Criar funções e plug-ins do Semantic Kernel
* Use o recurso de chamada automática de função do SDK do Semantic Kernel

## Configuração do Laboratório

### Pré-requisitos

Para realizar o exercício, você precisará ter as seguintes ferramentas instaladas no sistema:

* [Visual Studio Code](https://code.visualstudio.com)
* [O SDK mais recente do .NET 7.0](https://dotnet.microsoft.com/download/dotnet/7.0)
* [A extensão do C#](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp) para Visual Studio Code

### Preparar seu ambiente de desenvolvimento

No caso desses exercícios, um projeto inicial está disponível para você usar. Use as seguintes etapas para configurar o projeto inicial:

> [!IMPORTANT]
> Você deve ter o .NET Framework 8.0 instalado, bem como as extensões de extensões do VS Code para C# e o Gerenciador de Pacotes NuGet.

1. Baixe o arquivo zip localizado em `https://github.com/MicrosoftLearning/AZ-2005-Develop-AI-agents-OpenAI-Semantic-Kernel-SDK/blob/master/Allfiles/Labs/02/Lab-02-Starter.zip`.

1. Extraia o conteúdo do arquivo zip para um local fácil de localizar e lembrar, como uma pasta em sua Área de Trabalho.

1. Abra o Visual Studio Code e selecione **Arquivo** > **Abrir pasta.**

1. Navegue até a pasta **Starter** extraída e selecione **Selecionar pasta**.

1. Abra o arquivo **Program.cs** no editor de códigos.

## Exercício 1: Criar um plug-in com o SDK do Semantic Kernel

Para este exercício, você cria um ponto de extremidade para o serviço de modelo de linguagem grande (LLM). O SDK do Kernel Semântico usa esse ponto de extremidade para se conectar ao LLM e executar prompts. O SDK do Kernel Semântico dá suporte a LLMs HuggingFace, OpenAI e Open AI do Azure. N este exemplo, você usa o Open AI do Azure.

**Tempo estimado de conclusão do exercício**: 10 minutos

### Tarefa 1: Criar um recurso do OpenAI do Azure

1. Navegue até [https://portal.azure.com](https://portal.azure.com).

1. Crie um novo recurso do OpenAI do Azure usando as configurações padrão.

    > [!NOTE]
    > Se você já tiver um recurso do OpenAI do Azure, ignore esta etapa.

1. Após o recurso ser criado, escolha **Ir para o recurso**.

1. Na página de **Visão Geral**, selecione **Ir para o Estúdio do OpenAI do Azure**.

:::image type="content" source="../media/model-deployments.png" alt-text="Uma captura de tela da página de implantações do OpenAI do Azure.":::

1. Selecione **Criar implantação** e, em seguida, selecione **+Criar implantação**.

1. No pop-up **Implantar modelo** , selecione **gpt-35-turbo-16k**.

    Usar a versão padrão do Modelo

1. Insira um nome para sua implantação

1. Quando a implantação for concluída, navegue de volta para o recurso de OpenAI do Azure.

1. Em **Gerenciamento de Recursos**, acesse **Chaves e Pontos de Extremidade**.

    Você usará esses valores na próxima tarefa para criar seu kernel. Lembre-se de manter suas chaves privadas e seguras!

1. Retorne ao arquivo **Program.cs** no Visual Studio Code.

1. Atualize as seguintes variáveis com o nome da implantação dos Serviços OpenAI do Azure, a chave da API e o ponto de extremidade

    ```csharp
    string yourDeploymentName = "";
    string yourEndpoint = "";
    string yourApiKey = "";
    ```

    > [!NOTE]
    > O modelo de implantação deve ser "gpt-35-turbo-16k" para que alguns dos recursos semânticos do SDK do Kernel funcionem.

### Tarefa 2: Criar uma função nativa

Nesta tarefa, você criará uma função nativa que pode converter um valor de uma moeda base para uma moeda de destino.

1. Crie um novo arquivo chamado `CurrencyConverter.cs` na pasta **Plugins/ConvertCurrency**

1. No arquivo `CurrencyConverter.cs`, adicione o seguinte código para criar uma função de plug-in:

    ```c#
    using AITravelAgent;
    using System.ComponentModel;
    using Microsoft.SemanticKernel;

    class CurrencyConverter
    {
        [KernelFunction, 
        Description("Convert an amount from one currency to another")]
        public static string ConvertAmount()
        {
            var currencyDictionary = Currency.Currencies;
        }
    }
    ```

    Nesse código, use o decorador `KernelFunction` para declarar sua função nativa. Você também usará o decorador `Description` para adicionar uma descrição do que a função faz. Você pode usar `Currency.Currencies` para obter um dicionário de moedas e suas taxas de câmbio. Em seguida, adicione alguma lógica para converter um determinado valor de uma moeda para outra.

1. Modifique sua função `ConvertAmount` com o seguinte código:

    ```c#
    [KernelFunction, Description("Convert an amount from one currency to another")]
    public static string ConvertAmount(
        [Description("The target currency code")] string targetCurrencyCode, 
        [Description("The amount to convert")] string amount, 
        [Description("The starting currency code")] string baseCurrencyCode)
    {
        var currencyDictionary = Currency.Currencies;
        
        Currency targetCurrency = currencyDictionary[targetCurrencyCode];
        Currency baseCurrency = currencyDictionary[baseCurrencyCode];

        if (targetCurrency == null)
        {
            return targetCurrencyCode + " was not found";
        }
        else if (baseCurrency == null)
        {
            return baseCurrencyCode + " was not found";
        }
        else
        {
            double amountInUSD = Double.Parse(amount) * baseCurrency.USDPerUnit;
            double result = amountInUSD * targetCurrency.UnitsPerUSD;
            return @$"${amount} {baseCurrencyCode} is approximately 
                {result.ToString("C")} in {targetCurrency.Name}s ({targetCurrencyCode})";
        }
    }
    ```

    Nesse código, use o dicionário `Currency.Currencies` para obter o objeto `Currency` para as moedas de destino e base. Em seguida, use o objeto `Currency` para converter o valor da moeda base para a moeda de destino. Por fim, retorne uma cadeia de caracteres com o valor convertido. Em seguida, vamos testar seu plug-in.

1. No arquivo `Program.cs`, importe e invoque sua nova função de plug-in com o seguinte código:

    ```c#
    kernel.ImportPluginFromType<CurrencyConverter>();
    kernel.ImportPluginFromType<ConversationSummaryPlugin>();
    var prompts = kernel.ImportPluginFromPromptDirectory("Prompts");

    var result = await kernel.InvokeAsync("CurrencyConverter", 
        "ConvertAmount", 
        new() {
            {"targetCurrencyCode", "USD"}, 
            {"amount", "52000"}, 
            {"baseCurrencyCode", "VND"}
        }
    );

    Console.WriteLine(result);
    ```

    Nesse código, use o método `ImportPluginFromType` para importar seu plug-in. Em seguida, use o método `InvokeAsync` para invocar sua função de plug-in. O método `InvokeAsync` recebe o nome do plug-in, o nome da função e um dicionário de parâmetros. Por fim, imprima o resultado no console. Em seguida, execute o código para ter certeza de que está funcionando.

1. Insira `dotnet run` no terminal. A seguinte saída deve ser exibida:

    ```output
    $52000 VND is approximately $2.13 in US Dollars (USD)
    ```

    Agora que o plug-in está funcionando corretamente, vamos criar um prompt de linguagem natural que possa detectar quais moedas e valores o usuário deseja converter.

### Tarefa 3: Analisar a entrada do usuário com um prompt

Nesta tarefa, crie um prompt que analise a entrada do usuário para identificar a moeda de destino, a moeda base e o valor a ser convertido.

1. Crie uma nova pasta chamada `GetTargetCurrencies` na pasta **Prompts**

1. Na pasta `GetTargetCurrencies`, crie um novo arquivo chamado `config.json`

1. Insira o seguinte texto no arquivo `config.json`:

    ```output
    {
        "schema": 1,
        "type": "completion",
        "description": "Identify the target currency, base currency, and amount to convert",
        "execution_settings": {
            "default": {
                "max_tokens": 800,
                "temperature": 0
            }
        },
        "input_variables": [
            {
                "name": "input",
                "description": "Text describing some currency amount to convert",
                "required": true
            }
        ]
    }
    ```

1. Na pasta `GetTargetCurrencies`, crie um novo arquivo chamado `skprompt.txt`

1. Insira o seguinte texto no arquivo `skprompt.txt`:

    ```html
    <message role="system">Identify the target currency, base currency, and 
    amount from the user's input in the format target|base|amount</message>

    For example: 

    <message role="user">How much in GBP is 750.000 VND?</message>
    <message role="assistant">GBP|VND|750000</message>

    <message role="user">How much is 60 USD in New Zealand Dollars?</message>
    <message role="assistant">NZD|USD|60</message>

    <message role="user">How many Korean Won is 33,000 yen?</message>
    <message role="assistant">KRW|JPY|33000</message>

    <message role="user">{{$input}}</message>
    <message role="assistant">target|base|amount</message>
    ```

### Tarefa 4: Verifique seu trabalho

Nesta tarefa, execute seu aplicativo e verifique se o código está funcionando corretamente. 

1. Teste o novo prompt atualizando o arquivo `Program.cs` com o seguinte código:

    ```c#
    kernel.ImportPluginFromType<CurrencyConverter>();
    kernel.ImportPluginFromType<ConversationSummaryPlugin>();
    var prompts = kernel.ImportPluginFromPromptDirectory("Prompts");

    var result = await kernel.InvokeAsync(prompts["GetTargetCurrencies"],
        new() {
            {"input", "How many Australian Dollars is 140,000 Korean Won worth?"}
        }
    );

    Console.WriteLine(result);
    ```

1. Insira `dotnet run` no terminal. Você deve ver o seguinte resultado:

    ```output
    AUD|KRW|140000
    ```

    > [!NOTE]
    > Se o código não produzir a saída esperada, você poderá examinar o código na pasta **Solução**. Talvez seja necessário ajustar o prompt no arquivo `skprompt.txt` para produzir resultados mais exatos.

Agora você tem um plug-in que pode converter um valor de uma moeda para outra e um prompt que pode ser usado para analisar a entrada do usuário em um formato que a função `ConvertAmount` pode usar. Isso permitirá que os usuários convertam facilmente os valores monetários usando o agente de viagens com IA.

## Exercício 2: Automatizar a seleção de plug-ins com base na intenção do usuário

Neste exercício, você detecta a intenção do usuário e roteia a conversa para os plug-ins desejados. Você pode usar um plug-in fornecido para recuperar a intenção do usuário. Vamos começar!

**Tempo estimado de conclusão do exercício**: 10 minutos

### Tarefa 1: Usar o plug-in GetIntent

1. Atualize o arquivo `Program.cs` pelo seguinte código:

    ```c#
    kernel.ImportPluginFromType<CurrencyConverter>();
    kernel.ImportPluginFromType<ConversationSummaryPlugin>();
    var prompts = kernel.ImportPluginFromPromptDirectory("Prompts");

    Console.WriteLine("What would you like to do?");
    var input = Console.ReadLine();

    var intent = await kernel.InvokeAsync<string>(
        prompts["GetIntent"], 
        new() {{ "input",  input }}
    );

    ```

    Nesse código, você usa o prompt `GetIntent` para detectar a intenção do usuário. Em seguida, você armazena a intenção em uma variável chamada `intent`. Em seguida, você roteia a intenção para o plug-in `CurrencyConverter`.

1. Adicione o seguinte código ao seu arquivo `Program.cs`:

    ```c#
    switch (intent) {
        case "ConvertCurrency": 
            var currencyText = await kernel.InvokeAsync<string>(
                prompts["GetTargetCurrencies"], 
                new() {{ "input",  input }}
            );
            var currencyInfo = currencyText!.Split("|");
            var result = await kernel.InvokeAsync("CurrencyConverter", 
                "ConvertAmount", 
                new() {
                    {"targetCurrencyCode", currencyInfo[0]}, 
                    {"baseCurrencyCode", currencyInfo[1]},
                    {"amount", currencyInfo[2]}, 
                }
            );
            Console.WriteLine(result);
            break;
        default:
            Console.WriteLine("Other intent detected");
            break;
    }
    ```

    O plug-in `GetIntent` retorna os seguintes valores: ConvertCurrency, SuggestDestinations, SuggestActivities, Translate, HelpfulPhrases, Unknown. Você usa uma instrução `switch` para rotear a intenção do usuário para o plug-in apropriado. 
    
    Se a intenção do usuário for converter moeda, use o prompt `GetTargetCurrencies` para recuperar as informações de moeda. Em seguida, você usa o plug-in `CurrencyConverter` para converter o valor.

    Em seguida, adicione alguns casos para lidar com as outras intenções. Por enquanto, vamos usar o recurso de chamada automática de função do SDK do Kernel Semântico para rotear a intenção para os plug-ins disponíveis.

1. Crie a configuração de chamada automática de função adicionando o seguinte código ao arquivo `Program.cs`:

    ```c#
    kernel.ImportPluginFromType<CurrencyConverter>();
    kernel.ImportPluginFromType<ConversationSummaryPlugin>();
    var prompts = kernel.ImportPluginFromPromptDirectory("Prompts");

    OpenAIPromptExecutionSettings settings = new()
    {
        ToolCallBehavior = ToolCallBehavior.AutoInvokeKernelFunctions
    };

    Console.WriteLine("What would you like to do?");
    var input = Console.ReadLine();
    var intent = await kernel.InvokeAsync<string>(
        prompts["GetIntent"], 
        new() {{ "input",  input }}
    );
    ```

    Em seguida, adicione casos à instrução de alternância para as outras intenções.

1. Atualize o arquivo `Program.cs` pelo seguinte código:

    ```c#
    switch (intent) {
        case "ConvertCurrency": 
            // ...Code you entered previously...
            break;
        case "SuggestDestinations":
        case "SuggestActivities":
        case "HelpfulPhrases":
        case "Translate":
            var autoInvokeResult = await kernel.InvokePromptAsync(input!, new(settings));
            Console.WriteLine(autoInvokeResult);
            break;
        default:
            Console.WriteLine("Other intent detected");
            break;
    }
    ```

    Neste código, você usa a configuração `AutoInvokeKernelFunctions` para chamar automaticamente funções e prompts que são referenciados em seu kernel. Se a intenção do usuário é converter moeda, o plug-in `CurrencyConverter` executa sua tarefa. 
    
    Se a intenção do usuário for obter sugestões de destino ou atividade, traduzir uma frase ou obter frases úteis em um idioma, a configuração `AutoInvokeKernelFunctions` chamará automaticamente os plug-ins existentes que foram incluídos no código do projeto.

    Você também pode adicionar código para executar a entrada do usuário como um prompt para o modelo de linguagem grande (LLM) se ele não se enquadrar em nenhum desses casos de intenção.

1. Atualize o caso padrão com o seguinte código:

    ```c#
    default:
        Console.WriteLine("Sure, I can help with that.");
        var otherIntentResult = await kernel.InvokePromptAsync(input!, new(settings));
        Console.WriteLine(otherIntentResult);
        break;
    ```

    Agora, se o usuário tiver uma intenção diferente, o LLM poderá lidar com a solicitação do usuário. Vamos experimentar!

### Tarefa 2: Verifique seu trabalho

Nesta tarefa, execute seu aplicativo e verifique se o código está funcionando corretamente. 

1. Insira `dotnet run` no terminal. Quando solicitado, insira algum texto semelhante ao seguinte prompt:

    ```output
    What would you like to do?
    How many TTD is 50 Qatari Riyals?    
    ```

1. Você deverá ver uma saída semelhante à seguinte resposta:

    ```output
    $50 QAR is approximately $93.10 in Trinidadian Dollars (TTD)
    ```

1. Insira `dotnet run` no terminal. Quando solicitado, insira algum texto semelhante ao seguinte prompt:

    ```output
    What would you like to do?
    I want to go somewhere that has lots of warm sunny beaches and delicious, spicy food!
    ```

1. Você deverá ver uma saída semelhante à seguinte resposta:

    ```output
    Based on your preferences for warm sunny beaches and delicious, spicy food, I have a few destination recommendations for you:

    1. Thailand: Known for its stunning beaches, Thailand offers a perfect combination of relaxation and adventure. You can visit popular beach destinations like Phuket, Krabi, or Koh Samui, where you'll find crystal-clear waters and white sandy shores. Thai cuisine is famous for its spiciness, so you'll have plenty of mouthwatering options to try, such as Tom Yum soup, Pad Thai, and Green Curry.

    2. Mexico: Mexico is renowned for its beautiful coastal regions and vibrant culture. You can explore destinations like Cancun, Playa del Carmen, or Tulum, which boast stunning beaches along the Caribbean Sea. Mexican cuisine is rich in flavors and spices, offering a wide variety of dishes like tacos, enchiladas, and mole sauces that will satisfy your craving for spicy food.

    ...

    These destinations offer a perfect blend of warm sunny beaches and delicious, spicy food, ensuring a memorable trip for you. Let me know if you need any further assistance or if you have any specific preferences for your trip!
    ```

1. Insira `dotnet run` no terminal. Quando solicitado, insira algum texto semelhante ao seguinte prompt:

    ```output
    What would you like to do?
    Can you give me a recipe for chicken satay?

1. You should see a response similar to the following response:

    ```output
    Sure, I can help with that.
    Certainly! Here's a recipe for chicken satay:

    ...
    ```

    A intenção deve ser direcionada para o seu caso padrão e o LLM deve lidar com a solicitação de uma receita de satay de frango.

    > [!NOTE]
    > Se o código não produzir a saída esperada, você poderá revisar o código na pasta **Solução**.

Em seguida, vamos modificar a lógica de roteamento para fornecer algum histórico de conversas para determinados plug-ins. O fornecimento de histórico permite que os plug-ins recuperem respostas contextualmente relevantes às solicitações do usuário.

### Tarefa 3: Concluir o roteamento do plug-in

Neste exercício, você usa o histórico de conversa para fornecer contexto ao modelo de linguagem grande (LLM). Você também ajusta o código para que ele permita que o usuário continue a conversa, assim como um chatbot real. Vamos começar!

1. Modifique o código para usar um loop `do`-`while` para aceitar a entrada do usuário:

    ```c#
    string input;

    do 
    {
        Console.WriteLine("What would you like to do?");
        input = Console.ReadLine();

        // ...
    }
    while (!string.IsNullOrWhiteSpace(input));
    ```

    Agora você pode manter a conversa em andamento até que o usuário insira uma linha em branco.

1. Capture detalhes sobre a viagem do usuário modificando o caso `SuggestDestinations`:

    ```c#
    case "SuggestDestinations":
        chatHistory.AppendLine("User:" + input);
        var recommendations = await kernel.InvokePromptAsync(input!);
        Console.WriteLine(recommendations);
        break;
    ```

1. Use os detalhes da viagem no caso `SuggestActivities` com o seguinte código:

    ```c#
     case "SuggestActivities":
        var chatSummary = await kernel.InvokeAsync(
            "ConversationSummaryPlugin", 
            "SummarizeConversation", 
            new() {{ "input", chatHistory.ToString() }});
        break;
    ```

    Neste código, você usa a função interna `SummarizeConversation` para resumir o chat com o usuário. Em seguida, vamos usar o resumo para sugerir atividades no destino.

1. Estenda o caso `SuggestActivities`com o seguinte código:

    ```c#
    var activities = await kernel.InvokePromptAsync(
        input,
        new () {
            {"input", input},
            {"history", chatSummary},
            {"ToolCallBehavior", ToolCallBehavior.AutoInvokeKernelFunctions}
    });

    chatHistory.AppendLine("User:" + input);
    chatHistory.AppendLine("Assistant:" + activities.ToString());
    
    Console.WriteLine(activities);
    break;
    ```

    Neste código, você adiciona `input` e `chatSummary` como argumentos de kernel. Em seguida, o kernel invoca o prompt e o encaminha para o plug-in `SuggestActivities`. Você também acrescenta a entrada do usuário e a resposta do assistente ao histórico de chat e exibe os resultados. Em seguida, você precisa adicionar a variável `chatSummary` ao plug-in `SuggestActivities`.

1. Navegue até **Prompts/SuggestActivities/config.json** e abra o arquivo no Visual Studio Code

1. Em `input_variables`, adicione uma variável para o histórico de chat:

    ```json
    "input_variables": [
      {
          "name": "history",
          "description": "Some background information about the user",
          "required": false
      },
      {
          "name": "destination",
          "description": "The destination a user wants to visit",
          "required": true
      }
   ]
   ```

1. Navegue até **Prompts/SuggestActivities/skprompt.txt** e abra o arquivo

1. Substitua a metade inicial do prompt pelo seguinte prompt que usa a variável de histórico de chat:

    ```html 
    You are an experienced travel agent. 
    You are helpful, creative, and very friendly. 
    Consider the traveler's background: {{$history}}
    ```

    Deixe o restante do prompt como está. Agora, o plug-in usa o histórico de chat para fornecer contexto ao LLM.

### Tarefa 4: Verifique seu trabalho

Nesta tarefa, você executa seu aplicativo e verifica se o código está funcionando corretamente.

1. Compare os casos de comutador atualizados com o seguinte código:

    ```c#
    case "SuggestDestinations":
            chatHistory.AppendLine("User:" + input);
            var recommendations = await kernel.InvokePromptAsync(input!);
            Console.WriteLine(recommendations);
            break;
    case "SuggestActivities":

        var chatSummary = await kernel.InvokeAsync(
            "ConversationSummaryPlugin", 
            "SummarizeConversation", 
            new() {{ "input", chatHistory.ToString() }});

        var activities = await kernel.InvokePromptAsync(
            input!,
            new () {
                {"input", input},
                {"history", chatSummary},
                {"ToolCallBehavior", ToolCallBehavior.AutoInvokeKernelFunctions}
        });

        chatHistory.AppendLine("User:" + input);
        chatHistory.AppendLine("Assistant:" + activities.ToString());
        
        Console.WriteLine(activities);
        break;
    ```

1. Insira `dotnet run` no terminal. Quando solicitado, insira um texto semelhante ao seguinte:

    ```output
    What would you like to do?
    How much is 60 USD in new zealand dollars?
    ```

1. Você deve receber alguma saída semelhante à seguinte:

    ```output
    $60 USD is approximately $97.88 in New Zealand Dollars (NZD)
    What would you like to do?
    ```

1. Insira um prompt para sugestões de destino com algumas indicações de contexto, por exemplo:

    ```output
    What would you like to do?
    I'm planning an anniversary trip with my spouse, but they are currently using a wheelchair and accessibility is a must. What are some destinations that would be romantic for us?
    ```

1. Você deve receber alguma saída com recomendações de destinos acessíveis.

1. Insira um prompt para sugestões de atividade, por exemplo:

    ```output
    What would you like to do?
    What are some things to do in Barcelona?
    ```

1. Você deve receber recomendações que se ajustem ao contexto anterior, por exemplo, atividades acessíveis em Barcelona semelhantes às seguintes:

    ```output
    1. Visit the iconic Sagrada Família: This breathtaking basilica is an iconic symbol of Barcelona's architecture and is known for its unique design by Antoni Gaudí.

    2. Explore Park Güell: Another masterpiece by Gaudí, this park offers stunning panoramic views of the city, intricate mosaic work, and whimsical architectural elements.

    3. Visit the Picasso Museum: Explore the extensive collection of artworks by the iconic painter Pablo Picasso, showcasing his different periods and styles.
    ```

    > [!NOTE]
    > Se o código não produzir a saída esperada, você poderá examinar o código na pasta **Solução**.

Você pode continuar a testar o aplicativo com prompts e indicações de contexto diferentes. Ótimo trabalho! Você forneceu com êxito as indicações de contexto para o LLM e ajustou o código para permitir que o usuário continue a conversa.

### Revisão

Neste laboratório, você criou um ponto de extremidade para o serviço de modelo de linguagem grande (LLM), criou um objeto do Semantic Kernel, executou prompts usando o SDK do Semantic Kernel, criou funções e plug-ins do Semantic Kernel e usou o recurso de chamada automática de função do SDK do Semantic Kernel para rotear a intenção do usuário para os plug-ins apropriados. Você também forneceu contexto ao LLM usando o histórico de conversas e permitiu que o usuário continuasse a conversa. Parabéns por ter concluído este laboratório!
