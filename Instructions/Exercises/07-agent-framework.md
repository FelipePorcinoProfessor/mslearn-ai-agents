---
lab:
    title: 'Desenvolva um agente de IA do Azure com o SDK do Microsoft Agent Framework'
    description: 'Aprenda a usar o SDK do Microsoft Agent Framework para criar e usar um agente de chat de IA do Azure.'
    level: 300
    duration: 30
    islab: true
    status: 'released'
    layout: default
---

# Desenvolva um agente de chat de IA do Azure com o SDK do Microsoft Agent Framework

Neste exercício, você usará o Azure AI Agent Service e o Microsoft Agent Framework para criar um agente de IA que processa solicitações de despesas.

Este exercício deve levar aproximadamente **30** minutos para ser concluído.

> **Observação**: Algumas das tecnologias usadas neste exercício estão em versão prévia ou em desenvolvimento ativo. Você pode observar comportamentos inesperados, avisos ou erros.

## Pré-requisitos

Antes de iniciar este exercício, verifique se você tem:

- O [Visual Studio Code](https://code.visualstudio.com/) instalado no computador local
- Uma [assinatura do Azure](https://azure.microsoft.com/free/) ativa
- O [Python 3.13](https://www.python.org/downloads/) instalado
- O [Git](https://git-scm.com/downloads) instalado no computador local

> \* O Python 3.14 ainda não é compatível: algumas dependências não têm um build para a versão 3.14. Este laboratório foi testado com o Python 3.13.12.

## Criar um projeto do Foundry com a extensão Foundry Toolkit para VS Code

Como desenvolvedor, talvez você passe algum tempo trabalhando no portal do Foundry; mas também é provável que passe bastante tempo no Visual Studio Code. A extensão Foundry Toolkit oferece uma maneira conveniente de trabalhar com os recursos do projeto do Foundry sem sair do ambiente de desenvolvimento.

1. Abra o Visual Studio Code.

2. Selecione **Extensions** no painel esquerdo (ou pressione **Ctrl+Shift+X**).

3. Pesquise no marketplace de extensões a extensão `Foundry Toolkit` da Microsoft e selecione **Install**.

    > **Observação**: Atualmente, a extensão está listada como **Foundry Toolkit**, mas alguns rótulos e comandos do VS Code, ou capturas de tela mais antigas, ainda podem fazer referência a **AI Toolkit**. Neste laboratório, considere que esses nomes se referem à mesma experiência de extensão.

4. Depois de instalar a extensão, selecione o ícone dela na barra lateral para abrir a exibição do Foundry Toolkit.

    Será solicitado que você entre na sua conta do Azure, caso ainda não tenha feito isso.

5. Selecione **Create Project** em **Microsoft Foundry Resources**.

    Se um projeto padrão já estiver ativo, o nome do projeto aparecerá em **My Resources**. Você pode criar um novo projeto clicando com o botão direito do mouse no projeto ativo e selecionando **Switch Default Project**.

6. Selecione sua assinatura do Azure e o grupo de recursos e, em seguida, insira um nome para o projeto do Foundry a fim de criar um novo projeto para este exercício.

    Quando a implantação for concluída, o projeto deverá aparecer no painel do Foundry Toolkit como o projeto padrão.

## Implantar um modelo

No centro de qualquer projeto de IA generativa há pelo menos um modelo de IA generativa. Nesta tarefa, você implantará um modelo do Catálogo de Modelos para usar com seu agente.

1. Quando o pop-up "Project deployed successfully" aparecer, selecione o botão **Deploy a model**. Isso abrirá o Catálogo de Modelos.

   > **Dica**: Você também pode acessar o Catálogo de Modelos selecionando o ícone **+** ao lado de **Models** na seção Resources ou pressionando **F1** e executando o comando **Foundry Toolkit: Show model catalog**.

1. No Catálogo de Modelos, localize o modelo **gpt-5** (você pode usar a barra de pesquisa para encontrá-lo rapidamente).

1. Selecione **Deploy** ao lado do modelo gpt-5.

1. Defina as configurações de implantação:
   - **Deployment name**: insira um nome como "gpt-5"
   - **Deployment type**: selecione **Global Standard** (ou **Standard**, se Global Standard não estiver disponível)
   - **Model version**: mantenha o padrão
   - **Tokens per minute**: mantenha o padrão

1. Selecione **Deploy to Microsoft Foundry** no canto inferior esquerdo.

1. Aguarde a conclusão da implantação. O modelo implantado aparecerá na seção **Models** da exibição Resources.

1. Clique com o botão direito do mouse no nome da implantação do projeto e selecione **Copy Project Endpoint**. Você precisará dessa URL para conectar seu agente ao projeto do Foundry nas próximas etapas.

    ![Captura de tela da cópia do ponto de extremidade do projeto na extensão Foundry Toolkit para VS Code.](../Media/vs-code-endpoint.png)

## Clonar o repositório do código inicial

Neste exercício, você usará um código inicial que ajudará a conectar-se ao projeto do Foundry e a criar um agente capaz de processar dados de despesas. Você clonará esse código de um repositório do GitHub.

1. No VS Code, abra a Paleta de Comandos (**Ctrl+Shift+P** ou **View > Command Palette**).

1. Digite **Git: Clone** e selecione-o na lista.

1. Insira a URL do repositório:

    ```
   https://github.com/MicrosoftLearning/mslearn-ai-agents.git
    ```

1. Escolha um local no computador local para clonar o repositório.

1. Quando solicitado, selecione **Open** para abrir o repositório clonado no VS Code.

1. Depois que o repositório for aberto, selecione **File > Open Folder** e navegue até `mslearn-ai-agents/Labfiles/07-agent-framework`; em seguida, escolha **Select Folder**.

1. No painel Explorer, expanda a pasta **Python** para exibir os arquivos de código deste exercício.

1. Clique com o botão direito do mouse no arquivo **requirements.txt** e selecione **Open in Integrated Terminal**.

1. No terminal, insira o comando a seguir para instalar os pacotes Python necessários em um ambiente virtual:

    ```
   python -m venv labenv
   .\labenv\Scripts\Activate.ps1
   pip install -r requirements.txt
    ```

1. Abra o arquivo **.env**, substitua o espaço reservado **your_project_endpoint** pelo ponto de extremidade do seu projeto (copiado do recurso de implantação do projeto na extensão Foundry Toolkit) e verifique se a variável MODEL_DEPLOYMENT_NAME está definida como o nome da implantação do seu modelo. Use **Ctrl+S** para salvar o arquivo depois de fazer essas alterações.

Agora você está pronto para criar um agente de IA que usa uma ferramenta personalizada para processar dados de despesas.

## Criar um agente com uma ferramenta personalizada

> **Dica**: Ao adicionar código, mantenha a indentação correta. Use os comentários existentes como guia, inserindo o novo código no mesmo nível de indentação.

1. Abra o arquivo **agent-framework.py** no editor de código.

1. Examine o código no arquivo. Ele contém:
    - Algumas instruções **import** para adicionar referências a namespaces usados com frequência
    - Uma função *main* que carrega um arquivo contendo dados de despesas, solicita instruções ao usuário e, em seguida, chama...
    - Uma função **process_expenses_data**, na qual o código para criar e usar seu agente deve ser adicionado

1. Na parte superior do arquivo, depois da instrução **import** existente, localize o comentário **Adicionar referências** e adicione o código a seguir para fazer referência aos namespaces nas bibliotecas necessárias para implementar seu agente:

    ```python
   # Adicionar referências
   from agent_framework import tool, Agent
   from agent_framework.foundry import FoundryChatClient
   from azure.identity import AzureCliCredential
   from pydantic import Field
   from typing import Annotated
    ```

1. Próximo à parte inferior do arquivo, localize o comentário **Criar uma função de ferramenta para a funcionalidade de email** e adicione o código a seguir para definir uma função que seu agente usará para enviar emails (as ferramentas são uma forma de adicionar funcionalidade personalizada aos agentes):

    ```python
   # Criar uma função de ferramenta para a funcionalidade de email
   @tool(approval_mode="never_require")
   def submit_claim(
       to: Annotated[str, Field(description="Para quem enviar o email")],
       subject: Annotated[str, Field(description="O assunto do email.")],
       body: Annotated[str, Field(description="O corpo de texto do email.")]):
           print("\nPara:", to)
           print("Assunto:", subject)
           print(body, "\n")
    ```

    > **Observação**: A função *simula* o envio de um email imprimindo-o no console. Em um aplicativo real, você usaria um serviço SMTP ou algo semelhante para enviar o email de fato!

1. Logo acima do código **send_email**, na função **process_expenses_data**, localize o comentário **Criar um cliente de chat do Foundry** e adicione o código a seguir:

    (Lembre-se de manter o nível de indentação.)

    ```python
   # Criar um cliente de chat do Foundry
   client = FoundryChatClient(
       project_endpoint=os.getenv("PROJECT_ENDPOINT"),
       model=os.getenv("MODEL_DEPLOYMENT_NAME"),
       credential=AzureCliCredential()
   )
    ```

    Observe que o objeto **AzureCliCredential** permitirá que seu código se autentique na sua conta do Azure. Esse cliente será usado para interagir com os serviços de agente do Foundry.

2. Localize o comentário **Inicializar um agente com a ferramenta e as instruções** e adicione o código a seguir:

    (Lembre-se de manter o nível de indentação.)

    ```python
   # Inicializar um agente com a ferramenta e as instruções
   async with (
       Agent(
           client=client,
           name="ExpenseClaimAgent",
           instructions="""Você é um assistente de IA para o envio de solicitações de despesas.
                       A pedido do usuário, crie uma solicitação de despesas e use a função de plug-in para enviar um email para expenses@contoso.com com o assunto 'Solicitação de despesas' e um corpo que contenha as despesas discriminadas e um total.
                       Em seguida, confirme ao usuário que você fez isso. Não peça mais informações ao usuário; apenas use os dados fornecidos para criar o email.""",
           tools=[submit_claim],
       ) as agent,
   ):
    ```
    Nesse código, o objeto **Agent** é inicializado com o cliente, as instruções para o agente e a função de ferramenta definida para enviar emails.

1. Localize o comentário **Usar o agente para processar os dados de despesas** e adicione o código a seguir para criar uma thread na qual o agente será executado e, em seguida, invocá-lo com uma mensagem de chat.

    (Lembre-se de manter o nível de indentação):

    ```python
   # Usar o agente para processar os dados de despesas
   try:
       # Adicionar o prompt de entrada a uma lista de mensagens a serem enviadas
       prompt_messages = [f"{prompt}: {expenses_data}"]
       # Invocar o agente para a thread especificada com as mensagens
       response = await agent.run(prompt_messages)
       # Exibir a resposta
       print(f"\n# Agente:\n{response}")
   except Exception as e:
       # Algo deu errado
       print (e)
    ```

1. Examine o código concluído do agente, usando os comentários para entender o que cada bloco de código faz, e salve as alterações do código (**CTRL+S**).

## Testar o aplicativo

1. No terminal integrado, insira os comandos a seguir para executar o aplicativo:

    ```
   az login
    ```

    ```
   python agent-framework.py
    ```

    O comando `az login` permite que o AzureCliCredential se autentique na sua conta do Azure.

1. Quando perguntado o que fazer com os dados de despesas, insira o seguinte prompt:

    ```
   Enviar uma solicitação de despesas
    ```

1. Quando o aplicativo terminar, examine a saída. O agente deverá ter composto um email para uma solicitação de despesas com base nos dados fornecidos.

    > **Dica**: Se o aplicativo falhar porque o limite de taxa foi excedido, aguarde alguns segundos e tente novamente. Se não houver cota suficiente disponível na sua assinatura, talvez o modelo não consiga responder.

1. Quando terminar, insira `deactivate` no terminal para sair do ambiente virtual do Python.

## Limpar

Se você terminou de explorar o Azure AI Agent Service, deverá excluir os recursos criados neste exercício para evitar custos desnecessários do Azure.

### Excluir seu modelo

1. No VS Code, atualize a exibição **Azure Resources**.

1. Expanda a subseção **Models**.

1. Clique com o botão direito do mouse no modelo implantado e selecione **Delete**.

### Excluir o grupo de recursos

1. Abra o [portal do Azure](https://portal.azure.com).

1. Navegue até o grupo de recursos que contém seus recursos do Microsoft Foundry.

1. Selecione **Delete resource group** e confirme a exclusão.
