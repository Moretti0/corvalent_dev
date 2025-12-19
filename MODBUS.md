# Modbus Plugin - Plano de Evolução Enterprise (Revisado)

> **Objetivo:** Transformar o plugin Modbus em um Gateway Industrial de Alta Disponibilidade, focado em estabilidade de rede, controle granular via API e adaptação dinâmica às condições do chão de fábrica.

---

## 🚀 1. Funcionalidades Enterprise (Core)

Funcionalidades essenciais implementadas em C++23 para garantir robustez e eficiência.

### 1.1 "Report by Exception" (RBE) & Deadband
*   **Conceito:** Redução de tráfego publicando apenas mudanças significativas.
*   **Mecanismo:** Configuração de `delta` (variação mínima) e `max_interval` (heartbeat).
*   **Impacto:** Economia de 90% em armazenamento e banda.

### 1.2 Modbus Write Seguro (Firewall de Aplicação)
*   **Conceito:** Controle de escrita com validação de segurança.
*   **Mecanismo:** Whitelist de registros permitidos e validação de range (Min/Max) antes do envio ao PLC.
*   **Impacto:** Segurança operacional contra comandos destrutivos.

### 1.3 Virtual Modbus Server (Slave Mode)
*   **Conceito:** Expõe dados do Corgrid via porta 502 para SCADAs legados.
*   **Impacto:** Integração IT/OT bidirecional.

### 1.4 Leitura em Bloco Otimizada (Smart Batching)
*   **Conceito:** Agrupamento automático de leituras adjacentes em um único request Modbus.
*   **Impacto:** Aumento de throughput em redes seriais lentas.

### 1.5 Modbus Traffic Analyzer
*   **Conceito:** Métricas de saúde da rede (CRC, Timeouts).
*   **Impacto:** Manutenção preditiva do cabeamento.

---

## ⚡ 2. Novas Funcionalidades Disruptivas (Foco em Estabilidade e Controle)

### 2.1 Adaptive Polling Rate (Tempo de Resposta Adaptativo)
**Conceito:**
O plugin ajusta dinamicamente a frequência de leitura com base na saúde da rede, evitando o colapso do barramento em situações de instabilidade.

**Mecanismo (Algoritmo Smart Backoff):**
1.  **Monitoramento:** O plugin calcula a taxa de sucesso (Success Rate) e latência média (Avg RTT) em janela móvel (ex: últimos 10 ciclos).
2.  **Degradação:** Se detectar erros consecutivos (Timeouts/CRC) acima de um limiar, o plugin entra em *Protection Mode*, aumentando o intervalo de polling (ex: de 1s para 5s) automaticamente.
3.  **Recuperação:** Conforme a rede estabiliza (sucessos consecutivos), o intervalo é reduzido gradualmente até o valor nominal configurado.
4.  **Priorização:** Dispositivos críticos mantêm prioridade na fila, enquanto dispositivos instáveis são penalizados.

**Valor Enterprise:**
Impede que um dispositivo defeituoso "congele" toda a rede RS485 ao forçar o mestre a esperar timeouts repetidos. Mantém o sistema responsivo para os demais equipamentos.

### 2.2 Configuração Avançada via JSON (Endianness & Data Types)
**Conceito:**
Suporte total a dialetos Modbus complexos e formatação de dados industriais diretamente no `config/plugins/Modbus.json`.

**Recursos Suportados:**
*   **Word Swapping / Byte Swapping:** Controle total de *Endianness* (Big-Endian, Little-Endian, Mid-Little, Mid-Big) para ler floats e longs corretamente de PLCs de diferentes fabricantes (Siemens vs Rockwell vs Schneider).
*   **Data Types Estendidos:**
    *   `float32`, `float64` (IEEE 754)
    *   `int64`, `uint64` (Contadores de energia)
    *   `string` (Decodificação de texto ASCII em registradores)
    *   `bitmap` (Extração de bits individuais de uma Word para booleanos)
*   **Fatores de Escala:** `multiplier` e `offset` aplicados na borda (ex: Ler 1234 -> aplicar 0.1 -> publicar 123.4).

### 2.3 API RESTful Granular (Endpoints Dedicados)
**Conceito:**
Exposição completa das funções do plugin via API HTTP, permitindo gerenciamento em tempo real sem editar arquivos manualmente ou reiniciar. Segue padrão AWS OpenAPI.

**Endpoints Planejados:**

*   **Gerenciamento de Dispositivos:**
    *   `POST /api/v1/plugins/modbus/devices` - Registrar novo PLC/Medidor.
    *   `PUT /api/v1/plugins/modbus/devices/{id}` - Alterar config (IP, Baudrate, Timeout).
    *   `DELETE /api/v1/plugins/modbus/devices/{id}` - Remover dispositivo.
    *   `POST /api/v1/plugins/modbus/devices/{id}/test` - Testar conexão (Ping Modbus).

*   **Gerenciamento de Mapeamentos (Tags):**
    *   `POST /api/v1/plugins/modbus/mappings` - Adicionar ponto de leitura.
    *   `GET /api/v1/plugins/modbus/mappings?device={id}` - Listar tags de um dispositivo.

*   **Operações Imediatas (Ad-hoc):**
    *   `POST /api/v1/plugins/modbus/read` - Leitura síncrona de diagnóstico (fura a fila de polling).
        *   *Body:* `{"connection": "plc1", "address": 4001, "count": 1}`
    *   `POST /api/v1/plugins/modbus/write` - Escrita síncrona segura.
        *   *Body:* `{"connection": "plc1", "address": 4001, "value": 123}`

*   **Discovery:**
    *   `POST /api/v1/plugins/modbus/scan` - Iniciar varredura de rede (IP range ou Serial Bus).
    *   `GET /api/v1/plugins/modbus/scan/status` - Progresso do scan.

---

## 3. Integração Frontend

O plugin fornecerá os dados necessários para popular as interfaces:
*   **Gráficos de Latência:** Via métricas de telemetria para exibir a estabilidade da rede.
*   **Wizard de Mapeamento:** O frontend usa a API de `test` e `read` para ajudar o usuário a validar se o endereço 4001 é realmente a "Temperatura" antes de salvar o mapeamento definitivo.