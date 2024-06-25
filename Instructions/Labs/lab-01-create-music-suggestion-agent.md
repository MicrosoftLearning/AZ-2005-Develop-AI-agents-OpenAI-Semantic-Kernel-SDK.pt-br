---
lab:
  title: 'Laboratório: Desenvolver agentes de IA usando o OpenAI do Azure e o SDK do Semantic Kernel'
  module: 'Module 01: Build your kernel'
---

# Laboratório: Criar um agente de recomendação de música com IA
# Manual de laboratório do aluno

Neste laboratório, você criará o código de um agente de IA que pode gerenciar a biblioteca de músicas do usuário e oferecer recomendações personalizadas de músicas e shows. Você usará o SDK do Semantic Kernel para criar o agente de IA e conectá-lo ao serviço de modelo de linguagem grande (LLM). O SDK do Semantic Kernel permite que você crie um aplicativo inteligente que possa interagir com o serviço LLM e fornecer recomendações personalizadas ao usuário.

## Cenário do laboratório

Você é desenvolvedor de um serviço internacional de streaming de áudio. Recebeu a tarefa de integrar o serviço à IA para oferecer aos usuários uma experiência mais personalizada. A IA deve ser capaz de recomendar músicas e as próximas turnês de artistas com base no histórico de audição e nas preferências do usuário. Você decide usar o SDK do Semantic Kernel para criar um agente de IA que possa interagir com o serviço de modelo de linguagem grande (LLM).

## Objetivos

Ao concluir este laboratório, você fará o seguinte:

* Criar um ponto de extremidade para o serviço de modelo de linguagem grande (LLM)
* Criar um objeto Semantic Kernel
* Executar prompts usando o SDK do Semantic Kernel
* Criar funções e plug-ins do Semantic Kernel
* Usar o planejador Handlebars para automatizar tarefas

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

1. Baixe o arquivo zip localizado em `https://github.com/MicrosoftLearning/AZ-2005-Develop-AI-agents-OpenAI-Semantic-Kernel-SDK/blob/master/Allfiles/Labs/01/Lab-01-Starter.zip`.

1. Extraia o conteúdo do arquivo zip para um local fácil de localizar e lembrar, como uma pasta em sua Área de Trabalho.

1. Abra o Visual Studio Code e selecione **Arquivo** > **Abrir pasta.**

1. Navegue até a pasta **Starter** extraída e selecione **Selecionar pasta**.

1. Abra o arquivo **Program.cs** no editor de códigos.

## Exercício 1: Executar um prompt com o SDK do Semantic Kernel

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

1. Selecione **Criar Nova Implantação** e, em seguida, **Implantar Modelo**.

1. Em **Selecionar um modelo**, selecione **gpt-35-turbo-16k**.

    Usar a versão padrão do Modelo

1. Insira um nome para sua implantação

1. Quando a implantação for concluída, navegue de volta para o recurso de OpenAI do Azure.

1. Em **Gerenciamento de Recursos**, acesse **Chaves e Pontos de Extremidade**.

    Você usará os dados aqui na próxima tarefa para criar seu kernel. Lembre-se de manter suas chaves privadas e seguras!

### Tarefa 2: Criar o kernel

Neste exercício, você aprenderá a criar seu primeiro projeto SDK do Kernel Semântico. Você aprenderá a criar um novo projeto, adicionar o pacote NuGet do SDK do Kernel Semântico e adicionar uma referência ao SDK do Kernel Semântico. Vamos começar!

1. Retorne ao seu projeto do Visual Studio Code.

1. Abra o Terminal selecionando **Terminal** > **Novo Terminal**.

1. No Terminal, execute o seguinte comando para instalar o SDK do Kernel Semântico:

    `dotnet add package Microsoft.SemanticKernel --version 1.2.0`

1. Para criar o kernel, adicione o seguinte código ao seu arquivo "Program.cs":
    
    ```c#
    using Microsoft.SemanticKernel;

    var builder = Kernel.CreateBuilder();
    builder.AddAzureOpenAIChatCompletion(
        "your-deployment-name",
        "your-endpoint",
        "your-api-key",
        "deployment-model");
    var kernel = builder.Build();
    ```

    Substitua os espaços reservados pelos valores do recurso do Azure.

