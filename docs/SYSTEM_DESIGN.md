# Framework de Famílias de Estratégias Spot Multi-Exchange

## 1. Visão Geral do Projeto

Este documento descreve um **framework de famílias de estratégias de trading sistemático** voltado para mercados **spot** (inicialmente cripto, com foco em Binance Spot). Ele é projetado para operar em **single node**, com uma arquitetura preparada para **multi-exchange**.

O projeto é uma **plataforma estruturada** onde:

*   Famílias de estratégias (Trend Following, Dual Momentum, etc.) são implementadas de forma consistente.
*   Há um **núcleo comum de risco, execução, dados e observabilidade**.
*   Cada estratégia passa por um ciclo disciplinado de **pesquisa → backtest → paper trading → produção → monitoramento → eventual aposentadoria**.

---

### 1.1. Contexto

Os mercados de cripto, FX e ações spot exibem:

* Volatilidade elevada;
* Mudanças de regime frequentes;
* Risco de cauda significativo.
---

### 1.2. Objetivo Principal

O objetivo principal do projeto é:

> **Construir um framework unificado de famílias de estratégias sistemáticas para mercado spot long/flat, priorizando famílias com robusta evidência histórica de retorno ajustado ao risco, em uma arquitetura single node preparada para multi-exchange.**

Na prática, isso se traduz em:

* Ter um **núcleo arquitetural** bem definido (dados, estratégias, risco, execução, observabilidade, governança);
* Implementar as famílias de estratégias em uma **ordem lógica de maturidade**, começando por Trend Following/TSM e Dual Momentum, e progredindo para famílias mais complexas e/ou frágeis (Mean Reversion intraday, Market Making);
* Garantir que **cada decisão de trading seja explicável, auditável e reprodutível**;
* Permitir evolução incremental do sistema – mais famílias, mais exchanges, mais ativos – sem reescrever o núcleo.

---

### 1.3. Escopo Inicial

O **escopo inicial (MVP expandido)** do projeto abrange:

* **Ambiente de Execução**

  * Single node (uma máquina física ou virtual, possivelmente um SBC como Raspberry Pi de alto desempenho);
  * Sistema operacional Linux (ou equivalente) como alvo principal.

* **Mercados e Exchanges**

  * Foco inicial em **Binance Spot**;
  * Arquitetura pronta para multi-exchange, mas operação real inicialmente com uma única exchange.

* **Tipo de Operação**

  * Somente **mercado spot**, **modo long/flat** (sem short descoberto, sem alavancagem no MVP);
  * Possibilidade futura de incluir módulos opcionais que utilizem derivativos básicos (por exemplo, basis/funding), mas fora do núcleo obrigatório.

* **Famílias de Estratégias**

  * Núcleo inicial contemplando:

    * Trend Following / Time-Series Momentum (TSM);
    * Dual Momentum;
    * Cross-Sectional Momentum;
    * Mean Reversion de curto prazo (intraday/swing), em estágio posterior;
    * Value + Quality e Carry/Yield/Basis, onde houver dados e infraestrutura adequados.
  * Market Making estruturado é considerado uma família de **prioridade baixa** para fases posteriores, devido a requisitos de infra e competição.

* **Funcionalidades Transversais**

  * Módulo de gestão de risco **desacoplado** das estratégias;
  * Motor de backtest unificado e isomórfico à produção;
  * Observabilidade (logs estruturados, métricas, dashboards, alertas);
  * Governança de estratégias (pipeline de aprovação, versionamento, rollout/rollback).

---

### 1.4. Não-Escopo (fora do MVP)

Os seguintes itens são explicitamente considerados **fora do escopo inicial**:

*   **High-Frequency Trading (HFT) ou colocation**.
*   **Derivativos Complexos** (futuros alavancados, opções estruturadas, etc.).
*   **Gestão de Capital de Terceiros**.
*   **Sistemas Distribuídos de Alta Complexidade** (clusters, microserviços).
*   **Automação de Estratégias Puramente Discricionárias**.

---

### 1.5. Stakeholders e Perfis de Usuário

Mesmo em um contexto proprietário (um único operador/gestor), é útil explicitar os **stakeholders** e seus perfis:

* **Trader/Researcher Quantitativo**

  * Responsável por:

    * Propor novas famílias e instâncias de estratégias;
    * Executar backtests, analisar resultados, ajustar parâmetros;
    * Documentar hipóteses, riscos e métricas.

* **Responsável por Risco**

  * Pode ser a mesma pessoa em um setup enxuto, mas o “chapéu” é distinto:

    * Define limites de risco por trade, estratégia, família e portfólio;
    * Aprova ou rejeita estratégias para produção;
    * Monitora drawdown, exposição e acionamento de circuit breakers.

* **Responsável Operacional/DevOps**

  * Cuida de:

    * Infraestrutura do single node (sistema operacional, atualizações, segurança básica);
    * Deploys, monitoramento de serviços, backups;
    * Resposta a incidentes técnicos (queda de processo, problemas de disco/rede).

* **“Sistema” como Stakeholder**

  * A própria plataforma, enquanto conjunto de módulos, é tratada como algo que precisa:

    * De coerência interna;
    * De evolução consistente;
    * De observabilidade, para garantir sua própria saúde.

Em ambientes mais complexos, esses papéis podem ser distribuídos entre pessoas diferentes; no cenário minimalista, podem ser exercidos por uma mesma pessoa em momentos distintos.

---

### 1.6. Principais Premissas e Restrições

O projeto se apoia em algumas **premissas** e está sujeito a certas **restrições** importantes:

**Premissas**

*   O ambiente alvo é **single node** com recursos suficientes (CPU/RAM/SSD).
*   A **API da exchange é a "fonte da verdade"** para o universo de mercados (símbolos, filtros, timeframes).
*   As **famílias de estratégias** são baseadas em regras claras e documentadas.

**Restrições**

* Como o ambiente é single node:

  * Há limite natural de **capacidade de processamento e armazenamento**;
  * A frequência de execução e o número de estratégias simultâneas precisam ser compatíveis com o hardware disponível.

* Dependência de **APIs de exchange**:

  * A disponibilidade do sistema está parcialmente condicionada à saúde dos endpoints (REST/WebSocket);
  * Mudanças na API ou em políticas da exchange exigem capacidade de adaptação (schema discovery, adapters).

* O sistema **não assume infraestrutura de terceiros sofisticada**:

  * Não se baseia em serviços externos de dados proprietários, colocation ou FPGA;
  * As decisões são tomadas a partir de dados públicos das exchanges e histórico armazenado localmente.

---

Em resumo, a **Visão Geral do Projeto** define um framework que:

*   É pragmático (single node, uma exchange, foco em spot long/flat).
*   É ancorado em famílias de estratégias sistemáticas.
*   Prioriza gestão de risco, governança e observabilidade.
*   É projetado para evoluir em complexidade (mais estratégias, mais exchanges) de forma controlada.

## 2. Requisitos e Premissas do Sistema

Esta seção define **o que o sistema deve fazer** (requisitos funcionais), **como ele deve se comportar** (requisitos não funcionais) e **em quais condições se supõe que ele vai operar** (premissas).
Ela serve como contrato de referência para arquitetura, desenvolvimento, testes e operação.

---

### 2.1. Requisitos Funcionais

Os requisitos funcionais descrevem as capacidades concretas que o sistema precisa oferecer, do ponto de vista de trading algorítmico.

#### 2.1.1. Execução de Famílias de Estratégias Sistemáticas

* O sistema deve permitir **registrar e executar famílias de estratégias** (TSM, Dual Momentum, Cross-Sectional, Carry/Yield/Basis, Value/Quality, Mean Reversion, Market Making).
* Cada família pode ter **múltiplas instâncias** (variações de parâmetros, conjuntos de ativos, timeframes).
* Estratégias devem operar em modo **spot long/flat**, gerando **posições alvo** (porcentagem de alocação por mercado), e não ordens brutas.
* A posição alvo (ex: 100% alocado) é traduzida em valor nocional pelo **Módulo de Risco e Alocação**, que gerencia o capital disponível na moeda de cotação (`quote_asset`, ex: USDT) daquele mercado.

* **Validação de Lookback Mínimo**: Na fase de `prepare()` de cada estratégia, o sistema deve automaticamente calcular e verificar se o histórico de dados disponível atende aos `lookbacks mínimos` declarados pela estratégia. Caso contrário, um erro ou alerta crítico deve ser emitido, impedindo a execução da estratégia com dados insuficientes.
 
#### 2.1.2. Suporte a Ciclo Completo: Backtest → Paper → Live

O sistema deve suportar três modos principais para cada instância de estratégia:

* **Backtest** (offline, com dados históricos);
* **Paper trading** (online com dados em tempo real, mas ordens simuladas);
* **Live trading** (online com ordens reais junto à exchange).

A mudança de modo deve ser controlada por configuração e registrada em trilhas de governança.

#### 2.1.3. Orquestração por Timeframe e Evento

* O sistema deve possuir um **orquestrador/scheduler** capaz de:

  * Disparar a execução de estratégias em função de:

    * Fechamento de candles em timeframes específicos (ex.: 1m, 5m, 1h, 1d);
    * Janelas de tempo (por exemplo, a cada X minutos);
    * Eventualmente, eventos específicos (detecção de mudança de regime, alarme de risco etc.).
* O orquestrador deve conseguir coordenar **múltiplas estratégias e famílias simultaneamente**, respeitando prioridades configuradas.

#### 2.1.4. Aquisição e Normalização de Dados de Mercado

* O sistema deve:

  * Coletar **dados históricos** (backfill) e **dados em tempo real** (candles, eventualmente trades e book) da exchange;
  * **Normalizar** esses dados (timestamps em UTC, OHLCV padronizados, símbolos normalizados `base/quote`);
  * Detectar e sinalizar problemas de integridade (gaps, candles inconsistentes).

* Deve existir uma camada de **enriquecimento de dados** que:

  * Calcule indicadores e features necessárias para cada família (retornos, volatilidade, ranks, indicadores técnicos, etc.);
  * Garanta que as estratégias só recebam dados com **lookback suficiente**.

* Deve haver uma **Política de Saneamento de Dados** explícita:
  * Se um dado é marcado como `BAD` (ex: gap de candle irrecuperável), a estratégia para aquele ativo não deve rodar e um alerta deve ser emitido.
  * Se um dado é `QUESTIONABLE` (ex: outlier de volume), o sistema deve logar o evento e, por padrão, tentar usar o último dado válido conhecido, sinalizando a incerteza na trilha de auditoria.

#### 2.1.5. Gestão de Risco Multicamadas

O sistema deve possuir um módulo de risco desacoplado da estratégia, capaz de:

* Aplicar limites de risco:

  * Por trade;
  * Por instância de estratégia;
  * Por família de estratégias;
  * Por ativo/mercado;
  * Por exchange;
  * Globalmente (portfólio total).
* Implementar **circuit breakers**:

  * **Locais**: Desativar uma instância de estratégia se seu drawdown atingir um limite configurado (ex: `CB_STRATEGY_MAX_DRAWDOWN = 15%`).
  * **Globais**: Parar todas as operações (modo "somente saídas") se a perda diária do portfólio total exceder um limite crítico (ex: `CB_GLOBAL_DAILY_LOSS = 5%`).

Esse módulo deve atuar como **filtro obrigatório** entre sinais de estratégia e execução de ordens.

#### 2.1.6. Execução de Ordens Spot

* O sistema deve ser capaz de:

  * Traduzir posições alvo em **ordens spot concretas** (compra/venda) compatíveis com os filtros da exchange (lote mínimo, step, notional mínimo);
  * Enviar ordens via API da exchange (Binance Spot no MVP);
  * Calcular e registrar os **custos de transação** (fees) para cada `fill`, com base nas regras da exchange (maker/taker), e estimar o **slippage** (diferença entre preço esperado e executado).
  * Lidar com:

    * Respostas de sucesso;
    * Erros de validação;
    * Rate limits;
    * Timeouts.

* Deve haver suporte a, no mínimo:

  * Ordens **market**;
  * Ordens **limit** (incluindo cancelamentos).

#### 2.1.7. Persistência de Estado Operacional

O sistema deve **persistir seu estado operacional** para garantir recuperação após falhas e para auditoria. A arquitetura de estado é dividida em:

* **Estado em Memória (Hot State)**: Estruturas de dados otimizadas para acesso rápido durante o core loop, contendo posições atuais, ordens abertas e métricas de risco.
* **Estado Persistido (Warm/Cold State)**:
  * **Journal de Decisões e Ordens**: Um log de escrita antecipada (write-ahead log) para todas as intenções de trade e ordens enviadas, garantindo durabilidade.
  * **Banco de Dados Transacional (SQLite)**: Armazena o estado consolidado de posições, ordens, fills, e P&L.
  * **Snapshots Periódicos**: O estado completo é salvo em snapshots (ex: a cada hora e em shutdown controlado) para acelerar a recuperação.

* **Idempotência e Deduplicação**: Mecanismos como IDs de correlação (`correlation_id`) são usados para rastrear cada decisão de ponta a ponta, evitando o envio de ordens duplicadas após um reinício ou falha de comunicação.

Esse estado deve ser recuperável após um restart e reconciliado com a exchange.

#### 2.1.8. Observabilidade, Auditoria e Relatórios

Funcionalmente, o sistema deve:

* Gerar **logs estruturados** de decisões e eventos críticos;
* Expor **métricas** de sistema (CPU, memória, latência) e de negócio (P&L, exposição, drawdown);
* Manter uma **trilha de auditoria** de:
  * Custos completos por trade (fees + slippage estimado/real);

  * Decisões de risco;
  * Alterações de configuração;
  * Rollouts/rollbacks de estratégias.

Deve ser possível gerar **relatórios periódicos** (diários, semanais, mensais) de performance e risco.
Deve ser possível gerar automaticamente a **Ficha de Estratégia** (metadados, parâmetros, limites de risco, KPIs-alvo, changelog) a partir dos arquivos de configuração declarativa e do código-fonte.

---

### 2.2. Requisitos Não Funcionais

Os requisitos não funcionais definem qualidades esperadas do sistema, além do “o que ele faz”.

#### 2.2.1. Desempenho e Escalabilidade (Single Node)

* O sistema deve operar confortavelmente em **single node**, com capacidade de:

  * Executar dezenas de instâncias de estratégias em timeframes de minutos a horas;
  * Coletar e armazenar dados históricos de múltiplos símbolos, seguindo uma **política de retenção clara** (por exemplo, reter 5 anos para timeframes diários e semanais, mas expurgar dados de 1 minuto após 30-60 dias para gerenciar o uso de disco).
  * Manter latência de decisão compatível com trading não-HFT. Estes são **soft targets** (metas desejáveis), não SLAs (acordos de nível de serviço) críticos, mas servem como referência para o design:
    * Latência de chamadas REST à exchange: idealmente abaixo de 500ms.
    * Latência do core loop (do dado à ordem candidata): idealmente abaixo de 2 segundos para a maioria dos ciclos.

* Deve ser possível **ajustar a carga** (número de estratégias, ativos, timeframes) conforme os recursos da máquina, sem necessidade de reestruturação profunda.

* **Hardware de Referência (Exemplo)**: Como linha de base para dezenas de estratégias em timeframes de minutos, o sistema deve ser projetado para operar em hardware como:
  * **CPU**: 4+ cores
  * **RAM**: 16GB+
  * **Armazenamento**: 128GB+ SSD NVMe
  * **Rede**: Conexão estável de 100Mbps+
* O requisito de armazenamento (ex: 128GB SSD) deve ser validado contra a política de retenção e o universo de ativos, não sendo um valor fixo, mas uma consequência do escopo operacional desejado.

#### 2.2.2. Confiabilidade e Resiliência

* O sistema deve ser resiliente a:

  * Falhas temporárias de rede;
  * Erros de API da exchange;
  * Queda do processo ou reboot do nó.

