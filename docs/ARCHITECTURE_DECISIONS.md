# Registro de Decisões de Arquitetura (ADRs)

Este documento registra as principais decisões filosóficas e de design que moldaram o **Framework de Famílias de Estratégias Spot Multi-Exchange**. O objetivo é manter o documento principal do framework focado no "o quê" e "como", enquanto este arquivo captura o "porquê".

---

## ADR-001: Princípios Fundamentais e Escopo do Projeto

**Contexto:** O projeto visa criar uma plataforma de trading sistemático. É crucial definir desde o início os princípios norteadores e o que está deliberadamente fora do escopo para focar os esforços de desenvolvimento.

**Decisão:**

1.  **Prioridade Central:** A filosofia do projeto é **maximizar o retorno ajustado ao risco, com disciplina de gestão de risco e governança**. Isso significa que a robustez, a previsibilidade e a segurança são mais importantes do que a busca por lucros a qualquer custo. Evita-se improvisos operacionais e complexidade desnecessária para o contexto de `single node`.

2.  **Foco em Evidência Histórica:** O framework prioriza a implementação de famílias de estratégias com **evidência histórica e acadêmica robusta** (ex: Trend Following, Momentum, Value). A intenção não é inventar novas estratégias, mas sim implementar de forma sistemática e segura aquelas que já demonstraram eficácia em múltiplos mercados e regimes.

3.  **Definição de Não-Escopo:** Para manter o foco e a simplicidade no MVP, os seguintes itens são explicitamente excluídos:
    *   **High-Frequency Trading (HFT):** Exige infraestrutura de baixa latência (colocation) e otimizações que estão além do objetivo de um sistema single-node.
    *   **Derivativos Complexos:** O uso intensivo de futuros alavancados, opções, etc., como parte central do sistema, adiciona complexidade de risco e de margem que desvia o foco do framework spot. Serão considerados apenas como módulos opcionais futuros.
    *   **Sistemas Distribuídos:** A prioridade é a **robustez em single node**. Arquiteturas de microserviços ou clusters distribuídos, embora escaláveis, introduzem uma sobrecarga operacional e de complexidade que não se justifica no escopo inicial.
    *   **Gestão de Capital de Terceiros:** O sistema é desenhado para **trading proprietário**. A gestão de capital de terceiros implicaria requisitos regulatórios e de compliance significativamente mais complexos.

4.  **Princípios Operacionais de Maturidade:**
    *   **Zero-Touch Operation:** O sistema deve ser projetado para operar sem intervenção manual no dia a dia. Todas as ações (restarts, reconciliação, circuit breakers) devem ser automatizadas, com a intervenção humana sendo a exceção, não a regra.
    *   **Custo Consciente (Cost-Aware):** A arquitetura e as operações devem ser conscientes dos custos (infraestrutura, taxas de API, custos de transação). As decisões de design devem buscar um balanço ótimo entre performance e custo.
    *   **Progressive Delivery:** Novas estratégias ou versões são entregues progressivamente para mitigar riscos: `Shadow Mode` (rodando em produção sem executar ordens, apenas comparando sinais) → `Canary Release` (live com capital muito pequeno) → `Full Rollout` (live com capital alvo).

**Consequências:**
*   **Positivas:** Foco claro no desenvolvimento, redução de riscos operacionais e de complexidade, alinhamento com uma filosofia de investimento prudente.
*   **Negativas:** O sistema não explorará oportunidades em HFT ou em mercados de derivativos complexos no curto prazo. A escalabilidade horizontal é uma preocupação futura, não imediata.

---

## ADR-002: Modelo de Deploy Single-Node

**Contexto:** Uma das primeiras decisões de arquitetura é definir o modelo de deploy: um único nó (máquina) ou um sistema distribuído.

**Decisão:** O sistema será projetado e otimizado para ser executado em um **único nó** (servidor dedicado, VM ou hardware embarcado de alto desempenho).

**Justificativa:**
*   **Simplicidade Operacional:** Gerenciar um único nó é drasticamente mais simples do que gerenciar um cluster. Isso reduz a sobrecarga de DevOps, monitoramento e resposta a incidentes.
*   **Custo:** O custo de hardware e infraestrutura para um único nó é significativamente menor.
*   **Complexidade de Desenvolvimento:** Evita-se a complexidade inerente a sistemas distribuídos, como consistência de dados, comunicação entre serviços, descoberta de serviços e tolerância a falhas de rede particionada.
*   **Adequação ao Escopo:** Para estratégias de trading não-HFT (com timeframes de minutos a dias), um único nó moderno possui capacidade de processamento, memória e I/O mais do que suficiente.

