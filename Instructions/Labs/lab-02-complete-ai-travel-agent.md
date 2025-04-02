---
lab:
  title: 'Laboratório: Desenvolver agentes de IA usando o OpenAI do Azure e o SDK do Semantic Kernel'
  module: 'Module 01: Build your kernel'
---

# Laboratório: completar um assistente de viagem de IA
# Manual de laboratório do aluno

Neste laboratório, você vai concluir um agente de viagens de IA usando o SDK do Semantic Kernel. Você criará um ponto de extremidade para o serviço de modelo de linguagem grande (LLM), criará funções do Semantic Kernel e usará o recurso de chamada automática de funções do SDK do Semantic Kernel para encaminhar a intenção do usuário para os plug-ins apropriados, incluindo alguns plug-ins predefinidos que foram fornecidos. Você também fornecerá contexto ao LLM usando o histórico de conversas e permitirá que o usuário continue a conversa.

## Cenário do laboratório

Você é desenvolvedor em uma agência de viagens especializada em criar experiências de viagem personalizadas para seus clientes. Você recebeu a tarefa de criar um assistente de viagens com IA que possa ajudar os clientes a saber mais sobre destinos de viagem e planejar atividades para suas viagens. O agente de viagens com IA deve ser capaz de converter valores de câmbio, sugerir destinos e atividades, fornecer frases úteis em diferentes idiomas e traduzir frases. O agente de viagens com IA também deve ser capaz de fornecer respostas contextualmente relevantes às solicitações do usuário usando o histórico da conversa.

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
* [O SDK mais recente do .NET 8.0](https://dotnet.microsoft.com/en-us/download/dotnet/8.0)
* [A extensão do C#](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp) para Visual Studio Code

### Preparar seu ambiente de desenvolvimento

No caso desses exercícios, um projeto inicial está disponível para você usar. Use as seguintes etapas para configurar o projeto inicial:

> [!IMPORTANT]
> Você deve ter o .NET Framework 8.0 instalado, bem como as extensões de extensões do VS Code para C# e o Gerenciador de Pacotes NuGet.

1. Cole o URL a seguir em uma nova janela do navegador:
   
     `https://github.com/MicrosoftLearning/AZ-2005-Develop-AI-agents-OpenAI-Semantic-Kernel-SDK/blob/master/Allfiles/Labs/02/Lab-02-Starter.zip`

1. Baixe o arquivo zip clicando no botão <kbd>...</kbd> localizado no canto superior direito da página ou aperte <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>S</kbd>.

1. Extraia o conteúdo do arquivo zip para um local fácil de localizar e lembrar, como uma pasta em sua Área de Trabalho.

1. Abra o Visual Studio Code e selecione **Arquivo** > **Abrir pasta.**

1. Navegue até a pasta **Starter** extraída e selecione **Selecionar pasta**.

1. Abra o arquivo **Program.cs** no editor de códigos.

> [!NOTE]
> Se for solicitado a confiar nos arquivos na pasta, selecione **Sim, eu confio nos autores**.

## Exercício 1: Criar um plug-in com o SDK do Semantic Kernel

Para este exercício, você cria um ponto de extremidade para o serviço de modelo de linguagem grande (LLM). O SDK do Kernel Semântico usa esse ponto de extremidade para se conectar ao LLM e executar prompts. O SDK do Kernel Semântico dá suporte a LLMs HuggingFace, OpenAI e Open AI do Azure. N este exemplo, você usa o Open AI do Azure.

**Tempo estimado de conclusão do exercício**: 10 minutos

### Tarefa 1: Criar um recurso do OpenAI do Azure

1. Navegue até [https://portal.azure.com](https://portal.azure.com).

1. Crie um novo recurso do OpenAI do Azure usando as configurações padrão.

    > [!NOTE]
    > Se você já tiver um recurso do OpenAI do Azure, ignore esta etapa.

1. Após o recurso ser criado, escolha **Ir para o recurso**.

1. Na página **Visão geral**, clique em **Ir para o Portal da Fábrica de IA do Azure**.

1. Selecione **Criar Nova Implantação** e, em seguida, **A partir de modelos base**.

1. Na lista de modelos, selecione **gpt-35-turbo-16k**.

1. Selecione **Confirmar**

1. Insira um nome para sua implantação e deixe as opções padrão.

1. Quando a implantação for concluída, navegue de volta para o recurso OpenAI do Azure no portal do Azure.

1. Em **Gerenciamento de Recursos**, acesse **Chaves e Pontos de Extremidade**.

    Você usará os dados aqui na próxima tarefa para criar seu kernel. Lembre-se de manter suas chaves privadas e seguras!

### Tarefa 2: Criar um plug-in nativo

Nesta tarefa, você cria uma função nativa capaz de converter um valor em uma moeda base para uma moeda de destino.

1. Retorne ao seu projeto do Visual Studio Code.

1. Abra o arquivo **appsettings.json** e atualize os valores com a ID do modelo, o ponto de extremidade e a chave de API do Serviço OpenAI do Azure.

    ```json
    {
        "modelId": "gpt-35-turbo-16k",
        "endpoint": "",
        "apiKey": ""
    }
    ```

1. Navegue até o arquivo chamado **CurrencyConverterPlugin.cs** na pasta **Plugins**

1. **No arquivo CurrencyConverterPlugin.cs**, adicione o seguinte código no comentário **Criar uma função de kernel que obtenha a taxa de câmbio**:

    ```c#
    // Create a kernel function that gets the exchange rate
    [KernelFunction("convert_currency")]
    [Description("Converts an amount from one currency to another, for example USD to EUR")]
    public static decimal ConvertCurrency(decimal amount, string fromCurrency, string toCurrency)
    {
        decimal exchangeRate = GetExchangeRate(fromCurrency, toCurrency);
        return amount * exchangeRate;
    }
    ```

    Nesse código, você vai usar o decorador **KernelFunction** para declarar sua função nativa. Você também usará o decorador **Description** para adicionar uma descrição do que a função faz. Em seguida, adicione lógica para converter um determinado valor de uma moeda para outra.

1. Abra o arquivo **Program.cs**

1. Importe o plugin conversor de moeda sob o comentário **Adicionar plugins ao kernel**:

    ```c#
    // Add plugins to the kernel
    kernel.ImportPluginFromType<CurrencyConverterPlugin>();
    ```

    Em seguida, vamos testar seu plug-in.

1. Clique com o botão direito do mouse no arquivo **Program.cs** e clique em "Abrir no Terminal Integrado"

1. Insira `dotnet run` no terminal. 

    Insira uma solicitação imediata para converter moeda, por exemplo, "Quanto é 10 USD em Hong Kong?"

    Você verá algo semelhante à seguinte saída:

    ```output
    Assistant: 10 USD is equivalent to 77.70 Hong Kong dollars (HKD).
    ```

## Exercício 2: Criar um prompt de Handlebars

Neste exercício, você criará uma função usando um prompt de Handlebars. A função solicitará que o LLM crie um itinerário de viagem para o usuário. Vamos começar!

**Tempo estimado de conclusão do exercício**: 10 minutos

### Tarefa 1: Criar uma função a partir de um prompt de Handlebars

1. Adicione a seguinte diretiva `using` ao arquivo **Program.cs**:

    `using Microsoft.SemanticKernel.PromptTemplates.Handlebars;`

1. Adicione o seguinte código no comentário **Criar um prompt handlebars**:

    ```c#
    // Create a handlebars prompt
    string hbprompt = """
        <message role="system">Instructions: Before providing the user with a travel itinerary, ask how many days their trip is</message>
        <message role="user">I'm going to {{city}}. Can you create an itinerary for me?</message>
        <message role="assistant">Sure, how many days is your trip?</message>
        <message role="user">{{input}}</message>
        <message role="assistant">
        """;
    ```

    Neste código, você criará um prompt de few-shot usando o formato de modelo Handlebars. O prompt orientará o modelo a obter mais informações do usuário antes de criar um itinerário de viagem.

1. Adicione o seguinte código no comentário **Criar a configuração do modelo de prompt usando o formato handlebars**:

    ```c#
    // Create the prompt template config using handlebars format
    var templateFactory = new HandlebarsPromptTemplateFactory();
    var promptTemplateConfig = new PromptTemplateConfig()
    {
        Template = hbprompt,
        TemplateFormat = "handlebars",
        Name = "GetItinerary",
    };
    ```

    Esse código cria uma configuração de modelo Handlebars a partir do prompt. Você pode usá-lo para criar uma função de plugin.

1. Adicione o seguinte código no comentário **Criar uma função de plugin no prompt**: 

    ```c#
    // Create a plugin function from the prompt
    var promptFunction = kernel.CreateFunctionFromPrompt(promptTemplateConfig, templateFactory);
    var itineraryPlugin = kernel.CreatePluginFromFunctions("TravelItinerary", [promptFunction]);
    kernel.Plugins.Add(itineraryPlugin);
    ```

    Esse código cria uma função de plugin para o prompt e a adiciona ao kernel. Agora você já pode invocar a função.

1. No terminal, insira `dotnet run` para executar o código.

    Tente a seguinte entrada para solicitar ao LLM um itinerário.

    ```output
    Assistant: How may I help you?
    User: I'm going to Hong Kong, can you create an itinerary for me?
    Assistant: Sure! How many days will you be staying in Hong Kong?
    User: 10
    Assistant: Great! Here's a 10-day itinerary for your trip to Hong Kong:
    ...
    ```

    Agora você tem o início de um assistente de viagem de IA!. Vamos usar prompts e plug-ins para adicionar mais recursos

1.  Adicione o plugin de reserva de voo sob o comentário **Adicionar plugins ao kernel**:

    ```c#
    // Add plugins to the kernel
    kernel.ImportPluginFromType<CurrencyConverterPlugin>();
    kernel.ImportPluginFromType<FlightBookingPlugin>();
    ```

    Este plug-in simula reservas de voos usando o arquivo **flights.json** com detalhes fictícios. Em seguida, adicione alguns prompts adicionais do sistema ao assistente.

1.  Adicione o seguinte código no comentário **Adicionar mensagens de sistema ao bate-papo**:

    ```c#
    // Add system messages to the chat
    history.AddSystemMessage("The current date is 01/10/2025");
    history.AddSystemMessage("You are a helpful travel assistant.");
    history.AddSystemMessage("Before providing destination recommendations, ask the user about their budget.");
    ```

    Esses prompts ajudarão a criar uma experiência de usuário tranquila e a simular o plug-in de reserva de voos. Agora você já pode testar seu código.

1. Insira `dotnet run` no terminal.

    Tente inserir alguns dos seguintes prompts:

    ```output
    1. Can you give me some destination recommendations for Europe?
    2. I want to go to Barcelona, can you create an itinerary for me?
    3. How many Euros is 100 USD?
    4. Can you book me a flight to Barcelona?
    ```

    Experimente outras entradas e veja como seu assistente de viagem responde.

## Exercício 3: Pedir o consentimento do usuário para ações

Neste exercício, você adicionará uma função de invocação de filtro que solicita aprovação do usuário antes de permitir que o agente reserve um voo em seu nome. Vamos começar!

### Tarefa 1: Criar um filtro de invocação de função

1. Crie um novo arquivo chamado **PermissionFilter.cs**.

1. No novo arquivo, insira o código a seguir:

    ```c#
    #pragma warning disable SKEXP0001 
    using Microsoft.SemanticKernel;
    
    public class PermissionFilter : IFunctionInvocationFilter
    {
        public async Task OnFunctionInvocationAsync(FunctionInvocationContext context, Func<FunctionInvocationContext, Task> next)
        {
            // Check the plugin and function names
            
            await next(context);
        }
    }
    ```

    >[!NOTE] 
    > Na versão 1.30.0 do SDK do Semantic Kernel, os filtros de função estão sujeitos a alterações e exigem uma supressão de aviso. 

    Neste código, você implementará a interface `IFunctionInvocationFilter`. O `OnFunctionInvocationAsync`método é sempre chamado quando uma função é invocada em um assistente de IA.

1. Adicione o seguinte código para detectar quando a função `book_flight` é invocada:

    ```c#
    // Check the plugin and function names
    if ((context.Function.PluginName == "FlightBookingPlugin" && context.Function.Name == "book_flight"))
    {
        // Request user approval

        // Proceed if approved
    }
    ```

    Esse código usa o `FunctionInvocationContext` para determinar qual plug-in e função foram invocados.

1. Adicione a seguinte lógica para solicitar a permissão do usuário para reservar o voo:

    ```c#
    // Request user approval
    Console.WriteLine("System Message: The assistant requires an approval to complete this operation. Do you approve (Y/N)");
    Console.Write("User: ");
    string shouldProceed = Console.ReadLine()!;

    // Proceed if approved
    if (shouldProceed != "Y")
    {
        context.Result = new FunctionResult(context.Result, "The operation was not approved by the user");
        return;
    }
    ```

1. Navegue até o arquivo **Program.cs**.

1. Adicione o seguinte código no comentário **Adicionar filtros ao kernel**:

    ```c#
    // Add filters to the kernel
    kernel.FunctionInvocationFilters.Add(new PermissionFilter());
    ```

1. Insira `dotnet run` no terminal.

    Insira um prompt para reservar um voo. Você deverá ver uma resposta semelhante à seguinte:

    ```output
    User: Find me a flight to Tokyo on the 19
    Assistant: I found a flight to Tokyo on the 19th of January. The flight is with Air Japan and the price is $1200.
    User: Y
    System Message: The assistant requires an approval to complete this operation. Do you approve (Y/N)
    User: N
    Assistant: I'm sorry, but I am unable to book the flight for you.
    ```

    O agente deve pedir a aprovação do usuário antes de prosseguir com uma reserva.

### Revisão

Neste laboratório, você criou um ponto de extremidade para o serviço LLM (grande modelo de linguagem), criou um objeto do Kernel Semântico e executou prompts usando o SDK do Semantic Kernel. Você também criou plug-ins e usou as mensagens do sistema para orientar o modelo. Parabéns por ter concluído este laboratório!
