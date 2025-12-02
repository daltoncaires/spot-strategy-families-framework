
## 6. Família 1 – Trend Following / Time-Series Momentum (TSM)

A família **Trend Following / Time-Series Momentum (TSM)** é o pilar principal e a primeira a ser implementada no framework.

---

### 6.1. Descrição Conceitual

A estratégia de Time-Series Momentum (TSM) opera com base no retorno acumulado de um ativo em uma janela de lookback (ex: 3-12 meses). Se o retorno for positivo acima de um limiar, a estratégia busca uma posição comprada. Caso contrário, a posição é zerada (long/flat). A lógica pode ser aplicada a um ou múltiplos ativos.

---

### 6.3. Implementação

1. **Sinal de Tendência Baseado em Retorno Acumulado**

Para cada ativo `A` e timeframe principal `T` (por exemplo, diário):

* calcula-se o retorno acumulado em uma janela de lookback `L` (ex.: 90, 180, 252 candles):

  * pode ser retorno simples, log-return acumulado ou combinação de retornos diários;
* define-se um limiar `threshold` (por exemplo, 0% ou levemente positivo).

Regra básica:

* se `retorno_L > threshold` → sinal de **tendência de alta** → posição alvo positiva (ex.: 100% do capital designado para aquele ativo);
* se `retorno_L ≤ threshold` → **sem posição** naquele ativo (cash/stable).

2. **Posição Long/Flat e Interação com Risco**

A estratégia opera em modo long/flat, zerando a posição quando o sinal de tendência é negativo. Ela gera um percentual de alocação alvo, que é convertido em tamanho de posição pelo módulo de risco, respeitando todos os limites hierárquicos.

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

A família **Dual Momentum** combina Time-Series Momentum (TSM) e Cross-Sectional Momentum (CSM). Ela filtra ativos com tendência positiva (TSM) e, em seguida, ranqueia esse subconjunto para selecionar os de melhor desempenho relativo (CSM), sendo adequada para portfólios multi-ativos em spot.

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

### 7.3. Implementação

A implementação é **long-only com cash como contraparte**. Ativos "perdedores" não são vendidos a descoberto, mas simplesmente não recebem alocação. Quando poucos ativos passam no filtro TSM, o portfólio pode permanecer majoritariamente em cash/stablecoin.

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

A família **Cross-Sectional Momentum (CSM)** explora o comportamento relativo entre ativos de um mesmo universo, ranqueando-os pelo retorno passado e comprando os "vencedores". No contexto do projeto, a família é adaptada para o ambiente **spot long-only / long-flat**.

---

### 8.1. Descrição Conceitual

A estratégia funciona como um ranking: para um universo de ativos, calcula-se o retorno passado (ex: 3-12 meses), ordenam-se os ativos, e compram-se os "vencedores" (top 10-20%). Na adaptação long-only, os perdedores são ignorados (posição em cash/stable), sem venda a descoberto.

---

### 8.3. Implementação

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

A família **Carry / Yield / Basis** explora retornos de diferenças de juros (carry), pagamentos recorrentes (yield) ou deslocamentos entre mercados (basis). É uma família complementar às estratégias direcionais.

---

### 9.1. Descrição Conceitual

A família engloba três pilares: **Carry** (diferencial de juros), **Yield** (rendimentos recorrentes como staking) e **Basis/Funding** (diferença entre preço spot e de derivativos). A exploração de basis/funding é considerada um módulo avançado e opcional.

---

### 9.3. Implementação

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

A família **Value + Quality** é um pilar de médio/longo prazo, focado em capturar prêmios de valor (ativos "baratos") e qualidade (empresas rentáveis e estáveis), operando principalmente em ações spot long-only.

---

### 10.1. Descrição Conceitual

A estratégia combina fatores de **Value** (preço baixo em relação a métricas como P/L, P/B) e **Quality** (alta rentabilidade, baixa alavancagem). A família opera em modo long-only, com baixa frequência de rebalanceamento (trimestral a anual), servindo como uma âncora de longo prazo no portfólio.

---

### 10.3. Implementação

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

A família **Mean Reversion de Curto Prazo (Intraday / Swing)** busca capturar reversões de movimentos excessivos de curto prazo. Em contraste com o momentum, ela aposta em correções parciais.

---

### 11.1. Descrição Conceitual

A estratégia busca comprar em condições de "oversold" de curto prazo e reduzir exposição em "overbought", operando em modo spot long/flat. Pode ser aplicada em timeframes intraday (minutos, horas) ou swing (dias).

---

### 11.3. Implementação

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

A família **Market Making / Liquidity Providing** busca capturar o spread entre compra e venda. É uma estratégia avançada, dependente de microestrutura e infraestrutura de execução, e aparece no final do roadmap de implementação.

---

### 12.1. Descrição Conceitual

O market maker coloca ordens passivas de compra e venda para lucrar com o spread, gerenciando seu inventário para evitar exposição direcional excessiva. A estratégia opera em spot, com foco em pares líquidos e timeframes curtos, mas não HFT.

---

### 12.3. Implementação

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