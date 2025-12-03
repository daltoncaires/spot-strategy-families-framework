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
    *   **Zero-Touch Operation (como ideal):** O sistema deve ser projetado para operar sem intervenção manual em regime normal. Todas as ações rotineiras (restarts, reconciliação, execução de estratégias) devem ser automatizadas. A intervenção humana é tratada como uma camada de segurança essencial e a ação de último recurso para eventos anormais e excepcionais, não como parte da operação diária.
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
    *   **Escalabilidade Limitada:** A escalabilidade é vertical (mais CPU/RAM no mesmo nó), não horizontal. Se o número de estratégias ou o volume de dados crescer exponencialmente, pode ser necessário um redesenho futuro.
*   **Mitigação do SPOF:** O risco de SPOF é endereçado através de uma estratégia de resiliência avançada, incluindo replicação de estado em tempo real e um modelo de failover Hot-Warm (ver **ADR-014**).

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

### Evolução Futura: Alocação Dinâmica

A priorização estática de famílias serve como um **guardrail de segurança** e um ponto de partida robusto. No entanto, ela pode não ser a ideal do ponto de vista de otimização de portfólio.

**Decisão Futura:** Planeja-se introduzir um **"Otimizador de Alocação Dinâmica"** como uma camada superior de governança.

**Justificativa:**
*   Um otimizador pode ajustar os "budgets" de risco alocados a cada família com base em métricas como a **correlação recente** entre elas, a **volatilidade** e o **retorno ajustado ao risco (Sharpe/Sortino) de cada uma**.
*   Isso permitiria que o sistema, por exemplo, reduzisse a alocação em duas famílias de momentum que se tornaram altamente correlacionadas e aumentasse a alocação em uma família de Mean Reversion, se esta estiver atuando como um bom diversificador no regime de mercado atual.

**Implementação Planejada:**
*   O otimizador não terá permissão para violar os tetos de risco definidos pela prioridade estática. Ele atuará *dentro* das bandas permitidas.
*   Inicialmente, suas recomendações serão apenas isso: **sugestões** para o comitê de risco/operador.
*   Numa fase mais madura, ele poderá ser autorizado a fazer pequenos ajustes automáticos, sempre com monitoramento e alertas rigorosos.

**Consequências:** Esta evolução move o sistema de um modelo de alocação puramente baseado em regras estáticas para um **modelo híbrido**, que combina a segurança dos guardrails com a adaptabilidade da otimização de portfólio moderna.

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

### Família 8: Anti-Fragile
*   **Evidência:** Baseia-se no conceito de que certos ativos ou estratégias se beneficiam do caos e da desordem (alta volatilidade). A ideia é construir um portfólio que tenha uma exposição convexa a picos de volatilidade.
*   **Adaptação:** A implementação `long/flat` foca em ativar a estratégia apenas quando a volatilidade (implícita ou realizada, como VIX ou BVOL) ultrapassa um limiar crítico e há uma tendência clara. Ela complementa perfeitamente a TSM, que sofre em alta volatilidade, atuando como um "hedge de regime". A estratégia pode comprar ativos que se beneficiam da volatilidade ou simplesmente ativar uma versão mais agressiva de TSM apenas nesses períodos.
*   **Riscos:** O principal risco é o de "sinal falso" de volatilidade, onde o sistema entra em um movimento que rapidamente se reverte. Além disso, a medição da volatilidade implícita pode exigir fontes de dados externas (ex: Deribit para BVOL), adicionando complexidade.

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

### Rationale de Alocação Global (Global Volatility Targeting)

Além do dimensionamento por posição, o controle da alavancagem total do portfólio é um pilar de risco.

**Decisão:** O sistema implementará um mecanismo de **Global Volatility Targeting**. A alocação total do portfólio (exposição agregada) será ajustada dinamicamente para atingir uma meta de volatilidade anualizada constante (ex: 15%).

