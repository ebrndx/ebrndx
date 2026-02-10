---
layout: page
title: "Notifika"
subtitle: "Alertas Automatizados de Pacing de Mídia"
permalink: /projects/notifika/
---

[← Voltar para Projetos](/projects)

---

## Visão Geral

**Problema:** O time de mídia só descobria que uma frente estava gastando acima ou abaixo do planejado quando já era tarde para corrigir. Não havia alerta proativo — a informação dependia de alguém abrir um dashboard e conferir.

**Solução:** Sistema de alertas diários por e-mail que compara o gasto real MTD de cada frente com o planejado, classifica a severidade e entrega um resumo visual toda manhã na caixa de entrada do time.

**Stack:** Google Apps Script · BigQuery · E-mail (MailApp)

---

## O Problema

Em operações de mídia com múltiplas frentes rodando ao mesmo tempo, o controle de pacing é crítico. Gastar demais significa estourar o budget antes do fim do mês; gastar de menos significa perder janela de veiculação.

- O planejamento existia em planilhas e o gasto real vinha das plataformas — mas ninguém cruzava os dois de forma sistemática e diária
- Quando alguém detectava um desvio, já havia dias de atraso acumulado
- Sem classificação de severidade, tudo parecia urgente ou nada parecia urgente
- O time precisava de um alerta que chegasse até ele, não de mais um dashboard para lembrar de abrir

---

## A Solução

O Notifika funciona em duas camadas:

**Camada de dados (BigQuery):**
Uma view no BigQuery cruza o planejamento de mídia com os dados reais de gasto, calculando por frente: planejado MTD, gasto real MTD, delta percentual, direção (acima/abaixo) e nível de severidade. Toda a lógica de pacing vive na query — o script só consome o resultado.

**Camada de alerta (Apps Script):**
Um script agendado roda toda manhã, consulta a view do dia anterior e monta um e-mail HTML com:

- Cards de resumo — total de frentes ativas, alertas críticos, frentes em atenção e frentes no plano
- Tabela ordenada por severidade — as frentes mais fora do pacing aparecem primeiro, com ícones visuais (vermelho, laranja, amarelo, verde)
- Filtro automático — frentes sem plano de mídia disponível são separadas e listadas à parte, sem poluir a análise principal
- Nos primeiros dias do mês o alerta não dispara, porque o MTD ainda não é representativo

---

## Resultados

- Time de mídia passou a receber todo dia, antes de começar a trabalhar, um diagnóstico de pacing por frente
- Desvios críticos identificados no dia seguinte ao gasto, não mais dias depois
- Classificação de severidade eliminou o ruído — cada um sabe o que precisa de ação imediata
- O alerta vai até o time, sem depender de alguém lembrar de abrir um dashboard

---

[← Voltar para Projetos](/projects)