**Consequências:**
*   **Positivas:** Ciclos de desenvolvimento mais rápidos, menor custo operacional, maior facilidade de debug e manutenção.
*   **Negativas:**
    *   **Ponto Único de Falha (SPOF):** Se o nó falhar, todo o sistema para. Isso é mitigado com processos robustos de recuperação e restart, mas o risco de downtime existe.
    *   **Escalabilidade Limitada:** A escalabilidade é vertical (mais CPU/RAM no mesmo nó), não horizontal. Se o número de estratégias ou o volume de dados crescer exponencialmente, pode ser necessário um redesenho futuro. A arquitetura (ADR-003) é pensada para facilitar essa transição, mas ela não é trivial.

---

## ADR-003: Estilo Arquitetural (Ports & Adapters, Clean Architecture, DDD)

**Contexto:** É necessário escolher um estilo arquitetural que promova desacoplamento, testabilidade e manutenibilidade, especialmente considerando a necessidade de suportar múltiplas exchanges e famílias de estratégias.

**Decisão:** O sistema adotará uma combinação de:
1.  **Arquitetura Hexagonal (Ports & Adapters):** O núcleo da aplicação (domínio) define "portas" (interfaces) para funcionalidades externas (ex: `MarketDataPort`, `TradingPort`). A infraestrutura concreta (ex: `BinanceAdapter`, `SQLiteAdapter`) implementa essas portas como "adapters".
2.  **Clean Architecture:** As dependências apontam para dentro. O domínio no centro não conhece detalhes de infraestrutura. As camadas são: Entidades -> Casos de Uso -> Adapters/Infraestrutura.
3.  **Domain-Driven Design (DDD) Leve:** Foco na modelagem explícita do domínio de trading (ex: `Asset`, `Order`, `Position`, `StrategyFamily`) e no uso de uma Linguagem Ubíqua.

**Justificativa:**
*   **Testabilidade:** O domínio (lógica de estratégias, regras de risco) pode ser testado em isolamento, sem depender de conexões reais com exchanges ou bancos de dados. Mocks podem ser facilmente injetados nos testes.
*   **Portabilidade e Extensibilidade (Multi-Exchange):** Para adicionar uma nova exchange, basta criar um novo adapter. O núcleo do sistema permanece intocado. O mesmo vale para adicionar um novo tipo de banco de dados ou serviço de notificação.
*   **Evolutividade:** Novas famílias de estratégias podem ser adicionadas como novos módulos de domínio sem quebrar a infraestrutura existente.
*   **Clareza Conceitual:** A separação clara entre "o quê" (domínio) e "como" (infraestrutura) torna o código mais fácil de entender e manter.

**Consequências:**
*   **Positivas:** Alta coesão, baixo acoplamento, facilidade de teste e evolução. A arquitetura está preparada para crescer.
*   **Negativas:** Pode haver um "boilerplate" inicial maior para definir as portas e os adapters, o que pode parecer um exagero para um sistema simples. No entanto, esse investimento se paga rapidamente à medida que o sistema cresce em complexidade.

---

## ADR-004: Gestão do Universo de Mercados via Schema Discovery

**Contexto:** O sistema precisa saber em quais mercados pode operar (símbolos, timeframes) e quais são as regras de cada um (filtros de lote, notional mínimo). Uma abordagem é configurar isso estaticamente; outra é descobrir dinamicamente.

**Decisão:**
1.  A **API da exchange será tratada como a Fonte Única da Verdade (SSOT)** para o universo de mercados.
2.  O sistema implementará um mecanismo de **Schema Discovery** que, na inicialização e periodicamente, consulta a exchange para descobrir suas capacidades (símbolos, timeframes, filtros, tipos de ordem).
3.  Configurações estáticas serão usadas apenas para *restringir* o universo descoberto (ex: uma whitelist de símbolos), não para *definir* suas propriedades.

