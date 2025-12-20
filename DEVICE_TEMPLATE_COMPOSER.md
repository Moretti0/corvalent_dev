# CorGrid OS - Device Template Composer

**Document ID:** DEVICE-TEMPLATE-COMPOSER-V3
**Data:** 20/12/2025
**Propósito:** Guia funcional para criação de Device Templates e Sistema de Regras
**Autoridade:** Core Architecture Team

---

## Índice

1. [Visão Geral](#1-visão-geral)
2. [Estrutura do Device Template](#2-estrutura-do-device-template)
3. [Processo de Criação](#3-processo-de-criação)
4. [SensorSlot e Capabilities](#4-sensorslot-e-capabilities)
5. [Sistema de Regras](#5-sistema-de-regras)
6. [Operadores Temporais e Estatísticos](#6-operadores-temporais-e-estatísticos)
7. [Inferência AI/ML](#7-inferência-aiml)
8. [Formato de Armazenamento](#8-formato-de-armazenamento)
9. [API REST](#9-api-rest)
10. [Padronização e Escalabilidade](#10-padronização-e-escalabilidade)
11. [Roadmap](#11-roadmap)

---

## 1. Visão Geral

O **Device Template Composer** é o sistema de criação de templates de dispositivos do CorGrid OS. Permite modelar equipamentos industriais através de uma abordagem composicional, onde cada template define:

- **Metadados** - Identificação do dispositivo (fabricante, modelo, categoria)
- **Assets** - Recursos visuais (imagem 2D, modelo 3D, manual PDF)
- **Slots** - Pontos de conexão para sensores e atuadores
- **Regras Embutidas** - Automações que viajam com o template

### Filosofia: Templates como Blueprints Replicáveis

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│   TEMPLATE (Blueprint)              INSTÂNCIAS (Devices Reais)          │
│                                                                         │
│   ┌───────────────────┐             ┌───────────────────┐               │
│   │                   │             │ Gerador-001       │               │
│   │  GERADOR DIESEL   │────────────►│ Local: São Paulo  │               │
│   │  500KVA           │             └───────────────────┘               │
│   │                   │                                                 │
│   │  • 5 SensorSlots  │             ┌───────────────────┐               │
│   │  • 4 Regras       │────────────►│ Gerador-002       │               │
│   │  • Modelo 3D      │             │ Local: Curitiba   │               │
│   │                   │             └───────────────────┘               │
│   └───────────────────┘                                                 │
│                                     ┌───────────────────┐               │
│                        ────────────►│ Gerador-003       │               │
│                                     │ Local: Manaus     │               │
│                                     └───────────────────┘               │
│                                                                         │
│   1 Template = N Instâncias com mesmas regras e estrutura               │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Estrutura do Device Template

### 2.1. Componentes do Template

Um Device Template é composto por:

| Componente | Descrição |
|------------|-----------|
| **id** | Identificador único do template |
| **metadata** | Nome, fabricante, modelo, categoria, especificações |
| **assets** | Imagem 2D, modelo 3D (GLB/GLTF), manual PDF |
| **camera** | Posição inicial da câmera para visualização 3D |
| **slots** | Lista de pontos de conexão para sensores |
| **embedded_rules** | Regras de automação embutidas no template |

### 2.2. Categorias de Device

| Categoria | Descrição |
|-----------|-----------|
| sensor | Dispositivo de coleta de dados |
| actuator | Dispositivo de atuação/controle |
| controller | Controlador lógico (CLP, RTU) |
| gateway | Concentrador de comunicação |
| display | Interface de visualização |
| machine | Máquina/equipamento complexo |
| vehicle | Veículo ou equipamento móvel |
| panel | Painel elétrico/de controle |

### 2.3. Diagrama de Estrutura

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│                        DEVICE TEMPLATE STRUCTURE                        │
│                                                                         │
│   ┌─────────────────────────────────────────────────────────────────┐   │
│   │  METADATA                                                       │   │
│   │  ├── name: "Gerador Diesel 500KVA"                              │   │
│   │  ├── manufacturer: "Cummins"                                    │   │
│   │  ├── model: "C500D5"                                            │   │
│   │  ├── category: "machine"                                        │   │
│   │  └── specs: { power: "500kva", fuel: "diesel" }                 │   │
│   └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│   ┌─────────────────────────────────────────────────────────────────┐   │
│   │  ASSETS                                                         │   │
│   │  ├── image_2d: "/uploads/templates/gerador_2d.png"              │   │
│   │  ├── model_3d: "/uploads/templates/gerador_3d.glb"              │   │
│   │  └── manual: "/uploads/templates/gerador_manual.pdf"            │   │
│   └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│   ┌─────────────────────────────────────────────────────────────────┐   │
│   │  SLOTS (Pontos de Conexão)                                      │   │
│   │                                                                 │   │
│   │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  │   │
│   │  │ Slot: temp_motor│  │ Slot: rpm       │  │ Slot: nivel_comb│  │   │
│   │  │ Types: [temp]   │  │ Types: [rpm]    │  │ Types: [level]  │  │   │
│   │  │ Pos: [0,1.5,0]  │  │ Pos: [0.5,1,0]  │  │ Pos: [-1,0,0]   │  │   │
│   │  └─────────────────┘  └─────────────────┘  └─────────────────┘  │   │
│   │                                                                 │   │
│   └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│   ┌─────────────────────────────────────────────────────────────────┐   │
│   │  EMBEDDED RULES (Regras Embutidas)                              │   │
│   │  ├── Rule 1: Proteção Térmica                                   │   │
│   │  ├── Rule 2: Nível Baixo Combustível                            │   │
│   │  ├── Rule 3: Vibração Excessiva                                 │   │
│   │  └── Rule 4: Pressão Óleo Crítica                               │   │
│   └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Processo de Criação

### 3.1. Etapas de Criação

A criação de um Device Template segue 5 etapas:

```
┌────────┐   ┌────────┐   ┌────────┐   ┌────────┐   ┌────────┐
│ ETAPA  │──►│ ETAPA  │──►│ ETAPA  │──►│ ETAPA  │──►│ ETAPA  │
│   1    │   │   2    │   │   3    │   │   4    │   │   5    │
│        │   │        │   │        │   │        │   │        │
│ Info   │   │ Rede   │   │Capacid.│   │Integr. │   │Revisão │
│ Básica │   │        │   │        │   │        │   │        │
└────────┘   └────────┘   └────────┘   └────────┘   └────────┘
```

### 3.2. Etapa 1: Informações Básicas

| Campo | Obrigatório | Descrição |
|-------|-------------|-----------|
| name | Sim | Nome do template |
| type | Sim | physical ou virtual |
| category | Sim | Categoria do device |
| manufacturer | Não | Fabricante |
| model | Não | Modelo |
| description | Não | Descrição detalhada |

### 3.3. Etapa 2: Configuração de Rede

**Protocolos suportados:**

| Protocolo | Categoria | Parâmetros |
|-----------|-----------|------------|
| Modbus TCP | Industrial | IP, Porta, Unit ID |
| Modbus RTU | Industrial | Serial, Baud, Parity |
| OPC UA | Industrial | Endpoint, Security Mode |
| MQTT | IoT | Broker, Topic, QoS |
| REST API | Web | Endpoint, Auth Type |
| WebSocket | Realtime | URL, Subprotocol |
| CAN Bus | Automotive | Bitrate, Node ID |
| ProfiNet | Industrial | GSD, IP |
| EtherNet/IP | Industrial | IP, Assembly |

### 3.4. Etapa 3: Capabilities

Capacidades operacionais do device:

| Capability | Descrição |
|------------|-----------|
| monitoring | Coleta de dados de sensores |
| control | Envio de comandos para atuadores |
| dataProcessing | Processamento local de dados |
| visualization | Capacidade de exibir informações |

### 3.5. Etapa 4: Integração

| Opção | Descrição |
|-------|-----------|
| protocols | Lista de protocolos habilitados |
| apiEndpoints | Exposição via REST API |
| webInterface | Interface web embarcada |

### 3.6. Etapa 5: Revisão

Validação final e publicação do template na biblioteca.

---

## 4. SensorSlot e Capabilities

### 4.1. O que é um SensorSlot

Um **SensorSlot** é um ponto de conexão no template onde sensores ou atuadores reais serão acoplados. Cada slot define:

| Campo | Descrição |
|-------|-----------|
| id | Identificador único do slot |
| name | Nome descritivo |
| allowedTypes | Tipos de sensor/atuador aceitos |
| geometry | Posição 3D no modelo |
| ui2d | Posição na visualização 2D |

### 4.2. Tipos de Capabilities

**Sensores (Entrada de Dados):**

| Categoria | Capabilities |
|-----------|--------------|
| Environmental | Temperature, Humidity, Pressure, Illuminance, CO2, PM2.5, Noise |
| Energy | Voltage, Current, Active Power, Reactive Power, Energy, Power Factor |
| Motion | Acceleration (X/Y/Z), Gyroscope, Orientation, Magnetometer |
| Security | Occupancy, Door Contact, Vibration, Tamper, Smoke |
| Location | Latitude, Longitude, Altitude, Speed |

**Atuadores (Saída de Comandos):**

| Categoria | Capabilities |
|-----------|--------------|
| Switching | Switch, Dimmer, Relay, PWM |
| HVAC | Target Temp, Mode, Fan Speed, Valve Position |
| Motors | Target Speed, Direction, Torque Limit, E-Stop |
| Lighting | CCT, RGB, Hue, Saturation |
| Industrial | Analog Output (0-10V, 4-20mA), Digital Output, PID Setpoint |

### 4.3. Mapeamento Slot → Sensor

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│   DEVICE TEMPLATE                    SENSOR TEMPLATES (Biblioteca)      │
│                                                                         │
│   ┌───────────────────┐              ┌───────────────────┐              │
│   │ Slot: temp_motor  │◄────────────►│ PT100 Temperature │              │
│   │ allowedTypes:     │              │ Capabilities:     │              │
│   │  ['temperature']  │              │  • Temperature °C │              │
│   └───────────────────┘              └───────────────────┘              │
│                                                                         │
│   ┌───────────────────┐              ┌───────────────────┐              │
│   │ Slot: vibracao    │◄────────────►│ Vibration Sensor  │              │
│   │ allowedTypes:     │              │ Capabilities:     │              │
│   │  ['vibration']    │              │  • Velocity mm/s  │              │
│   └───────────────────┘              └───────────────────┘              │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 5. Sistema de Regras

### 5.1. Conceito de Regras Embutidas

Regras embutidas são automações que viajam com o template. Quando o template é instanciado, as regras são automaticamente ativadas para a nova instância.

### 5.2. Estrutura de uma Regra

| Campo | Descrição |
|-------|-----------|
| id | Identificador único |
| name | Nome descritivo |
| description | Descrição da lógica |
| dsl | Expressão DSL da regra |
| status | active, inactive, draft |
| priority | Prioridade de execução (1-100) |
| conditions | Lista de condições |
| actions | Lista de ações |

### 5.3. DSL - Domain Specific Language

A DSL do CorGrid segue a sintaxe:

```
WHEN <condições>
THEN <ações>
```

**Exemplos:**

```dsl
# Regra simples de temperatura
WHEN temperature > 80
THEN ALERT "Temperatura elevada"

# Regra com operador temporal
WHEN flow_rate < 5 FOR 60 seconds
THEN ALERT "Fluxo baixo prolongado"
 AND SHUTDOWN pump

# Regra com múltiplas condições
WHEN pump_status == "on" AND flow_rate < 5 AND motor_current > 2
THEN ALERT "Possível falha de bomba"
```

### 5.4. Tipos de Condições

| Tipo | Descrição | Exemplo |
|------|-----------|---------|
| MQTT | Valor de tópico MQTT | `sensors/temp > 30` |
| STATE | Estado de device | `device.pump.status == "running"` |
| TIME | Condição temporal | `time.hour >= 22` |
| COMPOSITE | Combinação lógica | `A AND B`, `A OR B` |

### 5.5. Tipos de Ações

| Tipo | Descrição | Exemplo |
|------|-----------|---------|
| PUBLISH | Publicar em MQTT | `PUBLISH {"cmd":"off"} TO device/relay` |
| LOG | Registrar log | `LOG WARNING "Alerta"` |
| NOTIFY | Enviar notificação | `NOTIFY "Título" VIA email` |
| HTTP | Chamada HTTP | `HTTP POST /api/alert` |
| SET | Alterar estado | `SET device.status = "warning"` |

---

## 6. Operadores Temporais e Estatísticos

### 6.1. Operadores de Comparação

| Operador | Símbolo | Descrição |
|----------|---------|-----------|
| Equal | `==` | Igualdade |
| NotEqual | `!=` | Diferença |
| GreaterThan | `>` | Maior que |
| LessThan | `<` | Menor que |
| GreaterOrEqual | `>=` | Maior ou igual |
| LessOrEqual | `<=` | Menor ou igual |
| Between | `between` | Entre valores |
| In | `in` | Na lista |

### 6.2. Operadores Lógicos

| Operador | Descrição |
|----------|-----------|
| AND | Todas verdadeiras |
| OR | Pelo menos uma verdadeira |
| NOT | Negação |
| XOR | Exclusivo OR |

### 6.3. Operadores Temporais

| Operador | Sintaxe | Descrição |
|----------|---------|-----------|
| FOR | `condition FOR X seconds` | Condição mantida por período |
| SINCE | `condition SINCE timestamp` | Condição desde momento |
| WITHIN | `condition WITHIN X minutes` | Ocorreu no período |
| RATE | `RATE(value, period)` | Taxa de variação |
| DEBOUNCE | `DEBOUNCE(condition, X seconds)` | Anti-flicker |
| THROTTLE | `THROTTLE(action, X seconds)` | Limita frequência |

**Exemplos:**

```dsl
# Condição sustentada por tempo
WHEN pressure < 2 FOR 60 seconds
THEN ALERT "Pressão baixa prolongada"

# Taxa de variação
WHEN RATE(temperature, 5 minutes) > 2
THEN ALERT "Temperatura subindo rapidamente"

# Anti-flicker
WHEN DEBOUNCE(temperature > 80, 30 seconds)
THEN ALERT "Temperatura elevada confirmada"
```

### 6.4. Operadores Estatísticos

| Operador | Sintaxe | Descrição |
|----------|---------|-----------|
| AVG | `AVG(field, period)` | Média no período |
| MIN | `MIN(field, period)` | Mínimo no período |
| MAX | `MAX(field, period)` | Máximo no período |
| SUM | `SUM(field, period)` | Soma no período |
| COUNT | `COUNT(field, period)` | Contagem no período |
| STDDEV | `STDDEV(field, period)` | Desvio padrão |
| PERCENTILE | `PERCENTILE(field, p, period)` | Percentil |
| TREND | `TREND(field, period)` | Tendência (up/down/stable) |

**Exemplos:**

```dsl
# Média móvel
WHEN AVG(temperature, 15 minutes) > 75
THEN ALERT "Média elevada"

# Tendência
WHEN TREND(pressure, 30 minutes) == "down" AND pressure < 3
THEN ALERT "Pressão caindo"
```

---

## 7. Inferência AI/ML

### 7.1. Integração com Modelos

O CorGrid suporta integração de modelos ONNX nas regras para decisões baseadas em Machine Learning.

### 7.2. Operadores ML

| Operador | Sintaxe | Descrição |
|----------|---------|-----------|
| PREDICT | `PREDICT(model_id, features)` | Predição |
| CLASSIFY | `CLASSIFY(model_id, features)` | Classificação |
| ANOMALY | `ANOMALY(model_id, features)` | Score de anomalia |
| FORECAST | `FORECAST(model_id, field, horizon)` | Previsão futura |

### 7.3. Exemplos

```dsl
# Manutenção preditiva
WHEN PREDICT("bearing_failure", [temperature, vibration, rpm]) > 0.7
THEN NOTIFY "Manutenção necessária" VIA email

# Detecção de anomalia
WHEN ANOMALY("vibration_model", [vib_x, vib_y, vib_z]) > 0.85
THEN ALERT "Padrão anômalo detectado"

# Classificação de estado
WHEN CLASSIFY("machine_state", [temp, pressure, rpm]) == "degraded"
THEN SET device.health_status = "warning"
```

### 7.4. Modelos Disponíveis

| Modelo ID | Tipo | Aplicação |
|-----------|------|-----------|
| anomaly_generic | Isolation Forest | Anomalias genéricas |
| bearing_failure | Random Forest | Falha de rolamentos |
| thermal_stress | Gradient Boosting | Estresse térmico |
| vibration_pattern | CNN | Padrões de vibração |
| energy_forecast | LSTM | Consumo energético |

---

## 8. Formato de Armazenamento

### 8.1. Estrutura JSON

Templates são armazenados em formato JSON:

```json
{
  "id": "tmpl_gerador_500kva_001",
  "version": "1.0.0",
  "created_at": "2025-12-20T10:30:00Z",
  "updated_at": "2025-12-20T14:45:00Z",
  "metadata": {
    "name": "Gerador Diesel 500KVA",
    "manufacturer": "Cummins",
    "model": "C500D5",
    "category": "machine",
    "description": "Template para geradores diesel de 500KVA",
    "specs": {
      "power_kva": 500,
      "fuel_type": "diesel",
      "voltage": "380V",
      "frequency": "60Hz"
    }
  },
  "assets": {
    "image_2d": "/uploads/templates/gerador_2d.png",
    "model_3d": "/uploads/templates/gerador_3d.glb",
    "manual": "/uploads/templates/gerador_manual.pdf"
  },
  "camera": {
    "position": [5, 3, 5],
    "target": [0, 1, 0]
  },
  "slots": [
    {
      "id": "slot_temp_motor",
      "name": "Temperatura Motor",
      "allowedTypes": ["temperature"],
      "geometry": {
        "mesh_target": "motor_housing",
        "position": [0.5, 1.2, 0]
      }
    }
  ],
  "embedded_rules": [
    {
      "id": "rule_thermal_protection",
      "name": "Proteção Térmica",
      "dsl": "WHEN slot_temp_motor.value > 110 FOR 30 seconds THEN SHUTDOWN",
      "priority": 100,
      "enabled": true
    }
  ]
}
```

---

## 9. API REST

### 9.1. Endpoints de Templates

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| GET | `/api/v1/templates` | Listar templates |
| GET | `/api/v1/templates/:id` | Obter template |
| POST | `/api/v1/templates` | Criar template |
| PUT | `/api/v1/templates/:id` | Atualizar template |
| DELETE | `/api/v1/templates/:id` | Excluir template |
| POST | `/api/v1/templates/:id/instantiate` | Criar instância |
| GET | `/api/v1/templates/:id/instances` | Listar instâncias |

### 9.2. Endpoints de Regras

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| GET | `/api/v1/rules` | Listar regras |
| GET | `/api/v1/rules/:id` | Obter regra |
| POST | `/api/v1/rules` | Criar regra |
| PUT | `/api/v1/rules/:id` | Atualizar regra |
| DELETE | `/api/v1/rules/:id` | Excluir regra |
| POST | `/api/v1/rules/validate` | Validar DSL |
| POST | `/api/v1/rules/:id/test` | Testar regra |

---

## 10. Padronização e Escalabilidade

### 10.1. Convenções de Nomenclatura

| Elemento | Padrão | Exemplo |
|----------|--------|---------|
| Template ID | `tmpl_{categoria}_{modelo}_{seq}` | `tmpl_gerador_500kva_001` |
| Slot ID | `slot_{funcao}` | `slot_temp_motor` |
| Rule ID | `rule_{funcao}` | `rule_thermal_protection` |
| Instance ID | `inst_{template}_{seq}` | `inst_gerador_001` |

### 10.2. Versionamento

Templates suportam versionamento semântico:

```json
{
  "version": "1.2.0",
  "version_history": [
    { "version": "1.0.0", "date": "2025-01-15", "changes": "Versão inicial" },
    { "version": "1.1.0", "date": "2025-03-20", "changes": "Adicionado slot vibração" },
    { "version": "1.2.0", "date": "2025-06-10", "changes": "Regras ML" }
  ]
}
```

### 10.3. Herança de Templates

Templates podem herdar de templates base:

```json
{
  "id": "tmpl_gerador_1000kva",
  "extends": "tmpl_gerador_500kva_001",
  "override": {
    "metadata.specs.power_kva": 1000,
    "slots": [
      { "id": "slot_extra_temp", "name": "Temperatura Secundária" }
    ]
  }
}
```

### 10.4. Tags e Categorização

```json
{
  "tags": ["energia", "backup", "diesel", "industrial"],
  "categories": {
    "primary": "power_generation",
    "secondary": ["industrial", "backup_systems"]
  }
}
```

---

## 11. Roadmap

### 11.1. Funcionalidades Atuais

| Feature | Status |
|---------|--------|
| Criação de Templates | ✅ Implementado |
| Slots e Capabilities | ✅ Implementado |
| Regras com DSL | ✅ Implementado |
| Operadores Temporais (FOR) | ✅ Implementado |

### 11.2. Funcionalidades Planejadas

| Feature | Status | Prioridade |
|---------|--------|------------|
| Operadores Estatísticos (AVG, STDDEV) | 🔄 Em Progresso | Alta |
| Integração ONNX Runtime | 📋 Planejado | Alta |
| Regras Cross-Device | 📋 Planejado | Média |
| Template Marketplace | 📋 Planejado | Baixa |

---

**Versão:** 3.0
**Última Atualização:** 20/12/2025
**Mantenedor:** Core Architecture Team
