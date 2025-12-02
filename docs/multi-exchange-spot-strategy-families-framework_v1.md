# Framework de Famílias de Estratégias Spot Multi-Exchange

Segue um esqueleto novo, já unificado, ordenado e com gaps preenchidos para o projeto das **famílias de estratégias com foco em retorno ajustado ao risco**, em um **sistema single node**, com **arquitetura multi-exchange** (inicialmente só **Binance Spot**).

A ideia é: **agora** só ter os tópicos/subtópicos. Depois você vai “preencher” cada seção.

---

# Projeto: Framework de Famílias de Estratégias Spot Multi-Exchange

## 1. Visão Geral do Projeto

1.1. Contexto
1.2. Objetivo Principal

* Identificar e implementar famílias de estratégias com histórico robusto de **retorno ajustado ao risco** em mercado **spot long/flat**
  1.3. Escopo Inicial
* Single node
* Uma exchange (Binance Spot)
* Suporte arquitetural pronto para multi-exchange
  1.4. Não-Escopo (fora do MVP)
* Derivativos complexos, HFT extremo, colocation etc.
  1.5. Stakeholders e Perfis de Usuário
  1.6. Principais Premissas e Restrições

---

## 2. Requisitos e Premissas do Sistema

2.1. Requisitos Funcionais

* Execução de estratégias sistemáticas/documentadas
* Suporte a múltiplas famílias de estratégias
* Backtest, paper trading e live trading
* Gestão de risco por estratégia e por portfólio
  2.2. Requisitos Não Funcionais
* Single node (CPU/RAM/SSD definidos)
* Resiliência (reinício, quedas de rede, falhas de exchange)
* Segurança (credenciais, API keys, permissões)
* Performance mínima (latência aceitável para spot)
  2.3. Premissas Operacionais
* Ambiente Linux como alvo principal
* Janela de operação 24/7 para cripto
  2.4. Critérios de Sucesso (KPIs de Projeto)
* % de uptime
* Cobertura de testes
* Quantidade de famílias de estratégias validadas

---

## 3. Arquitetura Geral

3.1. Visão de Alto Nível

* Single node com serviços lógicos desacoplados (não necessariamente processos separados)
  3.2. Estilo Arquitetural
* Hexagonal / Ports & Adapters
* Clean Architecture
* DDD (Domínio ≠ Infraestrutura)
  3.3. Componentes Principais
* Módulo de Orquestração / Core Loop
* Módulo de Estratégias
* Módulo de Dados de Mercado (Data Feeds)
* Módulo de Execução de Ordens
* Módulo de Risco e Alocação de Capital
* Módulo de Backtest & Research
* Módulo de Observabilidade (logs, métricas, alertas)
* Módulo de Governança & Configuração
  3.4. Fluxo Geral (Core Loop)
* Coleta de dados → Enriquecimento (indicadores) → Sinais de estratégia → Filtro de risco → Geração de ordens → Execução → Persistência → Observabilidade
  3.5. Estratégia de Escalabilidade e Evolução
* De single exchange → multi-exchange
* De single node → futura compatibilidade com cluster (não no MVP)

---

## 4. Universo de Mercados, Exchanges e Dados

4.1. Exchanges Suportadas

* Versão 1: Binance Spot
* Versão futura: outras exchanges (Kraken, Coinbase, etc.)
  4.2. Abstração Multi-Exchange
* Porta genérica de Exchange
* Adapters específicos por exchange
  4.3. Classes de Ativos e Símbolos
* Cripto pares spot (BTCUSDT, ETHUSDT, etc.)
  4.4. Timeframes Suportados
* Intraday (1m, 5m, 15m, 1h)
* Swing (4h, 1D)
  4.5. Fontes de Dados
* WebSocket (tempo real)
* REST (histórico, falhas do socket)
  4.6. Normalização, Qualidade e Validação de Dados
* Normalização de OHLCV
* Tratamento de buracos de dados
* Verificações de consistência
  4.7. Persistência de Dados
* Banco local (SQLite, Parquet etc.)
* Política de retenção e rotação

---

## 5. Framework de Estratégias e Famílias

5.1. Princípios

* Estratégias **sistemáticas e documentadas**
* Preferência por evidência acadêmica/empírica robusta
* Foco em **spot long/flat**
  5.2. Interface de Estratégia (Strategy API)
* Input: DataFrame/stream enriquecido
* Output: sinais (`BUY`, `SELL`, `FLAT`, posição alvo, tamanho)
  5.3. Ciclo de Vida de uma Estratégia
