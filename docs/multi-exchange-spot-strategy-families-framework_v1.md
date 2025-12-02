# Framework de Famílias de Estratégias Spot Multi-Exchange

## 1. Visão Geral do Projeto

Este documento descreve um **framework de famílias de estratégias de trading sistemático** voltado para mercados **spot** (inicialmente cripto, com foco em Binance Spot), orientado a **retorno ajustado ao risco** e projetado para operar em **single node**, porém com arquitetura preparada para **multi-exchange**.

Em vez de ser apenas “mais um bot de trading”, o projeto se propõe a ser uma **plataforma estruturada** onde:

* Famílias de estratégias com **evidência histórica robusta** (Trend Following, Dual Momentum, Cross-Sectional etc.) são implementadas de forma consistente;
* Há um **núcleo comum de risco, execução, dados e observabilidade**;
* Cada estratégia passa por um ciclo disciplinado de **pesquisa → backtest → paper trading → produção → monitoramento → eventual aposentadoria**.

A prioridade central é: **maximizar retorno ajustado ao risco, com disciplina de gestão de risco e governança**, evitando tanto improvisos operacionais quanto complexidade desnecessária para o contexto single node.

---

### 1.1. Contexto

Os mercados de cripto, FX e ações spot oferecem uma grande variedade de oportunidades, mas também exibem:

* Volatilidade elevada;
* Mudanças de regime frequentes;
* Risco de cauda significativo.

Paralelamente, a literatura acadêmica e empírica acumulou décadas de evidência sobre famílias de estratégias que apresentam, de forma relativamente consistente, **prêmios de risco e anomalias exploráveis**, como:

* **Time-Series Momentum (Trend Following)**;
* **Dual Momentum (TSM + Cross-Sectional)**;
* **Cross-Sectional Momentum**;
* Abordagens de **Value/Quality** em ações;
* **Mean Reversion** de curto prazo;
* Estratégias de **Carry/Yield/Basis**, quando a infraestrutura permite.

No entanto, na prática, muitos sistemas de trading “domésticos” misturam:

* Estratégias pouco testadas ou mal documentadas;
* Gestão de risco ad hoc;
* Falta de rastreabilidade das decisões;
* Acoplamento excessivo entre lógica de estratégia e infraestrutura (exchange, banco de dados, etc.).

Este projeto nasce justamente para **organizar** esse cenário: pegar o que há de mais sólido em termos de famílias de estratégias e encaixar dentro de uma arquitetura clara, extensível e operável em um ambiente realista (single node, uma exchange principal, sem HFT).

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

Alguns itens são explicitamente considerados **fora do escopo inicial** (embora possam ser considerados em fases futuras):

* **High-Frequency Trading (HFT) ou colocation**

  * Estratégias que exigem latência de microssegundos/milissegundos e acesso ultrabaixo nível à infraestrutura da exchange.

* **Derivativos Complexos**

  * Uso intensivo de futuros alavancados, opções estruturadas, spreads complexos, etc., como parte do núcleo do sistema.
  * Estratégias desse tipo só serão consideradas como módulos adicionais, depois que o framework spot estiver consolidado.

* **Gestão de Capital de Terceiros**

  * O sistema é desenhado como **trading proprietário** (capital próprio).
  * Qualquer evolução para operar recursos de terceiros ou prestar serviços de investimento exigirá avaliação regulatória e ajustes específicos (compliance ampliado).

* **Sistemas Distribuídos de Alta Complexidade**

  * No MVP, não há cluster de múltiplos nós, balanceadores de carga distribuídos ou arquitetura de microserviços full.
  * A prioridade é **robustez em single node**, com desenho que não impeça expansão futura.

* **Automação de Estratégias Puramente Discricionárias**

  * Estratégias que dependem de decisões humanas subjetivas em tempo real não fazem parte do framework – o foco é em **estratégias sistemáticas/documentadas**.

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

* O ambiente alvo é **single node**, com recursos suficientes (CPU/RAM/SSD) para:

  * Rodar estratégias em timeframes de minutos a dias;
  * Manter dados históricos locais para backtests;
  * Coletar e processar dados em tempo quase real.

* A **exchanges são tratadas como “fonte de verdade”** para:

  * Lista de símbolos, filtros de lote/notional, timeframes, tipos de ordem;
  * Disponibilidade de mercado (status de trading, pausas, delistings).

* As **famílias de estratégias** são baseadas em:

  * Evidência histórica robusta (papers, estudos, experiência prática);
  * Regras claras, sem “magia” ou sinais opacos.

* O foco é **spot long/flat**, sem dependência de alavancagem, margin ou derivativos para que a estratégia faça sentido.

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

* Nasce pragmático (single node, uma exchange, foco em spot long/flat);
* Se ancora em famílias de estratégias com forte evidência de retorno ajustado ao risco;
* Coloca gestão de risco, governança e observabilidade no mesmo nível de importância que a “inteligência” das estratégias;
* Se prepara, desde o desenho inicial, para crescer em complexidade (mais estratégias, mais exchanges, mais ativos) sem perder clareza e controle.


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

  * Locais (por estratégia/família);
  * Globais (parar o sistema ou entrar em modo seguro em caso de drawdown extremo ou incidentes).

Esse módulo deve atuar como **filtro obrigatório** entre sinais de estratégia e execução de ordens.

#### 2.1.6. Execução de Ordens Spot

* O sistema deve ser capaz de:

  * Traduzir posições alvo em **ordens spot concretas** (compra/venda) compatíveis com os filtros da exchange (lote mínimo, step, notional mínimo);
  * Enviar ordens via API da exchange (Binance Spot no MVP);
  * Lidar com:

    * Respostas de sucesso;
    * Erros de validação;
    * Rate limits;
    * Timeouts.

* Deve haver suporte a, no mínimo:

  * Ordens **market**;
  * Ordens **limit** (incluindo cancelamentos).

#### 2.1.7. Persistência de Estado Operacional

O sistema deve **persistir**:

* Posições abertas por ativo, por estratégia, por família;
* Ordens abertas e histórico de ordens/fills;
* P&L realizado e não realizado;
* Estado de risco (uso de budgets, drawdowns, exposições).

Esse estado deve ser recuperável após um restart e reconciliado com a exchange.

#### 2.1.8. Observabilidade, Auditoria e Relatórios

Funcionalmente, o sistema deve:

* Gerar **logs estruturados** de decisões e eventos críticos;
* Expor **métricas** de sistema (CPU, memória, latência) e de negócio (P&L, exposição, drawdown);
* Manter uma **trilha de auditoria** de:

  * Decisões de risco;
  * Alterações de configuração;
  * Rollouts/rollbacks de estratégias.

Deve ser possível gerar **relatórios periódicos** (diários, semanais, mensais) de performance e risco.

---

### 2.2. Requisitos Não Funcionais

Os requisitos não funcionais definem qualidades esperadas do sistema, além do “o que ele faz”.

#### 2.2.1. Desempenho e Escalabilidade (Single Node)

* O sistema deve operar confortavelmente em **single node**, com capacidade de:

  * Executar dezenas de instâncias de estratégias em timeframes de minutos a horas;
  * Coletar e armazenar dados históricos de múltiplos símbolos em timeframes típicos (1m–1d);
  * Manter latência de decisão compatível com trading não-HFT (ordens emitidas em segundos após o fechamento do candle, por exemplo).

* Deve ser possível **ajustar a carga** (número de estratégias, ativos, timeframes) conforme os recursos da máquina, sem necessidade de reestruturação profunda.

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

* O sistema será executado em **um único nó** (servidor dedicado, VM ou hardware embarcado de alto desempenho, como um Raspberry Pi potente).
* O sistema operacional alvo é **Linux** (ou derivado), com:

  * Acesso à internet estável;
  * Permissões de firewall configuráveis;
  * Recursos mínimos (CPU, RAM, SSD) adequados ao volume de dados/estratégias desejado.

#### 2.3.2. Conectividade com a Exchange

* Supõe-se que a exchange principal (inicialmente Binance Spot) oferece:

  * API REST estável para dados históricos e envio de ordens;
  * API WebSocket ou equivalente para dados em tempo real;
  * Documentação de filtros de mercado (lotes mínimos, notional, timeframes, etc.).

* Supõe-se também que:

  * A qualidade de dados da exchange é “suficiente” (eventuais outliers e gaps são gerenciáveis via lógica de saneamento);
  * A exchange seguirá sendo o **Single Source of Truth** para o universo de mercados e suas capacidades.

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

  * Define portas como contratos estáveis (por exemplo: `MarketDataPort`, `TradingPort`, `StoragePort`, `MetricsPort`, `NotificationPort`).
  * Cada exchange, banco de dados ou serviço externo é implementado como um adapter que “pluga” nessas portas.

* **Clean Architecture / Camadas em Anéis**

  * Núcleo de domínio no centro (entidades, regras de negócio, famílias de estratégias, políticas de risco).
  * Casos de uso na camada seguinte (orquestração do core loop, execução de backtests, promoção/rebaixamento de estratégias).
  * Infraestrutura e detalhes técnicos na camada externa (drivers de exchange, drivers de banco, APIs externas).

* **DDD leve focado em trading algorítmico**

  * Entidades principais:

    * **Asset / Symbol**, **Candle**, **Order**, **Fill/Trade**, **Position**, **Portfolio**,
    * **StrategyFamily**, **StrategyInstance**, **Signal**, **RiskLimit**, **AllocationPolicy**.
  * Linguagem ubíqua alinhada com o glossário para evitar ambiguidades na documentação e no código.

Esse estilo garante:

* **Testabilidade**: estratégias e regras de risco podem ser testadas sem subir exchange, banco, rede etc.;
* **Portabilidade**: migrar de Binance para outra exchange exige apenas novos adapters;
* **Evolutividade**: é possível introduzir novas famílias de estratégias sem refatorar o core.

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
   * Não conhece detalhes de exchange, storage ou logging. Recebe dados já normalizados e enriquecidos.

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
   * Também existe uma implementação “fake” para **backtest/simulação**, que consome os mesmos sinais.

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

Esse core loop é **idempotente onde possível**: reprocessar o mesmo bloco de dados não deve gerar ordens duplicadas, o que simplifica recuperação de falhas e replays para auditoria.

---

### 3.5. Estratégia de Escalabilidade e Evolução

Embora o alvo atual seja um **ambiente single node** e **single exchange (Binance Spot)**, a arquitetura foi planejada para evoluir sem reescrita massiva:

* **Escalabilidade Vertical (MVP)**

  * Otimização de CPU/RAM/IO no próprio nó (por exemplo, tuning de batch de indicadores, uso eficiente de disco);
  * Uso de filas assíncronas internas para desacoplar I/O pesado (coleta de dados, escrita em disco) da lógica de decisão.

* **Suporte Multi-Exchange por Design**

  * Ao adicionar uma nova exchange, basta implementar um novo adapter para `MarketDataPort` e `TradingPort`;
  * O domínio permanece intocado: mesmas famílias de estratégias, mesma lógica de risco e alocação.
  * Possibilidade futura de estratégias **cross-exchange** (arbitragem, alocação relativa) sem mudar o core.

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

Na versão inicial, o sistema opera com **uma única exchange**:

* **Binance Spot** como exchange primária.

No entanto, toda a arquitetura de dados e execução assume um cenário **multi-exchange**, ainda que o deployment seja single-node e, no MVP, single-exchange. Isso significa que:

* Existe uma **abstração de Exchange** (porta `ExchangePort` ou `MarketDataPort` + `TradingPort`) que define o “contrato mínimo”:

  * Listar símbolos negociáveis;
  * Listar filtros de trading (lote mínimo, step size, notional mínimo, limites de preço);
  * Listar timeframes/candlestick intervals suportados;
  * Coletar dados de mercado (candles, trades);
  * Enviar ordens spot e consultar status.

Cada exchange concreta (por exemplo, `BinanceExchangeAdapter`) implementa essa porta usando sua API própria (REST/WebSocket). A configuração apenas seleciona **qual adapter** será instanciado e, eventualmente, quais recursos dessa exchange serão habilitados por feature flag (ex.: permitir apenas mercado spot, bloquear margens, bloquear derivativos).

Para futuras exchanges (Kraken, Coinbase, etc.), o fluxo será:

1. Implementar um novo adapter que realize o **schema discovery** e normalize os dados para o modelo interno;
2. Registrar as capacidades da nova exchange (market data, tipos de ordem, timeframes) em uma estrutura interna;
3. Habilitar a exchange por **feature flag** (por exemplo: `features.exchange.kraken.enabled = true`) e definir quais famílias de estratégias podem operá-la.

---

### 4.2. Abstração Multi-Exchange

Do ponto de vista do domínio, o sistema não “enxerga” o nome da exchange; ele trabalha com:

* **Identificadores de mercado padronizados** (por exemplo, `ExchangeId`, `SymbolId`, `MarketId`);
* Interfaces genéricas de:

  * `get_historical_candles(market, timeframe, ...)`;
  * `stream_realtime_ticks(market, ...)`;
  * `send_order(market, side, quantity, price, order_type, ...)`.

Essa camada de abstração é responsável por:

* **Mapear símbolos nativos da exchange** (ex.: `BTCUSDT`, `BTC-USDT`, `XBTUSD`) para um formato interno padronizado;
* Conhecer **as capacidades daquela exchange** (por exemplo: se suporta stop-limit, OCO, post-only, iceberg, etc.);
* Expor essas capacidades ao restante do sistema via **feature flags de capacidade**, por exemplo:

  * `features.exchange.binance.spot.oco_orders = true`;
  * `features.exchange.binance.spot.margin_trading = false` (bloqueado pelo sistema mesmo que a exchange suporte).

Dessa forma, uma mesma família de estratégia pode operar em múltiplas exchanges, desde que os **requisitos mínimos de mercado** estejam disponíveis (por exemplo, necessidade de um mínimo de liquidez, de um conjunto de timeframes, ou suporte a tipos específicos de ordem).

---

### 4.3. Classes de Ativos e Símbolos

Inicialmente, o universo é focado em:

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

---

### 4.4. Timeframes Suportados

Um ponto fundamental é: **quem define os timeframes suportados não é o sistema, é a exchange**.

O sistema não “inventa” timeframes. Em vez disso:

1. Na inicialização (e periodicamente), o adapter da exchange consulta:

   * Lista de **intervalos de candles** suportados (tipicamente via endpoint de exchange ou documentação oficial);
2. Mapeia esses intervals para uma enum interna (`TimeframeDescriptor`), por exemplo:

   * `1s`, `1m`, `3m`, `5m`, `15m`, `30m`, `1h`, `4h`, `1d`, etc.;
3. Marca cada timeframe como:

   * `supported = true/false` para aquela exchange específica;
   * eventuais restrições (mínimo histórico disponível, limite de requisições).

Na configuração, as estratégias apenas escolhem **entre timeframes que foram previamente descobertos e marcados como válidos**. Se uma configuração tentar usar um timeframe não suportado pela exchange, o sistema:

* Rejeita a configuração em tempo de carga (falha de validação Pydantic/Schema);
* Ou desabilita a estratégia com log de erro claro.

A seleção de timeframes pode ser governada por **feature flags**, por exemplo:

* `features.timeframe.intraday_only = true` → desativa automaticamente estratégias que tentem usar `1d` em um contexto que só aceita intraday;
* `features.exchange.binance.timeframes.allowed = ["1m","5m","1h","1d"]` → ignora timeframes exóticos mesmo que a exchange suporte, para simplificar e reduzir carga.

Isso garante que:

* O núcleo do sistema seja **agnóstico à lista exata de timeframes**;
* Alterações futuras na exchange (adição/remoção de intervalos) possam ser absorvidas sem alterar código, apenas via novo ciclo de schema discovery.

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

* Uso de **SQLite** para:

  * Metadados de mercados (lista de símbolos, filtros, timeframes suportados);
  * Estado operacional (posições, ordens, trades);
  * Registros de execução de estratégias (sinais, decisões de risco, P&L agregado).

* Uso de formatos de arquivo do tipo **Parquet/CSV** (ou similar) para:

  * Séries históricas de candles por `exchange/symbol/timeframe`;
  * Armazenamento de séries derivadas usadas em pesquisas e backtests (indicadores, scores, labels).

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

Esses dois mecanismos juntos garantem que:

* O **universo de mercados é derivado da realidade da exchange**, não de suposições;
* Mudanças na exchange (adição ou remoção de timeframes, alterações de filtros) sejam detectadas e refletidas no sistema, com logs e, se necessário, desativação automática de estratégias afetadas;
* A evolução controlada (progressive delivery) de novas capacidades e novas exchanges seja feita de forma **segura e auditável**, sem alterar a base conceitual do sistema.

---

Com isso, o “Universo de Mercados, Exchanges e Dados” deixa de ser um conjunto de constantes hardcoded e passa a ser uma **camada viva, descoberta e validada em tempo de execução**, sobre a qual as famílias de estratégias podem operar com segurança, robustez e previsibilidade.

---

## 5. Framework de Estratégias e Famílias

O framework de estratégias é o “coração lógico” do sistema: é nele que as **famílias de estratégias** (Trend Following, Dual Momentum, Cross-Sectional, Carry, Value/Quality, Mean Reversion, Market Making) são definidas, versionadas, configuradas e orquestradas.
O objetivo é garantir que todas as estratégias, independentemente da família, sigam um **contrato comum**, possam ser **testadas de forma isomórfica** (mesma lógica em backtest e produção) e sejam facilmente ativadas/desativadas por configuração, sem alterações em código de infraestrutura.

---

### 5.1. Princípios

O framework de estratégias é guiado por alguns princípios centrais:

* **Sistematicidade e Documentação**
  Cada estratégia deve ser completamente especificada em termos de regras formais (sem elementos discricionários), com hipótese explícita, referências de evidência histórica e parâmetros bem definidos.
  A documentação da família deve incluir: premissa, regime de mercado em que tende a funcionar melhor, riscos típicos, métricas-alvo e limitações conhecidas.

* **Separação entre Família e Instância de Estratégia**
  A “família de estratégia” representa o **conceito** (por exemplo, “TSM long/flat em janelas de 3 a 12 meses”).
  Cada “instância de estratégia” é uma configuração concreta dessa família: conjunto de ativos, timeframes, parâmetros específicos, limites de risco e modo de operação (backtest/paper/live).
  Isso permite que uma mesma família tenha múltiplas instâncias em paralelo, com configurações diferentes, sem duplicação de lógica.

* **Desacoplamento de Infraestrutura**
  Estratégias não conhecem detalhes de exchange, banco de dados, filas ou logs. Elas recebem um contexto de dados já normalizado e enriquecido, e produzem **sinais de posição alvo**.
  A execução física (ordens), gestão de risco, logging e persistência são responsabilidade de outros módulos.

* **Reuso entre Backtest e Produção**
  A lógica de geração de sinais deve ser exatamente a mesma em backtest, paper trading e live trading.
  A diferença está apenas nos adapters: em backtest, o “executor” é simulado; em produção, é o adapter real (por exemplo, Binance Spot).

* **Configuração Declarativa e Type-Safe**
  Estratégias são ativadas/desativadas por configuração (YAML/JSON) validada por modelos fortes (Pydantic ou similar).
  Isso reduz a chance de erro de configuração e facilita versionamento, revisão e auditoria.

---

### 5.2. Interface de Estratégia (Strategy API)

Todas as estratégias, independentemente da família, implementam uma interface comum, a **Strategy API**, que define o contrato mínimo para interação com o orquestrador.

