---
lab:
    title: 'Criar agentes de IA com o portal e o VS Code'
    description: 'Crie um agente de IA usando o portal do Microsoft Foundry e a extensão Foundry Toolkit para VS Code, com ferramentas integradas como pesquisa de arquivos e interpretador de código.'
    level: 300
    duration: 45
    islab: true
    status: 'released'
---

# Criar agentes de IA com o portal e o VS Code

Neste exercício, você criará uma solução completa de agente de IA usando o portal do Microsoft Foundry e a extensão Foundry Toolkit para VS Code. Primeiro, você criará um agente básico no portal com dados de fundamentação e ferramentas integradas; depois, interagirá programaticamente com ele usando o VS Code para utilizar recursos avançados, como o interpretador de código para análise de dados.

Este exercício leva aproximadamente **45** minutos.

> **Observação**: Algumas das tecnologias usadas neste exercício estão em versão prévia ou em desenvolvimento ativo. Você pode encontrar comportamentos, avisos ou erros inesperados.

## Pré-requisitos

Antes de iniciar este exercício, verifique se você tem:

- Uma [assinatura do Azure](https://azure.microsoft.com/free/) com permissões e cota suficientes para provisionar recursos do Azure AI
- O [Visual Studio Code](https://code.visualstudio.com/) instalado no computador local
- O [Python 3.13](https://www.python.org/downloads/) instalado
- O [Git](https://git-scm.com/downloads) instalado no computador local
- Familiaridade básica com os serviços do Azure AI e a programação em Python

> \* O Python 3.14 ainda não é compatível: algumas dependências não têm uma compilação para 3.14. Este laboratório foi testado com o Python 3.13.12.

## Criar um projeto do Microsoft Foundry

O Microsoft Foundry usa projetos para organizar modelos, recursos, dados e outros ativos usados para desenvolver uma solução de IA.

1. Em um navegador da Web, abra o [portal do Foundry](https://ai.azure.com) em `https://ai.azure.com` e entre usando suas credenciais do Azure. Feche todas as dicas ou painéis de início rápido que forem abertos na primeira vez que você entrar e, se necessário, use o logotipo do **Foundry** no canto superior esquerdo para navegar até a página inicial.

    > **Importante**: Para este laboratório, você está usando a experiência **New** do Foundry.

1. No banner superior, selecione **Start building** para experimentar a nova experiência do Microsoft Foundry.

1. Quando solicitado, crie um projeto **novo** e insira um nome válido para o projeto (por exemplo, `it-support-agent-project`).

1. Expanda **Advanced options** e especifique as seguintes configurações:
    - **Microsoft Foundry resource**: *Um nome válido para o recurso do Foundry*
    - **Region**: *Selecione uma disponível perto de você*\**
    - **Subscription**: *Sua assinatura do Azure*
    - **Resource group**: *Selecione seu grupo de recursos ou crie um novo*

    > \* Alguns recursos do Azure AI têm limitações de cotas de modelo por região. Se um limite de cota for excedido mais adiante no exercício, talvez seja necessário criar outro recurso em uma região diferente.

1. Selecione **Create** e aguarde a criação do projeto.

1. Quando o projeto for criado, uma caixa de diálogo de boas-vindas poderá aparecer. Selecione **Next** para ler a mensagem de boas-vindas e, em seguida, selecione **Create agent**.

    Você também pode selecionar **Start building** na página inicial e selecionar **Create agents** no menu suspenso.

1. Defina **Agent name** como `it-support-agent` e crie o agente.

O playground será aberto para o agente recém-criado. Você verá que um modelo implantado disponível já está selecionado para você.

## Configurar o agente com instruções e dados de fundamentação

Agora que você criou um agente, vamos configurá-lo com instruções e adicionar dados de fundamentação.

1. No playground do agente, defina **Instructions** como:

    ```prompt
   Você é um Agente de Suporte de TI da Contoso Corporation.
   Você ajuda os funcionários com problemas técnicos e dúvidas sobre as políticas de TI.

   Diretrizes:
   - Seja sempre profissional e prestativo
   - Use a documentação das políticas de TI para responder às perguntas com precisão
   - Se não souber a resposta, admita isso e sugira entrar em contato diretamente com o suporte de TI
   - Ao criar tíquetes, colete todas as informações necessárias antes de prosseguir
    ```

1. Baixe o documento de política de TI do repositório do laboratório. Abra uma nova guia do navegador e navegue até:

    ```
   https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-agents/main/Labfiles/01-build-agent-portal-and-vscode/IT_Policy.txt
    ```

    Salve o arquivo no computador local.

    > **Observação**: Este documento contém políticas de TI de exemplo para redefinições de senha, solicitações de instalação de software e solução de problemas de hardware.

1. Retorne ao playground do agente. Na seção **Tools**, selecione **Add** e adicione **File search** e **</> Code interpreter**.

1. À direita de **Add**, selecione **Upload files**. Em **Attach files**, procure e carregue o arquivo `IT_Policy.txt` que você acabou de baixar e selecione **Attach**.

1. Aguarde a indexação do arquivo. Uma confirmação será exibida quando ele estiver pronto.

1. Agora vamos adicionar alguns dados de desempenho para o interpretador de código analisar. Baixe o arquivo de dados de desempenho do sistema em:

    ```
   https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-agents/main/Labfiles/01-build-agent-portal-and-vscode/system_performance.csv
    ```

    Salve este arquivo no computador local.

1. À direita de **</> Code interpreter**, selecione **+ Files** e carregue o arquivo `system_performance.csv` que você acabou de baixar.

    > **Observação**: Este arquivo CSV contém métricas de sistema simuladas (uso de CPU, memória e disco) ao longo do tempo que o agente pode analisar.

1. Salve o agente.

## Testar o agente

Vamos testar o agente para ver como ele responde usando os dados de fundamentação.

1. Na interface de chat no lado direito do playground, insira o seguinte prompt:

    ```
   Qual é a política para redefinições de senha?
    ```

1. Examine a resposta. O agente deve fazer referência ao documento de política de TI e fornecer informações precisas sobre os procedimentos de redefinição de senha.

1. Tente outro prompt:

    ```
   Como solicito um novo software?
    ```

1. Novamente, examine a resposta e observe como o agente usa os dados de fundamentação.

1. Agora teste o interpretador de código com uma solicitação de análise de dados:

    ```
   Você pode analisar os dados de desempenho do sistema e me dizer se há alguma tendência preocupante?
    ```

1. O agente deve usar o interpretador de código para analisar o arquivo CSV e fornecer insights sobre o desempenho do sistema.

1. Tente solicitar uma visualização:

    ```
   Crie um gráfico mostrando o uso da CPU ao longo do tempo a partir dos dados de desempenho
    ```

1. O agente usará o interpretador de código para gerar visualizações e análises.

Muito bem! Você criou um agente com dados de fundamentação, pesquisa de arquivos e recursos de interpretador de código. Na próxima seção, você interagirá programaticamente com esse agente usando o VS Code.

## Interagir com o agente usando o VS Code

Como desenvolvedor, você pode passar algum tempo trabalhando no portal do Foundry; mas também é provável que passe bastante tempo no Visual Studio Code. A extensão Foundry Toolkit para VS Code oferece uma maneira conveniente de trabalhar com os recursos do projeto do Foundry sem sair do ambiente de desenvolvimento.

### Instalar e configurar a extensão do VS Code

Se você já instalou a extensão Foundry Toolkit, pode ignorar esta seção.

1. Abra o Visual Studio Code.

2. Selecione **Extensions** no painel esquerdo (ou pressione **Ctrl+Shift+X**).

3. Procure no marketplace de extensões a extensão `Foundry Toolkit for VS Code` da Microsoft e selecione **Install**.

    A instalação da extensão Foundry Toolkit adicionará a extensão Foundry Toolkit ao VS Code.

    > **Observação**: Atualmente, a extensão está listada como **Foundry Toolkit**, mas alguns rótulos e comandos do VS Code, ou capturas de tela mais antigas, ainda podem fazer referência a **AI Toolkit**. Neste laboratório, considere esses nomes como referências à mesma experiência da extensão.

4. Depois de instalar a extensão, selecione o ícone do Foundry Toolkit na barra lateral.

    Será solicitado que você entre na sua conta do Azure caso ainda não tenha feito isso.

### Testar o agente no VS Code

Antes de escrever qualquer código, você pode interagir diretamente com o agente na interface da extensão.

1. Em **Microsoft Foundry Resources**, escolha **Set Default Project**.

    Se um projeto padrão já estiver ativo, o nome do projeto aparecerá na lista de recursos. Você pode selecionar outro projeto selecionando o mesmo ícone **Select project**.

2. Expanda a seção do projeto. Em **Prompt Agents**, você deverá ver o `it-support-agent` criado no portal. Selecione o nome do agente para abrir a interface do Agent Builder.

    O playground do agente aparecerá na interface do Agent Builder, permitindo interagir com o agente e configurar suas definições sem sair do VS Code.

3. No painel de chat do playground, digite uma pergunta como:

    ```
   Qual é a política para comunicar a perda ou o roubo de um dispositivo?
    ```

4. Examine a resposta do agente. Ele deve usar os dados de fundamentação carregados anteriormente para fornecer informações relevantes sobre a política de TI.

    > **Dica**: Você pode usar este playground integrado para testar rapidamente as instruções e os conhecimentos do agente sem escrever código.

## Criar uma aplicação cliente para interagir com o agente

Agora vamos criar uma aplicação cliente que interage programaticamente com o agente.

1. No VS Code, abra a Paleta de Comandos (**Ctrl+Shift+P** ou **View > Command Palette**).

1. Digite **Git: Clone** e selecione-o na lista.

1. Insira a URL do repositório:

    ```
   https://github.com/MicrosoftLearning/mslearn-ai-agents.git
    ```

1. Escolha um local no computador local para clonar o repositório.

1. Quando solicitado, selecione **Open** para abrir o repositório clonado no VS Code.

1. Depois que o repositório for aberto, selecione **File > Open Folder** e navegue até `mslearn-ai-agents/Labfiles/01-build-agent-portal-and-vscode/Python`; em seguida, escolha **Select Folder**.

1. No painel Explorer, abra o arquivo `agent_with_functions.py`. Se o arquivo estiver vazio, substitua seu conteúdo pelo código a seguir.

1. Use o código a seguir:

    ```python
   import base64
   import os
   from pathlib import Path

   from azure.ai.projects import AIProjectClient
   from azure.identity import DefaultAzureCredential
   from dotenv import load_dotenv


   OUTPUT_DIR = Path("agent_outputs")


   def get_output_path(filename):
       """Cria um caminho exclusivo para arquivos gerados."""
       OUTPUT_DIR.mkdir(exist_ok=True)
       file_name = Path(filename).name
       stem = Path(file_name).stem or "output"
       suffix = Path(file_name).suffix
       output_path = OUTPUT_DIR / file_name

       counter = 1
       while output_path.exists():
           output_path = OUTPUT_DIR / f"{stem}_{counter}{suffix}"
           counter += 1

       return output_path


   def save_bytes(file_bytes, filename):
       """Salva conteúdo binário em um arquivo local."""
       output_path = get_output_path(filename)
       with open(output_path, "wb") as file_handle:
           file_handle.write(file_bytes)
       return output_path


   def save_image(image_data, filename):
       """Salva dados de imagem em base64 em um arquivo."""
       return save_bytes(base64.b64decode(image_data), filename)


   def download_container_file(openai_client, annotation, downloaded_files):
       """Baixa um arquivo de contêiner citado uma vez e retorna seu caminho local."""
       cache_key = (annotation.container_id, annotation.file_id)
       if cache_key in downloaded_files:
           return downloaded_files[cache_key]

       file_content = openai_client.containers.files.content.retrieve(
           file_id=annotation.file_id,
           container_id=annotation.container_id,
       )
       output_path = save_bytes(
           file_content.read(),
           annotation.filename or f"{annotation.file_id}.bin",
       )
       downloaded_files[cache_key] = output_path
       return output_path


   def format_output_text(content_item, openai_client, downloaded_files):
       """Substitui citações de arquivos do sandbox por caminhos de arquivos locais."""
       text = content_item.text or ""
       replacements = []
       referenced_files = set()

       for annotation in content_item.annotations or []:
           if getattr(annotation, "type", "") != "container_file_citation":
               continue

           output_path = download_container_file(openai_client, annotation, downloaded_files)
           replacement_text = f"{annotation.filename} (salvo em {output_path})"
           referenced_files.add(output_path)

           start_index = getattr(annotation, "start_index", None)
           end_index = getattr(annotation, "end_index", None)
           if start_index is not None and end_index is not None:
               replacements.append((start_index, end_index, replacement_text))
               continue

           annotated_text = getattr(annotation, "text", "")
           if annotated_text:
               text = text.replace(annotated_text, replacement_text)

       for start_index, end_index, replacement_text in sorted(replacements, reverse=True):
           text = f"{text[:start_index]}{replacement_text}{text[end_index:]}"

       return text, referenced_files


   def main():
       # Inicializa o cliente do projeto
       load_dotenv()
       project_endpoint = os.environ.get("PROJECT_ENDPOINT")
       agent_name = os.environ.get("AGENT_NAME", "it-support-agent")

       if not project_endpoint:
           print("Erro: a variável de ambiente PROJECT_ENDPOINT não foi definida")
           print("Defina-a no arquivo .env ou no ambiente")
           return

       print("Conectando ao projeto do Microsoft Foundry...")
       credential = DefaultAzureCredential()
       project_client = AIProjectClient(
           credential=credential,
           endpoint=project_endpoint
       )

       # Obtém o cliente OpenAI para a API de Responses
       openai_client = project_client.get_openai_client()

       # Obtém o agente criado no portal
       print(f"Carregando o agente: {agent_name}")
       agent = project_client.agents.get(agent_name=agent_name)
       print(f"Conectado ao agente: {agent.name} (id: {agent.id})")

       # Cria uma conversa
       conversation = openai_client.conversations.create(items=[])
       print(f"Conversa criada (id: {conversation.id})")

       # Loop de chat
       print("\n" + "="*60)
       print("Agente de Suporte de TI pronto!")
       print("Faça perguntas, solicite análises de dados ou peça ajuda.")
       print("Digite 'exit' para sair.")
       print("="*60 + "\n")

       while True:
           user_input = input("Você: ").strip()

           if user_input.lower() in ['exit', 'quit', 'bye']:
               print("Até logo!")
               break

           if not user_input:
               continue

           # Adiciona a mensagem do usuário à conversa
           openai_client.conversations.items.create(
               conversation_id=conversation.id,
               items=[{"type": "message", "role": "user", "content": user_input}]
           )

           # Obtém a resposta do agente
           print("\n[O agente está pensando...]")
           response = openai_client.responses.create(
               conversation=conversation.id,
               extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
               input=""
           )

           # Exibe a resposta e salva localmente os arquivos gerados
           handled_output = False
           downloaded_files = {}
           referenced_files = set()
           image_count = 0

           if hasattr(response, "output") and response.output:
               for item in response.output:
                   item_type = getattr(item, "type", "")

                   if item_type == "message" and getattr(item, "content", None):
                       for content_item in item.content:
                           if getattr(content_item, "type", "") != "output_text":
                               continue

                           formatted_text, message_files = format_output_text(
                               content_item,
                               openai_client,
                               downloaded_files,
                           )
                           referenced_files.update(message_files)

                           if formatted_text:
                               print(f"\nAgente: {formatted_text}\n")
                               handled_output = True

                   elif hasattr(item, "text") and item.text:
                       print(f"\nAgente: {item.text}\n")
                       handled_output = True

                   elif item_type == "image":
                       image_count += 1
                       filename = f"chart_{image_count}.png"

                       if hasattr(item, "image") and hasattr(item.image, "data"):
                           file_path = save_image(item.image.data, filename)
                           print(f"\n[O agente gerou um gráfico - salvo em: {file_path}]")
                       else:
                           print("\n[O agente gerou uma imagem]")
                       handled_output = True

               for file_path in downloaded_files.values():
                   if file_path not in referenced_files:
                       print(f"\n[O agente gerou um arquivo - salvo em: {file_path}]")
                       handled_output = True

           if not handled_output and hasattr(response, "output_text") and response.output_text:
               print(f"\nAgente: {response.output_text}\n")

   if __name__ == "__main__":
       main()
    ```

1. Salve o arquivo `agent_with_functions.py` (**Ctrl+S** ou **File > Save**).

### Configurar o ambiente e executar a aplicação

1. No painel Explorer, você verá os arquivos `.env.example` e `requirements.txt` já presentes na pasta.

1. Duplique o arquivo `.env.example` e renomeie-o para `.env`.

1. No arquivo `.env`, substitua `your_project_endpoint_here` pelo endpoint real do projeto:

    ```
   PROJECT_ENDPOINT=<your_project_endpoint>
   AGENT_NAME=it-support-agent
    ```

    **Para obter o endpoint do projeto:** No VS Code, abra a extensão **Foundry Toolkit**, clique com o botão direito no projeto ativo e selecione **Copy Endpoint**. Se **Copy Endpoint** não estiver disponível na versão instalada do Foundry Toolkit, abra o portal do Microsoft Foundry, acesse o projeto e copie o endpoint do projeto na página de visão geral do projeto.

1. Salve o arquivo `.env` (**Ctrl+S** ou **File > Save**).

1. Abra um terminal no VS Code (**Terminal > New Terminal**) e navegue até o diretório de trabalho.

1. Instale os pacotes necessários e entre:

    ```bash
   python -m venv labenv
   .\labenv\Scripts\Activate.ps1
   pip install -r requirements.txt
    ```

    ```bash
   az login
    ```

1. Execute a aplicação:

    ```bash
   python agent_with_functions.py
    ```

## Testar a aplicação cliente

Quando o agente for iniciado, tente os prompts a seguir para testar diferentes recursos:

1. Teste a pesquisa de políticas com a pesquisa de arquivos:

    ```
   Qual é a política para redefinições de senha?
    ```

2. Solicite uma análise de dados com o interpretador de código:

    ```
   Analise os dados de desempenho do sistema e identifique os períodos em que o uso da CPU excedeu 80%
    ```

3. Solicite uma visualização:

    ```
   Crie um gráfico de linhas mostrando as tendências de uso da memória ao longo do tempo
    ```

    A aplicação salva os gráficos gerados e os arquivos citados na pasta `agent_outputs` e imprime o caminho do arquivo local no terminal.

4. Solicite uma análise estatística:

    ```
   Quais são os valores médio, mínimo e máximo do uso do disco nos dados de desempenho?
    ```

5. Análise combinada:

    ```
   Encontre alguma correlação entre o alto uso da CPU e o uso da memória nos dados de desempenho
    ```

Observe como o agente usa tanto a pesquisa de arquivos (para perguntas sobre políticas) quanto o interpretador de código (para análise de dados) para atender às suas solicitações. O interpretador de código analisará os dados CSV, realizará cálculos e poderá até gerar visualizações. Digite `exit` quando terminar os testes.

## Limpeza

Para evitar cobranças desnecessárias do Azure, exclua os recursos criados:

1. No portal do Foundry, navegue até o projeto
1. Selecione **Settings** > **Delete project**
1. Como alternativa, exclua todo o grupo de recursos no portal do Azure
