# Runbook de Operações

Este documento é um guia prático para o **Responsável Operacional/DevOps**. Ele descreve os procedimentos padrão para monitorar a saúde do sistema, responder a incidentes e executar tarefas de recuperação.

---

## 1. Monitoramento de Saúde (Health Checks)

O sistema possui mecanismos automáticos para verificar sua saúde. O operador deve monitorar os dashboards e alertas gerados por esses checks.

### 1.1. Heartbeat Interno

*   **O que é:** O core loop emite um sinal de "heartbeat" a cada ciclo de execução bem-sucedido.
*   **Cenário de Falha:** Um alerta é disparado se um heartbeat não for recebido dentro de um timeout configurado (ex: 3x o tempo de ciclo esperado).
*   **Ação do Operador:**
    1.  Verifique os logs do sistema em busca de erros `CRITICAL` ou `ERROR` que possam ter travado o processo.
    2.  Verifique os recursos do sistema (CPU, memória) para identificar gargalos.
    3.  Se o processo caiu, siga o procedimento de **Recuperação de Falha de Processo (Seção 3.1)**.

### 1.2. Health Checks Periódicos

A cada 60 segundos, o sistema verifica os seguintes pontos:

*   **Conectividade com a Exchange:** Ping no endpoint REST e status da conexão WebSocket.
*   **Atualidade dos Dados:** Idade do último candle recebido.
*   **Reconciliação de Posição:** Desvio (`drift`) entre o estado interno e o da exchange.
*   **Recursos do Sistema:** Uso de disco e memória.
*   **Sincronização de Tempo (NTP):** Status do serviço NTP e desvio do relógio (offset).

*   **Cenário de Falha:** Um health check crítico falha por múltiplas vezes consecutivas (ex: 3x).
*   **Ação do Sistema (Automática):** O sistema pode ser configurado para entrar automaticamente em **Modo Seguro (Seção 2.2)**.
*   **Ação do Operador:**
    1.  O sistema já deve ter entrado em Modo Seguro. Identifique qual check falhou através dos logs e alertas para investigar a causa raiz.
    2.  Siga o procedimento específico para o tipo de falha (ex: problema de rede, inconsistência de posição).

---

## 2. Circuit Breakers e Modo Seguro

Os Circuit Breakers (CBs) são mecanismos automáticos de proteção.

### 2.1. Tipos de Circuit Breakers

*   **Nível 1 - Por Estratégia (Instância):**
    *   **Gatilho:** Drawdown da instância > 5% em 1 dia, ou > 10% no mês.
    *   **Gatilho:** 5 trades perdedores consecutivos.
    *   **Ação:** Pausa novas entradas para a instância (`pause-new-entries`), permitindo apenas o fechamento de posições existentes. Requer revisão manual para reativar.

*   **Nível 2 - Por Família:**
    *   **Gatilho:** Drawdown agregado da família > 15% no mês.
    *   **Gatilho:** Correlação entre instâncias da mesma família excede um limiar (ex: `ρ > 0.9`), indicando perda de diversificação interna.
    *   **Ação:** Pausa novas entradas para toda a família e emite um alerta de alta prioridade.

*   **Nível 3 - Global (Portfólio Total):**
    *   **Gatilho:** Drawdown total do portfólio > 20% desde o último high-water mark.
    *   **Gatilho:** Perda intraday > 3% do equity total.
    *   **Gatilho:** Taxa de erros da API da exchange > 10 erros/minuto por um período sustentado.
    * **Gatilho:** Frequência de incidentes > 3 incidentes críticos (que exigem intervenção manual) em 7 dias.
    *   **Ação:** Ativação do **Modo Seguro Global (Safe Mode)**.

### 2.2. Procedimento em Modo Seguro (Safe Mode)

O Modo Seguro é o estado de proteção máxima do sistema.

1.  **Comportamento do Sistema:**
    *   Cancela todas as ordens abertas que não sejam de saída (stops/take-profits).
    *   Bloqueia imediatamente o envio de quaisquer novas ordens de entrada.
    *   Mantém as posições existentes, não força o fechamento para evitar a realização de perdas em pânico, a menos que uma política de "HALT ALL" seja explicitamente configurada.
    *   Envia um alerta de criticidade máxima.

2.  **Operação Degradada:**
    *   O sistema continua monitorando posições e P&L, mas o core loop de geração de sinais é desativado.
    *   Apenas fechamentos manuais de posição, após aprovação explícita, são permitidos.