* Configuração → Backtest → Validação → Paper trading → Produção → Monitoramento → Retirada/ajuste
  5.4. Modelo de Configuração
* YAML/JSON + Pydantic (parâmetros, ativos, timeframes)
  5.5. Integração com Multi-Exchange
* Estratégia independente de exchange
* Apenas “constraints” de instrumentação específicas
  5.6. Critérios de Priorização de Famílias
* Robustez de evidência histórica
* Diversificação (baixa correlação entre famílias)
* Viabilidade prática (custos, slippage, liquidez)
  5.7. Métricas de Avaliação de Estratégias
* Sharpe, Sortino, Calmar
* DD máximo, tempo em DD
* Hit ratio, payoff ratio
* Turnover, impacto de custos

---

## 6. Família 1 – Trend Following / Time-Series Momentum (TSM)

6.1. Descrição Conceitual
6.2. Evidência Histórica e Referências
6.3. Adaptação para Spot Long/Flat

* Regra geral: `retorno_lookback > 0` → comprado; `<= 0` → cash
  6.4. Variantes
* Diferentes janelas (3–12m, 50–200 dias, etc.)
* TSM em múltiplos timeframes
* TSM com gerenciamento de risco dinâmico (vol targeting)
  6.5. Sinais e Filtros
* Filtros de volatilidade, volume, regime de mercado
  6.6. Parametrização
* Lookback, limites de posição, tempo máximo de trade
  6.7. Métricas Específicas
* Proteção em crises, tempo em mercado
  6.8. Roadmap de Implementação
* Primeira família a entrar em produção

---

## 7. Família 2 – Dual Momentum (TSM + Cross-Sectional)

7.1. Descrição Conceitual
7.2. Evidência Histórica e Referências
7.3. Adaptação para Spot Long-Only

* Filtro TSM (retorno > 0)
* Ranking cross-sectional (top X% por retorno)
  7.4. Universo de Ativos
* Cesta de pares cripto/ações/FX
  7.5. Sinais e Regras de Seleção
  7.6. Parametrização
* Janela TSM, janela cross-sectional, tamanho da cesta
  7.7. Risco Específico
* Rotação de portfólio, concentração em poucos ativos
  7.8. Estratégias de Diversificação Interna

---

## 8. Família 3 – Cross-Sectional Momentum (Relativo entre Ativos)

8.1. Descrição Conceitual
8.2. Evidência Histórica e “Momentum Crashes”
8.3. Adaptação Long-Only

* Long vencedores + cash (sem short perdedores)
  8.4. Universo e Critérios de Ranking
  8.5. Parametrização
* Janela de retorno, percentis, rebalance
  8.6. Risco Específico
* Drawdowns violentos, correlação concentrada
  8.7. Mitigação de Risco
* Limites de posição, limites de turnover

---

## 9. Família 4 – Carry / Yield / Basis

9.1. Descrição Conceitual
9.2. Evidência Histórica (FX, Bonds, Cripto Basis)
9.3. Adaptação a Spot + Derivativos Simples (Opcional)

* Spot + futuros para capturar funding/basis
  9.4. Componentes de Yield em Spot Puro
* Dividendos, staking, yields on-chain
  9.5. Sinais e Regras
* Posições em ativos com maior “yield ajustado ao risco”
  9.6. Risco Específico
* Perfil “ganha devagar, perde rápido”
  9.7. Controles de Cauda e Stress Scenarios

---

## 10. Família 5 – Value + Quality (Principalmente Ações Spot)

10.1. Descrição Conceitual
10.2. Definição de Fatores de Value (P/E, P/B, EV/EBITDA, etc.)
10.3. Definição de Fatores de Quality (ROE, ROA, alavancagem, estabilidade)
10.4. Evidência Histórica em Ações
10.5. Limitações em Cripto (value nebuloso)
10.6. Adaptação a Portfólios Mistos (Ações + Cripto)
10.7. Sinais, Ranking e Rebalance
10.8. Horizonte de Investimento (Longo prazo)

---

## 11. Família 6 – Mean Reversion de Curto Prazo (Intraday / Swing)

11.1. Descrição Conceitual
11.2. Evidência Histórica (equities e cripto)
11.3. Adaptação para Spot Long/Flat

* Compra em quedas exageradas, venda em spikes
  11.4. Indicadores e Microestrutura
* Bandas (Bollinger, Keltner), RSI, ordem limitada, depth
  11.5. Horizonte Temporal