Em termos conceituais, uma estratégia expõe, no mínimo:

* Um método de **preparação**:

  * Responsável por validar a configuração, declarar os requisitos mínimos de dados (lookback, indicadores, features), e inicializar qualquer estado interno necessário.
    Exemplo: `prepare(strategy_config, market_context, data_schema)`.

* Um método de **execução principal** (core de decisão):

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

  * Lista de `markets` (por exemplo, `["BINANCE:BTCUSDT", "BINANCE:ETHUSDT"]`);
  * `timeframe` principal e, se aplicável, timeframes auxiliares;
  * Requisitos de liquidez mínima, spread máximo, volume mínimo.

* **Parâmetros da Família**

  * Por exemplo, para TSM: janelas de lookback (3, 6, 12 meses), regras de entrada/saída, thresholds de sinal, filtros adicionais (volatilidade mínima, volatilidade máxima, etc.).

* **Políticas de Risco Locais**

  * Risco máximo por trade da instância;
  * Limite de alocação por ativo dentro da instância;
  * Stop conditions específicas (por exemplo, desligar instância se drawdown > X%).

* **Metadados Operacionais**

  * Modo de execução (`backtest`, `paper`, `live`);
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

### 5.6. Critérios de Priorização de Famílias

Nem todas as famílias de estratégias têm o mesmo nível de robustez de evidência, nem os mesmos requisitos de infraestrutura. Por isso, o framework inclui um **modelo explícito de priorização**, que pode ser usado tanto no roadmap quanto na alocação de capital e recursos de engenharia.

Critérios principais:

* **Robustez de Evidência Histórica**
  Famílias com literatura mais ampla e replicada (por exemplo, Trend Following / Time-Series Momentum) recebem prioridade mais alta.

* **Retorno Ajustado ao Risco (Sharpe/Sortino/Calmar)**
  Famílias que historicamente exibem melhor perfil de retorno ajustado a risco, e menor vulnerabilidade a crashes extremos, são preferidas para os primeiros ciclos de produção.

* **Viabilidade Prática em Spot**
  Famílias que funcionam bem em spot long/flat, sem necessidade de derivativos complexos ou HFT (por exemplo, TSM, Dual Momentum, Value/Quality), têm prioridade sobre famílias que exigem latência baixíssima, acesso profundo ao book ou derivativos com alavancagem.

* **Complexidade Operacional**
  Famílias com menor complexidade de implementação e operação (por exemplo, TSM diário) são incluídas antes de famílias intraday sensíveis a custos e slippage (como Mean Reversion intraday ou Market Making).

Esse modelo orienta tanto o **roadmap** (qual família vem primeiro) quanto decisões de **alocação de capital** e de tempo de engenharia, mantendo alinhamento entre teoria, evidência e realidade operacional.

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

## 6. Família 1 – Trend Following / Time-Series Momentum (TSM)

A família **Trend Following / Time-Series Momentum (TSM)** é o pilar principal do framework.
Ela é tratada como a **primeira família de produção** porque combina:

* evidência empírica e acadêmica robusta em múltiplas classes de ativos;
* implementação relativamente simples;
* boa relação **retorno x risco**, com capacidade de reduzir exposição em grandes crises;
* perfeita aderência ao modo **spot long/flat** (sem necessidade de short ou derivativos).

---

### 6.1. Descrição Conceitual

Time-Series Momentum (TSM) parte de uma ideia simples:

> Se o retorno recente de um ativo foi positivo, há maior probabilidade de que a tendência de alta continue por algum tempo.
> Se o retorno recente foi negativo, há maior probabilidade de que a tendência de baixa continue.

Em termos práticos, para cada ativo:

* calcula-se o **retorno acumulado** em uma janela de lookback (ex.: 3, 6 ou 12 meses, ou X dias);
* se esse retorno é **positivo** acima de um certo limiar, a estratégia assume que a tendência é de alta e busca ficar **comprada**;
* se o retorno é **negativo** (ou não suficientemente positivo), a estratégia **não mantém posição** (fica em cash / stable).

No contexto do projeto:

* TSM é sempre implementado em versão **long/flat**, nunca short “seco”;
* a lógica pode ser aplicada a um único ativo (por exemplo, BTCUSDT) ou a um **conjunto de ativos**, avaliando a tendência individual de cada um.

Essa abordagem tende a:

* participar das grandes pernadas de tendência (bull markets);
* reduzir ou eliminar exposição em crises severas (quando os retornos acumulados tornam-se fortemente negativos).

---

### 6.2. Evidência Histórica e Referências

Embora este documento não liste exaustivamente os estudos, a família TSM/Trend Following se apoia em um corpo significativo de evidência:

* Estudos clássicos mostram que **time-series momentum** é observável em:

  * índices de ações,
  * moedas (FX),
  * commodities,
  * títulos de renda fixa (bonds),
    ao longo de décadas e em vários mercados.

* Em cripto, diversos trabalhos e backtests independentes sugerem que:

  * aplicar TSM em **BTC** e em **cestas de criptos** mantém poder explicativo;
  * o principal desafio está em custos, slippage e mudanças de regime, não na inexistência do efeito.

No projeto, TSM é tratado como:

* a família com **maior prioridade** na fase inicial;
* referência para avaliar a qualidade de outras famílias (Dual Momentum, Cross-Sectional, Mean Reversion etc.), tanto em performance quanto em robustez.

---

### 6.3. Adaptação para Spot Long/Flat

Para aderir ao escopo do projeto (spot long/flat, sem short), a adaptação de TSM segue alguns princípios:

1. **Sinal de Tendência Baseado em Retorno Acumulado**

Para cada ativo `A` e timeframe principal `T` (por exemplo, diário):

* calcula-se o retorno acumulado em uma janela de lookback `L` (ex.: 90, 180, 252 candles):

  * pode ser retorno simples, log-return acumulado ou combinação de retornos diários;
* define-se um limiar `threshold` (por exemplo, 0% ou levemente positivo).

Regra básica:

* se `retorno_L > threshold` → sinal de **tendência de alta** → posição alvo positiva (ex.: 100% do capital designado para aquele ativo);
* se `retorno_L ≤ threshold` → **sem posição** naquele ativo (cash/stable).

2. **Posição Long/Flat (sem short)**

* Em vez de inverter a posição (long → short) quando o sinal é negativo, a estratégia simplesmente **zera** a exposição;
* Essa abordagem é compatível com:

  * restrições de infraestrutura (spot, sem margem/alavancagem);
  * filosofia de reduzir risco em crises, ficando fora do mercado ou em stablecoins.

3. **Interação com o Módulo de Risco**

* A estratégia não decide “quantas unidades” comprar, mas **qual percentual de alocação alvo** por ativo (por exemplo, 0%, 50% ou 100% da parcela de capital designada à família TSM);
* O módulo de risco:

  * converte esses alvos em tamanhos de posição compatíveis com limites de risco (por trade, por ativo, por família, por exchange);
  * garante que limites globais de exposição não sejam ultrapassados (seção 13).

---

### 6.4. Variantes

A família TSM contempla uma série de **variantes estruturais**, todas com a mesma essência, mas com nuances diferentes:

1. **TSM Single Asset, Single Timeframe (versão mínima)**

   * Aplicada, por exemplo, apenas em BTCUSDT no timeframe diário;
   * Usa uma única janela de lookback (ex.: 200 dias);
   * É a versão candidata para **MVP de produção**.

2. **TSM Multi-Asset (mesma janela para vários ativos)**

   * Aplica a mesma regra de TSM para um **conjunto de ativos** (BTC, ETH, SOL, etc.);
   * Cada ativo pode estar em:

     * estado “tendência positiva” (comprado);
     * ou “sem tendência” (cash).

3. **TSM Multi-Horizonte (combinação de janelas)**

   * Combina sinais de janelas diferentes (curta, média, longa):

     * Ex.: 50, 100, 200 dias;
   * Pode utilizar:

     * média ponderada de sinais;
     * regras de consenso (“só entra se 2 de 3 horizontes forem positivos”).

4. **TSM com Target Volatility (Vol Target)**

   * Ajusta o tamanho da posição de acordo com a **volatilidade recente**:

     * em períodos de alta volatilidade, reduz posição;
     * em períodos de baixa volatilidade, aumenta posição (até limites de risco).
   * Visa manter um nível mais constante de risco por ativo/família.

5. **TSM com Filtro de Regime de Mercado**

   * Adiciona filtros baseados em:

     * volatilidade global;
     * liquidez;
     * ou mesmo em proxies de “risk-on/risk-off” (ex.: performance de índices amplos).
   * Em regimes considerados desfavoráveis, suspende ou reduz agressivamente a atuação.

Essas variantes são planejadas como **camadas evolutivas**: o MVP começa com uma forma simples de TSM e, gradualmente, incorpora versões mais sofisticadas.

---

### 6.5. Sinais e Filtros

Além do sinal principal de tendência (retorno no lookback), TSM utiliza uma série de **filtros** para tornar os sinais mais robustos:

1. **Sinal de Tendência Principal**

* `TSM_signal(A, L)` ∈ {+1, 0}, onde:

  * `+1` indica tendência de alta (retorno acumulado > threshold);
  * `0` indica ausência de tendência suficiente (permanece em cash).

2. **Filtros de Volatilidade**

* Filtros que evitam operar:

  * em volatilidade extremamente alta (cenário de “whipsaw”);
  * ou extremamente baixa (mercado sem direção clara).
* Exemplos:

  * ATR normalizado em relação ao preço;
  * Desvio-padrão de retornos em janela curta.

3. **Filtros de Liquidez**

* Garante que:

  * apenas ativos com volume médio acima de um mínimo operem;
  * não sejam abertas posições em ativos que sofreram queda súbita de volume ou tenham spreads muito largos (quando houver dados de book).

4. **Filtros de Integridade de Dados**

* Se o período de lookback apresenta:

  * buracos de dados significativos;
  * candles marcados como “suspeitos” ou “corrompidos”;
* A geração de sinal pode ser suspensa, para evitar decisões com base em dados ruins.

5. **Filtros de Overlap com Outras Famílias (futuro)**

* Em versões mais avançadas, TSM pode:

  * receber sinal verde ou amarelo do módulo de portfólio, dependendo de conflitos ou redundâncias com outras famílias (por exemplo, Dual Momentum fortemente alinhado).

Todos esses filtros se materializam como **flags** e/ou ajustes de intensidade do sinal (`+1`, `0`, ou diferentes pesos de posição alvo) antes do envio para o módulo de risco.

---

### 6.6. Parametrização

A família TSM é altamente parametrizável, mas, ao mesmo tempo, é necessário evitar **overfitting**.
Os principais grupos de parâmetros são:

1. **Parâmetros de Tendência**

* `lookback_window`

  * Janela em número de candles (ex.: 60, 120, 252).
* `threshold`

  * Limiar mínimo de retorno acumulado para considerar que há tendência (ex.: 0%, 3%).
* `aggregation_method`

  * Método de cálculo (retorno simples, log-return, média ponderada de retornos etc.).

2. **Parâmetros de Volatilidade e Filtros**

* `vol_window`

  * Janela para cálculo de volatilidade;
* `max_volatility` / `min_volatility`

  * Faixa de vol aceitável para operar;
* `liquidity_min_volume`

  * Volume mínimo médio nos últimos N candles.

3. **Parâmetros de Target Vol (quando aplicável)**

* `target_vol`

  * Volatilidade anualizada desejada para a posição;
* `vol_position_scaling`

  * Fórmula de ajuste de tamanho de posição em função da volatilidade recente.

4. **Parâmetros Operacionais**

* `rebalance_frequency`

  * Frequência de reavaliação da posição (por exemplo, a cada candle do timeframe principal, ou a cada X candles);
* `max_position_per_asset`

  * Cap local (por estratégia/instância) de exposição em um ativo específico;
* `max_positions`

  * Número máximo de ativos com posição simultânea na instância TSM multi-asset.

5. **Parâmetros de Risco (locais à família)**

* Risco máximo por trade, por instância e por família (coerentes com o módulo global de risco);
* Parâmetros de circuit breaker específicos para TSM (ex.: DD máximo da família antes de desativar).

Esses parâmetros são definidos em arquivos de configuração (YAML/JSON) por instância de estratégia, validados e versionados conforme descrito na seção de Governança (17).

---

### 6.7. Métricas Específicas

Além das métricas globais de performance (retorno, Sharpe, Sortino, Calmar, DD máximo), TSM é avaliado por algumas métricas específicas:

1. **Tempo em Posição vs Cash**

* Percentual de tempo em que:

  * a instância esteve comprada;
  * a instância esteve em cash.
* Ajuda a entender se a estratégia:

  * é mais “participativa” (sempre posicionada) ou
  * um filtro de grandes tendências (posicionada só em momentos específicos).

2. **Comportamento em Crises**

* Performance em janelas conhecidas de crise (ex.: grandes quedas de mercado):

  * quão cedo a estratégia saiu do mercado;
  * qual foi o DD na crise comparado ao buy & hold.

3. **Qualidade das Tendências Capturadas**

* Relação entre:

  * número de sinais de entrada;
  * tamanho médio das pernadas vencedoras;
  * número de falsos rompimentos (whipsaws).

4. **Estabilidade por Regime de Mercado**

* Quebra de resultados em:

  * períodos bull;
  * períodos bear;
  * períodos laterais.
* O objetivo é confirmar o comportamento esperado:

  * ganhar mais em mercados tendenciais;
  * perder pouco (ou pouco operar) em mercados lateralizados.

5. **Sensibilidade a Parâmetros**

* Avaliação de robustez:

  * se pequenas mudanças em `lookback_window` ou `threshold` mantêm performance similar;
* Estratégia considerada mais confiável quando **não depende de um único ponto ótimo**.

Essas métricas são usadas tanto em **backtest** quanto em **produção**, alimentando o processo de governança (manter, ajustar, reduzir capital ou desativar a instância).

---

### 6.8. Roadmap de Implementação

A implementação da família TSM será feita em etapas bem definidas, alinhadas com o restante do projeto:

1. **TSM Básico Single Asset (MVP Técnico)**

   * Implementar `TSMStrategy` para um único ativo (por exemplo, BTCUSDT) em timeframe diário;
   * Integrar à Strategy API (métodos `prepare`, `generate_signals`, `post_trade_update`);
   * Validar com backtests simples, sem otimização agressiva.

2. **Integração com Motor de Backtest e Risco**

   * Executar backtests com o engine unificado, já usando:

     * módulo de risco multicamadas;
     * modelagem de custos/slippage.
   * Gerar primeiro conjunto de relatórios formais de desempenho.

3. **Paper Trading em Binance Spot**

   * Rodar TSM básico em paper trading:

     * monitorar latência, integridade dos dados, frequência de ordens;
     * comparar sinais e execuções simuladas com o esperado.

4. **Promoção Controlada para Live (Capital Limitado)**

   * Iniciar live trading com:

     * alocação pequena de capital;
     * monitoramento reforçado de métricas e eventos de risco;
   * Ajustar parâmetros operacionais (por exemplo, rebalance frequency, limites de ordem).

5. **Extensão para TSM Multi-Asset**

   * Adicionar suporte para múltiplos símbolos (cesta de criptos) com:

     * seleção de universo baseada em liquidez e filtros de qualidade;
     * limites de exposição por ativo e por família.

6. **Adição de Target Volatility e Filtros de Regime**

   * Implementar variantes de TSM com:

     * posição ajustada à volatilidade;
     * filtros de regime mais sofisticados (por ex., desligar TSM em determinados cenários globais, se definido em governança).

7. **Integração com Multi-Exchange (fase posterior)**

   * Quando novas exchanges forem adicionadas:

     * permitir instâncias TSM por exchange;
     * avaliar TSM cross-exchange, respeitando limites de risco por exchange.

Ao final desse roadmap, TSM deve se consolidar como:

* a **família âncora** do portfólio;
* o **padrão de referência** para robustez e governança;
* a primeira linha de defesa na construção de um portfólio de estratégias com foco em **retorno ajustado ao risco em mercado spot**.


## 7. Família 2 – Dual Momentum (TSM + Cross-Sectional)

A família **Dual Momentum** combina duas ideias que, isoladamente, já têm bom histórico de evidência empírica:

* **Time-Series Momentum (TSM)** – tendência do próprio ativo ao longo do tempo;
* **Cross-Sectional Momentum (CSM)** – desempenho relativo entre ativos de um mesmo universo.

Na prática, a abordagem Dual Momentum:

1. **filtra** o universo de ativos por TSM (só entra quem está “em tendência favorável” individualmente);
2. **ranqueia** apenas esse subconjunto pelos retornos relativos (CSM);
3. seleciona os **melhores colocados** para alocação.

É, portanto, uma família naturalmente adequada para **portfólios multi-ativos em spot** e com foco forte em **retorno ajustado ao risco**.

---

### 7.1. Descrição Conceitual

A lógica básica de Dual Momentum pode ser descrita em três passos:

1. **Filtro de TSM (momentum absoluto / time-series)**
   Para cada ativo do universo:

   * calcula-se o **retorno acumulado** em uma janela de lookback (ex.: 6–12 meses ou N candles do timeframe de referência);
   * se o retorno é **positivo acima de um limiar** (por ex., > 0%), o ativo passa no filtro TSM;
   * se o retorno é **negativo ou muito baixo**, o ativo é temporariamente excluído do universo elegível.

2. **Ranqueamento Cross-Sectional (momentum relativo)**

   * Entre os ativos que passaram no filtro TSM, calcula-se um **score de performance relativa** (ex.: retorno no mesmo lookback ou em janela complementar);
   * Os ativos são ordenados do “melhor” (maior retorno) para o “pior” (menor retorno);
   * Seleciona-se o **top N** ou **top X%** para compor o portfólio.

3. **Alocação em Long-Only / Long-Flat**

   * Em versão neutra (long-short), normalmente se compram os vencedores e se vendem os perdedores;
   * No projeto, a abordagem é **long-only / long-flat**:

     * Apenas os vencedores são comprados;
     * Os não selecionados ficam em **cash/stable**.

O resultado é um portfólio que tende a:

* concentrar capital nos ativos “em tendência positiva” e que **superam os pares**;
* reduzir exposição ou ficar em cash quando o universo inteiro está com TSM fraco (todos “mal das pernas”).

---

### 7.2. Evidência Histórica e Referências

Dual Momentum se apoia em dois blocos de evidência:

1. **Time-Series Momentum (TSM)**

   * Forte evidência em diversas classes de ativos (ações, FX, commodities, bonds), ao longo de décadas;
   * Tende a capturar tendências de médio prazo e proteger em grandes crises (por ficar fora quando o ativo entra em tendência negativa).

2. **Cross-Sectional Momentum**

   * Estudos mostram que, dentro de um universo de ativos (por exemplo, ações de um índice), os **vencedores recentes tendem a continuar vencendo** por algum tempo, e perdedores permanecem fracos;
   * Em versão long/short, costuma ter retornos altos, porém com risco de “momentum crashes”.

Ao combinar ambos:

* Dual Momentum **filtra** por TSM para diminuir a probabilidade de entrar em ativos em “regime ruim”;
* Dentro dos filtrados, **ranqueia** por CSM para aproveitar a “seleção dos melhores cavalos da corrida”.

Na prática, essa combinação costuma:

* reduzir drawdowns em relação ao momentum puramente cross-sectional;
* melhorar Sharpe/Sortino, ao cortar parte dos períodos em que “todo o universo está ruim”.

