---
lab:
  title: 'Laboratório: Desenvolver agentes de IA usando o OpenAI do Azure e o SDK do Semantic Kernel'
  module: 'Module 01: Build your kernel'
---

# Laboratório: Criar um assistente de recomendação de música com IA
# Manual de laboratório do aluno

Neste laboratório, você criará o código de um assistente de IA que pode gerenciar a biblioteca de músicas do usuário e oferecer recomendações personalizadas de músicas e shows. Você usará o SDK do Semantic Kernel para criar o assistente de IA e conectá-lo ao serviço de LLM (grande modelo de linguagem). O SDK do Semantic Kernel permite que você crie um aplicativo inteligente que possa interagir com o serviço LLM e fornecer recomendações personalizadas ao usuário.

## Cenário do laboratório

Você é desenvolvedor de um serviço internacional de streaming de áudio. Recebeu a tarefa de integrar o serviço à IA para oferecer aos usuários uma experiência mais personalizada. A IA deve ser capaz de recomendar músicas e as próximas turnês de artistas com base no histórico de audição e nas preferências do usuário. Você decide usar o SDK do Semantic Kernel para criar um assistente de IA que possa interagir com o serviço de LLM(grande modelo de linguagem).

## Objetivos

Ao concluir este laboratório, você fará o seguinte:

* Criar um ponto de extremidade para o serviço de modelo de linguagem grande (LLM)
* Criar um objeto Semantic Kernel
* Executar prompts usando o SDK do Semantic Kernel
* Criar funções e plug-ins do Semantic Kernel
* Habilite a chamada automática de funções para automatizar tarefas

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
   
     `https://github.com/MicrosoftLearning/AZ-2005-Develop-AI-agents-OpenAI-Semantic-Kernel-SDK/blob/master/Allfiles/Labs/01/Lab-01-Starter.zip`

1. Baixe o arquivo zip clicando no botão `...` localizado no canto superior direito da página ou pressione <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>S</kbd>.

1. Extraia o conteúdo do arquivo zip para um local fácil de localizar e lembrar, como uma pasta em sua Área de Trabalho.

1. Abra o Visual Studio Code e selecione **Arquivo** > **Abrir pasta.**

1. Navegue até a pasta **Starter** extraída e selecione **Selecionar pasta**.

1. Abra o arquivo **Program.cs** no editor de códigos.

> [!NOTE]
> Se for solicitado a confiar nos arquivos na pasta, selecione **Sim, eu confio nos autores**. 

## Exercício 1: Executar um prompt com o SDK do Semantic Kernel

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

### Tarefa 2: Criar o kernel

Neste exercício, você aprenderá a criar seu primeiro projeto SDK do Kernel Semântico. Você aprenderá a criar um novo projeto, adicionar o pacote NuGet do SDK do Kernel Semântico e adicionar uma referência ao SDK do Kernel Semântico. Vamos começar!

1. Retorne ao seu projeto do Visual Studio Code.

1. Abra o arquivo **appsettings.json** e atualize os valores com a ID do modelo, o ponto de extremidade e a chave de API do Serviço OpenAI do Azure.

    ```json
    {
        "modelId": "gpt-35-turbo-16k",
        "endpoint": "",
        "apiKey": ""
    }
    ```

1. Abra o Terminal selecionando **Terminal** > **Novo Terminal**.

1. No Terminal, execute o seguinte comando para instalar o SDK do Kernel Semântico:

    `dotnet add package Microsoft.SemanticKernel --version 1.30.0`

1. 1. Adicione as seguintes diretivas `using` ao arquivo **Program.cs**:

    ```c#
    using Microsoft.SemanticKernel;
    using Microsoft.SemanticKernel.ChatCompletion;
    using Microsoft.SemanticKernel.Connectors.OpenAI;
    ```

1. Adicione o seguinte código no comentário **Create a kernel builder with Azure OpenAI chat completion**:

    ```c#
    // Create a kernel builder with Azure OpenAI chat completion
    var builder = Kernel.CreateBuilder();
    builder.AddAzureOpenAIChatCompletion(modelId, endpoint, apiKey);
    ```

1. Construa o kernel adicionando este código no comentário **Build the kernel**:

    ```c#
    // Build the kernel
    var kernel = builder.Build();
    ```

1. Adicione o seguinte código no comentário **Verify the endpoint and run a prompt**:

    ```c#
    // Verify the endpoint and run a prompt
    var result = await kernel.InvokePromptAsync("Who are the top 5 most famous musicians in the world?");
    Console.WriteLine(result);
    ```

