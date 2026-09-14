---
lab:
    title: 'Integrar um agente de IA com o Foundry IQ'
    description: 'Use o Azure AI Agent Service para desenvolver um agente que usa o Foundry IQ para pesquisar bases de conhecimento.'
    level: 300
    duration: 45
    islab: true
    status: 'released'
    layout: default
---

# Integrar um agente de IA com o Foundry IQ

Neste exercício, você usará o portal do Microsoft Foundry para criar um agente que se integra ao Foundry IQ a fim de pesquisar e recuperar informações de bases de conhecimento. Você criará um recurso de pesquisa, configurará uma base de conhecimento com dados de exemplo, criará um agente no portal e, em seguida, conectará a ele usando o Visual Studio Code para interagir programaticamente.

> **Dica**: O código usado neste exercício é baseado no SDK do Microsoft Foundry para Python. Você pode desenvolver soluções semelhantes usando os SDKs do Microsoft .NET, JavaScript e Java. Consulte [Bibliotecas de cliente do SDK do Microsoft Foundry](https://learn.microsoft.com/azure/ai-foundry/how-to/develop/sdk-overview) para obter detalhes.

Este exercício deve levar aproximadamente **45** minutos para ser concluído.

> **Observação**: Algumas das tecnologias usadas neste exercício estão em versão preliminar ou em desenvolvimento ativo. Você pode encontrar algum comportamento inesperado, avisos ou erros.

## Pré-requisitos

Antes de iniciar este exercício, verifique se você tem:

- Uma [assinatura do Azure](https://azure.microsoft.com/free/) com permissões para criar recursos de IA
- O [Visual Studio Code](https://code.visualstudio.com/) instalado no computador local
- O [Python 3.13](https://www.python.org/downloads/) instalado
- O [Git](https://git-scm.com/downloads) instalado no computador local
- Familiaridade básica com o portal do Microsoft Foundry e programação em Python

> \* O Python 3.14 ainda não é compatível: algumas dependências não têm uma compilação para a versão 3.14. Este laboratório foi testado com o Python 3.13.12.

## Criar um projeto do Foundry

Vamos começar criando um projeto do Foundry com a nova experiência do Foundry.

1. Em um navegador da Web, abra o [portal do Foundry](https://ai.azure.com) em `https://ai.azure.com` e entre usando suas credenciais do Azure. Feche todas as dicas ou painéis de início rápido que forem abertos na primeira vez que você entrar.

    > **Importante**: Verifique se a opção **New Foundry** está *Ativada* para que este laboratório use a interface do usuário atualizada.

1. Depois de alternar para o **New Foundry**, será solicitado que você selecione um projeto. Na lista suspensa, selecione **Create a new project**.
1. Na caixa de diálogo **Create a project**, insira um nome válido para o projeto (por exemplo, *agent-iq-lab*).
1. Confirme ou defina as seguintes configurações para o projeto:
    - **Foundry resource**: *Create a new Foundry resource or select an existing one*
    - **Subscription**: *Sua assinatura do Azure*
    - **Resource group**: *Crie ou selecione um grupo de recursos*
    - **Location**: *Selecione qualquer região disponível*\*

    > \* Alguns recursos de IA do Azure estão sujeitos a cotas de modelo limitadas por região. Se um limite de cota for excedido posteriormente no exercício, talvez seja necessário criar outro recurso em uma região diferente.

1. Selecione **Create** e aguarde a criação do projeto. Isso pode levar alguns minutos.
1. Quando o projeto for criado, você verá a página inicial do projeto.

## Criar um agente

1. Na página inicial, selecione a guia **Build** e, em seguida, na guia **Agents**, selecione **Create agent**.
1. Crie seu agente com um nome descritivo, como `product-expert-agent`.

Ao criar um agente, o modelo padrão (como `gpt-5`) será implantado. Depois que o agente for criado, você verá o playground do agente com esse modelo padrão selecionado automaticamente.

## Configurar seus dados e o Foundry IQ

Agora você configurará seu agente para usar o Foundry IQ e pesquisar a base de conhecimento.

1. Primeiro, dê ao agente as seguintes instruções:

    ```
   Você é um assistente de IA prestativo da Contoso, especializado em produtos para acampamento e trilhas ao ar livre.
   Você DEVE SEMPRE pesquisar a base de conhecimento para responder a perguntas sobre nossos produtos ou nosso
   catálogo de produtos. Forneça informações detalhadas e precisas e sempre cite suas fontes.
   Se não encontrar informações relevantes na base de conhecimento, diga isso claramente.
    ```

1. Selecione **Save** para salvar a configuração atual do agente.
1. Em seguida, na seção **Knowledge**, expanda a lista suspensa **Add** e selecione **Connect to Foundry IQ**.
1. Na janela de configuração do Foundry IQ, selecione **Connect to an AI Search resource** e, em seguida, **Create new resource**, o que deve abrir uma caixa de diálogo para criar o recurso.
1. Crie um recurso de pesquisa com as configurações padrão:
    - **Resource name**: *Um nome globalmente exclusivo*
    - **Subscription**: *Sua assinatura do Azure*
    - **Resource group**: *Use o mesmo grupo de recursos do projeto*
    - **Region**: *O mesmo local do projeto*
    - **Pricing tier**: Free *se disponível; caso contrário, escolha Basic*
    - **Foundry IQ Knowledge base capabilities**: Pause til next month

    > **Observação**: Se você tiver problemas para criar o recurso aqui, selecione o link na parte inferior do formulário para criá-lo no portal do Azure.

Agora você carregará documentos de informações de produtos de exemplo para conectar ao Foundry IQ.

1. Baixe os arquivos de informações de produtos de exemplo abrindo uma nova guia do navegador e navegando até `https://github.com/MicrosoftLearning/mslearn-ai-agents/raw/main/Labfiles/04-integrate-agent-with-foundry-iq/data/contoso-products.zip`
1. Extraia os arquivos do zip, que devem ser 3 PDFs detalhando os produtos da Contoso.
1. Abra uma nova guia e navegue até o portal do Azure em `https://portal.azure.com`. Na barra de pesquisa superior, pesquise **Storage accounts** e selecione **Storage accounts** na seção de serviços.
1. Crie uma conta de armazenamento com as seguintes configurações:
    - **Subscription**: *Sua assinatura do Azure*
    - **Resource group**: *Use o mesmo grupo de recursos do projeto*
    - **Storage account name**: *Um nome exclusivo para a conta de armazenamento*
    - **Region**: *O mesmo local do projeto*
    - **Primary service**: *Azure Blob Storage ou Azure Data Lake Storage*
    - **Performance**: *Standard*
    - **Redundancy**: *Locally-redundant storage (LRS)*
1. Depois de criada, acesse a conta de armazenamento criada e selecione **Upload** na barra superior.
1. No painel **Upload blob**, crie um novo contêiner chamado `contosoproducts`.
1. Procure os arquivos extraídos do arquivo zip, selecione os 3 arquivos PDF e selecione **Upload**.
1. Depois que os arquivos forem carregados, navegue até o serviço de pesquisa criado.
1. No painel esquerdo, em **Security + networking** > **Keys**, selecione **Both** para o controle de acesso à API e confirme a seleção. Quando terminar, deixe a guia do Portal do Azure aberta, volte para a guia do portal do Foundry e atualize a página.
1. Verifique se você está na página **Knowledge**, selecione **Create a knowledge base**, escolhendo **Azure Blob Storage** como sua fonte de conhecimento, e selecione **Connect**.
1. Configure sua fonte de conhecimento com as seguintes configurações:
    - **Name**: `ks-contosoproducts`
    - **Description**: `Contoso product catalog items`
    - **Storage account name**: *Selecione sua conta de armazenamento*
    - **Container name**: `contosoproducts`
    - **Authentication type**: *API Key*
    - **Content extraction mode**: *minimal*
    - **Embedding model**: *Selecione o modelo implantado disponível, provavelmente text-embedding-3-small*
    - **Chat completions model**: *Selecione o modelo implantado disponível, provavelmente gpt-5*
1. Selecione **Create**.
1. Na página de criação da base de conhecimento, selecione o modelo `gpt-5` na lista suspensa **Chat completions model**, deixando os demais padrões dos campos como estão.
1. Selecione **Save knowledge base** e atualize o navegador para verificar se o status da fonte de conhecimento está *active*. Se ainda não estiver, aguarde um minuto e atualize a página até que esteja.
1. Selecione o botão Voltar para retornar à página **Knowledge** e, em seguida, selecione o link **Manage** ao lado da lista suspensa *Connection*.
1. Role para baixo até **Connected resources**, onde você deverá ver o serviço de pesquisa. Selecione essa linha e localize a seção **Authentication**.
1. Selecione **Key authentication** e, em seguida, selecione **Edit authentication**.
1. Deixando a caixa de diálogo aberta, volte à guia do portal do Azure, que ainda deve estar na página **Keys** do serviço de pesquisa. Copie uma dessas chaves para a caixa de diálogo do Foundry e selecione **Save**.

As configurações do Foundry IQ agora devem estar concluídas.

## Testar o agente no playground

Antes de conectar pelo código, teste o agente no playground do portal.

1. Volte ao seu agente na página **Build** > **Agents** e selecione o agente criado.
2. Na página do agente, você deverá ver uma guia do playground selecionada. Localize a seção de conhecimento e adicione o Foundry IQ, selecionando a conexão e a base de conhecimento que você criou.
1. Experimente as seguintes consultas de teste para verificar se o agente consegue recuperar informações da base de conhecimento:
    - `Que tipos de barracas a Contoso oferece?`
    - `Conte-me quais mochilas estão disponíveis no tamanho XL.`
    - `Quais acessórios para acampamento estão disponíveis?`

1. Examine as respostas e observe:
    - O agente fornece informações específicas da base de conhecimento
    - Citações ou referências aos documentos de origem podem ser incluídas
    - O agente permanece concentrado nas informações dos produtos

1. Você também pode tentar interagir com seu agente no **Preview agent** para obter uma experiência de aplicativo Web mais refinada.

1. Na página de detalhes do agente, localize e copie as seguintes informações para um bloco de notas (você precisará delas mais tarde):
    - **Agent name**: Este é o nome que você criou (`product-expert-agent`)
    - **Project endpoint**: Encontrado nas configurações do projeto ou na página inicial

### Configurar o agente para exigir aprovação para chamadas de ferramentas

Quando você cria um agente no portal, a ferramenta Foundry IQ (conhecimento) é executada **sem** solicitar aprovação por padrão. Para garantir que seu aplicativo possa revisar e controlar cada consulta à base de conhecimento, você alterará o agente para exigir aprovação antes de usar ferramentas com a extensão Foundry Toolkit for VS Code.

> **Observação**: No momento, o portal do Foundry não expõe uma configuração para alterar esse comportamento de aprovação, portanto você o configurará na extensão Foundry Toolkit.

1. No Visual Studio Code, selecione **Extensions** no painel esquerdo (ou pressione **Ctrl+Shift+X**), pesquise no marketplace a extensão `Foundry Toolkit for VS Code` da Microsoft e selecione **Install** (se ela ainda não estiver instalada).

    > **Observação**: A extensão está listada atualmente como **Foundry Toolkit**, mas alguns rótulos, comandos ou capturas de tela antigas do VS Code ainda podem fazer referência ao **AI Toolkit**. Neste laboratório, considere esses nomes como referentes à mesma experiência da extensão.

1. Selecione o ícone **Foundry Toolkit** na barra lateral e entre na sua conta do Azure, se solicitado.
   
    > **Observação**: Se não conseguir entrar com a extensão Foundry Toolkit, talvez seja necessário selecionar a extensão do Azure. Entre nela e, em seguida, volte ao Foundry Toolkit para acessar seus recursos.

1. Em **Microsoft Foundry Resources**, escolha **Set Default Project** e selecione o projeto criado anteriormente.
1. Expanda a seção do projeto. Em **Prompt Agents**, selecione seu agente `product-expert-agent` para abrir a janela **Agent Builder**.
1. Na seção **Tools**, você já deverá ver uma ferramenta com um nome que começa com o prefixo `kb-knowledgebase`, seguido por um ID exclusivo (por exemplo, `kb-knowledgebase677-7w5fj`). Essa é a ferramenta de base de conhecimento do Foundry IQ, adicionada automaticamente quando você conectou o Foundry IQ no portal.

    > **Observação**: O agente lista mais de uma ferramenta. O portal do Foundry adiciona uma ferramenta **Web search** aos novos agentes por padrão, e você também poderá ver uma ferramenta autônoma **Azure AI Search**. O agente, na verdade, chama a ferramenta `kb-knowledgebase...` quando pesquisa sua base de conhecimento; portanto, definir a aprovação em qualquer outra ferramenta não terá efeito.

1. Selecione o ícone de reticências (**...**) na ferramenta `kb-knowledgebase...`, selecione **Ask for approval for all tools** e salve as alterações, se for solicitado.

Agora, seu agente solicitará aprovação sempre que usar o Foundry IQ para pesquisar a base de conhecimento; o aplicativo cliente que você concluirá a seguir cuidará disso.

## Conectar-se ao agente por meio de um aplicativo

Agora você criará um aplicativo Python para interagir programaticamente com seu agente. Arquivos iniciais foram fornecidos no repositório do GitHub para ajudar você a começar rapidamente.

### Preparar-se para desenvolver um aplicativo no Visual Studio Code

Agora vamos usar o Visual Studio Code para desenvolver um aplicativo. Os arquivos de código do aplicativo foram fornecidos em um repositório do GitHub.

1. Inicie o Visual Studio Code e abra a paleta de comandos (Shift+Ctrl+P). Em seguida, pesquise e execute o comando **Git: Clone** para clonar o repositório `https://github.com/MicrosoftLearning/mslearn-ai-agents` em uma pasta local (não importa qual pasta).
1. Quando o repositório tiver sido clonado, abra a pasta no Visual Studio Code.

    > **Observação**: Se o Visual Studio Code mostrar uma mensagem pop-up solicitando que você confie no código que está abrindo, clique na opção **Yes, I trust the authors** para continuar.

1. Aguarde enquanto arquivos adicionais são instalados para dar suporte aos projetos de código Python no repositório (se solicitado).

    > **Observação**: Se for solicitado que você instale os ativos necessários para compilar e depurar, selecione **Not Now**.

1. No painel **Explorer**, expanda a pasta **Labfiles/04-integrate-agent-with-foundry-iq/Python**.

    Os arquivos fornecidos incluem o código do aplicativo, as configurações e o código inicial do cliente do agente.

### Configurar as definições do aplicativo

1. No Visual Studio Code, na pasta **Labfiles/04-integrate-agent-with-foundry-iq/Python**, abra o arquivo de configuração **.env**.
1. No arquivo de código, substitua o espaço reservado **your_project_endpoint** pelo endpoint do projeto (copiado da página **Home** do projeto no portal do Foundry) e verifique se a variável AGENT_NAME está definida como o nome do agente (que deve ser *product-expert-agent*).
1. Depois de substituir o espaço reservado, salve o arquivo.

### Concluir o código do cliente do agente

> **Dica**: Ao adicionar o código, mantenha o recuo correto. Use os níveis de recuo dos comentários como guia.

1. No Visual Studio Code, na pasta **Labfiles/04-integrate-agent-with-foundry-iq/Python**, abra o arquivo de código **agent_client.py**.
1. Examine o código inicial fornecido, incluindo:
    - Instruções de importação e carregamento da configuração
    - A estrutura da função `send_message_to_agent()`
    - A função `display_conversation_history()`
    - O loop principal do programa

1. Localize o primeiro comentário **TODO** e adicione o código a seguir para conectar-se ao projeto, obter o cliente OpenAI, recuperar o agente e criar uma nova conversa:

    > **Dica**: Tenha cuidado para manter o nível correto de recuo.

    ```python
   # Conectar-se ao projeto e ao agente
   credential = DefaultAzureCredential(
       exclude_environment_credential=True,
       exclude_managed_identity_credential=True
   )
   project_client = AIProjectClient(
       credential=credential,
       endpoint=project_endpoint
   )

   # Obter o cliente OpenAI
   openai_client = project_client.get_openai_client()

   # Obter o agente
   agent = project_client.agents.get(agent_name=agent_name)
   print(f"Conectado ao agente: {agent.name} (id: {agent.id})\n")

   # Criar uma nova conversa
   conversation = openai_client.conversations.create(items=[])
   print(f"Conversa criada (id: {conversation.id})\n")
    ```

1. Localize o segundo comentário **TODO** dentro da função `send_message_to_agent()` e adicione o código a seguir para enviar mensagens e tratar respostas, incluindo solicitações de aprovação do MCP:

    ```python
   # Adicionar a mensagem do usuário à conversa
   openai_client.conversations.items.create(
       conversation_id=conversation.id,
       items=[{"type": "message", "role": "user", "content": user_message}],
   )

   # Armazenar no histórico da conversa (lado do cliente)
   conversation_history.append({
       "role": "user",
       "content": user_message
   })

   # Criar uma resposta usando o agente
   response = openai_client.responses.create(
       conversation=conversation.id,
       extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
       input=""
   )

   # Repetir até que uma resposta não tenha solicitações de aprovação pendentes (zero, uma ou várias)
   while True:
       approval_requests = [
           item for item in (getattr(response, "output", None) or [])
           if getattr(item, "type", None) == "mcp_approval_request"
       ]

       if not approval_requests:
           break

       approval_items = []
       for approval_request in approval_requests:
           print(f"[Aprovação necessária para: {approval_request.name}]\n")
           print(f"Servidor: {approval_request.server_label}")

           # Mostrar os argumentos da chamada da ferramenta para fins de transparência
           import json
           try:
               args = json.loads(approval_request.arguments)
               print(f"Argumentos: {json.dumps(args, indent=2)}\n")
           except Exception:
               print(f"Argumentos: {approval_request.arguments}\n")

           approval_input = input("Aprovar esta ação? (yes/no): ").strip().lower()
           approved = approval_input in ['yes', 'y']
           print("Aprovando ação...\n" if approved else "Ação negada.\n")

           approval_items.append({
               "type": "mcp_approval_response",
               "approval_request_id": approval_request.id,
               "approve": approved
           })

       # Enviar as decisões de aprovação e obter a próxima resposta
       openai_client.conversations.items.create(
           conversation_id=conversation.id,
           items=approval_items
       )

       response = openai_client.responses.create(
           conversation=conversation.id,
           extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
           input=""
       )

    ```

    > **Observação**: O agente nem sempre solicita aprovação e, ocasionalmente, pode solicitar aprovação para mais de uma chamada de ferramenta no mesmo turno. Repetir o loop até que `approval_requests` esteja vazio trata ambos os casos corretamente.

1. Depois de adicionar o código, salve o arquivo.

1. Examine o código, que agora usa a API de conversas para gerenciar as interações com o agente, da seguinte forma:
    - Uma conversa é criada e rastreada pelo ID
    - As mensagens do usuário são adicionadas à conversa usando `conversations.items.create()`
    - As respostas são geradas usando `responses.create()` com uma referência ao agente
    - **Tratamento de aprovação do MCP**: Quando o agente precisa acessar o Foundry IQ, ele solicita aprovação retornando um ou mais itens `mcp_approval_request` na saída da resposta
    - O código repete o processo, solicitando que você aprove ou negue cada solicitação pendente, até que o agente retorne uma resposta sem solicitações de aprovação pendentes (inclusive quando nenhuma aprovação foi necessária)
    - Após cada aprovação ou negação, um `mcp_approval_response` é adicionado à conversa e uma nova resposta é gerada
    - O agente recupera informações do Foundry IQ com base na sua decisão de aprovação

## Testar a integração

Agora você executará o aplicativo e testará a capacidade do agente de recuperar informações da base de conhecimento.

1. No Visual Studio Code, abra um terminal integrado para a pasta **Labfiles/04-integrate-agent-with-foundry-iq/Python** clicando com o botão direito do mouse na pasta e selecionando **Open in Integrated Terminal**.
1. Primeiro, crie um ambiente virtual e instale as dependências.

    ```
   python -m venv labenv
   ./labenv/Scripts/activate
   pip install -r requirements.txt
    ```

1. No painel do terminal, insira o comando a seguir para entrar no Azure.

    ```
   az login
    ```

    > **Observação**: Na maioria dos cenários, usar apenas *az login* será suficiente. No entanto, se você tiver assinaturas em vários locatários, talvez seja necessário especificar o locatário usando o parâmetro *--tenant*. Consulte [Entrar no Azure interativamente usando a CLI do Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively) para obter detalhes.

1. Quando solicitado, siga as instruções para abrir a página de entrada em uma nova guia e insira o código de autenticação fornecido e suas credenciais do Azure. Em seguida, conclua o processo de entrada na linha de comando, selecionando a assinatura que contém o recurso do Foundry, se solicitado.

1. No painel do terminal, execute o aplicativo:

    ```
   python agent_client.py
    ```

1. Quando o aplicativo for iniciado, teste o agente com as seguintes consultas:

    **Consulta 1 — Categorias de produtos:**

    ```
   Que tipos de produtos para atividades ao ar livre a Contoso oferece?
    ```

    Quando for solicitada aprovação, digite **yes** para permitir que o agente pesquise a base de conhecimento. Observe como o agente recupera informações de vários documentos da base de conhecimento.

    **Consulta 2 — Detalhes de produtos específicos:**

    ```
   Conte-me sobre os recursos de resistência às intempéries das suas barracas.
    ```

    Aprove a solicitação e observe como o agente fornece detalhes específicos do catálogo de barracas.

    **Consulta 3 — Comparações de produtos:**

    ```
   Qual é a diferença entre suas mochilas de uso diário e suas mochilas para expedição?
    ```

    Aprove a solicitação e veja como o agente consegue sintetizar informações do guia de mochilas.

    **Consulta 4 — Acessórios e complementos:**

    ```
   Quais acessórios para acampamento você recomendaria para uma viagem de fim de semana com trilhas?
    ```

    Aprove a solicitação e observe a capacidade do agente de fornecer recomendações com base na base de conhecimento.

    **Consulta 5 — Pergunta de acompanhamento:**

    ```
   Quanto esses itens costumam custar?
    ```

    Observe como o agente mantém o contexto da conversa a partir da consulta anterior.

1. Digite `history` para visualizar o histórico completo da conversa.

1. Digite `quit` quando terminar os testes.

### Examinar os resultados

Considere os seguintes aspectos das respostas do agente:

- **Fluxo de aprovação do MCP**: Cada vez que o agente precisa acessar a base de conhecimento, ele solicita aprovação, dando a você controle sobre o uso de ferramentas externas
- **Precisão**: O agente fornece informações diretamente dos documentos da base de conhecimento
- **Citações**: O agente pode incluir referências de origem ou IDs de documentos
- **Consciência do contexto**: O agente se lembra das mensagens anteriores da conversa
- **Fundamentação**: O agente indica quando não consegue encontrar informações relevantes na base de conhecimento
- **Tratamento de erros**: O aplicativo trata corretamente erros e problemas de conexão

## Resumo

Neste exercício, você:

- Criou um projeto e um agente do Foundry com a nova interface do usuário do Foundry
- Criou uma base de conhecimento com documentos de informações de produtos
- Configurou um agente no portal com o Foundry IQ habilitado
- Conectou-se ao agente pelo Visual Studio Code usando o SDK do Python
- Implementou um aplicativo cliente com tratamento de aprovação do MCP, histórico de conversas e tratamento de erros
- Testou a capacidade do agente de recuperar e sintetizar informações da base de conhecimento com aprovação controlada pelo usuário para acesso a ferramentas externas

Isso demonstra como integrar agentes de IA ao Foundry IQ para criar aplicativos inteligentes capazes de pesquisar e recuperar informações de bases de conhecimento empresariais, mantendo o contexto da conversa.

## Limpar

Se você terminou de explorar o Azure AI Agent Service e o Foundry IQ, exclua os recursos criados neste exercício para evitar custos desnecessários do Azure.

1. Em um navegador da Web, abra o [portal do Azure](https://portal.azure.com) em `https://portal.azure.com`.
1. Navegue até o grupo de recursos que contém os recursos do Foundry e do AI Search.
1. Na barra de ferramentas, selecione **Delete resource group**.
1. Insira o nome do grupo de recursos e confirme que deseja excluí-lo.