**Justificativa:**
*   **Robustez:** Evita erros causados por configurações estáticas desatualizadas. As exchanges mudam filtros e adicionam/removem mercados com frequência. Descobrir dinamicamente garante que o sistema opere com base na realidade atual.
*   **Manutenibilidade:** Reduz a necessidade de atualizar arquivos de configuração manualmente toda vez que a exchange muda algo.
*   **Adaptabilidade Multi-Exchange:** Facilita a integração de novas exchanges, pois o processo de descoberta pode ser padronizado, com cada adapter sendo responsável por traduzir a resposta da sua API para o schema interno do framework.

**Consequências:**
*   **Positivas:** O sistema é mais resiliente a mudanças externas e menos propenso a erros de configuração. A operação é mais segura.
*   **Negativas:** A inicialização do sistema se torna um pouco mais lenta e complexa, pois depende de chamadas de rede para a exchange. O sistema precisa ser capaz de lidar com falhas durante o processo de discovery.

---

## ADR-005: Priorização de Famílias de Estratégias

**Contexto:** O framework suportará múltiplas famílias de estratégias. A ordem de implementação e a alocação de capital entre elas devem seguir uma lógica clara, alinhada aos objetivos do projeto.

**Decisão:** A priorização das famílias seguirá um modelo explícito baseado nos seguintes critérios, em ordem de importância:
1.  **Robustez da Evidência Histórica:** Famílias com ampla documentação acadêmica e empírica de sucesso em múltiplos mercados e regimes (ex: TSM) têm prioridade máxima.
2.  **Viabilidade Prática em Spot Long/Flat:** Famílias que funcionam bem sem a necessidade de short selling, alavancagem ou derivativos complexos são priorizadas.
3.  **Retorno Ajustado ao Risco:** Famílias que historicamente apresentam melhor perfil de Sharpe/Sortino/Calmar e menor vulnerabilidade a crashes são preferidas.
4.  **Complexidade Operacional:** Famílias mais simples de implementar e operar (ex: TSM diário) vêm antes de estratégias mais frágeis e sensíveis a custos (ex: Mean Reversion intraday, Market Making).

**Justificativa:**
*   Este modelo garante que o portfólio seja construído sobre uma base sólida, começando com as estratégias mais confiáveis.
*   Alinha o roadmap de desenvolvimento com a filosofia de gestão de risco do projeto.
*   Fornece um framework objetivo para decisões de alocação de capital.

**Consequências:**
*   **Positivas:** O risco do portfólio é gerenciado de forma mais estruturada. O desenvolvimento foca primeiro no que tem maior probabilidade de gerar resultados consistentes.
*   **Negativas:** Famílias potencialmente muito lucrativas, mas mais arriscadas ou complexas (como Market Making), são postergadas, deixando "dinheiro na mesa" no curto prazo em troca de segurança e robustez.

---

## ADR-006: Rationale das Famílias de Estratégias

**Contexto:** Cada família de estratégia incluída no framework deve ter uma justificativa clara para sua existência, baseada em evidências e adaptada às restrições do projeto.

**Decisão:** Para cada família de estratégia, as seções de **Evidência Histórica**, **Adaptação para Spot Long/Flat** e **Riscos Específicos** servem como o "rationale" para sua inclusão e design.

### Família 1: Trend Following / Time-Series Momentum (TSM)
*   **Evidência:** É a família com a evidência mais robusta e longa na literatura, funcionando em diversas classes de ativos e décadas. Em cripto, estudos mostram que o efeito persiste. É a base de muitos fundos CTA de sucesso.
*   **Adaptação:** A adaptação para `long/flat` é natural e eficaz. A regra `retorno > 0 → long` e `retorno <= 0 → cash` permite participar de grandes tendências de alta e, crucialmente, ficar de fora de grandes crises (bear markets), o que melhora o retorno ajustado ao risco.
*   **Riscos:** O principal risco é o de "whipsaw" em mercados laterais, onde a estratégia pode gerar múltiplos sinais falsos de entrada e saída.

### Família 2: Dual Momentum
*   **Evidência:** Combina a robustez do TSM (momentum absoluto) com a força do Cross-Sectional Momentum (momentum relativo). A literatura mostra que o filtro TSM ajuda a mitigar os piores drawdowns do CSM puro, melhorando o Sharpe Ratio.
*   **Adaptação:** A implementação `long-only` é direta: filtra-se o universo por TSM positivo e, em seguida, ranqueia-se para comprar os melhores. O "lado short" é simplesmente não alocar capital.
*   **Riscos:** Risco de concentração em poucos ativos e risco de rotação brusca (aumentando custos). O filtro TSM ajuda a evitar os piores "momentum crashes", mas não os elimina completamente.