**Justificativa:**
*   **Controle de Risco Proativo:** Em vez de reagir a drawdowns, o sistema reduz a exposição proativamente quando a volatilidade do mercado aumenta, e a aumenta quando a volatilidade diminui.
*   **Melhora do Retorno Ajustado ao Risco:** Essa técnica tem forte evidência histórica de que melhora drasticamente os retornos ajustados ao risco, especialmente o **Calmar Ratio** (retorno sobre o drawdown máximo), pois ajuda a evitar as piores perdas durante períodos de pânico no mercado.
*   **Consistência de Risco:** Mantém o risco do portfólio mais estável ao longo do tempo, independentemente do regime de mercado.

**Consequências:**
*   **Positivas:** Redução de drawdowns, melhora do perfil de risco-retorno, maior robustez do portfólio.
*   **Negativas:** Em mercados de baixa volatilidade que sobem de forma contínua, a estratégia pode ficar mais alavancada (se permitido), e em "flash crashes" a reação pode não ser instantânea. Requer uma medição robusta da volatilidade realizada (ex: janela de 20-60 dias).

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

### Abordagem Dual de Backtesting: Pesquisa Vetorizada vs. Validação Isomórfica

**Contexto:** O motor isomórfico, apesar de fiel, é lento demais para a fase de pesquisa e otimização de parâmetros em larga escala (`parameter sweeps`). É necessário um modo de teste mais rápido para exploração inicial.

**Decisão:** O framework suportará dois modos de backtest distintos, formando um fluxo de trabalho em duas etapas:

1.  **Modo de Pesquisa Vetorizada:**
    *   **Objetivo:** Exploração rápida para identificar parâmetros promissores.
    *   **Implementação:** Utiliza a stack de alta performance (`Polars`, `DuckDB/Ibis`) para aplicar a lógica de sinais de forma vetorial sobre todo o dataset histórico de uma vez. A lógica de execução e risco é simplificada.
    *   **Uso:** Exclusivamente para a fase de pesquisa. Os resultados são considerados aproximações.

2.  **Modo de Validação Isomórfica:**
    *   **Objetivo:** Validação de alta fidelidade dos melhores candidatos a parâmetros encontrados no modo de pesquisa.
    *   **Implementação:** Utiliza o motor de backtest unificado e isomórfico descrito acima, simulando o `core loop` evento a evento.
    *   **Uso:** Obrigatório para qualquer estratégia ou configuração antes de ser promovida para `paper` ou `live trading`.

**Consequências e Mitigação de Riscos:**
*   **Positivas:** Otimiza o tempo do pesquisador, combinando velocidade na exploração com rigor na validação final.
*   **Negativas (Risco de Divergência):** A lógica do motor vetorizado pode divergir sutilmente da implementação isomórfica, especialmente em regras complexas de risco e execução.
*   **Mitigação:**
    *   O código de **geração de sinais** da estratégia deve ser compartilhado entre os dois modos para minimizar a divergência.
    *   Um conjunto de **"testes de coerência"** será mantido. Estes são backtests curtos executados em ambos os modos. O pipeline de CI falhará se os resultados (P&L, número de trades) divergirem além de um `epsilon` tolerável, forçando a sincronização das lógicas.

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

**Decisão:** O sistema evoluirá de um orquestrador de DAG customizado para uma arquitetura de processamento de dados baseada em **Ibis e DuckDB**.

**Justificativa:**
*   **Performance Vetorial:** Ferramentas como `DuckDB` são extremamente otimizadas para cálculos analíticos e vetoriais, superando em ordens de magnitude a performance de loops Python ou orquestradores genéricos para tarefas de manipulação de dados em larga escala (como o cálculo de features para múltiplos ativos).
*   **API Unificada com Ibis:** O `Ibis` fornece uma API declarativa e agnóstica ao backend (semelhante a um "Pandas preguiçoso"), que pode ser traduzida para SQL otimizado para o DuckDB. Isso mantém o código Python limpo e legível, enquanto a execução é delegada a um motor de alta performance.
*   **Simplicidade e Adequação ao Single-Node:** A combinação Ibis + DuckDB opera inteiramente dentro do processo Python, sem a necessidade de serviços externos, mantendo a simplicidade operacional do ambiente single-node, mas com um ganho de performance massivo.