---

### 7.3. Adaptação para Spot Long-Only

No escopo do projeto (spot, long/flat), Dual Momentum é implementado como **long-only com cash como contraparte**, sem short nem alavancagem. Os princípios de adaptação são:

1. **Nada de short estruturado**

   * Mesmo que haja ativos “perdedores”, a estratégia **não opera vendida** neles;
   * Em vez disso, simplesmente não aloca (ou reduz) capital nesses ativos.

2. **Cash/Stable como “ativo neutro”**

   * Quando:

     * poucos ativos passam no filtro TSM;
     * ou o universo inteiro está ruim;
   * parte relevante do portfólio pode permanecer em **cash/USDT/stablecoin**, reduzindo risco direcional.

3. **Alocação Proporcional apenas nos “elegíveis”**

   * O módulo de risco/portfólio distribui o capital da família entre:

     * Ativos selecionados pelo ranking (top N / X%);
     * Cash/stable (restante não utilizado).

4. **Aderência à infraestrutura spot**

   * Toda execução é feita por **compras/vendas spot** (sem margem), via adapters da exchange;
   * Tamanhos de posição respeitam filtros de lote/notional, definidos pela camada de schema discovery.

---

### 7.4. Universo de Ativos

Dual Momentum faz mais sentido quando há um **universo minimamente diversificado de ativos**, idealmente com:

* ativos correlacionados, porém com diferenças relevantes de performance (por ex., várias criptos grandes e médias; ou carteira de ações setoriais);
* liquidez suficiente para execução dos sinais.

No contexto do projeto:

1. **Versão inicial (cripto spot, uma exchange)**

   * Universo derivado de Binance Spot (ou exchange foco), com filtros de:

     * Volume médio mínimo;
     * Histórico disponível (janela suficiente para cálculo dos retornos);
     * Spreads razoáveis (quando houver dados).

   * Exemplos de universos:

     * “Top 10–20 criptos por market cap e liquidez em USDT”;
     * Subconjuntos temáticos (L1s, DeFi blue chips, etc.), se fizer sentido.

2. **Expansão futura**

   * Ao adicionar outras exchanges ou classes de ativos (ex.: ações tokenizadas, FX via outros canais), o universo pode ser:

     * Multiexchange;
     * Multiclasse (por exemplo, misturando cripto e proxies de outras classes, em implementações avançadas).

3. **Integração com Módulo de Dados**

   * O universo é definido via:

     * **feature flags** e configuração (quais mercados estão habilitados para Dual Momentum);
     * validação de schema discovery (só ativos em estado “TRADING” e com dados confiáveis).

---

### 7.5. Sinais e Regras de Seleção

O fluxo de decisão de Dual Momentum, em cada ciclo de rebalance, segue em linhas gerais:

1. **Cálculo de Momentum Absoluto (TSM)**

   * Para cada ativo `A` do universo, calculam-se:

     * `ret_TS(A)` = retorno acumulado em uma janela de lookback de time-series (ex.: 6 ou 12 meses / N candles).

   * Critério TSM:

     * Se `ret_TS(A) > threshold_TS` → ativo passa no filtro;
     * Caso contrário, é excluído da seleção atual.

2. **Cálculo de Momentum Relativo (Cross-Sectional)**

   * Entre os ativos que passaram em TSM:

     * Calcula-se um **score cross-sectional**, por exemplo:

       * `ret_CS(A)` = retorno em janela idêntica ou distinta (3–6 meses, ou outro período);
       * ou combinações (média ponderada de retornos de curto/médio prazo);
     * Os ativos são ordenados por `ret_CS(A)` (do maior para o menor).

3. **Seleção do Top N / Top X%**

   * Define-se:

     * `N` = número fixo de ativos a serem selecionados;
     * ou `p%` = percentil superior (ex.: top 20%).
   * O conjunto de ativos selecionados `S` é, então, aqueles com maior score cross-sectional dentro do filtro TSM.

4. **Definição de Posições Alvo**

   * A estratégia define, por ativo:

     * Se `A ∈ S` → alvo de alocação > 0 (por exemplo, peso igual, ou peso proporcional ao score);
     * Se `A ∉ S` → alvo = 0 (cash).

   * O módulo de risco/portfólio ajusta esses alvos:

     * Respeitando limites por ativo/família;
     * Normalizando para o capital disponível e restrições globais.

5. **Frequência de Rebalance**

   * As decisões de Dual Momentum são, por natureza, **menos frequentes** que estratégias intraday:

     * Rebalance semanal, quinzenal ou mensal é comum;
     * No projeto, isso é configurável, mas tende a seguir timeframes mais “calmos” (ex.: 1D, 3D, 1W).

---

### 7.6. Parametrização

Dual Momentum tem um conjunto de parâmetros que governa sua sensibilidade e comportamento:

1. **Parâmetros de Time-Series (TSM)**

* `ts_lookback_window`

  * Janela de retorno absoluto (ex.: 126, 252 candles diários);
* `ts_threshold`

  * Limiar mínimo para considerar TSM positivo (ex.: 0% ou > X%).

2. **Parâmetros Cross-Sectional**

* `cs_lookback_window`

  * Janela de retorno relativo (pode ser igual ou diferente da TS);
* `cs_score_method`

  * Método de ranking:

    * retorno simples;
    * log-return;
    * combinação de horizontes (ex.: 1m + 3m + 6m).

3. **Seleção e Tamanho do Portfólio**

* `selection_mode`

  * `top_n` (número fixo de ativos);
  * ou `top_percentile` (top X% do universo elegível).
* `max_assets`

  * Limite superior de ativos simultaneamente no portfólio Dual Momentum.

4. **Alocação Interna**

* `weighting_scheme`

  * Igualitária (cada ativo selecionado recebe o mesmo peso);
  * Proporcional ao score (ativos com maior momentum recebem maior peso);
  * Proporcional ao inverso da volatilidade (favorendo ativos menos voláteis).

5. **Parâmetros de Risco Locais**

* `max_exposure_per_asset`

  * Exposição máxima em um único ativo dentro da família;
* `family_risk_budget`

  * Parcela de capital total destinada à família Dual Momentum;
* `rebalance_frequency`

  * Intervalo entre rebalances (por ex., rebalance mensal com checks semanais).

6. **Filtros de Liquidez e Qualidade**

* `min_avg_volume`

  * Volume mínimo médio em janela recente;
* `min_price`

  * Preço mínimo em moeda de referência para evitar ativos “sub-penny” ou com micro-preços;
* `excluded_symbols`

  * Lista de mercados proibidos (por motivos de risco, liquidez ou governança).

Todos esses parâmetros são definidos por **instância de estratégia**, validados por modelos tipados e versionados para garantir rastreabilidade.

---

### 7.7. Risco Específico

Embora Dual Momentum tenha, em geral, um perfil de risco mais controlado do que momentum puramente cross-sectional, há riscos específicos que precisam ser explicitamente tratados:

1. **Risco de Concentração em Poucos Ativos**

* Em certos períodos, o filtro TSM + ranking CS pode:

  * Concentrar a alocação em apenas 1–3 ativos;
  * Isso aumenta o risco idiossincrático (eventos específicos de um projeto/token/ativo).
* Mitigações:

  * Limites por ativo (`max_exposure_per_asset`);
  * Mínimo de ativos selecionados (se não houver ativos suficientes, parte do portfólio permanece em cash).

2. **Risco de Rotação Brusca**

* Quando o ranking muda fortemente entre rebalances, pode haver:

  * Rotação frequente de ativos;
  * Aumento de custos e slippage.
* Mitigações:

  * `rebalance_band`: zona de tolerância para evitar trocas por diferenças pequenas de score;
  * Limitação da fração do portfólio que pode girar a cada rebalance (turnover cap).

3. **Risco de Regime em que “Tudo Vai Mal”**

* Em crises sistêmicas, o universo inteiro pode estar com TSM ruim:

  * Dual Momentum, nesse caso, tende a mandar o portfólio para cash;
  * Isso é desejável do ponto de vista de preservação, mas pode gerar períodos longos de inatividade.
* Esse comportamento está alinhado com o objetivo de **preservar capital em regimes extremos**.

4. **Risco de Momentum Crash**

* Em certas condições (forte reversão em ativos líderes), pode haver:

  * “Momentum crash” – quedas rápidas em ativos que estavam liderando o ranking;
* Embora o filtro TSM ajude, não elimina totalmente esse risco.
* Mitigações adicionais:

  * Limites de DD e circuit breakers locais por família;
  * Componentes complementares de portfólio (por exemplo, TSM mais defensivo, Value/Quality), para reduzir correlação.

---

### 7.8. Estratégias de Diversificação Interna

Por ser naturalmente um **portfólio de múltiplos ativos**, Dual Momentum já traz diversificação embutida. Ainda assim, algumas estratégias internas podem melhorar a qualidade dessa diversificação:

1. **Diversificação por Setor/Cluster (quando aplicável)**

* Em universos com clusters claros (ex.: diferentes setores, tipos de protocolo em cripto), é possível:

  * Impor limites de exposição por cluster (por ex., no máximo X% em L1s, X% em DeFi);
  * Evitar portfólio 100% concentrado em um único “tema”.

2. **Combinação com TSM Puro**

* No nível de família/portfólio:

  * Dual Momentum pode ser combinado com instâncias TSM single-asset (por ex., BTCUSDT TSM) para:

    * Garantir que grandes tendências em um ativo-chave não sejam perdidas;
    * Ao mesmo tempo, distribuir parte do capital em oportunidades relativas.

3. **Diversificação entre Parametrizações**

* Manter múltiplas instâncias de Dual Momentum com:

  * Janelas de TS/CS diferentes (curta vs longa);
  * Rebalance semanal vs mensal;
  * Regras de seleção distintas (top N vs top X%).
* O portfólio da família então é a combinação ponderada dessas instâncias, reduzindo a dependência de uma única escolha paramétrica.

4. **Uso de Cash como Amortecedor**

* Ao invés de forçar 100% do capital da família em ativos a cada rebalance:

  * Permitir explicitamente que uma fração relevante permaneça em cash, quando:

    * número de ativos elegíveis for pequeno;
    * scores forem muito próximos (baixa convicção relativa);
    * ou quando o módulo de risco global assim determinar (por motivos sistêmicos).

---

Com isso, a **Família 2 – Dual Momentum** se posiciona no framework como:

* um **degrau natural** após TSM puro,
* uma forma de **explorar diferenças relativas** de desempenho entre ativos,
* mantendo a filosofia de **spot long-only/long-flat**, foco em retorno ajustado ao risco e forte disciplina de risco e governança em todas as fases: pesquisa, backtest, paper e produção.


## 8. Família 3 – Cross-Sectional Momentum (Relativo entre Ativos)

A família **Cross-Sectional Momentum (CSM)** explora o comportamento relativo entre ativos de um mesmo universo:
em vez de perguntar “este ativo está em tendência?” (como em TSM), ela pergunta:

> “Quais ativos **se saíram melhor** e quais se saíram pior que os demais na janela recente – e se essa diferença tende a persistir?”

Na forma clássica, CSM costuma ser implementado como **long vencedores / short perdedores**, o que historicamente gera retornos médios elevados, mas também está sujeito a **drawdowns violentos** (os chamados *momentum crashes*).
No contexto deste projeto, a família é adaptada para o ambiente **spot long-only / long-flat**, o que reduz parte do risco, mas também parte da “edge” bruta.

---

### 8.1. Descrição Conceitual

O Cross-Sectional Momentum funciona essencialmente como um **ranking entre ativos**:

1. Define-se um **universo** de ativos (por ex., top 20–50 criptos por liquidez);
2. Calcula-se, para cada ativo, o **retorno passado** em uma ou mais janelas (por ex., 3 a 12 meses ou N candles);
3. Ordenam-se os ativos do melhor para o pior retorno;
4. Compram-se os **“vencedores”** (top 10–20%), ignorando ou, na forma clássica, vendendo os **“perdedores”** (bottom 10–20%).

Na nossa adaptação:

* A estratégia permanece **long-only**, ou seja:

  * Long nos vencedores;
  * Nada (cash/stable) nos demais;
* Não há venda a descoberto; qualquer “lado short” é representado por **não alocar capital**.

Enquanto TSM olha se o próprio ativo está “andando para cima” ou “para baixo” ao longo do tempo, CSM olha **quem está se saindo melhor dentro do grupo**, independentemente da direção absoluta do mercado (embora, na prática, isso também importe).

---

### 8.2. Evidência Histórica e Características

A literatura empírica mostra que estratégias de momentum relativo:

* Geram **retornos médios elevados** em vários mercados (ações, FX, às vezes commodities), quando construídas como *long vencedores / short perdedores*;
* Apresentam, porém, episódios de **crashes de momentum**:

  * Fases em que os “vencedores recentes” sofrem reversões brutais;
  * Em ações, isso é particularmente visível após períodos de forte stress ou mudanças bruscas de regime.

Em versão **long-only**, usada neste projeto:

* Parte da força do fator momentum é perdida (já que não se captura o lado short dos perdedores);
* Em compensação:

  * O risco extremo de certas configurações de momentum long/short é mitigado;
  * O perfil fica mais próximo de um **filtro de seleção relativa** dentro de um universo de ativos, adequado para portfólios spot.

Por essas características, dentro do framework:

* CSM é considerado **mais agressivo e frágil** que TSM e Dual Momentum;
* É priorizado depois dessas famílias, com alocação de capital comedida e forte disciplina de risco.

---

### 8.3. Adaptação para Spot Long-Only

A adaptação de CSM ao modo **spot long/flat** segue as diretrizes gerais do projeto:

1. **Apenas lado long**

   * Não há venda a descoberto nem uso de derivativos para replicar short;
   * Perdedores simplesmente **não entram na carteira**.

2. **Cash/Stable como contraparte neutra**

   * Parte do capital não alocado nos vencedores permanece em:

     * cash / stablecoins (ex.: USDT);
   * Isso funciona como “ativo neutro” em relação ao qual os vencedores são comparados.

3. **Integração com o Módulo de Risco**

   * A estratégia não define tamanhos absolutos de ordem, apenas **pesos alvo por ativo** (por ex., 5% em cada vencedor);
   * O módulo de risco:

     * Impõe limites por ativo, família, exchange;
     * Ajusta ou bloqueia alocações se limites forem violados.

4. **Possibilidade de Combinação com Filtro TSM (opcional)**

   * Uma variante importante (mais conservadora) combina CSM com TSM:

     * Apenas ativos com TSM positivo entram no ranking cross-sectional;
     * Isso aproxima a família do Dual Momentum, mas com maior ênfase no ranking relativo.

---

### 8.4. Universo de Ativos

CSM só faz sentido em um universo **mínimo e consistente de ativos**, com características comparáveis.

No contexto inicial de cripto spot:

* Universo típico:

  * Conjunto de ativos com volume médio e liquidez mínimos;
  * Em geral, top N pares em USDT (BTC, ETH, SOL, BNB, e outros com liquidez robusta).

Critérios de inclusão:

* Estar em estado `TRADING` na exchange (segundo schema discovery);
* Possuir histórico de preços suficiente para as janelas de lookback;
* Respeitar filtros mínimos de:

  * volume médio em janela recente;
  * preço mínimo em moeda de referência (evitando ativos extremamente “penny”).

A definição desse universo é feita por:

* Configuração declarativa (lista de símbolos alvo, critérios de volume);
* Combinação com resultados de schema discovery (disponibilidade real em cada exchange).

---

### 8.5. Sinais e Regras de Seleção

O fluxo de geração de portfólio em CSM, em cada rebalance, segue aproximadamente:

1. **Cálculo do Score de Momentum Relativo**

Para cada ativo `A` no universo:

* Calcula-se um ou mais retornos em janela(s) de lookback (por exemplo):

  * `ret_3m(A)`, `ret_6m(A)`, `ret_12m(A)`;
* Combina-se esses retornos em um **score cross-sectional**:

  * Pode ser um único horizonte (ex.: 6m) ou uma combinação (ex.: média ponderada de 3m + 6m + 12m);
* O resultado é `CSM_score(A)`.

2. **Ranking**

* Ordenam-se os ativos por `CSM_score(A)` do maior para o menor;
* Em caso de empates ou scores muito próximos, regras de desempate (ex.: maior liquidez, menor volatilidade, ID determinístico) podem ser usadas.

3. **Seleção dos Vencedores**

Configuração típica:

* Seleção por:

  * `top_n` vencedores (ex.: top 5 ou top 10);
  * ou `top_percentile` (ex.: top 20% do universo);
* Apenas esses ativos recebem **peso alvo > 0**;
* Os demais ativos têm **peso alvo = 0** (não entram na carteira).

4. **Atribuição de Pesos**

Formas comuns:

* **Peso igual**: cada vencedor recebe o mesmo peso ponderado dentro do capital destinado à família (por exemplo, 1/N do orçamento da família);
* **Peso proporcional ao score**: ativos com momentum relativo maior recebem peso maior, respeitando limites por ativo;
* **Peso proporcional ao inverso da volatilidade**: impondo uma forma de “risk parity” entre vencedores.

5. **Rebalance e Turnover**

* A carteira é reavaliada em frequência definida (por ex.: semanal, quinzenal, mensal);
* Em cada rebalance:

  * Ativos que deixam de estar no top_n têm posição reduzida a zero;
  * Ativos que entram passam a ter peso alvo positivo;
  * Podem ser aplicadas regras de amortecimento (por ex.: não vender totalmente se a diferença de score for marginal) para reduzir turnover.

---

### 8.6. Parametrização

A família CSM é parametrizada em vários níveis. Os principais grupos de parâmetros são:

1. **Horizontes de Lookback**

* `lookback_windows`

  * Lista ou conjunto de janelas de retorno (ex.: `[60, 120, 252]` candles);
* `score_aggregation_method`

  * Como combinar vários horizontes (média, média ponderada, pesos maiores para horizontes intermediários etc.).

2. **Critérios de Seleção**

* `selection_mode`

  * `top_n` ou `top_percentile`;
* `top_n`

  * Número de vencedores a serem selecionados;
* `top_percentile`

  * Percentual do universo (ex.: top 20%).

3. **Esquema de Pesos**

* `weighting_scheme`

  * Igualitário;
  * Proporcional ao score;
  * Proporcional ao score ajustado pela volatilidade;
* `max_weight_per_asset`

  * Teto de peso por ativo (mitiga concentração).

4. **Filtros de Liquidez e Qualidade**

* `min_avg_volume` (por janela recente);
* `min_price` (em moeda de referência);
* `exclusion_list`

  * Lista de ativos sempre excluídos (por decisão de governança).

5. **Parâmetros de Risco Locais**

* `family_risk_budget`

  * Parcela de capital total alocável à família CSM;
* `max_positions`

  * Número máximo de posições simultâneas;
* `max_exposure_per_asset`

  * Limite local de exposição em cada ativo;
* `rebalance_frequency`

  * Periodicidade de rebalance (ex.: semanal, mensal).

Todos esses parâmetros são especificados por **instância de estratégia**, validados (type-safe) e versionados, em alinhamento com a camada de Governança (seção 17).

---

### 8.7. Riscos Específicos (Momentum Crash e Outros)

CSM, mesmo em versão long-only, carrega riscos específicos que devem ser reconhecidos e tratados:

1. **Momentum Crash**