* Em caso de falha:

  * Deve **retomar** do último estado consistente (posições, ordens) após restart;
  * Deve evitar ordens duplicadas ou perda de controle de posição (reconciliation).

#### 2.2.3. Segurança

* O sistema deve seguir boas práticas de **Security by Design**:

  * Segregação de segredos (API keys fora do código, criptografia em repouso quando possível);
  * Princípio do menor privilégio (permissões mínimas em chaves, SO e módulos internos);
  * Proteção de logs contra vazamento de informações sensíveis.

* Deve incluir proteções contra:

  * Envio acidental de ordens acima dos limites de risco;
  * Erros de configuração críticas (ex.: modo live ativado sem intenção).

#### 2.2.4. Manutenibilidade e Extensibilidade

* O código deve ser **modular** e **organizado por domínios** (dados, estratégias, risco, execução, observabilidade, governança).

* Deve ser simples:

  * Adicionar novas famílias de estratégias, implementando apenas a Strategy API;
  * Adicionar novas exchanges, implementando apenas os adapters de dados e trading;
  * Evoluir o módulo de risco sem mudar estratégias.

* Configurações devem ser **declarativas**, com validação automática (Pydantic ou equivalente).

#### 2.2.5. Observabilidade

* O sistema deve ser **observável por padrão**:

  * Logs estruturados, métricas de negócio e de infraestrutura;
  * Dashboards e alertas configuráveis.

* Problemas operacionais e de risco devem ser **detectáveis rapidamente**, com diagnóstico facilitado (correlação entre logs, métricas e eventos).

---

### 2.3. Premissas Operacionais

As premissas operacionais são condições assumidas como verdade para o desenho do sistema.

#### 2.3.1. Ambiente de Execução

*   O sistema será executado em **um único nó** (servidor dedicado, VM ou hardware embarcado de alto desempenho).
*   O sistema operacional alvo é **Linux** (ou derivado), com acesso à internet estável e recursos adequados (CPU, RAM, SSD).

#### 2.3.2. Conectividade com a Exchange

* Supõe-se que a exchange principal (inicialmente Binance Spot) oferece:

  * API REST estável para dados históricos e envio de ordens;
  * API WebSocket ou equivalente para dados em tempo real;
  * Documentação de filtros de mercado (lotes mínimos, notional, timeframes, etc.).

A qualidade dos dados da exchange é considerada "suficiente", com outliers e gaps gerenciáveis por lógicas de saneamento.

#### 2.3.3. Perfil de Uso

* O sistema é utilizado para **trading proprietário**, não configurado inicialmente para:

  * Gerir capital de terceiros;
  * Prestar serviços de investimento para clientes finais.

* As janelas de operação visadas são:

  * **24/7** para cripto;
  * Estratégias em timeframes de intraday (minutos) até swing/médio prazo (dias/semanas).

---

### 2.4. Critérios de Sucesso (KPIs de Projeto)

Para avaliar se o sistema está cumprindo seu objetivo, são definidos alguns **indicadores-chave de performance (KPIs)** em duas dimensões: **técnica** e **de trading**.

#### 2.4.1. KPIs Técnicos

* **Uptime do Sistema**

  * % do tempo em que o core loop está operacional, com conexão válida à exchange e módulos principais ativos.

* **Taxa de Erros Críticos**

  * Número de incidentes que exigem intervenção manual (ex.: queda completa, falhas de reconciliação, bugs de risco/trading) por mês.

* **Latência de Decisão**

  * Tempo médio entre o fechamento de um candle relevante e o envio das ordens correspondentes.

* **Cobertura de Testes e Backtests de Regressão**

  * % de módulos críticos cobertos por testes automatizados;
  * Suite de backtests padrão executada em mudanças de versões de estratégia.

#### 2.4.2. KPIs de Trading e Risco

Não se espera que o sistema “garanta lucro”, mas há métricas-alvo que indicam se a **arquitetura + processo + famílias de estratégias** estão no caminho desejado:

* **Retorno Ajustado ao Risco**

  * Sharpe/Sortino/Calmar agregados por família e para o portfólio como um todo, comparados a benchmarks simples (buy & hold, por exemplo).

* **Controle de Drawdown**

  * Frequência e severidade de violação de limites de DD;
  * Quantidade de ativações de circuit breakers globais.

* **Aderência entre Backtest/Paper/Live**

  * Grau de coerência entre:

    * Resultados de backtest;
    * Desempenho observado em paper trading;
    * Desempenho real em produção, após ajustes razoáveis de custos/slippage.

* **Diversificação entre Famílias**

  * Correlação entre P&Ls de famílias distintas;
  * Capacidade do portfólio de manter performance aceitável em diferentes regimes de mercado (não depender de uma única abordagem).

---

Em conjunto, estes **Requisitos e Premissas do Sistema** estabelecem a base sobre a qual a arquitetura, o desenvolvimento e a operação serão conduzidos, garantindo que o framework evolua de forma coerente com o objetivo central: **usar famílias de estratégias robustas para buscar retorno ajustado ao risco, com disciplina, controle e capacidade de evolução contínua.**

---

## 3. Arquitetura Geral

A arquitetura do sistema foi desenhada para rodar em **single node**, com uso eficiente de recursos, mas já preparada para suportar **multi-exchange** e **múltiplas famílias de estratégias** com o mínimo de acoplamento possível.
A prioridade é manter o **domínio de trading independente da infraestrutura**, permitindo evoluir conectores de exchange, formas de armazenamento, modelos de risco e camadas de observabilidade sem reescrever as estratégias.

---

### 3.1. Visão de Alto Nível

Em alto nível, o sistema é um **motor de orquestração de estratégias** que:

1. **Coleta e normaliza dados de mercado** (candles, trades, book de ofertas, se disponível);
2. **Enriquece os dados** com indicadores técnicos e features específicas por família de estratégia;
3. **Entrega esses dados enriquecidos às estratégias**, organizadas em famílias (TSM, Dual Momentum, Mean Reversion etc.);
4. **Recebe sinais de posição alvo** (por exemplo: “BTCUSDT spot, 100% alocado” ou “zerar posição”);
5. **Passa os sinais pelo módulo de risco e alocação de capital**, que pode aceitá-los, reduzi-los ou rejeitá-los;
6. **Transforma sinais aprovados em ordens concretas** via adaptadores de exchange;
7. **Persiste todo o histórico de decisões e resultados**, alimentando backtests, monitoramento de performance e governança.

Fisicamente, tudo roda em um único nó (servidor, máquina local ou SBC como Raspberry Pi), em um único processo principal, com possíveis workers internos assíncronos para tarefas de I/O e agendamento.
Logicamente, o sistema é dividido em camadas bem definidas para que:

* O **código de domínio** (estratégias, risco, entidades) não dependa de detalhes de Binance, SQLite, Prometheus etc.;
* A **troca de exchange** seja feita trocando apenas o adapter, mantendo o mesmo conjunto de portas (interfaces);
* A **evolução para novas famílias de estratégia** seja incremental, plugando novos módulos sem quebrar a infraestrutura.

---

### 3.2. Estilo Arquitetural

O sistema adota uma combinação de:

* **Arquitetura Hexagonal (Ports & Adapters)**
* **Clean Architecture / Camadas em Anéis**
* **DDD leve focado em trading algorítmico**

---

### 3.3. Componentes Principais

Abaixo, os principais blocos lógicos da arquitetura:

1. **Orquestrador / Scheduler de Estratégias**

   * Responsável por coordenar quando cada estratégia roda (por timeframe, família, prioridade).
   * Enfileira tarefas de: coleta de dados, execução de sinais, flush de métricas, backtests agendados etc.
   * Implementação single-node, possivelmente sobre `asyncio` e filas in-process.

2. **Módulo de Domínio de Estratégias**

   * Contém as **famílias de estratégias** (TSM, Dual Momentum, Cross-Sectional, Carry, Value/Quality, Mean Reversion, Market Making).
   * Cada família expõe uma interface padrão:

     * `prepare(data_context)`, `generate_signals()`, `post_trade_update()`.

3. **Camada de Dados de Mercado (Market Data Layer)**

   * Implementa a `MarketDataPort`:

     * Subscribe/stream de candles via WebSocket;
     * Fetch de histórico via REST;
     * Acesso a dados locais (SQLite/Parquet) para backtests.
   * Responsável por **normalizar** campos (timestamps, OHLCV, símbolos, fuso horário) e detectar buracos ou inconsistências.

4. **Camada de Execução de Ordens (Trading Layer)**

   * Implementa a `TradingPort`:

     * Envia ordens spot (market, limit, stop-limit, se suportado);
     * Consulta open orders, posições, histórico de fills;
     * Trata erros de rede, rate limits, estados desconhecidos.
   * Inicialmente terá um **adapter para Binance Spot**, mas a interface é genérica para suportar outras exchanges.

5. **Módulo de Risco e Alocação de Capital**

   * De forma desacoplada da estratégia, decide:

     * Quanto capital alocar por família, por estratégia, por ativo;
     * Limites de risco por trade, por dia, por drawdown;
     * Quando ativar circuit breakers (pausar todas as estratégias ou famílias específicas).
   * Atua como **filtro** entre “sinal desejado” e “ordem real”.

6. **Módulo de Persistência e Estado**

   * Implementa a `StoragePort` para:

     * Persistência de candles, trades, ordens, P&L, métricas de performance por estratégia/família;
     * Armazenamento de configurações e metadados (config corrente, versões de estratégia, parâmetros).
   * Em single node, usa banco local (tipicamente SQLite + arquivos Parquet/CSV) com esquema versionado.

7. **Módulo de Backtest & Research**

   * Reutiliza o **mesmo domínio de estratégias e módulo de risco**, mas com:

     * Fonte de dados histórica;
     * Adapter de trading simulado;
     * Motor de execução offline (sem integrações externas).
   * Permite testar novas famílias de estratégias, parâmetros e combinações de portfólio antes da promoção para produção.

8. **Módulo de Observabilidade**

   * Camada transversal para logs estruturados, métricas, alertas e auditoria.
   * Expõe interfaces para:

     * `MetricsPort` (métricas de sistema e de trading),
     * `NotificationPort` (alertas em Telegram/email),
     * trilhas de auditoria (decisões de risco, mudanças de alocação, ativações de circuit breaker).

9. **Módulo de Governança & Configuração**

   * Coordena a configuração declarativa via YAML/JSON (por família, por estratégia, por exchange).
   * Garante versionamento e histórico de mudanças.
   * Suporta workflows de:

     * “cadastrar família de estratégia” → “backtest” → “paper trading” → “produção”.
   * **Deploy Atômico e Feature Flagging**: Deve suportar a ativação/desativação instantânea de estratégias ou famílias em produção via `feature flags` ou mecanismos de `deploy atômico` (ex: Blue-Green), permitindo rollbacks rápidos e seguros sem a necessidade de restart do sistema ou alteração de código.

---

### 3.4. Fluxo Geral (Core Loop)

O **core loop** é o coração do sistema e deve ser o mais simples e previsível possível, tanto em produção quanto em backtest. Em termos gerais:

1. **Agendamento**

   * O scheduler acorda em função do timeframe e da família de estratégia (ex.: TSM diário, Mean Reversion a cada minuto, Dual Momentum a cada fechamento de candle de 4h).

2. **Aquisição e Atualização de Dados**

   * O módulo de Market Data busca ou recebe o último candle/tick;
   * Atualiza o storage local (para garantir histórico consistente);
   * Normaliza timestamps, volumes, símbolos.

3. **Enriquecimento de Dados**

   * Funções de “data enrichment” calculam indicadores e features necessárias por família:

     * EMAs, ADX, retornos acumulados, rankings cross-section, volatilidade, spreads etc.
   * São respeitados os **lookbacks mínimos** para não gerar sinais com dados incompletos.

4. **Execução de Estratégias**

   * O Orquestrador entrega o contexto de dados para cada estratégia habilitada na família correspondente;
   * Cada estratégia decide **posição alvo** (por exemplo: 0%, 50% ou 100% do capital designado para aquele símbolo);
   * As estratégias não criam ordens diretamente, apenas descrevem intenções.

   * **Verificação de Dados Suficientes**: Antes da execução da estratégia, o Core Loop deve garantir que o `DataContext` fornecido atende aos `lookbacks mínimos` declarados pela estratégia. Se não, a execução é abortada para aquele ativo/timeframe e um alerta é gerado.
5. **Aplicação de Risco e Alocação**

   * O módulo de risco recebe a posição alvo e confere:

     * Limites de exposição, limites de DD, limites diários de perda, restrições de liquidez;
     * Regras específicas por família (por exemplo, TSM recebe prioridade em momentos de crise).
   * Se aprovado, o módulo de risco gera um “plano de execução” (quanto comprar/vender, em qual preço/tipo de ordem).

6. **Geração e Envio de Ordens**

   * O Trading Layer transforma o plano de execução em ordens concretas (Binance Spot no MVP);
   * Envia as ordens, trata respostas, e registra ordens abertas e fills;
   * Em backtest, o mesmo fluxo passa pelo adapter simulado, que calcula o fill com base nos dados históricos.

7. **Persistência, Métricas e Auditoria**

   * Todas as etapas (dados de entrada, sinais, decisões de risco, ordens, P&L resultante) são persistidas;
   * Métricas e logs estruturados são atualizados (tempo de decisão, latência até execução, slippage observado, drawdown corrente);
   * Essa trilha permite reconstruir a “linha do tempo” de qualquer decisão.

---

### 3.5. Estratégia de Evolução

*   **Escalabilidade Vertical (MVP)**: Otimização de CPU/RAM/IO no nó.
*   **Suporte Multi-Exchange**: A adição de novas exchanges requer apenas a implementação de novos adapters para as portas `MarketDataPort` e `TradingPort`, sem alterar o domínio.
* **Evolução de Famílias de Estratégias**

  * Novas famílias podem ser incluídas como novos módulos de domínio, registradas no orquestrador e no módulo de governança;
  * Reaproveitam todo o pipeline: dados, risco, execução, observabilidade.

* **Possível Evolução para Multi-Node (Futuro)**

  * A clara separação entre domínios, portas e adapters permite, no futuro, extrair componentes (por exemplo, Market Data ou Backtest) para processos ou nós separados, caso o volume de dados cresça;
  * Como o core loop já é pensado em termos de casos de uso bem definidos, é possível encapsulá-los em serviços sem quebra conceitual.

* **Resiliência e Segurança**

  * Estratégias de fallback (exchange → cache → simulado) e mecanismos de “auto-healing” podem ser progressivamente incorporados, seguindo o padrão já utilizado em projetos como ZeroPulse;
  * Segurança por design: credenciais de exchange fora do código, uso de `.env`/vault, e princípio do menor privilégio para APIs.

Em resumo, a Arquitetura Geral garante que:

* O sistema funciona bem **hoje** em um único nó operando Binance Spot;
* Mas está pronto para crescer em **largura** (mais famílias e estratégias) e em **extensão** (mais exchanges, mais mercados) sem perda de clareza nem explosão de complexidade.

---

## 4. Universo de Mercados, Exchanges e Dados

O universo de mercados define **onde** o sistema pode operar (exchanges, pares, ativos) e **com que granularidade de tempo** (timeframes). A filosofia aqui é tratar a **API da exchange como “fonte única da verdade” (SSOT)** para tudo que é estrutural: símbolos existentes, filtros de lote e notional mínimo, timeframes suportados, tipos de ordem, limites de rate limit etc.

A partir dessa “verdade” vinda da exchange, o sistema realiza **schema discovery** (descoberta do esquema/capacidades em tempo de execução) e ativa/desativa funcionalidades via **feature flags**. Nada crítico (como timeframes, limites de ordem ou tipos de ordem) é “chutado” em configuração estática sem ser checado contra o que a exchange realmente suporta.

---

### 4.1. Exchanges Suportadas

