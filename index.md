---
title: Exercícios de AI Agents em Português
permalink: index.html
layout: home
---

# Exercícios de AI Agents em Português

Bem-vindo ao material traduzido para português dos exercícios práticos de desenvolvimento de **AI Agents** no Microsoft Azure. As atividades usam o **Microsoft Foundry**, o **Foundry Agent Service**, o **Foundry Toolkit**, o **Agent Framework** e outras ferramentas do ecossistema Azure.

> **Observação:** os nomes de produtos, ferramentas, APIs, comandos e outros termos técnicos foram mantidos em inglês quando necessário para preservar a precisão técnica e facilitar a execução dos exercícios.

### Exercícios disponíveis

{% assign exercises = site.pages | where_exp:"page", "page.url contains '/Instructions/Exercises/'" | where_exp:"page", "page.lab.duration" | sort: "url" %}

{% for activity in labs  %}
{% if activity.lab.title %}
### [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }})


{% if activity.lab.level %}**Level**: {{activity.lab.level}} \| {% endif %}{% if activity.lab.duration %}**Duration**: {{activity.lab.duration}}{% endif %}

{% if activity.lab.description %}
*{{activity.lab.description}}*
{% endif %}
<hr>
{% endif %}
{% endfor %}

## Como usar este site

Selecione um exercício acima para abrir suas instruções completas. Os exemplos de código podem ser copiados diretamente para o ambiente de desenvolvimento, respeitando os pré-requisitos e as configurações descritas em cada atividade.

## Recursos relacionados

- [Microsoft Learn — Desenvolver AI Agents no Azure](https://learn.microsoft.com/training/paths/develop-ai-agents-on-azure/)
- [Microsoft Foundry](https://ai.azure.com)
- [Documentação do Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)

<style>
.exercise-card {
  border: 1px solid #d9e2f2;
  border-left: 5px solid #0078d4;
  border-radius: 8px;
  padding: 1rem 1.25rem;
  margin: 1rem 0;
  background: #f8fbff;
}
.exercise-card h3 { margin-top: 0; }
.exercise-card p:last-child { margin-bottom: 0; }
</style>