1. Clique com o botão direito do mouse na pasta **Iniciador** e selecione **Abrir no Terminal Integrado**.

1. No terminal, insira `dotnet run` para executar o código.

    Verifique se você vê uma resposta do modelo do OpenAI do Azure contendo os cinco músicos mais famosos do mundo.

    A resposta vem do modelo de IA aberta do Azure que você passou para o kernel. O SDK do Kernel Semântico consegue se conectar ao LLM (modelo de linguagem grande) e executar a solicitação. Observe a rapidez com que você foi capaz de receber respostas do LLM. O SDK do Kernel Semântico torna a criação de aplicativos inteligentes fácil e eficiente.

Você pode remover o código de verificação depois de confirmar sua resposta.

## Exercício 2: Criar plug-ins personalizados para biblioteca de música

Neste exercício, você criará plug-ins personalizados para sua biblioteca de música. Você cria funções que podem adicionar músicas à lista de músicas reproduzidas recentemente pelo usuário, obter a lista de músicas reproduzidas recentemente e fornecer recomendações personalizadas de músicas. Você também criará uma função que sugere um show com base na localização do usuário e nas músicas que ele ouviu recentemente.

**Tempo estimado de conclusão do exercício**: 15 minutos

### Tarefa 1: Criar um plug-in de biblioteca de música

Nesta tarefa, você criará um plug-in que permite adicionar músicas à lista de músicas tocadas recentemente pelo usuário e obter a lista das músicas tocadas recentemente. Para simplificar, as músicas reproduzidas recentemente serão armazenadas em um arquivo de texto.

1. Na pasta **Plugins**, abra o arquivo **MusicLibraryPlugin.cs**

1. No comentário **Create a kernel function to get recently played songs**, adicione o decorador de função de kernel:


    ```c#
    // Create a kernel function to get recently played songs
    [KernelFunction("GetRecentPlays")]
    public static string GetRecentPlays()
    ```

    O decorador `KernelFunction` declara a sua função nativa. Você usará um nome descritivo para a função para que a IA possa chamá-la corretamente. A lista de músicas reproduzidas recentemente pelo usuário será armazenada em um arquivo de texto chamado "RecentlyPlayed.txt".

1. No comentário **Create a kernel function to add a song to the recently played list**, adicione o decorador de função de kernel:

    ```c#
    // Create a kernel function to add a song to the recently played list
    [KernelFunction("AddToRecentPlays")]
    public static string AddToRecentlyPlayed(string artist,  string song, string genre)
    ```

    Quando essa classe de plug-in for adicionada ao kernel, ela será capaz de identificar e invocar as funções.

1. Navegue até o arquivo **Program.cs**.

1. Adicione o seguinte código em **Importar plug-ins para o kernel**:

    ```c#
    // Import plugins to the kernel
    kernel.ImportPluginFromType<MusicLibraryPlugin>();
    ```

1. No comentário **Create prompt execution settings**, adicione o seguinte código para invocar automaticamente a função:

    ```c#
    // Create prompt execution settings
    OpenAIPromptExecutionSettings openAIPromptExecutionSettings = new() 
    {
        FunctionChoiceBehavior = FunctionChoiceBehavior.Auto()
    };
    ```

    Usar essa configuração permitirá que o kernel invoque funções automaticamente sem a necessidade de especificá-las no prompt.

1. Adicione o seguinte código no comentário **Get chat completion service**:

    ```c#
    // Get chat completion service.
    var chatCompletionService = kernel.GetRequiredService<IChatCompletionService>();
    ChatHistory chatHistory = [];
    ```

1. Adicione o seguinte código no comentário **Create a helper function to await and output the reply from the chat completion service**:

    ```c#
    // Create a helper function to await and output the reply from the chat completion service
    async Task GetAssistantReply() {
        ChatMessageContent reply = await chatCompletionService.GetChatMessageContentAsync(
            chatHistory,
            kernel: kernel,
            executionSettings: openAIPromptExecutionSettings
        );
        chatHistory.AddAssistantMessage(reply.ToString());
        Console.WriteLine(reply.ToString());
    }
    ```


1. Adicione o seguinte código no comentário **Add system messages to the chat**:

    ```c#
    // Add system messages to the chat
    chatHistory.AddSystemMessage("When a user has played a song, add it to their list of recent plays.");
    chatHistory.AddSystemMessage("The listener has just played the song Danse by Tiara. It falls under these genres: French pop, electropop, pop.");
    await GetAssistantReply();
    ```