No MVP, o sistema opera com **uma única exchange**:

* **Binance Spot** como exchange primária.

A arquitetura se baseia em uma **abstração de Exchange** (as portas `MarketDataPort` e `TradingPort`), que define um contrato via `Protocol` Python.

```python
class ExchangeInterface(Protocol):
    # Métodos para obter informações, dados de mercado, gerenciar ordens, etc.
    async def get_symbols(self) -> list[Symbol]: ...
    async def get_ohlcv(self, symbol: str, timeframe: str, limit: int) -> pd.DataFrame: ...
    async def submit_order(self, order: OrderRequest) -> OrderResult: ...
    # ... (demais métodos conforme especificação completa)
```

Cada exchange concreta (por exemplo, `BinanceExchangeAdapter`) implementa essa porta usando sua API própria (REST/WebSocket). A configuração apenas seleciona **qual adapter** será instanciado e, eventualmente, quais recursos dessa exchange serão habilitados por feature flag (ex.: permitir apenas mercado spot, bloquear margens, bloquear derivativos).

Para futuras exchanges (Kraken, Coinbase, etc.), o fluxo será:

1. Implementar um novo adapter que realize o **schema discovery** e normalize os dados para o modelo interno;
2. Registrar as capacidades da nova exchange (market data, tipos de ordem, timeframes) em uma estrutura interna;
3. Habilitar a exchange por **feature flag** (por exemplo: `features.exchange.kraken.enabled = true`) e definir quais famílias de estratégias podem operá-la.

---

### 4.2. Abstração Multi-Exchange

Para garantir o desacoplamento, o sistema utiliza um padrão **Factory** para instanciar os adapters de exchange e um **Manager** para orquestrá-los. O domínio nunca interage diretamente com uma implementação concreta.

*   **ExchangeFactory:** Um componente responsável por criar a instância correta do adapter da exchange com base na configuração.
    ```python
    class ExchangeFactory:
        @staticmethod
        def create(exchange_id: str, config: dict) -> ExchangeInterface:
            if exchange_id == "binance_spot":
                return BinanceSpotAdapter(api_key=..., api_secret=...)
            # ... outras exchanges
            raise ValueError(f"Exchange {exchange_id} não suportada.")
    ```

*   **Identificadores de Mercado Padronizados:** O domínio opera com identificadores abstratos (ex: `ExchangeId`, `SymbolId`, `MarketId`).

* **Identificadores de mercado padronizados** (por exemplo, `ExchangeId`, `SymbolId`, `MarketId`);
* Interfaces genéricas de:

A camada de abstração mapeia símbolos nativos para um formato padronizado e expõe as capacidades da exchange (ex: tipos de ordem) via feature flags, permitindo que uma mesma estratégia opere em múltiplas exchanges.

* **Criptomoedas em mercado spot**, em pares como `BTCUSDT`, `ETHUSDT`, `SOLUSDT` etc.

Mesmo restrito a cripto spot, o sistema já foi pensado para:

* Representar explicitamente **classes de ativos** (Crypto, FX, Equities, Commodities, Índices);
* Definir **tipos de mercado** (Spot, Margin, Futures, Perpetuals), ainda que no MVP apenas **Spot** esteja habilitado.

Os símbolos são descritos por uma entidade interna, por exemplo `MarketDescriptor`, que contém:

* `exchange_id`;
* `symbol_raw` (como a exchange expõe, ex.: `BTCUSDT`);
* `symbol_normalizado` (modelo interno, ex.: `BTC/USDT`);
* `base_asset` e `quote_asset`;
* Metadados obtidos via schema discovery:

  * `min_qty`, `step_size`, `min_notional`, `price_tick_size`, etc.;
  * `status` (trading, paused, delisted).

O sistema só considera para uso real **símbolos em estado válido** (trading ativo, não suspenso, com filtros coerentes). A ativação de cada símbolo (por família de estratégia) é feita por:

* **configuração declarativa** (ex.: quais mercados a família TSM pode usar);
* combinada com **feature flags derivadas do schema discovery** (ex.: bloquear símbolos com liquidez insuficiente ou com filtros muito restritivos).

* **Filtro de Liquidez Mínima**: O sistema deve aplicar um filtro de liquidez mínimo (ex: `volume médio diário > 1,000,000 USDT`) como regra de negócio. Este filtro é verificado pelo `Schema Discovery` e usado para habilitar/desabilitar mercados via `Feature Flags`, impedindo que estratégias operem em ativos ilíquidos que poderiam causar slippage excessivo.

---

### 4.4. Timeframes Suportados

Os timeframes suportados são definidos pela exchange, não pelo sistema. O adapter da exchange consulta os intervalos de candles disponíveis, os mapeia para uma enum interna (`TimeframeDescriptor`) e os marca como suportados. As estratégias escolhem entre os timeframes válidos, e qualquer configuração inválida é rejeitada. A seleção pode ser governada por feature flags para limitar os timeframes utilizáveis.

---

### 4.5. Fontes de Dados

O sistema trabalha com três grandes tipos de fonte de dados:

1. **Dados de Mercado ao Vivo (Real-Time)**

   * Obtidos via **WebSocket** da exchange sempre que possível (candles agregados, trades, ou mesmo book de ofertas se necessário para determinadas famílias, como Market Making).
   * São usados para:

     * Atualizar o estado corrente das estratégias;
     * Coletar métricas em tempo real (volatilidade intraday, volume recente, spreads).

2. **Dados Históricos (Backfill e Backtest)**

   * Obtidos via **REST API** da exchange (por ex.: `klines`, `aggTrades`, `trades`).
   * São utilizados para:

     * Alimentar o armazenamento local e garantir histórico mínimo para indicadores (lookback);
     * Construir datasets de backtest para validação de estratégias.

3. **Dados Derivados Internos**

   * Combinação entre dados brutos e transformações internas:

     * Candles agregados de trades, quando necessário;
     * Indicadores técnicos (EMAs, ADX, retornos acumulados, rankings cross-section, volatilidade, etc.);
     * Sinais intermediários, scores de risco, labels para análises posteriores.

As famílias de estratégias **nunca acessam diretamente fontes externas**; elas recebem um **DataContext** já normalizado e enriquecido, isolando-as de detalhes de WebSocket, REST ou estrutura de banco.

---

### 4.6. Normalização, Qualidade e Validação de Dados

Como o sistema pode operar em múltiplas exchanges e, possivelmente, em múltiplas classes de ativos no futuro, a **normalização de dados** é essencial:

* **Normalização de Timestamps**

  * Todo dado de mercado é convertido para um fuso padrão (tipicamente UTC) e guardado como `datetime` bem definido, evitando confusões de timezone ou horário de verão.
  * A granularidade mínima (segundos/milissegundos) é preservada conforme necessário.

* **Normalização de Símbolos**

  * Cada exchange pode ter convenções diferentes (`BTCUSDT` vs `BTC-USDT` vs `XBTUSD`); o sistema mapeia isso para um formato interno unificado (`base/quote` com identificadores próprios).

* **Normalização de Precisão Numérica**

  * Preços e quantidades são representados com precisão suficiente (decimal/float de alta precisão) para evitar erros de arredondamento que violem filtros da exchange.

* **Validação de Integridade de Candles**

  * Verificação de buracos de candle (gaps temporais) e correções automáticas, quando possível;
  * Verificação de consistência entre `open`, `high`, `low`, `close`, `volume`;
  * Regras de rejeição ou marcação de “bad data” para períodos anômalos.

* **Política de Saneamento**: O sistema implementa uma política de saneamento explícita. Dados marcados como `BAD` (ex: gaps irrecuperáveis) impedem a execução da estratégia para o ativo afetado e geram alertas. Dados `QUESTIONABLE` (ex: outliers) são logados, e a estratégia pode prosseguir usando o último dado válido conhecido, com a incerteza registrada na trilha de auditoria.

#### 4.6.1. Regras de Validação e Tratamento de Gaps

* **Validações de Candle**:
  * `high >= max(open, close) >= low` (com uma pequena tolerância para dados ruidosos).
  * `volume >= 0`.
  * Timestamps devem seguir a sequência esperada para o timeframe, sem duplicações.
* **Tratamento de Gaps**:
  * **Gaps Pequenos (< 2 candles)**: O sistema pode tentar preencher o dado faltante com a informação do candle anterior (`forward-fill`) e marcar o dado como `QUESTIONABLE`.
  * **Gaps Grandes (>= 2 candles)**: O período é marcado como `MISSING` ou os candles como `BAD`, e um alerta é gerado. Estratégias são configuradas para decidir se pausam ou ignoram o ativo durante esse período.
* **Thresholds de Qualidade**:
  * **Detecção de Spikes**: Um movimento de preço que excede um múltiplo configurado do ATR (ex: `> 10 * ATR(20)`) marca o candle como `QUESTIONABLE`.
  * **Volume Anômalo**: Períodos com volume zero ou muito abaixo da média recente podem gerar um `WARNING` e marcar os dados como `QUESTIONABLE`.


* **Qualidade e Saneamento**

  * Regras de **data quality** podem marcar determinados intervalos como “não confiáveis” (por exemplo, durante falhas reconhecidas de exchange);
  * Famílias de estratégias podem optar por ignorar ou tratar de forma especial esses períodos.

Todo esse pipeline é descrito por um **schema interno de dados**, que define campos obrigatórios, tipos, possibilidades de `null`, e é aplicado de forma consistente em:

* Coleta ao vivo;
* Backfill histórico;
* Leitura de dados para backtest.

---

### 4.7. Persistência de Dados

A persistência de dados é projetada para um **ambiente single node**, priorizando:

* **Simplicidade operacional** (sem necessidade de cluster de banco);
* **Reprodutibilidade** (facilidade para reconstruir séries históricas no mesmo nó ou em outro ambiente);
* **Desempenho adequado** para volumes típicos de cripto spot.

As decisões principais são:

* Uso de **SQLite** para dados transacionais e de estado:

  * Metadados de mercados (lista de símbolos, filtros, timeframes suportados);
  * Estado operacional (posições, ordens, trades);
  * Registros de execução (sinais, decisões de risco, P&L agregado) e eventos de auditoria.

* Uso de formatos de arquivo do tipo **Parquet/CSV** (ou similar) para:

  * Séries históricas de candles por `exchange/symbol/timeframe`;
  * Armazenamento de séries derivadas usadas em pesquisas e backtests (indicadores, scores, labels).

* Em fases mais maduras (Fase 4+ do roadmap), para análises mais pesadas, pode-se introduzir **DuckDB** para consultas analíticas sobre os arquivos Parquet, combinando a simplicidade do SQLite para o estado transacional com a performance do DuckDB para analytics.

A organização típica é:

* Particionamento por:

  * `exchange_id`;
  * `symbol`;
  * `timeframe`;
  * janelas de tempo (por exemplo, ano/mês).
* Convenção de nomes consistente, permitindo localizar rapidamente:

  * `market_data/{exchange}/{symbol}/{timeframe}/YYYY-MM.parquet`.

Essa estrutura facilita:

* Reutilização de dados em múltiplos backtests;
* Inspeção manual para debug;
* Versão e migração de schema (novos campos podem ser adicionados mantendo compatibilidade com o formato atual).

---

### 4.8. Schema Discovery e Feature Flags

Um pilar do projeto é tratar a **API da exchange como fonte única da verdade (SSOT)** para o universo de mercados. Isso se concretiza em dois mecanismos centrais:

1. **Schema Discovery**

   * Na inicialização (e em ciclos periódicos), o sistema consulta a exchange para descobrir:

     * Quais símbolos existem e seus filtros (`min_qty`, `min_notional`, `step_size`, `price_tick_size`…);
     * Quais mercados estão em status `TRADING` vs `HALTED/DELISTED`;
     * Quais timeframes e endpoints de market data estão disponíveis;
     * Quais tipos de ordem podem ser usados (market, limit, stop-limit, OCO…).
   * Esses dados são traduzidos para um **schema interno versionado**, armazenado em SQLite/Parquet, e exposto ao restante do sistema como:

     * `MarketCapabilities`;
     * `TimeframeCapabilities`;
     * `OrderCapabilities`.

2. **Feature Flags**

   * Sobre esse schema descoberto, o sistema aplica **regras de habilitação** em diversos níveis:

     * Por exchange (`features.exchange.binance.enabled`);
     * Por mercado (`features.market.BTCUSDT.enabled`);
     * Por timeframe (`features.timeframe.1m.enabled`);
     * Por família de estratégia (`features.strategy_family.tsm.allowed_exchanges`).
   * Os feature flags podem:

     * Bloquear mercados considerados ilíquidos ou perigosos;
     * Limitar timeframes a um subconjunto “oficialmente suportado” pelo projeto;
     * Habilitar apenas algumas famílias de estratégias em uma exchange nova até que ela seja totalmente validada.

---
### 4.8.1. Modelo de Dados (Entidades Core)

As entidades de dados são definidas usando modelos tipados (Pydantic) para garantir consistência e validação.

```python
# Exemplo de definição de entidade com Pydantic
from enum import Enum
from datetime import datetime
from decimal import Decimal
from pydantic import BaseModel

class DataQuality(str, Enum):
    GOOD = "GOOD"
    QUESTIONABLE = "QUESTIONABLE"
    BAD = "BAD"

class Candle(BaseModel):
    exchange_id: str
    symbol: str
    timeframe: str # Ou um Enum de Timeframe
    timestamp: datetime
    open: Decimal
    high: Decimal
    low: Decimal
    close: Decimal
    volume: Decimal
    data_quality: DataQuality = DataQuality.GOOD

class Position(BaseModel):
    strategy_id: str
    symbol: str
    quantity: Decimal
    avg_entry_price: Decimal
```

Com isso, o “Universo de Mercados, Exchanges e Dados” deixa de ser um conjunto de constantes hardcoded e passa a ser uma **camada viva, descoberta e validada em tempo de execução**, sobre a qual as famílias de estratégias podem operar com segurança, robustez e previsibilidade.

---

## 5. Framework de Estratégias e Famílias

O framework de estratégias é o “coração lógico” do sistema: é nele que as **famílias de estratégias** (Trend Following, Dual Momentum, Cross-Sectional, Carry, Value/Quality, Mean Reversion, Market Making) são definidas, versionadas, configuradas e orquestradas.
O objetivo é garantir que todas as estratégias, independentemente da família, sigam um **contrato comum**, possam ser **testadas de forma isomórfica** (mesma lógica em backtest e produção) e sejam facilmente ativadas/desativadas por configuração, sem alterações em código de infraestrutura.

---

### 5.2. Interface de Estratégia (Strategy API)

Todas as estratégias, independentemente da família, implementam uma interface comum, a **Strategy API**, que define o contrato mínimo para interação com o orquestrador.

Em termos conceituais, uma estratégia expõe, no mínimo:

* Um método de **preparação**:

  * Responsável por validar a configuração, declarar os requisitos mínimos de dados (lookback, indicadores, features), e inicializar qualquer estado interno necessário.
    Exemplo: `prepare(strategy_config, market_context, data_schema)`.

* Um método de **execução principal** (core de decisão):
  * **Declaração de Requisitos de Dados**: Cada estratégia deve declarar explicitamente seus requisitos de dados, incluindo `lookbacks mínimos` para cada indicador ou feature utilizada.


  * Recebe um contexto de dados (por exemplo, um DataFrame ou estrutura equivalente) com candles recentes, indicadores e metadados de mercado.
  * Devolve um conjunto de **sinais de posição alvo** por mercado (por exemplo, “para BTCUSDT: 100% alocado”, “para ETHUSDT: zeroar”).
    Exemplo: `generate_signals(data_context) -> list[Signal]`.

* Um método de **atualização pós-trade**:

  * Permite que a estratégia seja informada sobre o resultado das ordens (fills, execuções parciais) e atualize qualquer estado interno dependente de execução real (por exemplo, para estratégias que consideram preço médio de entrada).
    Exemplo: `post_trade_update(trade_events)`.

