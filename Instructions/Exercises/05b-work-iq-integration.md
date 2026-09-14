---
lab:
    title: 'Work IQ - Inteligência do ambiente de trabalho para agentes de IA (opcional)'
    description: 'Crie agentes de IA que acessem dados do ambiente de trabalho do Microsoft 365 usando o Work IQ e o Model Context Protocol para preparação de reuniões, acompanhamento de projetos e itens de ação.'
    level: 300
    duration: 40
    islab: true
    status: 'released'
    layout: default
---

# Work IQ - Inteligência do ambiente de trabalho para agentes de IA

Neste laboratório, você criará um agente de IA que acessa os dados do seu ambiente de trabalho do Microsoft 365 usando o **Work IQ** — a camada de inteligência contextual da Microsoft criada com base no Model Context Protocol (MCP). Você criará um agente de inteligência do ambiente de trabalho capaz de preparar reuniões, acompanhar projetos, extrair itens de ação e responder a perguntas sobre o ambiente de trabalho usando dados reais do M365.

Este laboratório leva aproximadamente **40** minutos.

> **Observação:** Este é um **laboratório opcional/avançado** que requer uma licença do Microsoft 365 Copilot. Ele foi projetado para alunos corporativos, funcionários da Microsoft ou pessoas com acesso ao M365 Copilot. Contas padrão do M365 sem Copilot não funcionarão.

## Pré-requisitos

Antes de iniciar este laboratório, verifique se você tem:

- Conhecimento básico sobre agentes de IA e o Model Context Protocol (MCP)
- **Microsoft 365 com licença do Copilot**
- Aprovação do administrador de TI para o Work IQ (somente contas organizacionais)
- [Node.js 18](https://nodejs.org/en/download/) ou posterior instalado
- [Python 3.13](https://www.python.org/downloads/) instalado
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli) instalado (autenticado com `az login`)
- Dados ativos do M365 (emails, reuniões, chats do Teams) para consultar

> \* O Python 3.14 ainda não é compatível: algumas dependências não têm uma compilação para a versão 3.14. Este laboratório foi testado com o Python 3.13.12.

> **Importante:** O Work IQ **funciona somente** com contas habilitadas para o Microsoft 365 Copilot. Não é possível concluir este laboratório sem o Copilot.

## Instalar o Work IQ

1. Abra o terminal ou o prompt de comando.

2. Instale o Work IQ globalmente via npm:

    ```bash
   npm install -g @microsoft/workiq
    ```

3. Aceite o Contrato de Licença de Usuário Final:

    ```bash
   workiq accept-eula
    ```

4. Teste a instalação do Work IQ:

    ```bash
   workiq ask -q "Quais reuniões tenho hoje?"
    ```

5. **Se o teste for bem-sucedido** — você verá informações de reuniões do seu calendário do M365. Prossiga para a próxima tarefa!

6. **Se aparecer a mensagem "Admin consent required":**

   - O comando exibirá uma URL de consentimento
   - Envie essa URL ao administrador de TI com a mensagem: "Preciso de acesso ao Work IQ para o laboratório AI Agents do Microsoft Learn"
   - Aguarde a aprovação do administrador e tente o comando de teste novamente

7. **Se aparecer a mensagem "No M365 Copilot license":**

   - Infelizmente, não é possível concluir este laboratório sem uma licença do Copilot
   - Você ainda pode ler as instruções para entender os conceitos
   - Considere este laboratório opcional e retorne a ele quando tiver acesso ao Copilot

## Preparar-se para desenvolver um aplicativo no Visual Studio Code

Agora vamos usar o Visual Studio Code para desenvolver um aplicativo. Os arquivos de código do seu aplicativo foram fornecidos em um repositório do GitHub.

1. Inicie o Visual Studio Code e abra uma janela do terminal.

2. Digite o comando para clonar o repositório em uma pasta local (não importa qual pasta):

    ```bash
   git clone https://github.com/MicrosoftLearning/mslearn-ai-agents.git
    ```

3. Quando o repositório tiver sido clonado, abra a pasta no Visual Studio Code.

    > **Observação**: Se o Visual Studio Code mostrar uma mensagem pop-up solicitando que você confie no código que está abrindo, selecione **Yes, I trust the authors** para continuar.

4. Aguarde enquanto arquivos adicionais são instalados para dar suporte aos projetos de código Python no repositório (se solicitado).

    > **Observação**: Se for solicitado que você instale os ativos necessários para compilar e depurar, selecione **Not Now**.

5. No painel **Explorer**, expanda a pasta **Labfiles/05b-work-iq-integration/Python**.

    Os arquivos fornecidos incluem o código do aplicativo, as configurações e o código inicial do cliente do agente.

6. No terminal, digite o comando para criar um ambiente virtual do Python:

    ```bash
   python -m venv venv
    ```

7. Ative o ambiente virtual:

   **Windows:**

    ```bash
   venv\Scripts\activate
    ```

   **macOS/Linux:**

    ```bash
   source venv/bin/activate
    ```

8. Instale os pacotes Python necessários:

    ```bash
   pip install -r requirements.txt
    ```

9. Configure o arquivo `.env`:

   Na pasta do laboratório, abra o arquivo `.env` e atualize-o com o endpoint do seu projeto do Foundry:

    ```env
   PROJECT_ENDPOINT=https://your-project.services.ai.azure.com/api/projects/your-id
   MODEL_DEPLOYMENT_NAME=gpt-5
    ```

   > **Dica:** Para obter seu endpoint: no VS Code, abra a extensão **Foundry Toolkit**, clique com o botão direito do mouse no projeto ativo e selecione **Copy Endpoint**. O Foundry Toolkit está incluído na extensão Foundry Toolkit for VS Code.

### Verificar a configuração

Verifique se você tem:

- Work IQ instalado e acessível (`workiq --version` funciona)
- Consentimento do administrador aprovado (ou uma conta pessoal do M365 com Copilot)
- `workiq_lab.py` — aplicativo interativo principal
- `requirements.txt` — dependências do Python instaladas
- Arquivo `.env` configurado com o endpoint do seu projeto

## Explorar cenários de inteligência do ambiente de trabalho

Neste exercício, você executará um aplicativo interativo unificado que demonstra cinco cenários de inteligência do ambiente de trabalho usando um único agente de IA com ferramentas do Work IQ.

### Iniciar o aplicativo do laboratório

1. Verifique se você está no diretório do laboratório com o ambiente virtual ativado.

2. Execute o aplicativo do laboratório:

    ```bash
   python workiq_lab.py
    ```

3. O aplicativo irá:
   - Validar a configuração do Work IQ
   - Conectar-se ao seu projeto do Microsoft Foundry
   - Inicializar o cliente MCP do Work IQ
   - Criar um agente de inteligência do ambiente de trabalho
   - Exibir um menu interativo com cinco cenários

### Cenário de preparação para reuniões

Este cenário ajuda você a se preparar para reuniões reunindo o contexto relevante.

1. No menu principal, selecione **1 - Meeting Prep (Preparação para reuniões)**.

2. Quando solicitado, insira um tópico ou horário de reunião, por exemplo:
   - "minha reunião das 14h"
   - "sessão de planejamento do Q4"
   - "reunião diária da equipe"

3. O agente irá:
   - Encontrar os detalhes da sua reunião (horário, participantes, agenda)
   - Pesquisar emails recentes sobre o tópico
   - Procurar reuniões anteriores sobre esse assunto
   - Resumir os pontos e as decisões principais
   - Sugerir pontos para discussão

4. Revise a saída e observe:
   - Como as fontes são citadas (emails, reuniões, datas)
   - Como o agente sintetiza informações de várias fontes
   - O tempo economizado em comparação com uma pesquisa manual

**Reflexão:** Qual é a diferença em relação a pesquisar manualmente seu email e calendário?

### Cenário de status do projeto

Este cenário acompanha as atualizações do projeto em suas ferramentas de trabalho.

1. No menu principal, selecione **2 - Project Status (Status do projeto)**.

2. Insira o nome de um projeto em que você está trabalhando, por exemplo:
   - "Reformulação do site"
   - "OKRs do Q1"
   - "Integração de clientes"

3. O agente irá:
   - Pesquisar emails e mensagens do Teams sobre o projeto
   - Encontrar reuniões relacionadas e seus resultados
   - Identificar decisões e alterações recentes
   - Listar bloqueios ou problemas mencionados
   - Resumir as próximas etapas e os prazos

4. Analise os resultados:
   - Quão abrangente é a atualização de status?
   - Quais fontes o agente usou?
   - Isso poderia ser criado com APIs tradicionais? Qual é a diferença no esforço de desenvolvimento?

### Cenário de itens de ação

Este cenário extrai suas tarefas em aberto de várias fontes.

1. No menu principal, selecione **3 - Action Items (Itens de ação)**.

2. Escolha um intervalo de tempo (ou pressione Enter para usar "esta semana"):
   - "hoje"
   - "últimos 3 dias"
   - "este mês"

3. O agente irá:
   - Pesquisar notas de reuniões em busca de itens de ação atribuídos
   - Procurar emails relacionados a tarefas enviados a você
   - Verificar mensagens do Teams nas quais você foi mencionado
   - Identificar itens com prazos
   - Priorizar por urgência, se possível

4. Examine a saída:
   - Todos os seus itens de ação foram capturados?
   - Quão precisa é a priorização?
   - Onde os itens de ação foram encontrados (reuniões, emails, Teams)?

### Cenário de inteligência combinada

Este cenário demonstra o uso conjunto do **Work IQ** (dados do ambiente de trabalho) e do **Foundry IQ** (base de conhecimento).

> **Observação:** Este cenário requer o Azure AI Search configurado no seu projeto do Foundry com uma base de conhecimento indexada.

1. No menu principal, selecione **4 - Combined Intelligence (Inteligência combinada)**.

2. Insira um tópico que exista tanto nas discussões do seu ambiente de trabalho quanto na documentação oficial:
   - "política de trabalho remoto"
   - "relatórios de despesas"
   - "diretrizes de segurança"

3. O agente irá:
   - Pesquisar dados do ambiente de trabalho (Work IQ): emails, reuniões, discussões do Teams
   - Pesquisar a base de conhecimento (Foundry IQ): documentos oficiais, políticas, procedimentos
   - Comparar as discussões do ambiente de trabalho com a documentação oficial
   - Identificar lacunas ou inconsistências
   - Fornecer um resumo abrangente com fontes identificadas

4. Compare as duas perspectivas:
   - O que está documentado oficialmente em comparação com o que é discutido informalmente?
   - Existem contradições?
   - Qual fonte está mais atualizada?

**Insight principal:**

- O **Work IQ** informa o que as pessoas realmente estão fazendo e dizendo
- O **Foundry IQ** informa o que está documentado oficialmente
- **Juntos**, eles fornecem o contexto completo para a tomada de decisões

### Cenário de consulta personalizada

Este cenário permite explorar seus dados do ambiente de trabalho com suas próprias perguntas.

1. No menu principal, selecione **5 - Custom Query (Consulta personalizada)**.

2. Experimente diferentes tipos de perguntas sobre o ambiente de trabalho:

   **Pesquisas de email:**

    ```
   Encontre emails do meu gerente sobre o orçamento
    ```

   **Resumos de reuniões:**

    ```
   O que foi decidido na reunião diária de ontem?
    ```

   **Atividade da equipe:**

    ```
   O que a equipe de engenharia discutiu esta semana?
    ```

   **Descoberta de documentos:**

    ```
   Mostre documentos compartilhados sobre políticas de segurança
    ```

3. Faça experiências com:
   - Diferentes intervalos de tempo
   - Diferentes fontes de dados (emails, reuniões ou Teams)
   - Diferentes níveis de especificidade
   - Perguntas de acompanhamento para refinar os resultados

4. Observe o que funciona bem:
   - Consultas específicas geralmente funcionam melhor do que consultas vagas
   - Incluir intervalos de tempo melhora a relevância
   - Nomes e palavras-chave ajudam a restringir os resultados

## Explorar e experimentar

Agora que você concluiu todos os cenários, reserve de 5 a 10 minutos para explorar por conta própria.

### Testar casos extremos

1. Tente fazer consultas sobre dados que você não possui — como o agente responde?

2. Faça perguntas ambíguas — como o agente lida com elas?

3. Pesquise informações muito antigas — quais são os limites?

### Explorar diferentes estilos de consulta

1. **Muito específica**: "Encontre o email de João sobre o orçamento do Q3 enviado em 15 de janeiro"

2. **Muito ampla**: "Conte-me sobre os desenvolvimentos recentes"

3. **Comparativa**: "Compare as discussões desta semana com as da semana passada"

### Ver os recursos do Work IQ

No menu principal, selecione **6 - View Work IQ Capabilities (Exibir recursos do Work IQ)** para revisar:

- Visão geral da arquitetura
- Fontes de dados disponíveis
- Modelo de segurança e privacidade
- Comparação entre Work IQ e Foundry IQ
- Casos de uso comuns

## Entender o código

Vamos examinar os principais padrões usados neste laboratório.

### Padrão 1: inicialização do cliente MCP do Work IQ

```python
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client

# Armazena os parâmetros do servidor para reutilização
self.workiq_server_params = StdioServerParameters(
    command="npx",
    args=["-y", "@microsoft/workiq", "mcp"]
)

# Obtém as ferramentas disponíveis do servidor MCP do Work IQ
async def _fetch():
    async with stdio_client(self.workiq_server_params) as (read, write):
        async with ClientSession(read, write) as session:
            await session.initialize()
            tools_result = await session.list_tools()
            return tools_result.tools

raw_tools = asyncio.run(_fetch())
```

Em vez de manter uma conexão persistente, uma nova sessão MCP é aberta para cada operação. `StdioServerParameters` armazena o comando e os argumentos usados para iniciar o subprocesso do servidor MCP do Work IQ a cada vez.

### Padrão 2: criar um agente com ferramentas do Work IQ

```python
from azure.ai.projects.models import PromptAgentDefinition, FunctionTool

# Converte as ferramentas MCP em objetos FunctionTool
workiq_tools = [
    FunctionTool(
        name=tool.name,
        description=tool.description,
        parameters=tool.inputSchema,
    )
    for tool in raw_tools
]

# Cria um agente com ferramentas do Work IQ
self.agent = self.project_client.agents.create_version(
    agent_name="workplace-intelligence-agent",
    definition=PromptAgentDefinition(
        model=self.model_deployment,
        instructions="Você é um assistente de inteligência do ambiente de trabalho...",
        tools=workiq_tools  # Ferramentas do Work IQ adicionadas aqui
    )
)

# Mantém um mapa das ferramentas brutas para pesquisa durante a execução
self.raw_tools_map = {tool.name: tool for tool in raw_tools}
```

Cada ferramenta MCP é encapsulada em um objeto `FunctionTool` e passada para um `PromptAgentDefinition`. O mapa de ferramentas brutas permite uma pesquisa eficiente quando o agente chama uma ferramenta pelo nome.

### Padrão 3: executar consultas com a Responses API

```python
# Cria a conversa
conversation = self.openai_client.conversations.create(
    items=[{"type": "message", "role": "user", "content": query}]
)

# Cria uma resposta com o agente
response = self.openai_client.responses.create(
    conversation=conversation.id,
    extra_body={"agent_reference": {"name": self.agent.name, "type": "agent_reference"}}
)
```

Isso usa o padrão da Responses API (não o padrão antigo de Runs/Threads) para uma execução mais limpa do agente.

### Padrão 4: loop de chamadas de ferramentas

Após a resposta inicial, o agente pode solicitar uma ou mais chamadas de ferramentas do Work IQ. Elas devem ser executadas e retornadas para que a conversa continue:

```python
from openai.types.responses.response_input_param import FunctionCallOutput

while True:
    if response.status == "failed":
        break

    input_list = []
    for item in response.output:
        if item.type == "function_call":
            kwargs = json.loads(item.arguments)

            # Chama a ferramenta do Work IQ via MCP
            async def _execute():
                async with stdio_client(self.workiq_server_params) as (read, write):
                    async with ClientSession(read, write) as session:
                        await session.initialize()
                        return await session.call_tool(item.name, kwargs)

            result = asyncio.run(_execute())
            input_list.append(
                FunctionCallOutput(
                    type="function_call_output",
                    call_id=item.call_id,
                    output=result.content[0].text,
                )
            )

    if input_list:
        # Envia os resultados das ferramentas de volta e continua
        response = self.openai_client.responses.create(
            input=input_list,
            previous_response_id=response.id,
            extra_body={"agent_reference": {"name": self.agent.name, "type": "agent_reference"}}
        )
    else:
        break  # Não há mais chamadas de ferramentas — resposta final pronta
```

O loop continua até que o agente produza uma resposta sem chamadas de função pendentes; nesse momento, `response.output_text` contém a resposta final.

## Limpar recursos

O laboratório limpa automaticamente o agente quando você sai:

```python
self.openai_client.agents.delete_version(
    agent_name=self.agent.name,
    version=self.agent.version
)
```

Nenhum recurso do Azure é criado neste laboratório (o Work IQ usa sua licença do M365), portanto, não é necessária nenhuma limpeza adicional.

## Solução de problemas

### "Work IQ command not found"

**Solução:** Instale o Work IQ:

```bash
npm install -g @microsoft/workiq
```

### "Admin consent required"

**Solução:**

1. Execute `workiq mcp` para obter a URL de consentimento
2. Envie-a ao administrador de TI para aprovação
3. Ou use uma conta pessoal do M365 com Copilot

### "No M365 Copilot license"

**Solução:** Este laboratório requer o Copilot. Escolha uma destas opções:

- Compre uma licença do M365 Copilot (US$ 30/mês)
- Use uma conta organizacional com Copilot
- Leia o laboratório para entender os conceitos sem realizar as atividades práticas

### "MCP server not responding"

**Solução:** Teste o Work IQ diretamente:

```bash
workiq ask -q "Quais reuniões tenho?"
```

Se isso falhar, reinstale:

```bash
npm install -g @microsoft/workiq
```

### "No data returned"

**Solução:**

- Verifique se sua conta do M365 tem emails, reuniões e atividade no Teams
- Tente consultas mais amplas
- Verifique se sua consulta corresponde aos seus dados reais