**Consequências:**
*   **Positivas:** Aumento drástico na velocidade de cálculo de features, menor consumo de CPU para tarefas de dados, código de transformação de dados mais expressivo e testável.
*   **Negativas:** Adiciona novas dependências ao projeto (`ibis-framework`, `duckdb`). Requer uma curva de aprendizado para a equipe se adaptar à API do Ibis e ao pensamento de "lazy evaluation".

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

## ADR-013: Stack de Tecnologia para Performance

**Contexto:** Para garantir que o sistema possa escalar em número de ativos e estratégias em um ambiente single-node, a escolha da stack de tecnologia para manipulação de dados e comunicação é crítica. As decisões devem priorizar a eficiência de CPU, memória e uso de rede.

**Decisões:**

1.  **Uso de Polars em vez de Pandas:**
    *   **Decisão:** O `Polars` será adotado como o framework primário para manipulação de DataFrames em memória.
    *   **Justificativa:** O Polars é construído em Rust e projetado para paralelismo desde o início. Em benchmarks para tarefas comuns de análise de séries temporais e cálculo de features em múltiplos ativos, ele demonstra ser de **10x a 50x mais rápido** que o Pandas, além de ter um uso de memória mais eficiente. Sua API "lazy" (preguiçosa) permite a otimização de queries complexas.

2.  **Coleta de Dados via WebSocket e Candle Builder Local:**
    *   **Decisão:** Em vez de fazer polling (consultas periódicas) da API REST para obter novos candles, o sistema priorizará a conexão a um **stream WebSocket** de trades da exchange e construirá os candles (OHLCV) localmente.
    *   **Justificativa:** Esta abordagem **reduz drasticamente o uso de rate limits** da API, pois uma única conexão WebSocket substitui centenas ou milhares de chamadas REST. Além disso, fornece dados com menor latência e maior granularidade, permitindo a construção de timeframes customizados.

3.  **Cache de Features em Memória com TTL:**
    *   **Decisão:** Features que são computacionalmente caras e possuem lookbacks longos (ex: retorno de 252 dias, volatilidade anualizada) serão cacheadas em memória com um Time-To-Live (TTL).
    *   **Justificativa:** Recalcular um retorno de 252 dias a cada novo candle de 1 minuto é extremamente ineficiente. Um cache que armazena o valor calculado e o invalida apenas quando necessário (ex: a cada hora ou a cada novo dia) reduz significativamente a carga de CPU do core loop, liberando recursos para outras tarefas.

**Consequências:**
*   **Positivas:**
    *   Performance geral do sistema significativamente melhorada.
    *   Redução drástica no consumo de rate limits da exchange, tornando o sistema mais robusto e escalável para mais ativos.
    *   Menor latência na detecção de novos eventos de mercado.
    *   Uso mais eficiente de CPU e memória.
*   **Negativas:**
    *   A implementação do "candle builder" local a partir de um stream de trades adiciona complexidade à camada de dados.
    *   A gestão de cache (invalidação, consistência) introduz uma nova classe de possíveis bugs se não for bem implementada.
    *   Adoção de Polars requer adaptação do código e da equipe, que pode estar mais familiarizada com Pandas.

---

## ADR-014: Estratégia de Resiliência e Recuperação de Estado

**Contexto:** O modelo single-node (ADR-002) introduz um Ponto Único de Falha (SPOF). A perda ou corrupção do estado local (SQLite DB) em caso de falha de hardware ou crash de software é o maior risco operacional do sistema.

**Decisões:**

1.  **Replicação de Estado com Litestream:**
    *   **Decisão:** O banco de dados SQLite será replicado em tempo real para um armazenamento de objetos compatível com S3 (ex: AWS S3, MinIO) usando o **Litestream**.
    *   **Justificativa:** O Litestream captura as escritas do SQLite a nível de página e as envia para um local remoto de forma contínua. Isso resolve 90% do risco de perda de estado por falha de disco ou corrupção do nó principal. A recuperação se torna uma questão de baixar o último snapshot do S3 e aplicar o WAL replicado.