2.  **Ação do Operador:**
    *   **NÃO REATIVE IMEDIATAMENTE.**
    *   **Investigue a Causa Raiz:** Analise os logs e métricas que levaram ao acionamento do Modo Seguro (ex: drawdown, erro de API, etc.).
    *   **Avalie o Estado do Mercado e do Portfólio:** Verifique as posições abertas e as condições atuais do mercado.
    *   **Decida sobre as Posições:** Decida se as posições existentes devem ser fechadas manualmente.
    *   **Saída do Modo Seguro:** A saída **não é automática**. Requer uma intervenção manual e autorização explícita via configuração ou comando de governança, somente após a análise e resolução da causa raiz.

### 2.3. Política de Rampa de Realocação Pós-Incidente

Após a ocorrência de um incidente grave que ativou o `Safe Mode` Global, o retorno à operação normal com 100% do capital alocado apresenta um risco elevado. Para mitigar isso, uma política de realocação gradual ("rampa") deve ser seguida.

1.  **Estado de Recuperação (`Recovery Mode`):**
    *   Após sair do `Safe Mode`, o sistema não entra diretamente em `Live Mode` com 100% do capital. Ele deve ser colocado em um estado intermediário de **`Recovery Mode`**.
    *   Neste modo, o módulo de risco global aplica um fator de escala redutor a todo o capital. Por exemplo, `recovery_capital_factor = 0.10` (10%).

2.  **Critérios para Aumento da Rampa:**
    *   O operador pode solicitar um aumento no `recovery_capital_factor` (ex: de 10% para 25%) através de um comando de governança (`/set_capital_factor 0.25`).
    *   Este aumento só deve ser solicitado se os seguintes critérios forem atendidos:
        1.  **Período de Estabilidade:** O sistema operou por um período mínimo sem novos incidentes críticos (ex: 7 dias).
        2.  **Performance Aceitável:** O drawdown do portfólio durante o `Recovery Mode` se manteve dentro de limites estritos (ex: < 2%).
        3.  **Causa Raiz Resolvida:** A causa raiz do incidente original foi documentada, corrigida (se for um bug) e validada.

3.  **Fluxo Gradual:**
    *   A realocação de volta aos 100% deve ser feita em etapas (ex: 10% → 25% → 50% → 75% → 100%), com um período de observação entre cada aumento.
    *   Isso garante que qualquer instabilidade residual no sistema ou no mercado cause um impacto financeiro limitado.

---

## 3. Failover e Disaster Recovery (DR)

Procedimentos para recuperar o sistema após uma falha, desde um simples reinício até um cenário de desastre.

### 3.1. Recuperação de Falha de Processo (Restart)

*   **Cenário:** O processo principal do bot caiu ou foi reiniciado.
*   **Procedimento de Recuperação Automática:**
    1.  No restart, o sistema recarrega o último estado persistido (posições, ordens) do banco de dados local (SQLite).
    2.  Realiza um **reconciliation** com a exchange para validar o estado.
    3.  Reconstrói os buffers de dados de mercado a partir do histórico local e faz um backfill de dados recentes via API REST.
    4.  Retoma o core loop em modo consistente.
*   **Ação do Operador:**
    1.  Monitore os logs do restart para garantir que a reconciliação foi bem-sucedida.
    2.  Verifique o dashboard de saúde para confirmar que o sistema voltou a operar normalmente.

### 3.2. Gestão de Falhas de Conectividade

*   **Cenário:** Falhas temporárias de rede ou erros de API da exchange.
*   **Comportamento do Sistema:**
    *   Aplica políticas de **retry com backoff exponencial** para requisições falhas.
    *   Se a falha persistir, um **Circuit Breaker de Infraestrutura** pode ser acionado, pausando temporariamente novas ordens para a exchange afetada.
*   **Ação do Operador:**
    1.  Se os alertas de falha de conectividade persistirem, verifique a saúde da rede do nó e o status da API da exchange.
    2.  Considere pausar manualmente as operações se a instabilidade for prolongada.

### 3.3. Plano de Disaster Recovery (DR)

Este plano detalha os procedimentos para restaurar o sistema após uma falha crítica (ex: corrupção de dados, falha de hardware).
*   **RPO (Recovery Point Objective):** < 1 segundo (graças à replicação contínua do Litestream, conforme definido no **ADR-014**).
*   **RTO (Recovery Time Objective):** < 10 minutos para restauração manual, < 2 minutos com failover Hot-Warm.

#### 3.3.1. Estratégia de Backups