1. Execute o código inserindo `dotnet run` no terminal.

    Os prompts de mensagem do sistema que você adicionou devem invocar as funções do plug-in e devem ver a seguinte saída:

    ```output
    Added 'Danse' to recently played
    ```

    Se você abrir o arquivo **Files/RecentlyPlayed.txt**, verá a nova música adicionada à lista.

### Tarefa 2: Fornecer recomendações personalizadas de músicas

Nesta tarefa, você criará um prompt que fornece recomendações de músicas personalizadas para o usuário com base nas músicas reproduzidas recentemente. O prompt combina funções nativas para gerar uma recomendação de música. Você também criará uma função a partir do prompt para torná-lo reutilizável.

1. No arquivo **Program.cs**, adicione o seguinte código no comentário **Create a song suggester function using a prompt**:

    ```c#
    // Create a song suggester function using a prompt
    var songSuggesterFunction = kernel.CreateFunctionFromPrompt(
        promptTemplate: @"This is a list of music available to the user:
        {{MusicLibraryPlugin.GetMusicLibrary}} 

        This is a list of music the user has recently played:
        {{MusicLibraryPlugin.GetRecentPlays}}

        Based on their recently played music, suggest a song from
        the list to play next",
        functionName: "SuggestSong",
        description: "Suggest a song to the user"
    );

    kernel.Plugins.AddFromFunctions("SuggestSongPlugin", [songSuggesterFunction]);
    ```

    Neste código, você criará uma função a partir do prompt e a adicionará aos plug-ins do kernel.

1. No comentário **Invoke the song suggester function with a prompt from the user***, adicione o seguinte código:

    ```c#
    // Invoke the song suggester function with a prompt from the user
    chatHistory.AddUserMessage("What song should I play next?");
    await GetAssistantReply();
    ```

    Agora seu aplicativo pode invocar automaticamente suas funções de plug-in de acordo com a solicitação do usuário. Vamos estender esse código para fornecer recomendações de shows também com base nas informações do usuário.

### Tarefa 3: Fornecer recomendações personalizadas de shows

Nesta tarefa, você criará um plug-in que solicita ao LLM que sugira um show com base nas músicas que o usuário ouviu recentemente e na localização.

1. No arquivo **Program.cs**, importe o plug-in shows no comentário **Import plugins to the kernel**:

    ```c#
    // Import plugins to the kernel 
    kernel.ImportPluginFromType<MusicLibraryPlugin>();
    kernel.ImportPluginFromType<MusicConcertsPlugin>();
    ```

1. Adicione o seguinte código no comentário **Create a concert suggester function using a prompt**:

    ```c#
    // Create a concert suggester function using a prompt
    var concertSuggesterFunction = kernel.CreateFunctionFromPrompt(
        promptTemplate: @"This is a list of the user's recently played songs:
        {{MusicLibraryPlugin.GetRecentPlays}}

        This is a list of upcoming concert details:
        {{MusicConcertsPlugin.GetConcerts}}

        Suggest an upcoming concert based on the user's recently played songs. 
        The user lives in {{$location}}, 
        please recommend a relevant concert that is close to their location.",
        functionName: "SuggestConcert",
        description: "Suggest a concert to the user"
    );

    kernel.Plugins.AddFromFunctions("SuggestConcertPlugin", [concertSuggesterFunction]);
    ```

    Este prompt de função pega a biblioteca de músicas e as informações dos próximos shows, bem como um local do usuário e fornece uma recomendação.

1. Adicione o seguinte prompt para invocar a nova função de plug-in:

    ```c#
    // Invoke the concert suggester function with a prompt from the user
    chatHistory.AddUserMessage("Can you recommend a concert for me? I live in Washington");
    await GetAssistantReply();
    ```

1. Insira `dotnet run` no terminal

    Você deverá ver uma saída semelhante à seguinte resposta:

    ```output
    I recommend you attend the concert of Lisa Taylor. She will be performing in Seattle, Washington, USA on 2/22/2024. Enjoy the show!
    ```
    
    A resposta do LLM pode variar. Tente ajustar o prompt e o local para ver quais outros resultados você pode gerar.

Agora seu assistente pode executar ações diferentes automaticamente com base na entrada do usuário. Ótimo trabalho!

### Revisão

Neste laboratório, você criou um assistente de IA que pode gerenciar a biblioteca de músicas do usuário e oferecer recomendações personalizadas de músicas e shows. Você usou o SDK do Semantic Kernel para criar o assistente de IA e conectá-lo ao serviço de LLM (grande modelo de linguagem). Você criou plug-ins personalizados para sua biblioteca de músicas e ativou a chamada automática de funções para fazer o assistente responder de forma dinâmica à entrada de usuário. Parabéns por ter concluído este laboratório!