2.  **Durabilidade do SQLite via WAL e Configurações:**
    *   **Decisão:** O SQLite será configurado para operar em modo **WAL (Write-Ahead Logging)** com `PRAGMA synchronous=FULL` e checkpoints frequentes.
    *   **Justificativa:** O modo WAL melhora a concorrência e a performance de leitura, enquanto `synchronous=FULL` garante que as transações sejam completamente escritas em disco antes de serem confirmadas, garantindo durabilidade mesmo em caso de queda de energia ou crash do sistema operacional.

3.  **Failover Hot-Warm (Opcional, Fase Avançada):**
    *   **Decisão:** Para sistemas que exigem alta disponibilidade, será implementado um padrão de failover **Hot-Warm**. Um segundo nó (Warm) idêntico é mantido sincronizado com o nó principal (Hot) via Litestream e `rsync`. Ferramentas como `keepalived` gerenciam um IP virtual que aponta para o nó ativo.
    *   **Justificativa:** Se o nó Hot falhar, o `keepalived` automaticamente move o IP virtual para o nó Warm, que assume as operações. Isso reduz o RTO (Recovery Time Objective) para menos de 2 minutos, transformando um evento de desastre em um breve período de indisponibilidade.

**Consequências:**
*   **Positivas:**
    *   Aumento drástico na resiliência do sistema. O risco de perda de estado se torna próximo de zero.
    *   O RPO (Recovery Point Objective) é reduzido para segundos.
    *   O RTO (Recovery Time Objective) é drasticamente reduzido, especialmente com o failover Hot-Warm.
    *   Mantém a simplicidade da filosofia single-node (apenas um nó ativo por vez), mas com a segurança de um sistema distribuído.
*   **Negativas:**
    *   Adiciona dependências externas: Litestream e um bucket S3.
    *   A configuração do failover Hot-Warm (keepalived, rsync) adiciona complexidade operacional ao setup inicial.

---

## ADR-016: Estratégia de Gerenciamento de Rate Limits em Múltiplas Camadas

**Contexto:** O sistema interage intensamente com as APIs da exchange, que impõem limites estritos de requisições (rate limits). Exceder esses limites pode resultar em bloqueios temporários ou até mesmo bans, paralisando as operações. É necessária uma estratégia robusta e explícita para gerenciar o consumo de API.

**Decisão:** Será implementada uma estratégia de defesa em três camadas para o gerenciamento de rate limits:

1.  **Camada 1: Centralização e Deduplicação de Requisições.**
    *   Nenhuma estratégia ou componente acessa a API da exchange diretamente. Todas as requisições de dados são canalizadas através de um `DataOrchestrator` central.
    *   Este orquestrador é responsável por agregar, deduplicar e agrupar requisições. Se múltiplas estratégias solicitam o mesmo dado (ex: candle de `BTC/USDT` em `1h`), apenas uma chamada à API é feita, e o resultado é distribuído para todos os solicitantes. Isso elimina a principal fonte de desperdício de rate limits.

2.  **Camada 2: Controle Proativo com Token Bucket.**
    *   O `ExchangeAdapter`, componente que se comunica com a API, implementará o algoritmo **Token Bucket** para cada classe de endpoint da exchange (ex: `klines`, `order`, `account`).
    *   Cada bucket é configurado com a capacidade e a taxa de preenchimento correspondentes aos limites oficiais da exchange. Antes de fazer uma chamada, o adapter consome um "token" do bucket correspondente. Se não houver tokens, a chamada é adiada até que um token esteja disponível. Isso impede proativamente que o sistema exceda os limites.

3.  **Camada 3: Reação a Falhas com Backoff Exponencial.**
    *   Como uma camada final de segurança, se o sistema receber um erro de rate limit da exchange (ex: HTTP `429` ou `418`), um mecanismo de **backoff exponencial** será ativado para a classe de endpoint afetada.
    *   As requisições para aquele endpoint serão pausadas, e as novas tentativas ocorrerão em intervalos crescentes (ex: 1s, 2s, 4s, 8s...), dando tempo para a API se recuperar e evitando agravar o problema.