### Família 3: Cross-Sectional Momentum (CSM)
*   **Evidência:** O efeito de que "vencedores continuam vencendo" é um dos mais fortes na literatura de fatores. No entanto, é conhecido por seus "momentum crashes" (reversões violentas).
*   **Adaptação:** A versão `long-only` mitiga parte do risco extremo (pois não está vendida nos perdedores que revertem para cima), mas também perde parte da "edge". É tratada como uma família mais agressiva.
*   **Riscos:** O principal risco é o "momentum crash". Mesmo na versão long-only, se os líderes do ranking sofrem uma correção severa, o portfólio é duramente impactado.

### Família 4: Carry / Yield / Basis
*   **Evidência:** Estratégias de carry são um prêmio de risco clássico em FX e bonds. Em cripto, o funding/basis positivo em bull markets é uma fonte de retorno bem documentada.
*   **Adaptação:** A adaptação para spot foca inicialmente em **yield puro** (staking, juros em stablecoins), que não exige derivativos. O basis trading (long spot + short perp) é um módulo avançado e opcional.
*   **Riscos:** O perfil de risco é "ganhar devagar, perder rápido". Há risco de cauda significativo (crashes de carry), risco de contraparte (falha de plataforma/protocolo) e risco de depeg (para stablecoins).

### Família 5: Value + Quality
*   **Evidência:** Fatores de Value e Quality estão entre os mais estudados em ações, com histórico de retornos anormais no longo prazo. A combinação (bons negócios a bons preços) tende a ser mais robusta que o Value puro.
*   **Adaptação:** A família é naturalmente `long-only` e de baixa rotação. Sua aplicação em cripto é limitada pela falta de métricas de "valor" claras, sendo mais relevante para uma futura expansão multi-classe (ações).
*   **Riscos:** O principal risco é o de longos períodos de underperformance em relação a estratégias de "growth". Há também o risco de "value traps" (empresas baratas por motivos estruturais).

### Família 6: Mean Reversion de Curto Prazo
*   **Evidência:** Há evidência de reversão à média em horizontes de curto prazo em muitos mercados, mas o efeito é menos robusto e mais sensível a custos e microestrutura.
*   **Adaptação:** A adaptação `long/flat` foca em comprar quedas exageradas ("oversold") e realizar lucros ou zerar a posição em altas exageradas ("overbought"), sem operar vendido.
*   **Riscos:** Altamente sensível a custos e slippage. Funciona mal em regimes de tendência forte ("comprar faca caindo"). Alto risco de overfitting em backtests.

### Família 7: Market Making
*   **Evidência:** É uma das fontes de lucro mais estruturais dos mercados, mas em nível profissional, é um jogo de HFT e baixa latência.
*   **Adaptação:** Para um ambiente single-node, a adaptação envolve operar com spreads mais largos e frequência moderada, focando em capturar liquidez de forma menos competitiva.
*   **Riscos:** Risco de inventário (ficar "preso" com uma posição grande em um movimento adverso), risco de cauda (gaps de preço) e risco de latência (ser executado em preços "stale").

---

## ADR-007: Modelo de Risco Desacoplado e Hierárquico

**Contexto:** A gestão de risco é o pilar central do projeto. É preciso decidir se o risco será gerenciado dentro de cada estratégia ou de forma centralizada.

**Decisão:**
1.  A gestão de risco será um **módulo central e desacoplado**.
2.  As estratégias **não gerenciam capital nem enviam ordens**. Elas apenas produzem **sinais de posição alvo** (ex: "BTCUSDT: 100% alocado").
3.  O módulo de risco atua como um **filtro obrigatório** entre os sinais das estratégias e a camada de execução.
4.  O risco será organizado em **camadas hierárquicas**: Trade -> Estratégia -> Família -> Ativo -> Exchange -> Portfólio Global.

