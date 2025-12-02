**Famílias de estratégias** com histórico mais robusto de retorno **ajustado ao risco** – *ordem aproximada* pensando em um algotrader para mercado spot (ações, FX ou cripto) - base em literatura acadêmica + prática de fundos quantitativos.

Organizado por:

* Foco em **estratégias sistemáticas/documentadas**
* Adaptação para **spot long/flat** (sem precisar de derivativos, ou só opcional)
* Ordem aproximada de “robustez de evidência + retorno histórico”

---

## 1. Trend Following / Time-Series Momentum (TSM)

**Ideia:** se o ativo teve retorno positivo na janela recente (ex.: 3–12 meses), tende a continuar subindo; se teve retorno negativo, tende a cair. Em spot long/flat você normalmente faz:

* Compra se `retorno_lookback > 0`
* Fica em *cash* se `retorno_lookback < 0`

**Evidência:**

* Estudos clássicos mostram que TSM é lucrativo em índices de ações, moedas, commodities e bonds por décadas, gerando retornos anormais com baixa correlação com fatores tradicionais. ([ScienceDirect][1])
* Há evidência recente de TSM em ações europeias 2000–2020, com anomalia persistente. ([PMC][2])
* Em cripto, trabalhos como “Catching Crypto Trends” mostram que trend following clássico aplicado a BTC e cestas de criptos continua funcionando (com cuidado com custos). ([SSRN][3])

**Por que fica no topo?**

* Funciona em muitos mercados e períodos
* Dá pra fazer em **spot** só indo de **100% cash → comprado**
* Normalmente tem bom Sharpe e protege relativamente bem em crises (porque tende a estar fora em grandes quedas)

---

## 2. Dual Momentum (TSM + Cross-Sectional)

**Ideia:** combinar:

* **Time-series momentum:** só opera ativos com tendência a favor
* **Cross-sectional momentum:** entre os que estão “ok”, escolhe os que tiveram maiores retornos relativos

Estudos de “dual momentum” mostram retornos mensais bem elevados em ações dos EUA e internacionais, ao combinar os dois tipos de momentum. ([SSRN][4])

**Em spot:**
Você pode fazer:

* Universo de ativos spot (ações, pares cripto, etc.)
* Filtra só os que têm retorno 6–12m > 0 (TSM)
* Dentro deles, pega o top X% por retorno (cross-sectional) para ficar comprado

---

## 3. Cross-Sectional Momentum (relativo entre ativos)

**Ideia:** ranquear ativos pelo retorno passado (3–12 meses):

* Long nos “vencedores” (top 10–20%)
* Em versão neutra seria long vencedores / short perdedores, mas em **spot** você fica só no lado long ou long + cash.

A literatura mostra **retornos médios altos**, mas com **grandes drawdowns**, inclusive episódios de “momentum crashes”. ([Alpha Architect][5])

**Por que fica abaixo de TSM?**

* Tende a ter drawdowns bem mais violentos
* Em versão apenas long, você perde parte da edge (sem short dos perdedores)

---

## 4. Carry / Yield / Basis (quando acessível a partir do spot)

Nem sempre é “puro spot”, mas vale citar porque costuma ser muito lucrativo:

* Em FX: comprar moedas de juros altos e vender de juros baixos (carry trade)
* Em cripto: comprar spot e vender futuro perp/quanto há funding positivo (basis/funding arbitrage)

Essas estratégias tendem a ter retornos médios altos, mas comportamento de “ganha devagar, perde rápido” em crises. ([scielo.org.pe][6])

No **spot puro**, você às vezes consegue parte do *carry* via:

* Staking / yields on-chain
* Selecionar ativos com “yield real” (dividendos, staking, etc.)

---

## 5. Value + Quality (mais para ações spot)

Em equities spot, estratégias fatoriais de:

* **Value** (P/B, P/E baixos, etc.)
* **Quality** (ROE/ROA altos, baixa alavancagem, margens estáveis)

historicamente geraram retornos anormais de longo prazo em várias bolsas – embora a discussão sobre se é “prêmio de risco” ou “ineficiência” continue viva. Elas tendem a ter **volatilidade menor**, mas com períodos longos de underperformance.