1. Para verificar se o kernel e o ponto de extremidade estão funcionando, insira o seguinte código:

    ```c#
    var result = await kernel.InvokePromptAsync(
        "Who are the top 5 most famous musicians in the world?");
    Console.WriteLine(result);
    ```

1. Execute o código e verifique se você vê uma resposta do modelo do OpenAI do Azure contendo os 5 músicos mais famosos do mundo.

    A resposta vem do modelo de IA aberta do Azure que você passou para o kernel. O SDK do Kernel Semântico consegue se conectar ao LLM (modelo de linguagem grande) e executar a solicitação. Observe a rapidez com que você foi capaz de receber respostas do LLM. O SDK do Kernel Semântico torna a criação de aplicativos inteligentes fácil e eficiente.

## Exercício 2: Criar plug-ins personalizados para biblioteca de música

Neste exercício, você criará plug-ins personalizados para sua biblioteca de música. Você cria funções que podem adicionar músicas à lista de músicas reproduzidas recentemente pelo usuário, obter a lista de músicas reproduzidas recentemente e fornecer recomendações personalizadas de músicas. Você também criará uma função que sugere um show com base na localização do usuário e nas músicas que ele ouviu recentemente.

**Tempo estimado de conclusão do exercício**: 15 minutos

### Tarefa 1: Criar um plug-in de biblioteca de música

Nesta tarefa, você criará um plug-in que permite adicionar músicas à lista de músicas tocadas recentemente pelo usuário e obter a lista das músicas tocadas recentemente. Para simplificar, as músicas reproduzidas recentemente serão armazenadas em um arquivo de texto.

1. Crie uma pasta no diretório "Lab01-Project" e nomeie-a "Plug-ins."

1. Na pasta "Plugins", crie um novo arquivo "MusicLibrary.cs"

    Primeiro, crie algumas funções rápidas para obter e adicionar músicas à lista "Tocadas Recentemente" do usuário.

1. Insira o seguinte código:

    ```c#
    using System.ComponentModel;
    using System.Text.Json;
    using System.Text.Json.Nodes;
    using Microsoft.SemanticKernel;

    public class MusicLibraryPlugin
    {
        [KernelFunction, 
        Description("Get a list of music recently played by the user")]
        public static string GetRecentPlays()
        {
            string content = File.ReadAllText($"Files/RecentlyPlayed.txt");
            return content;
        }
    }
    ```

    Nesse código, você vai usar o decorador `KernelFunction` para declarar sua função nativa. Você também usará o decorador `Description` para adicionar uma descrição do que a função faz. A lista de músicas reproduzidas recentemente pelo usuário será armazenada em um arquivo de texto chamado "RecentlyPlayed.txt". Em seguida, você pode adicionar código para adicionar uma música à lista.

1. Adicione o seguinte código à classe `MusicLibraryPlugin`:

    ```c#
    [KernelFunction, Description("Add a song to the recently played list")]
    public static string AddToRecentlyPlayed(
        [Description("The name of the artist")] string artist, 
        [Description("The title of the song")] string song, 
        [Description("The song genre")] string genre)
    {
        // Read the existing content from the file
        string filePath = "Files/RecentlyPlayed.txt";
        string jsonContent = File.ReadAllText(filePath);
        var recentlyPlayed = (JsonArray) JsonNode.Parse(jsonContent);

        var newSong = new JsonObject
        {
            ["title"] = song,
            ["artist"] = artist,
            ["genre"] = genre
        };

        recentlyPlayed.Insert(0, newSong);
        File.WriteAllText(filePath, JsonSerializer.Serialize(recentlyPlayed,
            new JsonSerializerOptions { WriteIndented = true }));

        return $"Added '{song}' to recently played";
    }
    ```

    Nesse código, você vai criar uma função que aceita o artista, a música e o gênero como cadeias de caracteres. Além da `Description` da função, você também vai adicionar descrições das variáveis de entrada de dados. O arquivo "RecentlyPlayed.txt" contém uma lista formatada em json das músicas que o usuário tocou recentemente. Esse código lê o conteúdo existente no arquivo, o analisa e adiciona a nova música à lista. Depois disso, a lista atualizada é gravada novamente no arquivo.

