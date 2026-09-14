---
lab:
    title: 'Estender agentes com ferramentas do Model Context Protocol (MCP)'
    description: 'Estenda os recursos dos agentes integrando ferramentas de servidor do Model Context Protocol (MCP).'
    level: 300
    duration: 60
    islab: true
    status: 'released'
    layout: default
---

# Estender agentes com ferramentas do Model Context Protocol (MCP)

Neste exercício, você usará a extensão Foundry Toolkit for VS Code para criar um agente que pode usar ferramentas de servidor do Model Context Protocol (MCP) para acessar fontes de dados e APIs externas. O agente poderá recuperar informações atualizadas e interagir com serviços personalizados por meio de ferramentas MCP.

Este exercício deve levar aproximadamente **60** minutos para ser concluído.

> **Observação**: Algumas das tecnologias usadas neste exercício estão em versão prévia ou em desenvolvimento ativo. Você pode encontrar comportamentos inesperados, avisos ou erros.

## Pré-requisitos

Antes de começar este exercício, verifique se você tem:

- O [Visual Studio Code](https://code.visualstudio.com/) instalado no computador local
- Uma [assinatura do Azure](https://azure.microsoft.com/free/) ativa
- O [Python 3.13](https://www.python.org/downloads/) instalado
- O [Git](https://git-scm.com/downloads) instalado no computador local

> \* O Python 3.14 ainda não é compatível: algumas dependências não têm um build para a versão 3.14. Este laboratório foi testado com o Python 3.12.

## Criar um projeto do Foundry com a extensão Foundry Toolkit for VS Code

Como desenvolvedor, você pode passar algum tempo trabalhando no portal do Foundry; mas também é provável que passe bastante tempo no Visual Studio Code. A extensão Foundry Toolkit for VS Code oferece uma maneira conveniente de trabalhar com os recursos de projetos do Foundry sem sair do ambiente de desenvolvimento.

1. Abra o Visual Studio Code.

2. Selecione **Extensions** no painel esquerdo (ou pressione **Ctrl+Shift+X**).

3. Pesquise no marketplace de extensões a extensão `Foundry Toolkit`, da Microsoft, e selecione **Install**.

    > **Observação**: Atualmente, a extensão está listada como **Foundry Toolkit**, mas alguns rótulos e comandos do VS Code, ou capturas de tela mais antigas, ainda podem fazer referência a **AI Toolkit**. Neste laboratório, considere que esses nomes se referem à mesma experiência de extensão.

4. Depois de instalar a extensão, selecione o ícone dela na barra lateral para abrir a exibição Foundry Toolkit.

    Você deverá receber uma solicitação para entrar na sua conta do Azure, caso ainda não tenha feito isso.

5. Selecione **Create Project** em **Microsoft Foundry Resources**.

    Se um projeto padrão já estiver ativo, o nome do projeto aparecerá em **My Resources**. Você pode criar um novo projeto clicando com o botão direito do mouse no projeto ativo e selecionando **Switch Default Project in Azure Resources**.

6. Selecione sua assinatura do Azure e o grupo de recursos e, em seguida, insira um nome para o projeto do Foundry a fim de criar um novo projeto para este exercício.

    Quando a implantação for concluída, o projeto deverá aparecer no painel Foundry Toolkit como o projeto padrão.

## Implantar um modelo

No núcleo de qualquer projeto de IA generativa, há pelo menos um modelo de IA generativa. Nesta tarefa, você implantará um modelo do Model Catalog para usar com seu agente.

1. Quando o pop-up "Project deployed successfully" aparecer, selecione o botão **Deploy a new model**. Isso abrirá o Model Catalog.

   > **Dica**: Você também pode acessar o Model Catalog selecionando o ícone **+** ao lado de **Models** na seção Resources ou pressionando **F1** e executando o comando **Foundry Toolkit: Show model catalog**.

1. No Model Catalog, localize o modelo **gpt-5** (você pode usar a barra de pesquisa para encontrá-lo rapidamente).

1. Selecione **Deploy** ao lado do modelo gpt-5.

1. Defina as configurações de implantação:
   - **Deployment name**: Insira um nome, como "gpt-5"
   - **Deployment type**: Selecione **Global Standard** (ou **Standard** se Global Standard não estiver disponível)
   - **Model version**: Mantenha o padrão
   - **Tokens per minute**: Aumente o limite de Tokens per Minute para 150000 ou mais.

1. Selecione **Deploy to Microsoft Foundry** no canto inferior esquerdo.

1. Aguarde a conclusão da implantação. O modelo implantado aparecerá na seção **Models** na exibição Resources.

1. Clique com o botão direito do mouse no nome da implantação do projeto e selecione **Copy Project Endpoint**. Você precisará dessa URL para conectar seu agente ao projeto do Foundry nas próximas etapas.

    ![Captura de tela da cópia do endpoint do projeto na extensão Foundry Toolkit para VS Code.](../Media/vs-code-endpoint.png)

## Clonar o repositório de código inicial

Neste exercício, você usará um código inicial que ajudará a se conectar ao seu projeto do Foundry e a criar um agente que usa ferramentas de servidor MCP.

1. No VS Code, abra a Paleta de Comandos (**Ctrl+Shift+P** ou **View > Command Palette**).

1. Digite **Git: Clone** e selecione-o na lista.

1. Insira a URL do repositório:

    ```
   https://github.com/MicrosoftLearning/mslearn-ai-agents.git
    ```

1. Escolha um local no computador local para clonar o repositório.

1. Quando solicitado, selecione **Open** para abrir o repositório clonado no VS Code.

1. Depois que o repositório for aberto, selecione **File > Open Folder** e navegue até `mslearn-ai-agents/Labfiles/03-mcp-integration`; em seguida, escolha **Select Folder**.

1. No painel Explorer, expanda a pasta **Python** para exibir os arquivos de código deste exercício.

1. Clique com o botão direito do mouse no arquivo **requirements.txt** e selecione **Open in Integrated Terminal**.

1. No terminal, insira o comando a seguir para instalar os pacotes Python necessários em um ambiente virtual:

    ```
   python -m venv labenv
   .\labenv\Scripts\Activate.ps1
   pip install -r requirements.txt
    ```

1. Abra o arquivo **.env**, substitua o espaço reservado **your_project_endpoint** pelo endpoint do seu projeto (copiado do recurso de implantação do projeto na extensão Foundry Toolkit) e verifique se a variável MODEL_DEPLOYMENT_NAME está definida como o nome da implantação do seu modelo. Use **Ctrl+S** para salvar o arquivo depois de fazer essas alterações.

Agora você está pronto para criar um agente de IA que usa ferramentas de servidor MCP para acessar fontes de dados e APIs externas.

## Conectar um Azure AI Agent a um servidor MCP remoto

Nesta tarefa, você se conectará a um servidor MCP remoto, preparará o agente de IA e executará um prompt de usuário.

1. Abra o arquivo **agent.py** no editor de código.

   > **Dica**: Ao adicionar código, mantenha a indentação correta. Use os níveis de indentação dos comentários como guia.

1. Localize o comentário **Add references** e adicione o código a seguir para importar as classes:

    ```python
   # Adicionar referências
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
   from azure.ai.projects.models import PromptAgentDefinition, MCPTool
   from openai.types.responses.response_input_param import McpApprovalResponse, ResponseInputParam
    ```

1. Localize o comentário **Connect to the agents client** e adicione o código a seguir para se conectar ao projeto do Azure AI usando as credenciais atuais do Azure.

    ```python
   # Conectar ao cliente de agentes
   with (
       DefaultAzureCredential() as credential,
       AIProjectClient(endpoint=project_endpoint, credential=credential) as project_client,
       project_client.get_openai_client() as openai_client,
   ):
    ```

1. Abaixo do comentário **Initialize agent MCP tool**, adicione o código a seguir:

    ```python
   # Inicializar a ferramenta MCP do agente
   mcp_tool = MCPTool(
       server_label="api-specs",
       server_url="https://learn.microsoft.com/api/mcp",
       require_approval="always",
   )
    ```

    Esse código se conectará ao servidor MCP remoto do Microsoft Learn Docs. Esse é um serviço hospedado na nuvem que permite aos clientes acessar informações confiáveis e atualizadas diretamente da documentação oficial da Microsoft.

1. Abaixo do comentário **Create a new agent with the MCP tool**, adicione o código a seguir:

    ```python
   # Criar um novo agente com a ferramenta MCP
   agent = project_client.agents.create_version(
       agent_name="MyAgent",
       definition=PromptAgentDefinition(
           model=model_deployment,
           instructions="Você é um agente prestativo que pode usar ferramentas MCP para ajudar os usuários. Use as ferramentas MCP disponíveis para responder a perguntas e realizar tarefas.",
           tools=[mcp_tool],
       ),
   )
   print(f"Agente criado (id: {agent.id}, nome: {agent.name}, versão: {agent.version})")
    ```

    Nesse código, você fornece instruções para o agente e fornece a ele as definições da ferramenta MCP.

1. Localize o comentário **Create a conversation thread** e adicione o código a seguir:

    ```python
   # Criar um thread de conversa
   conversation = openai_client.conversations.create()
   print(f"Conversa criada (id: {conversation.id})")
    ```

1. Localize o comentário **Send initial request that will trigger the MCP tool** e adicione o código a seguir:

    ```python
   # Enviar a solicitação inicial que acionará a ferramenta MCP
   response = openai_client.responses.create(
       conversation=conversation.id,
       input="Forneça os comandos da CLI do Azure para criar um Azure Container App com uma identidade gerenciada.",
      extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
   )
    ```

1. Localize o comentário **Process any MCP approval requests that were generated** e adicione o código a seguir:

    ```python
   # Processar todas as solicitações de aprovação MCP geradas
   # O agente pode emitir várias chamadas de ferramentas, cada uma exigindo sua própria aprovação,
   # por isso repetimos o loop até que não reste nenhuma.
   while True:
       # Coletar todas as solicitações de aprovação MCP da resposta mais recente
       input_list: ResponseInputParam = []
       for item in response.output:
           if item.type == "mcp_approval_request":
               if item.server_label == "api-specs" and item.id:
                   # Aprovar automaticamente a solicitação MCP para permitir que o agente prossiga
                   input_list.append(
                       McpApprovalResponse(
                           type="mcp_approval_response",
                           approve=True,
                           approval_request_id=item.id,
                       )
                   )

       # Não são necessárias mais aprovações -> o agente produziu sua resposta final
       if not input_list:
           break

       # Enviar a resposta de aprovação e recuperar a próxima resposta
       response = openai_client.responses.create(
           input=input_list,
           previous_response_id=response.id,
           extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
       )

   print(f"\nResposta do agente: {response.output_text}")
    ```

    Esse código verifica se há solicitações de aprovação MCP na resposta do agente e as aprova automaticamente.

1. Localize o comentário **Clean up resources by deleting the agent version** e adicione o código a seguir:

    ```python
   # Limpar os recursos excluindo a versão do agente
   project_client.agents.delete_version(agent_name=agent.name, agent_version=agent.version)
   print("Agente excluído")
    ```

1. Salve o arquivo de código (*CTRL+S*) quando terminar.

## Testar a conexão com o servidor MCP remoto

Agora você está pronto para executar o aplicativo e ver como o agente usa a ferramenta MCP para recuperar informações do servidor MCP remoto do Microsoft Learn Docs.

1. No terminal integrado, insira o comando a seguir para executar o aplicativo:

    ```
   az login
    ```

    ```
   python agent.py
    ```

1. Aguarde o agente processar o prompt, usando o servidor MCP para encontrar uma ferramenta adequada para recuperar as informações solicitadas. Você deverá ver uma saída semelhante à seguinte:

    ````
   Agente criado (id: MyAgent:2, nome: MyAgent, versão: 2)
   Conversa criada (id: conv_086911ecabcbc05700BBHIeNRoPSO5tKPHiXRkgHuStYzy27BS)

   Resposta do agente: Aqui estão os comandos da CLI do Azure para criar um Azure Container App com uma identidade gerenciada:

   **1. Para uma identidade gerenciada atribuída pelo sistema**
    ```sh
    az containerapp create \
    --name <CONTAINERAPP_NAME> \
    --resource-group <RESOURCE_GROUP> \
    --environment <CONTAINERAPPS_ENVIRONMENT> \
    --image <CONTAINER_IMAGE> \
    --identity 'system'
    ```

   [continuação...]

   Agente excluído

    ````

    Observe que o agente conseguiu invocar a ferramenta MCP para atender automaticamente à solicitação.

1. Você pode atualizar a entrada na solicitação para pedir informações diferentes. Em cada caso, o agente tentará encontrar a documentação técnica usando a ferramenta MCP.

## Criar um servidor MCP com ferramentas personalizadas

Além de se conectar a servidores MCP remotos, você também pode criar suas próprias ferramentas de servidor MCP personalizadas e conectá-las ao agente. Um servidor Model Context Protocol (MCP) é um componente que hospeda ferramentas que podem ser chamadas. Essas ferramentas são funções Python que podem ser expostas a agentes de IA. Quando as ferramentas são anotadas com `@mcp.tool()`, elas se tornam detectáveis pelo cliente, permitindo que um agente de IA as chame de forma autônoma durante uma conversa ou tarefa. Nesta tarefa, você adicionará ferramentas que permitirão a um agente realizar consultas e recomendações de inventário.

1. Abra o arquivo **server.py** no editor de código.

    Neste arquivo de código, você definirá as ferramentas que o agente poderá usar para simular um serviço de back-end da loja de varejo. Observe o código de configuração do servidor na parte superior do arquivo. Ele usa `FastMCP` para iniciar rapidamente uma instância de servidor MCP chamada "Inventory". Esse servidor hospedará as ferramentas que você definir e as disponibilizará ao agente durante o laboratório.

1. Abaixo do comentário **Add references**, adicione o código a seguir:

    ```python
   # Adicionar referências
   from fastmcp import FastMCP
    ```

1. Abaixo do comentário **Create an MCP server**, adicione o código a seguir para criar uma nova instância de servidor MCP:

    ```python
   # Criar um servidor MCP
   mcp = FastMCP(name="Inventory")
    ```

    Esse código inicializa um novo servidor MCP com o rótulo "Inventory".

1. Localize o comentário **Add an inventory check mcp tool** e adicione o decorador a seguir acima da definição da função, que deverá ficar assim:

    ```python
   # Adicionar uma ferramenta MCP de verificação de inventário
   @mcp.tool()
   def get_inventory_levels() -> dict:
      # continuação...
    ```

    Esse dicionário representa um inventário de exemplo. O decorador `@mcp.tool()` registra a função como uma ferramenta no servidor MCP, permitindo que o LLM descubra sua função.

1. Localize o comentário **Add a weekly sales mcp tool** e adicione o decorador a seguir acima da definição da função, que deverá ficar assim:

    ```python
   # Adicionar uma ferramenta MCP de vendas semanais
   @mcp.tool()
   def get_weekly_sales() -> dict:
      # continuação...
    ```

1. Localize o comentário **Run the MCP server** e adicione o código a seguir para iniciar o servidor:

    ```python
   # Executar o servidor MCP
   mcp.run(show_banner=False)
    ```

    Esse código inicia o servidor MCP, disponibilizando suas ferramentas para descoberta e uso pelo agente. Definir `show_banner=False` impede que o banner de inicialização seja impresso em stdout, o que corromperia o protocolo MCP stdio.

1. Salve o arquivo (*CTRL+S*).

## Implementar um cliente MCP para se conectar ao servidor MCP personalizado3

Um cliente MCP é o componente que se conecta ao servidor MCP para descobrir e chamar ferramentas. Você pode entendê-lo como a ponte entre o agente e as funções hospedadas no servidor, permitindo o uso dinâmico de ferramentas em resposta aos prompts do usuário.

1. Navegue até o arquivo **client.py**.

1. Localize o comentário **Add references** e adicione o código a seguir para importar as classes:

    ```python
   # Adicionar referências
   from mcp import ClientSession, StdioServerParameters
   from mcp.client.stdio import stdio_client
    ```

1. No método **connect_to_server**, localize o comentário **Start the MCP server** e adicione o código a seguir:

    ```python
   # Iniciar o servidor MCP
   stdio_transport = await exit_stack.enter_async_context(stdio_client(server_params))
   stdio, write = stdio_transport
    ```

    Em uma configuração de produção padrão, o servidor seria executado separadamente do cliente. Mas, para os fins deste laboratório, o cliente é responsável por iniciar o servidor usando o transporte de entrada/saída padrão. Isso cria um canal de comunicação leve entre os dois componentes e simplifica a configuração do desenvolvimento local.

1. Localize o comentário **Create an MCP client session** e adicione o código a seguir:

    ```python
   # Criar uma sessão de cliente MCP
   session = await exit_stack.enter_async_context(ClientSession(stdio, write))
   await session.initialize()
    ```

    Isso cria uma nova sessão de cliente usando os fluxos de entrada e saída da etapa anterior. Chamar `session.initialize` prepara a sessão para descobrir e chamar ferramentas registradas no servidor MCP.

1. Abaixo do comentário **List available tools**, adicione o código a seguir para verificar se o cliente se conectou ao servidor:

    ```python
   # Listar ferramentas disponíveis
   response = await session.list_tools()
   tools = response.tools
   print("\nConectado ao servidor com as ferramentas:", [tool.name for tool in tools])
    ```

    Agora sua sessão de cliente está pronta para ser usada com o Azure AI Agent.

## Conectar as ferramentas MCP ao seu agente

Nesta tarefa, você conectará as ferramentas do servidor MCP ao seu agente para que ele possa chamá-las em resposta aos prompts do usuário.

> **Dica**: Ao adicionar código, mantenha a indentação correta. Use os níveis de indentação dos comentários como guia.

1. No método **chat_loop**, localize o comentário **Build a function for each tool** e adicione o código a seguir:

    ```python
   # Criar uma função para cada ferramenta
   def make_tool_func(tool_name):
       async def tool_func(**kwargs):
           result = await session.call_tool(tool_name, kwargs)
           return result

       tool_func.__name__ = tool_name
       return tool_func

   # Armazenar as funções em um dicionário para facilitar o acesso ao processar chamadas de funções
   functions_dict = {tool.name: make_tool_func(tool.name) for tool in tools}
    ```

    Esse código encapsula dinamicamente as ferramentas disponíveis no servidor MCP para que possam ser chamadas pelo agente de IA. Cada ferramenta é transformada em uma função assíncrona que o agente pode invocar.

1. Localize o comentário **Create FunctionTool definitions for the agent** e adicione o código a seguir:

    ```python
   # Criar definições FunctionTool para o agente
   mcp_function_tools: FunctionTool = []
   for tool in tools:
       function_tool = FunctionTool(
           name=tool.name,
           description=tool.description,
           parameters={
               "type": "object",
               "properties": {},
               "additionalProperties": False,
           },
           strict=True
       )
       mcp_function_tools.append(function_tool)
    ```

1. Localize o comentário **Create the agent** e adicione o código a seguir:

    ```python
   # Criar o agente
   agent = project_client.agents.create_version(
       agent_name="inventory-agent",
       definition=PromptAgentDefinition(
           model=model_deployment,
           instructions="""
           Você é um assistente de inventário. Veja algumas diretrizes gerais:
           - Recomende reabastecimento se o inventário do item < 10 e as vendas semanais > 15
           - Recomende liquidação se o inventário do item > 20 e as vendas semanais < 5
           """,
           tools=mcp_function_tools
       ),
   )
    ```

   Com essas instruções e ferramentas, o agente pode invocar as ferramentas para recuperar dados de inventário e vendas e, em seguida, usar essas informações para fornecer respostas úteis ao usuário.

1. Localize o comentário **Process function calls** e adicione o código a seguir:

    ```python
   # Processar chamadas de funções
   for item in response.output:
       if item.type == "function_call":
           # Recuperar a ferramenta de função correspondente
           function_name = item.name
           kwargs = json.loads(item.arguments)
           required_function = functions_dict.get(function_name)

           # Invocar a função
           output = await required_function(**kwargs)

           # Acrescentar o texto da saída
           input_list.append(
              FunctionCallOutput(
                 type="function_call_output",
                 call_id=item.call_id,
                 output=output.content[0].text,
              )
           )
    ```

    Esse código verifica se há chamadas de funções na resposta do agente, invoca a função de ferramenta correspondente e prepara a saída para ser enviada de volta ao agente.

1. Localize o comentário **Send function call outputs back to the model and retrieve a response** e adicione o código a seguir:

    ```python
   # Enviar as saídas das chamadas de funções de volta ao modelo e recuperar uma resposta
   if input_list:
      response = openai_client.responses.create(
            input=input_list,
            previous_response_id=response.id,
            extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
      )
   print(f"Resposta do agente: {response.output_text}")
    ```

1. Salve o arquivo de código (*CTRL+S*) quando terminar.

## Testar as ferramentas MCP personalizadas com seu agente

1. No terminal integrado, insira o comando a seguir para executar o aplicativo:

    ```
   python client.py
    ```

1. Quando solicitado, insira um prompt como:

    ```
   Mostre os níveis atuais de inventário de todos os produtos.
    ```

    > **Dica**: Se o aplicativo falhar porque o limite de taxa foi excedido, aguarde alguns segundos e tente novamente. Se não houver cota suficiente disponível em sua assinatura, talvez o modelo não consiga responder.

    Você deverá ver uma saída semelhante à seguinte:

    ```
    MessageRole.AGENT:
    Resposta do agente: Aqui estão os níveis atuais de inventário de todos os itens:

   - Hidratante: 6
   - Shampoo: 8
   - Spray corporal: 28
   [continuação ...]

   Você gostaria de recomendações para reabastecimento ou liquidação? Se quiser, posso verificar as vendas semanais para orientar você.
    ```

    Observe que o agente conseguiu chamar as ferramentas MCP para recuperar dados de inventário e vendas e, em seguida, usar essas informações para fornecer uma resposta útil ao usuário.

1. Você pode continuar a conversa, se quiser. O thread é *stateful*, portanto mantém o histórico da conversa — o que significa que o agente tem o contexto completo para cada resposta.

    Tente inserir prompts como:

    ```
   Há algum produto que deva ser reabastecido?
    ```

    ```
   Quais produtos você recomendaria para liquidação?
    ```

    ```
   Quais são os produtos mais vendidos nesta semana?
    ```

1. Insira `quit` para sair do aplicativo.

    Você também pode usar `deactivate` para sair do ambiente virtual do Python no terminal.

## Limpar

Quando terminar de explorar a extensão Foundry Toolkit for VS Code, você deverá limpar os recursos para evitar custos desnecessários do Azure.

### Excluir seu modelo

1. No VS Code, atualize a exibição **Azure Resources**.

1. Expanda a subseção **Models**.

1. Clique com o botão direito do mouse no modelo implantado e selecione **Delete**.

### Excluir o grupo de recursos

1. Abra o [portal do Azure](https://portal.azure.com).

1. Navegue até o grupo de recursos que contém seus recursos do Microsoft Foundry.

1. Selecione **Delete resource group** e confirme a exclusão.