**Justificativa:**
*   **Controle Centralizado:** Garante que nenhuma estratégia, por um bug ou design ruim, possa violar os limites de risco globais e comprometer o portfólio inteiro.
*   **Segurança:** Isola a lógica de risco em um único local, facilitando a auditoria, o teste e a manutenção.
*   **Flexibilidade:** Permite que as estratégias sejam focadas apenas na geração de sinais, sem se preocuparem com o capital disponível, o estado do portfólio ou os limites globais.
*   **Governança:** Facilita a alocação de capital entre famílias (estática ou dinâmica), pois o módulo de risco é o ponto de controle para implementar essas políticas.

**Consequências:**
*   **Positivas:** Sistema muito mais seguro e robusto. As decisões de risco são consistentes e auditáveis.
*   **Negativas:** O fluxo de decisão tem um passo a mais (sinal -> risco -> ordem), o que pode adicionar uma pequena latência. A lógica do módulo de risco se torna mais complexa, pois precisa consolidar sinais de múltiplas estratégias.

### Rationale de Dimensionamento de Posição (Position Sizing)

Dentro do módulo de risco, a decisão de *quanto* alocar a um trade específico (position sizing) é crítica.

**Decisão:** O framework oferece múltiplos métodos de dimensionamento, mas **recomenda o uso de modelos baseados em risco percentual e volatilidade**, e **desaconselha fortemente o uso do Critério de Kelly completo**.

**Justificativa:**
*   **Risco Percentual Fixo:** Simples e robusto. Dimensiona a posição para que uma perda até o stop-loss represente um percentual fixo do capital (ex: 0.5%). Fácil de entender e controlar.
*   **Dimensionamento por Volatilidade (Volatility Targeting):** Ajusta o tamanho da posição inversamente à volatilidade do ativo. Em períodos de alta volatilidade, as posições são menores, mantendo o risco (em valor) mais constante.
*   **Critério de Kelly:** Embora teoricamente ótimo para maximizar o crescimento do capital no longo prazo, o "Full Kelly" é extremamente perigoso na prática. Ele exige conhecimento preciso das probabilidades e payoffs (o que é impossível nos mercados financeiros) e leva a uma volatilidade de portfólio e drawdowns inaceitáveis.
*   **Kelly Fracionário:** Uma abordagem mais segura é usar uma fração do Kelly (ex: 1/4 ou 1/2 do tamanho sugerido). O framework permite isso, mas trata como uma opção avançada que exige um entendimento profundo de seus riscos.

**Consequências:** A priorização de métodos de dimensionamento mais simples e robustos em detrimento do Kelly completo aumenta a previsibilidade e a segurança do portfólio, mesmo que isso signifique não maximizar teoricamente a taxa de crescimento em cenários ideais.

---

## ADR-008: Framework de Backtesting Isomórfico

**Contexto:** Os resultados de backtest precisam ser o mais próximo possível da realidade da produção para serem confiáveis.

**Decisão:** O sistema adotará um **engine de backtest unificado e isomórfico**.

**Justificativa:**
*   **Isomorfismo:** A palavra-chave é reuso. O motor de backtest reutiliza **exatamente o mesmo código de domínio** da produção, incluindo:
    *   A `Strategy API` e a implementação das estratégias.
    *   O módulo de risco e alocação de capital.
    *   A lógica de gestão de estado (posições, P&L).
*   A única diferença entre backtest e produção está nos **adapters**:
    *   **Fonte de Dados:** Dados históricos de arquivos locais vs. stream de dados ao vivo da exchange.
    *   **Executor:** Um `TradingPortSimulado` que simula fills com base nos candles históricos vs. um `TradingPort` real que envia ordens para a exchange.
*   **Confiabilidade:** Esta abordagem minimiza a divergência entre "o que foi testado" e "o que roda em produção", um dos maiores problemas em sistemas de trading. Qualquer bug fix ou melhoria na lógica de risco beneficia ambos os ambientes simultaneamente.
*   **Validação Estatística:** Para combater o overfitting, o framework de validação deve incluir, como prática padrão, técnicas como **divisão In-Sample/Out-of-Sample (IS/OOS)** e **Walk-Forward Analysis**.

**Consequências:**
*   **Positivas:** Backtests muito mais confiáveis. Redução drástica de bugs que só aparecem em produção. Facilita o uso de backtests como testes de regressão.
*   **Negativas:** O motor de backtest pode ser um pouco mais lento do que soluções altamente otimizadas e vetorizadas, pois simula o loop de eventos de forma mais fiel.

