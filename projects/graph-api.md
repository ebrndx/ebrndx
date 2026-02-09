---
layout: page
title: "Pipelines de Integração de APIs"
subtitle: "Meta Graph API & AppsFlyer → BigQuery"
permalink: /projects/graph-api/
---

[← Voltar para Projetos](/projects)

---

## Visão Geral

**Problema:** Dados de performance de social media (Meta) e de atribuição de app (AppsFlyer) eram coletados manualmente ou dependiam de conectores pagos com limitações de granularidade, cobertura e controle sobre o pipeline.

**Solução:** Pipelines automatizados em Google Apps Script que coletam dados diretamente das APIs da Meta e da AppsFlyer e armazenam no BigQuery com deduplicação, particionamento e agendamento automático.

**Stack:** Google Apps Script · Meta Graph API · AppsFlyer Master API · BigQuery · AI-assisted coding

---

## Meta Graph API

### O Problema

A coleta de dados de social media apresentava desafios operacionais concretos:

- **Granularidade limitada:** Conectores padrão não expunham todas as métricas disponíveis na API (ex: breakdown de reações, métricas de vídeo por threshold)
- **Custo de conectores:** Ferramentas como Supermetrics cobravam por volume e não cobriam todos os endpoints necessários
- **Fragilidade manual:** Exportações manuais dependiam de quem executava e quando, sem garantia de consistência
- **Cobertura parcial:** Stories do Instagram e métricas de Reels exigiam coleta separada, sem consolidação

### A Solução

Pipeline modular com scripts dedicados por plataforma e tipo de conteúdo:

**Facebook Pages:**
- Coleta de posts (imagens, carousels, vídeos, reels) com 50+ métricas por publicação
- Métricas de alcance (orgânico, pago, total), engajamento detalhado (reações por tipo, cliques por categoria), e video insights (views por threshold, watch time, completion rate)
- Agregações mensais de página (alcance, seguidores, visitas ao perfil)

**Instagram:**
- Feed (imagens, carousels, vídeos, reels) com métricas de alcance, engajamento, saves e shares
- Stories com views, replies, shares e follows
- Métricas de conta (alcance mensal, profile views, seguidores)

**Arquitetura:**
- Autenticação via Bearer token com retry automático e backoff exponencial
- Paginação cursor-based para volumes grandes (100+ páginas, 1000+ mídias)
- Cache de respostas da API por execução para otimizar chamadas
- Staging table temporária no BigQuery com MERGE (upsert) para deduplicação
- Particionamento por data de snapshot para queries eficientes

---

## AppsFlyer Master API

### O Problema

O acompanhamento de métricas de atribuição de app (instalações, compras, registros) dependia de exports manuais ou dashboards limitados da própria plataforma, sem cruzamento com dados de mídia e sem histórico granular no data warehouse.

### A Solução

Pipeline diário em Google Apps Script que consome a Master Aggregated Data API da AppsFlyer:

**Eventos de conversão:**
- Coleta de 9 eventos de atribuição: purchases (standard, SD, eSIM, eSIM PME, eSIM foreigner), registros (standard, foreigner, PME) e ativações
- Granularidade por dia de instalação, campanha, adset e ad
- Filtro por padrão de nomenclatura de campanha para manter apenas dados relevantes

**Métricas de instalação:**
- Cost, impressions, clicks, installs e sessions por campanha/adset/ad
- Mesma granularidade e janela temporal dos eventos

**Arquitetura:**
- Janela de coleta com lag de 2 dias e lookback de 14 dias para capturar atribuições tardias
- Staging table com autodetect de schema (CSV da API carregado direto no BigQuery)
- Mapeamento dinâmico de colunas — o script detecta automaticamente variações de nome entre versões da API
- Delete + Insert por janela de datas para garantir idempotência
- Tabelas particionadas por `install_day` para performance de query
- Trigger automático diário às 06:00 (America/Sao_Paulo)
- Evolução de schema automática: novas colunas adicionadas via `ALTER TABLE` quando necessário

---

## Destaques Técnicos

**Comuns aos dois pipelines:**
- **Idempotente:** Re-execuções não geram duplicatas
- **Graceful degradation:** Falha em um item não interrompe a coleta dos demais
- **BigQuery nativo:** Dados disponíveis diretamente no warehouse, sem intermediários
- **AI-assisted coding:** Desenvolvimento acelerado com apoio de IA para construção dos scripts e mapeamento de métricas

**Meta Graph API:**
- Rate limiting com espera adaptativa (erro 80004)
- Detecção automática de formato de mídia (image, carousel, video, reel)
- MERGE (upsert) para deduplicação

**AppsFlyer:**
- Schema detection dinâmico — adapta-se a mudanças de nomenclatura da API sem quebrar
- Parsing multi-formato de datas (ISO, dd/mm/yyyy, timestamp)
- Gestão automática de triggers (criação/remoção)

---

## Resultados

- Coleta automatizada de métricas de social media e atribuição de app em um único data warehouse
- Substituição de exportações manuais e dependência de conectores pagos
- Dados granulares de Meta e AppsFlyer disponíveis no BigQuery para dashboards e análises cruzadas
- Pipelines resilientes com retry, deduplicação e monitoramento de execução
- Visão unificada de mídia + atribuição para otimização de investimento

---

[← Voltar para Projetos](/projects)