Além disso, a interface pode incluir:

* Métodos para **expor diagnósticos** (por exemplo, scores internos, indicadores chave, sinais intermediários), que alimentam o módulo de observabilidade;
* Métodos para **relatar requisitos de dados** (por exemplo, lookback mínimo em barras para cada timeframe, conjunto de indicadores necessários), permitindo que a camada de dados prepare o contexto de forma adequada.

Essa interface é implementada por classes concretas de cada família, como `TSMStrategy`, `DualMomentumStrategy`, `MeanReversionIntradayStrategy`, etc., todas derivadas de uma **classe base abstrata** (`BaseStrategy`).

---

### 5.3. Ciclo de Vida de uma Estratégia

O ciclo de vida de uma estratégia dentro do sistema segue etapas claras e auditáveis:

1. **Definição Conceitual (Design da Família)**
   A família de estratégia é descrita em documento próprio (hipótese, evidência, riscos, métricas desejadas).
   Essa descrição é a base para implementação e para avaliação futura.

2. **Implementação Técnica (Classe da Família)**
   A família é codificada como uma classe que implementa a Strategy API, usando apenas dependências de domínio (sem acoplamento direto a exchange, banco, etc.).

3. **Configuração de Instâncias**
   Com base na família, são criadas **instâncias de estratégia** definidas em arquivos de configuração (YAML/JSON):

   * Quais mercados/pares serão usados;
   * Quais timeframes;
   * Quais parâmetros (por exemplo, janelas de lookback, thresholds de retorno, filtros adicionais);
   * Quais limites de risco locais (por estratégia).

4. **Backtest Inicial (Research)**
   Cada instância é testada com dados históricos, usando o motor de backtest unificado.
   Os resultados (retorno, Sharpe, drawdown, estabilidade por regime de mercado) são analisados e documentados.
   Eventuais ajustes de parâmetros são feitos nessa fase.

5. **Revisão e Aprovação (Governança)**
   Um pipeline de aprovação (formal ou leve, conforme maturidade) decide se a instância pode progredir para paper trading.
   A decisão é documentada (melhorias esperadas, riscos assumidos).

6. **Paper Trading (Ambiente Simulado em Tempo Real)**
   A instância é executada em tempo real, porém com ordens simuladas, para avaliar comportamento sob latência real, buracos de dados, falhas de exchange etc.
   Problemas operacionais são identificados e corrigidos antes de qualquer risco financeiro real.

7. **Produção (Live Trading)**
   Após passar pelo pipeline de aprovação, a instância é promovida para produção, com capital real.
   O módulo de risco global define a alocação de capital, e a instância é monitorada de perto em termos de performance e drawdown.

8. **Monitoramento Contínuo e Manutenção**
   A performance da estratégia é acompanhada ao longo do tempo. Se a instância entra em regime de subperformance prolongada ou quebra limites de risco, pode ser:

   * Desativada;
   * Rebaixada para paper trading;
   * Reconfigurada com novos parâmetros (gerando uma nova versão).

Esse ciclo de vida é comum a todas as famílias, o que facilita padronizar relatórios, auditorias e decisões de mudança.

---

### 5.4. Modelo de Configuração

Toda estratégia é configurada de forma **declarativa**, com base em arquivos de configuração validados por modelos tipados (Pydantic ou equivalente). Isso garante que:

* Configurações inválidas sejam pegas no momento de carga (e não apenas em runtime);
* Seja possível versionar e comparar configurações de forma rastreável.

O modelo de configuração típico inclui:

* **Identificação da Estratégia e Família**

  * `family`: nome da família (TSM, DualMomentum, etc.);
  * `strategy_id`: identificador único daquela instância específica (ex.: `TSM_BTCUSDT_1D_v1`).

* **Contexto de Mercado**
* **Versão e Modo de Operação**
  * `version`: "1.0.0" (seguindo versionamento semântico).
  * `mode`: `backtest` | `paper` | `live`.

* **Contexto de Mercado**

  * `markets`:
    ```yaml
    - exchange: binance_spot
      symbol: BTC/USDT
    ```
  * `timeframe`: `1d`.

* **Parâmetros da Estratégia**
  * `parameters`:
    ```yaml
    lookback_window: 200
    threshold: 0.0
    vol_window: 20
    ```

  * Lista de `markets` (por exemplo, `["BINANCE:BTCUSDT", "BINANCE:ETHUSDT"]`);
  * `timeframe` principal e, se aplicável, timeframes auxiliares;
  * Requisitos de liquidez mínima, spread máximo, volume mínimo.

* **Parâmetros da Família**

  * Por exemplo, para TSM: janelas de lookback (3, 6, 12 meses), regras de entrada/saída, thresholds de sinal, filtros adicionais (volatilidade mínima, volatilidade máxima, etc.).

* **Políticas de Risco Locais**

  * `risk`:
    ```yaml
    max_position_pct: 0.30  # 30% do capital da estratégia
    max_daily_loss_pct: 0.02
    ```
  * Prioridade relativa da instância no escalonador;
  * Flags de debug, logging extra, exportação de sinais.

Ao carregar a configuração, o sistema valida:

* Coerência com o que foi descoberto da exchange (mercados existentes, timeframes suportados);
* Coerência interna de parâmetros (por exemplo, lookbacks não superiores à janela histórica disponível).

Apenas estratégias com configuração válida são habilitadas pelo orquestrador.

---

### 5.5. Integração com Multi-Exchange

O framework de estratégias foi desenhado para que **uma mesma família** possa operar em múltiplas exchanges, sem alteração de código da família. Isso é obtido por:

* Uso de **identificadores abstratos de mercado**, como `MarketId` ou `Exchange:Symbol`, em vez de strings específicas de API;
* Resolução da lista de mercados válidos, timeframes e filtros a partir do módulo de schema discovery;
* Aplicação de **feature flags** para limitar, por família, quais exchanges podem ser usadas em determinado estágio do projeto (por exemplo, inicialmente apenas Binance Spot para TSM).

Estratégias não se preocupam com detalhes como:

* Diferenças de notação de símbolo (`BTCUSDT` vs `BTC-USD`);
* Diferenças de tipos de ordem disponíveis (por exemplo, se stop-limit existe ou não em uma exchange).

Esses detalhes são encapsulados pelos adapters de exchange, que expõem um comportamento homogêneo ao domínio. Quando uma família precisa de capacidades específicas (por exemplo, Market Making em nível de book de ofertas), ela declara isso explicitamente, e o sistema só a habilita para exchanges que atendam a esses requisitos.

---

### 5.7. Métricas de Avaliação de Estratégias

Todas as famílias e instâncias de estratégias são avaliadas sob um conjunto comum de métricas, além de métricas específicas relevantes para cada tipo.

Métricas globais:

* **Retorno Absoluto** e **Retorno Anualizado**;
* **Volatilidade** anualizada;
* **Sharpe Ratio** e **Sortino Ratio** (retorno ajustado ao risco);
* **Calmar Ratio** (relação entre retorno e drawdown máximo);
* **Drawdown Máximo** e **tempo em drawdown**;
* **Hit Ratio** (percentual de trades vencedores) e **Payoff Ratio** (ganho médio / perda média);
* **Turnover** e impacto estimado de custos (taxas + slippage).

Métricas específicas por família podem incluir:

* Para **TSM e Dual Momentum**:
  Performance por regime de mercado (bull, bear, sideways), tempo em posição vs tempo em cash.

* Para **Cross-Sectional Momentum**:
  Sensibilidade a crashes de momentum, concentração de risco em poucos ativos.

* Para **Carry/Yield/Basis**:
  Comportamento em crises de liquidez, eventos de compressão brusca de basis.

* Para **Value/Quality**:
  Horizon de convergência e períodos prolongados de underperformance vs benchmarks.

* Para **Mean Reversion Intraday**:
  Sensibilidade a custos, slippage, mudanças de microestrutura; distribuição de P&L por horário do dia.

* Para **Market Making**:
  P&L de spread vs P&L de inventário, medidas de adverse selection, exposição intra-dia (inventory risk).

Todas essas métricas são produzidas tanto em **backtest** quanto em **produção**, permitindo:

* Comparar se o comportamento real se aproxima do esperado;
* Identificar rapidamente quando uma estratégia entra em regime de degradação;
* Alimentar decisões de governança (refinamento, redução de capital, desligamento, substituição).

---

Com esse framework, o sistema consegue tratar cada família de estratégia como um **módulo de domínio plugável**, com ciclo de vida, critérios de priorização e métricas bem definidos, mantendo ao mesmo tempo **homogeneidade de interface** e **flexibilidade para evoluir** o portfólio de abordagens ao longo do tempo.

---


## 13. Gestão de Risco e Alocação de Capital

A gestão de risco é um módulo central e desacoplado. As estratégias sugerem posições alvo, mas a decisão final de alocação e execução pertence a este módulo, que organiza o risco em camadas hierárquicas para proteger o capital.

1. **Risco por Trade**

   * Define o **máximo de capital arriscado em um único trade** (por exemplo, 0,5% do equity total, ou X% da alocação daquela estratégia).
   * Considera:

     * Tamanho da posição;
     * Distância até o stop (quando aplicável);
     * Volatilidade do ativo (por exemplo, via ATR ou desvio-padrão).

2. **Risco por Estratégia (Instância)**

   * Cada instância de estratégia possui um **“risk budget” próprio**, que limita:

     * Exposição máxima total dessa instância (por exemplo, até 10% do portfólio);
     * Número máximo de posições simultâneas;
     * Perda máxima diária/semanal/mensal exclusiva daquela instância.
   * Se a instância atingir seus limites, ela é automaticamente:

     * Colocada em modo “somente saída” (não abre novas posições); ou
     * Temporariamente desativada, dependendo da política definida.

3. **Risco por Família de Estratégias**

   * Famílias mais robustas (por exemplo, Trend Following / TSM) podem ter um **risk budget maior**; famílias mais frágeis ou experimentais (por exemplo, Mean Reversion intraday, Market Making) recebem menor fração do capital.
   * A soma das exposições de todas as instâncias dentro da mesma família não pode ultrapassar:

     * Um teto de **alocação de capital** (por exemplo, até 40% do portfólio em TSM, até 15% em Mean Reversion);
     * Limites de drawdown específicos da família.

4. **Risco por Ativo / Mercado**

   * Define limites para exposição em ativos individuais (por exemplo, BTCUSDT não pode representar mais que 30% do portfólio total, mesmo somando várias estratégias).
   * Pode incluir:

     * Limites de concentração por ativo;
     * Regras específicas para ativos mais voláteis ou ilíquidos (limites mais conservadores).

5. **Risco por Exchange**

   * Em cenários multi-exchange, ainda que o sistema rode em single node, é possível:

     * Limitar a exposição agregada por exchange (por exemplo, no máximo 60% do patrimônio em uma única exchange para mitigar risco de custódia);
     * Definir políticas de diversificação entre exchanges quando houver suporte.

6. **Risco Global de Portfólio**

   * Camada mais alta, que observa:

     * Drawdown total do sistema;
     * Exposição total por classe de ativo (cripto, FX, ações);
     * Correlação entre famílias.
   * É responsável por acionar **circuit breakers globais** quando o comportamento do portfólio entra em zona crítica.

### 13.2. Limites e Guardrails (Camadas de Risco)

Os **guardrails** são regras numéricas que não podem ser quebradas, funcionando como um “guia de rodovia” para o sistema:

1. **Risco Máximo por Trade**

   * Expressado normalmente como percentual do patrimônio total (ou do capital alocado à família).
   * Exemplo:

     * Nenhum trade pode representar uma perda potencial (até o stop ou cenário de stress) maior que 0,5% do equity global.
   * O cálculo pode usar:

     * Distância até o stop (em %);
     * Volatilidade do ativo (em ATR ou desvio-padrão).

2. **Drawdown Máximo Diário/Mensal**

   * Definido tanto em nível de **estratégia/família** quanto em **portfólio global**:

     * Exemplo: perda diária máxima de 3% do patrimônio; perda mensal máxima de 10%.
   * Quando o limite diário é atingido:

     * O sistema entra em modo “flat”: fecha posições abertas conforme a política e bloqueia novas entradas até o próximo dia;
     * Ou adota um modo degradado (apenas algumas famílias seguem ativas, por exemplo TSM de maior prazo).

3. **Notional Máximo por Ativo e por Exchange**

   * Limites em valor nominal (por exemplo, em USDT ou moeda de referência) que um ativo ou uma exchange pode concentrar.
   * Exemplo:

     * `max_notional_per_symbol = 30% do equity` para BTCUSDT;
     * `max_notional_per_exchange = 60% do equity` para Binance, se houver outra exchange disponível.

4. **Limites de Liquidez e Tamanho Relativo de Ordem**

   * Guardrails baseados em:

     * Volume médio diário;
     * Profundidade de book, quando disponível.
   * Evita que ordens muito grandes sejam enviadas em ativos de baixa liquidez, reduzindo risco de slippage extremo.

5. **Limites de Número de Posições**

   * Número máximo de posições abertas por:

     * Estratégia;
     * Família;
     * Portfólio completo.
   * Isso impede um estado em que dezenas de posições pequenas tornem o sistema difícil de manejar, gerando custos administrativos e risco operacional.

Esses guardrails são parametrizados e versionados, permitindo ajustes ao longo do tempo sem alteração da lógica central de risco.

---

### 13.3. Alocação entre Famílias

A alocação de capital entre famílias pode ser feita de duas formas principais:
1.  **Regra Fixa (Alocação Estática):** Uma fração do equity é atribuída a cada família com base em critérios como robustez da evidência histórica.
2.  **Regra Dinâmica (Alocação Adaptativa):** A alocação varia com base na performance recente ajustada ao risco, estabilidade e correlação entre famílias.

Independente da regra (fixa ou dinâmica), o módulo de risco mantém:

* **Cap de alocação por família** (ninguém ultrapassa seu teto máximo, mesmo em cenários excepcionais);
* Logs detalhados de decisões de realocação, permitindo reconstruir por que, em determinado dia, uma família teve sua alocação aumentada ou reduzida.

---

### 13.4. Controle de Exposição Cambial (FX vs Cripto vs Fiat)

Mesmo em um contexto primariamente de cripto spot, o sistema precisa de uma **visão consolidada da exposição cambial**, especialmente quando operações envolvem:

* Diversos pares (BTC/USDT, ETH/USDT, BNB/BTC etc.);
* Conversão final para uma moeda de referência (por exemplo, USD ou BRL);
* Eventual inclusão futura de FX e ações.

O módulo de risco acompanha:

1. **Exposição por Moeda Base e Moeda de Cotação**

   * Quanto do patrimônio está exposto a BTC, ETH, stablecoins (USDT, USDC), fiat (USD, EUR, BRL) etc.
   * Considera:

     * Posições spot;
     * Saldos ociosos na exchange;
     * Eventuais yields (staking, juros).

2. **Moeda de Referência de Risco**

   * Todos os limites e métricas de risco são calculados em uma **moeda de referência única** (por exemplo, USD-equivalente), para evitar distorções.
   * Isso inclui:

     * Risco por trade;
     * Drawdown global;
     * Exposição por ativo e família.

3. **Concentração em Cripto vs Fiat**

   * Permite, por política, limitar o quanto do patrimônio total pode ficar:

     * 100% exposto a cripto;
     * ou exigir que um percentual mínimo permaneça em stable/fiat em determinados regimes de mercado.

4. **Cenários de Stress de FX**

   * Para portfólios com múltiplas moedas, podem ser definidos cenários de stress (por exemplo, depreciação rápida de uma moeda) para avaliar impacto e ajustar limites.

Esse controle assegura que o portfólio não fique, por acidente, hiperconcentrado em uma única moeda ou cluster de ativos correlacionados.

---

### 13.5. Circuit Breakers e Modo Seguro