* Episódios em que:

  * Ativos líderes (vencedores) sofrem correções repentinas e severas;
  * O portfólio CSM, concentrado nesses líderes, é duramente impactado.
* Em mercados de cripto, isso pode ocorrer após:

  * Fases de euforia em um grupo de tokens (L1, DeFi, memecoins etc.);
  * Mudanças de narrativa ou crashes sistêmicos.

Mitigações:

* Limites de perda (DD) por família e por instância;
* Circuit breakers específicos (pausar CSM quando drawdown acima de X%);
* Combinação com famílias mais defensivas (TSM, Value/Quality, etc.) no portfólio global.

2. **Crowding e Rotação Exagerada**

* Quando muitos ativos têm retorno muito parecido:

  * O ranking torna-se instável;
  * Pequenas variações de preço produzem reordenações grandes;
  * Isso aumenta **turnover** e custos.

Mitigações:

* “Bandas de inércia” no ranking:

  * Só troca um ativo por outro se a diferença de score ultrapassa um delta mínimo;
* Limite de fração da carteira que pode ser girada em um rebalance.

3. **Risco de Universos Pequenos**

* Em universos pequenos (por ex., poucos ativos elegíveis), o CSM pode:

  * Tornar-se praticamente um “TSM disfarçado” ou
  * Ficar hiperconcentrado em 1–2 ativos.

Mitigações:

* Teto por ativo (`max_exposure_per_asset`);
* Exigir tamanho mínimo de universo para ativar a estratégia (se muito pequeno, manter parte maior em cash);
* Combinar CSM com outras famílias para evitar dependência excessiva.

---

### 8.8. Papel no Portfólio e Diversificação Interna

Dentro do portfólio geral do sistema:

* CSM é tratado como uma família de **potencial de retorno elevado**, porém com:

  * maior risco de cauda;
  * maior sensibilidade a custos, slippage e regime.

Por isso:

1. **Alocação Controlada**

   * Em relação a TSM e Dual Momentum, CSM tende a ter:

     * Menor fatia do budget total de risco/capital;
     * Maior monitoramento de drawdown e turnover.

2. **Complementaridade com TSM e Dual Momentum**

   * CSM pode capturar:

     * forças relativas entre ativos que TSM não enxerga (por exemplo, quando quase todos têm TSM positivo, mas alguns se destacam mais);
   * Em portfólio conjunto:

     * TSM fornece proteção de regime (fica flat em crises);
     * Dual Momentum e CSM adicionam nuances relativas dentro do universo.

3. **Diversificação entre Instâncias CSM**

   * Manter múltiplas instâncias:

     * Com diferentes horizontes de lookback;
     * Com rebalance em frequências diferentes;
     * Com universos distintos (por liquidez, setor, faixa de market cap).
   * Isso reduz dependência de uma única parametrização e aumenta robustez de portfólio.

---

### 8.9. Roadmap de Implementação da Família 3

A implementação de CSM segue um caminho gradativo, alinhado à maturidade do sistema:

1. **Versão Piloto – CSM Simples em Universo Reduzido**

   * Universo pequeno de criptos altamente líquidas;
   * Um único horizonte de lookback (por ex., 3 ou 6 meses);
   * Pesos iguais entre vencedores;
   * Backtest com modelagem conservadora de custos e slippage.

2. **Integração com Motor de Backtest e Risco**

   * Validar:

     * Sensibilidade a custos;
     * Risco de drawdowns e “momentum crash”;
   * Ajustar limites de risco locais da família.

3. **Paper Trading com Capital Simulado**

   * Executar CSM em paper:

     * Medir turnover real em relação à liquidez;
     * Verificar impacto de ruídos intraday sobre o ranking;
     * Garantir robustez operacional (latência, integridade de dados).

4. **Live com Capital Limitado (se aprovado)**

   * Iniciar com parcela pequena do budget da família;
   * Acompanhar:

     * Consistência com o comportamento de backtest/paper;
     * Custos efetivos (fees + slippage) vs modelados.

5. **Versões Avançadas**

   * Introduzir:

     * Vários horizontes de lookback combinados;
     * Esquemas de peso mais elaborados (por score, por risco);
     * Filtros adicionais (por volatilidade, cluster, regime de mercado);
   * Estudar combinação formal com TSM (ex.: só ranquear ativos com TSM positivo).

6. **Integração com Multi-Exchange (futuro)**

   * Quando o sistema suportar mais exchanges:

     * Possibilidade de CSM por exchange;
     * Ou CSM unificado sobre ativos de múltiplas exchanges, respeitando risco e limites de custódia.

---

Dessa forma, a **Família 3 – Cross-Sectional Momentum** entra no framework como um componente:

* de **maior potencial de retorno**,
* mas também de **maior fragilidade e volatilidade**,

o que justifica sua posição **abaixo de TSM e Dual Momentum na ordem de prioridade**, sempre integrada a um módulo de risco robusto e a um processo de governança rigoroso de parametrização, monitoramento e eventual desativação.


## 9. Família 4 – Carry / Yield / Basis

A família **Carry / Yield / Basis** explora retornos que não dependem primariamente de “acertar a direção” do preço, mas de:

* **diferenças estruturais de juros / rendimentos** (carry),
* **pagamentos recorrentes** (yield),
* **deslocamentos entre mercados relacionados** (basis, funding).

Em termos simples, são estratégias que tentam responder:

> “Onde eu sou remunerado por simplesmente *segurar* um ativo ou uma posição estrutural, assumindo determinados riscos de cauda, liquidez e contraparte?”

No framework, essa família é tratada como **complementar** às famílias puramente direcionais (TSM, Dual Momentum, CSM), com papel relevante, mas **prioridade operacional menor** no início, sobretudo porque:

* Muitas das oportunidades de basis/funding envolvem **derivativos** (futuros, perpétuos);
* Há riscos específicos de **crise / crash** em carry (ganha devagar, pode perder rápido);
* Para manter o foco em **spot long/flat** no MVP, parte dessa família entra primeiro via **yield “puro” em spot** (staking, juros em stablecoins, etc.), e só depois via basis estruturado.

---

### 9.1. Descrição Conceitual

Em macro, a família engloba três pilares:

1. **Carry (Juro / Rendimento Relativo)**

   * Em FX: comprar moedas de **juros altos** e vender de **juros baixos** (carry trade).
   * Em renda fixa: carregar títulos com cupom acima do custo de financiamento.
   * Em cripto: carregar posições onde há um “juro implícito” (staking, rewards, juros de lending, etc.).

2. **Yield (Rendimentos Recorrentes em Spot)**

   * Rendimento explícito por **segurar** determinado ativo:

     * staking on-chain;
     * juros em plataformas de lending centralizadas/descentralizadas;
     * dividendos (no caso de ações, em expansões futuras);
   * Tipicamente, um retorno relativamente estável *enquanto o regime se mantém*, mas com risco de:

     * desvalorização do principal;
     * falhas de protocolo/plataforma.

3. **Basis / Funding (Cripto e Derivativos)**

   * Diferença entre:

     * preço do ativo no mercado à vista (spot);
     * e preço no mercado de futuros/perpétuos (basis/funding).
   * Estratégias clássicas:

     * **Long spot + short perp/futuro** quando há funding positivo ou basis elevada, capturando o spread ao longo do tempo;
     * Em ambientes específicos, também o inverso, mas menos comum e mais arriscado.

No projeto, toda exploração de basis/funding é considerada **módulo avançado e opcional**, pois:

* exige integração com **mercados de derivativos**;
* aumenta o risco operacional (alavancagem, liquidação, risco de exchange).

---

### 9.2. Evidência Histórica e Características

Famílias de carry e basis são amplamente documentadas:

* Em FX, o **carry trade** (long high-yield, short low-yield) é um fenômeno clássico:

  * tende a gerar retornos positivos em períodos de estabilidade;
  * sofre perdas abruptas em episódios de *flight to quality*/crises financeiras.

* Em cripto:

  * O Funding/Basis positivo prolongado em bull markets permitiu, historicamente, estratégias de:

    * comprar spot e vender perp/futuro, capturando funding periódico ou basis ao longo do tempo;
  * Essas estratégias exibem perfil típico de:

    * “ganha devagar, perde rápido” se ocorrer desalinhamento estrutural, squeezes, falhas de exchange ou de gestão de margem.

* Yield em staking/lending:

  * Quando bem selecionado, pode gerar renda relativamente estável;
  * Mas carrega riscos não triviais:

    * smart contract;
    * risco de plataforma / custódia;
    * depeg de stablecoins;
    * risco regulatório.

Dentro do framework, essa família é vista como:

* Potencialmente **muito lucrativa**, especialmente em certos regimes (bull markets prolongados, funding consistentemente positivo etc.);
* Mas com **riscos de cauda** e de infraestrutura que exigem:

  * módulos de risco específicos;
  * governança mais rígida;
  * e, por isso, **entrada tardia** no roadmap em relação às famílias de momentum.

---

### 9.3. Adaptação ao Contexto Spot Long/Flat

Dado que o projeto é orientado a **spot long/flat** em um primeiro momento, a adaptação dessa família segue alguns eixos:

1. **Foco inicial em Yield “spot-based”**

   * Priorizar estratégias que não exijam alavancagem ou derivativos, tais como:

     * selecionar ativos com yield explícito (staking líquido, programas de juros em stablecoins, etc.);
     * compor uma “camada de renda” do portfólio, com alocação limitada e fortemente controlada por risco.

2. **Basis / Funding como módulo avançado e opt-in**

   * Estratégias de:

     * long spot / short perp;
     * ou aproveitamento de basis em futuros;
   * Só entram quando:

     * houver integração segura com derivativos;
     * o módulo de risco estiver maduro para lidar com:

       * alavancagem implícita;
       * risco de liquidação;
       * risco de exchange multiplicado (spot + derivativos).

3. **Sem uso de alavancagem agressiva no núcleo**

   * Mesmo quando derivativos forem integrados:

     * as estratégias de basis devem ser **dimensionadas** para não depender de alavancagem alta para serem viáveis;
     * alavancagem, se usada, deve ser parte explícita do módulo de risco e governança, nunca “escondida” na estratégia.

4. **Carry em camadas superiores**

   * Em fases futuras e com eventuais integrações a outros mercados (FX, renda fixa, ações):

     * a lógica de carry FX (juros de moedas) e de prêmios de crédito pode ser adicionada;
     * sempre mantendo a separação clara entre:

       * estratégia de carry;
       * e mecanismos de funding/alavancagem.

---

### 9.4. Fontes de Carry/ Yield Consideradas no Projeto

No escopo cripto/spot (e extensível a outras classes), consideramos pelo menos quatro tipos de fonte de retorno para esta família:

1. **Yield em Stablecoins e Ativos “Quase Cash”**

   * Plataformas que remuneram:

     * saldos em USDT/USDC/BUSD etc. (centralizadas ou DeFi);
   * Exemplo de raciocínio:

     * parte do capital em cash/stable pode ser alocado em instrumentos de baixo risco relativo (ainda que com risco de contraparte/depeg);
   * Sempre com:

     * limites de exposição por protocolo/ativo;
     * avaliação de risco de plataforma.

2. **Staking e Participação em Protocolos (PoS, Liquidity Pools etc.)**

   * Ativos que pagam:

     * recompensas de staking;
     * yield por prover liquidez.
   * Nesse contexto:

     * o retorno vem tanto do yield quanto da eventual valorização do ativo;
     * mas há altas doses de risco de preço, risco técnico e de governança do protocolo.

3. **Basis / Funding em Cripto (Módulo Avançado)**

   * Estratégias do tipo:

     * compra de spot + short em perp/futuro quando funding é positivo ou basis está alta;
     * o retorno é o “carry” da diferença de preços/funding ao longo do tempo.
   * Requer:

     * integração com mercados de derivativos;
     * margens alocadas na exchange;
     * regras de risco muito claras (nunca deixar posição próxima da liquidação, circuit breakers etc.).

4. **Yield em Ações (Dividendos) – Futuro Multi-Classe**

   * Em uma eventual expansão para equities:

     * seleção de ações com histórico de dividendos, buybacks, etc.;
     * integração com a família Value/Quality.
   * Embora não seja foco imediato, a estrutura de carry/yield pode ser reutilizada para essa finalidade.

Cada uma dessas fontes é tratada como **sub-bloco** dentro da família, com perfis de risco, parametrização e governança próprios.

---

### 9.5. Sinais e Construção de Portfólio

Ao contrário de estratégias puramente direcionais, aqui os **sinais** refletem, em geral:

* a *atratividade de carry/yield* de uma posição;
* ponderada pelo risco de preço, contraparte, liquidez e regime.

De forma simplificada, os sinais podem ser construídos como:

1. **Score de Yield Normalizado por Risco**

Para cada oportunidade `O` (por exemplo, stablecoin X em plataforma Y, staking de ativo Z, basis em par A/B):

* calcular:

  * `yield_bruto` (APR/APY nominal, funding médio, basis implícito etc.);
  * `risco_relativo`, que pode combinar:

    * volatilidade do ativo subjacente;
    * histórico de estabilidade do protocolo/plataforma;
    * risco de depeg (para stablecoins);
    * risco regulatório/operacional (classificado qualitativamente ou quantitativamente);
  * derivar um `yield_ajustado_ao_risco` = função(`yield_bruto`, `risco_relativo`).

2. **Ranking e Seleção**

* As oportunidades são ranqueadas por `yield_ajustado_ao_risco`;
* Seleciona-se:

  * top N;
  * ou aquelas acima de um limiar mínimo;
* Aplica-se, então, limites de:

  * exposição máxima por protocolo, ativo, exchange;
  * diversificação mínima (não concentrar tudo em uma única fonte de yield).

3. **Posição Alvo**

* Em vez de decidir “comprar/vender” diretamente, a família produz:

  * **percentuais alvo** de alocação em cada oportunidade;
* O módulo de risco/portfólio:

  * converte isso em tamanhos efetivos;
  * integra com as demais famílias (TSM, Dual Momentum etc.), de acordo com o budget global.

4. **Sinais de Desligamento**

* Para carry/yield, sinais de *saída* são tão importantes quanto os de entrada:

  * queda abrupta de yield (funding colapsa, rewards diminuem);
  * aumento dos riscos (notícias adversas sobre protocolo/plataforma, depeg, mudanças regulatórias);
  * deterioração de liquidez.

Nesses casos, a estratégia deve ser capaz de:

* enviar rapidamente posição alvo = 0;
* acionar, se necessário, alertas e circuit breakers específicos.

---

### 9.6. Parametrização

Os principais blocos de parâmetros para a família Carry / Yield / Basis incluem:

1. **Fontes de Yield Ativadas**

* Flags para:

  * `enable_stable_yield`;
  * `enable_staking`;
  * `enable_basis_perp`;
  * `enable_lending_defi`, etc.

No MVP, pode-se ativar apenas uma ou duas fontes mais seguras/operacionais.

2. **Alocação de Capital**

* `family_risk_budget`

  * parcela do portfólio global destinada a carry/yield;
* `max_exposure_per_source`

  * limite por plataforma/protocolo/exchange;
* `max_exposure_per_asset`

  * limite por ativo individual.

3. **Parâmetros de Seleção**

* `min_yield_bruto`

  * yield mínimo para considerar uma oportunidade;
* `min_yield_ajustado`

  * yield ajustado ao risco mínimo (após penalizações por risco).

4. **Parâmetros de Risco Específicos**

* `max_leverage` (para módulos de basis, se e quando existirem);
* `liquidation_buffer`

  * distância mínima até o preço de liquidação (em %), abaixo da qual reduzir automaticamente a posição;
* `dd_limit_per_carry_module`

  * drawdown máximo permitido por sub-bloco (ex.: basis, staking), antes de desativar automaticamente.

5. **Janela de Observação de Yield e Risco**

* `lookback_yield`

  * janela para cálculo de yield médio (especialmente importante em funding volátil);
* `lookback_risk`

  * janela para estimar volatilidade/risco relativo.

Todos os parâmetros devem ser definidos em configuração, validados e versionados, conforme governança do sistema.

---

### 9.7. Riscos Específicos e Guardrails

A família Carry / Yield / Basis possui um conjunto de riscos **qualitativamente diferentes** das outras:

1. **Risco de Cauda / Crashes de Carry**

* “Ganha devagar, perde rápido”:

  * carry funciona bem em períodos estáveis;
  * pode sofrer perdas grandes em crises de liquidez, depeg de stable, falhas de exchange/protocolo ou squeezes de basis.
* Guardrails:

  * orçamento de risco separado para essa família;
  * limites rígidos de DD local;
  * desativação automática (circuit breakers) em cenários de stress.

2. **Risco de Contraparte e Plataforma**

* Em yield centralizado (CEX, plataformas de lending), há risco direto:

  * da empresa (falência, fraude);
  * regulatório (bloqueio de saques, congelamento de contas).
* Em DeFi, há risco:

  * de smart contract;
  * de governança (mudança de regras, hacks, exploits).

Guardrails:

* Lista de plataformas e protocolos **explicitamente permitidos** (whitelist);
* Limite de exposição por plataforma e por tipo de produto;
* Acompanhamento de notícias/eventos críticos (mesmo que parcialmente manual, integrado ao fluxo de governança).

3. **Risco de Depeg (Stablecoins)**

* Stablecoins podem perder paridade com a moeda de referência (USD, por exemplo);
* Isso é especialmente grave em estratégias de yield em stable.

Mitigações:

* Diversificação entre diferentes stablecoins;
* Limites de exposição por stable;
* Monitoramento de preços vs paridade (alertas se depeg > X%).

4. **Risco de Liquidação (para Basis com Derivativos)**

* Em estratégias com short perp/futuro:

  * variações adversas de preço podem aproximar a posição de limites de liquidação;
* Mesmo com hedge teórico (long spot), a mecânica de margem gera risco real.

Guardrails:

* Definir explicitamente que o sistema **não opera próximo de margem mínima**;
* Regras automáticas de redução de posição se a margem disponível ficar abaixo de determinado buffer;
* Circuit breaker específico para esse módulo.

---

### 9.8. Papel no Portfólio e Roadmap de Implementação

No portfólio global, a família Carry / Yield / Basis é vista como:

* um **complemento de retorno** que pode:

  * melhorar o Sharpe do portfólio;
  * gerar fluxo de caixa (yield) em períodos de baixa volatilidade;
* mas que **não deve ser o núcleo principal** devido aos riscos de cauda e à dependência de infraestrutura/protocolos.

**Roadmap sugerido:**

1. **Fase 1 – Yield “Seguro/Simples” em Spot**

   * Implementar um módulo de seleção de oportunidades de yield em:

     * stablecoins com perfil de risco mais baixo;
     * staking de ativos “blue chip”, se aplicável.
   * Alocação modesta de capital, forte disciplina de risco.

2. **Fase 2 – Integração com Módulo de Risco e Portfólio Global**

   * Avaliar desempenho:

     * como componente de “renda”;
     * sua correlação com outras famílias (TSM, Dual Momentum etc.);
   * Ajustar limites de budget e critérios de seleção.

3. **Fase 3 – Basis / Funding Simples (opcional, já com derivativos)**

   * Introduzir módulos específicos para:

     * long spot + short perp em situações de funding robustamente positivo;
   * Com:

     * alavancagem restrita;
     * mecanismos fortes de monitoramento e proteção contra liquidação.

4. **Fase 4 – Expansão Multi-Classe (FX, Ações, Bonds) – Futuro**

   * Quando e se o sistema incorporar outros mercados:

     * adicionar carry em FX (juros de moedas);
     * estratégias de curva de juros e prêmios de crédito;
     * yield em ações (dividendos) integradas com a família Value/Quality.