Em cripto isso é bem menos estabelecido (a própria definição de “valor intrínseco” é nebulosa), então eu colocaria isso como prioridade mais baixa para um algotrader focado em cripto spot.

---

## 6. Mean Reversion de Curto Prazo (intraday / swing)

**Ideia:** comprar quedas rápidas exageradas e vender spikes de alta no curto prazo, muitas vezes usando bandas, RSI, microestrutura, etc.

* Em ações *large cap*, há bastante evidência de reversão de curtíssimo prazo (1–5 dias) em certas condições
* Em cripto, alguns backtests mostram que funciona em períodos/pares específicos, mas com forte risco de *regime change* ([Quantified Strategies][7])

Essa família é:

* Potencialmente muito lucrativa
* Mas **mais frágil** a custos, slippage e mudanças de microestrutura

Por isso eu colocaria **abaixo** de momentum e dual momentum em termos de robustez.

---

## 7. Market Making / Liquidity Providing

Em teoria, boa parte dos lucros estruturais em mercados vem de:

* Prover liquidez (capturar *spread*)
* Ganhar com rebates
* Rebalancear inventário com lógica de microestrutura

Em HFT isso pode ter **Sharpe altíssimo**, mas:

* Você compete com players profissionais
* Precisa de latência baixa, infra pesada e muito cuidado com caudas gordas

Para um **algotrader “de varejo” em spot**, eu não colocaria como primeira prioridade, embora o potencial bruto possa ser enorme se bem feito.

---

## Colocando tudo em uma “ordem prática” para você

Se eu tivesse que sugerir uma **ordem de implementação/estudo para mercado spot**, pensando em **retorno financeiro histórico + robustez + viabilidade prática**, eu faria:

1. **Trend Following / Time-Series Momentum (long/flat)**
2. **Dual Momentum (TSM + cross-sectional, long-only)**
3. **Cross-Sectional Momentum (long-only)**
4. **Carry/Yield/Basis onde der pra extrair via spot + derivativos básicos**
5. **Value + Quality (se envolver ações spot)**
6. **Mean Reversion de curto prazo (intraday/swing)**
7. **Market Making estruturado (se você tiver infra adequada)**

---

## Como isso conversa com o seu algotrader (tipo AlgoOrchestrator)

Pra um sistema como o que você está montando, uma rota interessante seria:

1. Implementar **um módulo genérico de TSM / trend following em spot**
2. Estender pra **dual momentum** (camada cross-sectional em cima)
3. Adicionar **módulos de mean reversion intraday** como estratégias complementares
4. Só depois mexer com carry/basis e MM, se você tiver infra/exchange compatíveis

Se quiser, na próxima mensagem eu posso montar um **esqueleto de parâmetros e sinais** (em pseudo-código/Python ou Pine) para uma estratégia de **TSM “clássica” em spot** já pensada para encaixar no seu orquestrador.

[1]: https://www.sciencedirect.com/science/article/pii/S0304405X11002613?utm_source=chatgpt.com "Time series momentum"
[2]: https://pmc.ncbi.nlm.nih.gov/articles/PMC9879792/?utm_source=chatgpt.com "Time series momentum: Evidence from the European ..."
[3]: https://papers.ssrn.com/sol3/Delivery.cfm/5209907.pdf?abstractid=5209907&mirid=1&utm_source=chatgpt.com "Catching Crypto Trends - A Tactical Approach for Bitcoin ..."
[4]: https://papers.ssrn.com/sol3/Delivery.cfm/SSRN_ID2720600_code586501.pdf?abstractid=2720600&utm_source=chatgpt.com "The Enduring Effect of Time-Series Momentum on Stock ..."
[5]: https://alphaarchitect.com/cross-sectional-momentum/?utm_source=chatgpt.com "Minimizing the Risk of Cross-Sectional Momentum Crashes -"
[6]: https://www.scielo.org.pe/scielo.php?pid=S2077-18862022000200328&script=sci_arttext&utm_source=chatgpt.com "Risk-managed time-series momentum: an emerging ..."
[7]: https://www.quantifiedstrategies.com/backtesting-crypto-trading-strategies-what-works-in-volatile-markets/?utm_source=chatgpt.com "Backtesting Crypto Trading Strategies: What Works in ..."
