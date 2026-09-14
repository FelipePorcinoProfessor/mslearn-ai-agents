---
lab:
    title: 'Implante agentes no Microsoft Teams e no Copilot'
    description: 'Publique agentes de IA no Microsoft Teams e no Microsoft 365 Copilot para acesso empresarial'
    level: 300
    duration: 40
    islab: true
    status: 'released'
    layout: default
---

# Implemente agentes no Microsoft Teams e no Copilot

Neste laboratório, você aprenderá a publicar agentes de IA no **Microsoft Teams** e no **Microsoft 365 Copilot**, para que os funcionários possam acessá-los onde já trabalham. Você criará um agente simples no portal do Foundry, adicionará grounding de conhecimento e o implantará em ambas as plataformas.

Este laboratório se concentra nos **fluxos de implantação e publicação**, não no desenvolvimento de agentes.

Este laboratório leva aproximadamente **40** minutos.

> **Observação**: A publicação no Microsoft 365 Copilot requer uma licença do Copilot. A implantação no Teams funciona com contas padrão do Microsoft 365.

## Pré-requisitos

Antes de iniciar este laboratório, verifique se você tem:

- Uma [assinatura do Azure](https://azure.microsoft.com/free/) com permissões para criar recursos de IA
- Uma **conta do Microsoft 365** com acesso ao Teams
- Uma **licença do Microsoft 365 Copilot** (opcional, para a implantação no Copilot)
- Familiaridade básica com o portal do Microsoft Foundry

## Criar um projeto do Foundry

O Microsoft Foundry usa projetos para organizar modelos, recursos, dados e outros ativos usados para desenvolver uma solução de IA.

1. Em um navegador da Web, abra o [portal do Foundry](https://ai.azure.com) em `https://ai.azure.com` e entre usando suas credenciais do Azure. Feche todas as dicas ou painéis de início rápido que forem abertos na primeira vez que você entrar e, se necessário, use o logotipo do **Foundry** no canto superior esquerdo para navegar até a página inicial.

    > **Importante**: Para este laboratório, você está usando a nova experiência do **Foundry**.

1. No banner superior, selecione **Start building** para experimentar a nova experiência do Microsoft Foundry.

1. Quando solicitado, crie um **novo** projeto e insira um nome válido para o projeto (por exemplo, *m365-lab*).

1. Expanda **Advanced options** e especifique as seguintes configurações:
    - **Foundry resource**: *Crie um novo recurso do Foundry ou selecione um existente*
    - **Subscription**: *Sua assinatura do Azure*
    - **Resource group**: *Crie ou selecione um grupo de recursos*
    - **Location**: *Selecione qualquer região disponível*\

    > \* Alguns recursos de IA do Azure têm limitações de cotas de modelos por região. Se um limite de cota for excedido posteriormente no exercício, talvez seja necessário criar outro recurso em uma região diferente.

1. Selecione **Create** e aguarde a criação do projeto.

2. Quando o projeto for criado, uma caixa de diálogo de boas-vindas poderá aparecer. Selecione **Next** para ler a mensagem de boas-vindas e, em seguida, selecione **Create agent**.

    Você também pode selecionar **Start building** na página inicial e selecionar **Create agents** no menu suspenso.

3. Defina o **Agent name** como `enterprise-knowledge-agent` e crie o agente.

O playground será aberto para o agente recém-criado. Você verá que um modelo implantado disponível já está selecionado para você.

## Configurar o agente com instruções e dados de grounding

Agora que você criou um agente, vamos configurá-lo com instruções e conhecimento para prepará-lo para a publicação.

1. Defina **Instructions** como:

    ```
   Você é um Assistente de Conhecimento Empresarial da Contoso Corporation.

   Sua função:
   - Responder a perguntas sobre as políticas e os procedimentos da empresa
   - Fornecer informações precisas a partir dos documentos carregados
   - Ser profissional, prestativo e conciso
   - Se não souber a resposta, dizer isso e sugerir com quem entrar em contato

   Sempre cite suas fontes ao fazer referência a políticas específicas.
    ```

2. Selecione **Save** para salvar a configuração atual do agente.

3. Baixe os documentos de política de exemplo. Abra novas guias do navegador e salve cada arquivo:

    **Política de Segurança de TI:**

    ```
   https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-agents/main/Labfiles/05a-m365-teams-integration/Python/sample_documents/it_security_policy.txt
    ```

    **Política de Trabalho Remoto:**

    ```
   https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-agents/main/Labfiles/05a-m365-teams-integration/Python/sample_documents/remote_work_policy.txt
    ```

4. Retorne à configuração do agente e role até a seção **Tools**.

5. Selecione **Upload files**.

6. Uma janela pop-up para anexar arquivos aparecerá. Anexe os arquivos baixados anteriormente.

7. Quando terminar, selecione **Attach**.

## Testar o agente no playground

1. No playground, faça uma pergunta sobre segurança de TI:

    ```
   Quais são os requisitos de senha para o meu laptop?
    ```

2. O agente deve fornecer informações específicas da política de segurança de TI (mínimo de 12 caracteres, letras maiúsculas, letras minúsculas, números, caracteres especiais etc.)

3. Tente fazer uma pergunta sobre trabalho remoto:

    ```
   Qual é o horário principal dos funcionários remotos?
    ```

4. O agente deve responder com informações da política de trabalho remoto (das 9h às 15h)

5. Tente outra consulta:

    ```
   Qual criptografia é exigida nos laptops da empresa?
    ```

6. Observe como o agente encontra o documento correto e fornece respostas precisas sobre os requisitos do BitLocker

    Agora seu agente tem grounding de conhecimento e pode responder a perguntas com base nos documentos da empresa.

7. Selecione **Save**.

## Publicar no Microsoft Teams

Agora você publicará o agente no Microsoft Teams para que os funcionários possam conversar diretamente com ele no Teams. Quando você publica no Teams, o portal do Foundry automaticamente:

- Cria um Azure Bot Service
- Gera um manifesto de aplicativo do Teams
- Empacota ícones e configurações do aplicativo
- Fornece um pacote de aplicativo para download

### Preparar as informações do aplicativo

Antes de publicar, reúna estas informações:

| Campo | Valor |
|-------|-------|
| **App Name** | Enterprise Knowledge Agent |
| **Short Description** | Assistente de IA para as políticas da empresa |
| **Full Description** | Assistente de IA empresarial que responde a perguntas sobre as políticas da empresa, os procedimentos de TI e os recursos para funcionários |
| **Developer Name** | Seu nome ou o nome da empresa |
| **Website URL** | <https://contoso.com> (um espaço reservado é suficiente para o laboratório) |
| **Privacy Policy URL** | <https://contoso.com/privacy> |
| **Terms of Use URL** | <https://contoso.com/terms> |

### Criar ícones do aplicativo

Você precisará de dois ícones para o aplicativo do Teams:

1. **Ícone colorido** (192x192 pixels)
   - Versão totalmente colorida do logotipo do aplicativo
   - Formato PNG

2. **Ícone de contorno** (32x32 pixels)
   - Contorno branco sobre fundo transparente
   - Formato PNG
   - Usado na barra lateral do Teams

> **Opção rápida para este laboratório**: Crie um quadrado colorido simples com texto ou iniciais usando o PowerPoint, o Paint ou uma ferramenta online como o Canva.

### Publicar no portal

1. No portal do Foundry, abra seu agente (**Build** → **Agents** → **enterprise-knowledge-agent**)

2. Selecione o botão **Publish** na parte superior da página

3. Selecione **Publish to Teams and Microsoft 365 Copilot**.

4. Selecione **Continue**

### Configurar os detalhes do aplicativo do Teams

Preencha o formulário de configuração:

**Informações básicas:**

- **App Name**: Enterprise Knowledge Agent
- **Short Description**: Assistente de IA para as políticas da empresa
- **Full Description**: Assistente de IA empresarial que responde a perguntas sobre as políticas da empresa, os procedimentos de TI e os recursos para funcionários

**Informações do desenvolvedor:**

- **Developer Name**: Seu nome
- **Website**: <https://contoso.com>
- **Privacy Policy**: <https://contoso.com/privacy>
- **Terms of Use**: <https://contoso.com/terms>

**Ícones do aplicativo:**

- Carregue seu **ícone colorido** (192x192 px)
- Carregue seu **ícone de contorno** (32x32 px)

**Escopo do aplicativo:**

- Selecione **Personal** para acesso a conversas individuais
- Opcionalmente, selecione **Team** para acesso a canais

Selecione **Prepare Agent**

### Implantar no Teams

Depois que o pacote do agente estiver preparado (isso leva de 1 a 2 minutos), você poderá implantá-lo no Teams:

1. Quando o pacote estiver pronto, selecione **Continue the in-product publishing flow**

2. Escolha o escopo da publicação:
   - **Individual scope**: O agente aparece em "Your agents" na loja de agentes do Teams. Nenhuma aprovação do administrador é necessária. É a melhor opção para testes pessoais.
   - **Organization (tenant) scope**: O agente aparece em "Built by your org" para todos os usuários. Requer aprovação do administrador.

3. Para este laboratório, selecione **Individual scope**

4. Selecione **Submit**

5. Aguarde a conclusão da publicação (você verá uma mensagem de sucesso)

> **Alternativa se a publicação direta falhar**: Se a caixa de diálogo de publicação retornar um erro **400** e sua conta do Microsoft 365 tiver permissão para publicar aplicativos personalizados, abra a guia **Download & customize** e siga as instruções.

6. Seu agente agora está disponível no Teams! Encontre-o em **Apps** → **Your agents**

### Testar o agente no Teams

1. A conversa do agente deverá abrir após a instalação (ou localize-o em **Apps** → **Your agents**)

2. Envie uma saudação:

    ```
   Olá! Em que você pode me ajudar?
    ```

3. Teste uma consulta de conhecimento:

    ```
   Quais são os requisitos de senha do laptop?
    ```

4. Tente outra pergunta:

    ```
   Quais métodos de MFA são compatíveis?
    ```

5. O agente deve responder com informações do documento de política de segurança de TI!

**🎉 Parabéns!** Seu agente agora está disponível no Microsoft Teams!

### Solução de problemas da implantação no Teams

**Não consigo encontrar o agente no Teams (após a publicação direta):**

- Verifique a seção **Apps** → **Your agents** no Teams
- Aguarde de 1 a 2 minutos para que o agente apareça após a publicação
- Verifique se a publicação foi concluída com êxito no portal do Foundry

**Não consigo carregar o aplicativo (carregamento manual):**

- Verifique se o arquivo manifest.zip não está corrompido (baixe-o novamente, se necessário)
- Verifique se o administrador do Teams não desabilitou o carregamento de aplicativos personalizados
- Verifique se os ícones têm os tamanhos corretos (192x192 e 32x32)

**O agente não responde:**

- Aguarde 30 segundos após a instalação para que o bot seja inicializado
- Verifique se o Azure Bot Service foi criado (isso é mostrado durante a publicação)
- Primeiro, teste o agente no playground do Foundry

**As respostas são genéricas (sem conhecimento):**

- Verifique se a pesquisa de arquivos está habilitada no agente
- Confirme se os documentos foram carregados e indexados
- Teste as consultas de conhecimento no playground do Foundry

## Publicar no Microsoft 365 Copilot

Agora você publicará o agente como uma extensão do Microsoft 365 Copilot, permitindo que os usuários o acessem diretamente no Copilot. Quando você publica no Copilot, o agente se torna uma **extensão do Copilot** (também chamada de plugin ou agente declarativo). Os usuários podem:

- Invocar o agente usando @menções no Copilot
- Acessar o conhecimento do agente junto com os recursos do Copilot
- Alternar perfeitamente entre o Copilot e o agente

> **Observação**: Esta seção requer uma licença do Microsoft 365 Copilot. Se você não tiver uma, poderá ler as etapas para entender o processo.

### Publicar no portal

1. Retorne ao portal do Foundry (**<https://ai.azure.com>**)

2. Navegue até seu agente (**Build** → **Agents** → **enterprise-knowledge-agent**)

3. Selecione o botão **Publish**

4. Selecione **Publish to Teams and Microsoft 365 Copilot**

5. Selecione **Continue**

> **Observação**: Este é o mesmo fluxo de publicação usado para o Teams. O agente fica disponível no Teams e no Copilot por meio de um único processo de publicação.

### Configurar os detalhes da publicação

Se você ainda não publicou este agente, preencha a configuração (igual à seção do Teams):

- **Name**: Enterprise Knowledge Agent
- **Description**: Assistente de IA para as políticas de TI da empresa
- **Icons**: Carregue seus ícones de 192x192 e 32x32
- **Publisher information**: Seu nome e URLs de espaço reservado

### Escolher o escopo da publicação

Selecione seu escopo de distribuição:

| Escopo | Visibilidade | Aprovação do administrador | Melhor para |
|-------|-----------|----------------|----------|
| **Shared** | Em "Your agents" na loja de agentes | Não obrigatória | Testes pessoais, equipes pequenas |
| **Organization** | Em "Built by your org" para todos os usuários | Obrigatória | Distribuição em toda a organização |

Para este laboratório, selecione **Shared scope** para obter acesso imediato sem aprovação do administrador.

### Concluir a publicação

1. Selecione **Prepare Agent** e aguarde o empacotamento (1 a 2 minutos)

2. Selecione **Continue the in-product publishing flow**

3. Confirme a seleção do escopo e selecione **Publish**

4. Aguarde a conclusão da publicação

### Acessar no Microsoft 365 Copilot

Depois de publicado com o escopo compartilhado, seu agente ficará imediatamente disponível:

1. Abra o **Microsoft 365 Copilot** (copilot.microsoft.com ou nos aplicativos do Microsoft 365)

2. Procure a loja de agentes ou o painel **Extensions**

3. Encontre seu agente em **Your agents** (para o escopo compartilhado)

4. Inicie uma conversa:

    ```
   @Enterprise Knowledge Agent Quais são os requisitos de segurança do laptop?
    ```

5. Ou selecione seu agente e faça uma pergunta diretamente:

    ```
   Quais métodos de MFA são compatíveis com os sistemas da empresa?
    ```

6. O Copilot encaminha a consulta ao seu agente e retorna informações da política de segurança de TI

> **Observação**: Para o **escopo da organização**, um administrador precisa primeiro aprovar o aplicativo no [Centro de administração do Microsoft 365](https://admin.cloud.microsoft/?#/agents/all/requested), em **Requests**. Depois de aprovado, o agente aparecerá em **Built by your org** para todos os usuários.

## Limpeza

Para evitar cobranças desnecessárias, limpe os recursos quando terminar.

### Excluir o agente

1. No portal do Foundry, vá para **Build** → **Agents**

2. Localize **enterprise-knowledge-agent**

3. Selecione o menu **...** → **Delete**

4. Confirme a exclusão

Isso também remove:

- O Azure Bot Service
- As configurações associadas
- As implantações publicadas

### Desinstalar do Teams

1. Abra o Microsoft Teams

2. Vá para **Apps** → **Manage your apps**

3. Localize **Enterprise Knowledge Agent**

4. Selecione **...** → **Uninstall**

5. Confirme a desinstalação

### Remover a extensão do Copilot

Se você publicou no Copilot:

1. A extensão fica inativa quando o agente é excluído
2. Os usuários verão um erro se tentarem usá-la
3. O administrador talvez precise removê-la do catálogo da organização