1. Atualize seu arquivo "Program.cs" com o seguinte código:

    ```c#
    var kernel = builder.Build();
    kernel.ImportPluginFromType<MusicLibraryPlugin>();

    var result = await kernel.InvokeAsync(
        "MusicLibraryPlugin", 
        "AddToRecentlyPlayed", 
        new() {
            ["artist"] = "Tiara", 
            ["song"] = "Danse", 
            ["genre"] = "French pop, electropop, pop"
        }
    );
    
    Console.WriteLine(result);
    ```

    Nesse código, você vai importar o `MusicLibraryPlugin` para o kernel usando `ImportPluginFromType`. Em seguida, você vai chamar `InvokeAsync` com o nome do plugin e o nome da função que quer chamar. Você também vai repassar o artista, a música e o gênero como argumentos.

1. Execute o código inserindo `dotnet run` no terminal.

    A seguinte saída deve ser exibida:

    ```output
    Added 'Danse' to recently played
    ```

    Se você abrir o arquivo "RecentlyPlayed.txt", verá a nova música adicionada à lista.

### Tarefa 2: Fornecer recomendações personalizadas de músicas

Nesta tarefa, você criará um prompt que fornece recomendações de músicas personalizadas para o usuário com base nas músicas reproduzidas recentemente. O prompt combina funções nativas para gerar uma recomendação de música. Você também criará uma função a partir do prompt para torná-lo reutilizável.

1. No seu arquivo do `MusicLibraryPlugin.cs`, adicione a seguinte função:

    ```c#
    [KernelFunction, Description("Get a list of music available to the user")]
    public static string GetMusicLibrary()
    {
        string dir = Directory.GetCurrentDirectory();
        string content = File.ReadAllText($"Files/MusicLibrary.txt");
        return content;
    }
    ```

    Essa função fará a leitura da lista de músicas disponíveis em um arquivo chamado "MusicLibrary.txt". O arquivo contém uma lista formatada em json das músicas disponíveis para o usuário.

1. Atualize seu arquivo "Program.cs" com o código a seguir:

    ```c#
    var kernel = builder.Build();
    kernel.ImportPluginFromType<MusicLibraryPlugin>();
    
    string prompt = @"This is a list of music available to the user:
        {{MusicLibraryPlugin.GetMusicLibrary}} 

        This is a list of music the user has recently played:
        {{MusicLibraryPlugin.GetRecentPlays}}

        Based on their recently played music, suggest a song from
        the list to play next";

    var result = await kernel.InvokePromptAsync(prompt);
    Console.WriteLine(result);
    ```

Nesse código, você vai combinar suas funções nativas com um prompt semântico. As funções nativas são capazes de recuperar dados do usuário que o modelo de linguagem grande (LLM) não conseguiu acessar por conta própria, e o LLM é capaz de gerar uma recomendação de música com base na entrada de texto.

1. Para testar seu código, insira `dotnet run` no terminal.

    Você deverá ver uma resposta semelhante ao seguinte resultado:

    ```output 
    Based on the user's recently played music, a suggested song to play next could be "Sabry Aalil" by Yasemin since the user seems to enjoy pop and Egyptian pop music.
    ```

    >[!NOTE] A música recomendada pode ser diferente da que mostramos aqui.

1. Modifique o código para criar uma função a partir do prompt:

    ```c#
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

    var result = await kernel.InvokeAsync(songSuggesterFunction);
    Console.WriteLine(result);
    ```

    Nesse código, você cria uma função a partir de um prompt que sugere uma música para o usuário. Em seguida, você a adiciona aos plugins do kernel. Por fim, você informará ao kernel para executar a função.

### Tarefa 3: Fornecer recomendações personalizadas de shows

Nesta tarefa, você criará um plug-in que recupera detalhes dos próximos shows. Você também criará um plug-in que solicita ao LLM que sugira um show com base nas músicas que o usuário ouviu recentemente e na sua localização.

1. Na pasta "Plugins", crie um novo arquivo chamado "MusicConcertPlugin.cs"

1. No arquivo MusicConcertPlugin", adicione o seguinte código:

    ```c#
    using System.ComponentModel;
    using Microsoft.SemanticKernel;

    public class MusicConcertPlugin
    {
        [KernelFunction, Description("Get a list of upcoming concerts")]
        public static string GetTours()
        {
            string content = File.ReadAllText($"Files/ConcertDates.txt");
            return content;
        }
    }
    ```

    Nesse código, você criará uma função que lê a lista dos próximos shows de um arquivo chamado "ConcertDates.txt". O arquivo contém uma lista formatada em json dos próximos shows. Em seguida, você precisa criar um prompt para pedir ao LLM que sugira um show.