---

Com isso, a **Família 4 – Carry / Yield / Basis** se posiciona no framework como uma camada de:

* **retorno estrutural**,
* **exposta a riscos de cauda e contraparte**,

que exige **governança e gestão de risco particularmente rigorosas**, motivo pelo qual:

* entra no roadmap **após** as famílias de momentum (TSM, Dual, Cross-Sectional);
* opera inicialmente com **escopo reduzido e bem controlado** (yield spot),
* e só depois se expande para basis/derivativos, à medida que o restante da arquitetura (risco, execução, observabilidade e segurança) tiver maturidade adequada.


## 10. Família 5 – Value + Quality (Principalmente Ações Spot)

A família **Value + Quality** é pensada, neste framework, como um **pilar de médio/longo prazo**, orientado a:

* capturar prêmios de **valor** (ativos “baratos” em relação a lucros, patrimônio, fluxo de caixa);
* filtrá-los por critérios de **qualidade** (rentabilidade, alavancagem, estabilidade, governança);
* operar, majoritariamente, em **ações spot long-only**, com baixa rotatividade e horizonte de investimento mais longo.

Ela se encaixa menos naturalmente no universo cripto puro – onde conceitos de “valor intrínseco” e “qualidade contábil” são muito mais nebulosos – e ganha relevância à medida que o sistema evolui para:

* integrar **ações spot** (ETFs, ações individuais, eventualmente BDRs ou equivalentes tokenizados);
* trabalhar com dados fundamentais (fundamentals) e fatores clássicos.

Por isso, no roadmap, esta família tem **prioridade abaixo** das famílias de momentum (TSM, Dual, CSM) e da família Carry/Yield em cripto, mas é importante para uma visão de portfólio mais ampla e interclasses.

---

### 10.1. Descrição Conceitual

A combinação **Value + Quality** é inspirada diretamente na literatura de **fatores** em ações:

* **Value**: comprar ativos com preço “barato” em relação a métricas fundamentais, tais como:

  * P/L (price/earnings),
  * P/B (price/book),
  * EV/EBITDA,
  * dividend yield,
  * entre outros.

* **Quality**: entre os ativos “baratos”, priorizar os que apresentam:

  * alta rentabilidade (ROE, ROA, margens saudáveis);
  * baixa alavancagem (dívida controlada);
  * estabilidade de lucros e fluxos de caixa;
  * menor probabilidade de problemas contábeis ou de governança.

Em termos práticos:

> A estratégia busca “bons negócios a bons preços”,
> evitando a clássica armadilha de comprar apenas “coisas baratas” que são baratas por motivos estruturais (value traps).

No contexto do projeto:

* a família é **long-only**, naturalmente compatível com spot;
* opera em **frequência baixa** (rebalance trimestral, semestral ou anual, tipicamente);
* serve como **âncora estável** de longo prazo em um portfólio que também terá componentes de alta rotatividade (momentum, mean reversion, etc.).

---

### 10.2. Evidência Histórica e Características

A literatura de finanças quantitativas mostra que:

* Estratégias de **Value** tendem a apresentar:

  * retornos médios acima do mercado em horizontes de anos;
  * maior volatilidade em determinados períodos (especialmente em bull markets de “growth”).

* Estratégias de **Quality**:

  * focam em empresas lucrativas, com balanços sólidos;
  * frequentemente exibem retornos ajustados ao risco superiores e drawdowns um pouco menores.

A combinação **Value + Quality**:

* tenta capturar o prêmio de valor,
* mas filtrando “lixos baratos” (empresas de baixa qualidade),
* resultando em carteiras que:

  * muitas vezes têm **volatilidade menor** que o mercado;
  * podem enfrentar **longos períodos de underperformance** relativa (anos em que growth domina o estilo value).

Essas características tornam a família:

* menos adequada como **primeiro foco** para trading cripto (por falta de fundamentos consolidados),
* porém extremamente útil quando o framework passar a operar ações spot e ETFs com dados fundamentais disponíveis.

---

### 10.3. Adaptação ao Contexto Spot Long-Only

A família Value + Quality encaixa-se naturalmente no modo **spot long-only**:

1. **Apenas lado comprado (long)**

   * Não há short estruturado;
   * A estratégia compra ações consideradas:

     * baratas de acordo com critérios de valuation;
     * de boa qualidade de acordo com os filtros.

2. **Horizonte de Investimento Longo**

   * Rebalanceamentos pouco frequentes:

     * trimestrais, semestrais ou anuais;
   * Evita transformar uma estratégia concebida para horizontes de anos em algo “tático de curto prazo”, o que tende a quebrar a tese original.

3. **Uso de Cash/ETFs de Mercado como Alternativa Neutra**

   * Quando:

     * o universo de ações elegíveis está pouco atrativo;
     * ou há incerteza elevada (ex.: crises sistêmicas);
   * é possível:

     * manter parte do capital em cash;
     * ou, em fases avançadas, em ETFs amplos de mercado como componente neutro.

4. **Integração com Risco e Portfólio Global**

   * A família não opera isoladamente:

     * recebe um **budget de risco/capital** definido pelo módulo de alocação;
     * compartilha exposição com outras famílias (momentum, carry etc.);
   * No portfólio final, Value + Quality tende a:

     * contribuir com estabilidade e prêmios de longo prazo;
     * atuar como contrapeso aos estilos mais “quentes” (momentum, cripto trend-following etc.).

---

### 10.4. Universo de Ativos

A família Value + Quality exige **dados fundamentais** e, portanto, é naturalmente orientada a:

* **ações spot** (mercados tradicionais);
* **ETFs** (como proxies de fatores ou setores);
* eventuais equivalentes tokenizados com métricas econômicas claras.

No desenho do projeto:

1. **Fase inicial (cripto-only)**

   * A família é mais **conceitual e preparatória**:

     * pode não estar ativa ou operar apenas com proxies muito limitados;
     * por exemplo, selecionar “blue chips” cripto por métricas simplificadas (market cap, seniority do protocolo, liquidez, estabilidade), mas isso é uma aproximação fraca de Value/Quality clássico.

2. **Fase de integração com ações (multi-classe)**
   Quando houver conexão com:

   * bolsa(s) de ações,
   * provedores de dados fundamentais (APIs de fundamentals, arquivos de balanço, etc.),

   o universo típico incluirá:

   * Ações de índices amplos (S&P 500, MSCI, índices locais relevantes);
   * Ações mid/large cap com liquidez mínima;
   * ETFs setoriais ou fatoriais (Value, Quality, Dividend etc.).

3. **Critérios Mínimos de Inclusão**

   * Histórico de dados fundamentalistas consistente (2–5 anos, no mínimo);
   * Liquidez mínima (volume médio, free float);
   * Ausência de crises contábeis graves recentes (quando identificáveis).

---

### 10.5. Sinais e Regras de Seleção

Os sinais em Value + Quality são derivados principalmente de **fatores fundamentais**:

1. **Métricas de Valor (Value)**

   Exemplos de indicadores:

   * P/L (preço / lucro);
   * P/B (preço / valor patrimonial);
   * EV/EBITDA;
   * dividend yield;
   * price-to-cash-flow.

   Essas métricas podem ser:

   * normalizadas por setor/indústria;
   * transformadas em **scores** (por exemplo, z-scores em relação à média do setor).

2. **Métricas de Qualidade (Quality)**

   Exemplos:

   * ROE (return on equity);
   * ROA (return on assets);
   * Margem líquida, margem operacional;
   * Estabilidade de lucros (variância de EPS);
   * Níveis de dívida (Dívida/EBITDA, Dívida/Patrimônio).

   Também convertidos em scores e, muitas vezes, combinados em um score de **qualidade composto**.

3. **Combinação Value + Quality**

   * Passo 1: filtrar o universo por **qualidade mínima**:

     * excluir empresas com qualidade muito baixa (baixa rentabilidade, alta alavancagem, instabilidade extrema de lucros);
   * Passo 2: dentro desse subconjunto “ok em qualidade”:

     * ranquear por **barateza** (value);
   * Passo 3: opcionalmente:

     * combinar os dois scores (value e quality) em um score único;
     * ou aplicar pesos relativos (por ex., 60% peso value, 40% quality), conforme tese adotada.

4. **Seleção de Portfólio**

   * Seleção por:

     * `top_n` ativos com maior score combinado;
     * ou `top_percentile` (ex.: top 10–20% do universo filtrado).
   * Posições são definidas em pesos alvo, respeitando:

     * limites de exposição por ativo;
     * diversificação setorial/geográfica, se aplicável.

5. **Frequência de Rebalance**

   * Tipicamente baixa:

     * trimestral, semestral ou anual;
   * Em cada rebalance:

     * ativos que deixaram de ser atrativos (perderam qualidade, ficaram caros) podem ser vendidos;
     * ativos que passaram a ser atrativos entram ou aumentam peso.

---

### 10.6. Parametrização

Os principais grupos de parâmetros da família Value + Quality incluem:

1. **Definição de Métricas e Scores**

   * `value_metrics`

     * lista de métricas de valor usadas (P/L, P/B, EV/EBITDA etc.);
   * `quality_metrics`

     * lista de métricas de qualidade;
   * `value_score_method` / `quality_score_method`

     * forma de combinar múltiplas métricas em um único score (média, média ponderada, ranks etc.);
   * `score_combination_method`

     * como combinar value e quality (ex.: valor_*a_ + qualidade_*b_).

2. **Filtros de Universo**

   * `min_liquidity` (volume médio);
   * `min_market_cap` (capitalização mínima);
   * `min_quality_threshold`

     * pontuação mínima de qualidade para entrar no universo elegível.

3. **Seleção de Ativos**

   * `selection_mode`

     * `top_n` ou `top_percentile`;
   * `top_n` / `top_percentile`

     * parâmetros concretos (ex.: top 30 ações ou top 15%);
   * `max_weight_per_asset`

     * limite por ativo individual;
   * `sector_constraints` (quando aplicável)

     * limites de exposição por setor/indústria;
   * `country_constraints` (em portfólios globais)

     * limites por país/região.

4. **Rebalance e Risco Local**

   * `rebalance_frequency`

     * trimestral, semestral, anual;
   * `family_risk_budget`

     * parcela de capital total destinada à família;
   * `dd_limit_family`

     * drawdown máximo tolerável na família antes de acionar circuit breaker local.

Todos esses parâmetros são documentados e versionados, para viabilizar backtests, paper trading e transição segura para live.

---

### 10.7. Riscos Específicos

A família Value + Quality possui riscos bem conhecidos:

1. **Períodos Longos de Underperformance**

   * Estratégias de value (e mesmo value + quality) podem:

     * ficar anos “atrás do mercado” em fases dominadas por growth ou por outras dinâmicas de fluxo;
   * Isso pode gerar:

     * pressão psicológica;
     * tentação de abandonar a tese perto do “ponto de virada”.

   No framework:

   * Esse risco é aceito como parte da natureza da família;
   * Mitigado pela diversificação com outras famílias (momentum, carry etc.);
   * E monitorado por métricas de performance de médio/longo prazo.

2. **Value Traps (Armadi­lhas de Valor)**

   * Empresas baratas que:

     * são baratas por problemas estruturais e nunca “reprecificam” para cima;
   * A combinação com quality ajuda a evitar parte dessas armadilhas, mas não todas.

3. **Risco de Mudanças Estruturais**

   * Mudanças na economia, tecnologia ou regulação podem:

     * reduzir a relevância de algumas métricas (ex.: determinados setores em declínio estrutural);
   * O framework precisa permitir:

     * revisão periódica das métricas e da própria tese da família.

4. **Risco de Dados Fundamentais**

   * Dependência de:

     * qualidade das demonstrações financeiras;
     * atualidade das bases de dados (atrasos, revisões);
     * eventuais fraudes ou distorções contábeis.
   * Mitigações:

     * uso de fornecedores de dados confiáveis;
     * filtros adicionais de consistência (ex.: excluir empresas com histórico recente de problemas contábeis evidentes).

---

### 10.8. Papel no Portfólio e Roadmap de Implementação

No portfólio global, a família Value + Quality é:

* uma **perna de longo prazo**,
* com menor giro e, frequentemente, com perfil de risco mais “suave” que cripto puro ou estratégias de alto turnover.

**Papel no Portfólio:**

* Servir como:

  * fonte de **prêmio de risco estrutural**;
  * componente que tende a:

    * contribuir com retornos positivos em horizontes plurianuais;
    * diversificar estilos dominados por momentum e carry.

* Mitigar:

  * exposição excessiva a um único tipo de dinâmica (ex.: apenas tendências de cripto).

**Roadmap de Implementação:**

1. **Fase Conceitual (cripto-only)**

   * Documentar a família, métricas e parâmetros;
   * Definir como ela será integrada no módulo de portfólio quando ações forem introduzidas.

2. **Fase de Dados e Infra (integração com equities)**

   * Conectar a:

     * fornecedor(es) de dados fundamentais;
     * exchanges/venues de ações spot;
   * Validar pipelines de dados (fundamentals + preços).

3. **Backtests Estruturados de Value + Quality em Ações**

   * Aplicar a metodologia a:

     * universos conhecidos (ex.: S&P 500, índices locais);
   * Analisar:

     * performance histórica, drawdown, períodos de underperformance;
     * correlação com outras famílias (TSM, Dual, CSM, Carry).

4. **Paper Trading em Ações/ETFs**

   * Testar operação real (ordens, rebalance trimestral) em paper;
   * Ajustar pesos, filtros, rebalance frequency.

5. **Live com Alocação Gradual**

   * Iniciar com parte modesta do capital do portfólio;
   * Monitorar aderência entre backtest/paper/live em horizontes mais longos.

6. **Integração Total ao Portfólio Multi-Classe**

   * Quando o sistema estiver plenamente multi-ativo:

     * tratar Value + Quality como um dos “pilares de longo prazo”;
     * ajustar orçamento de capital entre famílias, considerando regimes e objetivos.

---

Assim, a **Família 5 – Value + Quality** representa, dentro do framework:

* a transição do universo cripto puro para uma visão mais ampla de **investimento multi-classe**,
* ancorada em fatores clássicos e robustos da literatura de equities,
* com foco em **retorno ajustado ao risco em horizontes mais longos**, complementando as famílias mais rápidas e táticas (momentum, carry, mean reversion, etc.).


## 11. Família 6 – Mean Reversion de Curto Prazo (Intraday / Swing)

A família **Mean Reversion de Curto Prazo (Intraday / Swing)** busca capturar movimentos de **excesso de curto prazo** – quedas ou altas rápidas e “exageradas” – assumindo que:

> “Movimentos extremos no curto prazo tendem a ser parcialmente corrigidos, retornando a uma faixa mais ‘normal’ de preço.”

Em contraste com as famílias de **momentum** (que tentam seguir tendências), a mean reversion de curto prazo aposta em **reversões parciais** dentro de janelas pequenas: minutos, horas ou poucos dias.

No contexto do projeto, essa família é tratada como:

* **potencialmente muito lucrativa**,
* mas **mais frágil** a:

  * custos de transação e slippage;
  * mudanças de microestrutura;
  * regimes de mercado em forte tendência;
* portanto, com **prioridade abaixo** das famílias de momentum (TSM, Dual, CSM) e com alocação de capital mais comedida.

---

### 11.1. Descrição Conceitual

A lógica de **mean reversion de curto prazo** parte da observação de que:

* Em muitos mercados, especialmente em horizontes intraday/curtos,
  **movimentos extremos** (spikes de alta ou quedas abruptas) costumam ser:

  * exagerados pela liquidez momentânea, ordens grandes, stops, liquidations etc.;
  * seguidos por um movimento oposto de correção parcial.

Em termos práticos, a estratégia procura:

* **comprar “oversold de curto prazo”** (quedas fortes e rápidas, indicadores de exaustão de venda);
* **vender ou reduzir exposição em “overbought de curto prazo”** (altas fortes, exaustão de compra);

sempre dentro da lógica **spot long/flat**:

* lado long em reversão de baixa;
* redução/saída em reversões de alta;
* sem operar short “seco” no núcleo do framework.

Esse tipo de abordagem pode ser aplicado em:

* **intraday** (timeframes de 1m, 5m, 15m, 1h);
* **swing curto** (1–5 dias), com critérios de reversão em janelas curtas.

---

### 11.2. Características e Fragilidades

Estratégias de mean reversion de curto prazo têm algumas características importantes:

* Podem apresentar **alta taxa de acerto** (muitos trades pequenos vencedores);
* Porém, quando erram ou o regime muda:

  * podem sofrer **perdas grandes** em movimentos fortes e persistentes de tendência;
  * são muito sensíveis a custos (fees + slippage), especialmente em regimes intraday.

No projeto, isso se traduz em:

* Tratamento da família como **mais “tática” e menos “estrutural”**;
* Necessidade de:

  * modelagem de custos muito realista em backtest;
  * monitoramento estreito de performance live vs simulada;
  * limites de risco locais restritivos.

---

### 11.3. Adaptação ao Contexto Spot Long/Flat

Para aderir ao escopo “spot long/flat” (sem short descoberto), a família é adaptada da seguinte forma:

1. **Long em Oversold / Redução em Overbought**

* Em vez de:

  * short agressivo em overbought,
* O sistema prefere:

  * **não aumentar posição** ou **reduzir** posição existente quando o ativo está sobrecomprado;
  * focar em **entradas compradas em momentos de oversold de curto prazo**.

2. **Sem Alavancagem Exposta no Núcleo**

* Mesmo que algumas exchanges ofereçam margin trading intraday, o núcleo da família:

  * opera com **spot puro**, sem alavancagem explícita;
  * qualquer extensão com alavancagem seria considerada módulo avançado, dependente de regras extras de risco.

3. **Timeframes Compatíveis com Single Node**

* A execução é pensada para:

  * timeframes de minutos a horas (ex.: 5m, 15m, 1h),
  * que sejam compatíveis com a capacidade de processamento de um single node sem HFT:

    * não se busca microsegundos/milisegundos de latência;
    * a lógica é “rápida”, mas não de alta frequência.

4. **Integração Forte com o Módulo de Risco**

* Como a estratégia tende a gerar:

  * mais trades por unidade de tempo;
  * maior exposição a ruído,
* O módulo de risco impõe:

  * limites por dia (número de trades, perdas diárias, volume operado);
  * circuit breakers específicos para a família.

---

### 11.4. Universo de Ativos e Timeframes

A mean reversion de curto prazo funciona melhor em ativos com:

* **liquidez razoável** intraday;
* **spreads** relativamente estreitos;
* presença de ruídos e micro-movimentos que possam ser explorados.

No contexto inicial em cripto spot:

1. **Ativos Alvo**

* Pares como BTCUSDT, ETHUSDT e outras criptos de alta liquidez;
* Opcionalmente, alguns altcoins com boa liquidez, mas sempre com:

  * filtros mínimos de volume;
  * spreads acompanhados (para evitar execução ruim).

2. **Timeframes Alvo**

* Intraday: 5m, 15m, 1h;
* Swing curto: 4h, 1D (para estratégias de reversão mais “lentas”).

3. **Seleção Dinâmica via Configuração**

* Por instância de estratégia, é possível definir:

  * quais ativos e timeframes serão operados;
  * constraints de liquidez mínima e período de operação (por exemplo, não operar em horários de liquidez extremamente baixa, se houver padrão claro).

