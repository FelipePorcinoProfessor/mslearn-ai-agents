---
lab:
    title: 'Conectar-se a agentes remotos com o protocolo A2A'
    description: 'Use o protocolo A2A para colaborar com agentes remotos.'
    level: 300
    duration: 30
    islab: true
    status: 'released'
    layout: default
---

# Conectar-se a agentes remotos com o protocolo A2A

Neste exercício, você usará o Azure AI Agent Service com o protocolo A2A para criar agentes remotos simples que interagem entre si. Esses agentes ajudarão redatores técnicos a preparar suas publicações de blog para desenvolvedores. Um agente de títulos gerará um título, e um agente de esboço usará o título para desenvolver um esboço conciso para o artigo. Vamos começar.

> **Dica**: O código usado neste exercício baseia-se no Microsoft Foundry SDK para Python. Você pode desenvolver soluções semelhantes usando os SDKs para Microsoft .NET, JavaScript e Java. Consulte [Bibliotecas de cliente do Microsoft Foundry SDK](https://learn.microsoft.com/azure/ai-foundry/how-to/develop/sdk-overview) para obter detalhes.

Este exercício deve levar aproximadamente **30** minutos para ser concluído.

> **Observação**: Algumas das tecnologias usadas neste exercício estão em versão prévia ou em desenvolvimento ativo. Você pode encontrar algum comportamento inesperado, avisos ou erros.

## Pré-requisitos

Antes de iniciar este exercício, verifique se você tem:

- [Visual Studio Code](https://code.visualstudio.com/) instalado em seu computador local
- Uma [assinatura do Azure](https://azure.microsoft.com/free/) ativa
- [Python 3.13](https://www.python.org/downloads/) instalado
- [Git](https://git-scm.com/downloads) instalado em seu computador local

> \* O Python 3.14 ainda não é compatível: algumas dependências não têm uma compilação para a versão 3.14. Este laboratório foi testado com o Python 3.13.12.

## Criar um projeto do Foundry com a extensão Foundry Toolkit para VS Code

Como desenvolvedor, você pode passar algum tempo trabalhando no portal do Foundry; mas também é provável que passe bastante tempo no Visual Studio Code. A extensão Foundry Toolkit para VS Code oferece uma maneira conveniente de trabalhar com os recursos do projeto do Foundry sem sair do ambiente de desenvolvimento.

1. Abra o Visual Studio Code.

2. Selecione **Extensions** no painel esquerdo (ou pressione **Ctrl+Shift+X**).

3. Pesquise no marketplace de extensões a extensão `Foundry Toolkit` da Microsoft e selecione **Install**.

    > **Observação**: Atualmente, a extensão está listada como **Foundry Toolkit**, mas alguns rótulos e comandos do VS Code, ou capturas de tela mais antigas, ainda podem fazer referência a **AI Toolkit**. Neste laboratório, considere que esses nomes se referem à mesma experiência da extensão.

4. Depois de instalar a extensão, selecione o ícone dela na barra lateral para abrir a exibição do Foundry Toolkit.

    Será solicitado que você entre em sua conta do Azure, caso ainda não tenha feito isso.

5. Selecione **Create Project** em **Microsoft Foundry Resources**.

    Se já houver um projeto padrão ativo, o nome do projeto aparecerá em **My Resources**. Você pode criar um novo projeto clicando com o botão direito do mouse no projeto ativo e selecionando **Switch Default Project in Azure Extension**.

6. Selecione sua assinatura do Azure e o grupo de recursos e, em seguida, insira um nome para o projeto do Foundry a fim de criar um novo projeto para este exercício.

    Quando a implantação for concluída, o projeto deverá aparecer no painel do Foundry Toolkit como o projeto padrão.

## Implantar um modelo

No núcleo de qualquer projeto de IA generativa há pelo menos um modelo de IA generativa. Nesta tarefa, você implantará um modelo do Model Catalog para usar com seu agente.

1. Quando o pop-up "Project deployed successfully" aparecer, selecione o botão **Deploy a new model**. Isso abrirá o Model Catalog.

   > **Dica**: Você também pode acessar o Model Catalog selecionando o ícone **+** ao lado de **Models** na seção Resources ou pressionando **F1** e executando o comando **Foundry Toolkit: Show model catalog**.

1. No Model Catalog, localize o modelo **gpt-5** (você pode usar a barra de pesquisa para encontrá-lo rapidamente).

1. Selecione **Deploy** ao lado do modelo gpt-5.

1. Defina as configurações de implantação:
   - **Deployment name**: insira um nome como "gpt-5"
   - **Deployment type**: selecione **Global Standard** (ou **Standard**, se Global Standard não estiver disponível)
   - **Model version**: mantenha o padrão
   - **Tokens per minute**: mantenha o padrão

1. Selecione **Deploy to Microsoft Foundry** no canto inferior esquerdo.

1. Aguarde a conclusão da implantação. O modelo implantado aparecerá na seção **Models** do modo de exibição Resources.

1. Clique com o botão direito do mouse no nome da implantação do projeto e selecione **Copy Project Endpoint**. Você precisará dessa URL para conectar seu agente ao projeto do Foundry nas próximas etapas.

    ![Captura de tela da cópia do ponto de extremidade do projeto na extensão Foundry Toolkit do VS Code.](../Media/vs-code-endpoint.png)

## Clonar o repositório do código inicial

Para este exercício, você usará um código inicial que ajudará a se conectar ao projeto do Foundry e a criar um agente capaz de processar dados de despesas. Você clonará esse código de um repositório do GitHub.

1. No VS Code, abra a Paleta de Comandos (**Ctrl+Shift+P** ou **View > Command Palette**).

1. Digite **Git: Clone** e selecione-o na lista.

1. Insira a URL do repositório:

    ```
   https://github.com/MicrosoftLearning/mslearn-ai-agents.git
    ```

1. Escolha um local em seu computador local para clonar o repositório.

1. Quando solicitado, selecione **Open** para abrir o repositório clonado no VS Code.

1. Depois que o repositório for aberto, selecione **File > Open Folder**, navegue até `mslearn-ai-agents/Labfiles/09-build-remote-agents-with-a2a` e selecione **Select Folder**.

1. No painel Explorer, expanda a pasta **Python** para exibir os arquivos de código deste exercício.

1. No modo de exibição Explorer, navegue até a pasta **Labfiles/09-build-remote-agents-with-a2a/Python** para localizar o código inicial deste exercício.

    Os arquivos fornecidos incluem:

    ```output
   python
   ├── outline_agent/
   │   ├── agent.py
   │   ├── agent_executor.py
   │   └── server.py
   ├── routing_agent/
   │   ├── agent.py
   │   └── server.py
   ├── title_agent/
   │   ├── agent.py
   |   ├── agent_executor.py
   │   └── server.py
   ├── client.py
   └── run_all.py
    ```

    Cada pasta de agente contém o código do agente de IA do Azure e um servidor para hospedar o agente. O **routing agent** é responsável por descobrir e se comunicar com os agentes **title** e **outline**. O **client** permite que os usuários enviem prompts ao routing agent. `run_all.py` inicia todos os servidores e executa o cliente.

1. Clique com o botão direito do mouse no arquivo **requirements.txt** e selecione **Open in Integrated Terminal**.

1. No terminal, insira o comando a seguir para instalar os pacotes Python necessários em um ambiente virtual:

    ```
   python -m venv labenv
   .\labenv\Scripts\Activate.ps1
   pip install -r requirements.txt
    ```

1. Abra o arquivo **.env**, substitua o espaço reservado **your_project_endpoint** pelo ponto de extremidade do seu projeto (copiado do recurso de implantação do projeto na extensão Foundry Toolkit) e verifique se a variável MODEL_DEPLOYMENT_NAME está definida como o nome da implantação do seu modelo. Use **Ctrl+S** para salvar o arquivo depois de fazer essas alterações.

## Criar um agente detectável

Nesta tarefa, você criará o agente de títulos que ajuda redatores a criar títulos modernos para seus artigos. Você também definirá as habilidades e o cartão do agente exigidos pelo protocolo A2A para tornar o agente detectável.

> **Dica**: Ao adicionar código, mantenha a indentação correta. Use os comentários existentes como guia, inserindo o novo código no mesmo nível de indentação.

1. Abra o arquivo **title_agent/agent.py** no editor de código.

1. Localize o comentário **Create the agents client** e adicione o código a seguir para se conectar ao projeto de IA do Azure:

    > **Dica**: Tenha cuidado para manter o nível correto de indentação.

    ```python
   # Criar o cliente dos agentes
   self.client = AgentsClient(
       endpoint=os.environ['PROJECT_ENDPOINT'],
       credential=DefaultAzureCredential(
           exclude_environment_credential=True,
           exclude_managed_identity_credential=True
       )
   )
    ```

1. Localize o comentário **Create the title agent** e adicione o código a seguir para criar o agente:

    ```python
   # Criar o agente de títulos
   self.agent = self.client.create_agent(
       model=os.environ['MODEL_DEPLOYMENT_NAME'],
       name='title-agent',
       instructions="""
       Você é um assistente de escrita prestativo.
       Dado um tópico sobre o qual o usuário deseja escrever, sugira um único título claro e atraente para uma publicação de blog.
       """,
   )
    ```

1. Localize o comentário **Create a thread for the chat session** e adicione o código a seguir para criar a thread do chat:

    ```python
   # Criar uma thread para a sessão de chat
   thread = self.client.threads.create()
    ```

1. Localize o comentário **Send user message** e adicione este código para enviar o prompt do usuário:

    ```python
   # Enviar a mensagem do usuário
   self.client.messages.create(thread_id=thread.id, role=MessageRole.USER, content=user_message)
    ```

1. No comentário **Create and run the agent**, adicione o código a seguir para iniciar a geração da resposta do agente:

    ```python
   # Criar e executar o agente
   run = self.client.runs.create_and_process(thread_id=thread.id, agent_id=self.agent.id)
    ```

    O código fornecido no restante do arquivo processará e retornará a resposta do agente.

1. Salve o arquivo de código (*CTRL+S*). Agora você está pronto para compartilhar as habilidades e o cartão do agente com o protocolo A2A.

1. Abra o arquivo **title_agent/server.py** no editor de código.

1. Localize o comentário **Define agent skills** e adicione o código a seguir para especificar a funcionalidade do agente:

    ```python
   # Definir as habilidades do agente
   skills = [
       AgentSkill(
           id='generate_blog_title',
           name='Gerar título do blog',
           description='Gera um título de blog com base em um tópico',
           tags=['title'],
           examples=[
               'Você pode me dar um título para este artigo?',
           ],
       ),
   ]
    ```

1. Localize o comentário **Create agent card** e adicione este código para definir os metadados que tornam o agente detectável:

    ```python
   # Criar o cartão do agente
   agent_card = AgentCard(
       name='Agente de títulos do Microsoft Foundry',
       description='Um agente inteligente gerador de títulos desenvolvido com o Foundry. '
       'Posso ajudar você a gerar títulos atraentes para seus artigos.',
       url=f'http://{host}:{port}/',
       version='1.0.0',
       default_input_modes=['text'],
       default_output_modes=['text'],
       capabilities=AgentCapabilities(),
       skills=skills,
   )
    ```

1. Localize o comentário **Create agent executor** e adicione o código a seguir para inicializar o executor do agente usando o cartão do agente:

    ```python
   # Criar o executor do agente
   agent_executor = create_foundry_agent_executor(agent_card)
    ```

    O executor do agente atuará como um wrapper para o agente de títulos que você criou.

1. Localize o comentário **Create request handler** e adicione o código a seguir para tratar as solicitações recebidas usando o executor:

    ```python
   # Criar o manipulador de solicitações
   request_handler = DefaultRequestHandler(
       agent_executor=agent_executor, task_store=InMemoryTaskStore()
   )
    ```

1. No comentário **Create A2A application**, adicione este código para criar a instância do aplicativo compatível com A2A:

    ```python
   # Criar o aplicativo A2A
   a2a_app = A2AStarletteApplication(
       agent_card=agent_card, http_handler=request_handler
   )
    ```

    Este código cria um servidor A2A que compartilhará as informações do agente de títulos e tratará as solicitações recebidas para esse agente usando o executor do agente de títulos.

1. Salve o arquivo de código (*CTRL+S*) quando terminar.

## Habilitar mensagens entre os agentes

Nesta tarefa, você usará o protocolo A2A para permitir que o routing agent envie mensagens aos outros agentes. Você também permitirá que o agente de títulos receba mensagens implementando a classe do executor do agente.

1. Abra o arquivo **routing_agent/agent.py** no editor de código.

    O routing agent atua como um orquestrador que trata as mensagens dos usuários e determina qual agente remoto deve processar a solicitação.

    Quando uma mensagem do usuário é recebida, o routing agent:
    - Inicia uma thread de conversa.
    - Usa o método `create_and_process` para avaliar o agente mais adequado à mensagem do usuário.
    - A mensagem é encaminhada ao agente apropriado por HTTP usando a função `send_message`.
    - O agente remoto processa a mensagem e retorna uma resposta.

    Por fim, o routing agent captura a resposta e a retorna ao usuário por meio da thread.

    Observe que o método `send_message` é assíncrono e deve ser aguardado para que a execução do agente seja concluída com êxito.

1. Adicione o código a seguir sob o comentário **Retrieve the remote agent's A2A client using the agent name**:

    ```python
   # Recuperar o cliente A2A do agente remoto usando o nome do agente
   client = self.remote_agent_connections[agent_name]
    ```

1. Localize o comentário **Construct the payload to send to the remote agent** e adicione o código a seguir:

    ```python
   # Construir o payload a ser enviado ao agente remoto
   payload: dict[str, Any] = {
       'message': {
           'role': 'user',
           'parts': [{'kind': 'text', 'text': task}],
           'messageId': message_id,
       },
   }
    ```

1. Localize o comentário **Wrap the payload in a SendMessageRequest object** e adicione o código a seguir:

    ```python
   # Encapsular o payload em um objeto SendMessageRequest
   message_request = SendMessageRequest(id=message_id, params=MessageSendParams.model_validate(payload))
    ```

1. Adicione o código a seguir sob o comentário **Send the message to the remote agent client and await the response**:

    ```python
   # Enviar a mensagem ao cliente do agente remoto e aguardar a resposta
   send_response: SendMessageResponse = await client.send_message(message_request=message_request)
    ```

1. Salve o arquivo de código (*CTRL+S*) quando terminar. Agora o routing agent pode descobrir e enviar mensagens ao agente de títulos. Vamos criar o código do executor do agente para tratar essas mensagens recebidas do routing agent.

1. Abra o arquivo **title_agent/agent_executor.py** no editor de código.

    A implementação da classe `AgentExecutor` deve conter os métodos `execute` e `cancel`. O método cancel foi fornecido para você. O método `execute` inclui um objeto `TaskUpdater` que gerencia eventos e sinaliza ao chamador quando a tarefa é concluída. Vamos adicionar a lógica para a execução da tarefa.

1. No método `execute`, adicione o código a seguir sob o comentário **Process the request**:

    ```python
   # Processar a solicitação
   await self._process_request(context.message.parts, context.context_id, updater)
    ```

1. No método `_process_request`, adicione o código a seguir sob o comentário **Get the title agent**:

    ```python
   # Obter o agente de títulos
   agent = await self._get_or_create_agent()
    ```

1. Adicione o código a seguir sob o comentário **Update the task status**:

    ```python
   # Atualizar o status da tarefa
   await task_updater.update_status(
       TaskState.working,
       message=new_agent_text_message('O agente de títulos está processando sua solicitação...', context_id=context_id),
   )
    ```

1. Localize o comentário **Run the agent conversation** e adicione o código a seguir:

    ```python
   # Executar a conversa do agente
   responses = await agent.run_conversation(user_message)
    ```

1. Localize o comentário **Update the task with the responses** e adicione o código a seguir:

    ```python
   # Atualizar a tarefa com as respostas
   for response in responses:
       await task_updater.update_status(
           TaskState.working,
           message=new_agent_text_message(response, context_id=context_id),
       )
    ```

1. Localize o comentário **Mark the task as complete** e adicione o código a seguir:

    ```python
   # Marcar a tarefa como concluída
   final_message = responses[-1] if responses else 'Tarefa concluída.'
   await task_updater.complete(
       message=new_agent_text_message(final_message, context_id=context_id)
   )
    ```

    Agora seu agente de títulos foi encapsulado com um executor de agente que o protocolo A2A usará para tratar mensagens. Muito bem!

## Testar o aplicativo

1. No terminal integrado, insira os comandos a seguir para executar o aplicativo:

    ```
   az login
    ```

    ```
   python run_all.py
    ```

    O aplicativo usa as credenciais da sua sessão autenticada do Azure para se conectar ao projeto e criar e executar o agente. Você deverá ver alguma saída de cada servidor à medida que ele for iniciado.

1. Aguarde até que o prompt de entrada apareça e, em seguida, insira um prompt como:

    ```
   Crie um título e um esboço para um artigo sobre programação em React.
    ```

    Após alguns instantes, você deverá ver uma resposta do agente com os resultados.

1. Insira `quit` para sair do programa e parar os servidores.

    Você também pode usar `deactivate` para sair do ambiente virtual do Python no terminal.

## Limpar

Se você terminou de explorar o Azure AI Agent Service, deverá excluir os recursos criados neste exercício para evitar custos desnecessários do Azure.

1. Retorne à guia do navegador que contém o portal do Azure (ou reabra o [portal do Azure](https://portal.azure.com) em `https://portal.azure.com` em uma nova guia do navegador) e exiba o conteúdo do grupo de recursos no qual você implantou os recursos usados neste exercício.
1. Na barra de ferramentas, selecione **Delete resource group**.
1. Insira o nome do grupo de recursos e confirme que deseja excluí-lo.