Os **circuit breakers (CBs)** são mecanismos automáticos de proteção que entram em ação quando o sistema detecta situações críticas. O sistema implementa CBs em múltiplos níveis (Estratégia, Família, Global) e um **Modo Seguro** para proteção máxima.

Os procedimentos detalhados de operação e resposta a esses eventos estão documentados no **OPERATIONS_RUNBOOK.md**.

---

### 13.6. Instrumentação para Medição de Risco

A gestão de risco só é eficaz se for **medida, monitorada e auditada continuamente**. Por isso, o sistema inclui uma camada de instrumentação dedicada ao risco, integrada à observabilidade geral.

Elementos principais:

1. **Métricas em Tempo Real**

   * Exposição atual por:

     * Estratégia, família, ativo, exchange, moeda;
   * Risco por trade (estimado) e risco agregado;
   * Drawdown corrente (intradiário, mensal, máximo desde o início);
   * Utilização de “risk budget” por camada (quanto de cada limite já foi consumido).

2. **Eventos de Risco**

   * Registro estruturado de:

     * Ativações de circuit breakers;
     * Rejeição ou redução de ordens por limites de risco;
     * Mudanças de alocação entre famílias;
     * Alterações manuais de parâmetros de risco.
   * Esses eventos formam uma trilha de auditoria que permite responder “por que não entrou naquela operação?” ou “por que a estratégia X foi desligada em tal dia?”.

3. **Relatórios Periódicos**

   * Geração de relatórios diários/semanais/mensais com:

     * Distribuição de P&L por estratégia/família;
     * Evolução de exposição e drawdown;
     * Violação de limites (mesmo se tratadas automaticamente).
   * Esses relatórios podem servir tanto para uso interno (ajustes de parâmetros) quanto para eventual uso regulatório ou apresentação a terceiros.

4. **Dashboards de Risco**

   * Visualizações em tempo real (ou quase real) com:

     * Heatmaps de exposição;
     * Indicadores de “proximidade” a limites (por exemplo, barra de progressão mostrando o quanto falta para atingir o DD máximo mensal);
     * Alertas visuais quando circuit breakers são acionados.

5. **Integração com Notificações**

   * Eventos críticos de risco (por exemplo, disparo de breaker global, perda diária máxima) disparam notificações via e-mail, Telegram ou outro canal definido, garantindo que operadores humanos sejam informados rapidamente.

---

Em conjunto, a “Gestão de Risco e Alocação de Capital” garante que o sistema não seja apenas um conjunto de estratégias independentes, mas um **portfólio coerente e protegido**, no qual:

* Cada família opera dentro de um budget bem definido;
* Choques e regimes adversos sejam absorvidos com mecanismos automáticos de defesa;
* Decisões e eventos de risco sejam observáveis, reprodutíveis e auditáveis.


---

## 14. Backtesting, Simulação e Validação

O módulo de **Backtesting, Simulação e Validação** avalia o comportamento histórico das estratégias, sua robustez estatística e a coerência com a produção. A estrutura de backtest inclui uma fonte de dados históricos padronizada, um motor de execução temporal e configurações de cenário.

   * Candles históricos por `exchange/symbol/timeframe`, obtidos da própria exchange e armazenados em formato local (ex.: Parquet/CSV) com schema consistente.
   * Os mesmos mecanismos de normalização descritos no módulo de dados (timestamps, OHLCV, símbolos, precisão numérica) são aplicados também aos dados usados em backtest.

2. **Motor de Execução Temporal**

   * Um “replay” da linha do tempo, avançando candle a candle (ou tick a tick, quando aplicável), simulando o fluxo real:

     * Entrada de dados → enriquecimento com indicadores → chamada das estratégias → aplicação de risco → geração de ordens → execução simulada → atualização de posições.
   * Esse motor pode operar:

     * Em modo **single-pass** (percorre o período uma única vez);
     * Em múltiplas execuções (por exemplo, para grid search de parâmetros ou otimizações mais elaboradas).

3. **Configurações de Cenário**

   * Cada backtest é descrito por uma configuração explícita:

     * Período histórico (datas de início/fim);
     * Universo de mercados/timeframes;
     * Estratégias/instâncias ativas;
     * Modelo de custos e slippage;
     * Parâmetros específicos de risco (guardrails, circuit breakers, limites de alocação).

Essa estrutura permite reproduzir com fidelidade o comportamento operacional do sistema, respeitando os mesmos limites de risco e a mesma lógica de orquestração usada em produção, apenas substituindo a origem dos dados (histórico em vez de stream ao vivo) e o executor (simulado em vez de exchange real).

---

### 14.2. Engine de Backtest Unificado

O sistema utiliza um **engine de backtest unificado** que reutiliza a mesma `Strategy API`, módulo de risco e gestão de estado da produção. A única diferença é o `TradingPortSimulado`, que simula a execução de ordens com base em dados históricos.

---

### 14.3. Modelagem de Custos e Slippage

Um backtest sem modelagem adequada de custos tende a ser **irrealisticamente otimista**, especialmente para estratégias intraday ou sensíveis à microestrutura. Por isso, o sistema inclui um modelo flexível de:

1. **Custos de Transação (Taxas/Fee)**

   * Por exchange, por tipo de ordem e por par de ativos, considerando:

     * Taxas maker/taker padrão;
     * Possíveis descontos (por exemplo, uso de token da exchange, níveis de volume).
   * Os custos são aplicados a cada execução (fill), impactando o P&L de forma granular.

2. **Slippage**

   * A diferença entre o preço teórico da ordem e o preço efetivo de execução é modelada de forma configurável, com opções como:

     * **Slippage fixo** em pontos base (bps) sobre o preço de execução;
     * Slippage proporcional à **volatilidade do ativo** naquele período;
     * Slippage baseado em **participação de volume** (ex.: se a ordem for grande em relação ao volume médio do candle, aplicar um custo maior).
   * Em cenários avançados, o modelo pode considerar a proximidade do preço de entrada com as extremidades do candle (high/low), evitando execuções “perfeitas” no pior/ melhor preço.

3. **Limitação por Liquidez**

   * Em ativos de baixa liquidez, o simulador pode limitar:

     * O tamanho máximo executável em um candle;
     * A velocidade de execução (dividindo ordens grandes ao longo de múltiplos candles).
   * Isso evita backtests onde uma estratégia executo volumes irreais sem impacto no mercado.

Todos esses elementos são configuráveis por cenário de backtest e podem ser ajustados para diferentes **“níveis de conservadorismo”** (mais ou menos pessimista em relação a custos e slippage).

---

### 14.4. Cenários e Campanhas de Teste

O módulo de backtesting suporta desde **testes pontuais simples** até **campanhas complexas** cobrindo múltiplos regimes de mercado.

1. **Cenários Históricos Específicos**

   * Backtests focados em períodos de interesse:

     * Grandes crises (ex.: quedas abruptas de mercado);
     * Bull markets prolongados;
     * Fases de lateralização.
   * Objetivo: avaliar como a estratégia se comporta em regimes extremos ou particularmente relevantes.

2. **Testes de Regime (Bull / Bear / Sideways)**

   * O histórico pode ser particionado em subperíodos classificados por regime de mercado, e a performance de cada estratégia é analisada separadamente em cada regime.
   * Isso ajuda a responder perguntas como:

     * “TSM perde muito em fases laterais? Quanto?”
     * “Mean Reversion é segura o suficiente em mercados em forte tendência?”

3. **Campanhas de Otimização de Parâmetros**

   * Execução de séries de backtests variando sistematicamente:

     * Lookbacks, thresholds, filtros adicionais;
     * Parâmetros de alocação e de risco.
   * Resultados são consolidados e analisados em termos de:

     * Robustez (sensibilidade a pequenas mudanças de parâmetros);
     * Estabilidade de métricas (Sharpe, DD) entre janelas e regimes.

4. **Ensaios de Portfólio de Famílias**

   * Backtests em que várias famílias de estratégias são executadas simultaneamente, usando o mesmo período e universo de ativos, com alocação de capital definida (fixa ou dinâmica).
   * Permite avaliar:

     * Diversificação entre famílias;
     * Benefícios reais de combinar TSM, Dual Momentum, Mean Reversion etc.

Esses cenários podem ser descritos e versionados em arquivos de configuração próprios, o que facilita a repetição das campanhas com novas versões de código.

---

### 14.5. Validação Estatística

A validação estatística busca diferenciar resultados genuínos de overfitting. Os mecanismos incluem: **Divisão In-Sample / Out-of-Sample (IS/OOS)**, **Walk-Forward Analysis**, **Validação Cruzada Temporal**, **Análise de Robustez de Parâmetros** e **Testes de Significância Simples**.

---

### 14.6. Reprodutibilidade

A reprodutibilidade é tratada como requisito essencial: dado um conjunto de condições (código, dados, configurações), o sistema deve ser capaz de **reproduzir exatamente o mesmo resultado de backtest**.

Para isso, são adotadas as seguintes práticas:

1. **Versionamento de Código e Configurações**

   * Cada execução de backtest referencia:

     * O commit ou versão do código;
     * A versão dos arquivos de configuração (estratégias, risco, campanhas de teste).
   * Isso permite comparar resultados entre versões e identificar mudanças de comportamento devido a alterações de lógica.

2. **Controle de Dados**

   * Datasets históricos usados nos backtests são:

     * Identificados por checksums ou hashes;
     * Armazenados de forma imutável ou versionada (por exemplo, snapshots de mercado até determinada data).
   * Garantia de que, ao repetir um backtest com o mesmo dataset, os dados são exatamente os mesmos.

3. **Seeds e Determinismo**

   * Quando elementos aleatórios são utilizados (por exemplo, em alguma técnica de simulação ou seleção de dados), são usados **seeds fixos** e registrados para reprodutibilidade.
   * O motor de backtest é desenhado para ser determinístico dado o conjunto {dados, parâmetros, seed}.

4. **Registro de Execuções (Run Metadata)**

   * Cada backtest gera um registro com:

     * ID da execução;
     * Data/hora;
     * Estratégias envolvidas;
     * Intervalos de tempo;
     * Versões de código/dados;
     * Principais métricas de resultado.
   * Esse registro permite reabrir um “run” passado, comparar com runs atuais e auditar a evolução da estratégia.

---

### 14.7. Ferramentas de Análise de Resultados

Por fim, o sistema inclui um conjunto de ferramentas para análise e visualização dos resultados de backtests, de forma a tornar a validação mais objetiva e prática.

1. **Relatórios Padronizados de Performance**

   * Para cada backtest (e para o portfólio combinado), são gerados:

     * Tabelas com métricas principais (retorno, Sharpe, Sortino, Calmar, DD, hit ratio, payoff);
     * Quebras por período (mensal, trimestral, anual) e por regime de mercado.

2. **Curvas de Equity e Drawdown**

   * Gráficos de evolução do patrimônio ao longo do tempo;
   * Gráficos de drawdown (profundidade e duração), por estratégia e por portfólio.

3. **Distribuição de Retornos**

   * Histogramas de retornos diários/por trade;
   * Boxplots ou quantis para analisar caudas (eventos extremos).

4. **Análises de Correlação**

   * Correlação entre estratégias/famílias, tanto em termos de retornos quanto de sinais;
   * Auxilia na construção de portfólios mais descorrelacionados.

5. **Visualização de Sinais no Gráfico de Preço**

   * Sobreposição de entradas/saídas no gráfico do ativo, com marcação das regiões de interesse (por exemplo, pontos de TSM, reversões, zonas de carry);
   * Importante para inspeção qualitativa e detecção de comportamentos estranhos (entradas fora de contexto, clustering de sinais próximos demais etc.).

6. **Exportação e Integração**

   * Capacidade de exportar resultados e séries de P&L para formatos comuns (CSV, Parquet, relatórios) para análise em ferramentas externas (planilhas, notebooks, BI).

---

Com esse conjunto de mecanismos de **Backtesting, Simulação e Validação**, o sistema não apenas “roda estratégias no passado”, mas fornece uma base sólida para:

* Entender em quais condições cada família tende a funcionar melhor ou pior;
* Medir, com rigor, o risco de overfitting;
* Verificar se o comportamento em produção está alinhado com o que foi observado em ambiente controlado;
* Tomar decisões informadas sobre promoção, ajuste ou desligamento de estratégias ao longo do tempo.


---

## 15. Execução em Produção e Core Loop

O **core loop** operacional é o ciclo contínuo que conecta em produção:

> Dados de mercado → Estratégias → Risco → Ordens → Estado → Métricas

Em alto nível, a cada “batida” de loop (trigger por tempo, por fechamento de candle ou por evento), o fluxo ocorre em etapas:

1. **Disparo do Scheduler / Orquestrador**

   * O Scheduler acorda com base em:

     * Timeframes configurados para cada instância de estratégia (por ex.: a cada fechamento de candle de 1m, 5m, 1h, 1d);
     * Janelas de agregação de dados (para estratégias que trabalham com múltiplos timeframes);
     * Eventuais triggers assíncronos (ex.: recebimento de novo candle via WebSocket).

2. **Atualização de Dados**

   * O módulo de Market Data:

     * Coleta o último pedaço de dados necessário (novo candle, novo batch de candles, correções);
     * Atualiza o armazenamento local (SQLite/Parquet) mantendo consistência e evitando duplicidade;
     * Executa verificações básicas de integridade (timestamps, gaps óbvios).

3. **Enriquecimento de Dados**

   * Funções de “data enrichment” calculam ou atualizam:

     * Indicadores técnicos (EMAs, retornos, volatilidade, ranks, etc.);
     * Features específicas por família (por ex.: scores de momentum, bandas de reversão, sinais intermediários).
   * Respeita-se o **lookback mínimo** solicitado por cada estratégia, evitando gerar sinais com dados incompletos.

4. **Execução das Estratégias**

   * Para cada instância ativa cujo timeframe/time window foi disparado:

     * O Orquestrador entrega um `DataContext` preparado (dados + indicadores + metadados);
     * A estratégia, via Strategy API, executa `generate_signals()` e retorna **posições alvo** por mercado:

       * Exemplo: “BTCUSDT: alvo 100% long”, “ETHUSDT: alvo 0% (cash)”.

5. **Aplicação de Risco e Alocação**

   * O módulo de Risco recebe cada posição alvo e:

     * Verifica limites por trade, estratégia, família, ativo, exchange e portfólio global;
     * Ajusta ou rejeita posições que violem limites;
     * Constrói um **plano de execução** (quanto comprar/vender, em que ordem, com que prioridade).
   * O resultado dessa etapa é um conjunto de **ordens candidatas** (planos de trade) aprovadas pelo risco.

6. **Geração e Envio de Ordens para a Exchange**

   * O Trading Layer:

     * Converte o plano de execução em ordens concretas de acordo com as capacidades da exchange (market/limit, etc.);
     * Aplica arredondamentos de acordo com filtros de lote/step size/notional;
     * Envia as ordens através da API (Binance Spot no MVP);
     * Registra o ID, estado inicial (NEW) e qualquer retorno da exchange.

7. **Atualização de Estado e P&L**

   * A partir de:

     * Retornos da exchange (fills, partial fills, cancels);
     * Consultas periódicas a open orders/positions;
   * O módulo de Estado atualiza:

     * Posições por ativo, por estratégia, por família;
     * P&L realizado e não realizado;
     * Exposição por camada de risco.
   * O resultado é persistido em banco local, mantendo uma visão consistente do portfólio.

8. **Observabilidade e Auditoria**

   * O loop registra:

     * Dados de entrada (preços, indicadores);
     * Sinais gerados;
     * Decisões de risco (por que uma ordem foi reduzida ou rejeitada);
     * Ordens efetivamente enviadas e seus resultados;
     * KPIs operacionais (latência de decisão, tempo de round-trip até a exchange).
   * Esses registros alimentam logs, dashboards, alertas e trilhas de auditoria.

