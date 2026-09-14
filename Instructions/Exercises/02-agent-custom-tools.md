---
lab:
    title: 'Usar uma função personalizada em um agente de IA'
    description: 'Aprenda a usar funções para adicionar recursos personalizados aos seus agentes.'
    level: 300
    duration: 50
    islab: true
    status: 'released'
layout: default
---

# Usar uma função personalizada em um agente de IA

Neste exercício, você explorará a criação de um agente que pode usar funções personalizadas como uma ferramenta para concluir tarefas. O agente atuará como um assistente de astronomia que pode fornecer informações sobre eventos astronômicos e calcular o custo do aluguel de telescópios com base nas entradas do usuário. Você definirá as ferramentas de função e implementará a lógica para processar as chamadas de função feitas pelo agente.

> **Dica**: o código usado neste exercício é baseado no Microsoft Foundry SDK para Python. Você pode desenvolver soluções semelhantes usando os SDKs para Microsoft .NET, JavaScript e Java. Consulte [bibliotecas cliente do Microsoft Foundry SDK](https://learn.microsoft.com/azure/ai-foundry/how-to/develop/sdk-overview) para obter detalhes.

Este exercício deve levar aproximadamente **50** minutos para ser concluído.

> **Observação**: algumas das tecnologias usadas neste exercício estão em versão prévia ou em desenvolvimento ativo. Você poderá encontrar algum comportamento inesperado, avisos ou erros.

## Pré-requisitos

Antes de iniciar este exercício, verifique se você tem:

- O [Visual Studio Code](https://code.visualstudio.com/) instalado em seu computador local
- Uma [assinatura do Azure](https://azure.microsoft.com/free/) ativa
- O [Python 3.13](https://www.python.org/downloads/) instalado
- O [Git](https://git-scm.com/downloads) instalado em seu computador local

> \* O Python 3.14 ainda não é compatível: algumas dependências não têm uma compilação para a versão 3.14. Este laboratório foi testado com o Python 3.13.12.

## Criar um projeto do Foundry com a extensão Foundry Toolkit for VS Code

Como desenvolvedor, você pode passar algum tempo trabalhando no portal do Foundry, mas também é provável que passe bastante tempo no Visual Studio Code. A extensão Foundry Toolkit for VS Code oferece uma maneira conveniente de trabalhar com os recursos do projeto do Foundry sem sair do ambiente de desenvolvimento.

1. Abra o Visual Studio Code.

2. Selecione **Extensions** no painel esquerdo (ou pressione **Ctrl+Shift+X**).

3. Procure no marketplace de extensões a extensão `Foundry Toolkit for VS Code` da Microsoft e selecione **Install**.

    A instalação da extensão Foundry Toolkit adicionará a extensão Foundry Toolkit ao VS Code.

    > **Observação**: atualmente, a extensão aparece como **Foundry Toolkit**, mas alguns rótulos e comandos do VS Code, ou capturas de tela mais antigas, ainda podem fazer referência a **AI Toolkit**. Neste laboratório, considere que esses nomes se referem à mesma experiência de extensão.

4. Depois de instalar a extensão, selecione o ícone do Foundry Toolkit na barra lateral.

    Se ainda não tiver feito isso, você deverá receber uma solicitação para entrar em sua conta do Azure.

5. Selecione **Create Project** em **Microsoft Foundry Resources**.

    Se já houver um projeto padrão ativo, o nome do projeto aparecerá em **My Resources**. Você pode criar um novo projeto clicando com o botão direito do mouse no projeto ativo e selecionando **Switch Default Project in Azure Extension**.

6. Selecione sua assinatura do Azure e o grupo de recursos e, em seguida, insira um nome para o projeto do Foundry a fim de criar um novo projeto para este exercício.

    Quando a implantação for concluída, você deverá ver o projeto aparecer no painel do Foundry Toolkit como o projeto padrão.

## Implantar um modelo

No núcleo de qualquer projeto de IA generativa, há pelo menos um modelo de IA generativa. Nesta tarefa, você implantará um modelo do Model Catalog para usar com seu agente.

1. Quando o pop-up "Project deployed successfully" aparecer, selecione o botão **Deploy a new model**. Isso abrirá o Model Catalog.

   > **Dica**: você também pode acessar o Model Catalog selecionando o ícone **+** ao lado de **Models** na seção Resources ou pressionando **F1** e executando o comando **Foundry Toolkit: Show model catalog**.

1. No Model Catalog, localize o modelo **gpt-5** (você pode usar a barra de pesquisa para encontrá-lo rapidamente).

1. Selecione **Deploy** ao lado do modelo gpt-5.

1. Defina as configurações da implantação:
   - **Deployment name**: insira um nome como "gpt-5"
   - **Deployment type**: selecione **Global Standard** (ou **Standard** se Global Standard não estiver disponível)
   - **Model version**: deixe o valor padrão
   - **Tokens per minute**: deixe o valor padrão

1. Selecione **Deploy to Microsoft Foundry** no canto inferior esquerdo.

1. Aguarde a conclusão da implantação. O modelo implantado aparecerá na seção **Models** do modo de exibição Resources.

1. Clique com o botão direito do mouse no nome da implantação do projeto e selecione **Copy Project Endpoint**. Você precisará dessa URL para conectar seu agente ao projeto do Foundry nas próximas etapas.

    ![Captura de tela da cópia do endpoint do projeto na extensão Foundry Toolkit do VS Code.](../Media/vs-code-endpoint.png)

## Clonar o repositório do código inicial

Neste exercício, você usará um código inicial que ajudará a conectar-se ao seu projeto do Foundry e criar um agente que usa ferramentas de função personalizadas.

1. No VS Code, abra a Paleta de Comandos (**Ctrl+Shift+P** ou **View > Command Palette**).

1. Digite **Git: Clone** e selecione-o na lista.

1. Insira a URL do repositório:

    ```
   https://github.com/MicrosoftLearning/mslearn-ai-agents.git
    ```

1. Escolha um local em seu computador local para clonar o repositório.

1. Quando solicitado, selecione **Open** para abrir o repositório clonado no VS Code.

1. Depois que o repositório for aberto, selecione **File > Open Folder** e navegue até `mslearn-ai-agents/Labfiles/02-agent-custom-tools`. Em seguida, escolha **Select Folder**.

1. No painel Explorer, expanda a pasta **Python** para exibir os arquivos de código deste exercício.

1. Clique com o botão direito do mouse no arquivo **requirements.txt** e selecione **Open in Integrated Terminal**.

1. No terminal, insira o comando a seguir para instalar os pacotes Python necessários em um ambiente virtual:

    ```
   python -m venv labenv
   .\labenv\Scripts\Activate.ps1
   pip install -r requirements.txt
    ```

1. Abra o arquivo **.env**, substitua o espaço reservado **your_project_endpoint** pelo endpoint do seu projeto (copiado do recurso de implantação do projeto na extensão Foundry Toolkit do VS Code) e verifique se a variável MODEL_DEPLOYMENT_NAME está definida como o nome da implantação do modelo. Use **Ctrl+S** para salvar o arquivo depois de fazer essas alterações.

Agora você está pronto para criar um agente de IA que usa ferramentas do servidor MCP para acessar fontes de dados e APIs externas.

## Criar uma função para o agente usar

1. Abra o arquivo **functions.py** e examine o código existente.

    Esse arquivo inclui várias funções que você pode usar como ferramentas para seu agente. As funções usam arquivos de exemplo localizados na pasta **data** para recuperar informações sobre eventos e locais astronômicos.

1. Localize o comentário **Determinar o próximo evento astronômico visível para um determinado local** e adicione o código a seguir:

    ```python
   # Determinar o próximo evento astronômico visível para um determinado local
   def next_visible_event(location: str) -> str:
       """Retorna o próximo evento astronômico visível para um local."""
       today = int(datetime.now().strftime("%m%d"))
       loc = location.lower().replace(" ", "_")

       # Recuperar o próximo evento visível a partir do local, começando pelos eventos posteriores neste ano
       for name, event_type, date, date_str, locs in EVENTS:
           if loc in locs and date >= today:
               return json.dumps({"event": name, "type": event_type, "date": date_str, "visible_from": sorted(locs)})

       return json.dumps({"message": f"Nenhum evento futuro encontrado para {location}."})
    ```

    Essa função verifica os dados de eventos de exemplo para encontrar o próximo evento astronômico visível a partir de um local especificado e retorna os detalhes do evento como uma cadeia de caracteres JSON. Em seguida, vamos criar um agente que possa usar essa função.

## Conectar-se ao projeto do Foundry

1. Abra o arquivo **agent.py**.

   > **Dica**: ao adicionar código, mantenha a indentação correta. Use os níveis de indentação dos comentários como guia.

1. Localize o comentário **Adicionar referências** e adicione o código a seguir para importar as classes necessárias para criar um agente de IA do Azure que usa uma ferramenta de função:

    ```python
   # Adicionar referências
   from azure.ai.projects import AIProjectClient
   from azure.ai.projects.models import FunctionTool
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects.models import PromptAgentDefinition, FunctionTool
   from openai.types.responses.response_input_param import FunctionCallOutput, ResponseInputParam
   from functions import next_visible_event, calculate_observation_cost, generate_observation_report
    ```

    Observe que as funções definidas no arquivo **functions.py** são importadas para que possam ser usadas como ferramentas do agente.

1. Localize o comentário **Conectar ao cliente do projeto** e adicione o código a seguir:

    ```python
   # Conectar ao cliente do projeto
   with (
       DefaultAzureCredential() as credential,
       AIProjectClient(endpoint=project_endpoint, credential=credential) as project_client,
       project_client.get_openai_client() as openai_client,
   ):
    ```

## Definir as ferramentas de função

Nesta tarefa, você definirá cada uma das ferramentas de função que o agente pode usar. Os parâmetros de cada ferramenta de função são definidos usando um esquema JSON, que especifica o nome, o tipo, a descrição e outros atributos de cada parâmetro da função.

1. Localize o comentário **Definir a ferramenta de função de evento** e adicione o código a seguir:

    ```python
   # Definir a ferramenta de função de evento
   event_tool = FunctionTool(
       name="next_visible_event",
       description="Obter o próximo evento visível em um determinado local.",
       parameters={
           "type": "object",
           "properties": {
               "location": {
                   "type": "string",
                   "description": "continente onde encontrar o próximo evento visível (por exemplo, 'north_america', 'south_america', 'australia')",
               },
           },
           "required": ["location"],
           "additionalProperties": False,
       },
       strict=True,
   )
    ```

1. Localize o comentário **Definir a ferramenta de função de custo de observação** e adicione o código a seguir:

    ```python
   # Definir a ferramenta de função de custo de observação
   cost_tool = FunctionTool(
       name="calculate_observation_cost",
       description="Calcular o custo de uma observação com base na categoria do telescópio, no número de horas e no nível de prioridade.",
       parameters={
           "type": "object",
           "properties": {
               "telescope_tier": {
                   "type": "string",
                   "description": "a categoria do telescópio (por exemplo, 'standard', 'advanced', 'premium')",
               },
               "hours": {
                   "type": "number",
                   "description": "o número de horas da observação",
               },
               "priority": {
                   "type": "string",
                   "description": "o nível de prioridade da observação (por exemplo, 'low', 'normal', 'high')",
               },
           },
           "required": ["telescope_tier", "hours", "priority"],
           "additionalProperties": False,
       },
       strict=True,
   )
    ```

1. Localize o comentário **Definir a ferramenta de função de geração de relatório de observação** e adicione o código a seguir:

    ```python
   # Definir a ferramenta de função de geração de relatório de observação
   report_tool = FunctionTool(
       name="generate_observation_report",
       description="Gerar um relatório que resume uma observação astronômica",
       parameters={
           "type": "object",
           "properties": {
               "event_name": {
                   "type": "string",
                   "description": "o nome do evento astronômico que está sendo observado",
               },
               "location": {
                   "type": "string",
                   "description": "o local do observador",
               },
               "telescope_tier": {
                   "type": "string",
                   "description": "a categoria do telescópio usado para a observação (por exemplo, 'standard', 'advanced', 'premium')",
               },
               "hours": {
                   "type": "number",
                   "description": "o número de horas que o telescópio foi usado para a observação",
               },
               "priority": {
                   "type": "string",
                   "description": "o nível de prioridade da observação (por exemplo, 'low', 'normal', 'high')",
               },
               "observer_name": {
                   "type": "string",
                   "description": "o nome da pessoa que realizou a observação",
               },
           },
           "required": ["event_name", "location", "telescope_tier", "hours", "priority", "observer_name"],
           "additionalProperties": False,
       },
       strict=True,
   )
    ```

## Criar o agente que usa as ferramentas de função

Agora que você definiu as ferramentas de função, pode criar um agente que use essas ferramentas para concluir tarefas.

1. Localize o comentário **Criar um novo agente com as ferramentas de função** e adicione o código a seguir:

    ```python
   # Criar um novo agente com as ferramentas de função
   agent = project_client.agents.create_version(
       agent_name="astronomy-agent",
       definition=PromptAgentDefinition(
           model=model_deployment,
           instructions=
               """Você é um assistente de observações astronômicas que ajuda os usuários a encontrar
               informações sobre eventos astronômicos e calcular custos de aluguel de telescópios.
               Use as ferramentas disponíveis para ajudar os usuários com suas dúvidas.""",
           tools=[event_tool, cost_tool, report_tool],
       ),
   )
    ```

## Enviar uma mensagem ao agente e processar a resposta

Agora que você criou o agente com as ferramentas de função, pode enviar mensagens ao agente e processar suas respostas.

1. Localize o comentário **Criar uma conversa para a sessão de chat** e adicione o código a seguir:

    ```python
   # Criar uma conversa para a sessão de chat
   conversation = openai_client.conversations.create()
    ```

    Esse código cria a sessão de chat com o agente.

1. Localize o comentário **Criar uma lista para armazenar as saídas das chamadas de função que serão enviadas de volta como entrada para o agente** e adicione o código a seguir:

    ```python
   # Criar uma lista para armazenar as saídas das chamadas de função que serão enviadas de volta como entrada para o agente
   input_list: ResponseInputParam = []
    ```

1. Localize o comentário **Enviar um prompt ao agente** e adicione o código a seguir:

    ```python
   # Enviar um prompt ao agente
   openai_client.conversations.items.create(
       conversation_id=conversation.id,
       items=[{"type": "message", "role": "user", "content": user_input}],
   )
    ```

1. Localize o comentário **Recuperar a resposta do agente, que pode incluir chamadas de função** e adicione o código a seguir:

    ```python
   # Recuperar a resposta do agente, que pode incluir chamadas de função
   response = openai_client.responses.create(
       conversation=conversation.id,
       extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
       input=input_list,
   )

   # Verificar se o status da execução indica falha
   if response.status == "failed":
       print(f"Falha na resposta: {response.error}")
    ```

    Neste código, você envia um prompt do usuário ao agente e recupera a resposta. Você também verifica se a resposta indica uma falha e, se isso ocorrer, imprime o erro.

## Processar chamadas de função e exibir a resposta do agente

1. Localize o comentário **Processar chamadas de função** e adicione o código a seguir para tratar todas as chamadas de função feitas pelo agente:

    ```python
   # Processar chamadas de função
   for item in response.output:
       if item.type == "function_call":
           # Recuperar a ferramenta de função correspondente
           function_name = item.name
           result = None
           if item.name == "next_visible_event":
               result = next_visible_event(**json.loads(item.arguments))
           elif item.name == "calculate_observation_cost":
               result = calculate_observation_cost(**json.loads(item.arguments))
           elif item.name == "generate_observation_report":
               result = generate_observation_report(**json.loads(item.arguments))

           # Acrescentar o texto da saída
           input_list.append(
               FunctionCallOutput(
                   type="function_call_output",
                   call_id=item.call_id,
                   output=result,
               )
           )
    ```

    Esse código itera pelos itens da resposta do agente para verificar se há chamadas de função. Se uma chamada de função for encontrada, ele recuperará a ferramenta de função correspondente, executará a função com os argumentos fornecidos e acrescentará o resultado à lista de entradas que será enviada de volta ao agente.

1. Localize o comentário **Enviar as saídas das chamadas de função de volta ao modelo e recuperar uma resposta** e adicione o código a seguir:

    ```python
   # Enviar as saídas das chamadas de função de volta ao modelo e recuperar uma resposta
   if input_list:
       response = openai_client.responses.create(
           input=input_list,
           previous_response_id=response.id,
           extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
       )
   # Exibir a resposta do agente
   print(f"AGENTE: {response.output_text}")
    ```

    Esse código verifica se há saídas de chamadas de função na lista de entradas e, se houver, envia-as de volta ao agente como entrada para recuperar uma resposta atualizada. Por fim, ele imprime a resposta do agente.

1. Localize o comentário **Excluir o agente quando terminar** e adicione o código a seguir:

    ```python
   # Excluir o agente quando terminar
   project_client.agents.delete_version(agent_name=agent.name, agent_version=agent.version)
   print("Agente excluído.")
    ```

1. Examine o código completo que você adicionou ao arquivo. Agora ele deverá incluir seções que:
   - Importam as bibliotecas necessárias
    - Conectam-se ao projeto do Foundry e ao cliente OpenAI
    - Definem as ferramentas de função que o agente usará
    - Criam um agente com essas ferramentas de função
    - Enviam uma mensagem ao agente e recuperam a resposta
    - Processam as chamadas de função feitas pelo agente e enviam as saídas de volta ao agente
    - Exibem a resposta do agente
    - Excluem o agente quando terminarem

1. Salve o arquivo de código (**CTRL+S**) quando terminar.

## Executar o aplicativo do agente

1. No terminal integrado, insira o comando a seguir para executar o aplicativo:

    ```
   az login
    ```

    ```
   python agent.py
    ```

1. Quando solicitado, insira um prompt como:

    ```
   Encontre o próximo evento que posso observar da América do Sul e informe o custo de 5 horas de uso de um telescópio premium com prioridade normal.
    ```

    Observe que esse prompt solicita que o agente use as duas ferramentas de função definidas: `next_visible_event` e `calculate_observation_cost`. O agente consegue invocar as duas funções no mesmo turno da conversa e usar as saídas dessas chamadas de função para fornecer uma resposta útil ao usuário.

    > **Dica**: se o aplicativo falhar porque o limite de taxa foi excedido, aguarde alguns segundos e tente novamente. Se não houver cota suficiente disponível em sua assinatura, talvez o modelo não consiga responder.

    Você deverá ver uma saída semelhante à seguinte:

    ```output
   AGENTE: O próximo evento astronômico que você pode observar da América do Sul é a Conjunção Júpiter-Vênus, que ocorrerá em 1º de maio.
   O custo de 5 horas de uso de um telescópio premium com prioridade normal para essa observação será de $1,875.
    ```

1. Insira um prompt de acompanhamento para gerar um relatório de observação, como:

    ```
   Gere essas informações em um relatório para Bellows College.
    ```

    Você deverá ver uma resposta semelhante à seguinte:

    ```output
   AGENTE: Aqui está seu relatório para Bellows College:

   - Próximo evento astronômico visível: Conjunção Júpiter-Vênus
   - Data: 1º de maio
   - Visível a partir de: América do Sul
   - Detalhes da observação:
       - Categoria do telescópio: Premium
       - Duração: 5 horas
       - Prioridade: Normal
   - Custo da observação: $1,875

   Um relatório formal foi gerado para Bellows College.
    ```

    No explorador de arquivos, você verá que um novo arquivo chamado `report-<event-type>.txt` foi criado. Ele contém o relatório gerado. Você pode abrir esse arquivo para exibir o conteúdo do relatório.

1. Insira `quit` para sair do aplicativo.

    Você também pode usar `deactivate` para sair do ambiente virtual do Python no terminal.

## Limpar

Quando terminar de explorar a extensão Foundry Toolkit for VS Code, limpe os recursos para evitar incorrer em custos desnecessários do Azure.

### Excluir seu modelo

1. No VS Code, atualize o modo de exibição **Azure Resources**.

1. Expanda a subseção **Models**.

1. Clique com o botão direito do mouse no modelo implantado e selecione **Delete**.

### Excluir o grupo de recursos

1. Abra o [portal do Azure](https://portal.azure.com).

1. Navegue até o grupo de recursos que contém seus recursos do Microsoft Foundry.

1. Selecione **Delete resource group** e confirme a exclusão.
