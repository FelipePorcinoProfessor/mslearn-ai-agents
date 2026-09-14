---
title: Desenvolva soluções de IA generativa no Azure
permalink: index.html
layout: default
---

<style>
  .exercise-intro { max-width: 52rem; margin: 0 auto 2rem; font-size: 1.08rem; line-height: 1.7; }
  .exercise-note { margin: 1.5rem 0 2.25rem; padding: 1rem 1.25rem; border-left: 4px solid #1a45a5; border-radius: 0 8px 8px 0; background: #f4f7fc; }
  .exercise-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(17rem, 1fr)); gap: 1rem; margin: 1.5rem 0 2.5rem; }
  .exercise-card { display: flex; flex-direction: column; min-height: 12rem; padding: 1.25rem; border: 1px solid #e1e5ec; border-radius: 12px; background: #fff; box-shadow: 0 2px 8px rgba(20, 40, 80, .06); transition: transform .18s ease, box-shadow .18s ease, border-color .18s ease; }
  .exercise-card:hover, .exercise-card:focus-within { transform: translateY(-3px); border-color: #8ca9df; box-shadow: 0 8px 20px rgba(20, 40, 80, .12); }
  .exercise-card h3 { margin: 0 0 .65rem; font-size: 1.08rem; line-height: 1.35; }
  .exercise-card h3 a { color: #173f91; text-decoration: none; }
  .exercise-card h3 a:hover, .exercise-card h3 a:focus { text-decoration: underline; }
  .exercise-description { flex: 1; margin: 0 0 1rem; color: #4b5563; font-size: .93rem; line-height: 1.55; }
  .exercise-meta { display: flex; flex-wrap: wrap; gap: .45rem; margin-top: auto; color: #5b6472; font-size: .78rem; }
  .exercise-badge { padding: .2rem .55rem; border-radius: 999px; background: #eef3fc; color: #173f91; font-weight: 600; }
  @media (max-width: 600px) { .exercise-grid { grid-template-columns: 1fr; } }
</style>

<div class="exercise-intro">
  <p>Os exercícios a seguir foram criados para oferecer uma experiência prática de aprendizagem, na qual você explorará tarefas comuns realizadas por desenvolvedores ao criar soluções de IA generativa no Microsoft Azure.</p>
</div>

<div class="exercise-note">
  <strong>Observação:</strong> para concluir os exercícios, você precisará de uma assinatura do Azure com permissões e cota suficientes para provisionar os recursos do Azure e os modelos de IA generativa necessários. Se ainda não tiver uma, você poderá criar uma <a href="https://azure.microsoft.com/free">conta do Azure</a>. Há uma opção de avaliação gratuita para novos usuários, com créditos para os primeiros 30 dias.
</div>

## Exercícios

<div class="exercise-grid">
  {%- assign exercises = site.pages | where_exp: "page", "page.path contains 'Instructions/Exercises/'" | sort: "path" -%}
  {%- for exercise in exercises -%}
    {%- if exercise.lab.title -%}
    <article class="exercise-card">
      <h3><a href="{{ exercise.url | relative_url }}">{{ exercise.lab.title }}</a></h3>
      {%- if exercise.lab.description -%}
      <p class="exercise-description">{{ exercise.lab.description }}</p>
      {%- endif -%}
      <div class="exercise-meta">
        {%- if exercise.lab.level -%}<span class="exercise-badge">Nível {{ exercise.lab.level }}</span>{%- endif -%}
        {%- if exercise.lab.duration -%}<span class="exercise-badge">{{ exercise.lab.duration }} min</span>{%- endif -%}
      </div>
    </article>
    {%- endif -%}
  {%- endfor -%}
</div>

> **Dica:** embora você possa concluir estes exercícios individualmente, eles foram projetados para complementar os módulos do [Microsoft Learn](https://aka.ms/mslearn-generative-ai), que apresentam uma explicação mais aprofundada dos conceitos subjacentes.
