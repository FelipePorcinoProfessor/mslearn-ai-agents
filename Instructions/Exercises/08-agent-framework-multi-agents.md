---
lab:
    title: 'Desenvolva uma solução multiagente com o Microsoft Agent Framework'
    description: 'Aprenda a configurar vários agentes para colaborar usando o SDK do Microsoft Agent Framework'
    level: 300
    duration: 30
    islab: true
    status: 'released'
---

# Desenvolva uma solução multiagente com o Microsoft Agent Framework

Neste exercício, você praticará o uso do padrão de orquestração sequencial no SDK do Microsoft Agent Framework. Você criará um pipeline simples de três agentes que trabalham juntos para processar comentários de clientes e sugerir os próximos passos. Você criará os seguintes agentes:

- O agente Summarizer condensará os comentários brutos em uma frase curta e neutra.
- O agente Classifier categorizará os comentários como Positivo, Negativo ou uma solicitação de recurso.
- Por fim, o agente Recommended Action recomendará uma etapa de acompanhamento apropriada.

Você aprenderá a usar o SDK do Microsoft Agent Framework para dividir um problema, encaminhá-lo aos agentes corretos e produzir resultados acionáveis. Vamos começar!

Este exercício deve levar aproximadamente **30** minutos para ser concluído.

> **Observação**: algumas das tecnologias usadas neste exercício estão em versão prévia ou em desenvolvimento ativo. Você pode encontrar comportamentos inesperados, avisos ou erros.

## Pré-requisitos

Antes de iniciar este exercício, verifique se você tem:

- o [Visual Studio Code](https://code.visualstudio.com/) instalado no computador local;
- uma [assinatura do Azure](https://azure.microsoft.com/free/) ativa;
- o [Python 3.13](https://www.python.org/downloads/) instalado;
- o [Git](https://git-scm.com/downloads) instalado no computador local.

> \* O Python 3.14 ainda não é compatível: algumas dependências não têm um build para a versão 3.14. Este laboratório foi testado com o Python 3.13.12.

## Criar um projeto do Foundry com a extensão Foundry Toolkit para VS Code

Como desenvolvedor, você pode passar algum tempo trabalhando no portal do Foundry; mas também é provável que passe bastante tempo no Visual Studio Code. A extensão Foundry Toolkit para VS Code oferece uma maneira conveniente de trabalhar com os recursos do projeto do Foundry sem sair do ambiente de desenvolvimento.

1. Abra o Visual Studio Code.

2. Selecione **Extensions** no painel esquerdo (ou pressione **Ctrl+Shift+X**).

3. Pesquise no marketplace de extensões a extensão `Foundry Toolkit`, da Microsoft, e selecione **Install**.

    > **Observação**: a extensão está listada atualmente como **Foundry Toolkit**, mas alguns rótulos e comandos do VS Code, ou capturas de tela mais antigas, ainda podem fazer referência ao **AI Toolkit**. Neste laboratório, considere que esses nomes se referem à mesma experiência de extensão.

4. Depois de instalar a extensão, selecione o ícone dela na barra lateral para abrir a exibição do Foundry Toolkit.

    Será solicitado que você entre na sua conta do Azure, caso ainda não tenha feito isso.

5. Selecione **Create Project** em **Microsoft Foundry Resources**.

    Se já houver um projeto padrão ativo, o nome do projeto aparecerá em **My Resources**. Você pode criar um novo projeto clicando com o botão direito do mouse no projeto ativo e selecionando **Switch Default Project in Azure Extension**.

6. Selecione sua assinatura do Azure e o grupo de recursos e insira um nome para o projeto do Foundry a fim de criar um novo projeto para este exercício.

    Quando a implantação for concluída, o projeto deverá aparecer no painel do Foundry Toolkit como o projeto padrão.

## Implantar um modelo

No centro de qualquer projeto de IA generativa há pelo menos um modelo de IA generativa. Nesta tarefa, você implantará um modelo do Model Catalog para usar com seu agente.

1. Quando o pop-up "Project deployed successfully" aparecer, selecione o botão **Deploy a new model**. Isso abrirá o Model Catalog.

   > **Dica**: você também pode acessar o Model Catalog selecionando o ícone **+** ao lado de **Models**, na seção Resources, ou pressionando **F1** e executando o comando **Foundry Toolkit: Show model catalog**.

1. No Model Catalog, localize o modelo **gpt-5** (você pode usar a barra de pesquisa para encontrá-lo rapidamente).

1. Selecione **Deploy** ao lado do modelo gpt-5.

1. Configure as definições de implantação:
   - **Deployment name**: insira um nome como "gpt-5";
   - **Deployment type**: selecione **Global Standard** (ou **Standard**, se Global Standard não estiver disponível);
   - **Model version**: mantenha o padrão;
   - **Tokens per minute**: mantenha o padrão.

1. Selecione **Deploy to Microsoft Foundry** no canto inferior esquerdo.

1. Aguarde a conclusão da implantação. O modelo implantado aparecerá na seção **Models** da exibição Resources.

1. Clique com o botão direito do mouse no nome da implantação do projeto e selecione **Copy Project Endpoint**. Você precisará dessa URL para conectar seu agente ao projeto do Foundry nas próximas etapas.

    ![Captura de tela da cópia do endpoint do projeto na extensão Foundry Toolkit do VS Code.](../Media/vs-code-endpoint.png)

## Clonar o repositório do código inicial

Neste exercício, você usará um código inicial que ajudará a conectar-se ao projeto do Foundry e criar uma solução multiagente capaz de processar comentários de clientes. Você clonará esse código de um repositório do GitHub.

1. No VS Code, abra a Command Palette (**Ctrl+Shift+P** ou **View > Command Palette**).

1. Digite **Git: Clone** e selecione-o na lista.

1. Insira a URL do repositório:

    ```
   https://github.com/MicrosoftLearning/mslearn-ai-agents.git
    ```

1. Escolha um local no computador local para clonar o repositório.

1. Quando solicitado, selecione **Open** para abrir o repositório clonado no VS Code.

1. Depois que o repositório for aberto, selecione **File > Open Folder**, navegue até `mslearn-ai-agents/Labfiles/08-agent-orchestration` e escolha **Select Folder**.

1. No painel Explorer, expanda a pasta **Python** para exibir os arquivos de código deste exercício.

1. Clique com o botão direito do mouse no arquivo **requirements.txt** e selecione **Open in Integrated Terminal**.

1. No terminal, insira o comando a seguir para instalar os pacotes Python necessários em um ambiente virtual:

    ```
   python -m venv labenv
   .\labenv\Scripts\Activate.ps1
   pip install -r requirements.txt
    ```

1. Abra o arquivo **.env**, substitua o espaço reservado **your_project_endpoint** pelo endpoint do seu projeto (copiado do recurso de implantação do projeto na extensão Foundry Toolkit) e verifique se a variável MODEL_DEPLOYMENT_NAME está definida como o nome da implantação do modelo. Use **Ctrl+S** para salvar o arquivo depois de fazer essas alterações.

## Criar agentes de IA

Agora você está pronto para criar os agentes da sua solução multiagente! Vamos começar!

1. Abra o arquivo **agents.py** no editor de código.

1. Na parte superior do arquivo, sob o comentário **Adicionar referências**, adicione o código a seguir para referenciar os namespaces nas bibliotecas necessárias para implementar seu agente:

    ```python
   # Adicionar referências
   from agent_framework import Message
   from agent_framework.foundry import FoundryChatClient
   from agent_framework.orchestrations import SequentialBuilder
   from azure.identity import AzureCliCredential
    ```

1. Na função **main**, reserve um momento para revisar as instruções dos agentes. Essas instruções definem o comportamento de cada agente na orquestração.

1. Adicione o código a seguir sob o comentário **Criar o cliente de chat**:

    ```python
   # Criar o cliente de chat
   credential = AzureCliCredential()
   chat_client = FoundryChatClient(
       credential=credential,
       project_endpoint=os.getenv("AZURE_AI_PROJECT_ENDPOINT"),
       model=os.getenv("AZURE_AI_MODEL_DEPLOYMENT_NAME"),
   )
    ```

    Observe que o objeto **AzureCliCredential** permitirá que seu código se autentique na sua conta do Azure. O objeto **FoundryChatClient** conecta-se ao projeto do Foundry usando o endpoint e o nome da implantação do modelo da configuração do .env.

1. Adicione o código a seguir sob o comentário **Criar agentes**:

    (Não se esqueça de manter o nível de indentação.)

    ```python
   # Criar agentes
   summarizer_agent = chat_client.as_agent(
       name="summarizer",
       instructions=summarizer_instructions,
   )

   classifier_agent = chat_client.as_agent(
       name="classifier",
       instructions=classifier_instructions,
   )

   action_agent = chat_client.as_agent(
       name="action",
       instructions=action_instructions,
   )
    ```

## Criar uma orquestração sequencial

1. Na função **main**, localize o comentário **Inicializar os comentários atuais** e adicione o código a seguir:

    (Não se esqueça de manter o nível de indentação.)

    ```python
   # Inicializar os comentários atuais
   feedback="""
   Uso o dashboard todos os dias para monitorar métricas, e, no geral, ele funciona bem.
   Mas, quando trabalho até tarde da noite, a tela brilhante agride bastante meus olhos.
   Se você adicionasse uma opção de modo escuro, a experiência seria muito mais confortável.
   """
    ```

1. Sob o comentário **Criar uma orquestração sequencial**, adicione o código a seguir para definir uma orquestração sequencial com os agentes que você definiu:

    ```python
   # Criar orquestração sequencial
   workflow = SequentialBuilder(
       participants=[summarizer_agent, classifier_agent, action_agent],
       output_from="all",
   ).build()
    ```

    Os agentes processarão os comentários na ordem em que forem adicionados à orquestração. O parâmetro `output_from="all"` garante que as saídas de todos os agentes sejam coletadas.

1. Adicione o código a seguir sob o comentário **Executar e coletar saídas**:

    ```python
   # Executar e coletar saídas
   result = await workflow.run(f"Comentários do cliente: {feedback}")
   outputs = result.get_outputs()
    ```

    Esse código executa a orquestração e coleta a saída de cada um dos agentes participantes.

1. Adicione o código a seguir sob o comentário **Exibir saídas**:

    ```python
   # Exibir saídas
   i = 1
   for response in outputs:
       for msg in cast(list[Message], response.messages):
           name = msg.author_name or ("assistant" if msg.role == "assistant" else "user")
           print(f"{'-' * 60}\n{i:02d} [{name}]\n{msg.text}")
           i += 1
    ```

    Esse código formata e exibe as mensagens das saídas do fluxo de trabalho coletadas da orquestração.

1. Use o comando **CTRL+S** para salvar suas alterações no arquivo de código.

## Testar o aplicativo

Agora você está pronto para executar seu código e observar seus agentes de IA colaborando.

1. No terminal integrado, insira os comandos a seguir para executar o aplicativo:

    ```
   az login
    ```

    ```
   python agents.py
    ```

1. Você deverá ver uma saída semelhante à seguinte:

    ```output
   Usuário solicita uma opção de modo escuro para um uso noturno mais confortável.
   Solicitação de recurso
   Registrar como solicitação de melhoria para adicionar o modo escuro e aumentar o conforto do usuário durante o uso noturno.
   ------------------------------------------------------------
   01 [summarizer]
   Usuário solicita uma opção de modo escuro para um uso noturno mais confortável.
   ------------------------------------------------------------
   02 [classifier]
   Solicitação de recurso
   ------------------------------------------------------------
   03 [action]
   Registrar como solicitação de melhoria para adicionar o modo escuro e aumentar o conforto do usuário durante o uso noturno.
    ```

1. Opcionalmente, você pode tentar executar o código usando entradas de comentários diferentes, como:

    ```output
   Entrei em contato com o suporte ao cliente ontem porque não conseguia acessar minha conta. O representante respondeu quase imediatamente, foi educado e profissional e resolveu o problema em poucos minutos. Sinceramente, foi uma das melhores experiências de suporte que já tive.
    ```

1. Quando terminar, insira `deactivate` no terminal para sair do ambiente virtual do Python.

## Limpar os recursos

Se você terminou de explorar o Azure AI Agent Service, deverá excluir os recursos criados neste exercício para evitar custos desnecessários do Azure.

### Excluir seu modelo

1. No VS Code, atualize a exibição **Azure Resources**.

1. Expanda a subseção **Models**.

1. Clique com o botão direito do mouse no modelo implantado e selecione **Delete**.

### Excluir o grupo de recursos

1. Abra o [portal do Azure](https://portal.azure.com).

1. Navegue até o grupo de recursos que contém seus recursos do Microsoft Foundry.

1. Selecione **Delete resource group** e confirme a exclusão.