1. Na pasta "Prompts", crie uma nova pasta chamada "SuggestConcert"

1. Crie um arquivo "config.json" na pasta "SuggestConcert" com o seguinte conteúdo:

    ```json
    {
        "schema": 1,
        "type": "completion",
        "description": "Suggest a concert to the user",
        "execution_settings": {
            "default": {
                "max_tokens": 4000,
                "temperature": 0
            }
        },
        "input_variables": [
            {
                "name": "location",
                "description": "The user's location",
                "required": true
            }
        ]
    }
    ```

1. Crie um arquivo "skprompt.txt" na pasta "SuggestConcert" com o seguinte conteúdo:

    ```output
    This is a list of the user's recently played songs:
    {{MusicLibraryPlugin.GetRecentPlays}}

    This is a list of upcoming concert details:
    {{MusicConcertsPlugin.GetConcerts}}

    Suggest an upcoming concert based on the user's recently played songs. 
    The user lives in {{$location}}, 
    please recommend a relevant concert that is close to their location.
    ```

    Este prompt ajuda o LLM a filtrar a entrada do usuário e a recuperar apenas o destino do texto. Em seguida, você invoca o planejador para criar um plano que combine os plug-ins juntos para atingir a meta.

1. Abra o arquivo "Program.cs" e atualize-o com o seguinte código:

    ```c#
    var kernel = builder.Build();    
    kernel.ImportPluginFromType<MusicLibraryPlugin>();
    kernel.ImportPluginFromType<MusicConcertsPlugin>();
    var prompts = kernel.ImportPluginFromPromptDirectory("Prompts");

    var songSuggesterFunction = kernel.CreateFunctionFromPrompt(
    // code omitted for brevity
    );

    kernel.Plugins.AddFromFunctions("SuggestSongPlugin", [songSuggesterFunction]);

    string location = "Redmond WA USA";
    var result = await kernel.InvokeAsync<string>(prompts["SuggestConcert"],
        new() {
            { "location", location }
        }
    );
    Console.WriteLine(result);
    ```

1. Insira `dotnet run` no terminal

    Você deverá ver uma saída semelhante à seguinte resposta:

    ```output
    Based on the user's recently played songs and their location in Redmond WA USA, a relevant concert recommendation would be the upcoming concert of Lisa Taylor in Seattle WA, USA on February 22, 2024. Lisa Taylor is an indie-folk artist, and her music genre aligns with the user's recently played songs, such as "Loanh Quanh" by Ly Hoa. Additionally, Seattle is close to Redmond, making it a convenient location for the user to attend the concert.
    ```

    Tente ajustar o prompt e o local para ver quais outros resultados você pode gerar.

## Exercício 3: Automatizar sugestões com o planejador do Handlebars

O planejador do Handlebars é útil quando são necessárias várias etapas para realizar uma tarefa. Os planejadores usam IA para selecionar e combinar os plug-ins registrados no kernel em uma série de etapas para atingir uma meta. Neste exercício, você gerará um modelo de plano usando o planejador do Handlebars e o utilizará para automatizar sugestões.

**Tempo estimado de conclusão do exercício**: 10 minutos

### Tarefa 1: Gerar um modelo de plano

Nesta tarefa, você gerará um modelo de plano usando o planejador do Handlebars. O modelo de plano será usado para automatizar sugestões com base na entrada do usuário.

1. Instale o planejador de barras de identificador inserindo o seguinte no terminal:

    `dotnet add package Microsoft.SemanticKernel.Planners.Handlebars --version 1.2.0-preview`

    Em seguida, você substituirá o prompt SuggestConcert e usará o planejador do Handlebars para executar a tarefa.

1. Atualize o arquivo 'Program.cs' com o seguinte código:

    ```c#
    var kernel = builder.Build();
    kernel.ImportPluginFromType<MusicLibraryPlugin>();
    kernel.ImportPluginFromType<MusicConcertPlugin>();
    kernel.ImportPluginFromPromptDirectory("Prompts");

    var planner = new HandlebarsPlanner(new HandlebarsPlannerOptions() { AllowLoops = true });

    string location = "Redmond WA USA";
    string goal = @$"Based on the user's recently played music, suggest a 
        concert for the user living in ${location}";

    var plan = await planner.CreatePlanAsync(kernel, goal);
    var result = await plan.InvokeAsync(kernel);

    Console.WriteLine($"{result}");
    ```