*   **Estado do Sistema (SQLite):** Replicação contínua em tempo real do arquivo `portfolio.db` para um bucket S3 via **Litestream**. O Litestream gerencia snapshots e a replicação do WAL (Write-Ahead Log).
*   **Logs e Artefatos:** Rotação diária, com arquivos compactados e movidos para armazenamento de longo prazo.
*   **Código e Configuração:** Versionamento via Git em repositório remoto.

#### 3.3.2. Procedimento de Recuperação

O procedimento abaixo descreve os passos lógicos. É **mandatório** que estes passos sejam encapsulados em um **script de recuperação totalmente automatizado** (ex: `disaster-recovery.sh`) para eliminar o risco de erro humano durante um evento de crise. A função do operador é acionar o script e supervisionar sua execução, não executar os passos manualmente.

**Teste Automatizado (DR Drill):** A confiabilidade deste script é crítica. O pipeline de CI/CD **deve** incluir um job periódico (ex: semanal) que executa um "DR Drill": provisiona um ambiente de teste, simula uma falha, executa o script `disaster-recovery.sh` e valida que o sistema foi restaurado a um estado consistente. Qualquer falha neste job deve ser tratada como um build quebrado.

1.  **Parar o Sistema:**
    *   **Ação Automática do Script:** Interrompe todos os processos do bot para evitar operações com estado inconsistente.
    *   **Comando Exemplo:** `systemctl stop trading-bot.service` (executado pelo script).

2.  **Restaurar o Estado (SQLite):**
    *   **Ação Automática do Script:** Usa o comando do Litestream para restaurar o banco de dados a partir do bucket S3 para o ponto mais recente.
    *   **Comando Exemplo:** `litestream restore -o data/portfolio.db s3://my-bucket/portfolio.db` (executado pelo script).

3.  **Reconciliar com a Exchange (Passo Crítico):**
    *   **Objetivo:** Garantir que o estado restaurado do bot esteja 100% sincronizado com o estado real da conta na exchange.
    *   **Procedimento (Automatizado pelo Script de Recuperação):**
        1.  O script inicia o sistema em um modo especial de **reconciliação automática**.
        2.  Ele consulta as posições e ordens abertas na exchange.
        3.  Compara com o estado restaurado e aplica as correções necessárias (ajusta quantidades, remove posições fantasma, adota posições órfãs).
        4.  Gera um **relatório de reconciliação** detalhado e o envia via Telegram, informando ao operador exatamente quais divergências foram encontradas e como foram resolvidas. A intervenção manual só é necessária se o script falhar.

4.  **Reiniciar em Modo de Verificação (`PAPER_TRADING`):**
    *   **Ação:** Inicie o sistema em modo `PAPER_TRADING`.
    *   **Propósito:** Permitir que o sistema processe dados de mercado ao vivo e gere sinais, mas sem enviar ordens reais. Isso valida a saúde do sistema após a recuperação.
    *   **Ação Automática do Script:** O script reinicia o bot com a flag `--mode=paper` e aguarda um período de verificação (ex: 15 minutos).

5.  **Retornar ao Modo de Produção (`LIVE_TRADING`):**
    *   **Ação do Operador (Ponto de Decisão):** Após revisar o relatório de reconciliação e o status do sistema em modo `paper`, o operador dá o comando final para o script promover o sistema para `LIVE_TRADING`.
    *   **Ação Automática do Script:** Altera o modo de operação para `live` e envia uma notificação de "Sistema Restaurado e Operacional".

---

## 4. Console de Operações e Interface Humana

Toda a interação do operador com o sistema deve ocorrer através de uma interface de controle canônica (ex: um bot de Telegram seguro ou uma CLI), e não por acesso direto ao servidor (SSH), que deve ser reservado apenas para emergências de infraestrutura. A interface é dividida em comandos de consulta (seguros, read-only) e comandos de governança (que alteram o estado e exigem permissões elevadas).

### 4.1. Comandos de Consulta (Read-Only)

Estes comandos são seguros para serem executados a qualquer momento e servem para diagnóstico e monitoramento.

*   `/status`: Retorna um resumo geral da saúde do sistema: modo de operação (Live, Paper, Safe), P&L do dia, número de estratégias ativas, e status dos health checks.
*   `/status <strategy_id>`: Retorna o status detalhado de uma instância de estratégia específica (P&L, exposição, drawdown, últimos trades).
*   `/portfolio`: Mostra a alocação de capital atual por ativo e por família de estratégia.
*   `/positions`: Lista todas as posições abertas no portfólio.
*   `/open_orders`: Lista todas as ordens abertas na exchange.
*   `/risk_limits`: Exibe os limites de risco globais e o consumo atual de cada "risk budget".
*   `/health`: Executa e retorna o resultado dos health checks periódicos sob demanda.