**Justificativa:**
*   **Robustez:** A combinação de uma defesa proativa (Token Bucket) com uma reativa (Backoff Exponencial) e uma otimização na fonte (Centralização) cria um sistema altamente resiliente a variações de carga e instabilidades da API.
*   **Eficiência:** A centralização e deduplicação garantem o uso mais eficiente possível dos limites de API disponíveis, permitindo que o sistema escale para mais estratégias e ativos sem esgotar os recursos.
*   **Observabilidade:** Esta arquitetura permite a criação de métricas detalhadas sobre o consumo de API por endpoint, a utilização de cada token bucket e a frequência de acionamento do backoff, facilitando o diagnóstico de gargalos.
*   **Segurança Operacional:** Evita o risco de uma única estratégia "bugada" ou excessivamente agressiva derrubar o acesso à API para todo o sistema.

**Consequências:**
*   **Positivas:**
    *   Redução drástica do risco de banimento por parte da exchange.
    *   Operação mais estável e previsível.
    *   Melhor escalabilidade em termos de número de estratégias e ativos monitorados.
*   **Negativas:**
    *   A implementação do `DataOrchestrator` e do `TokenBucketManager` adiciona complexidade à camada de infraestrutura de dados e execução.
    *   A configuração dos buckets requer um mapeamento cuidadoso dos limites documentados pela exchange para cada endpoint.

---

## ADR-017: Estratégia de Testes Automatizados em Pirâmide

**Contexto:** Um sistema de trading algorítmico é, por natureza, complexo e de alto risco. Bugs podem levar a perdas financeiras diretas. É crucial ter uma estratégia de testes automatizados que seja robusta, abrangente e que equilibre velocidade de feedback com fidelidade ao ambiente de produção.

**Decisão:** O projeto adotará uma estratégia de testes automatizados baseada no modelo clássico de **Pirâmide de Testes**, com três camadas distintas e critérios de qualidade claros para cada uma.

1.  **Base da Pirâmide: Testes Unitários (`pytest`)**
    *   **Escopo:** Testar a menor unidade de código (funções, métodos de classes) de forma isolada.
    *   **Foco:** Lógica de domínio pura, como cálculos de indicadores, regras de geração de sinal de uma estratégia, funções de utilidade e validações de configuração. Todas as dependências externas (APIs, banco de dados, rede) são substituídas por mocks.
    *   **Critério Mínimo:** Cobertura de testes de no mínimo **80%** para todos os módulos críticos de domínio (estratégias, risco). Pull Requests devem manter ou aumentar a cobertura de testes.

2.  **Meio da Pirâmide: Testes de Integração**
    *   **Escopo:** Testar a interação entre dois ou mais componentes do sistema.
    *   **Foco:**
        *   **Adapters vs. Infraestrutura Real (ou Testnet):** Validar que o `BinanceSpotAdapter` consegue se comunicar corretamente com a **Testnet da Binance**, cobrindo autenticação, envio de ordens e tratamento de erros.
        *   **Fluxo de Decisão Interno:** Validar a interação entre `Strategy` -> `RiskModule` -> `StateManager` para garantir que os sinais são corretamente filtrados e o estado é atualizado.
        *   **Persistência:** Validar que o `StateManager` consegue salvar e carregar o estado corretamente usando o `SQLiteAdapter`.
    *   **Critério Mínimo:** Existência de um conjunto de testes de integração que cubra o "caminho feliz" e os principais cenários de erro para cada `Adapter` de infraestrutura.