Esse core loop é **contínuo e parametrizável**, podendo operar desde baixa frequência (estratégias diárias) até intraday moderado, respeitando as limitações de um ambiente single node e de um investor “não-HFT”.

---

### 15.2. Modo Single Exchange (Binance Spot)

No estado atual do projeto, a execução em produção é **focada em uma única exchange: Binance Spot**. Isso traz algumas características específicas:

1. **Adapter Dedicado**

   * Um adapter `BinanceSpotAdapter` implementa:

     * `MarketDataPort` (coleta de candles/trades via REST e WebSocket);
     * `TradingPort` (envio de ordens, consulta de estado, cancelamentos).
   * Este adapter encapsula:

     * Detalhes de autenticação (API keys, assinaturas);
     * Mapeamento de erros da API Binance para exceções/estados internos;
     * Tratamento de rate limits e backoff.

2. **Configurações Específicas de Binance**

   * Timeframes suportados, símbolos disponíveis, filtros de lote e notional mínimo são obtidos via **schema discovery** e armazenados.
   * O módulo de risco e de ordens usa essas informações para:

     * Validar tamanhos de ordens;
     * Ajustar arredondamentos;
     * Evitar rejeições por parâmetros inválidos.

3. **Monitoramento de Saúde da Exchange**

   * O sistema mantém métricas de:

     * Latência média de chamadas REST e WebSocket;
     * Taxa de erros (timeouts, 429 - rate limit, 5xx);
     * Integridade dos dados de candles (detecção de buracos anormais).
   * Com base nisso, pode:

     * Entrar em modos de degradação (por exemplo, reduzir frequência de ordens);
     * Acionar circuit breakers específicos da exchange (seção 13).

4. **Alinhamento com Estratégias Spot Long/Flat**

   * Como o foco é mercado spot, as estratégias são construídas para:

     * Operar somente com ordens de compra/venda de spot (sem alavancagem ou margem, no MVP);
     * Trabalhar com lógica long/flat (sem descoberto short), simplificando o modelo de risco.

O modo single exchange permite um desenvolvimento mais seguro e controlado, servindo de base sólida para expandir para outras exchanges no futuro.

---

### 15.3. Evolução para Multi-Exchange

A lógica de execução é construída com multi-exchange em mente, utilizando portas genéricas para dados e trading, permitindo a coexistência de múltiplos adapters (ex: `BinanceSpotAdapter`, `KrakenSpotAdapter`). O orquestrador agrupa ordens por exchange, e o módulo de risco considera limites de exposição e alocação entre elas.

   * O módulo de risco (ver seção 13) passa a considerar:

     * Limites de exposição por exchange;
     * Distribuição de capital entre exchanges para reduzir risco de custódia;
     * Diferentes níveis de confiança em cada exchange (por exemplo, exchanges mais novas com limites mais conservadores).

4. **Estratégias Cross-Exchange (Futuro)**

   * Em fases futuras, famílias que dependem de oportunidades cross-exchange (por exemplo, arbitragem simples) podem ser introduzidas sem mudanças no core loop, apenas adicionando:

     * Estratégias que consomem dados de múltiplas exchanges;
     * Regras específicas de risco/inventário para esse tipo de abordagem.

Essa evolução é incremental: o núcleo do core loop permanece o mesmo, com a adição de adapters e políticas de risco específicas.

---

### 15.4. Gestão de Estado

A gestão de estado é um pilar da robustez do sistema, garantindo consistência, auditabilidade e recuperação segura. A arquitetura se baseia em um `PortfolioState` como fonte única da verdade, com atualizações atômicas e reconciliação contínua.

1.  **`PortfolioState` como Fonte Única da Verdade (SSOT):**
    *   Um objeto central, mantido em memória para acesso rápido e persistido em banco, que encapsula todo o estado relevante: posições, saldos, ordens abertas, P&L e status de risco.
    *   Qualquer decisão de risco ou de estratégia é baseada neste estado canônico.

2.  **Princípio do Escritor Único e Event Bus:**
    *   Para evitar condições de corrida e garantir atomicidade, as modificações no `PortfolioState` seguem o **Single Writer Principle**.
    *   Apenas um componente (o `StateManager`) tem permissão para escrever no estado.
    *   Outros componentes (ex: `Execution` com um novo `fill`, `Risk` com um `circuit breaker`) publicam eventos em um **Event Bus** interno (ex: `asyncio.Queue`). O `StateManager` consome esses eventos sequencialmente e aplica as atualizações de forma atômica.

3.  **Persistência em Camadas:**
    *   **Journal de Transações:** Cada evento que modifica o estado é primeiro escrito em um log de transações (journal) para garantir durabilidade (Write-Ahead Logging).
    *   **Banco de Dados Transacional (SQLite):** Após o journal, o estado consolidado é atualizado no banco de dados SQLite, que serve como a camada de persistência principal para recuperação.
    *   **Snapshots Periódicos:** Para acelerar a recuperação, snapshots completos do `PortfolioState` são salvos em disco (ex: em formato Parquet) em intervalos regulares.

4.  **Reconciliação com a Exchange:**
    *   Periodicamente (e sempre no restart), um processo de **reconciliação** compara o `PortfolioState` interno com o estado real da conta na exchange.
    *   **Posições Órfãs:** Posições existentes na exchange mas não no estado local são identificadas, registradas com um `reason_code: RECOVERED_ORPHAN` e incorporadas ao estado.
    *   **Posições Fantasma:** Posições existentes no estado local mas não na exchange são removidas, e um alerta `CRITICAL` é gerado, pois isso pode indicar uma perda não registrada.
    *   **Divergências:** Para qualquer divergência, **o estado da exchange prevalece**. O estado local é ajustado para corresponder à realidade, e o evento é logado para auditoria.

   * Isso garante:

     * Recuperação após falhas ou restart;
     * Base histórica consistente para auditoria.

3. **Reconciliation com a Exchange**

---

### 15.5. Failover e Recuperação

O sistema é projetado para ser resiliente a falhas de processo e de rede. Ele inclui mecanismos de recuperação de estado, reconciliação com a exchange e políticas de retry/backoff.

Os procedimentos detalhados de recuperação para operadores estão documentados no **OPERATIONS_RUNBOOK.md**.

---

### 15.6. Modos de Operação

O sistema suporta diferentes modos de operação, com o **mesmo core de lógica**, mas diferentes destinos para os sinais e ordens:

1. **Backtest (Offline)**

   * Execução completamente histórica, sem interação com exchange real.
   * Usa dados históricos e executor simulado (já descrito em 14).

2. **Paper Trading (Online Simulado)**

   * Usa dados ao vivo da exchange, mas:

     * Não envia ordens reais;
     * Simula ordens e fills localmente.
   * Ideal para:

     * Testar novas estratégias em ambiente de produção;
     * Validar performance sob latência e falhas reais sem risco financeiro.

3. **Live Trading (Produção Real)**

   * Usa dados ao vivo e envia ordens reais à exchange.
   * Respeita todos os guardrails de risco e circuit breakers.
   * Pode coexistir com instâncias em paper trading:

     * Por exemplo, algumas estratégias da família TSM em live, outras da família Mean Reversion ainda em paper.

A transição de uma instância entre modos (paper → live ou live → paper) é controlada pelo módulo de governança, com logs e versionamento, evitando mudanças silenciosas.

---

### 15.7. Segurança na Execução de Ordens

A execução de ordens é uma área crítica, onde erros podem causar perdas diretas. O sistema adota uma série de práticas para garantir **segurança e previsibilidade**:

1. **Validação Estrita Antes do Envio**

   * Cada ordem candidata passa por:

     * Validação de parâmetros (quantidade, preço, tipo) conforme filtros de mercado (schema discovery da exchange);
     * Verificação de limites de risco (ver seção 13);
     * Checagem de saldo disponível e reservas de margem internas (mesmo que seja só spot).

2. **Idempotência e De-Duplicação**

   * O core loop evita reenvio acidental de ordens:

     * Cada plano de execução recebe um ID interno;
     * O sistema marca quais ordens já foram enviadas;
     * Em caso de falha de comunicação, usa reconciliation e consultas diretas à exchange para decidir se deve ou não reenviar.

3. **Política de Cancelamento**

   * Em condições de erro persistente, circuit breaker ou shutdown controlado:

     * O sistema pode opcionalmente:

       * Cancelar ordens abertas;
       * Ou manter ordens de saída (stops/TP) enquanto fecha o resto do sistema.
   * A política é configurável, mas sempre documentada e auditável.

4. **Limites de Frequência e Volume de Ordens**

   * Para evitar:

     * Bater em rate limits da exchange;
     * Gerar “explosões” de ordens devido a loops lógicos.
   * O módulo de execução:

     * Aplica limites de ordens por unidade de tempo;
     * Agrupa ajustes pequenos de posição;
     * Implementa throttling quando necessário.

5. **Logs e Assinaturas de Decisão**

   * Cada ordem é acompanhada por:

     * Qual estratégia/família a originou;
     * Qual era o contexto (preço, indicadores, sinais);
     * Qual foi a decisão do módulo de risco.
   * Isso permite responder, a qualquer momento: “por que essa ordem foi enviada?” e “por que com esse tamanho/preço?”.

6. **Proteção Contra Erros de Configuração**

   * Mudanças de parâmetros críticos (por exemplo, tamanho máximo de posição, modo de operação live/paper) podem:

     * Exigir confirmação adicional (dupla validação);
     * Ser registradas em log de alto nível;
     * Disparar notificações para o operador.

---

Com essa estrutura de **Execução em Produção e Core Loop**, o sistema busca equilibrar:

* **Fidelidade operacional** (rodar em produção o mais próximo possível do que foi testado);
* **Robustez** (tolerância a falhas de infraestrutura e de exchange);
* **Segurança** (limites de risco, validações e auditoria);
* **Evolutividade** (facilidade de adicionar novas exchanges, famílias de estratégias e modos de operação, sem reescrever o núcleo).

---

### 15.8. Gerenciamento de Rate Limits e Saúde do Sistema

O sistema gerencia proativamente os **rate limits** da exchange através de mecanismos como Token Bucket, backoff exponencial e throttling.

A saúde do sistema é monitorada continuamente via **heartbeats** do core loop e **health checks** periódicos que verificam conectividade, atualidade dos dados e recursos do sistema.

Os procedimentos operacionais para lidar com alertas de rate limit e falhas de health check estão descritos no **OPERATIONS_RUNBOOK.md**.

---

## 16. Observabilidade e Telemetria

A camada de **Observabilidade e Telemetria** é transversal e visa responder o que está acontecendo, o que aconteceu e por que o sistema tomou certas decisões. Ela permite operação com detecção precoce de problemas, análise pós-mortem e auditoria completa.

Os logs são a base textual da observabilidade. No sistema, eles seguem os seguintes princípios:

* **Formato Estruturado (tipicamente JSON)**

  * Cada entrada de log é um objeto estruturado, contendo campos como:

    * `timestamp_utc`
    * `log_level` (DEBUG, INFO, WARNING, ERROR, CRITICAL)
    * `component` (market_data, strategy, risk, trading, infra, governance)
    * `strategy_id`, `family`, `exchange_id`, `symbol` (quando aplicável)
    * `correlation_id` / `trace_id` para vincular eventos relacionados
    * `message` (resumo humano-legível)
    * `details` (payload adicional, parâmetros, respostas de API, etc.)

* **Categorias de Logs por Domínio**

  * **Market Data**: problemas de coleta, gaps de candle detectados, atrasos no WebSocket.
  * **Estratégias**: sinais gerados, cenários inesperados (por exemplo, ausência de dados suficientes).
  * **Risco**: rejeição/redução de posições, disparo de circuit breakers.
  * **Trading**: envio de ordens, respostas da exchange, erros, retries.
  * **Infraestrutura**: falhas de rede, erros de disco, timeouts de serviços externos.
  * **Governança**: mudanças de configuração, ativações/desativações de estratégias.

* **Níveis de Log Ajustáveis**

  * Em produção, o padrão é **INFO + WARNING + ERROR**, com DEBUG habilitado apenas em janelas específicas ou para componentes específicos (feature flag) para não sobrecarregar o sistema.
  * Em backtest e ambientes de teste, níveis mais verbosos podem ser ativados.

* **Proteção de Dados Sensíveis**

  * Credenciais, tokens, chaves de API, e qualquer dado sensível **nunca** são logados em claro.
  * Campos potencialmente sensíveis são mascarados ou omitidos.

* **Destino e Retenção**

  * Logs são direcionados para:

    * Arquivos locais com rotação (log rotation);
    * Ou coletor centralizado (stack de observabilidade, se disponível).
  * Políticas de retenção são definidas por criticidade:

    * Logs de auditoria e risco tendem a ter retenção mais longa;
    * Logs de debug detalhado têm retenção curta.

---

### 16.2. Métricas de Sistema

As **métricas de sistema** medem a saúde e a capacidade da infraestrutura em single node, permitindo identificar gargalos e prevenir quedas.

Principais grupos de métricas:

* **Recursos de Máquina**

  * Uso de CPU (por núcleo e total).
  * Uso de memória (RAM utilizada, swap, buffers).
  * Utilização de disco: espaço livre, I/O, latência média de leitura/escrita.
  * Uso de rede: tráfego de entrada/saída, erros de pacote.

* **Latência e Performance Interna**

  * Latência do core loop por estratégia/família (tempo decorrido entre “disparo” e “ordens prontas”).
  * Latência de chamadas à exchange (REST, WebSocket).
  * Latência de acesso a banco de dados local (consultas críticas).

* **Erros e Falhas de Componentes**

  * Contadores de:

    * Erros por componente (`market_data_errors`, `trading_errors`, `risk_errors`).
    * Timeouts de rede, timeouts de disco, falhas de reconexão.
  * Taxa de erro (por minuto / hora), útil para detecção de incidentes.

Essas métricas são publicadas em formato adequado para consumo por uma base de dados de séries temporais (ex.: Prometheus ou equivalente), permitindo construção de dashboards e alertas.

---

### 16.3. Métricas de Negócio

As **métricas de negócio** respondem à pergunta: “do ponto de vista de trading, o que está acontecendo com o portfólio?”.

Principais métricas:

* **P&L (Realizado e Não Realizado)**

  * Por:

    * Estratégia (`strategy_id`);
    * Família de estratégias (`family`);
    * Exchange (`exchange_id`);
    * Ativo/símbolo (`symbol`).
  * Em tempo quase real (com atualização a cada ciclo de loop ou a cada mudança de posição).

* **Exposição**

  * Exposição nominal e percentual:

    * Por ativo (BTC, ETH, etc.);
    * Por família;
    * Por exchange;
    * Por moeda (BTC-equivalente, USD-equivalente etc.).
  * Indicadores de concentração (por exemplo, % do portfólio em um único ativo).

* **Risco e Drawdown**

  * Drawdown atual e máximo (intradiário, mensal, histórico).
  * Uso de “risk budget” por camada:

    * Quanto do limite de DD diário já foi utilizado;
    * Quanto do risco máximo por trade está sendo normalmente usado.

* **Atividade de Trading**

  * Número de ordens por unidade de tempo (estratégia/família/total).
  * Turnover (volume negociado) por período.
  * Taxas pagas (fees) e slippage estimado.

* **Eficiência das Estratégias**

  * Hit ratio (percentual de trades vencedores).
  * Payoff médio (ganho médio / perda média).
  * Métricas agregadas (Sharpe, Sortino, Calmar) em janelas móveis.

Essas métricas permitem identificar rapidamente estratégias degradando, famílias superexpostas ou comportamento anormal (por exemplo, volume de ordens muito acima do esperado).

---

### 16.4. Dashboards e Alertas

A partir de logs e métricas, são construídos **dashboards** e **alertas automáticos** que suportam operação e tomada de decisão.

#### Dashboards

A stack de observabilidade evolui com o projeto:

*   **Fase 1 (Minimalista):** Logs estruturados em JSON e métricas exportadas para arquivos CSV para análise offline.
*   **Fase 2+ (Stack Local):** Introdução de **Prometheus + Grafana** (rodando localmente via Docker) para dashboards em tempo real e alertas robustos.

Os painéis principais em Grafana incluem:

1. **Dashboard de Saúde do Sistema**

   * CPU, RAM, disco, rede;
   * Latência de chamadas a exchange;
   * Erros por componente;
   * Estado de health checks (UP/DOWN, modo normal vs degradado).

2. **Dashboard de Trading e Risco**

   * P&L por estratégia/família;
   * Exposição por ativo, família, exchange;
   * Drawdown atual e histórico;
   * Utilização de limites (proximidade do DD diário/mensal máximo).

3. **Dashboard de Estratégias**

   * Atividade por estratégia:

     * Número de sinais gerados, número de ordens enviadas;
     * Taxa de utilização de capital;
   * Performance acumulada e por janela (dia, semana, mês);
   * Indicadores específicos da família (por exemplo, quanto do tempo TSM está em cash vs posicionado).

Esses painéis permitem, em uma visão consolidada, acompanhar em tempo real a “vida” do sistema.

#### Alertas

Alertas automáticos são configurados com base em limiares e condições específicas, por exemplo:

* **Operacionais**

  * Falha recorrente em chamadas à exchange;
  * Aumento abrupto na taxa de erros de trading ou risco;
  * Latência do core loop acima de um limite.

* **De Risco**

  * Alcançar X% do drawdown diário ou mensal;
  * Exposição em um ativo acima do limiar de concentração;
  * Acionamento de circuito breaker (local ou global).

* **De Governança**

  * Alterações em parâmetros de risco críticos;
  * Promoção de estratégia de paper para live;
  * Desativação de família ou instância.

Os alertas são enviados para canais configuráveis (por exemplo, Telegram, e-mail), permitindo resposta humana rápida quando necessário.

---

### 16.5. Tracing de Estratégias

O **Tracing** permite seguir a cadeia de decisão (dado -> sinal -> risco -> ordem -> P&L) usando `trace_id` para correlacionar eventos. Cada ciclo de decisão é associado a um `trace_id`, que também registra as versões de código e configuração da estratégia.
* **Registro de Decisões Intermediárias**

  * Além do sinal final (posição alvo), podem ser registrados:

    * Sinais intermediários (scores, indicadores chave, filtros ativados);
    * Motivos de rejeição ou ajuste feitos pelo módulo de risco.

* **Consulta e Reconstituição**

  * A partir de um trade ou ordem específicos, é possível:

    * Recuperar o `trace_id` associado;
    * Consultar todos os eventos daquela cadeia;
    * Reconstituir o “raciocínio” do sistema naquele instante.

Isso é crucial para:

* Análise pós-incidente (“por que essa posição tão grande foi aberta?”);
* Explicabilidade (especialmente se o sistema evoluir para incluir componentes mais complexos, como ML);
* Auditoria (interna, de terceiros ou regulatória).

---

### 16.6. Auditoria de Decisões de Trading

A **auditoria** é a camada que transforma logs e traces em um **registro formal e navegável** das decisões mais importantes do sistema.

Componentes principais:

1. **Trilha de Decisões de Trading**

   * Para cada ordem executada, registra-se:

     * Quem (qual estratégia/família) originou o sinal;
     * Quando o sinal foi gerado (timestamp);
     * Qual era o contexto de mercado (preço, volatilidade, regime) e de portfólio (exposição, DD, uso de budget);
     * Qual foi a decisão do módulo de risco (aprovado, reduzido, bloqueado);
     * Qual ordem foi enviada de fato (tamanho, preço, tipo);
     * Resultado (fills, P&L associado).

2. **Trilha de Configuração e Risco**

   * Todas as mudanças em:

     * Parâmetros de estratégias;
     * Limites de risco (trade, estratégia, família, global);
     * Modos de operação (paper ↔ live);
   * São registradas com:

     * Timestamp;
     * Usuário/ator (quando há interação manual);
     * Diff da configuração (antes/depois).

3. **Eventos de Governança**

   * Promoção de estratégias (de backtest → paper → live);
   * Desativação ou suspensão de estratégias;
   * Ações manuais de emergência (por exemplo, fechamento manual de posições).

4. **Consultas e Relatórios de Auditoria**

   * Interface (consulta ou relatórios) permitindo responder a perguntas como:

     * “Liste todas as ordens da estratégia X no dia Y com seus motivos”.
     * “Quais mudanças de risco foram feitas no último mês e por quem?”.
     * “Quais circuit breakers foram disparados no ano corrente?”.

5. **Integração com Compliance (se aplicável)**

   * Em contextos regulados, essa camada de auditoria pode ser adaptada para:

     * Exportar relatórios em formatos específicos;
     * Integrar com sistemas de arquivamento de longo prazo.

---

Com a combinação de **logs estruturados**, **métricas de sistema e negócio**, **dashboards e alertas**, **tracing detalhado** e **auditoria formal**, a camada de Observabilidade e Telemetria transforma o sistema em algo:

* Transparente (é possível ver e entender o que está acontecendo);
* Confiável (problemas são detectados e tratados com rapidez);
* Responsável (cada decisão de trading é rastreável e explicável);
* Pronto para escalar, tanto em complexidade de estratégias quanto em requisitos de governança.

---

## 17. Governança de Estratégias e Change Management

A camada de **Governança de Estratégias e Change Management** define o processo de como as estratégias são desenvolvidas, modificadas, registradas e auditadas. Toda estratégia passa por um pipeline de aprovação padronizado:

1. **Proposta Conceitual**

   * Descrição da ideia em nível de família ou instância:

     * Qual hipótese a estratégia explora (ex.: tendência, reversão, carry, value)?
     * Em quais mercados/timeframes pretende atuar?
     * Quais são os riscos conhecidos e cenários onde tende a falhar?
   * Referências de evidência (artigos, estudos, backtests anteriores).

2. **Especificação Técnica**

   * Tradução da ideia em:

     * Regras objetivas (entradas, saídas, filtros, limites básicos);
     * Parâmetros configuráveis (lookbacks, thresholds, etc.);
     * Dependências de dados e indicadores (requisitos de lookback, features necessárias).
   * Nesta etapa, já se pensa no **contrato da Strategy API**.

3. **Implementação de Família / Instância**

   * Codificação da estratégia como módulo que implementa `BaseStrategy`;
   * Criação de arquivos de configuração (YAML/JSON) para *instâncias* específicas.

4. **Backtest e Pesquisa**

   * Execução de backtests com o engine unificado (seção 14):

     * Cenários múltiplos (bull, bear, sideways);
     * Períodos longos o suficiente para evitar conclusões baseadas em amostras pequenas;
     * Validação básica de robustez (sensibilidade a parâmetros, IS/OOS).

5. **Revisão de Risco**

   * Avaliação pelo módulo de risco (e, se houver, por um “Risk Owner” humano):

     * Perfil de drawdown;
     * Risco de concentração e correlação com estratégias já existentes;
     * Adequação aos guardrails globais.
   * Ajustes de limites locais (por trade, por instância, por família) podem ser sugeridos aqui.

6. **Revisão de Operações (Operacionalidade)**

   * Verificação de:

     * Dependências externas (ex.: necessidade de dados de book, tipos de ordem específicos);
     * Compatibilidade com a infraestrutura atual (single node, Binance Spot);
     * Complexidade operacional (quantidade de ordens, sensibilidade à latência, etc.).

7. **Decisão de Promoção para Paper Trading**

   * Se as etapas anteriores forem satisfatórias:

     * A estratégia é marcada como “aprovada para paper trading”;
     * Um plano de observação é definido (por exemplo, rodar em paper por X semanas).

8. **Paper Trading e Avaliação em Tempo Real**

   * Execução em paper trading:

     * Comparação entre comportamento real e backtest;
     * Avaliação da estabilidade operacional (falhas, latência, integração com risk).
   * Ao fim do período, gera-se um **relatório de paper trading**.

9. **Decisão de Produção (Live Trading)**

   * Com base nos relatórios de backtest e paper:

     * Aprovação (com capital inicial limitado e plano de aumento progressivo);
     * Reprovação (ideia arquivada ou redesenhada);
     * “Estacionamento” (ficar em paper por mais tempo).

Todas essas etapas são registradas na camada de auditoria (seção 16) com timestamps, versão de código e de configuração, para rastreabilidade total.

---

### 17.2. Versionamento de Estratégias e Configurações

O versionamento de estratégias (`strategy_version`) e configurações (`config_version`) é crucial. A `strategy_version` muda com alterações na lógica ou dependências de dados. A `config_version` rastreia mudanças em parâmetros, mercados e flags. A combinação de ambas define um comportamento completo e rastreável para cada execução.

Além do código, há a **configuração**, que também é versionada:

* `config_version` (por exemplo, `TSM_BTCUSDT_1D_config_v2`);
* Inclui:

  * Parâmetros (lookbacks, thresholds, limites locais de risco);
  * Universo de mercados/timeframes;
  * Flags de ativação/desativação.

A combinação `strategy_version + config_version` define **um comportamento completo** da instância.
Qualquer backtest ou execução em produção referencia ambos, por exemplo:

> “Run #123 – TSM_BTCUSDT_1D – strategy_version 1.0.3 – config_version v2 – período 2020–2024”.

As mudanças de configuração relevantes (especialmente de risco) são logadas como eventos de governança e podem requerer aprovação adicional.

---

### 17.3. Processo de Rollout e Rollback

**Rollout** é o processo de colocar uma nova versão (de código ou config) em operação.
**Rollback** é a capacidade de voltar rapidamente a uma versão anterior se algo der errado.

#### 17.3.1. Rollout Controlado

O processo típico de rollout inclui:

1. **Backtest com Nova Versão**

   * Reexecutar, ao menos, um subconjunto de cenários padrão usando a nova combinação `strategy_version + config_version`.

2. **Paper Trading Prioritário**

   * Em muitos casos, especialmente para mudanças significativas:

     * A nova versão é primeiro rodada em paper trading, lado a lado com a versão antiga (live), por um período de observação.

3. **Deploy Gradual em Produção**

   * Quando aprovado para live:

     * Pode-se iniciar com **alocação de capital reduzida**;
     * Aumentar a alocação gradualmente, se a performance e o comportamento operacional estiverem de acordo com o esperado.

4. **Monitoramento Intensivo Pós-Rollout**

   * Nas primeiras janelas de tempo após o rollout:

     * Alertas e dashboards são ajustados para sensibilidade maior;
     * Qualquer comportamento anormal (ex.: aumento anômalo de erros, ordens excessivas, divergência com sinais esperados) é motivo para revisão.

#### 17.3.2. Rollback Rápido


1. Identificação do problema:
   * **Rollback Instantâneo via Feature Flagging**: O sistema deve permitir o rollback instantâneo de uma estratégia ou família para uma versão anterior (ou desativação completa) através da alteração de `feature flags` em tempo de execução, sem a necessidade de um novo deploy ou restart do processo principal.

   1. Identificação do problema:


2.  **Ação de Governança:**

   * Decisão explícita de:

     * Voltar para versão anterior;
     * Ou desativar a instância/família por segurança.

3.  **Execução Técnica:**

   * Troca de configuração para a versão anterior (ou desativação);
   * Eventual ajuste de posições:

     * Ex.: fechamento de posições abertas pela versão nova, se julgado necessário.

4. Registro em Auditoria:

   * O rollback é registrado com:

     * Motivo;
     * Versões envolvidas (nova e antiga);
     * Impacto observado até o momento da reversão.

---

### 17.4. Critérios para Desativação ou Rebalance

Estratégias e famílias não são “eternas”: podem perder eficácia, ficar incompatíveis com o ambiente de mercado ou se tornar operacionalmente problemáticas.
A governança define **critérios explícitos** para:

* Desativar, rebaixar ou reconfigurar uma instância ou família;
* Rebalancear a alocação de capital entre famílias.

#### 17.4.1. Critérios Quantitativos

Alguns exemplos de critérios que podem disparar revisão/desativação:

* **Drawdown Sustentado**

  * DD acima de um limiar pré-definido (por exemplo, 1,5× o DD máximo observado em backtest) por um período relevante.

* **Degradação de Métricas Ajustadas ao Risco**

  * Sharpe/Sortino/Calmar caindo abaixo de thresholds em janelas móveis (por exemplo, últimos 6 ou 12 meses).

* **Mudança de Regime Estrutural**

  * Estratégia que depende de característica de mercado que claramente mudou (por exemplo, redução estrutural de volatilidade/volume em determinado segmento).

* **Desalinhamento com Backtest/Paper**

  * Performance real muito distante da faixa observada em backtests e paper trading, mesmo após ajuste de custos/slippage.

#### 17.4.2. Critérios Qualitativos/Operacionais

* **Problemas Operacionais Repetidos**

  * Estratégia que constantemente gera ordens na borda das capacidades da exchange, causando muitos erros ou rejeições.
  * Estratégia que exige infraestrutura que ainda não é robusta o suficiente (por exemplo, book em alta frequência).

* **Complexidade de Manutenção Excessiva**

  * Estratégia que exige atenção manual constante, tuning frequente, retrabalho.

#### 17.4.3. Ações Possíveis

Diante de critérios de alerta, as ações podem ser:

* **Desativação Temporária**

  * Instância passa para modo “somente saída” e, em seguida, é desligada;
  * Pode ser reavaliada no futuro com novos parâmetros ou em novo regime de mercado.

* **Rebaixamento**

  * De live → paper trading, mantendo monitoramento sem risco financeiro.

* **Rebalanceamento de Capital**

  * Redução de alocação da família/instância com performance inferior;
  * Realocação para famílias mais robustas no momento.

Todas as decisões são registradas em log de governança, incluindo motivos (quantitativos e qualitativos), data e responsáveis.

---

### 17.5. Documentação Obrigatória por Estratégia

Para cada estratégia (família + instância), existe um **conjunto mínimo de documentação obrigatória**, que garante clareza, reprodutibilidade e qualidade de decisão.

Elementos essenciais:

1. **Ficha da Família de Estratégia**

   * Nome, tipo (TSM, Dual Momentum, Mean Reversion, etc.);
   * Hipótese teórica (qual ineficiência ou prêmio de risco explora);
   * Regimes de mercado em que tende a performar melhor/pior;
   * Principais riscos (ex.: crashes de momentum, regime change, squeezes de liquidez).

2. **Especificação da Instância**

   * `strategy_id`, `strategy_version`, `config_version`;
   * Universo de mercados/timeframes;
   * Parâmetros específicos e justificativa dos principais (por exemplo, por que lookback de 6 meses, e não 3/12?);
   * Requisitos de dados (indicadores, lookback mínimo).

3. **Resumo de Backtests**

   * Períodos testados (datas, exchanges, universos de ativos);
   * Métricas principais (retorno, Sharpe, Sortino, Calmar, DD máximo, etc.);
   * Resultados por regime de mercado;
   * Análise de robustez:

     * Sensibilidade a parâmetros;
     * IS/OOS, walk-forward quando aplicado;
     * Comparação com benchmarks simples.

4. **Resumo de Paper Trading**

   * Período de paper trading;
   * Comparação entre comportamento esperado (backtest) e observado;
   * Incidentes operacionais (falhas, ajustes necessários).

5. **Parâmetros de Risco e Políticas**

   * Limites de risco por trade/instância;
   * Condições de circuit breaker específicas;
   * Papel desta estratégia no portfólio (por exemplo, “componente de tendência de médio prazo”, “componente de diversificação descorrelacionada” etc.).

6. **Histórico de Mudanças (Changelog)**

   * Lista de mudanças de código (`strategy_version`) e configuração (`config_version`), com:

     * Data;
     * Descrição;
     * Impacto esperado;
     * Vinculação a backtests relevantes.