---

### 11.5. Sinais e Estrutura de Regras

A família de mean reversion de curto prazo pode usar uma combinação de:

* indicadores clássicos de sobrecompra/sobrevenda;
* bandas de volatilidade;
* microestrutura (distância ao VWAP, gaps, movimentos extremos intraday);
* padrões de candle de exaustão.

De forma genérica, os blocos principais de sinais são:

1. **Identificação de “Oversold” de Curto Prazo (Entrada Long)**

Critérios típicos:

* Preço se afasta rapidamente **para baixo** de uma média curta (ex.: EMA 20) acima de um certo múltiplo de ATR;
* Indicadores como RSI, Stochastic, CCI abaixo de limiares (ex.: RSI < 30, ou ainda mais sensível < 20 em intraday);
* Toques em bandas inferiores (Bollinger, Keltner) com confirmação de exaustão (pavio longo, volume de capitulação etc.).

2. **Identificação de “Overbought” de Curto Prazo (Saída / Redução)**

* Preço se afasta **para cima** de média, também em múltiplos de ATR;
* RSI, Stochastic, outros indicadores em níveis altos (ex.: RSI > 70/80);
* Toques em bandas superiores com sinais de fraqueza (pavio superior longo, volume decrescente etc.).

No núcleo do framework:

* o foco é em **entradas longas em oversold** e **saídas/realizações em overbought**;
* evitar se expor em full long em movimentos extremamente esticados para cima.

3. **Filtros de Tendência e Regime**

Para reduzir a probabilidade de comprar “faca caindo” em tendência forte, são utilizados filtros, por exemplo:

* Não operar mean reversion **contra** uma tendência forte definida em timeframe superior:

  * ex.: se a tendência diária está claramente de baixa, diminuir agressividade em compras contra movimento;
* Usar ADX/CCI, slope de EMAs ou outros critérios para:

  * diferenciar entre “ruído” e “tendência de verdade”.

4. **Gestão de Trade e Saídas**

Além da entrada em reversão, são usadas regras de:

* **Take Profit (TP) de curto alcance**

  * Ex.: alvo de retorno de 1–2 ATRs acima da entrada;
* **Stop Loss (SL) relativamente curto**

  * Para evitar “ficar preso” em movimentos que não revertem;
* **Tempo máximo em trade**

  * Se a reversão não ocorre em X candles, sair da posição (setup “falhou”);
* Em alguns casos, **parciais**:

  * realizar parte no primeiro alvo, mover stop para breakeven etc.

---

### 11.6. Parametrização

A família Mean Reversion de Curto Prazo é bastante sensível a parâmetros. Eles devem ser configurados com cuidado, sempre com testes robustos e evitando overfitting.

Principais grupos de parâmetros:

1. **Sinais de Oversold/Overbought**

* `rsi_oversold`, `rsi_overbought`;
* `atr_multiplicador_entrada` (quão extremo deve ser o movimento em ATR);
* `bandas_lookback` e `bandas_desvio` (para Bollinger/Keltner).

2. **Filtros de Tendência**

* `trend_tf` (timeframe de referência para tendência, ex.: 1H, 4H, 1D);
* `trend_filter_method` (slope de EMA, ADX, posição vs EMA longa etc.);
* parâmetros de tolerância para operar “contra a tendência” com size reduzido ou não operar.

3. **Gestão de Trade (Risco por Trade)**

* `rr_target` (relação risco/retorno por trade, ex.: 1:1, 2:1);
* `stop_loss_atr_mult` (atr para definir o stop);
* `take_profit_atr_mult`;
* `max_trade_duration_candles` (cancelar trade se não andar até X candles).

4. **Frequência e Horário de Operação**

* `allowed_time_windows`

  * janelas de horário em que a estratégia está ativa (pode ser relevante em mercados com ciclos intraday);
* `max_trades_per_day`

  * para limitar a hiperatividade em dias muito voláteis.

5. **Parâmetros de Risco Locais**

* `max_exposure_per_asset_mean_reversion`;
* `daily_loss_limit_family` (perda diária máxima para a família, antes de desligar);
* `max_open_trades_family` (número máximo de trades simultâneos).

Todos esses parâmetros são:

* definidos por instância de estratégia;
* validados em configuração;
* versionados para permitir reprodutibilidade e governança.

---

### 11.7. Riscos Específicos e Guardrails

A família Mean Reversion de Curto Prazo tem riscos distintos, que exigem guardrails fortes:

1. **Risco de Regime (Tendência Forte)**

* Em tendências fortes (bull ou bear), a lógica “vai voltar para média” falha repetidamente;
* Isso pode gerar **séries de pequenas perdas** (várias tentativas de reversão que não se confirmam).

Guardrails:

* Filtros de tendência rigorosos;
* Desligar ou reduzir agressividade da família em períodos de tendência muito forte;
* Circuit breakers locais: se a família perde X% em um dia/semana, suspender.

2. **Sensibilidade a Custos e Slippage**

* Como há mais trades intraday:

  * os custos podem corroer grande parte (ou toda) a edge;
* Simulações que ignoram corretamente fees e slippage tendem a ser **ilusoriamente otimistas**.

Guardrails:

* Modelagem realista de custos em backtest;
* Execução real em paper trading antes de live;
* Limitação de número de trades por dia e por ativo.

3. **Risco de Overfitting**

* Mean reversion intraday é terreno fértil para overfitting:

  * ajuste excessivo de parâmetros a um período histórico específico;
* Estratégias que parecem ótimas em backtest podem colapsar em produção.

Guardrails:

* Divisão IS/OOS, cross-validation temporal, walk-forward;
* Avaliar robustez de parâmetros (faixas de parâmetros e não pontos únicos);
* Exigir período de paper trading significativo antes de alocação real.

4. **Risco de Liquidez Local**

* Em altcoins ou mercados menos líquidos:

  * movimentos “bonitos” em gráfico não se traduzem em execuções decentes;
* O slippage pode ser enorme, destruindo a expectativa de reversão lucrativa.

Guardrails:

* Operar mean reversion apenas em ativos e pares com liquidez robusta;
* Limites de tamanho de ordem por ativo/timeframe.

---

### 11.8. Papel no Portfólio e Roadmap de Implementação

No portfólio global, a família Mean Reversion de Curto Prazo é:

* um **complemento tático** às famílias estruturais (TSM, Dual Momentum, Value/Quality etc.);
* capaz de:

  * gerar retorno em regimes de mercado **laterais** ou com volatilidade de curto prazo;
  * suavizar períodos em que momentum está sofrendo.

Porém, devido à sua fragilidade:

* deve ter **alocação de capital inicial modesta**;
* ser introduzida **após** a estabilização de TSM, Dual Momentum, CSM e dos módulos de risco/execução.

**Roadmap sugerido:**

1. **Fase 1 – Protótipo em Backtest (Single Asset / Few TFs)**

   * Implementar uma instância simples:

     * por exemplo, mean reversion em BTCUSDT 15m ou 1h;
   * Testar diferentes combinações de sinais e filtros, com custos bem modelados.

2. **Fase 2 – Validação Robusta (Backtest Avançado)**

   * Aplicar:

     * IS/OOS, walk-forward;
     * análise de sensibilidade de parâmetros;
     * simulação de regimes de tendência forte vs lateralização.

3. **Fase 3 – Paper Trading em Produção**

   * Executar em tempo real com ordens simuladas:

     * medir slippage implícito;
     * validar latência do core loop para sinais intraday;
     * ajustar limites de número de trades, janelas de operação.

4. **Fase 4 – Live com Capital Limitado**

   * Alocação pequena de capital, sob monitoramento intenso:

     * daily loss limit rigoroso;
     * circuit breakers por sequência de perdas.

5. **Fase 5 – Expansão Controlada**

   * Adicionar:

     * mais ativos (apenas os de maior liquidez);
     * variações de setup (outros timeframes, outros indicadores de reversão);
   * Avaliar:

     * correlação com demais famílias;
     * impacto real em retorno ajustado ao risco do portfólio.

---

Dessa forma, a **Família 6 – Mean Reversion de Curto Prazo (Intraday / Swing)** se insere no framework como:

* uma camada **tática e oportunista**,
* com alto potencial, porém alta sensibilidade a custos e regime,

que só deve ser explorada **após** a consolidação das famílias mais robustas, sempre acompanhada de **modelagem de risco conservadora**, processos de validação rigorosos e observabilidade reforçada.


## 12. Família 7 – Market Making / Liquidity Providing

A família **Market Making / Liquidity Providing** busca capturar o **spread** entre compra e venda e, em alguns casos, rebates ou incentivos de liquidez oferecidos pela exchange, atuando como:

> “quem sempre está disposto a comprar um pouco abaixo do mercado e vender um pouco acima, ganhando na diferença — desde que consiga controlar o risco de inventário e de cauda”.

Dentro do framework, esta família é considerada:

* **avançada**,
* altamente dependente de **microestrutura de mercado** e de **infraestrutura de execução**,
* com potencial de **Sharpe muito alto** quando bem implementada,
* mas **pouco adequada** como prioridade inicial em ambiente single node, sem colocation nem foco em latência ultra-baixa.

Por isso, ela aparece **no fim da ordem de implementação**: é planejada conceitualmente, preparada na arquitetura, mas só entra em produção quando o restante do ecossistema (dados, risco, execução, observabilidade) estiver maduro.

---

### 12.1. Descrição Conceitual

Em linhas gerais, o market maker:

1. **Coloca ordens passivas** (limit) de compra abaixo do preço atual e de venda acima do preço atual;
2. Tenta **ser executado em ambos os lados**:

   * compra mais barato;
   * vende mais caro;
   * realiza lucro no spread, mesmo que o preço não se mova muito;
3. Administra seu **inventário**:

   * evita ficar excessivamente comprado ou vendido;
   * ajusta preços de bid/ask quando está “cheio” de inventário em uma direção.

Em mercados spot sem derivativos:

* a posição líquida tende a oscilar em torno de zero;
* o objetivo não é “apostar na direção”, mas **girar inventário** em torno de um valor justo, ganhando micro-lucros repetidos.

No contexto do projeto:

* a família trabalha em **spot**, sem alavancagem explícita,
* com foco em pares líquidos (por exemplo, BTCUSDT, ETHUSDT),
* usando timeframes operacionais de **curto prazo**, mas não HFT.

---

### 12.2. Características e Desafios

Market making tem algumas características específicas:

* **Potencial de alta frequência de trades**:

  * muitos fills pequenos ao longo do dia;
* **Perfil de risco assimétrico**:

  * ganha frequentemente pequenos spreads;
  * pode perder muito em movimentos bruscos de preço (gap, rug, eventos de cauda);
* **Altíssima sensibilidade a latência e microestrutura**:

  * quanto mais rápida a reação a mudanças de preço, book e fluxos de ordens, maior a capacidade de:

    * evitar “ser atropelado”;
    * recotar preços de forma competitiva.

Em um ambiente **single node sem colocation**:

* é irrealista competir com HFT profissional estrito;
* o foco precisa ser em:

  * **estratégias de liquidez “mais lentas”**, com spreads mais largos;
  * horizontes um pouco maiores (segundos a minutos, não microssegundos).

---

### 12.3. Adaptação ao Contexto Spot, Single Node e Long/Flat

Para aderir ao escopo do projeto, a família de Market Making é adaptada de forma conservadora:

1. **Spot Only, sem alavancagem no núcleo**

   * Todas as posições são em **spot**, sem margem;
   * Evita-se risco adicional de liquidação da própria posição do MM.

2. **Inventário com Faixa Alvo (Inventory Bands)**

   * Em vez de ficar long/short “infinito”, define-se:

     * um **inventário alvo** (por exemplo, neutro);
     * e uma **faixa permitida** (por exemplo, ±X unidades ou ±Y% do capital);
   * A lógica de quotes ajusta:

     * spreads e tamanhos de ordens para voltar o inventário à faixa desejada.

3. **Spreads Mais Largos e Frequência Moderada**

   * Considerando que o sistema não é HFT:

     * os spreads cotados são deliberadamente **menos agressivos** (mais distantes do mid price);
     * a prioridade é **EV positivo após custos**, não competir pelo 1º lugar no book.

4. **Integração Forte com Risco e Circuit Breakers**

   * A família responde ao módulo de risco:

     * limites de exposição por par;
     * limites de DD diários/semanais;
     * desligamento automático em eventos extremos.

5. **Orientação Long/Flat em Termos de Risco Direcional**

   * Embora market making envolva ficar “long e short” no curto prazo, o objetivo é:

     * manter o inventário médio próximo de um alvo conservador;
     * evitar “virar aposta direcional” disfarçada.

---

### 12.4. Universo de Ativos e Requisitos de Infraestrutura

Para market making funcionar de forma minimamente eficiente, são necessários:

1. **Ativos com Boa Liquidez e Book Profundo**

   * Pairs com:

     * volume de negociação robusto;
     * spreads relativamente estreitos;
     * presença de outros market makers, para garantir fluxo.

2. **Acesso a Dados de Book em Tempo Quase Real**

   * Idealmente:

     * stream de **order book** (depth) via WebSocket;
     * no mínimo, **best bid/ask** atualizado com baixa latência.

3. **Infraestrutura Local Estável**

   * Single node com:

     * conexão de internet estável e baixa oscilação de latência;
     * capacidade de processar atualizações de book em tempo adequado para o horizonte escolhido.

4. **Integração com Exchange Spot**

   * No MVP, foco em **Binance Spot**:

     * uso de streams de book (ou, em versão mais simples, apenas last price + best bid/ask);
     * envio de ordens limit com cancelamento/amendamento frequentes (dentro de limites de rate limit).

---

### 12.5. Lógica de Cotações, Inventário e Execução

A lógica básica de Market Making pode ser decomposta em três blocos:

#### 12.5.1. Definição de Preços de Bid/Ask

Em cada ciclo (ou evento de atualização de preço/book):

1. Define-se o **mid price** de referência:

   * média de bid e ask atuais;
   * ou alguma outra função (por exemplo, VWAP curto, preço justo calculado por modelo simples).

2. Define-se um **spread alvo bruto**:

   * `spread_bruto = mid_price * spread_percentual_base`,
   * ou `spread_bruto` mínimo absoluto em ticks.

3. Ajusta-se o spread conforme inventário e volatilidade:

   * Se o inventário está muito **comprado**:

     * aproximar o ask do mid (para vender mais facilmente);
     * afastar o bid (comprar só mais barato).
   * Se o inventário está muito **vendido/baixo**:

     * aproximar o bid do mid;
     * afastar o ask.

O resultado são preços candidatos de:

* `bid_quote = mid_price - ajuste_inferior`;
* `ask_quote = mid_price + ajuste_superior`;

respeitando filtros de preço da exchange.

#### 12.5.2. Gestão de Inventário

A estratégia acompanha continuamente:

* inventário em unidades do ativo (por exemplo, BTC);
* inventário em valor notional (por exemplo, em USDT).

Regras típicas:

* Definir uma **faixa de inventário ideal**: `[inv_min, inv_max]`;

* Se o inventário se aproxima de `inv_max` (muito comprado):

  * reduzir tamanhos de ordens de compra;
  * aumentar agressividade para vender (asks mais próximos do mid);
  * eventualmente, pausar compras até voltar à zona alvo.

* Se se aproxima de `inv_min` (muito pouco ativo, “vendido” na prática):

  * fazer o inverso.

#### 12.5.3. Execução de Ordens e Rotina de Cancelamento/Recolocação

* Ordens são enviadas como **limit** com validade curta (time-in-force adequado, ex.: GTC com rotina de cancelamento periódico ou IOC, dependendo da estratégia);
* Com frequência regular (ou em reação a eventos):

  * o sistema cancela ordens antigas não executadas;
  * recalcula novos preços e reinsere ordens.

Tudo isso respeitando:

* **limites de rate limit** da exchange;
* volumetria de ordens permitida pelo módulo de risco;
* limites de exposição desejados.

---

### 12.6. Parametrização

A família de Market Making possui muitos parâmetros; para manter controle, é essencial reduzi-los a um conjunto claro e governável.

Principais grupos de parâmetros:

1. **Estrutura de Spread**

* `spread_base_percent`

  * spread percentual alvo ao redor do mid;
* `spread_min_ticks`

  * mínimo de ticks que se deseja entre mid e bid/ask;
* `spread_volatility_adj`

  * fator para aumentar spreads em alta volatilidade.

2. **Inventário**

* `inventory_target`

  * alvo (pode ser neutro, 0);
* `inventory_band_units` ou `inventory_band_pct`

  * largura da banda permitida;
* `inventory_skew_sensitivity`

  * o quanto os preços de bid/ask são deslocados quando o inventário escapa da zona ideal.

3. **Tamanho das Ordens**

* `order_size_base`

  * tamanho padrão da ordem;
* `order_size_max`

  * limite de tamanho por ordem;
* `order_size_dynamic`

  * se e como o tamanho de ordem varia com liquidez, spread ou inventário.

4. **Frequência de Atualização**

* `quote_refresh_interval`

  * intervalo mínimo entre refreshes de cotações (em segundos);
* `max_orders_per_minute`

  * limite de ordens (incluindo cancelamentos) por janela de tempo.

5. **Filtros de Mercado**

* `min_spread_market`

  * spread mínimo do próprio mercado (se o spread “natural” for muito pequeno, pode não valer a pena fazer MM);
* `min_depth`

  * profundidade mínima de book, para evitar atuar em livros “rasos” demais.

6. **Parâmetros de Risco Locais**

* `max_inventory_units` / `max_inventory_notional`

  * limite absoluto de inventário;
* `dd_limit_family`

  * drawdown máximo diário/semana para a família;
* `max_quote_notional`

  * notional máximo somado das ordens passivas abertas simultaneamente.

---

### 12.7. Riscos Específicos e Guardrails

A família Market Making traz um conjunto de riscos próprios:

1. **Risco de Cauda / Gap de Preço**

* Em eventos bruscos (notícias, liquidações em cascata):

  * ordens passivas podem ser executadas em massa contra a posição;
  * o inventário pode disparar para um lado, resultando em grandes perdas se o preço continuar se movendo na direção adversa.

Guardrails:

* `kill-switch` local:

  * se um candle/variação ultrapassar certo múltiplo de ATR ou X% em intervalo curto, o módulo:

    * cancela todas as ordens;
    * entra em modo de observação (não recota até estabilizar);
* limites de inventário rígidos e automações que nunca permitam excedê-los.

2. **Risco de Latência**

* Em ambientes com latência variável:

  * cotações podem se tornar “stale” (desatualizadas) rapidamente;
  * aumentando a probabilidade de execução ruim.

Guardrails:

* Intervalos de refresh compatíveis com a latência observada;
* Desativação automática se:

  * latência de ida e volta para a exchange exceder certo limiar por período prolongado.

3. **Risco de Rate Limit / Bloqueio de API**

* Excesso de ordens e cancelamentos pode:

  * atingir rate limits;
  * resultar em bloqueios temporários.

Guardrails:

* Controle rígido de volume de ordens;
* Monitoramento de respostas de API (códigos de erro específicos);
* Modo de degradação:

  * se aproximando do rate limit, reduzir frequência de refresh e/ou número de símbolos.

4. **Risco de Transformar MM em Aposta Direcional Involuntária**

* Sem gestão de inventário adequada:

  * o “market maker” vira “holder” sem querer.

Guardrails:

* Regras de inventory bands respeitadas pelo módulo de risco;
* Proibição de abrir novas ordens no mesmo sentido quando inventário atinge extremo permitido.

---

### 12.8. Papel no Portfólio e Roadmap de Implementação

Dentro do portfólio global, Market Making / Liquidity Providing é visto como:

* um **componente avançado**, potencialmente com **Sharpe alto**,
* mas que **não deve ser a base** do sistema em ambiente non-HFT e single node.

**Papel no Portfólio:**

* Complementar:

  * pode gerar retorno com baixa correlação com estratégias estritamente direcionais;
  * pode aproveitar períodos de mercado mais estável, com fluxo contínuo em torno de uma faixa de preços.
* Mas sempre com:

  * orçamento de risco e de capital limitado;
  * foco em não comprometer o sistema em eventos de cauda.

**Roadmap sugerido:**

1. **Fase 1 – Simulador Off-line de Book (Backtest de Microestrutura Simplificado)**

   * Implementar modelo simplificado de:

     * mid price simulado;
     * execuções contra ordens passivas;
   * Testar lógicas de spread, inventário e refresh **sem tocar na exchange real**.

2. **Fase 2 – Paper Trading com Book em Tempo Real (Sem Ordens Reais)**

   * Consumir dados de book (ou best bid/ask) da exchange;
   * Simular execuções localmente:

     * validar se a lógica de quotes faz sentido no mundo real;
     * estudar impacto de latência e volatilidade.

3. **Fase 3 – Live Experimental com Volume Muito Pequeno**

   * Enviar ordens reais de MM em 1–2 pares muito líquidos;
   * Alocar capital minúsculo, com limites de risco extremamente apertados;
   * Monitorar:

     * eficiência do spread (lucro médio por trade após fees);
     * comportamento em movimentos bruscos.

4. **Fase 4 – Otimização e Ajustes Finos**

   * Ajustar spreads, tamanhos de ordens, bandas de inventário;
   * Refletir aprendizados em parâmetros e documentação.

5. **Fase 5 – Integração Gradual no Portfólio**

   * Se a família mostrar:

     * robustez operacional;
     * performance consistente após custos;
   * Pode receber aumento **moderado** de budget de capital;
   * Sempre com monitoramento reforçado e integração plena com o módulo de risco.

---

Assim, a **Família 7 – Market Making / Liquidity Providing** é planejada como:

* um **módulo avançado**,
* de alto potencial mas de alta complexidade e risco específico,

que só deve entrar em produção quando:

* a infraestrutura de dados, risco e execução estiver madura;
* o operador tiver clareza sobre as limitações de um ambiente single node;
* e houver disciplina suficiente para tratá-la como componente **complementar**, não como pilar principal do sistema.

---

## 13. Gestão de Risco e Alocação de Capital

A gestão de risco é tratada como um **módulo central e desacoplado** das estratégias.
As famílias (TSM, Dual Momentum, Mean Reversion etc.) **não decidem quanto capital usar** nem conhecem detalhes de limites globais; elas apenas sugerem **posições alvo**. A decisão final – quanto realmente será alocado e se alguma ordem será bloqueada – pertence ao módulo de Risco e Alocação de Capital.

O objetivo é:

* Proteger o capital em diferentes níveis (trade, estratégia, família, portfólio, exchange);
* Garantir que **nenhuma família ou estratégia isolada possa “quebrar” o sistema**;
* Permitir alocação de capital alinhada à prioridade das famílias (robustez de evidência + retorno ajustado ao risco);
* Manter tudo **observável e auditável**, com trilhas claras de por que uma posição foi ou não permitida.

---

### 13.1. Camadas de Risco

O sistema organiza o risco em **camadas hierárquicas**, que se sobrepõem para criar uma malha de proteção:

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

O módulo de risco avalia todas essas camadas antes de transformar um **sinal de posição alvo** em ordens reais. Se qualquer camada for violada, a posição é reduzida ou bloqueada, e o evento é registrado para auditoria.

---

### 13.2. Limites e Guardrails

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

A alocação de capital entre famílias de estratégias segue a mesma filosofia geral do projeto: **priorizar abordagens com evidência mais robusta e melhor retorno ajustado ao risco**, ao mesmo tempo em que se busca diversificação.

Há dois modos principais:

1. **Regra Fixa (Alocação Estática)**

   * Uma fração do equity é atribuída a cada família, com base em:

     * Robustez da evidência histórica (por exemplo, TSM e Dual Momentum com pesos maiores);
     * Objetivos do portfólio (por exemplo, preferência por estratégias de tendência vs estratégias de reversão).
   * Exemplo ilustrativo:

     * 40% em **Trend Following / TSM**;
     * 25% em **Dual Momentum**;
     * 15% em **Cross-Sectional Momentum**;
     * 10% em **Mean Reversion**;
     * 10% distribuídos entre **Value/Quality** e **Carry/Yield/Basis**, conforme disponibilidade de mercado.
   * Essa regra é simples, transparente e fácil de auditar.

2. **Regra Dinâmica (Alocação Adaptativa)**

   * A alocação entre famílias varia ao longo do tempo, de acordo com:

     * Performance recente ajustada ao risco (por exemplo, Sharpe ou Calmar em janela móvel);
     * Estabilidade dos resultados (variação de drawdown);
     * Correlação entre famílias (busca de descorrelação).
   * Exemplo:

     * A cada mês, recalcular o peso ótimo aproximado entre famílias com base em um conjunto de critérios (por exemplo, score que combina Sharpe recente, drawdown e correlação).
   * É uma camada opcional, aplicada com cuidado para não gerar um sistema **hiper-reativo** (overfitting em janelas curtas).

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

Os **circuit breakers** são mecanismos automáticos de proteção que entram em ação quando o sistema detecta situações críticas. Eles operam em três níveis:

1. **Circuit Breaker Local (Estratégia / Família)**

   * Disparado quando:

     * Uma instância ultrapassa sua perda diária/mensal máxima;
     * Uma família atinge seu drawdown máximo definido;
     * É detectado comportamento anômalo (por exemplo, sequência muito longa de trades perdedores fora do padrão histórico).
   * Ação típica:

     * Desativar novas entradas para aquela instância/família;
     * Permitir apenas saídas (fechamento ordenado das posições).

2. **Circuit Breaker Global de Portfólio**

   * Disparado quando:

     * O drawdown total do sistema atinge thresholds críticos;
     * Há perda intradiária muito acima do esperado em termos estatísticos;
     * problemas graves de mercado (crashes, falhas generalizadas em exchanges).
   * Ação típica:

     * Forçar fechamento de todas as posições (quando isso é operacionalmente possível);
     * Entrar em **modo seguro** (safe mode), onde:

       * Ou nenhuma estratégia é permitida operar;
       * Ou apenas um subconjunto extremamente conservador (por exemplo, TSM diário com filtros fortes).

3. **Circuit Breaker de Infraestrutura / Exchange**

   * Disparado quando:

     * A exchange apresenta falhas recorrentes de API, erros de integridade de dados, ou desconexões prolongadas;
     * O módulo de monitoramento identifica comportamento inconsistente entre dados recebidos e dados esperados.
   * Ação típica:

     * Suspender temporariamente o envio de novas ordens para aquela exchange;
     * Bloquear qualquer automatismo que dependa de dados potencialmente corrompidos;
     * Notificar operadores para avaliação manual.

O **modo seguro** é um estado explicitamente definido do sistema, com:

* Conjunto mínimo de estratégias habilitadas (ou nenhuma);
* Restrições ainda mais rígidas de tamanho de posição;
* Aumento de log detalhado e notificações.

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

O módulo de **Backtesting, Simulação e Validação** é responsável por responder, com o máximo de rigor possível, às perguntas:

* “Se essa estratégia existisse no passado, **como ela teria se comportado**?”
* “Esse comportamento é **estatisticamente razoável** ou é apenas fruto de sorte/overfitting?”
* “O que vemos em produção está **coerente** com o que foi observado nos testes?”

Toda a arquitetura foi desenhada para que backtests, simulações e ambiente de produção compartilhem **a mesma lógica de domínio**, diferindo apenas nos adapters (dados históricos vs dados ao vivo, execução simulada vs execução real).

---

### 14.1. Estrutura de Backtest

A estrutura de backtest do sistema é organizada em torno de três elementos centrais:

1. **Fonte de Dados Históricos Padronizada**

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

O sistema adota um **engine de backtest unificado**, isto é, um motor que reutiliza:

* A mesma **Strategy API** (métodos `prepare`, `generate_signals`, `post_trade_update`);
* O mesmo **módulo de risco e alocação de capital**;
* A mesma lógica de **gestão de estado** (posições, ordens, P&L).

A única diferença é o adapter de execução:

* Em produção: `TradingPort` conectado à exchange (Binance Spot no MVP);
* Em backtest: `TradingPortSimulado`, que implementa:

  * Regras de execução baseadas nos preços dos candles;
  * Modelos de slippage e parcial fills;
  * Regras de rejeição de ordens incoerentes (ex.: preço fora da faixa de negociação daquele candle).

Benefícios dessa abordagem:

* Diminuição da chance de divergência entre “o que foi testado” e “o que está rodando”;
* Correções e melhorias em lógica de risco ou execução beneficiam tanto produção quanto backtest;
* Facilita o uso de backtesting como ferramenta de **regressão**: qualquer mudança de código pode ser validada contra uma suíte padrão de cenários históricos.

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

A validação estatística é o conjunto de processos que tenta diferenciar **resultados genuínos** de **overfitting/ruído**. Entre os mecanismos previstos estão:

1. **Divisão In-Sample / Out-of-Sample (IS/OOS)**

   * O período total de backtest é dividido em:

     * Janela de calibração (IS), usada para ajustar parâmetros;
     * Janela de validação (OOS), usada apenas para avaliar se o desempenho se mantém.
   * Parâmetros que funcionam muito bem em IS mas colapsam em OOS são considerados suspeitos.

2. **Walk-Forward Analysis**

   * Em vez de uma única divisão IS/OOS, são feitas várias janelas móveis:

     * Calibra em uma janela histórica;
     * Testa em uma janela imediatamente à frente;
     * Avança o bloco e repete o processo.
   * Isso simula o comportamento de uma estratégia que é periodicamente recalibrada, aproximando-se mais do uso real.

3. **Validação Cruzada Temporal**

   * Estruturas parecidas com cross-validation, mas respeitando a ordem temporal (sem embaralhar dados), permitindo múltiplas combinações de janelas para reduzir o risco de conclusões baseadas em um único recorte histórico.

4. **Análise de Robustez de Parâmetros**

   * Verificação de que as métricas de performance são razoavelmente estáveis em uma **vizinhança de parâmetros** (por exemplo, lookbacks próximos, thresholds ligeiramente diferentes).
   * Estratégias que dependem de um ponto específico extremamente “afinado” e que colapsam com mudanças mínimas tendem a ser classificadas como pouco robustas.

5. **Testes de Significância Simples**

   * Comparação com benchmarks simples (buy & hold, estratégias aleatórias, random entries com mesmo turnover), para avaliar se a vantagem é realmente acima do ruído.
   * Em alguns casos, podem ser aplicados testes estatísticos simples (por exemplo, comparação de distribuição de retornos) para reforçar a análise.

Os resultados dessas etapas de validação são registrados como parte da **documentação da família de estratégia**, servindo de base para decisões de aprovação ou rejeição.

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

A **execução em produção** é a materialização de tudo que foi definido em arquitetura, estratégias, risco e dados.
O objetivo deste módulo é garantir que:

* O sistema opere em **tempo real**, de forma previsível e observável;
* O mesmo **fluxo conceitual do backtest** seja respeitado (isomorfismo);
* Erros de infraestrutura (rede, API, disco) não se traduzam automaticamente em decisões de trading ruins;
* A execução de ordens seja **segura, rastreável e reversível** sempre que possível.

A seguir, detalha-se o **core loop operacional**, o comportamento no modo single exchange (Binance Spot), a evolução natural para multi-exchange, a gestão de estado, mecanismos de failover e recuperação, modos de operação e princípios de segurança na execução.

---

### 15.1. Core Loop Operacional

O **core loop** é o ciclo contínuo que, em produção, conecta:

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

Embora a produção inicial seja em Binance Spot, toda a lógica de execução é construída com **multi-exchange em mente**:

1. **Portas Genéricas para Market Data e Trading**

   * Estratégias não “sabem” que estão na Binance; elas operam sobre `MarketId` abstratos.
   * Múltiplos adapters podem coexistir:

     * `BinanceSpotAdapter`, `KrakenSpotAdapter`, `CoinbaseSpotAdapter`, etc.

2. **Orquestração de Execução por Exchange**

   * O Orquestrador é capaz de:

     * Agrupar ordens por `exchange_id`;
     * Respeitar limites de rate limit e políticas específicas de cada exchange;
     * Atribuir janelas de execução diferentes conforme capacidades (por exemplo, algumas exchanges podem ter latência maior).

3. **Risco e Alocação entre Exchanges**

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

A **gestão de estado** em produção garante que o sistema sempre saiba:

* Quais posições estão abertas;
* Quais ordens estão pendentes;
* Qual é o P&L atual (realizado e não realizado);
* Qual é a exposição por estratégia, família, ativo e exchange.

Os principais componentes da gestão de estado são:

1. **Estado Interno (In-Memory)**

   * Estruturas de dados mantidas em memória, com:

     * Posições consolidadas;
     * Ordens recentes e status;
     * Exposição corrente por camada de risco.
   * Otimizadas para consulta rápida pelo módulo de risco e pelas estratégias.

2. **Estado Persistido (Banco de Dados)**

   * Todas as informações relevantes são persistidas em:

     * Banco local (SQLite) para ordens, fills, posições, P&L;
     * Arquivos estruturados (Parquet/CSV) para séries de P&L e dados de mercado.
   * Isso garante:

     * Recuperação após falhas ou restart;
     * Base histórica consistente para auditoria.

3. **Reconciliation com a Exchange**

   * Periodicamente (e sempre que há sinais de inconsistência), o sistema realiza um **reconciliation**:

     * Compara o estado interno com o estado reportado pela exchange (open orders, saldos, posições);
     * Ajusta o estado interno em caso de divergências (por exemplo, cancels manuais ou fills não recebidos por uma perda temporária de conexão).
   * Esse processo reduz o risco de “estado fantasma” (por exemplo, acreditar que não há posição quando existe, ou vice-versa).

4. **Versionamento e “Snapshots”**

   * O sistema pode gerar **snapshots de estado** em momentos-chave (antes e depois de grandes mudanças, início/fim de dia…), para facilitar análise posterior e eventuais replays.

---

### 15.5. Failover e Recuperação

Em ambiente real, falhas são inevitáveis: queda de rede, travamento de processo, instabilidade da exchange, falha de disco etc. O sistema inclui mecanismos para **falhar com graça** e recuperar o estado com segurança.

1. **Failover Local (Single Node)**

   * Em caso de queda do processo:

     * No restart, o sistema:

       * Recarrega o estado persistido (posições, ordens);
       * Realiza reconciliation com a exchange;
       * Reconstrói buffers de dados de mercado a partir do histórico local + backfill REST;
       * Retoma o core loop em modo consistente.
   * Estratégias podem ter um pequeno período de “warm-up” para recálculo de indicadores.

2. **Gestão de Retries e Backoff**

   * Para falhas de rede/API:

     * São aplicadas políticas de retry com backoff exponencial;
     * Logs detalham falhas e tentativas;
     * Em caso de falha repetida, circuit breakers específicos da exchange podem ser acionados (suspensão temporária de novas ordens).

3. **Modo Degradado**

   * Se determinadas funcionalidades falham (por exemplo, book em tempo real indisponível, mas candles ainda funcionam):

     * Estratégias que dependem do recurso falho podem ser automaticamente desativadas;
     * Estratégias menos sensíveis seguem operando (por exemplo, TSM diário).

4. **Replays e Auditoria Pós-Falha**

   * Por meio das trilhas de logs e dados persistidos, é possível:

     * Reproduzir o estado antes da falha;
     * Reexecutar parte do core loop em modo “somente simulação” para entender impactos;
     * Documentar eventos críticos para melhorar políticas futuras.

Mesmo em single node, o objetivo é que uma falha pontual não se traduza em perda de controle do portfólio.

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

## 16. Observabilidade e Telemetria

A camada de **Observabilidade e Telemetria** tem como objetivo responder, em tempo real e de forma retroativa, a três perguntas fundamentais:

1. **O que está acontecendo agora?**
2. **O que aconteceu ao longo do tempo?**
3. **Por que o sistema tomou determinada decisão de trading?**

Ela é transversal a todo o sistema: coleta sinais desde a aquisição de dados de mercado, passando por estratégias, módulo de risco, execução de ordens, até infraestrutura (CPU, memória, disco, rede).
O foco é permitir **operação touch-zero**, com capacidade de **detecção precoce de problemas**, análise pós-mortem e **auditoria completa** das decisões de trading.

---

### 16.1. Logs Estruturados

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

Ao menos três painéis principais são previstos:

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

**Tracing** é a capacidade de seguir, ponta a ponta, a cadeia:

> Dado de mercado → Sinal → Decisão de Risco → Ordem → Execução → P&L

No sistema, isso é feito com o uso de identificadores de correlação e registros detalhados em cada etapa:

* **Correlation/Trace IDs**

  * Cada “ciclo de decisão” (por estratégia/timeframe) recebe um `trace_id`.
  * Todos os eventos gerados nesse ciclo (logs de estratégia, risco, ordens, fills) carregam esse mesmo `trace_id`.

* **Ligação com Versões de Estratégia e Configuração**

  * Para cada `trace_id`, são também registradas:

    * Versão de código da estratégia (`strategy_version`);
    * Versão de configuração (`config_version`), incluindo parâmetros de risco.

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

A camada de **Governança de Estratégias e Change Management** define *como* novas ideias viram estratégias em produção, *como* essas estratégias são modificadas ao longo do tempo e *como* tudo isso é registrado, auditado e, quando necessário, revertido com segurança.

O objetivo é evitar dois extremos igualmente perigosos:

* De um lado, um sistema rígido demais, em que qualquer mudança é difícil e lenta;
* De outro, um “laboratório solto”, em que ajustes de parâmetros e deploys em produção são feitos de forma ad hoc, sem registro ou avaliação estruturada.

Aqui, governança não é burocracia: é **disciplina mínima** para proteger capital, preservar reprodutibilidade e permitir evolução contínua do portfólio de estratégias.

---

### 17.1. Pipeline de Aprovação de Estratégias

Toda estratégia (e, em menor grau, cada nova instância ou grande revisão de parâmetros) passa por um **pipeline padronizado de aprovação**, com etapas bem definidas:

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

Em um sistema em constante evolução, **versionar estratégias e configurações** é crítico para entender mudanças de comportamento ao longo do tempo.

#### 17.2.1. Versão de Estratégia (Código)

Cada família/instância de estratégia possui atributos como:

* `strategy_family` (ex.: `TSM`, `DUAL_MOMENTUM`);
* `strategy_id` (ex.: `TSM_BTCUSDT_1D`);
* `strategy_version` (ex.: `1.0.3`).

A `strategy_version` muda quando há alterações de:

* Lógica de decisão (regras de entrada/saída/filtros);
* Dependências de dados (novos indicadores, mudanças de cálculo);
* Implementação interna que possa impactar o comportamento (por exemplo, fix de bug que altere sinais históricos).