3.  **Topo da Pirâmide: Testes End-to-End (Backtests de Regressão)**
    *   **Escopo:** Validar o sistema completo, de ponta a ponta, em um ambiente que simula a produção.
    *   **Foco:** Utilizar o **engine de backtest isomórfico** para executar um conjunto de **"Backtests de Referência"** (ou "Golden Backtests").
    *   **Implementação:**
        1.  Um conjunto de backtests é definido com estratégias, parâmetros e períodos históricos fixos.
        2.  Os resultados-chave (P&L final, drawdown máximo, número de trades) são salvos como um "snapshot de ouro".
        3.  O pipeline de CI/CD executa esses backtests a cada mudança no código de domínio ou risco. O teste falha se os novos resultados divergirem do snapshot além de uma tolerância mínima, indicando uma regressão.
    *   **Critério Mínimo:** Existência de pelo menos um backtest de regressão para cada família de estratégia principal (TSM, Dual Momentum). Pull Requests que alterem a lógica de domínio ou risco devem passar nesses testes.
    *   **Otimização de CI/CD com Cache de Artefatos:** Para mitigar a lentidão no pipeline, será implementado um mecanismo de cache inteligente. Antes de executar um "Golden Backtest", o pipeline de CI/CD calculará um hash baseado no código-fonte da estratégia, seus parâmetros e o dataset histórico utilizado. Se um artefato de resultado com esse mesmo hash já existir em um armazenamento (cache), a execução do backtest será pulada (`skipped`), e o resultado em cache será usado para a validação. Isso acelera drasticamente o feedback para o desenvolvedor, sem comprometer o rigor do teste.

**Justificativa:**
*   **Segurança e Confiança:** A combinação de testes em diferentes níveis de granularidade aumenta drasticamente a confiança de que o sistema se comportará como esperado em produção.
*   **Feedback Rápido:** Testes unitários são rápidos e rodam a cada commit, fornecendo feedback imediato ao desenvolvedor.
*   **Detecção de Regressões:** Os backtests de regressão são a principal defesa contra mudanças que quebram sutilmente a lógica de uma estratégia, algo difícil de pegar com testes unitários.
*   **Clareza e Manutenibilidade:** Uma estratégia de testes bem definida torna o código mais fácil de manter e refatorar, pois os testes servem como uma rede de segurança.

**Consequências:**
*   **Positivas:**
    *   Redução significativa do risco de bugs em produção.
    *   Aumento da qualidade e da manutenibilidade do código.
    *   Processo de desenvolvimento mais disciplinado e seguro.
*   **Negativas:**
    *   Maior investimento de tempo na escrita e manutenção dos testes.
    *   O pipeline de CI/CD se torna mais lento devido à execução dos backtests de regressão, exigindo otimização e, possivelmente, a execução seletiva de testes.

---

## ADR-018: Gestão de Versões de Configuração e Rollback em Tempo de Execução

**Contexto:** O sistema depende de arquivos de configuração para definir o comportamento das estratégias (parâmetros, limites de risco). É crucial ter um método para versionar essas configurações e, mais importante, para reverter rapidamente uma configuração problemática em produção sem a necessidade de um re-deploy completo, que é um processo lento.

**Decisão:** Será implementada uma estratégia híbrida para gestão e rollback de configurações.

1.  **Modelo de Armazenamento Híbrido:**
    *   **Git como Fonte Única da Verdade (SSOT):** Todos os arquivos de configuração (`.yml`) são armazenados e versionados no Git. As mudanças seguem o fluxo de Pull Request, garantindo revisão e auditabilidade. A `config_version` é um campo dentro do próprio arquivo.
    *   **SQLite como Registro Operacional:** Na inicialização, o sistema carrega todos os arquivos de configuração do disco, os valida e os armazena em uma tabela `execution_configs` no banco de dados SQLite. Esta tabela atua como um registro histórico de todas as configurações já validadas, contendo o conteúdo da configuração, `strategy_id`, `config_version` e um timestamp.