### 4.2. Comandos de Governança (Write)

Estes comandos alteram o estado do sistema e devem ser protegidos por autenticação forte e, para ações críticas, um controle duplo (dois operadores).

*   `/set_mode <safe|paper|live>`: Altera o modo de operação global do sistema. Requer confirmação explícita.
*   `/pause_strategy <strategy_id>`: Pausa novas entradas para uma estratégia específica.
*   `/resume_strategy <strategy_id>`: Retoma a operação de uma estratégia pausada.
*   `/exit_position <symbol>`: Inicia um fechamento controlado da posição no símbolo especificado.
*   `/rollback_config <strategy_id> <target_version>`: Executa o rollback de uma configuração de estratégia para uma versão anterior validada (conforme ADR-018).

### 4.3. Kill Switch de Emergência (Via Comando Remoto)

*   **O que é:** Um mecanismo de parada de emergência para cessar toda a atividade de trading de forma imediata. É a ação de último recurso em caso de comportamento anômalo, instabilidade severa de mercado ou suspeita de comprometimento.
*   **Gatilho:** Execução de um comando `/kill_switch` na interface de governança.
*   **Ação do Sistema (< 5 segundos):**
    1.  **`cancel_all_orders`**: Cancela imediatamente todas as ordens abertas na exchange.
    2.  **`reduce_only`**: Coloca todas as estratégias e o módulo de execução em modo "somente redução", impedindo a abertura de novas posições.
    3.  **`safe_mode`**: Ativa o **Modo Seguro Global (Safe Mode)**, que bloqueia a geração de novos sinais e envia um alerta de criticidade máxima.
*   **Ação do Operador:**
    1.  **NÃO REATIVE IMEDIATAMENTE.**
    2.  O acionamento do kill switch é um evento grave. Siga o **Procedimento em Modo Seguro (Seção 2.2)** para investigar a causa raiz antes de considerar a reativação do sistema.
    3.  Verifique os logs para confirmar que as ações (`cancel_all_orders`, `reduce_only`, `safe_mode`) foram executadas com sucesso.
    4.  Reconcilie manualmente o estado das posições na exchange para garantir que não há posições ou ordens inesperadas.

---

## 5. Resposta a Incidentes de Segurança

*   **Cenário:** Suspeita de vazamento de API keys ou acesso não autorizado.
*   **Procedimento de Resposta Imediata:**
    1.  **Ative o Modo Seguro Global** para parar todas as atividades de trading.
    2.  **Revogue as API keys comprometidas** diretamente na interface da exchange.
    3.  **Rotacione as chaves:** Gere novas API keys com as permissões corretas (trading, sem saques). Atualize o segredo no **GitHub Actions Secrets** (`Settings > Secrets and variables > Actions` do repositório) e acione o workflow de deploy/restart para que o sistema passe a usar a nova chave.
    4.  **Analise os Logs:** Investigue os logs de acesso e de trading em busca de atividades anômalas.
    5.  Documente o incidente e a resposta.
    6.  Somente após a resolução completa, saia do Modo Seguro e retome as operações.

---

## 6. Gerenciamento de Rate Limits (ADR-016)

*   **Cenário:** O sistema começa a receber erros `429 (Too Many Requests)` da exchange.
*   **Comportamento do Sistema:**
    *   **Camada 1 (Token Bucket):** O sistema deve, na maior parte do tempo, evitar o erro gerenciando proativamente as chamadas.
    *   **Camada 2 (Backoff Exponencial):** Ao receber um erro 429, o sistema automaticamente pausa as requisições para o endpoint afetado e tenta novamente em intervalos crescentes. Um alerta de nível `WARNING` é gerado.
    *   **Camada 3 (Circuit Breaker):** Se os erros persistirem mesmo com o backoff, um **Circuit Breaker de Infraestrutura** pode ser acionado, pausando todas as operações com a exchange e gerando um alerta `CRITICAL`.