1. Insira `dotnet run` no terminal

    Você deverá ver uma resposta semelhante à seguinte saída:

    ```output
    Based on the user's recently played songs, the artist "Mademoiselle" has an upcoming concert in Seattle WA, USA on February 22, 2024, which is close to Redmond WA. Therefore, the recommended concert for the user would be Mademoiselle's concert in Seattle.
    ```

    Em seguida, você alterará o código para gerar o modelo de plano do Handlebars.

1. Atualize o arquivo 'Program.cs' com o seguinte código:

    ```c#
    var plan = await planner.CreatePlanAsync(kernel, goal);
    Console.WriteLine("Plan:");
    Console.WriteLine(plan);
    ```

    Agora você pode ver o plano gerado. Em seguida, modifique o plano para incluir sugestões de músicas ou adicionar uma música à lista de músicas tocadas recentemente pelo usuário.

1. Amplie seu código com o seguinte trecho:

    ```c#
    var plan = await planner.CreatePlanAsync(kernel, 
        @$"If add song:
        Add a song to the user's recently played list.
        
        If concert recommendation:
        Based on the user's recently played music, suggest a concert for 
        the user living in a given location.

        If song recommendation:
        Suggest a song from the music library to the user based on their 
        recently played songs.");

    Console.WriteLine("Plan:");
    Console.WriteLine(plan);
    ```

1. Digite `dotnet run` no terminal para ver a saída dos planos que você criou.

    Você deverá ver um modelo semelhante à seguinte saída:

    ```output
    Plan:
    {{!-- Step 1: Identify Key Values --}}
    {{set "location" location}}
    {{set "addSong" addSong}}
    {{set "concertRecommendation" concertRecommendation}}
    {{set "songRecommendation" concertRecommendation}}

    {{!-- Step 2: Use the Right Helpers --}}
    {{#if addSong}}
        {{set "song" song}}
        {{set "artist" artist}}
        {{set "genre" genre}}
        {{set "songAdded" (MusicLibraryPlugin-AddToRecentlyPlayed artist=artist song=song genre=genre)}}  
        {{json songAdded}}
    {{/if}}

    {{#if concertRecommendation}}
        {{set "concertSuggested" (Prompts-SuggestConcert location=location recentlyPlayedSongs=recentlyPlayedSongs musicLibrary=musicLibrary)}}
        {{json concertSuggested}}
    {{/if}}

    {{#if songRecommendation}}
        {{set "songSuggested" (SuggestSongPlugin-SuggestSong recentlyPlayedSongs=recentlyPlayedSongs musicLibrary=musicLibrary)}}
        {{json songSuggested}}
    {{/if}}

    {{!-- Step 3: Output the Result --}}
    {{json "Goal achieved"}}
    ```

     Observe a sintaxe `{{#if ...}}`. Essa sintaxe atua como uma instrução condicional que o planejador de barras de identificador pode usar, semelhante a um bloco `if`-`else` tradicional em C#. A instrução `if` precisa ser fechada com `{{/if}}`.

    Em seguida, você usará esse modelo gerado para criar seu próprio plano do Handlebars. 

1. Crie um arquivo chamado 'handlebarsTemplate.txt' com o seguinte texto:

    ```output
    {{set "addSong" addSong}}
    {{set "concertRecommendation" concertRecommendation}}
    {{set "songRecommendation" songRecommendation}}

    {{#if addSong}}
        {{set "song" song}}
        {{set "artist" artist}}
        {{set "genre" genre}}
        {{set addedSong (MusicLibraryPlugin-AddToRecentlyPlayed artist song genre)}}  
        Output The following content, do not make any modifications:
        {{json addedSong}}     
    {{/if}}

    {{#if concertRecommendation}}
        {{set "location" location}}
        {{set "concert" (Prompts-SuggestConcert location)}}
        Output The following content, do not make any modifications:
        {{json concert}}
    {{/if}}

    {{#if songRecommendation}}
        {{set "recentlyPlayedSongs" (MusicLibraryPlugin-GetRecentPlays)}}
        {{set "musicLibrary" (MusicLibraryPlugin-GetMusicLibrary)}}
        {{set "song" (SuggestSongPlugin-SuggestSong recentlyPlayedSongs musicLibrary)}}
        Output The following content, do not make any modifications:
        {{json song}}
    {{/if}}
    ```

    Nesse modelo, você adicionará uma instrução ao LLM para não executar nenhuma geração de texto e garantir que a saída seja estritamente gerenciada pelos plug-ins. Agora vamos testar o modelo!