2.  **Mecanismo de Rollback em Tempo de Execução ("Hot Swap"):**
    *   O sistema implementará um mecanismo de governança (ex: via comando de um bot no Telegram) para permitir o rollback de uma configuração em tempo de execução.
    *   **Fluxo de Rollback:**
        1.  Um operador envia um comando, como `/rollback_config <strategy_id> <target_version>`.
        2.  O sistema consulta a tabela `execution_configs` no SQLite para encontrar a configuração histórica solicitada.
        3.  Se a configuração for encontrada e válida, o sistema atualiza seu estado em memória para marcar aquela versão como a **ativa** para a `strategy_id` especificada.
        4.  No próximo ciclo de execução, o `core loop` utilizará a nova configuração ativa, revertendo o comportamento da estratégia de forma quase instantânea.

**Justificativa:**
*   **Segurança e Agilidade:** Combina a segurança e o processo de revisão do Git com a agilidade de um rollback em tempo de execução. O tempo para corrigir uma configuração de risco inadequada (Mean Time To Recover - MTTR) é reduzido de minutos (para um re-deploy) para segundos.
*   **Auditabilidade Completa:** O Git mantém o histórico de "quem" e "por que" mudou a configuração. O SQLite mantém o registro de "quais" configurações foram efetivamente carregadas e executadas pelo sistema.
*   **Desacoplamento:** Desacopla o ciclo de vida das configurações do ciclo de vida do deploy da aplicação. Mudanças de parâmetros não exigem, por padrão, uma nova build ou restart do sistema.
*   **Resiliência:** O sistema sempre tem acesso a um histórico de configurações válidas, podendo reverter para um estado seguro conhecido mesmo que o acesso ao Git esteja indisponível no momento do incidente.

**Consequências:**
*   **Positivas:**
    *   Redução drástica do risco associado a configurações incorretas.
    *   Operação mais flexível e responsiva.
    *   Processo de governança claro e robusto.
*   **Negativas:**
        *   A lógica de inicialização do sistema se torna mais complexa, pois precisa popular e gerenciar a tabela de registro de configurações.
        *   Requer a implementação de uma interface de governança segura (ex: bot com autenticação) para executar os comandos de ativação e rollback.
        *   A criticidade do banco de dados SQLite aumenta, reforçando a importância da estratégia de replicação e backup (ADR-014).
    
    ---
    
    ## ADR-019: Política de Seleção de Tipo de Ordem
    
    **Contexto:** O sistema precisa decidir entre usar ordens `MARKET` (execução garantida, custo incerto) e `LIMIT` (custo garantido, execução incerta). Deixar essa decisão para cada estratégia individualmente criaria inconsistência e aumentaria o risco de implementações ingênuas.
    
    **Decisão:**
    1.  O **Módulo de Execução**, e não as estratégias, será responsável por selecionar o tipo de ordem.
    2.  A seleção será baseada em uma **matriz de decisão de "Política de Ordem"** configurável, que considera a **intenção** do trade.
    3.  As estratégias emitem sinais com uma `urgency` (urgência), e o Módulo de Execução mapeia essa intenção para um tipo de ordem concreto.
    
    **Justificativa:**
    *   **Centralização da Lógica de Execução:** Garante que a lógica de microestrutura e de otimização de custos resida em um único local, facilitando a manutenção e a melhoria.
    *   **Consistência:** Todas as estratégias se beneficiam da mesma política de execução robusta, evitando que uma estratégia mal implementada cause custos excessivos por mau uso de ordens a mercado.
    *   **Desacoplamento:** As estratégias podem se concentrar em gerar sinais de alfa, enquanto o Módulo de Execução se especializa em executar esses sinais da forma mais eficiente e segura possível.
    
    ### Matriz de Política de Ordem (Exemplo Padrão)
    
    | Intenção da Ordem | Urgência | Tipo de Ordem Padrão | Fallback (se não executado) |
    | :--- | :--- | :--- | :--- |
    | **Entrada/Aumento de Posição** | Normal | `LIMIT` (com `post-only` se disponível) | N/A (ordem pode expirar) |
    | **Rebalanceamento Normal** | Normal | `LIMIT` | N/A (ordem pode expirar) |
    | **Realização de Lucro (Take Profit)**| Normal | `LIMIT` | N/A (ordem pode expirar) |
    | **Stop Loss** | Emergência | `MARKET` | N/A |
    | **Acionamento de Circuit Breaker** | Emergência | `MARKET` | N/A |
    | **Saída por Delisting de Ativo** | Emergência | `MARKET` | N/A |
    
    **Configuração e Overrides:**
    *   A política padrão será definida globalmente.
    *   Será possível, via configuração, que famílias de estratégias mais agressivas ou específicas (ex: Market Making) solicitem políticas de ordem customizadas, mas isso será a exceção, não a regra.
    *   A lógica de fallback (ex: "se a ordem LIMIT não for executada em N segundos, cancele e envie uma ordem MARKET com 10% do tamanho") também será parte deste módulo.
    
    **Consequências:**
    *   **Positivas:** Redução do slippage e dos custos de transação, comportamento de execução mais previsível e seguro, maior desacoplamento entre alfa e execução.
    *   **Negativas:** Aumenta a complexidade do Módulo de Execução, que agora precisa conter essa lógica de política de ordem.