* Intraday (minutos/horas) e swing (1–5 dias)
  11.6. Sensibilidade a Custos e Slippage
  11.7. Controle de Regime
* On/off dependendo de volatilidade, spread, horário
  11.8. Parametrização Específica

---

## 12. Família 7 – Market Making / Liquidity Providing

12.1. Descrição Conceitual
12.2. Posição no Roadmap (Prioridade Baixa para varejo)
12.3. Requisitos de Infra

* Latência, estabilidade de conexão, limites de API
  12.4. Modelos de Precificação
* Mid-price, spread dinâmico, inventário alvo
  12.5. Gestão de Inventário
  12.6. Risco de Cauda (crashes, gaps, news)
  12.7. Métricas Específicas
* P&L por spread, inventory P&L, adverse selection

---

## 13. Gestão de Risco e Alocação de Capital

13.1. Camadas de Risco

* Por trade
* Por estratégia
* Por família de estratégias
* Por exchange/ativo
  13.2. Limites e Guardrails
* Risco máximo por trade
* DD máximo diário/mensal
* Notional máximo por ativo e por exchange
  13.3. Alocação entre Famílias
* Regra fixa (ex.: 40% TSM, 30% Dual, etc.)
* Regra dinâmica baseada em performance recente
  13.4. Controle de Exposição Cambial (FX vs cripto vs fiat)
  13.5. Circuit Breakers e Modo Seguro
* Modo degradado (somente TSM, por exemplo)
  13.6. Instrumentação para medição de Risco

---

## 14. Backtesting, Simulação e Validação

14.1. Estrutura de Backtest

* Single-node, execução sequencial ou paralelismo limitado
  14.2. Engine de Backtest Unificado
* Mesma lógica de produção (isomorfismo)
  14.3. Modelagem de Custos e Slippage
  14.4. Cenários e Campanhas de Teste
* Diferentes regimes de mercado
* “Crises” históricas, bull/bear, sideways
  14.5. Validação Estatística
* OOS, walk-forward, cross-validation temporal
  14.6. Reprodutibilidade
* Seeds, versionamento de datasets, versionamento de código
  14.7. Ferramentas de Análise de Resultados
* Relatórios, gráficos, distribuição de retornos, correlações

---

## 15. Execução em Produção e Core Loop

15.1. Core Loop Operacional

* Scheduling, frequência, orquestração de múltiplas estratégias
  15.2. Modo Single Exchange (Binance Spot)
* Gestão de limites de API rate
  15.3. Evolução para Multi-Exchange
* Estratégia de enfileiramento de ordens por exchange
  15.4. Gestão de Estado
* Posições, ordens abertas, P&L
  15.5. Failover e Recuperação
  15.6. Modos de Operação
* Backtest → Paper → Live
  15.7. Segurança na Execução de Ordens
* Confirmação, validações de tamanho/prioridade, regras de emergência

---

## 16. Observabilidade e Telemetria

16.1. Logs Estruturados
16.2. Métricas de Sistema

* CPU, memória, I/O, latência
  16.3. Métricas de Negócio
* P&L por estratégia/família, Sharpe, drawdown
  16.4. Dashboards e Alertas
  16.5. Tracing de Estratégias
* De sinal → ordem → trade → P&L
  16.6. Auditoria de Decisões de Trading

---

## 17. Governança de Estratégias e Change Management

17.1. Pipeline de Aprovação de Estratégias

* Revisão técnica, revisão de risco
  17.2. Versionamento de Estratégias e Configurações
  17.3. Processo de Rollout e Rollback
  17.4. Critérios para Desativação ou Rebalance
  17.5. Documentação Obrigatória por Estratégia
* Hypothesis, evidência, parâmetros, riscos conhecidos

---

## 18. Segurança e Compliance

18.1. Gestão de Credenciais e Segredos
18.2. Permissões de API e Escopo Mínimo
18.3. Logs de Segurança
18.4. Requisitos Regulatórios (se aplicável)
18.5. Testes de Segurança (linters, scanners)

---

## 19. Roadmap de Evolução

19.1. Fases do Projeto

* Fase 1: Infra mínima + TSM em Binance Spot
* Fase 2: Dual Momentum + Cross-Sectional
* Fase 3: Mean Reversion de curto prazo
* Fase 4: Carry/Yield/Basis e Value/Quality
* Fase 5: Market Making (se fizer sentido)
  19.2. Marcos e Entregáveis
  19.3. Riscos de Projeto e Mitigações

---