---

## ADR-009: Decisões de Módulos Transversais

**Contexto:** Vários módulos (Execução, Observabilidade, Governança, etc.) contêm decisões de design importantes que definem a operação do sistema.

**Decisões:**

*   **Execução (Core Loop Explícito):** A execução do sistema é baseada em um **core loop explícito e agendado**. Esta decisão torna o fluxo de controle previsível e fácil de depurar, em contraste com sistemas puramente reativos a eventos, que podem ser mais difíceis de rastrear. A estratégia de evolução para multi-exchange é baseada na arquitetura de adapters, garantindo que o core loop não precise ser modificado.

*   **Observabilidade (Por Padrão):** A filosofia é de **"observabilidade por padrão"**. Todos os componentes são instrumentados desde o início com logs estruturados e métricas. O uso de `correlation_id` para **tracing** é mandatório para permitir o rastreamento de uma decisão de ponta a ponta (do dado ao P&L). Isso é crucial para a auditoria e a análise de causa raiz.

*   **Governança (Pipeline e Versionamento):** A governança é implementada através de um **pipeline de aprovação** claro (Pesquisa -> Backtest -> Paper -> Live) e um **versionamento rigoroso** de código e configurações. Isso garante que nenhuma mudança seja feita em produção de forma ad-hoc, preservando a reprodutibilidade e o controle.

*   **Segurança (Security by Design):** O princípio de **"Security by Design"** é adotado, com práticas como a segregação de segredos e o princípio do menor privilégio. A decisão de focar em **trading proprietário** no MVP simplifica drasticamente os requisitos de compliance e segurança, evitando a complexidade regulatória associada à gestão de capital de terceiros.

*   **Roadmap (Documento Estratégico):** Toda a seção de **Roadmap** é, por natureza, um documento de rationale. Ela justifica a ordem de implementação das fases e funcionalidades, alinhando a evolução técnica com a estratégia de negócio e de risco do projeto. Ela deve ser mantida como um documento vivo, separado do framework técnico principal.

*   **CI/CD (Ambientes Reprodutíveis):** A decisão de usar **Docker e Docker Compose** para o ambiente de desenvolvimento garante a reprodutibilidade e consistência entre os ambientes de desenvolvimento, teste e produção. O **Git Workflow** baseado em Pull Requests com revisão obrigatória e CI/CD automatizado reforça a qualidade e a segurança do código.

**Consequências:**
*   **Positivas:** O sistema se torna mais robusto, seguro, auditável e fácil de manter. A evolução é controlada e alinhada com os princípios do projeto.
*   **Negativas:** Há uma sobrecarga de processo (ex: pipeline de aprovação, versionamento) que pode tornar o desenvolvimento inicial um pouco mais lento, mas que se paga com a redução de erros e incidentes em produção.

---

## ADR-010: Orquestração de Pipeline via DAG Runner

**Contexto:** As operações do sistema (coleta de dados, cálculo de features, geração de sinais, execução) formam um pipeline com dependências. É preciso um orquestrador para gerenciar essa execução. As alternativas comuns são `cron`, bibliotecas de agendamento, ou frameworks complexos como Apache Airflow.

**Decisão:** O sistema utilizará um **orquestrador leve e customizado, baseado em um executor de Grafos Acíclicos Dirigidos (DAGs)**, em vez de adotar uma ferramenta externa pesada.

**Justificativa:**
*   **Leveza e Adequação ao Single-Node:** Ferramentas como Airflow são projetadas para ambientes distribuídos e trazem uma sobrecarga significativa (webserver, scheduler, workers, banco de metadados) que é excessiva para um ambiente single-node. Um DAG runner interno, possivelmente construído sobre `asyncio`, é muito mais leve.
*   **Controle e Customização:** Um orquestrador próprio permite um controle fino sobre a execução, priorização de tarefas, e integração com o estado interno do sistema (ex: pausar um DAG se um circuit breaker for ativado), algo mais complexo de se fazer com ferramentas genéricas.
*   **Simplicidade Operacional:** Evita a dependência de um serviço externo complexo, simplificando o deploy e a manutenção. O orquestrador é apenas mais uma parte do código do bot, não um sistema separado.