---

## ADR-020: Gestão de Provedores de Dados Externos

**Contexto:** Certas famílias de estratégias, como `Value + Quality` e `Anti-Fragile`, dependem de dados que não são fornecidos pela API da exchange (ex: dados fundamentais de empresas, superfícies de volatilidade implícita, taxas de juros). É preciso definir uma arquitetura para integrar essas fontes sem acoplar o domínio a implementações específicas e gerenciando os riscos associados.

**Decisão:**
1.  **Ports Dedicados para Dados Externos:** Para cada tipo de dado externo, será criada uma "porta" (interface) específica no domínio. Exemplos: `FundamentalDataPort`, `ImpliedVolatilityPort`.
2.  **Adapters por Provedor:** A implementação concreta para cada provedor de dados (ex: um provedor de dados fundamentais X, ou Deribit para volatilidade implícita) será encapsulada em um `Adapter` que implementa a porta correspondente.
3.  **Cache e Política de Degradação:** O sistema implementará uma camada de cache agressiva para esses dados, que são geralmente de baixa frequência. Se um provedor externo falhar, o sistema deve ter uma política de degradação clara.
4.  **Associação de Custo:** A decisão de habilitar uma família que dependa de um provedor de dados pago deve ser explícita e considerar o custo-benefício (custo da assinatura vs. alfa esperado ou benefício de diversificação).

**Justificativa:**
*   **Desacoplamento:** Mantém o domínio agnóstico em relação a qual provedor de dados está sendo usado, permitindo a troca de provedores sem alterar a lógica das estratégias.
*   **Resiliência:** O cache e a política de degradação impedem que uma falha em um serviço de terceiros não-crítico paralise todo o sistema. As estratégias afetadas podem ser pausadas individualmente.
*   **Consciência de Custo:** Força uma análise explícita dos custos operacionais, alinhado ao princípio de `Cost-Aware` (ADR-001).

### Política de Degradação

*   **Estado `STALE`:** Se um `Adapter` de dados externos não consegue atualizar suas informações dentro de um TTL configurado (ex: 24 horas para dados fundamentais), os dados em cache são marcados como "stale" (obsoletos).
*   **Ação:** Estratégias que dependem dessa porta de dados recebem a informação com a flag `stale`. A política padrão da estratégia deve ser a de **não gerar novos sinais de entrada** com dados obsoletos, mantendo apenas a gestão das posições existentes. Um alerta de nível `WARNING` é gerado.
*   **Ação Crítica:** Se os dados permanecerem obsoletos por um período crítico (ex: 72 horas), o alerta escala para `CRITICAL`, e o módulo de risco pode ser configurado para forçar a saída das posições afetadas.

**Consequências:**
*   **Positivas:** Permite a expansão do framework para estratégias mais sofisticadas de forma controlada e segura, sem comprometer a robustez do core system.
*   **Negativas:** Adiciona complexidade à camada de dados (mais adapters, lógica de cache/TTL, monitoramento de serviços de terceiros). O custo operacional do sistema pode aumentar se provedores pagos forem utilizados.    