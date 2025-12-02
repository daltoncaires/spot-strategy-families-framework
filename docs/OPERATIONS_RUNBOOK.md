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

*   **Cenário de Falha:** Um health check crítico falha por múltiplas vezes consecutivas (ex: 3x).
*   **Ação do Sistema (Automática):** O sistema pode ser configurado para entrar automaticamente em **Modo Seguro (Seção 2.2)**.
*   **Ação do Operador:**
    1.  Identifique qual check está falhando (conectividade, dados, etc.) através dos logs e alertas.
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

*   **RPO (Recovery Point Objective):** Limitar a perda de dados transacionais a no máximo **5 minutos** de atividades recentes.
*   **RTO (Recovery Time Objective):** Restaurar o sistema a um estado operacional (em modo `PAPER` ou `SHADOW`) em **até 30 minutos** após a detecção da falha.

#### 3.3.1. Estratégia de Backups

*   **Estado do Sistema (SQLite):** Backups automáticos do arquivo `portfolio.db` a cada 5 minutos para um local secundário (ex: HD externo, armazenamento em nuvem).
*   **Logs e Artefatos:** Rotação diária, com arquivos compactados e movidos para armazenamento de longo prazo.
*   **Código e Configuração:** Versionamento via Git em repositório remoto.

#### 3.3.2. Procedimento de Recuperação

1.  **Parar o Sistema:**
    *   **Ação:** Interrompa imediatamente todos os processos do bot para evitar operações com estado inconsistente.
    *   **Comando Exemplo:** `systemctl stop trading-bot.service`

2.  **Restaurar o Estado (SQLite):**
    *   **Ação:** Identifique o último backup válido do arquivo de estado (`portfolio.db`) e restaure-o para o diretório de dados.

3.  **Reconciliar com a Exchange (Passo Crítico):**
    *   **Objetivo:** Comparar o estado restaurado com o estado real da conta na exchange.
    *   **Procedimento:**
        1.  Execute o script de reconciliação dedicado.
        2.  **Posições Órfãs (na exchange, não no bot):** O script deve criar a posição no estado local com um `reason_code: RECOVERED_ORPHAN` e notificar o operador.
        3.  **Posições Fantasma (no bot, não na exchange):** O script deve remover a posição do estado local e gerar um alerta `CRITICAL`, pois isso pode indicar uma perda não registrada.
        4.  **Divergências:** Para qualquer divergência de quantidade ou preço, **o estado da exchange prevalece**. O estado local é ajustado.

4.  **Reiniciar em Modo de Verificação (`PAPER_TRADING`):**
    *   **Ação:** Inicie o sistema em modo `PAPER_TRADING`.
    *   **Propósito:** Permitir que o sistema processe dados de mercado ao vivo e gere sinais, mas sem enviar ordens reais. Isso valida a saúde do sistema após a recuperação.
    *   **Duração:** Monitore por um período configurável (ex: 15-30 minutos).

5.  **Retornar ao Modo de Produção (`LIVE_TRADING`):**
    *   **Ação:** Somente após a verificação bem-sucedida, alterne o sistema para o modo `LIVE_TRADING`.
    *   **Notificação:** Envie uma notificação de "Sistema Restaurado e Operacional" para o canal de alertas.

---

## 4. Resposta a Incidentes de Segurança

*   **Cenário:** Suspeita de vazamento de API keys ou acesso não autorizado.
*   **Procedimento de Resposta Imediata:**
    1.  **Ative o Modo Seguro Global** para parar todas as atividades de trading.
    2.  **Revogue as API keys comprometidas** diretamente na interface da exchange.
    3.  **Rotacione as chaves:** Gere novas API keys com as permissões corretas (trading, sem saques) e atualize a configuração de segredos do sistema.
    4.  **Analise os Logs:** Investigue os logs de acesso e de trading em busca de atividades anômalas.
    5.  Documente o incidente e a resposta.
    6.  Somente após a resolução completa, saia do Modo Seguro e retome as operações.

---

## 5. Gerenciamento de Rate Limits

*   **Cenário:** O sistema começa a receber erros `429 (Too Many Requests)` da exchange.
*   **Comportamento do Sistema:**
    *   O sistema possui um mecanismo de **Token Bucket** e **Throttling Automático** para operar abaixo dos limites.
    *   Ao receber um erro 429, ele aplica um **backoff exponencial**.
    *   Se os erros persistirem, um **Circuit Breaker de Infraestrutura** é acionado.
*   **Ação do Operador:**
    1.  Verifique os dashboards de métricas para identificar qual componente ou estratégia está gerando um volume excessivo de requisições.
    2.  Considere ajustar os parâmetros de `quote_refresh_interval` (para Market Making) ou `rebalance_frequency` para estratégias muito ativas.

---

## 6. Procedimentos para Eventos Excepcionais de Mercado

| Evento | Detecção | Ação Automática do Sistema | Ação do Operador |
| :--- | :--- | :--- | :--- |
| **Delisting de Ativo** | O `Schema Discovery` detecta que o status de um símbolo mudou para `DELISTING` ou `BREAK`. | O Módulo de Risco coloca um **hard block** no símbolo, impedindo novas entradas. Se houver uma posição aberta, emite uma **ordem de saída a mercado imediata** (`ReasonCode: EXIT_FORCED_DELISTING`). | N/A (Ação automática é final). |
| **Halt de Trading** | O Módulo de Execução recebe um erro da API indicando que o trading para o símbolo está suspenso. | O Módulo de Risco coloca um **hard block** no símbolo, impedindo novas entradas. **NÃO tenta fechar a posição aberta.** Um alerta `CRITICAL` é enviado. | Monitorar o status da exchange e aguardar a retomada do trading. |
| **API Downtime Prolongado** | Falha de conexão com a API da exchange após múltiplas tentativas com backoff. | O sistema entra em modo de **"somente-redução" (reduce-only)**, impedindo novas posições. O Módulo de Risco continua a gerenciar as posições abertas com os dados disponíveis. | Monitorar o status da exchange. Acionar a reconciliação manual após a restauração da conexão, se necessário. |
| **Flash Crash** | O Módulo de Risco detecta uma queda de preço percentual anormalmente grande em um curto período (ex: >15% em 1 minuto). | **Circuit Breaker de Símbolo:** Bloqueia novas entradas para esse símbolo por um período configurável (ex: 5 minutos). As regras de stop-loss para posições abertas continuam a funcionar. | Revisar o mercado e, se necessário, reautorizar o trading para o símbolo manualmente. |

---