7. **Decisões de Governança Relevantes**
   * **Geração Automática**: A Ficha de Estratégia deve ser gerada automaticamente a partir dos arquivos de configuração declarativa (YAML/JSON) e do código-fonte da estratégia, garantindo que esteja sempre atualizada e consistente.
   * Promoções (backtest → paper → live);
   * Reduções de capital;
   * Desativações e motivos;
   * Rollbacks executados.

Essa documentação pode ser mantida em repositórios (por exemplo, Markdown + controle de versão) e/ou em um sistema interno de catálogo de estratégias. O importante é que ela seja:

* Fácil de consultar;
* Sincronizada com o código e as configurações;
* Atualizada sempre que houver mudança relevante.

---

Com essa camada de **Governança de Estratégias e Change Management**, o sistema deixa de ser apenas um conjunto de algoritmos rodando em produção e se torna um **processo controlado de pesquisa, validação, promoção, monitoramento e aposentadoria de estratégias**, capaz de:

* Proteger capital;
* Aprender com sucessos e erros;
* Evoluir de forma contínua e responsável.

---

## 18. Segurança e Compliance

A camada de **Segurança e Compliance** foca em proteger capital, credenciais e dados, seguindo o princípio de **Security by Design**. A gestão de segredos, como API keys, é um ponto crítico.

Principais práticas:

* **Segregação entre código e segredos**

  * API keys, tokens e senhas **não são armazenados em repositórios de código**.
  * São carregados via:

    * Variáveis de ambiente;
    * Arquivos de configuração **criptografados**;
    * Ou soluções de secrets vault (quando disponíveis).

* **Princípio do Menor Privilégio**

  * As chaves de exchange usadas pelo sistema:

    * São configuradas, sempre que possível, apenas com permissões de **trading**, sem permissão de saque/withdraw;
    * Podem ter IPs fixos autorizados (whitelisting), quando a exchange oferece esse recurso.

* **Rotação Periódica de Segredos**

  * Processo definido para:

    * Rotacionar API keys com periodicidade (mensal, trimestral, conforme política);
    * Invalidar imediatamente chaves suspeitas, com logging e registro em auditoria.

* **Proteção em Descanso (At Rest)**

  * Segredos armazenados em disco (quando inevitável) são:

    * Mantidos em arquivos separados;
    * Criptografados;
    * Com permissões de sistema operacional restritas ao usuário do serviço.

* **Monitoramento de Acessos a Segredos**

  * Acesso a componentes que carregam ou manipulam segredos é logado em nível alto de criticidade, sem expor o conteúdo do segredo.

---

### 18.2. Princípio do Menor Privilégio e Controle de Acesso

Mesmo em single node, aplica-se o **Princípio do Menor Privilégio** em todas as camadas:

* **Separação de Papéis Lógicos**

  * Módulos de:

    * Estratégia;
    * Risco;
    * Trading;
    * Observabilidade;
    * Governança.
  * Têm responsabilidades bem definidas – por exemplo, estratégia não acessa credenciais nem envia ordens diretamente.

* **Controle de Acesso a Funções Sensíveis**

  * Ações como:

    * Alterar parâmetros de risco globais;
    * Trocar de modo paper → live;
    * Rotacionar chaves;
    * Executar rollbacks de versão;
  * São tratadas como operações de **governança**, exigindo:

    * Autorização explícita;
    * Registro em log de alto nível (evento de governança).

* **Privilégios de Sistema Operacional**

  * O processo do bot roda com um usuário de sistema dedicado, com:

    * Permissões mínimas sobre o filesystem;
    * Sem privilégios de administração desnecessários;
    * Acesso restrito apenas às pastas necessárias (dados, logs, configs).

---

### 18.3. Segurança de Rede e Perímetro

Mesmo em arquitetura single node, o nó precisa ser protegido contra acessos externos não autorizados e tráfego malicioso.

Medidas típicas:

* **Hardening do Host**

  * Firewall configurado para:

    * Permitir apenas o tráfego essencial (saída para APIs de exchange, serviços de atualização e notificações);
    * Bloquear portas desnecessárias;
  * Atualizações regulares do sistema operacional e pacotes.

* **Conexões Criptografadas**

  * Todo tráfego entre o sistema e as exchanges utiliza **HTTPS/TLS**;
  * Se houver painéis de controle ou APIs internas expostas, essas interfaces:

    * Usam HTTPS;
    * Podem ser protegidas ainda por VPN ou túnel seguro, dependendo do cenário.

* **Proteção contra Exposição Involuntária**

  * Não expor diretamente na internet serviços que não precisem ser públicos (por exemplo, bancos de dados, coletores de métricas);
  * Se dashboards forem acessíveis remotamente:

    * Aplicar autenticação adequada;
    * Preferir acesso via túnel/VPN.

* **Rate Limits Internos**

  * O próprio sistema respeita e monitora limites de requisição das exchanges, reduzindo o risco de banimento ou bloqueio temporário.

---

### 18.4. Proteção de Dados e Privacidade

Embora o foco do sistema seja **dados de mercado e execução** (menos sensíveis que dados pessoais), ainda assim existe preocupação com proteção e privacidade:

* **Dados em Trânsito**

  * Conexões com exchanges e serviços externos usam protocolos seguros (TLS).
  * Em cenários com múltiplos componentes (futuro multi-node), comunicação interna também deve ser cifrada, conforme necessidade.

* **Dados em Descanso**

  * Bancos locais (SQLite, Parquet etc.) podem ser:

    * Armazenados em discos criptografados (LUKS/BitLocker, dependendo do ambiente);
    * Protegidos por permissões restritivas no sistema operacional.

* **Logs sem Dados Sensíveis**

  * Logs nunca incluem:

    * Chaves, segredos, tokens;
    * Informações que permitam reconstituir credenciais.
  * Qualquer dado que possa ser sensível é:

    * Mascarado;
    * Truncado;
    * Ou completamente omitido.

* **Dados de Terceiros**

  * Caso, em expansões futuras, haja integração com usuários externos ou dados de clientes:

    * Exigirá camada adicional de **governança de dados** (consentimento, minimização de dados, eventual adequação à LGPD/GDPR, dependendo da jurisdição).

---

### 18.5. Desenvolvimento Seguro (Secure SDLC)

A segurança não é tratada apenas na infraestrutura, mas também no **ciclo de desenvolvimento de software**:

* **Revisão de Código**

  * Mudanças em módulos críticos (risco, trading, credenciais, governança) passam por revisão obrigatória;
  * Revisores verificam:

    * Adesão a padrões de segurança;
    * Evitar lógicas perigosas (loops de envio de ordens, bypass de risco etc.).

* **Análise Estática e Varredura de Vulnerabilidades**

  * Utilização de ferramentas de:

    * Análise estática de código (ex.: linters de segurança típicos para Python);
    * Varredura de dependências (verificação de CVEs em bibliotecas usadas).
  * Execução periódica ou integrada ao pipeline de CI.

* **Gerência de Dependências**

  * Dependências externas são:

    * Fixadas em versões específicas (pinning);
    * Atualizadas de forma controlada (testes + backtests de regressão);
    * Verificadas contra bases de vulnerabilidades conhecidas.

* **Ambientes Separados**

  * Separação clara entre:

    * Desenvolvimento;
    * Teste/QA;
    * Produção.
  * Sempre que possível:

    * Uso de **testnet** das exchanges para testes de integração;
    * Só usar chaves de produção em ambiente estritamente controlado.

---

### 18.6. Ambientes, Testnet e Uso Responsável

A operação segura também depende de **ambientes bem delimitados**:

* **Ambiente de Desenvolvimento**

  * Utiliza dados históricos ou dados de mercado “mockados”;
  * Não deve ter acesso a credenciais de produção.

* **Ambiente de Testes / QA**

  * Pode usar:

    * Testnet de exchange (quando disponível);
    * Ou operação em “dry-run” total (execução simulada de ordens).
  * Ideal para:

    * Testes integrados do core loop;
    * Avaliar comportamento de estratégias sob condições de rede reais, sem risco financeiro.

* **Ambiente de Produção**

  * Usa apenas chaves reais de exchange, com:

    * Controle rígido de acesso;
    * Monitoramento intensivo (observabilidade).

* **Uso Responsável**

  * O sistema deve ser operado como **bot proprietário**, sem prestação de serviço a terceiros, a menos que sejam feitas as adequações regulatórias específicas (ver 18.7).
  * A responsabilidade pelo uso correto recai sobre o operador/proprietário, incluindo:

    * Respeito a limites operacionais;
    * Testes pré-deploy;
    * Monitoramento contínuo.

---

### 18.7. Requisitos Regulatórios e Compliance

O sistema é projetado para **trading proprietário**, operando capital próprio, o que simplifica as obrigações regulatórias. No entanto, qualquer evolução para gerir capital de terceiros ou oferecer serviços de investimento exigirá autorização de órgãos reguladores e adequação a normas específicas.
* **Registro e Retenção de Dados**

  * Mesmo como trading proprietário, manter:

    * Registros de ordens, execuções e decisões de risco;
    * Logs de governança;
    * Backups periódicos.
  * Facilita:

    * Eventual comprovação de boa-fé;
    * Investigações internas em caso de incidente.

* **Compliance com Políticas Internas**

  * Definição de:

    * Políticas internas de risco;
    * Limites de alavancagem (quando aplicável);
    * Procedimentos de aprovação de novas estratégias.
  * O sistema implementa essas políticas tecnicamente, mas o operador é responsável por zelar pela aderência.

> **Importante:** este documento não substitui aconselhamento jurídico.
> Qualquer evolução do sistema para cenários regulatórios mais complexos deve ser acompanhada de consultoria legal específica, considerando a jurisdição em que se opera.

---

### 18.8. Resposta a Incidentes e Continuidade de Negócios

O sistema possui um plano de resposta a incidentes para cenários como vazamento de credenciais, comportamento anômalo de estratégias e falhas de infraestrutura. Políticas de backup e recuperação de dados garantem a continuidade das operações.

Os procedimentos detalhados para operadores estão no **OPERATIONS_RUNBOOK.md**.

---

Com esse conjunto de práticas e mecanismos, a camada de **Segurança e Compliance** garante que o sistema:

* Proteja as credenciais e o capital sob gestão;
* Respeite princípios básicos de segurança de software e infraestrutura;
* Esteja pronto para evoluir, caso seja necessário atender exigências regulatórias mais rigorosas;
* Permita operar com confiança um **framework de famílias de estratégias** em ambiente real, ainda que em single node e inicialmente com uma única exchange.

---

## 19. Roadmap de Evolução

O Roadmap de Evolução define a progressão do sistema em fases lógicas, desde um protótipo até um framework maduro, multi-família e multi-ativo.

---

## 20. Ambiente de Desenvolvimento, Testes e CI/CD

Esta seção define o ambiente e os processos para garantir um desenvolvimento de software consistente, testável e seguro.

### 20.1. Setup de Desenvolvimento Local

*   **Ambiente Reprodutível:** O ambiente de desenvolvimento local é gerenciado via **Docker e Docker Compose**.
*   **Dependências Core:** O projeto utiliza **Python 3.11+** como base. As bibliotecas principais são gerenciadas por um gerenciador de pacotes (como Poetry ou pip com `requirements.txt`).
*   **Configuração de IDE:** Recomenda-se o uso de IDEs como VS Code ou PyCharm, com plugins para `black`, `ruff` e `mypy`.

### 20.2. Testes Automatizados

*   **Testes Unitários:** Utiliza-se `pytest`. Módulos críticos de domínio devem ter cobertura de testes superior a 80%.
*   **Testes de Integração:** Testes que validam a interação entre componentes, como a comunicação com adapters de exchange (usando mocks ou a **testnet** da exchange).
*   **Testes End-to-End (E2E):** Execução de um conjunto de **backtests de referência** que validam o pipeline completo, de dados a P&L, para garantir que mudanças no código não causem regressões inesperadas nos resultados das estratégias.

### 20.3. Pipeline de Integração e Deploy Contínuo (CI/CD)

*   **Linting e Formatação:** O pipeline de CI (ex: GitHub Actions) executa automaticamente `black` (formatação), `ruff` (linting rápido) e `mypy` (checagem de tipos) em cada Pull Request.
*   **Execução de Testes:** Todos os testes (unitários, integração) são executados no pipeline. Um PR só pode ser mesclado se todos os testes passarem.
*   **Deploy Controlado:** O deploy para ambientes de produção é feito de forma controlada, idealmente com tags de versão e um processo que permita um rollback rápido.
*   **Geração de Documentação:** O pipeline pode incluir um passo para gerar automaticamente diagramas de arquitetura (usando Mermaid) e outras partes da documentação a partir do código, garantindo que a documentação reflita o estado atual do sistema.


### 20.4. Git Workflow

*   **Feature Branches:** Todo o desenvolvimento é feito em `feature branches` a partir da branch principal (`main`).
*   **Pull Requests (PRs):** Mudanças só são incorporadas à branch principal através de Pull Requests, que exigem a aprovação de ao menos um outro revisor e a passagem de todos os checks de CI.
*   **Versionamento Semântico:** O projeto segue o Semantic Versioning (ex: `1.2.5`). As versões são gerenciadas através de tags no Git.
*   **Changelog:** Um `CHANGELOG.md` é mantido, idealmente de forma semi-automatizada, para registrar as mudanças significativas em cada versão.

---

## 21. Política de Gerenciamento de Artefatos

O sistema gera um grande volume de artefatos (logs, métricas, dados de backtest, etc.). Para evitar um "overflow" de dados e garantir a eficiência do armazenamento e da consulta, uma política de gerenciamento é implementada.

### 21.1. Agregação de Dados

*   **Princípio:** Em vez de salvar um arquivo para cada evento individual, os dados são agrupados em arquivos maiores por um período de tempo (ex: por dia) e por chaves de partição (ex: por símbolo).
*   **Formato:** **Apache Parquet** é o formato padrão para dados agregados, por ser colunar e otimizado para queries analíticas.
*   **Exemplo:**
    *   **Ruim:** `artifacts/scores/BTCUSDT_2024-01-01_14-30-00.json`
    *   **Bom:** `artifacts/scores/date=2024-01-01/symbol=BTCUSDT.parquet`

### 21.2. Compactação

*   **Algoritmo:** **ZSTD (nível 3)** é o padrão de compressão para artefatos armazenados, oferecendo um excelente balanço entre taxa de compressão e velocidade.

### 21.3. Rotação e Ciclo de Vida (Hot/Warm/Cold)

Uma política de ciclo de vida move os artefatos entre diferentes camadas de armazenamento para otimizar custo e performance.

| Camada | Propósito | Período de Retenção | Armazenamento Típico | Ação no Final |
| :--- | :--- | :--- | :--- | :--- |
| **Hot** | Acesso rápido para operações em tempo real. | 7-30 dias | SSD Local | Mover para Warm. |
| **Warm** | Análises de curto e médio prazo. | 30-90 dias | SSD/HDD Local | Mover para Cold. |
| **Cold** | Armazenamento de longo prazo para auditoria e retreinamento. | 1-5 anos | Armazenamento em nuvem de baixo custo ou HD externo. | Descartar ou arquivar. |

### 21.4. Indexação de Artefatos

*   Para facilitar a busca, um **índice de artefatos** é mantido em um banco de dados (SQLite).
*   Este índice armazena metadados sobre cada arquivo (caminho, checksum, timestamps, tags), permitindo queries rápidas sem a necessidade de listar milhões de arquivos no sistema de armazenamento.

### 21.5. Purga de Emergência

*   Um procedimento automatizado monitora o uso de disco. Se um limiar crítico (ex: 90%) for atingido, um processo de purga de emergência é iniciado, removendo os artefatos mais antigos e menos críticos (ex: caches intermediários, logs de debug) para liberar espaço e garantir a continuidade da operação.

## 21. Política de Gerenciamento de Artefatos