*   **Ação do Operador:**
    1.  **Verifique os Dashboards de Métricas:**
        *   **`rate_limit_token_bucket_usage{endpoint="..."}`:** Procure por endpoints cujo bucket está consistentemente em 0. Isso indica um gargalo.
        *   **`rate_limit_backoff_triggered_total{endpoint="..."}`:** Verifique se o contador de backoffs está aumentando para algum endpoint. Isso confirma que o sistema está reagindo a erros reais.
        *   **`api_calls_total{strategy_id="..."}`:** Identifique se alguma estratégia específica está gerando um volume desproporcional de requisições.
    2.  **Analise a Causa Raiz:**
        *   Um aumento no uso de tokens é esperado durante alta volatilidade.
        *   Se o problema for persistente e concentrado em uma estratégia, pode haver um bug ou uma configuração muito agressiva.
    *   **Ação Corretiva:**
        *   Considere ajustar os parâmetros da estratégia problemática (ex: aumentar `rebalance_frequency` ou `quote_refresh_interval`).
        *   Se o problema for geral, pode ser necessário revisar a configuração dos Token Buckets para alinhá-los com novos limites da exchange.

---

## 7. Diagnóstico Automático de Incidentes

Para acelerar a análise de causa raiz e reduzir o trabalho manual do operador, o sistema deve gerar um **"Dossiê de Incidente"** automaticamente sempre que um evento crítico for acionado (ex: Circuit Breaker Global, erro grave de reconciliação, acionamento do Kill Switch).

*   **Gatilho:** Evento de sistema classificado como `CRITICAL`.
*   **Ação Automática do Sistema:**
    1.  O sistema coleta e agrega um conjunto de dados de diagnóstico relacionados ao evento, usando o `correlation_id` para rastrear a cadeia de causalidade.
    2.  Gera um relatório formatado (HTML ou Markdown) contendo:
        *   **Resumo do Incidente:** Timestamp, tipo de evento, estratégia/ativo/módulo envolvido.
        *   **Snapshot do Estado:** Posições, P&L, e limites de risco no momento do evento.
        *   **Gráfico de Mercado:** Gráfico do preço do ativo relevante nos minutos que antecederam o incidente.
        *   **Logs Relevantes:** Extrato dos logs (nível `INFO` e acima) da(s) componente(s) envolvida(s) nos 5 minutos anteriores ao evento.
        *   **Curva de P&L Recente:** Gráfico do P&L da estratégia ou portfólio nas últimas 24 horas.
    3.  O dossiê é salvo em um diretório de artefatos (`/artifacts/incidents/<incident_id>.html`) e um link é enviado para o canal de alertas do operador.

*   **Ação do Operador:**
    *   Em vez de coletar manualmente os dados de diferentes fontes (dashboards, logs, gráficos), o operador começa a análise a partir do dossiê consolidado, economizando tempo crítico e reduzindo a chance de perder informações importantes.

---

## 8. Procedimentos para Eventos Excepcionais de Mercado

| Evento | Detecção | Ação Automática do Sistema | Ação do Operador |
| :--- | :--- | :--- | :--- |
| **Delisting de Ativo** | O `Schema Discovery` detecta que o status de um símbolo mudou para `DELISTING` ou `BREAK`. | O Módulo de Risco coloca um **hard block** no símbolo, impedindo novas entradas. Se houver uma posição aberta, emite uma **ordem de saída a mercado imediata** (`ReasonCode: EXIT_FORCED_DELISTING`). | N/A (Ação automática é final). |
| **Halt de Trading** | O Módulo de Execução recebe um erro da API indicando que o trading para o símbolo está suspenso. | O Módulo de Risco coloca um **hard block** no símbolo, impedindo novas entradas. **NÃO tenta fechar a posição aberta.** Um alerta `CRITICAL` é enviado. | Monitorar o status da exchange e aguardar a retomada do trading. |
| **API Downtime Prolongado** | Falha de conexão com a API da exchange após múltiplas tentativas com backoff. | O sistema entra em modo de **"somente-redução" (reduce-only)**, impedindo novas posições. O Módulo de Risco continua a gerenciar as posições abertas com os dados disponíveis. | Monitorar o status da exchange. Acionar a reconciliação manual após a restauração da conexão, se necessário. |
| **Flash Crash** | O Módulo de Risco detecta uma queda de preço percentual anormalmente grande em um curto período (ex: >15% em 1 minuto). | **Circuit Breaker de Símbolo:** Bloqueia novas entradas para esse símbolo por um período configurável (ex: 5 minutos). As regras de stop-loss para posições abertas continuam a funcionar. | Revisar o mercado e, se necessário, reautorizar o trading para o símbolo manualmente. |

---