Cada commit relevante para estratégias deve atualizar a versão de forma clara, permitindo:

* Associar resultados de backtest/paper/live a uma versão exata de código;
* Comparar performance entre versões.

#### 17.2.2. Versão de Configuração

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

Para rollback eficaz, o sistema precisa:

* Manter os binários/código e configs da versão anterior disponíveis;
* Saber exatamente qual combinação estava em produção (`strategy_version`, `config_version`, parâmetros de risco) antes da mudança.

O processo de rollback normalmente é:

1. Identificação do problema:

   * Métricas de risco ou erro saem do esperado;
   * Bugs ou comportamento errado de lógica são detectados.

2. Ação de Governança:

   * Decisão explícita de:

     * Voltar para versão anterior;
     * Ou desativar a instância/família por segurança.

3. Execução Técnica:

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

A camada de **Segurança e Compliance** define como o sistema protege:

* o **capital** sob gestão,
* as **credenciais** de acesso às exchanges,
* os **dados** (de mercado, de execução e de logs),

e, ao mesmo tempo, se mantém alinhado aos requisitos regulatórios e às boas práticas de mercado para um **sistema proprietário de trading algorítmico**.

O foco é **Security by Design**: segurança não é um “add-on”, mas um conjunto de decisões incorporadas à arquitetura, ao desenvolvimento e à operação desde o início.

---

### 18.1. Gestão de Credenciais e Segredos

A gestão de segredos é tratada como ponto crítico, principalmente por envolver **API keys de exchanges** e eventuais credenciais de notificações (e-mail, Telegram etc.).

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

O sistema foi concebido, primeiro, como **ferramenta de trading proprietário (proprietary trading)**, isto é, operando apenas capital próprio.
Nesse contexto:

* Não há, a princípio, prestação de serviço de investimento a terceiros;
* As obrigações regulatórias são, em geral, menos complexas (mas **podem variar por jurisdição**).

Ainda assim, são consideradas as seguintes dimensões:

* **Classificação da Atividade**

  * Caso, em etapas futuras, o sistema passe a:

    * Gerir capital de terceiros;
    * Oferecer sinais ou execução como serviço;
  * Pode ser necessária:

    * Autorização de órgão regulador (ex.: CVM, SEC, ou equivalente local);
    * Adequação a regras de suitability, informação ao investidor, segregação de contas.

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

### 18.8. Resposta a Incidentes e Continuidade

Por fim, a camada de Segurança e Compliance define **como reagir quando algo foge do esperado**, tanto em termos técnicos quanto de risco:

* **Plano de Resposta a Incidentes**

  * Definição de fluxos para incidentes como:

    * Vazamento suspeito de credenciais;
    * Comportamento anômalo de estratégias (como ordens em loop);
    * Falhas graves de exchange ou perda de conectividade prolongada.
  * Em geral, inclui:

    * Ativação de circuit breakers globais (desalocar posições, bloquear novas ordens);
    * Revogação/rotação de chaves;
    * Análise posterior e documentação do incidente.

* **Backups e Recuperação**

  * Backups regulares de:

    * Banco de dados de estado (posições, ordens, P&L);
    * Configurações e versões de estratégias;
    * Datasets críticos de backtest.
  * Procedimentos para:

    * Restaurar o sistema em caso de perda de dados ou falha de hardware;
    * Reconstruir o estado a partir da exchange (reconciliation) após recuperação.

* **Testes de Continuidade**

  * Periodicamente, simular:

    * Queda de processo e restart;
    * Falha de rede temporária;
    * Erro em uma exchange específica.
  * Verificar se:

    * O sistema volta ao ar em estado consistente;
    * As proteções de risco funcionam conforme esperado;
    * Auditoria registra adequadamente o evento.

---

Com esse conjunto de práticas e mecanismos, a camada de **Segurança e Compliance** garante que o sistema:

* Proteja as credenciais e o capital sob gestão;
* Respeite princípios básicos de segurança de software e infraestrutura;
* Esteja pronto para evoluir, caso seja necessário atender exigências regulatórias mais rigorosas;
* Permita operar com confiança um **framework de famílias de estratégias** em ambiente real, ainda que em single node e inicialmente com uma única exchange.

---

## 19. Roadmap de Evolução

O Roadmap de Evolução define **como** o sistema sai de um protótipo focado em uma única família de estratégias (TSM, spot, single exchange) e progride, em etapas controladas, até um **framework maduro**, multi-família, preparado para multi-exchange e multi-ativo, mantendo sempre o foco em:

* **retorno ajustado ao risco**
* **robustez operacional**
* **simplicidade compatível com single node**

Não há compromisso com datas fixas: as fases são **blocos lógicos**, que podem ser ajustados em função de tempo disponível, capital, resultados de pesquisa e restrições externas (por exemplo, acesso a novos mercados).

---

### 19.1. Princípios Orientadores do Roadmap

Antes das fases, o roadmap se apoia em alguns princípios:

1. **Segurança e controle > velocidade de expansão**

   * Nenhuma nova família de estratégia entra em produção sem:

     * backtest coerente com a tese;
     * período mínimo em paper trading;
     * integração com o módulo de risco e observabilidade.

2. **Complexidade progressiva**

   * Começar pelo **simples e robusto** (TSM, spot, single asset);
   * Só depois adicionar:

     * multi-ativo, multi-família;
     * estratégias mais frágeis (mean reversion intraday, market making);
     * integrações com derivativos e outras classes de ativos.

3. **Isomorfismo backtest ↔ produção**

   * Sempre que possível, a **mesma lógica de estratégia e de risco** usada em produção é usada no backtest (mesmo código, modos diferentes);
   * Evita divergências “fantasiosas” entre o que funciona no histórico e o que acontece na prática.

4. **Desacoplamento e modularidade**

   * Estratégias não enviam ordens: geram sinais de posição alvo;
   * Módulo de risco decide o que pode ou não ser executado;
   * Módulo de execução fala com a exchange;
   * Observabilidade e governança registram tudo.

5. **Preparado para multi-exchange, operando inicialmente com uma só**

   * Desde cedo, os módulos devem ser pensados com **camada de abstração por exchange** (adapters), mesmo que apenas Binance Spot esteja ativa no início.

---

### 19.2. Fase 0 – Fundação Arquitetural e Organizacional

**Objetivo:** estabelecer a base técnica e organizacional mínima para todo o restante do projeto.

Principais entregas:

* **Estrutura de repositório e módulos**

  * Separação clara entre:

    * dados (ingestão, caching);
    * estratégias;
    * risco;
    * execução;
    * observabilidade;
    * governança/configuração.

* **Modelo de configuração central (ex.: YAML + Pydantic)**

  * Estratégias, exchanges, símbolos, timeframes, limites de risco e modos (backtest/paper/live) definidos de forma declarativa.

* **Padrões de desenvolvimento e CI básico**

  * Formatação, lint, testes unitários iniciais;
  * Pipeline simples para rodar testes e (quando aplicável) backtests de sanidade.

* **Segurança mínima**

  * API keys fora do código (variáveis de ambiente ou arquivo criptografado);
  * Usuário de sistema dedicado;
  * Princípio do menor privilégio em acessos.

Essa fase não mira lucro nem produção, mas **preparar terreno** para tudo que vem depois.

---

### 19.3. Fase 1 – MVP Técnico: TSM Single Asset em Spot

**Objetivo:** colocar em pé a primeira estratégia completa, ponta a ponta, em modo ainda experimental.

Escopo principal:

* **Família 1 – TSM básica, single asset (por exemplo, BTCUSDT)**

  * Timeframe diário (ou 4h/1D), lookback único;
  * Regras simples de:

    * compra quando retorno L > 0;
    * ficar em cash quando retorno L ≤ 0.

* **Aquisição de dados de mercado (Binance Spot)**

  * Backfill histórico de OHLCV;
  * Coleta periódica de novos candles;
  * Normalização de timestamps e símbolos.

* **Motor de backtest mínimo**

  * Capaz de:

    * simular TSM em histórico;
    * modelar custos básicos (fees, slippage fixo simplificado);
    * calcular métricas simples (retorno, DD, Sharpe).

* **Módulo de execução simplificado**

  * Enviar ordens spot reais ou simuladas (dependendo do modo);
  * Reconciliar posição com a exchange.

* **Logs estruturados e P&L básico**

  * Logar decisões da estratégia;
  * Manter P&L por estratégia/ativo.

A saída desejada desta fase é: **um pipeline inteiro funcionando** com TSM em um único ativo, provando que a arquitetura “fecha” do dado à ordem e de volta à auditoria.

---

### 19.4. Fase 2 – Endurecimento do Núcleo: Risco, Observabilidade e Governança

**Objetivo:** transformar o MVP em um núcleo confiável, apto a operar TSM com pequeno capital real.

Eixos principais:

* **Módulo de Gestão de Risco Multicamadas (mínimo viável)**

  * Limites por trade e por instância TSM;
  * Limite de DD diário/semanal/mensal global;
  * Circuit breaker global simples:

    * excedeu DD → para novas entradas, só permite saídas.

* **Backtest mais fiel à produção**

  * Uso das mesmas funções de:

    * cálculo de sinais;
    * aplicação de risco;
    * regra de execução simplificada.
  * Melhor modelagem de custos:

    * fees variáveis;
    * slippage proporcional à volatilidade ou volume.

* **Observabilidade inicial**

  * Logs estruturados por componente (dados, estratégia, risco, trading);
  * Métricas básicas:

    * latência do loop;
    * número de ordens;
    * P&L por dia;
    * drawdown corrente.

* **Governança mínima de estratégias**

  * Processo simples de:

    * “TSM v1.0” → backtest → paper → live;
    * versionamento de estratégia e de configuração.

Ao fim dessa fase, TSM single asset pode ser operada em **modo live com capital pequeno**, com riscos controlados e capacidade de entender o que o sistema está fazendo.

---

### 19.5. Fase 3 – Expansão de Famílias de Momentum em Spot Multi-Ativo

**Objetivo:** sair do “TSM single asset” e evoluir para um **núcleo multi-ativo e multi-família de momentum**, ainda em mercado spot e single exchange.

Componentes:

* **Multi-ativo para TSM (Família 1)**

  * Estender TSM para operar uma **cesta de ativos** (ex.: top N pares cripto líquidos);
  * Introduzir:

    * filtros de liquidez;
    * limites de exposição por ativo.

* **Família 2 – Dual Momentum**

  * Implementar:

    * filtro TSM (momentum absoluto) sobre o universo;
    * ranking cross-sectional (momentum relativo);
    * seleção de top N/X% ativos para alocação;
  * Integração com módulo de portfólio:

    * distribuição de capital entre TSM “single asset”, TSM multi-ativo e Dual Momentum.

* **Família 3 – Cross-Sectional Momentum (CSM long-only)**

  * Implementar CSM puro, com:

    * ranking por retornos passados;
    * top N vencedores long-only;
  * Alocação controlada (capital pequeno inicialmente), dada maior fragilidade da família.

* **Portfolio Manager e Alocação entre Famílias**

  * Introdução de um módulo de **alocação de capital entre famílias**:

    * orçamento de risco/capital por família (TSM, Dual, CSM);
    * limites de concentração por ativo e por exchange.

* **Preparação para Multi-Exchange (camada de abstração)**

  * Mesmo continuando com Binance Spot como **única exchange ativa**, introduzir:

    * interface genérica de “exchange adapter”;
    * estrutura de configuração que permita incluir novas exchanges no futuro sem reescrever o núcleo.

Ao final da Fase 3, o sistema já opera:

* um conjunto de ativos;
* três famílias de momentum (TSM, Dual, CSM);
* com módulo de risco e portfólio coordenando a exposição global.

---

### 19.6. Fase 4 – Estratégias Táticas de Curto Prazo e Infra Intraday

**Objetivo:** introduzir, de forma cuidadosa, a **Família 6 – Mean Reversion de Curto Prazo (Intraday/Swing)** e endurecer a infraestrutura intraday.

Componentes:

* **Versão inicial de Mean Reversion de curto prazo**

  * Em 1–2 ativos de alta liquidez (por exemplo, BTCUSDT, ETHUSDT);
  * Timeframes como 15m, 1h, 4h;
  * Sinais baseados em:

    * oversold/overbought (RSI, bandas, ATR, etc.);
    * filtros de tendência em timeframe superior.

* **Melhoria na infraestrutura de dados intraday**

  * Coleta confiável de candles intraday;
  * Detecção e correção (ou marcação) de gaps;
  * Testes de performance da ingestão em single node.

* **Refinamento do módulo de risco para alta frequência relativa**

  * Limites de:

    * número de trades por dia/família;
    * perda diária por família de mean reversion;
  * Circuit breakers específicos:

    * sequência de perdas;
    * DD intradiário.

* **Otimização e pesquisa estruturada**

  * Estrutura para rodar:

    * backtests de grade (grid) de parâmetros;
    * análises de sensibilidade;
    * pequenos experimentos de otimização (ex.: busca heurística, não necessariamente algo pesado como Optuna nesta fase).

* **Observabilidade reforçada para intraday**

  * Métricas de:

    * latência do core loop em timeframes mais curtos;
    * slippage efetivo vs modelado;
    * distribuição de P&L por hora/sessão.

Ao término desta fase, a família Mean Reversion deve estar, pelo menos, em **paper trading bem-validado**, com eventual entrada em live com capital muito controlado.

---

### 19.7. Fase 5 – Carry/Yield em Spot e Primeiros Passos em Basis

**Objetivo:** introduzir, de forma conservadora, a **Família 4 – Carry / Yield / Basis**, começando por yield spot e, só depois, experiments com basis/funding.

Etapas:

* **Módulo de Yield Spot (estável e sem alavancagem)**

  * Seleção de:

    * oportunidades de yield em stablecoins;
    * staking de ativos “blue chips”;
  * Foco em:

    * exposição pequena do portfólio;
    * limites de contraparte e protocolo bem definidos.

* **Integração do Carry/Yield com o Portfólio Global**

  * Definição de um **budget de risco/capital** para a família;
  * Métricas específicas de:

    * yield bruto e ajustado ao risco;
    * correlação com outras famílias.

* **Planejamento do módulo de Basis/Funding (opcional)**

  * Em nível de design:

    * como integrar derivativos (perp/futuros) no framework;
    * como o módulo de risco lidará com margem e risco de liquidação;
  * Eventuais testes inicias em **testnet** ou ambiente simulado:

    * long spot + short perp em cenários de funding positivo.

* **Reforço de segurança e compliance**

  * Especialmente se houver contato com derivativos:

    * garantir segregação de chaves;
    * avaliar implicações regulatórias mínimas, mesmo como trading proprietário.

Ao final da Fase 5, a família Carry/Yield deve, idealmente, atuar como **camada pequena de yield estrutural**, com basis ainda em estágio experimental, se for o caso.

---

### 19.8. Fase 6 – Value + Quality e Integração Multi-Classe

**Objetivo:** preparar e introduzir a **Família 5 – Value + Quality**, focada em ações spot, e dar os primeiros passos reais em **multi-classe de ativos**.

Passos principais:

* **Integração com fonte(s) de dados de ações e fundamentals**

  * Conectores para:

    * preços de ações spot (via corretoras ou APIs de mercado);
    * dados fundamentais (P/L, P/B, ROE, margens, dívida etc.).

* **Implementação da família Value + Quality em ações**

  * Definição de universo (ex.: índice amplo, ações líquidas);
  * Cálculo de scores de Value e Quality;
  * Seleção de portfólio long-only de médio/longo prazo;
  * Rebalance de baixa frequência (trimestral, semestral).

* **Extensão do módulo de risco para multi-classe**

  * Limites de exposição por classe (cripto, ações, stable/yield);
  * Visualizações e métricas de risco agregadas multi-ativo.

* **Ajustes na alocação de capital entre famílias**

  * Inserir Value + Quality como:

    * “perna de longo prazo”;
    * com peso crescente à medida que a infraestrutura e a convicção aumentem.

* **Abstração madura de multi-exchange**

  * Com ações e cripto:

    * consolidar o modelo de adapters de exchange/corretora;
    * permitir que estratégias sejam parametrizadas por “fonte de execução” (exchange X, broker Y).

Ao final da Fase 6, o sistema evolui de um “algotrader cripto” para um **framework multi-ativo**, com famílias pensadas para horizontes e estilos diferentes.

---

### 19.9. Fase 7 – Market Making Estruturado e Microestrutura

**Objetivo:** ativar, com muita cautela, a **Família 7 – Market Making / Liquidity Providing**, inicialmente em formato experimental.

Etapas:

* **Simulador de microestrutura e book off-line**

  * Modelo simplificado de:

    * evolução de mid price;
    * execuções contra ordens passivas;
  * Testes das lógicas de:

    * spreads;
    * bands de inventário;
    * cancelamento/recolocação de ordens.

* **Paper Trading com dados de book reais**

  * Consumir book da exchange (ao menos best bid/ask, idealmente depth);
  * Simular execuções localmente;
  * Avaliar:

    * eficiência de spreads;
    * risco de inventário;
    * sensibilidade a latência.

* **Live Experimental com Volume Ínfimo**

  * Operar 1–2 pares extremamente líquidos (ex.: BTCUSDT, ETHUSDT);
  * Notional muito pequeno;
  * Limites de risco muito apertados;
  * Observabilidade reforçada.

* **Integração como Componente Complementar de Portfólio**

  * Se mostrar robustez:

    * alocação moderada de capital;
    * monitoramento contínuo de DD e risco de cauda.

Dadas as exigências dessa família, ela permanece sempre como **módulo avançado opcional**, não obrigatório para o sucesso do framework.

---

### 19.10. Trilhas Transversais Contínuas

Ao longo de todas as fases, há trilhas que **não são “fases isoladas”**, mas iniciativas contínuas:

1. **Segurança e Compliance**

   * Refino contínuo de:

     * gestão de chaves;
     * hardening de ambiente;
     * procedimentos de resposta a incidentes;
   * Avaliação regular de riscos regulatórios, especialmente se:

     * forem adicionadas novas classes de ativos;
     * houver qualquer passo na direção de capital de terceiros.

2. **Governança e Documentação**

   * Manter sempre atualizados:

     * fichas de estratégias (família, instância, parâmetros, riscos);
     * changelog de versões;
     * registros de decisões de promoção/desativação de estratégias.

3. **Observabilidade e Telemetria**

   * Evoluir dashboards e alertas conforme o sistema fica mais complexo:

     * novas famílias;
     * multi-exchange;
     * multi-ativo.

4. **Performance e Eficiência em Single Node**

   * Monitorar custo computacional de:

     * ingestão de dados;
     * geração de sinais;
     * backtests;
   * Ajustar:

     * número de estratégias ativas;
     * frequência de reprocessamentos;
     * caching local (datasets, features).

5. **Pesquisa e Otimização Estruturadas**

   * Criar e manter uma “esteira de pesquisa”:

     * experimentos em novos parâmetros e variações de família;
     * backtests padronizados;
     * relatórios comparáveis;
   * Sempre com visão de evitar overfitting e preservar robustez.

---

Em conjunto, o **Roadmap de Evolução** garante que o sistema:

* não tente “abraçar o mundo” de uma vez;
* avance de forma **ordenada e disciplinada**,
* sempre com a visão central em mente:

> construir, em single node, um **framework de famílias de estratégias sistemáticas** para mercado spot (e, depois, multi-ativo), com foco em **retorno ajustado ao risco**, governança forte e capacidade de evolução incremental.

