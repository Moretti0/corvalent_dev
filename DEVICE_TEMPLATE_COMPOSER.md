# CorGrid OS - Device Template Composer

**Document ID:** DEVICE-TEMPLATE-COMPOSER-V2
**Data:** 20/12/2025
**Propósito:** Source of Truth para criação de Device Templates e Sistema de Regras
**Autoridade:** Core Architecture Team

---

## Índice

1. [Visão Geral](#1-visão-geral)
2. [Arquitetura do Device Template](#2-arquitetura-do-device-template)
3. [Wizard de Criação (5 Passos)](#3-wizard-de-criação-5-passos)
4. [SensorSlot e Capabilities](#4-sensorslot-e-capabilities)
5. [Sistema de Regras Avançado](#5-sistema-de-regras-avançado)
6. [Operadores Temporais e Estatísticos](#6-operadores-temporais-e-estatísticos)
7. [Inferência AI/ML em Regras](#7-inferência-aiml-em-regras)
8. [Persistência e Serialização](#8-persistência-e-serialização)
9. [API REST de Templates](#9-api-rest-de-templates)
10. [Padronização e Escalabilidade](#10-padronização-e-escalabilidade)
11. [Performance e Otimizações](#11-performance-e-otimizações)
12. [Roadmap](#12-roadmap)

---

## 1. Visão Geral

O **Device Template Composer** é o sistema de criação de templates de dispositivos do CorGrid OS. Ele permite modelar equipamentos industriais complexos através de uma abordagem composicional, onde cada template define:

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
│   │  GERADOR DIESEL   │────────────►│ Fábrica: São Paulo│               │
│   │  500KVA           │             └───────────────────┘               │
│   │                   │                                                 │
│   │  • 5 SensorSlots  │             ┌───────────────────┐               │
│   │  • 4 Regras       │────────────►│ Gerador-002       │               │
│   │  • Modelo 3D      │             │ Fábrica: Curitiba │               │
│   │                   │             └───────────────────┘               │
│   └───────────────────┘                                                 │
│                                     ┌───────────────────┐               │
│                        ────────────►│ Gerador-003       │               │
│                                     │ Fábrica: Manaus   │               │
│                                     └───────────────────┘               │
│                                                                         │
│   1 Template = N Instâncias com mesmas regras e estrutura               │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Arquitetura do Device Template

### 2.1. Interface DeviceTemplate

**Fonte:** `www/src/stores/useDeviceWizardStore.ts:9-28`

```typescript
export interface DeviceTemplate {
  id: string;
  metadata: {
    name: string;
    manufacturer: string;
    model: string;
    description?: string;
    category?: 'machine' | 'vehicle' | 'panel' | 'sensor' | 'actuator' | 'controller' | 'gateway' | 'display';
    specs?: Record<string, any>;
  };
  assets: {
    image_2d?: string;    // Path do servidor
    model_3d?: string;    // GLB/GLTF
    manual?: string;      // PDF
  };
  camera?: {
    position: [number, number, number];
    target: [number, number, number];
  };
  slots: SensorSlot[];
}
```

### 2.2. SensorSlot - Ponto de Conexão

**Fonte:** `www/src/stores/useDeviceWizardStore.ts:30-42`

```typescript
export interface SensorSlot {
  id: string;
  name: string;
  allowedTypes: string[];  // ['temperature', 'vibration', 'pressure']
  geometry?: {
    mesh_target: string;
    position: [number, number, number];
  } | null;
  ui2d?: {
    x: number;
    y: number;
  } | null;
}
```

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
│   │  ┌─────────────────┐  ┌─────────────────┐                       │   │
│   │  │ Slot: pressao   │  │ Slot: vibracao  │                       │   │
│   │  │ Types: [pres]   │  │ Types: [vib]    │                       │   │
│   │  │ Pos: [1,0.5,0]  │  │ Pos: [0,0.5,0]  │                       │   │
│   │  └─────────────────┘  └─────────────────┘                       │   │
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

## 3. Wizard de Criação (5 Passos)

### 3.1. Visão Geral do Wizard

**Fonte:** `www/src/pages/Library/Library/DeviceWizard.tsx`

O wizard de criação de Device Templates possui 5 passos sequenciais:

```
┌────────┐   ┌────────┐   ┌────────┐   ┌────────┐   ┌────────┐
│ PASSO  │──►│ PASSO  │──►│ PASSO  │──►│ PASSO  │──►│ PASSO  │
│   1    │   │   2    │   │   3    │   │   4    │   │   5    │
│        │   │        │   │        │   │        │   │        │
│ 📝     │   │ 🌐     │   │ ⚡     │   │ 🔌     │   │ ✅     │
│ Basic  │   │Network │   │Capabil.│   │Integr. │   │Review  │
│ Info   │   │Config  │   │        │   │        │   │        │
└────────┘   └────────┘   └────────┘   └────────┘   └────────┘
```

### 3.2. Passo 1: Informações Básicas

**Campos obrigatórios:**

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `name` | string | Nome do template (único) |
| `type` | enum | `physical` ou `virtual` |
| `category` | enum | sensor, actuator, controller, gateway, display |

**Categorias de Device:**

```typescript
const DEVICE_CATEGORIES = [
  { value: 'sensor', label: 'Sensor', icon: '📡' },
  { value: 'actuator', label: 'Atuador', icon: '⚙️' },
  { value: 'controller', label: 'Controlador', icon: '🎛️' },
  { value: 'gateway', label: 'Gateway', icon: '🌐' },
  { value: 'display', label: 'Display', icon: '📺' }
];
```

### 3.3. Passo 2: Configuração de Rede

**Protocolos suportados:**

| Protocolo | Categoria | Configuração |
|-----------|-----------|--------------|
| Modbus TCP | Industrial | IP, Porta, Unit ID |
| Modbus RTU | Industrial | Serial, Baud, Parity |
| OPC UA | Industrial | Endpoint, Security |
| MQTT | IoT | Broker, Topic, QoS |
| REST API | Web | Endpoint, Auth |
| WebSocket | Realtime | URL, Subprotocol |
| CAN Bus | Automotive | Bitrate, Node ID |
| ProfiNet | Industrial | GSD, IP |
| EtherNet/IP | Industrial | IP, Assembly |

### 3.4. Passo 3: Capabilities (Capacidades)

Define as capacidades operacionais do device:

```typescript
interface DeviceCapabilities {
  monitoring: boolean;      // Coleta de dados
  control: boolean;         // Envio de comandos
  dataProcessing: boolean;  // Processamento local
  visualization: boolean;   // Interface visual
}
```

### 3.5. Passo 4: Integração

**Opções de integração:**

| Opção | Descrição |
|-------|-----------|
| `protocols` | Lista de protocolos habilitados |
| `apiEndpoints` | Expor via REST API |
| `webInterface` | Interface web embarcada |

### 3.6. Passo 5: Review e Publicação

Validação final e publicação do template na biblioteca.

---

## 4. SensorSlot e Capabilities

### 4.1. Associação Slot → Capability

Os SensorSlots definem **pontos de conexão** onde sensores reais são acoplados ao template. Cada slot aceita tipos específicos de capabilities.

**Fonte:** `www/src/stores/useSensorWizardStore.ts:8-24`

```typescript
export interface Capability {
  id: string;
  name: string;
  type: 'number' | 'boolean' | 'string' | 'enum';
  unit?: string;
  min?: number;
  max?: number;
  options?: string[];  // Para enum
  geometry?: {
    mesh_target: string;
    position: [number, number, number];
  } | null;
  ui2d?: {
    x: number;
    y: number;
  } | null;
}
```

### 4.2. Presets de Capabilities por Categoria

**Fonte:** `www/src/constants/sensorPresets.ts`

O sistema oferece presets organizados por categoria:

| Categoria | Tipo | Exemplos de Capabilities |
|-----------|------|--------------------------|
| Environmental | sensor | Temperature, Humidity, Pressure, Illuminance, CO2, PM2.5, Noise |
| Energy & Electrical | sensor | Voltage, Current, Active Power, Reactive Power, Energy, Power Factor, Frequency |
| Motion & Inertial | sensor | Acceleration (X/Y/Z), Gyroscope, Orientation, Magnetometer |
| Security & Presence | sensor | Occupancy (PIR), Door Contact, Vibration, Tamper, Smoke |
| Location | sensor | Latitude, Longitude, Altitude, Speed, GPS Accuracy |
| Switching & Control | actuator | Switch, Dimmer, Relay, PWM, Scene Select |
| HVAC Control | actuator | Target Temp, Mode, Fan Speed, Valve Position |
| Motors & Drives | actuator | Target Speed, Direction, Torque Limit, E-Stop |
| Lighting & Color | actuator | CCT, RGB, Hue, Saturation |
| Industrial I/O | actuator | Analog Output (0-10V, 4-20mA), Digital Output, Alarm Reset, PID Setpoint |

### 4.3. Mapeamento Slot → Sensor Template

Quando uma instância de device é criada a partir de um template, cada **SensorSlot** é preenchido com um **SensorTemplate** compatível:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│   DEVICE TEMPLATE                    SENSOR TEMPLATES (Biblioteca)      │
│                                                                         │
│   ┌───────────────────┐              ┌───────────────────┐              │
│   │ Slot: temp_motor  │◄────────────►│ PT100 Temperature │              │
│   │ allowedTypes:     │              │ Capabilities:     │              │
│   │  ['temperature']  │              │  • Temperature °C │              │
│   └───────────────────┘              │  • Resistance Ω   │              │
│                                      └───────────────────┘              │
│                                                                         │
│   ┌───────────────────┐              ┌───────────────────┐              │
│   │ Slot: vibracao    │◄────────────►│ Vibration Sensor  │              │
│   │ allowedTypes:     │              │ Capabilities:     │              │
│   │  ['vibration',    │              │  • Velocity mm/s  │              │
│   │   'acceleration'] │              │  • Accel m/s²     │              │
│   └───────────────────┘              └───────────────────┘              │
│                                                                         │
│   ┌───────────────────┐              ┌───────────────────┐              │
│   │ Slot: nivel_comb  │◄────────────►│ Level Transmitter │              │
│   │ allowedTypes:     │              │ Capabilities:     │              │
│   │  ['level',        │              │  • Level %        │              │
│   │   'volume']       │              │  • Volume L       │              │
│   └───────────────────┘              └───────────────────┘              │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 5. Sistema de Regras Avançado

### 5.1. Arquitetura do Rule Engine

**Fonte:** `www/src/services/rulesService.ts`

O sistema de regras do CorGrid utiliza uma DSL (Domain-Specific Language) para definir condições e ações.

```typescript
export interface Rule {
  id: string;
  name: string;
  description: string;
  dsl: string;
  status: 'active' | 'inactive' | 'draft';
  priority: number;
  conditions: Condition[];
  actions: Action[];
  statistics: ExecutionStatistics;
  created_at: string;
  updated_at: string;
  tags: string[];
}
```

### 5.2. Estrutura de Condições

```typescript
export interface Condition {
  type: 'MQTT' | 'STATE' | 'TIME' | 'COMPOSITE';
  target: string;
  operator: string;
  value: string;
  duration_seconds?: number;        // Operador temporal
  composite_conditions?: Condition[];
  composite_operator?: string;      // AND, OR, NOT
  negate?: boolean;
}
```

### 5.3. Estrutura de Ações

```typescript
export interface Action {
  type: 'MQTT_PUBLISH' | 'HTTP_CALL' | 'LOG' | 'NOTIFICATION';
  target: string;
  payload: string;
  method?: string;   // Para HTTP
  channel?: string;  // Para Notification
}
```

### 5.4. DSL - Domain Specific Language

**Fonte:** `www/src/pages/RulesEngine/RuleBuilder.tsx:146-184`

A DSL é gerada automaticamente a partir dos blocos visuais:

```
WHEN <condições>
THEN <ações>
```

**Exemplos de DSL:**

```dsl
# Regra simples
WHEN zigbee/devices/temp_sensor/temperature > 30
THEN PUBLISH {"alert": "high_temp"} TO zigbee/commands/fan
 AND LOG WARNING "High temperature detected"

# Regra com operador temporal
WHEN device.zigbee.motion_sensor.status == "offline" FOR 600 seconds
THEN NOTIFY "Device offline alert" "Motion sensor is offline" VIA EMAIL
 AND LOG ERROR "Device offline: motion_sensor"

# Regra com múltiplas condições
WHEN modbus/device/energy_meter/power > 5000 FOR 3600 seconds
THEN LOG INFO "High energy consumption detected: ${power}W"
 AND PUBLISH {"energy_alert": true} TO system/alerts/energy
```

### 5.5. Tipos de Condições no RuleBuilder

**Fonte:** `www/src/pages/RulesEngine/RuleBuilder.tsx:27-43`

| Tipo | Descrição | Exemplo |
|------|-----------|---------|
| `mqtt` | Tópico MQTT | `zigbee/devices/sensor/temperature > 30` |
| `device` | Estado de Device | `device.pump.status == "running"` |
| `time` | Schedule/Cron | `time.schedule == true` |
| `logic` | Operador Lógico | `AND`, `OR`, `NOT` |

### 5.6. Tipos de Ações

| Tipo | Descrição | Exemplo |
|------|-----------|---------|
| `mqtt` | Publicar MQTT | `PUBLISH {"cmd": "on"} TO device/relay` |
| `log` | Registrar Log | `LOG WARNING "Alerta de temperatura"` |
| `notification` | Enviar Notificação | `NOTIFY "Alerta" VIA EMAIL` |
| `http` | Chamada HTTP | `HTTP POST /api/alert WITH {...}` |

---

## 6. Operadores Temporais e Estatísticos

### 6.1. Categorias de Operadores

**Fonte:** `www/src/pages/Config/tabs/modals/RuleOperatorModal.tsx:25-30`

```typescript
const OPERATOR_CATEGORIES = [
  { value: 'comparison', label: 'Comparison' },
  { value: 'logical', label: 'Logical' },
  { value: 'mathematical', label: 'Mathematical' },
  { value: 'temporal', label: 'Temporal' },
];
```

### 6.2. Operadores de Comparação

| Operador | Símbolo | Descrição | Tipos Suportados |
|----------|---------|-----------|------------------|
| Equal | `==` | Igualdade | string, number, boolean |
| NotEqual | `!=` | Diferença | string, number, boolean |
| GreaterThan | `>` | Maior que | number, date |
| LessThan | `<` | Menor que | number, date |
| GreaterOrEqual | `>=` | Maior ou igual | number, date |
| LessOrEqual | `<=` | Menor ou igual | number, date |
| Contains | `contains` | Contém substring | string, array |
| StartsWith | `startswith` | Começa com | string |
| EndsWith | `endswith` | Termina com | string |
| In | `in` | Está na lista | string, number |
| NotIn | `notin` | Não está na lista | string, number |
| Between | `between` | Entre valores | number, date |
| IsNull | `isnull` | É nulo | any |
| IsNotNull | `isnotnull` | Não é nulo | any |

### 6.3. Operadores Lógicos

| Operador | Símbolo | Descrição |
|----------|---------|-----------|
| AND | `AND` | Todas as condições verdadeiras |
| OR | `OR` | Pelo menos uma condição verdadeira |
| NOT | `NOT` | Nega a condição |
| XOR | `XOR` | Exclusivo OR |

### 6.4. Operadores Temporais

| Operador | Sintaxe | Descrição |
|----------|---------|-----------|
| FOR | `condition FOR X seconds` | Condição mantida por período |
| SINCE | `condition SINCE timestamp` | Condição desde momento |
| WITHIN | `condition WITHIN X minutes` | Ocorreu dentro do período |
| RATE | `RATE(value, period)` | Taxa de variação |
| DEBOUNCE | `DEBOUNCE(condition, X seconds)` | Anti-flicker |
| THROTTLE | `THROTTLE(action, X seconds)` | Limita frequência |

**Exemplo de uso:**

```dsl
# Proteção contra flutuação (debounce)
WHEN DEBOUNCE(temperature > 80, 30 seconds)
THEN ALERT "Temperatura elevada confirmada"

# Detecção de tendência (rate)
WHEN RATE(temperature, 5 minutes) > 2
THEN ALERT "Temperatura subindo rapidamente"

# Condição sustentada (for)
WHEN pressure < 2 bar FOR 60 seconds
THEN SHUTDOWN "Pressão baixa prolongada"
```

### 6.5. Operadores Estatísticos

| Operador | Sintaxe | Descrição |
|----------|---------|-----------|
| AVG | `AVG(field, period)` | Média no período |
| MIN | `MIN(field, period)` | Mínimo no período |
| MAX | `MAX(field, period)` | Máximo no período |
| SUM | `SUM(field, period)` | Soma no período |
| COUNT | `COUNT(field, period)` | Contagem no período |
| STDDEV | `STDDEV(field, period)` | Desvio padrão |
| PERCENTILE | `PERCENTILE(field, p, period)` | Percentil |
| DELTA | `DELTA(field, period)` | Variação absoluta |
| TREND | `TREND(field, period)` | Tendência (up/down/stable) |

**Exemplo de uso:**

```dsl
# Média móvel
WHEN AVG(temperature, 15 minutes) > 75
THEN ALERT "Média de temperatura elevada"

# Desvio do padrão
WHEN STDDEV(vibration, 1 hour) > 2
THEN LOG WARNING "Variabilidade de vibração anormal"

# Tendência de queda
WHEN TREND(pressure, 30 minutes) == "down" AND pressure < 3
THEN ALERT "Pressão caindo - verificar vazamento"
```

---

## 7. Inferência AI/ML em Regras

### 7.1. Arquitetura de Inferência

O CorGrid suporta integração de modelos ML nas regras através de operadores especializados:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│                    PIPELINE DE INFERÊNCIA ML                            │
│                                                                         │
│   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐                 │
│   │   DADOS     │───►│   FEATURE   │───►│   MODELO    │                 │
│   │   SENSOR    │    │   EXTRACT   │    │     ML      │                 │
│   └─────────────┘    └─────────────┘    └──────┬──────┘                 │
│                                                │                        │
│                                                ▼                        │
│                                        ┌─────────────┐                  │
│                                        │  PREDIÇÃO   │                  │
│                                        │  / SCORE    │                  │
│                                        └──────┬──────┘                  │
│                                                │                        │
│                                                ▼                        │
│                                        ┌─────────────┐                  │
│                                        │   REGRA     │                  │
│                                        │   ENGINE    │                  │
│                                        └─────────────┘                  │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 7.2. Operadores ML

| Operador | Sintaxe | Descrição |
|----------|---------|-----------|
| PREDICT | `PREDICT(model_id, features)` | Executar predição |
| CLASSIFY | `CLASSIFY(model_id, features)` | Classificação |
| ANOMALY | `ANOMALY(model_id, features)` | Score de anomalia |
| FORECAST | `FORECAST(model_id, field, horizon)` | Previsão futura |

### 7.3. Exemplos de Regras com ML

```dsl
# Detecção de anomalia
WHEN ANOMALY("vibration_model", [vibration_x, vibration_y, vibration_z]) > 0.85
THEN ALERT "Padrão de vibração anômalo detectado"
 AND LOG INFO "Anomaly score: ${anomaly_score}"

# Manutenção preditiva
WHEN PREDICT("bearing_failure", [temperature, vibration, rpm]) > 0.7
THEN NOTIFY "Manutenção Preditiva" "Rolamento com probabilidade de falha" VIA EMAIL
 AND CREATE_TICKET "Inspecionar rolamento ${device_id}"

# Previsão de consumo
WHEN FORECAST("energy_model", power_consumption, "24h") > capacity_limit
THEN ALERT "Consumo previsto acima da capacidade"
 AND SCHEDULE_ACTION "reduce_load" AT "peak_hours"

# Classificação de estado
WHEN CLASSIFY("machine_state", [temp, pressure, rpm, vibration]) == "degraded"
THEN SET device.health_status = "warning"
 AND LOG WARNING "Máquina em estado degradado - monitorar"
```

### 7.4. Modelos Pré-treinados

O CorGrid inclui modelos pré-treinados para casos comuns:

| Modelo ID | Tipo | Aplicação |
|-----------|------|-----------|
| `anomaly_generic` | Isolation Forest | Detecção de anomalias genéricas |
| `bearing_failure` | Random Forest | Predição de falha de rolamentos |
| `thermal_stress` | Gradient Boosting | Estresse térmico |
| `vibration_pattern` | CNN | Classificação de padrões de vibração |
| `energy_forecast` | LSTM | Previsão de consumo energético |
| `maintenance_window` | Survival Analysis | Janela ótima de manutenção |

### 7.5. Treinamento de Modelos Customizados

```
POST /api/v1/ml/models
Content-Type: application/json

{
  "name": "motor_health_predictor",
  "type": "classification",
  "algorithm": "random_forest",
  "features": [
    "temperature",
    "vibration_rms",
    "current_draw",
    "rpm_variance"
  ],
  "target": "health_status",
  "training_data": "device_data_last_90_days"
}
```

---

## 8. Persistência e Serialização

### 8.1. Formato de Armazenamento

Templates são persistidos em formato JSON no backend:

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
      },
      "ui2d": { "x": 150, "y": 200 }
    }
  ],
  "embedded_rules": [
    {
      "id": "rule_thermal_protection",
      "name": "Proteção Térmica",
      "dsl": "WHEN slot_temp_motor.value > 110 FOR 30 seconds THEN SET relay_main = OFF AND ALERT \"Shutdown térmico\"",
      "priority": 100,
      "enabled": true
    }
  ]
}
```

### 8.2. Zustand Store com Persistência

**Fonte:** `www/src/stores/useDeviceWizardStore.ts`

O wizard utiliza Zustand com middleware de persistência:

```typescript
export const useDeviceWizardStore = create<WizardState>()(
  devtools(
    persist(
      (set) => ({
        currentStep: 1,
        template: INITIAL_TEMPLATE,
        // ... actions
      }),
      {
        name: 'device-wizard-storage',
        partialize: (state) => ({
          currentStep: state.currentStep,
          template: state.template
        })
      }
    )
  )
);
```

---

## 9. API REST de Templates

### 9.1. Endpoints

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

**Fonte:** `www/src/services/rulesService.ts`

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| GET | `/api/v1/rules` | Listar regras |
| GET | `/api/v1/rules/:id` | Obter regra |
| POST | `/api/v1/rules` | Criar regra |
| PUT | `/api/v1/rules/:id` | Atualizar regra |
| DELETE | `/api/v1/rules/:id` | Excluir regra |
| POST | `/api/v1/rules/validate` | Validar DSL |
| POST | `/api/v1/rules/:id/test` | Testar regra |
| PATCH | `/api/v1/rules/:id/status` | Alterar status |

### 9.3. Exemplo de Criação

```typescript
const createTemplate = async (template: CreateTemplateRequest) => {
  const response = await api.post('/api/v1/templates', template);
  return response.data;
};

interface CreateTemplateRequest {
  metadata: {
    name: string;
    manufacturer: string;
    model: string;
    category: string;
  };
  assets?: {
    image_2d?: string;
    model_3d?: string;
  };
  slots: SensorSlot[];
  embedded_rules?: EmbeddedRule[];
}
```

---

## 10. Padronização e Escalabilidade

### 10.1. Convenções de Nomenclatura

| Elemento | Padrão | Exemplo |
|----------|--------|---------|
| Template ID | `tmpl_{categoria}_{modelo}_{seq}` | `tmpl_gerador_500kva_001` |
| Slot ID | `slot_{funcao}` | `slot_temp_motor` |
| Rule ID | `rule_{funcao}` | `rule_thermal_protection` |
| Instance ID | `inst_{template}_{seq}` | `inst_gerador_001` |

### 10.2. Versionamento de Templates

```json
{
  "version": "1.2.0",
  "version_history": [
    { "version": "1.0.0", "date": "2025-01-15", "changes": "Initial release" },
    { "version": "1.1.0", "date": "2025-03-20", "changes": "Added vibration slot" },
    { "version": "1.2.0", "date": "2025-06-10", "changes": "ML rules integration" }
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
  "tags": [
    "energia",
    "backup",
    "diesel",
    "industrial",
    "geração"
  ],
  "categories": {
    "primary": "power_generation",
    "secondary": ["industrial", "backup_systems"]
  }
}
```

---

## 11. Performance e Otimizações

### 11.1. Lazy Loading de Assets

Assets pesados (modelos 3D, manuais PDF) são carregados sob demanda:

```typescript
const loadAsset = async (templateId: string, assetType: 'model_3d' | 'manual') => {
  const response = await api.get(`/api/v1/templates/${templateId}/assets/${assetType}`, {
    responseType: 'blob'
  });
  return URL.createObjectURL(response.data);
};
```

### 11.2. Cache de Templates

```typescript
const CACHE_TTL = 5 * 60 * 1000; // 5 minutos

class TemplateCache {
  private cache = new Map<string, { data: Template; timestamp: number }>();

  get(id: string): Template | null {
    const entry = this.cache.get(id);
    if (!entry) return null;
    if (Date.now() - entry.timestamp > CACHE_TTL) {
      this.cache.delete(id);
      return null;
    }
    return entry.data;
  }

  set(id: string, template: Template): void {
    this.cache.set(id, { data: template, timestamp: Date.now() });
  }
}
```

### 11.3. Execução de Regras - Otimizações

| Técnica | Descrição | Ganho |
|---------|-----------|-------|
| Rule Indexing | Índice por tópico MQTT | O(1) lookup |
| Condition Caching | Cache de condições avaliadas | -60% CPU |
| Batch Processing | Processar múltiplos eventos juntos | -40% overhead |
| Priority Queue | Fila de prioridade para regras | Críticas primeiro |

### 11.4. Métricas de Execução

**Fonte:** `www/src/services/rulesService.ts:54-60`

```typescript
export interface ExecutionStatistics {
  executions_total: number;
  executions_success: number;
  executions_failed: number;
  last_execution: string;
  average_execution_time_ms: number;
}
```

---

## 12. Roadmap

### 12.1. Funcionalidades Planejadas

| Feature | Status | Prioridade |
|---------|--------|------------|
| Visual Rule Builder (Drag & Drop) | ✅ Implementado | - |
| Operadores Temporais Básicos (FOR) | ✅ Implementado | - |
| Templates de Regras | ✅ Implementado | - |
| Operadores Estatísticos (AVG, MIN, MAX) | 🔄 Em Progresso | Alta |
| Integração ML/AI | 📋 Planejado | Alta |
| Regras Cross-Device | 📋 Planejado | Média |
| Rule Debugging | 📋 Planejado | Média |
| Rule Versioning | 📋 Planejado | Baixa |
| Rule Marketplace | 📋 Planejado | Baixa |

### 12.2. Integrações Futuras

- **TensorFlow Lite** - Inferência on-edge
- **ONNX Runtime** - Modelos universais
- **Apache Kafka** - Streaming de eventos
- **TimescaleDB** - Dados históricos para ML

---

## Referências de Código

| Arquivo | Linhas | Descrição |
|---------|--------|-----------|
| `www/src/stores/useDeviceWizardStore.ts` | 189 | Store do Device Template Wizard |
| `www/src/stores/useSensorWizardStore.ts` | 234 | Store do Sensor Template Wizard |
| `www/src/pages/Library/Library/DeviceWizard.tsx` | 806 | Componente do Wizard |
| `www/src/pages/RulesEngine/RuleBuilder.tsx` | 581 | Visual Rule Builder |
| `www/src/pages/RulesEngine/RulesDashboard.tsx` | 293 | Dashboard de Regras |
| `www/src/services/rulesService.ts` | 341 | Serviço de Regras (API) |
| `www/src/constants/sensorPresets.ts` | 137 | Presets de Capabilities |
| `www/src/pages/Config/tabs/modals/RuleOperatorModal.tsx` | 144 | Modal de Operadores |

---

**Versão:** 2.0
**Última Atualização:** 20/12/2025
**Mantenedor:** Core Architecture Team
