---
title: Desenvolver soluções de IA generativa com o OpenAI do Azure e o Kernel Semântico
permalink: index.html
layout: home
---

Os exercícios a seguir foram elaborados para proporcionar uma experiência de aprendizado prática, na qual você explorará tarefas comuns que os desenvolvedores realizam ao criar soluções da IA generativa com o Kernel Semântico e o OpenAI do Azure.

> **Observação**: para concluir os exercícios, você precisará de uma assinatura do Azure na qual tenha permissões e cotas suficientes para provisionar os recursos do Azure e os modelos de IA generativa necessários. Caso ainda não tenha uma conta do Azure, inscreva-se para obter uma [conta do Azure](https://azure.microsoft.com/free). Há uma opção de avaliação gratuita para novos usuários que inclui créditos para os primeiros 30 dias.

## Exercícios

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions'" %} {% for activity in labs  %}
<hr>
### [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }})

{{activity.lab.description}}

{% endfor %}

> **Observação**: embora você possa concluir esses exercícios sozinho, eles foram projetados para complementar os módulos do [Microsoft Learn](https://learn.microsoft.com/training/paths/develop-ai-agents-azure-open-ai-semantic-kernel-sdk/), nos quais você encontrará um aprofundamento em alguns dos conceitos subjacentes nos quais esses exercícios se baseiam.