### Tarefa 2: Usar o planejador do Handlebars para automatizar sugestões

Nesta tarefa, você criará uma função a partir do modelo de plano do Handlebars e a usará para automatizar sugestões com base na entrada do usuário.

1. Remova os planos de barras de identificador modificando o código existente:

    ```c#
    var builder = Kernel.CreateBuilder();
    builder.AddAzureOpenAIChatCompletion(
        "your-deployment-name",
        "your-endpoint",
        "your-api-key",
        "deployment-model");
    var kernel = builder.Build();
    kernel.ImportPluginFromType<MusicLibraryPlugin>();
    kernel.ImportPluginFromType<MusicConcertsPlugin>();
    kernel.ImportPluginFromPromptDirectory("Prompts");
    
    var songSuggesterFunction = kernel.CreateFunctionFromPrompt(
        promptTemplate: @"Based on the user's recently played music:
        {{$recentlyPlayedSongs}}
        recommend a song to the user from the music library:
        {{$musicLibrary}}",
        functionName: "SuggestSong",
        description: "Suggest a song to the user"
    );

    kernel.Plugins.AddFromFunctions("SuggestSongPlugin", [songSuggesterFunction]);
    ```

1. Adicione um código que faça a leitura do arquivo de modelo e crie uma função:

    ```c#
    string template = File.ReadAllText($"handlebarsTemplate.txt");

    var handlebarsPromptFunction = kernel.CreateFunctionFromPrompt(
        new() {
            Template = template,
            TemplateFormat = "handlebars"
        }, new HandlebarsPromptTemplateFactory()
    );
    ```

    Neste código, você passa um objeto `Template` para o método `CreateFunctionFromPrompt` do kernel junto com o `TemplateFormat`. `CreateFunctionFromPrompt` também aceita um tipo `IPromptTemplateFactory` que diz ao kernel como analisar um determinado modelo. Como você está usando um modelo de barras de identificador, use o tipo `HandlebarsPromptTemplateFactory`.

    A seguir, vamos executar a função com alguns argumentos e conferir os resultados.

1. Adicione o seguinte código ao seu arquivo `Program.cs`:

    ```c#
    string location = "Redmond WA USA";
    var templateResult = await kernel.InvokeAsync(handlebarsPromptFunction,
        new() {
            { "location", location },
            { "concertRecommendation", true },
            { "songRecommendation", false },
            { "addSong", false },
            { "artist", "" },
            { "song", "" },
            { "genre", "" }
        });

    Console.WriteLine(templateResult);
    ```

1. Insira `dotnet run` no terminal para ver a saída do seu modelo de planejador.

    Você deverá ver uma resposta semelhante à seguinte saída:

    ```output
    Based on the user's recently played songs, Ly Hoa seems to be a relevant artist. The closest concert to Redmond WA, USA would be in Portland OR, USA on April 16th, 2024.  
    ```

    O prompt conseguiu sugerir um show ao usuário com base na lista de músicas reproduzidas recentemente e na localização do usuário. Você também pode tentar definir outras variáveis como verdadeiras e ver o que acontece!

Agora seu código será capaz de executar ações diferentes com base na entrada do usuário. Ótimo trabalho!

### Revisão

Neste laboratório, você criou um agente de IA que pode gerenciar a biblioteca de músicas do usuário e oferecer recomendações personalizadas de músicas e shows. Você usou o SDK do Semantic Kernel para criar o agente de IA e conectá-lo ao serviço de modelo de linguagem grande (LLM). Você criou plug-ins personalizados para sua biblioteca de músicas, usou o planejador do Handlebars para automatizar as sugestões e criou uma função do modelo de plano do Handlebars para automatizar as sugestões com base na entrada do usuário. Parabéns por ter concluído este laboratório!