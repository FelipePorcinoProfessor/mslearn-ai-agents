---
lab:
    title: 'Criar um fluxo de trabalho no Microsoft Foundry'
    description: 'Use o portal do Microsoft Foundry para criar fluxos de trabalho para agentes de IA.'
    level: 300
    duration: 45
    islab: true
    status: 'released'
---

# Criar um fluxo de trabalho no Microsoft Foundry

Neste exercício, você usará o portal do Microsoft Foundry para criar um fluxo de trabalho. Os fluxos de trabalho são ferramentas baseadas em interface do usuário que permitem definir sequências de ações envolvendo agentes de IA. Neste exercício, você criará um fluxo de trabalho que ajuda a resolver solicitações de suporte ao cliente.

**Visão geral do fluxo de trabalho**

- Coletar tickets de suporte recebidos

    O fluxo de trabalho começa com uma matriz predefinida de problemas de suporte ao cliente. Cada item da matriz representa um ticket de suporte individual enviado à ContosoPay.

- Processar os tickets um de cada vez

    Um loop for-each itera sobre a matriz, garantindo que cada ticket de suporte seja tratado de forma independente, usando a mesma lógica de fluxo de trabalho.

- Classificar cada ticket com um agente de IA

    Para cada ticket, o fluxo de trabalho invoca um Triage Agent para classificar o problema como Billing, Technical ou General, juntamente com uma pontuação de confiança.

- Tratar a incerteza com lógica condicional

    Se a pontuação de confiança estiver abaixo de um limite definido, o fluxo de trabalho recomendará informações adicionais para esse ticket.

- Encaminhar com base na categoria do problema

    Os problemas de Billing são sinalizados para escalonamento e removidos do caminho de resolução automatizada.
    Os problemas Technical e General continuam pelo tratamento automatizado.

- Gerar uma resposta recomendada

    Para tickets que não sejam de Billing, o fluxo de trabalho invoca um Resolution Agent para redigir uma resposta de suporte apropriada à categoria.

Este exercício deve levar aproximadamente **30** minutos para ser concluído.

> **Observação**: O construtor de fluxos de trabalho no Microsoft Foundry está atualmente em versão prévia. Você pode encontrar algum comportamento inesperado, avisos ou erros. Se encontrar algum problema que bloqueie seu progresso, talvez seja necessário começar novamente com um novo projeto e fluxo de trabalho.

## Pré-requisitos

Antes de iniciar este exercício, verifique se você tem:

- [Visual Studio Code](https://code.visualstudio.com/) instalado em seu computador local
- Uma [assinatura do Azure](https://azure.microsoft.com/free/) ativa
- O [Python 3.13](https://www.python.org/downloads/) instalado
- O [Git](https://git-scm.com/downloads) instalado em seu computador local

> \* O Python 3.14 ainda não é compatível: algumas dependências não têm uma compilação para 3.14. Este laboratório foi testado com o Python 3.13.12.

## Criar um projeto do Foundry

Vamos começar criando um projeto do Foundry.

1. Em um navegador da Web, abra o [portal do Foundry](https://ai.azure.com) em `https://ai.azure.com` e entre usando suas credenciais do Azure.

1. Verifique se a opção **New Foundry** está definida como *On*.

    ![Captura de tela da opção New Foundry.](../Media/ai-foundry-toggle.png)

2. Talvez seja solicitado que você crie um novo projeto antes de continuar para a experiência do New Foundry. Selecione **Create a new project**.

    ![Captura de tela da solicitação para criar um novo projeto.](../Media/ai-foundry-new-project.png)

    Se essa solicitação não aparecer, selecione o menu suspenso de projetos no canto superior esquerdo e, em seguida, selecione **Create new project**.

3. Insira um nome para seu projeto do Foundry na caixa de texto e selecione **Create**.

    Aguarde alguns instantes para que o projeto seja criado. A nova página inicial do portal do Foundry deverá aparecer com seu projeto selecionado.

4. Feche a caixa de diálogo **Welcome to the new Microsoft Foundry** se ela aparecer.

    A caixa de diálogo pode solicitar que você crie um agente, o que não é necessário neste momento. Os agentes serão criados em uma etapa posterior.

## Criar um fluxo de trabalho de triagem do suporte ao cliente

Nesta seção, você criará um fluxo de trabalho que ajuda a fazer a triagem e responder às solicitações de suporte ao cliente de uma empresa fictícia chamada ContosoPay. O fluxo de trabalho usa dois agentes de IA que classificam e respondem aos tickets de suporte.

1. Na página inicial do portal do Foundry, selecione **Build** no menu da barra de ferramentas.

1. No menu à esquerda, selecione **Agents** e, em seguida, selecione a guia **Workflows**.

1. No canto superior direito, selecione **Create** > **Blank workflow** para criar um novo fluxo de trabalho em branco.

    O tipo de fluxo de trabalho que você criará neste exercício é um fluxo de trabalho sequencial. No entanto, começar com um fluxo de trabalho em branco simplificará o processo de adicionar os nós necessários.

1. Selecione **Save** no visualizador para salvar seu novo fluxo de trabalho. Na caixa de diálogo, insira um nome para seu fluxo de trabalho, como *ContosoPay-Customer-Support-Triage*, e selecione **Save**.

## Criar uma variável de matriz de tickets

1. No visualizador do fluxo de trabalho, selecione o ícone **+** (sinal de adição) para adicionar um novo nó.

1. No menu de ações do fluxo de trabalho, em **Data transformation**, selecione **Set variable** para adicionar um nó que inicializa uma matriz de tickets de suporte.

2. No editor do nó **Set variable**, insira um nome para uma nova variável, como *SupportTickets*.

    ![Captura de tela da criação de uma nova variável no nó Set variable.](../Media/node-new-variable.png)

    A nova variável deve aparecer como `Local.SupportTickets`.

3. No campo **To value**, insira a matriz a seguir, que contém tickets de suporte de exemplo:

    ```output
   [
    "A API retorna um erro 403 ao criar faturas, mas nossa chave de API não mudou.",
    "Existe uma maneira de exportar todas as faturas como CSV?",
    "Fui cobrado duas vezes pela mesma fatura na última sexta-feira e meu cliente também está vendo dois recibos. Alguém pode corrigir isso?"]
    ```

4. Selecione **Done** para salvar o nó.

## Adicionar um loop for-each para processar os tickets

1. Selecione o ícone **+** (sinal de adição) abaixo de **Set variable** e crie um nó **For each** para processar cada ticket de suporte na matriz.

1. No editor do nó **For each**, defina o campo **Select the items to loop for each** como a variável criada anteriormente: `Local.SupportTickets`.

1. No campo **Loop Value Variable**, crie uma nova variável chamada `CurrentTicket`.

1. Selecione **Done** para salvar o nó.

## Invocar um agente para classificar o ticket

1. Selecione o ícone **+** (sinal de adição) dentro do nó **For each** para adicionar um novo nó que classifica o ticket de suporte atual.

2. No menu de ações do fluxo de trabalho, em **Invoke**, selecione **Agent** para adicionar um nó de agente.

3. No editor do nó **Agent**, em **Select an agent**, selecione **Create new agent**.

4. Insira um nome de agente, como *Triage-Agent*, e selecione **Create**.

### Configurar as definições do agente

1. No editor, em **Details**, selecione o botão **Parameters** próximo ao nome do modelo.

    ![Captura de tela do botão Parameters no editor do agente.](../Media/agent-parameters.png)

2. No painel **Parameters**, ao lado de **Text format**, selecione **JSON Schema**.

3. No painel **Add response format**, insira a definição a seguir e selecione **Save**:

    ```json
   {
   "name": "category_response",
   "schema": {
       "type": "object",
       "properties": {
           "customer_issue": {
               "type": "string"
           },
           "category": {
               "type": "string"
           },
           "confidence": {
               "type": "number"
           }
       },
       "additionalProperties": false,
       "required": [
           "customer_issue",
           "category",
           "confidence"
       ]
   },
   "strict": true
   }
    ```

4. No painel Agent Details, defina o campo **Instructions** com o seguinte prompt:

    ```output
   Classifique a descrição do problema do usuário em exatamente UMA categoria da lista abaixo. Forneça uma pontuação de confiança de 0 a 1.

   Billing
   - Cobranças, reembolsos, pagamentos duplicados
   - Pagamentos ausentes ou incorretos
   - Preços de assinaturas ou cobranças de faturas

   Technical
   - Erros de API, integrações, webhooks
   - Bugs da plataforma ou comportamento inesperado

   General
   - Perguntas sobre como fazer algo
   - Disponibilidade de recursos
   - Exportações de dados, relatórios ou navegação na interface do usuário

   Regras importantes
   - Perguntas sobre exportar, visualizar ou baixar faturas são General, não Billing
   - Billing se aplica SOMENTE quando o dinheiro foi cobrado, reembolsado ou pago incorretamente
    ```

5. Selecione **Node settings** para configurar a entrada e a saída do agente.

6. Defina o campo **Input message** como a variável `Local.CurrentTicket`.

7. Em **Save agent output message as**, crie uma nova variável chamada `TriageOutputText`.

8. Em **Save the output json_object as**, crie uma nova variável chamada `TriageOutputJson`.

9. Selecione **Done** para salvar o nó.

## Tratar classificações com baixa confiança

1. Selecione o ícone **+** (sinal de adição) abaixo do nó **Invoke agent** para adicionar um novo nó que trata classificações com baixa confiança.

1. No menu de ações do fluxo de trabalho, em **Flow**, selecione **If/Else** para adicionar um nó de lógica condicional.

1. No editor do nó **If/Else**, selecione o botão **Add a path** para criar a condição do ramo if e, em seguida, selecione o ícone de lápis para editar a condição.

1. Defina o campo **Condition** com a expressão a seguir para verificar se a pontuação de confiança é maior que 0.6:

    ```output
   Local.TriageOutputJson.confidence > 0.6
    ```

1. Selecione **Done** para salvar o nó.

## Recomendar informações adicionais para tickets com baixa confiança

1. No visualizador, no ramo **Else** do nó de condição **If/Else**, selecione o ícone **+** (sinal de adição) para adicionar um novo nó que recomenda informações adicionais para tickets com baixa confiança.

1. No menu de ações do fluxo de trabalho, em **Basics**, selecione **Deliver a message** para adicionar uma atividade de envio de mensagem.

1. No editor do nó **Deliver a message**, defina o campo **Message to send** com a resposta a seguir:

    ```output
   A classificação do ticket de suporte tem baixa confiança. Solicitando mais detalhes sobre o problema: "{Local.CurrentTicket}"
    ```

1. Selecione **Done** para salvar o nó.

## Encaminhar o ticket com base na categoria

Nesta seção, você adicionará lógica condicional para encaminhar o ticket com base na categoria classificada, caso a pontuação de confiança seja suficientemente alta.

1. No visualizador, no ramo **If** do nó **If/Else condition**, selecione o ícone **+** (sinal de adição) para adicionar um novo nó que encaminha o ticket com base na categoria.

1. No menu de ações do fluxo de trabalho, em **Flow**, selecione **If/Else** para adicionar outro nó de lógica condicional.

1. No editor do nó **If/Else**, selecione o botão **Add a path** para criar a condição do ramo if e, em seguida, selecione o ícone de lápis para editar a condição.

1. Defina a **If Condition** com a expressão a seguir para verificar se a categoria do ticket é "Billing":

    ```output
   Local.TriageOutputJson.category = "Billing"
    ```

1. Selecione o ícone **+** (sinal de adição) no ramo **If** do nó **If/Else** para adicionar um novo nó que redige uma resposta para tickets que não sejam de Billing.

1. No menu de ações do fluxo de trabalho, em **Basics**, selecione **Deliver a message** para adicionar uma atividade de envio de mensagem.

1. No editor do nó **Deliver a message**, defina **Message to send** com a resposta a seguir:

    ```output
   Encaminhe o problema de Billing para a equipe de suporte humano.
    ```

1. Selecione **Done** para salvar o nó.

## Gerar uma resposta recomendada

1. No visualizador, selecione o ícone **+** (sinal de adição) abaixo do ramo **Else** do segundo nó **If/Else** para adicionar um novo nó que redige uma resposta para tickets que não sejam de Billing.

2. No menu de ações do fluxo de trabalho, em **Invoke**, selecione **Agent** para adicionar um nó de agente.

3. No editor do nó **Agent**, selecione **Create new agent**.

4. Insira um nome de agente, como *Resolution-Agent*, e selecione **Create**.

5. No editor do agente, defina o campo **Instructions** com o seguinte prompt:

    ```output
   Você é um assistente de resolução de suporte ao cliente da ContosoPay, uma plataforma B2B de pagamentos e faturamento.

   Sua tarefa é redigir uma resposta de suporte clara, profissional e amigável com base na categoria do problema e na mensagem do cliente.

   Diretrizes:
   Se a categoria do problema for Technical:
   Sugira de 1 a 2 etapas comuns de solução de problemas em alto nível.

   Evite solicitar logs, credenciais ou dados confidenciais.

   Não dê a entender que a culpa é do cliente.
   Se a categoria do problema for General:
   Forneça uma explicação ou orientação concisa e útil.
   Mantenha a resposta com menos de 5 frases.

   Tom:
   Profissional, calmo e prestativo
   Claro e conciso
   Sem emojis

   Saída:
   Retorne somente o texto da resposta redigida.
   Não inclua raciocínio interno ou análise.
    ```

6. Selecione **Node settings** para configurar a entrada e a saída do agente.

7. Defina o campo **Input message** como a variável `Local.TriageOutputText`.

8. Em **Save agent output message as**, crie uma nova variável chamada `ResolutionOutputText`.

9. Selecione **Done** para salvar o nó.

## Visualizar o fluxo de trabalho

1. Selecione o botão **Save** para salvar todas as alterações no fluxo de trabalho.

1. Selecione o botão **Preview** para iniciar o fluxo de trabalho.

1. Na janela de chat exibida, insira algum texto para disparar o fluxo de trabalho, como `Iniciar o processamento dos tickets de suporte.`

1. Observe o fluxo de trabalho enquanto ele processa cada ticket de suporte em sequência. Revise as mensagens geradas pelo fluxo de trabalho na janela de chat.

    Você deverá ver uma saída indicando que os problemas de Billing estão sendo escalonados, enquanto os problemas Technical e General recebem respostas redigidas. Por exemplo:

    ```output
   Ticket atual:
   A API retorna um erro 403 ao criar faturas, mas nossa chave de API não mudou.


   O Copilot disse:
   Obrigado por entrar em contato sobre o erro 403 ao criar faturas. Esse erro normalmente indica um problema de permissões ou acesso.
   Verifique se sua chave de API tem as permissões necessárias para a criação de faturas e se sua solicitação está sendo enviada ao endpoint correto.
   Se o problema persistir, tente gerar novamente sua chave de API e atualizá-la na integração para verificar se isso resolve o problema.
    ```

## Usar seu fluxo de trabalho em um aplicativo cliente

Agora que você criou e testou seu fluxo de trabalho no portal do Foundry, também pode invocá-lo a partir do seu próprio código usando o SDK do Azure AI Projects. Isso permite integrar o fluxo de trabalho aos seus aplicativos ou automatizar sua execução.

### Clonar o repositório de código inicial

Neste exercício, você usará um código inicial que ajudará a se conectar ao seu projeto do Foundry e invocar um fluxo de trabalho.

1. No VS Code, abra a Paleta de Comandos (**Ctrl+Shift+P** ou **View > Command Palette**).

1. Digite **Git: Clone** e selecione-o na lista.

1. Insira a URL do repositório:

    ```
   https://github.com/MicrosoftLearning/mslearn-ai-agents.git
    ```

1. Escolha um local em seu computador local para clonar o repositório.

1. Quando solicitado, selecione **Open** para abrir o repositório clonado no VS Code.

1. Quando o repositório for aberto, selecione **File > Open Folder** e navegue até `mslearn-ai-agents/Labfiles/06-build-workflow-ms-foundry`, depois escolha **Select Folder**.

1. No painel Explorer, expanda a pasta **Python** para visualizar os arquivos de código deste exercício.

### Configurar o aplicativo

1. No navegador, retorne ao visualizador do fluxo de trabalho no portal do Foundry.

2. Selecione **Code** no canto superior direito do visualizador. Em seguida, selecione **.env variables** para visualizar as variáveis de ambiente necessárias para se conectar ao seu projeto do Foundry a partir do código.

3. Copie o valor da variável **AZURE_EXISTING_AIPROJECT_ENDPOINT**, que é a URL do endpoint do seu projeto do Foundry. Você precisará desse valor para se conectar ao seu projeto no VS Code.

4. No VS Code, clique com o botão direito do mouse no arquivo **requirements.txt** e selecione **Open in Integrated Terminal**.

5. No terminal, insira o comando a seguir para instalar os pacotes Python necessários em um ambiente virtual:

    ```
   python -m venv labenv
   .\labenv\Scripts\Activate.ps1
   pip install -r requirements.txt
    ```

6. Abra o arquivo **.env**, substitua o espaço reservado **your_project_endpoint** pelo endpoint do seu projeto (copiado da guia de código do visualizador do fluxo de trabalho). Use **Ctrl+S** para salvar o arquivo depois de fazer essas alterações.

### Invocar o fluxo de trabalho a partir do código

Agora você está pronto para criar um projeto que invoca um fluxo de trabalho. Vamos começar!

1. Abra o arquivo **workflow.py** no editor de código.

1. Revise o código no arquivo, observando que ele contém strings para o nome e as instruções de cada agente.

1. Localize o comentário **Add references** e adicione o código a seguir para importar as classes necessárias:

    ```python
   # Adicione as referências
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
    ```

2. Localize o comentário **Connect to the agents client** e adicione o código a seguir para criar um AgentsClient conectado ao seu projeto:

    ```python
   # Conecte-se ao cliente do projeto de IA
   with (
       DefaultAzureCredential() as credential,
       AIProjectClient(endpoint=endpoint, credential=credential) as project_client,
       project_client.get_openai_client() as openai_client,
   ):
    ```

    Agora você adicionará código que usa o AgentsClient para criar vários agentes, cada um com uma função específica no processamento de um ticket de suporte.

    > **Dica**: Ao adicionar o código subsequente, mantenha o nível correto de indentação.

3. Localize o comentário **Specify the workflow** e adicione o código a seguir:

    ```python
    # Especifique o fluxo de trabalho
    workflow = {
        "name": "ContosoPay-Customer-Support-Triage"
    }
    ```

    Use o nome e a versão do fluxo de trabalho criado no portal do Foundry.

4. Localize o comentário **Create a conversation and run the workflow** e adicione o código a seguir para criar uma conversa e invocar seu fluxo de trabalho:

    ```python
   # Crie uma conversa e execute o fluxo de trabalho
   conversation = openai_client.conversations.create()
   print(f"Conversa criada (id: {conversation.id})")

   stream = openai_client.responses.create(
       conversation=conversation.id,
       extra_body={"agent_reference" : {"name" : workflow["name"], "type": "agent_reference"}},
       input="Iniciar",
       stream=True,
   )
    ```

    Esse código transmite a saída da execução do fluxo de trabalho para o console, permitindo ver o fluxo de mensagens enquanto o fluxo de trabalho processa cada ticket.

5. Localize o comentário **Process events from the workflow run** e adicione o código a seguir para processar a saída transmitida e imprimir mensagens no console:

    ```python
   # Processe os eventos da execução do fluxo de trabalho
   for event in stream:
       if (event.type == "response.completed"):
           print("\nResposta concluída:")
           response = openai_client.responses.retrieve(event.response.id)
           print_workflow_output(response.output_text)
    ```

    Esse código aguarda a conclusão da resposta do fluxo de trabalho e, em seguida, recupera e imprime o texto de saída final no console. A função `print_workflow_output` é uma função auxiliar definida no arquivo de código que formata a saída para facilitar a leitura.

6. Localize o comentário **Clean up resources** e insira o código a seguir para excluir a conversa quando ela não for mais necessária:

    ```python
   # Limpe os recursos
   openai_client.conversations.delete(conversation_id=conversation.id)
   print("\nConversa excluída")
    ```

7. Use o comando **CTRL+S** para salvar suas alterações no arquivo de código.

## Testar o aplicativo cliente

Agora você está pronto para executar seu código e observar a colaboração entre seus agentes de IA.

1. No terminal integrado, execute os comandos a seguir:
    ```
   az login
    ```

    ```
   python workflow.py
    ```

1. Aguarde um momento para que o fluxo de trabalho processe os tickets. Enquanto o fluxo de trabalho é executado, você deverá ver no console uma saída indicando o progresso, incluindo mensagens geradas pelos agentes e atualizações de status para cada ação no fluxo de trabalho.

1. Quando o fluxo de trabalho for concluído, você deverá ver uma saída semelhante à seguinte:

    ```output
   Resposta concluída:
   Ticket atual:
   A API retorna um erro 403 ao criar faturas, mas nossa chave de API não mudou.{"customer_issue":"A API retorna um erro 403 ao criar faturas, a chave de API não mudou.","category":"Technical","confidence":1}Obrigado por entrar em contato sobre o erro 403 ao criar faturas com a API. Esse erro normalmente está relacionado a problemas de permissões. Verifique se sua chave de API tem as permissões necessárias para a criação de faturas e se a URL do endpoint está correta. Se o problema persistir, tente gerar novamente a chave de API e atualizá-la no aplicativo.
   ...
    ```

    Na saída, você pode ver como o fluxo de trabalho conclui cada ticket de suporte, incluindo a classificação de cada ticket e a resposta recomendada ou o escalonamento. Muito bem!

2. Quando terminar, insira `deactivate` no terminal para sair do ambiente virtual do Python.

## Limpar

Se você terminou de explorar os fluxos de trabalho no Microsoft Foundry, exclua os recursos criados neste exercício para evitar custos desnecessários do Azure.

1. Navegue até o [portal do Azure](https://portal.azure.com) em `https://portal.azure.com` e visualize o conteúdo do grupo de recursos no qual seu projeto do Foundry foi implantado.

1. Na barra de ferramentas, selecione **Delete resource group**.
1. Insira o nome do grupo de recursos e confirme que deseja excluí-lo.