**Consequências:**
*   **Positivas:** Performance otimizada para o caso de uso, menor consumo de recursos, maior controle sobre o fluxo de execução e menos dependências externas.
*   **Negativas:** Exige o desenvolvimento e manutenção de um componente customizado. A interface gráfica e a robustez de um sistema como Airflow não estarão disponíveis "de graça".

---

## ADR-011: Visão de Risco Agregada para Multi-Exchange

**Contexto:** Em um ambiente multi-exchange, os limites de risco podem ser aplicados por exchange ou de forma global. Por exemplo, um limite de exposição de 30% em BTC poderia significar 30% na Binance e 30% na Kraken, ou 30% no total.

**Decisão:** Todos os limites de risco por ativo são aplicados de forma **agregada e global**, com base em um **identificador canônico do ativo**, independentemente da exchange onde a posição está.

**Justificativa:**
*   **Segurança e Visão Holística:** Esta é a única abordagem que fornece uma visão real do risco do portfólio. Aplicar limites por exchange permitiria que o risco total em um ativo escalasse linearmente com o número de exchanges, criando uma brecha de segurança perigosa.
*   **Consistência:** Garante que a política de risco (ex: "não mais que 30% do capital em BTC") seja sempre verdadeira, não importando como as posições estão distribuídas fisicamente.
*   **Prevenção de Arbitragem de Risco:** Impede que estratégias (ou operadores) contornem os limites de risco simplesmente movendo a atividade para uma nova exchange.

**Consequências:**
*   **Positivas:** O modelo de risco é muito mais robusto e seguro. As regras de risco são mais simples de definir e auditar.
*   **Negativas:** Requer uma camada de abstração mais robusta para:
    1.  Mapear símbolos específicos de cada exchange (ex: `BTCUSDT`, `XBT-USD`) para um símbolo canônico (`BTC`).
    2.  Agregar posições e P&L de múltiplas exchanges em tempo real para alimentar o módulo de risco global.

---

## ADR-012: Guardrails para Integração de IA/LLM no Ciclo de Otimização

**Contexto:** O uso de Modelos de Linguagem Grandes (LLMs) e outras formas de IA para sugerir ou otimizar estratégias de trading é uma área de grande interesse. No entanto, isso introduz riscos significativos se não for controlado.

**Decisão:**
1.  LLMs e outros modelos de IA são estritamente confinados a um papel de **"ferramenta de sugestão"** ou **"assistente de pesquisa"**.
2.  Esses modelos **não têm permissão de escrita direta** em código de produção, configurações de estratégias ou parâmetros de risco.
3.  Qualquer proposta gerada por uma IA (seja um novo trecho de código para uma estratégia, um novo conjunto de parâmetros ou uma nova configuração) **deve, obrigatoriamente, passar pelo mesmo pipeline de governança humano** (ADR-009), incluindo:
    *   Revisão de código por um desenvolvedor.
    *   Execução de backtests rigorosos.
    *   Período em paper trading.
    *   Aprovação explícita de um responsável por risco.

**Justificativa:**
*   **Segurança e Controle (Human-in-the-Loop):** Esta decisão impõe uma barreira de segurança crítica, garantindo que nenhuma decisão autônoma e não auditada por um humano possa impactar o capital em produção. LLMs podem "alucinar" ou gerar código sutilmente falho que só seria pego em testes rigorosos.
*   **Responsabilidade e Auditoria:** Mantém a cadeia de responsabilidade clara. A decisão final de colocar uma estratégia em produção é sempre de um humano, que se baseou em dados e evidências (sejam elas geradas por IA ou não).
*   **Prevenção de Overfitting Sofisticado:** IAs podem ser extremamente boas em encontrar padrões espúrios em dados históricos. O pipeline de governança humano atua como um filtro para questionar a robustez e a lógica econômica por trás das sugestões da IA.

**Consequências:**
*   **Positivas:** Aumento drástico da segurança, controle e previsibilidade. Evita os riscos catastróficos associados a sistemas de IA autônomos em ambientes complexos como os mercados financeiros.
*   **Negativas:** O ciclo de otimização pode ser mais lento do que um sistema totalmente automatizado por IA. A vantagem competitiva não vem da velocidade da IA, mas da sinergia entre as sugestões da IA e o julgamento e rigor do processo humano.

---