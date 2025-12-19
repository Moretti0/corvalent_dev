# 🔧 CorGrid OS - Device Template Composer

```
╔═══════════════════════════════════════════════════════════════════════════════╗
║                                                                               ║
║    ██████╗ ███████╗██╗   ██╗██╗ ██████╗███████╗                               ║
║    ██╔══██╗██╔════╝██║   ██║██║██╔════╝██╔════╝                               ║
║    ██║  ██║█████╗  ██║   ██║██║██║     █████╗                                 ║
║    ██║  ██║██╔══╝  ╚██╗ ██╔╝██║██║     ██╔══╝                                 ║
║    ██████╔╝███████╗ ╚████╔╝ ██║╚██████╗███████╗                               ║
║    ╚═════╝ ╚══════╝  ╚═══╝  ╚═╝ ╚═════╝╚══════╝                               ║
║                                                                               ║
║     ████████╗███████╗███╗   ███╗██████╗ ██╗      █████╗ ████████╗███████╗     ║
║     ╚══██╔══╝██╔════╝████╗ ████║██╔══██╗██║     ██╔══██╗╚══██╔══╝██╔════╝     ║
║        ██║   █████╗  ██╔████╔██║██████╔╝██║     ███████║   ██║   █████╗       ║
║        ██║   ██╔══╝  ██║╚██╔╝██║██╔═══╝ ██║     ██╔══██║   ██║   ██╔══╝       ║
║        ██║   ███████╗██║ ╚═╝ ██║██║     ███████╗██║  ██║   ██║   ███████╗     ║
║        ╚═╝   ╚══════╝╚═╝     ╚═╝╚═╝     ╚══════╝╚═╝  ╚═╝   ╚═╝   ╚══════╝     ║
║                                                                               ║
║          COMPOSER - Construa Equipamentos Industriais Complexos              ║
║                                                                               ║
╚═══════════════════════════════════════════════════════════════════════════════╝
```

---

## 📊 Visão Executiva

| Característica | Especificação |
|----------------|---------------|
| **Conceito** | Composição Hierárquica de Devices (Nested Templates) |
| **Profundidade** | Até 10 níveis de aninhamento |
| **Componentes por Device** | Ilimitado |
| **Regras por Template** | Até 100 regras embutidas |
| **Herança de Monitoramento** | Automática (pai herda status dos filhos) |
| **Formato** | Industrial Digital Twin Model (.idtm) |
| **API** | REST criptografada (TLS 1.3 + AES-256-GCM) |

---

## 🎯 O Conceito: Devices Compostos

No mundo industrial real, equipamentos complexos são compostos por múltiplos sub-componentes. O **Device Template Composer** permite modelar essa realidade de forma nativa.

### Paradigma Tradicional vs CorGrid

```
╔═══════════════════════════════════════════════════════════════════════════════╗
║                                                                               ║
║   ❌ ABORDAGEM TRADICIONAL                                                    ║
║   ────────────────────────                                                    ║
║                                                                               ║
║   ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐       ║
║   │  Motor   │  │  Bomba   │  │  Relé    │  │ Sensor   │  │ Gerador  │       ║
║   │          │  │          │  │          │  │  Nível   │  │  Diesel  │       ║
║   └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘       ║
║        │             │             │             │             │              ║
║        └─────────────┴─────────────┴─────────────┴─────────────┘              ║
║                                    │                                          ║
║                          ┌─────────▼─────────┐                                ║
║                          │   LÓGICA EXTERNA  │                                ║
║                          │  (Código Custom)  │                                ║
║                          └───────────────────┘                                ║
║                                                                               ║
║   Problemas:                                                                  ║
║   • Devices são "ilhas" sem relação estrutural                               ║
║   • Regras de correlação ficam em código externo                             ║
║   • Dificuldade de replicar configurações                                    ║
║   • Manutenção complexa e propensa a erros                                   ║
║                                                                               ║
╠═══════════════════════════════════════════════════════════════════════════════╣
║                                                                               ║
║   ✅ ABORDAGEM CORGRID - DEVICES COMPOSTOS                                    ║
║   ────────────────────────────────────────                                    ║
║                                                                               ║
║                    ┌─────────────────────────────────────┐                    ║
║                    │        🏭 GERADOR A DIESEL          │                    ║
║                    │         (Device Composto)           │                    ║
║                    │                                     │                    ║
║                    │  ┌───────────────────────────────┐  │                    ║
║                    │  │         COMPONENTES           │  │                    ║
║                    │  │                               │  │                    ║
║                    │  │  ┌─────┐ ┌─────┐ ┌─────┐     │  │                    ║
║                    │  │  │⚡   │ │💧   │ │🔌   │     │  │                    ║
║                    │  │  │Motor│ │Bomba│ │Relé │     │  │                    ║
║                    │  │  └──┬──┘ └──┬──┘ └──┬──┘     │  │                    ║
║                    │  │     │       │       │        │  │                    ║
║                    │  │     └───────┼───────┘        │  │                    ║
║                    │  │             │                │  │                    ║
║                    │  │         ┌───┴───┐            │  │                    ║
║                    │  │         │📊     │            │  │                    ║
║                    │  │         │Sensor │            │  │                    ║
║                    │  │         │Nível  │            │  │                    ║
║                    │  │         └───────┘            │  │                    ║
║                    │  └───────────────────────────────┘  │                    ║
║                    │                                     │                    ║
║                    │  ┌───────────────────────────────┐  │                    ║
║                    │  │      REGRAS EMBUTIDAS         │  │                    ║
║                    │  │  • Nível baixo → Parar bomba  │  │                    ║
║                    │  │  • Motor quente → Alerta      │  │                    ║
║                    │  │  • Falha relé → Emergência    │  │                    ║
║                    │  └───────────────────────────────┘  │                    ║
║                    │                                     │                    ║
║                    └─────────────────────────────────────┘                    ║
║                                                                               ║
║   Benefícios:                                                                 ║
║   • Estrutura reflete a realidade física                                     ║
║   • Regras viajam com o template                                             ║
║   • Replicação: 1 template = N geradores idênticos                           ║
║   • Manutenção centralizada no template                                      ║
║                                                                               ║
╚═══════════════════════════════════════════════════════════════════════════════╝
```

---

## 🏗️ Arquitetura de Composição

### Hierarquia de Aninhamento

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│   📐 NÍVEIS DE COMPOSIÇÃO                                                   │
│   ───────────────────────                                                   │
│                                                                             │
│   NÍVEL 0 (Raiz)                                                            │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                    🏭 PLANTA INDUSTRIAL                              │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                       │                                     │
│   NÍVEL 1 (Sistemas)                  │                                     │
│   ┌───────────────────┬───────────────┴───────────────┬─────────────────┐  │
│   │                   │                               │                 │  │
│   ▼                   ▼                               ▼                 │  │
│   ┌─────────┐   ┌─────────┐                     ┌─────────┐             │  │
│   │Sistema  │   │Sistema  │        ...          │Sistema  │             │  │
│   │Elétrico │   │Hidráuli.│                     │HVAC     │             │  │
│   └────┬────┘   └────┬────┘                     └────┬────┘             │  │
│        │             │                               │                  │  │
│   NÍVEL 2 (Equipamentos)                             │                  │  │
│   ┌────┴────┐   ┌────┴────┐                     ┌────┴────┐             │  │
│   │Gerador  │   │ Bomba   │                     │Chiller  │             │  │
│   │ Diesel  │   │Principal│                     │Central  │             │  │
│   └────┬────┘   └────┬────┘                     └────┬────┘             │  │
│        │             │                               │                  │  │
│   NÍVEL 3 (Componentes)                              │                  │  │
│   ┌────┴────┬────────┴──────┬─────────┐        ┌────┴────┐             │  │
│   │  Motor  │  Alternador   │ Painel  │        │Compres. │             │  │
│   │ Diesel  │   Elétrico    │Controle │        │         │             │  │
│   └────┬────┘  └────────────┘ └───────┘        └─────────┘             │  │
│        │                                                                │  │
│   NÍVEL 4 (Sensores/Atuadores)                                          │  │
│   ┌────┴────┬────────────┬────────────┐                                 │  │
│   │Sensor   │  Sensor    │  Sensor    │                                 │  │
│   │Temp.    │  Rotação   │  Vibração  │                                 │  │
│   └─────────┘  └──────────┘  └─────────┘                                │  │
│                                                                             │
│   ✨ Até 10 níveis de profundidade suportados                              │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 🔧 Exemplo Prático: Gerador a Diesel

### Estrutura Completa do Template

```
╔═══════════════════════════════════════════════════════════════════════════════╗
║                                                                               ║
║   🏭 TEMPLATE: GERADOR A DIESEL 500KVA                                        ║
║   ═══════════════════════════════════                                         ║
║                                                                               ║
║   ┌─────────────────────────────────────────────────────────────────────────┐║
║   │                                                                         │║
║   │   METADADOS                                                             │║
║   │   ─────────                                                             │║
║   │   Nome: Gerador Diesel 500KVA Industrial                                │║
║   │   Fabricante: Cummins / Caterpillar                                     │║
║   │   Categoria: Equipamento de Geração                                     │║
║   │   Tags: energia, backup, diesel, industrial                             │║
║   │                                                                         │║
║   └─────────────────────────────────────────────────────────────────────────┘║
║                                                                               ║
║   ┌─────────────────────────────────────────────────────────────────────────┐║
║   │                                                                         │║
║   │   COMPONENTES ANINHADOS                                                 │║
║   │   ─────────────────────                                                 │║
║   │                                                                         │║
║   │   gerador_diesel_500kva/                                                │║
║   │   │                                                                     │║
║   │   ├── 🔧 motor_diesel/                                                  │║
║   │   │   │   Tipo: Motor Diesel 6 Cilindros                                │║
║   │   │   │   Potência: 550 HP                                              │║
║   │   │   │                                                                 │║
║   │   │   ├── 🌡️ sensor_temperatura_motor                                   │║
║   │   │   │       Range: -40°C a 150°C                                      │║
║   │   │   │       Protocolo: Modbus RTU                                     │║
║   │   │   │       Alerta: > 95°C                                            │║
║   │   │   │       Crítico: > 110°C                                          │║
║   │   │   │                                                                 │║
║   │   │   ├── 📊 sensor_rotacao_rpm                                         │║
║   │   │   │       Range: 0 - 3000 RPM                                       │║
║   │   │   │       Nominal: 1800 RPM                                         │║
║   │   │   │       Protocolo: Modbus RTU                                     │║
║   │   │   │                                                                 │║
║   │   │   ├── 📳 sensor_vibracao                                            │║
║   │   │   │       Range: 0 - 50 mm/s                                        │║
║   │   │   │       Alerta: > 10 mm/s                                         │║
║   │   │   │       Protocolo: Modbus RTU                                     │║
║   │   │   │                                                                 │║
║   │   │   └── 🛢️ sensor_pressao_oleo                                        │║
║   │   │           Range: 0 - 10 bar                                         │║
║   │   │           Mínimo: 2 bar (running)                                   │║
║   │   │           Protocolo: Modbus RTU                                     │║
║   │   │                                                                     │║
║   │   ├── 💧 sistema_combustivel/                                           │║
║   │   │   │   Capacidade: 1000 litros                                       │║
║   │   │   │                                                                 │║
║   │   │   ├── 📊 sensor_nivel_tanque                                        │║
║   │   │   │       Range: 0 - 100%                                           │║
║   │   │   │       Alerta: < 20%                                             │║
║   │   │   │       Crítico: < 10%                                            │║
║   │   │   │       Protocolo: 4-20mA                                         │║
║   │   │   │                                                                 │║
║   │   │   ├── 🔌 bomba_transferencia                                        │║
║   │   │   │       Tipo: Atuador ON/OFF                                      │║
║   │   │   │       Vazão: 50 L/min                                           │║
║   │   │   │       Protocolo: Relé Digital                                   │║
║   │   │   │                                                                 │║
║   │   │   └── 🌡️ sensor_temperatura_combustivel                             │║
║   │   │           Range: -20°C a 80°C                                       │║
║   │   │           Protocolo: Modbus RTU                                     │║
║   │   │                                                                     │║
║   │   ├── ⚡ alternador/                                                     │║
║   │   │   │   Potência: 500 KVA                                             │║
║   │   │   │   Tensão: 380V / 220V                                           │║
║   │   │   │                                                                 │║
║   │   │   ├── 📊 sensor_tensao_saida                                        │║
║   │   │   │       Range: 0 - 500V                                           │║
║   │   │   │       Nominal: 380V ±5%                                         │║
║   │   │   │       Protocolo: Modbus RTU                                     │║
║   │   │   │                                                                 │║
║   │   │   ├── 📊 sensor_corrente_saida                                      │║
║   │   │   │       Range: 0 - 1000A                                          │║
║   │   │   │       Protocolo: Modbus RTU                                     │║
║   │   │   │                                                                 │║
║   │   │   ├── 📊 sensor_frequencia                                          │║
║   │   │   │       Range: 0 - 70 Hz                                          │║
║   │   │   │       Nominal: 60 Hz ±0.5%                                      │║
║   │   │   │       Protocolo: Modbus RTU                                     │║
║   │   │   │                                                                 │║
║   │   │   └── 🌡️ sensor_temperatura_enrolamento                             │║
║   │   │           Range: 0 - 200°C                                          │║
║   │   │           Alerta: > 120°C                                           │║
║   │   │           Protocolo: Modbus RTU                                     │║
║   │   │                                                                     │║
║   │   ├── 🎛️ painel_controle/                                               │║
║   │   │   │   Tipo: CLP com IHM                                             │║
║   │   │   │                                                                 │║
║   │   │   ├── 🔘 botao_partida                                              │║
║   │   │   │       Tipo: Input Digital                                       │║
║   │   │   │       Protocolo: DI                                             │║
║   │   │   │                                                                 │║
║   │   │   ├── 🔘 botao_parada                                               │║
║   │   │   │       Tipo: Input Digital                                       │║
║   │   │   │       Protocolo: DI                                             │║
║   │   │   │                                                                 │║
║   │   │   ├── 🔘 botao_emergencia                                           │║
║   │   │   │       Tipo: Input Digital (NC)                                  │║
║   │   │   │       Protocolo: DI                                             │║
║   │   │   │                                                                 │║
║   │   │   └── 🔌 rele_principal                                             │║
║   │   │           Tipo: Output Digital                                      │║
║   │   │           Carga: 500A                                               │║
║   │   │           Protocolo: DO                                             │║
║   │   │                                                                     │║
║   │   └── 🌀 sistema_arrefecimento/                                         │║
║   │       │                                                                 │║
║   │       ├── 🌡️ sensor_temperatura_radiador                                │║
║   │       │       Range: 0 - 120°C                                          │║
║   │       │       Protocolo: Modbus RTU                                     │║
║   │       │                                                                 │║
║   │       ├── 📊 sensor_nivel_agua                                          │║
║   │       │       Tipo: Float Switch                                        │║
║   │       │       Protocolo: DI                                             │║
║   │       │                                                                 │║
║   │       └── 🔌 ventilador_radiador                                        │║
║   │               Tipo: Motor 3 velocidades                                 │║
║   │               Protocolo: 0-10V                                          │║
║   │                                                                         │║
║   └─────────────────────────────────────────────────────────────────────────┘║
║                                                                               ║
║   TOTAL: 1 Device Pai + 5 Subsistemas + 18 Sensores/Atuadores               ║
║                                                                               ║
╚═══════════════════════════════════════════════════════════════════════════════╝
```

---

## 📜 Regras Embutidas no Template

O diferencial revolucionário: **regras de automação viajam com o template**.

### Estrutura de Regras

```
╔═══════════════════════════════════════════════════════════════════════════════╗
║                                                                               ║
║   📜 REGRAS DO TEMPLATE: GERADOR DIESEL 500KVA                                ║
║   ════════════════════════════════════════════                                ║
║                                                                               ║
║   ┌─────────────────────────────────────────────────────────────────────────┐║
║   │                                                                         │║
║   │   REGRA 1: PROTEÇÃO TÉRMICA DO MOTOR                                    │║
║   │   ──────────────────────────────────                                    │║
║   │                                                                         │║
║   │   CONDIÇÃO:                                                             │║
║   │   ┌─────────────────────────────────────────────────────────────────┐  │║
║   │   │  motor_diesel.sensor_temperatura_motor.value > 110°C            │  │║
║   │   │                      AND                                         │  │║
║   │   │  duration > 30 seconds                                           │  │║
║   │   └─────────────────────────────────────────────────────────────────┘  │║
║   │                                                                         │║
║   │   AÇÕES:                                                                │║
║   │   ┌─────────────────────────────────────────────────────────────────┐  │║
║   │   │  1. SET painel_controle.rele_principal = OFF                    │  │║
║   │   │  2. ALERT "Shutdown térmico - Motor superaquecido"              │  │║
║   │   │  3. LOG level=CRITICAL, event="thermal_shutdown"                │  │║
║   │   │  4. NOTIFY group="manutencao", priority=HIGH                    │  │║
║   │   └─────────────────────────────────────────────────────────────────┘  │║
║   │                                                                         │║
║   │   RECUPERAÇÃO:                                                          │║
║   │   ┌─────────────────────────────────────────────────────────────────┐  │║
║   │   │  WHEN motor_diesel.sensor_temperatura_motor.value < 80°C        │  │║
║   │   │  THEN ENABLE manual_restart                                      │  │║
║   │   └─────────────────────────────────────────────────────────────────┘  │║
║   │                                                                         │║
║   └─────────────────────────────────────────────────────────────────────────┘║
║                                                                               ║
║   ┌─────────────────────────────────────────────────────────────────────────┐║
║   │                                                                         │║
║   │   REGRA 2: NÍVEL BAIXO DE COMBUSTÍVEL                                   │║
║   │   ───────────────────────────────────                                   │║
║   │                                                                         │║
║   │   CONDIÇÃO:                                                             │║
║   │   ┌─────────────────────────────────────────────────────────────────┐  │║
║   │   │  sistema_combustivel.sensor_nivel_tanque.value < 20%            │  │║
║   │   └─────────────────────────────────────────────────────────────────┘  │║
║   │                                                                         │║
║   │   AÇÕES (Nível 20%):                                                    │║
║   │   ┌─────────────────────────────────────────────────────────────────┐  │║
║   │   │  1. ALERT "Nível de combustível baixo - Reabastecer"            │  │║
║   │   │  2. SET sistema_combustivel.bomba_transferencia = ON            │  │║
║   │   │  3. NOTIFY group="operacao", priority=MEDIUM                    │  │║
║   │   └─────────────────────────────────────────────────────────────────┘  │║
║   │                                                                         │║
║   │   CONDIÇÃO CRÍTICA:                                                     │║
║   │   ┌─────────────────────────────────────────────────────────────────┐  │║
║   │   │  sistema_combustivel.sensor_nivel_tanque.value < 10%            │  │║
║   │   └─────────────────────────────────────────────────────────────────┘  │║
║   │                                                                         │║
║   │   AÇÕES (Nível 10%):                                                    │║
║   │   ┌─────────────────────────────────────────────────────────────────┐  │║
║   │   │  1. SET painel_controle.rele_principal = OFF (Shutdown seguro)  │  │║
║   │   │  2. ALERT "CRÍTICO: Shutdown por combustível baixo"             │  │║
║   │   │  3. NOTIFY group="emergencia", priority=CRITICAL                │  │║
║   │   └─────────────────────────────────────────────────────────────────┘  │║
║   │                                                                         │║
║   └─────────────────────────────────────────────────────────────────────────┘║
║                                                                               ║
║   ┌─────────────────────────────────────────────────────────────────────────┐║
║   │                                                                         │║
║   │   REGRA 3: CONTROLE AUTOMÁTICO DE ARREFECIMENTO                         │║
║   │   ─────────────────────────────────────────────                         │║
║   │                                                                         │║
║   │   LÓGICA PROPORCIONAL:                                                  │║
║   │   ┌─────────────────────────────────────────────────────────────────┐  │║
║   │   │  temp = motor_diesel.sensor_temperatura_motor.value             │  │║
║   │   │                                                                 │  │║
║   │   │  IF temp < 70°C:                                                │  │║
║   │   │      sistema_arrefecimento.ventilador_radiador = 0% (OFF)       │  │║
║   │   │                                                                 │  │║
║   │   │  IF temp >= 70°C AND temp < 85°C:                               │  │║
║   │   │      sistema_arrefecimento.ventilador_radiador = 33% (LOW)      │  │║
║   │   │                                                                 │  │║
║   │   │  IF temp >= 85°C AND temp < 95°C:                               │  │║
║   │   │      sistema_arrefecimento.ventilador_radiador = 66% (MEDIUM)   │  │║
║   │   │                                                                 │  │║
║   │   │  IF temp >= 95°C:                                               │  │║
║   │   │      sistema_arrefecimento.ventilador_radiador = 100% (HIGH)    │  │║
║   │   └─────────────────────────────────────────────────────────────────┘  │║
║   │                                                                         │║
║   └─────────────────────────────────────────────────────────────────────────┘║
║                                                                               ║
║   ┌─────────────────────────────────────────────────────────────────────────┐║
║   │                                                                         │║
║   │   REGRA 4: PROTEÇÃO DE PRESSÃO DE ÓLEO                                  │║
║   │   ────────────────────────────────────                                  │║
║   │                                                                         │║
║   │   CONDIÇÃO:                                                             │║
║   │   ┌─────────────────────────────────────────────────────────────────┐  │║
║   │   │  motor_diesel.sensor_pressao_oleo.value < 2 bar                 │  │║
║   │   │                      AND                                         │  │║
║   │   │  motor_diesel.sensor_rotacao_rpm.value > 500 RPM                │  │║
║   │   └─────────────────────────────────────────────────────────────────┘  │║
║   │                                                                         │║
║   │   AÇÕES:                                                                │║
║   │   ┌─────────────────────────────────────────────────────────────────┐  │║
║   │   │  1. SET painel_controle.rele_principal = OFF (Shutdown imediato)│  │║
║   │   │  2. ALERT "EMERGÊNCIA: Pressão de óleo crítica"                 │  │║
║   │   │  3. LOCK restart_disabled = true                                │  │║
║   │   │  4. NOTIFY group="emergencia", priority=CRITICAL                │  │║
║   │   └─────────────────────────────────────────────────────────────────┘  │║
║   │                                                                         │║
║   └─────────────────────────────────────────────────────────────────────────┘║
║                                                                               ║
║   ┌─────────────────────────────────────────────────────────────────────────┐║
║   │                                                                         │║
║   │   REGRA 5: MONITORAMENTO DE QUALIDADE DE ENERGIA                        │║
║   │   ──────────────────────────────────────────────                        │║
║   │                                                                         │║
║   │   CONDIÇÃO (Subtensão):                                                 │║
║   │   ┌─────────────────────────────────────────────────────────────────┐  │║
║   │   │  alternador.sensor_tensao_saida.value < 361V (380V - 5%)        │  │║
║   │   │                      OR                                          │  │║
║   │   │  alternador.sensor_frequencia.value < 59.5 Hz                   │  │║
║   │   └─────────────────────────────────────────────────────────────────┘  │║
║   │                                                                         │║
║   │   AÇÕES:                                                                │║
║   │   ┌─────────────────────────────────────────────────────────────────┐  │║
║   │   │  1. ALERT "Qualidade de energia fora do padrão"                 │  │║
║   │   │  2. LOG event="power_quality_issue", voltage=$value, freq=$freq │  │║
║   │   │  3. INCREMENT counter="power_events"                            │  │║
║   │   │  4. IF counter > 5 in 1h: NOTIFY group="manutencao"             │  │║
║   │   └─────────────────────────────────────────────────────────────────┘  │║
║   │                                                                         │║
║   └─────────────────────────────────────────────────────────────────────────┘║
║                                                                               ║
╚═══════════════════════════════════════════════════════════════════════════════╝
```

---

## 🔄 Fluxo de Criação de Device Composto

### Wizard de Composição (6 Passos)

```
╔═══════════════════════════════════════════════════════════════════════════════╗
║                                                                               ║
║                    🧙 WIZARD DE DEVICE COMPOSTO                               ║
║                                                                               ║
║  ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐      ║
║  │ PASSO  │  │ PASSO  │  │ PASSO  │  │ PASSO  │  │ PASSO  │  │ PASSO  │      ║
║  │   1    │─►│   2    │─►│   3    │─►│   4    │─►│   5    │─►│   6    │      ║
║  │        │  │        │  │        │  │        │  │        │  │        │      ║
║  │ 📝     │  │ 🧩     │  │ 🔗     │  │ 📜     │  │ 🎲     │  │ ✅     │      ║
║  │ Meta   │  │Compon. │  │Conexões│  │ Regras │  │ 3D     │  │Validar │      ║
║  └────────┘  └────────┘  └────────┘  └────────┘  └────────┘  └────────┘      ║
║                                                                               ║
║  ─────────────────────────────────────────────────────────────────────────   ║
║  ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓░░░░░░░░░░░░░░░░░░░░  66%          ║
║                                                                               ║
╚═══════════════════════════════════════════════════════════════════════════════╝
```

---

### 📝 Passo 1: Metadados do Device Pai

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  📝 IDENTIFICAÇÃO DO DEVICE COMPOSTO                            │
│  ───────────────────────────────────                            │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ Nome do Template                                         │   │
│  │ ┌───────────────────────────────────────────────────┐   │   │
│  │ │ Gerador Diesel 500KVA Industrial                  │   │   │
│  │ └───────────────────────────────────────────────────┘   │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ Tipo de Device                                           │   │
│  │ ┌───────────────────────────────────────────────────┐   │   │
│  │ │ ○ Device Simples (sem componentes internos)       │   │   │
│  │ │ ● Device Composto (com sub-componentes)       ◄── │   │   │
│  │ │ ○ Device Virtual (agregação de dados)             │   │   │
│  │ └───────────────────────────────────────────────────┘   │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ Categoria Industrial                                     │   │
│  │ ┌───────────────────────────────────────────────────┐   │   │
│  │ │ ⚡ Geração de Energia                          ▼  │   │   │
│  │ └───────────────────────────────────────────────────┘   │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ Tags                                                     │   │
│  │ ┌───────────────────────────────────────────────────┐   │   │
│  │ │ energia  backup  diesel  500kva  cummins          │   │   │
│  │ └───────────────────────────────────────────────────┘   │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│                                        [ Próximo → ]            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

### 🧩 Passo 2: Adicionar Componentes

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  🧩 COMPOSIÇÃO DE COMPONENTES                                   │
│  ────────────────────────────                                   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                                                         │   │
│  │  BIBLIOTECA DE TEMPLATES         COMPONENTES ADICIONADOS │   │
│  │  ───────────────────────         ─────────────────────── │   │
│  │                                                         │   │
│  │  ┌──────────────────┐            ┌──────────────────┐   │   │
│  │  │ 🔍 Buscar...     │            │ Gerador Diesel   │   │   │
│  │  └──────────────────┘            │ 500KVA           │   │   │
│  │                                  │                  │   │   │
│  │  ┌──────────────────┐    ──►     │ ├── 🔧 Motor     │   │   │
│  │  │ ⚡ Motor Diesel  │            │ │   Diesel       │   │   │
│  │  │    6 Cilindros   │            │ │                │   │   │
│  │  └──────────────────┘            │ ├── 💧 Sistema   │   │   │
│  │                                  │ │   Combustível  │   │   │
│  │  ┌──────────────────┐    ──►     │ │                │   │   │
│  │  │ 💧 Sistema       │            │ ├── ⚡ Alterna-  │   │   │
│  │  │    Combustível   │            │ │   dor          │   │   │
│  │  └──────────────────┘            │ │                │   │   │
│  │                                  │ ├── 🎛️ Painel    │   │   │
│  │  ┌──────────────────┐    ──►     │ │   Controle     │   │   │
│  │  │ ⚡ Alternador    │            │ │                │   │   │
│  │  │    Elétrico      │            │ └── 🌀 Sistema   │   │   │
│  │  └──────────────────┘            │     Arrefecim.   │   │   │
│  │                                  │                  │   │   │
│  │  ┌──────────────────┐            └──────────────────┘   │   │
│  │  │ 🎛️ Painel        │                                   │   │
│  │  │    Controle      │            [ 🗑️ Remover ]         │   │
│  │  └──────────────────┘            [ 📋 Duplicar ]        │   │
│  │                                  [ ⚙️ Configurar ]      │   │
│  │  ┌──────────────────┐                                   │   │
│  │  │ 🌀 Sistema       │                                   │   │
│  │  │    Arrefecimento │                                   │   │
│  │  └──────────────────┘                                   │   │
│  │                                                         │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  💡 Dica: Arraste templates da biblioteca para adicionar        │
│                                                                 │
│                             [ ← Anterior ]  [ Próximo → ]       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

### 🔗 Passo 3: Conexões Internas

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  🔗 MAPEAMENTO DE CONEXÕES INTERNAS                             │
│  ──────────────────────────────────                             │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                                                         │   │
│  │   ┌─────────────────────────────────────────────────┐   │   │
│  │   │                                                 │   │   │
│  │   │              DIAGRAMA DE CONEXÕES               │   │   │
│  │   │                                                 │   │   │
│  │   │     ┌───────────┐         ┌───────────┐        │   │   │
│  │   │     │   MOTOR   │◄───────►│ ALTERNADOR│        │   │   │
│  │   │     │   DIESEL  │  Eixo   │  ELÉTRICO │        │   │   │
│  │   │     └─────┬─────┘         └─────┬─────┘        │   │   │
│  │   │           │                     │              │   │   │
│  │   │           │ Combustível         │ Energia      │   │   │
│  │   │           │                     │              │   │   │
│  │   │     ┌─────▼─────┐         ┌─────▼─────┐        │   │   │
│  │   │     │  SISTEMA  │         │  PAINEL   │        │   │   │
│  │   │     │COMBUSTÍVEL│         │ CONTROLE  │        │   │   │
│  │   │     └───────────┘         └─────┬─────┘        │   │   │
│  │   │                                 │              │   │   │
│  │   │           ┌─────────────────────┘              │   │   │
│  │   │           │ Controle                           │   │   │
│  │   │           ▼                                    │   │   │
│  │   │     ┌───────────┐                              │   │   │
│  │   │     │  SISTEMA  │                              │   │   │
│  │   │     │ARREFECIM. │                              │   │   │
│  │   │     └───────────┘                              │   │   │
│  │   │                                                 │   │   │
│  │   └─────────────────────────────────────────────────┘   │   │
│  │                                                         │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ CONEXÕES DEFINIDAS                                       │   │
│  │ ──────────────────                                       │   │
│  │                                                         │   │
│  │  ✅ motor → alternador (mecânica: eixo)                 │   │
│  │  ✅ sistema_combustivel → motor (fluido: diesel)        │   │
│  │  ✅ alternador → painel_controle (elétrica: potência)   │   │
│  │  ✅ painel_controle → sistema_arrefecimento (controle)  │   │
│  │                                                         │   │
│  │  [ + Adicionar Conexão ]                                │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│                             [ ← Anterior ]  [ Próximo → ]       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

### 📜 Passo 4: Regras e Automações

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  📜 REGRAS EMBUTIDAS NO TEMPLATE                                │
│  ───────────────────────────────                                │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                                                         │   │
│  │  REGRAS ATIVAS                        [ + Nova Regra ]  │   │
│  │  ─────────────                                          │   │
│  │                                                         │   │
│  │  ┌─────────────────────────────────────────────────┐   │   │
│  │  │ 🔴 REGRA 1: Proteção Térmica                     │   │   │
│  │  │    SE temp_motor > 110°C POR 30s                 │   │   │
│  │  │    ENTÃO desligar_gerador + alerta_crítico       │   │   │
│  │  │    Prioridade: CRÍTICA        [ ⚙️ ] [ 🗑️ ]     │   │   │
│  │  └─────────────────────────────────────────────────┘   │   │
│  │                                                         │   │
│  │  ┌─────────────────────────────────────────────────┐   │   │
│  │  │ 🟠 REGRA 2: Combustível Baixo                    │   │   │
│  │  │    SE nivel_tanque < 20%                         │   │   │
│  │  │    ENTÃO ativar_bomba + notificar_operacao       │   │   │
│  │  │    Prioridade: ALTA           [ ⚙️ ] [ 🗑️ ]     │   │   │
│  │  └─────────────────────────────────────────────────┘   │   │
│  │                                                         │   │
│  │  ┌─────────────────────────────────────────────────┐   │   │
│  │  │ 🟢 REGRA 3: Controle Arrefecimento               │   │   │
│  │  │    PROPORCIONAL temp_motor → velocidade_ventil   │   │   │
│  │  │    70°C=0%, 85°C=33%, 95°C=66%, 100°C=100%       │   │   │
│  │  │    Prioridade: NORMAL         [ ⚙️ ] [ 🗑️ ]     │   │   │
│  │  └─────────────────────────────────────────────────┘   │   │
│  │                                                         │   │
│  │  ┌─────────────────────────────────────────────────┐   │   │
│  │  │ 🔴 REGRA 4: Pressão Óleo Crítica                 │   │   │
│  │  │    SE pressao_oleo < 2bar E rpm > 500            │   │   │
│  │  │    ENTÃO shutdown_emergencia + bloquear_restart  │   │   │
│  │  │    Prioridade: EMERGÊNCIA     [ ⚙️ ] [ 🗑️ ]     │   │   │
│  │  └─────────────────────────────────────────────────┘   │   │
│  │                                                         │   │
│  │  ┌─────────────────────────────────────────────────┐   │   │
│  │  │ 🟡 REGRA 5: Qualidade de Energia                 │   │   │
│  │  │    SE tensao < 361V OU frequencia < 59.5Hz       │   │   │
│  │  │    ENTÃO log_evento + incrementar_contador       │   │   │
│  │  │    Prioridade: MÉDIA          [ ⚙️ ] [ 🗑️ ]     │   │   │
│  │  └─────────────────────────────────────────────────┘   │   │
│  │                                                         │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ TIPOS DE REGRAS DISPONÍVEIS                              │   │
│  │                                                         │   │
│  │  📊 Threshold      - Valor ultrapassa limite            │   │
│  │  ⏱️ Duration       - Condição mantida por tempo         │   │
│  │  📈 Trend          - Tendência de aumento/queda         │   │
│  │  🔄 Proporcional   - Saída proporcional à entrada       │   │
│  │  🔗 Correlação     - Múltiplos sensores combinados      │   │
│  │  ⚡ Evento         - Trigger por mudança de estado      │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│                             [ ← Anterior ]  [ Próximo → ]       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

### 🎲 Passo 5: Modelo 3D Composto

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  🎲 MODELO 3D DO DEVICE COMPOSTO                                │
│  ───────────────────────────────                                │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                                                         │   │
│  │              ┌───────────────────────────────┐          │   │
│  │             /                               /│          │   │
│  │            /      🏭 GERADOR DIESEL        / │          │   │
│  │           /          (Preview 3D)         /  │          │   │
│  │          ┌───────────────────────────────┐   │          │   │
│  │          │                               │   │          │   │
│  │          │   ┌─────┐  ┌─────┐  ┌─────┐   │   │          │   │
│  │          │   │Motor│══│Alter│══│Painel│  │   │          │   │
│  │          │   └──┬──┘  └─────┘  └──┬──┘   │   │          │   │
│  │          │      │                 │      │   │          │   │
│  │          │   ┌──┴──┐          ┌───┴──┐   │   │          │   │
│  │          │   │Comb.│          │Arref.│   │   │          │   │
│  │          │   └─────┘          └──────┘   │   │          │   │
│  │          │                               │   │          │   │
│  │          │   [ 🔄 Rotacionar ] [ 🔍 Zoom ]   │          │   │
│  │          │                               │   │          │   │
│  │          └───────────────────────────────┘───┘          │   │
│  │                                                         │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ OPÇÕES DE MODELO 3D                                      │   │
│  │ ──────────────────                                       │   │
│  │                                                         │   │
│  │  ○ Upload modelo único (GLTF/GLB)                       │   │
│  │  ● Auto-compor a partir dos componentes            ◄── │   │
│  │  ○ Usar placeholder genérico                            │   │
│  │                                                         │   │
│  │  ┌───────────────────────────────────────────────────┐ │   │
│  │  │ POSICIONAMENTO DOS COMPONENTES                     │ │   │
│  │  │                                                    │ │   │
│  │  │  Motor:        X: 0    Y: 0    Z: 0               │ │   │
│  │  │  Alternador:   X: 1.5  Y: 0    Z: 0               │ │   │
│  │  │  Painel:       X: 3    Y: 0    Z: 0               │ │   │
│  │  │  Combustível:  X: 0    Y: -1   Z: 0               │ │   │
│  │  │  Arrefecim.:   X: 3    Y: -1   Z: 0               │ │   │
│  │  └───────────────────────────────────────────────────┘ │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│                             [ ← Anterior ]  [ Próximo → ]       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

### ✅ Passo 6: Validação e Publicação

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  ✅ VALIDAÇÃO DO TEMPLATE                                       │
│  ────────────────────────                                       │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                                                         │   │
│  │  CHECKLIST DE VALIDAÇÃO                                 │   │
│  │  ──────────────────────                                 │   │
│  │                                                         │   │
│  │  ✅ Metadados completos                                 │   │
│  │     Nome, categoria, tags definidos                     │   │
│  │                                                         │   │
│  │  ✅ Estrutura de componentes válida                     │   │
│  │     5 subsistemas, 18 sensores/atuadores                │   │
│  │                                                         │   │
│  │  ✅ Conexões internas mapeadas                          │   │
│  │     4 conexões definidas, sem ciclos                    │   │
│  │                                                         │   │
│  │  ✅ Regras sem conflitos                                │   │
│  │     5 regras validadas, prioridades definidas           │   │
│  │                                                         │   │
│  │  ✅ Protocolos configurados                             │   │
│  │     Modbus RTU, DI/DO mapeados                          │   │
│  │                                                         │   │
│  │  ✅ Modelo 3D gerado                                    │   │
│  │     Auto-composição bem-sucedida                        │   │
│  │                                                         │   │
│  │  ─────────────────────────────────────────────────────  │   │
│  │  RESULTADO: TEMPLATE VÁLIDO ✅                          │   │
│  │                                                         │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ RESUMO DO TEMPLATE                                       │   │
│  │                                                         │   │
│  │  Nome:          Gerador Diesel 500KVA Industrial        │   │
│  │  Componentes:   5 subsistemas                           │   │
│  │  Sensores:      18                                      │   │
│  │  Atuadores:     4                                       │   │
│  │  Regras:        5                                       │   │
│  │  Tamanho:       2.4 MB (incluindo 3D)                   │   │
│  │                                                         │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│                 [ ← Anterior ]  [ 📤 Publicar Template ]        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📊 Propagação de Status (Health Aggregation)

Uma das inovações mais poderosas: **status agregado automaticamente**.

```
╔═══════════════════════════════════════════════════════════════════════════════╗
║                                                                               ║
║   📊 PROPAGAÇÃO AUTOMÁTICA DE STATUS                                          ║
║   ══════════════════════════════════                                          ║
║                                                                               ║
║   PRINCÍPIO: O status do pai é o PIOR status entre seus filhos               ║
║                                                                               ║
║   ┌─────────────────────────────────────────────────────────────────────────┐║
║   │                                                                         │║
║   │   CENÁRIO 1: TODOS SAUDÁVEIS                                            │║
║   │   ──────────────────────────                                            │║
║   │                                                                         │║
║   │                  ┌─────────────────────┐                                │║
║   │                  │ 🟢 GERADOR DIESEL   │                                │║
║   │                  │    Status: HEALTHY  │                                │║
║   │                  └──────────┬──────────┘                                │║
║   │                             │                                           │║
║   │       ┌─────────────────────┼─────────────────────┐                     │║
║   │       │                     │                     │                     │║
║   │   ┌───┴───┐             ┌───┴───┐             ┌───┴───┐                 │║
║   │   │ 🟢    │             │ 🟢    │             │ 🟢    │                 │║
║   │   │ Motor │             │ Comb. │             │ Alter.│                 │║
║   │   └───────┘             └───────┘             └───────┘                 │║
║   │                                                                         │║
║   └─────────────────────────────────────────────────────────────────────────┘║
║                                                                               ║
║   ┌─────────────────────────────────────────────────────────────────────────┐║
║   │                                                                         │║
║   │   CENÁRIO 2: UM COMPONENTE EM ALERTA                                    │║
║   │   ──────────────────────────────────                                    │║
║   │                                                                         │║
║   │                  ┌─────────────────────┐                                │║
║   │                  │ 🟡 GERADOR DIESEL   │ ◄── Propaga WARNING            │║
║   │                  │    Status: WARNING  │                                │║
║   │                  └──────────┬──────────┘                                │║
║   │                             │                                           │║
║   │       ┌─────────────────────┼─────────────────────┐                     │║
║   │       │                     │                     │                     │║
║   │   ┌───┴───┐             ┌───┴───┐             ┌───┴───┐                 │║
║   │   │ 🟡    │ ◄───────    │ 🟢    │             │ 🟢    │                 │║
║   │   │ Motor │   Temp 92°C │ Comb. │             │ Alter.│                 │║
║   │   └───────┘             └───────┘             └───────┘                 │║
║   │                                                                         │║
║   └─────────────────────────────────────────────────────────────────────────┘║
║                                                                               ║
║   ┌─────────────────────────────────────────────────────────────────────────┐║
║   │                                                                         │║
║   │   CENÁRIO 3: COMPONENTE CRÍTICO                                         │║
║   │   ─────────────────────────────                                         │║
║   │                                                                         │║
║   │                  ┌─────────────────────┐                                │║
║   │                  │ 🔴 GERADOR DIESEL   │ ◄── Propaga CRITICAL           │║
║   │                  │    Status: CRITICAL │                                │║
║   │                  └──────────┬──────────┘                                │║
║   │                             │                                           │║
║   │       ┌─────────────────────┼─────────────────────┐                     │║
║   │       │                     │                     │                     │║
║   │   ┌───┴───┐             ┌───┴───┐             ┌───┴───┐                 │║
║   │   │ 🟢    │             │ 🔴    │ ◄───────    │ 🟢    │                 │║
║   │   │ Motor │             │ Comb. │  Nível 8%  │ Alter.│                 │║
║   │   └───────┘             └───────┘             └───────┘                 │║
║   │                                                                         │║
║   └─────────────────────────────────────────────────────────────────────────┘║
║                                                                               ║
║   HIERARQUIA DE STATUS:                                                       ║
║   🟢 HEALTHY < 🟡 WARNING < 🟠 DEGRADED < 🔴 CRITICAL < ⚫ OFFLINE           ║
║                                                                               ║
╚═══════════════════════════════════════════════════════════════════════════════╝
```

---

## 🌐 Casos de Uso Industriais

### Caso 1: Subestação Elétrica

```
╔═══════════════════════════════════════════════════════════════════════════════╗
║                                                                               ║
║   ⚡ SUBESTAÇÃO ELÉTRICA 13.8kV                                               ║
║   ═════════════════════════════                                               ║
║                                                                               ║
║   ┌─────────────────────────────────────────────────────────────────────────┐║
║   │                                                                         │║
║   │   subestacao_13_8kv/                                                    │║
║   │   │                                                                     │║
║   │   ├── transformador_principal/                                          │║
║   │   │   ├── sensor_temperatura_oleo                                       │║
║   │   │   ├── sensor_nivel_oleo                                             │║
║   │   │   ├── sensor_temperatura_enrolamento                                │║
║   │   │   └── rele_buchholz                                                 │║
║   │   │                                                                     │║
║   │   ├── disjuntor_entrada/                                                │║
║   │   │   ├── sensor_posicao                                                │║
║   │   │   ├── contador_operacoes                                            │║
║   │   │   └── bobina_abertura                                               │║
║   │   │                                                                     │║
║   │   ├── barramento_principal/                                             │║
║   │   │   ├── sensor_tensao_fase_a                                          │║
║   │   │   ├── sensor_tensao_fase_b                                          │║
║   │   │   ├── sensor_tensao_fase_c                                          │║
║   │   │   └── sensor_corrente_neutro                                        │║
║   │   │                                                                     │║
║   │   └── banco_capacitores/                                                │║
║   │       ├── sensor_potencia_reativa                                       │║
║   │       ├── contator_estagio_1                                            │║
║   │       ├── contator_estagio_2                                            │║
║   │       └── contator_estagio_3                                            │║
║   │                                                                         │║
║   │   REGRAS EMBUTIDAS:                                                     │║
║   │   • Sobrecarga transformador → Reduzir carga                            │║
║   │   • Fator de potência < 0.92 → Acionar capacitores                      │║
║   │   • Buchholz ativado → Abrir disjuntor imediatamente                    │║
║   │   • Desequilíbrio de fases > 5% → Alertar operação                      │║
║   │                                                                         │║
║   └─────────────────────────────────────────────────────────────────────────┘║
║                                                                               ║
║   📈 RESULTADO: 1 template = 50 subestações padronizadas                     ║
║                                                                               ║
╚═══════════════════════════════════════════════════════════════════════════════╝
```

---

### Caso 2: Linha de Envase

```
╔═══════════════════════════════════════════════════════════════════════════════╗
║                                                                               ║
║   🏭 LINHA DE ENVASE DE BEBIDAS                                               ║
║   ═════════════════════════════                                               ║
║                                                                               ║
║   ┌─────────────────────────────────────────────────────────────────────────┐║
║   │                                                                         │║
║   │   linha_envase_pet/                                                     │║
║   │   │                                                                     │║
║   │   ├── soprador_pet/                                                     │║
║   │   │   ├── sensor_temperatura_forno                                      │║
║   │   │   ├── sensor_pressao_ar                                             │║
║   │   │   ├── contador_garrafas_sopradas                                    │║
║   │   │   └── sensor_rejeito                                                │║
║   │   │                                                                     │║
║   │   ├── enchedora/                                                        │║
║   │   │   ├── sensor_nivel_tanque                                           │║
║   │   │   ├── sensor_temperatura_produto                                    │║
║   │   │   ├── balanca_dosagem (x12 bicos)                                   │║
║   │   │   └── contador_garrafas_cheias                                      │║
║   │   │                                                                     │║
║   │   ├── tamponadora/                                                      │║
║   │   │   ├── sensor_torque                                                 │║
║   │   │   ├── sensor_presenca_tampa                                         │║
║   │   │   └── contador_tampas                                               │║
║   │   │                                                                     │║
║   │   ├── rotuladora/                                                       │║
║   │   │   ├── sensor_tensao_bobina                                          │║
║   │   │   ├── camera_inspecao_rotulo                                        │║
║   │   │   └── contador_rotulos                                              │║
║   │   │                                                                     │║
║   │   └── empacotadora/                                                     │║
║   │       ├── sensor_filme_restante                                         │║
║   │       ├── sensor_temperatura_solda                                      │║
║   │       └── contador_pacotes                                              │║
║   │                                                                         │║
║   │   REGRAS EMBUTIDAS:                                                     │║
║   │   • Rejeito soprador > 2% → Ajustar temperatura                         │║
║   │   • Peso fora ±0.5% → Parar linha + alertar qualidade                   │║
║   │   • Torque tampa fora spec → Recalibrar tamponadora                     │║
║   │   • Rótulo desalinhado → Ejetar garrafa                                 │║
║   │   • OEE < 85% → Notificar supervisão                                    │║
║   │                                                                         │║
║   └─────────────────────────────────────────────────────────────────────────┘║
║                                                                               ║
║   📈 RESULTADO: OEE calculado automaticamente por linha                      ║
║                                                                               ║
╚═══════════════════════════════════════════════════════════════════════════════╝
```

---

### Caso 3: Estação de Tratamento de Água

```
╔═══════════════════════════════════════════════════════════════════════════════╗
║                                                                               ║
║   💧 ESTAÇÃO DE TRATAMENTO DE ÁGUA (ETA)                                      ║
║   ══════════════════════════════════════                                      ║
║                                                                               ║
║   ┌─────────────────────────────────────────────────────────────────────────┐║
║   │                                                                         │║
║   │   eta_50_mil_m3/                                                        │║
║   │   │                                                                     │║
║   │   ├── captacao/                                                         │║
║   │   │   ├── bomba_captacao_1/                                             │║
║   │   │   │   ├── sensor_corrente                                           │║
║   │   │   │   ├── sensor_vibração                                           │║
║   │   │   │   └── sensor_temperatura_motor                                  │║
║   │   │   ├── bomba_captacao_2/ (backup)                                    │║
║   │   │   └── sensor_nivel_rio                                              │║
║   │   │                                                                     │║
║   │   ├── floculacao/                                                       │║
║   │   │   ├── dosadora_sulfato/                                             │║
║   │   │   │   ├── sensor_nivel_tanque                                       │║
║   │   │   │   └── bomba_dosadora                                            │║
║   │   │   ├── sensor_turbidez_entrada                                       │║
║   │   │   └── agitador_rapido                                               │║
║   │   │                                                                     │║
║   │   ├── decantacao/                                                       │║
║   │   │   ├── sensor_turbidez_saida                                         │║
║   │   │   ├── raspador_lodo                                                 │║
║   │   │   └── valvula_descarte_lodo                                         │║
║   │   │                                                                     │║
║   │   ├── filtracao/                                                        │║
║   │   │   ├── filtro_1/ ... filtro_8/                                       │║
║   │   │   │   ├── sensor_pressao_entrada                                    │║
║   │   │   │   ├── sensor_pressao_saida                                      │║
║   │   │   │   └── valvula_retrolavagem                                      │║
║   │   │   └── sensor_turbidez_filtrada                                      │║
║   │   │                                                                     │║
║   │   ├── desinfeccao/                                                      │║
║   │   │   ├── dosadora_cloro/                                               │║
║   │   │   │   ├── sensor_nivel_tanque                                       │║
║   │   │   │   └── bomba_dosadora                                            │║
║   │   │   ├── sensor_cloro_residual                                         │║
║   │   │   └── sensor_ph                                                     │║
║   │   │                                                                     │║
║   │   └── reservatorio/                                                     │║
║   │       ├── sensor_nivel                                                  │║
║   │       ├── sensor_turbidez_final                                         │║
║   │       └── sensor_cloro_final                                            │║
║   │                                                                         │║
║   │   REGRAS EMBUTIDAS:                                                     │║
║   │   • Turbidez entrada alta → Aumentar dosagem sulfato                    │║
║   │   • Delta pressão filtro > 0.5 bar → Iniciar retrolavagem               │║
║   │   • Cloro residual < 0.2 ppm → Aumentar dosagem cloro                   │║
║   │   • Nível reservatório < 30% → Alertar operação                         │║
║   │   • Bomba 1 falha → Partir bomba 2 automaticamente                      │║
║   │                                                                         │║
║   └─────────────────────────────────────────────────────────────────────────┘║
║                                                                               ║
║   📈 RESULTADO: Qualidade da água garantida 24/7 com automação               ║
║                                                                               ║
╚═══════════════════════════════════════════════════════════════════════════════╝
```

---

## 📈 Escalabilidade e Números

```
╔═══════════════════════════════════════════════════════════════════════════════╗
║                                                                               ║
║                        📈 ESCALABILIDADE DO SISTEMA                           ║
║                                                                               ║
║   ┌─────────────────────────────────────────────────────────────────────────┐║
║   │                                                                         │║
║   │   CAPACIDADE POR GATEWAY                                                │║
║   │   ──────────────────────                                                │║
║   │                                                                         │║
║   │   Devices Compostos:  █████████████████████████████████  10.000        │║
║   │   Níveis Aninhamento: ████████████                        10 níveis    │║
║   │   Componentes/Device: ████████████████████████████████████ 1.000       │║
║   │   Regras/Template:    ██████████████████████████           100         │║
║   │   Regras Ativas:      █████████████████████████████████████ 100.000    │║
║   │                                                                         │║
║   └─────────────────────────────────────────────────────────────────────────┘║
║                                                                               ║
║   ┌─────────────────────────────────────────────────────────────────────────┐║
║   │                                                                         │║
║   │   PERFORMANCE DE REGRAS                                                 │║
║   │   ─────────────────────                                                 │║
║   │                                                                         │║
║   │   Avaliação de Regra:     < 1ms   ████████████████████████████████     │║
║   │   Propagação de Status:   < 5ms   ██████████████████████████████       │║
║   │   Execução de Ação:       < 10ms  ████████████████████████             │║
║   │   Latência Total:         < 50ms  ██████████████████                   │║
║   │                                                                         │║
║   └─────────────────────────────────────────────────────────────────────────┘║
║                                                                               ║
║   ┌─────────────────────────────────────────────────────────────────────────┐║
║   │                                                                         │║
║   │   REPLICAÇÃO DE TEMPLATES                                               │║
║   │   ───────────────────────                                               │║
║   │                                                                         │║
║   │   ┌────────────────┐                                                   │║
║   │   │   1 TEMPLATE   │                                                   │║
║   │   │   "Gerador     │──────┬──────┬──────┬──────┬──────► N instâncias   │║
║   │   │   Diesel"      │      │      │      │      │                       │║
║   │   └────────────────┘      ▼      ▼      ▼      ▼                       │║
║   │                        Gerador Gerador Gerador Gerador                 │║
║   │                        #001    #002    #003    #NNN                    │║
║   │                                                                         │║
║   │   Tempo de instanciação: < 100ms                                       │║
║   │   Herança de regras: Automática                                        │║
║   │   Atualização de template: Propaga para todas as instâncias            │║
║   │                                                                         │║
║   └─────────────────────────────────────────────────────────────────────────┘║
║                                                                               ║
╚═══════════════════════════════════════════════════════════════════════════════╝
```

---

## 💡 Inovações Disruptivas

### 1. Template DNA - Herança de Comportamento

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│  🧬 TEMPLATE DNA - HERANÇA COMPORTAMENTAL                                   │
│  ─────────────────────────────────────────                                  │
│                                                                             │
│  Assim como DNA biológico, templates carregam "genes" de comportamento:    │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                                                                     │   │
│  │   TEMPLATE "Motor Industrial"                                       │   │
│  │   ─────────────────────────────                                     │   │
│  │                                                                     │   │
│  │   GENES (Comportamentos Herdáveis):                                 │   │
│  │   ┌───────────────────────────────────────────────────────────┐    │   │
│  │   │ 🧬 gene_protecao_termica                                   │    │   │
│  │   │    → Desligar se temperatura > limite por X segundos       │    │   │
│  │   │                                                            │    │   │
│  │   │ 🧬 gene_vibracao_excessiva                                 │    │   │
│  │   │    → Alertar manutenção se vibração > threshold            │    │   │
│  │   │                                                            │    │   │
│  │   │ 🧬 gene_ciclo_partida                                      │    │   │
│  │   │    → Soft-start com rampa de 5 segundos                    │    │   │
│  │   │                                                            │    │   │
│  │   │ 🧬 gene_manutencao_preditiva                               │    │   │
│  │   │    → Calcular MTBF baseado em horas de operação            │    │   │
│  │   └───────────────────────────────────────────────────────────┘    │   │
│  │                                                                     │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  HERANÇA:                                                                   │
│  ─────────                                                                  │
│                                                                             │
│  ┌────────────────┐    ┌────────────────┐    ┌────────────────┐            │
│  │ Motor Bomba    │    │ Motor Ventilador│    │ Motor Compressor│           │
│  │ (herda genes)  │    │ (herda genes)   │    │ (herda genes)   │           │
│  │                │    │                 │    │                 │           │
│  │ + gene_cavit.  │    │ + gene_filtro   │    │ + gene_pressao  │           │
│  └────────────────┘    └─────────────────┘    └─────────────────┘           │
│                                                                             │
│  ✨ Comportamentos comuns definidos uma vez, herdados por todos!           │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

### 2. Shadow Testing - Teste de Regras em Sombra

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│  👥 SHADOW TESTING - VALIDAÇÃO SEGURA DE REGRAS                             │
│  ───────────────────────────────────────────────                            │
│                                                                             │
│  Teste novas regras sem afetar a produção:                                  │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                                                                     │   │
│  │   DADOS DE SENSORES (PRODUÇÃO)                                      │   │
│  │   ─────────────────────────────                                     │   │
│  │            │                                                        │   │
│  │            ▼                                                        │   │
│  │   ┌───────────────┐        ┌───────────────────────────────────┐   │   │
│  │   │               │        │                                   │   │   │
│  │   │  REGRAS       │        │  REGRAS EM SHADOW MODE            │   │   │
│  │   │  PRODUÇÃO     │        │  (Nova versão sendo testada)      │   │   │
│  │   │               │        │                                   │   │   │
│  │   │  ✅ Executa   │        │  📊 Avalia mas NÃO executa        │   │   │
│  │   │     ações     │        │     Apenas loga o que FARIA       │   │   │
│  │   │               │        │                                   │   │   │
│  │   └───────┬───────┘        └───────────────┬───────────────────┘   │   │
│  │           │                                │                        │   │
│  │           ▼                                ▼                        │   │
│  │   ┌───────────────┐        ┌───────────────────────────────────┐   │   │
│  │   │ ATUADORES     │        │ RELATÓRIO COMPARATIVO             │   │   │
│  │   │ (ações reais) │        │                                   │   │   │
│  │   └───────────────┘        │ • Regra atual: 5 acionamentos/dia │   │   │
│  │                            │ • Nova regra: 3 acionamentos/dia  │   │   │
│  │                            │ • Economia estimada: 40%          │   │   │
│  │                            │                                   │   │   │
│  │                            │ [ ✅ Promover para Produção ]     │   │   │
│  │                            └───────────────────────────────────┘   │   │
│  │                                                                     │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ✨ Zero risco ao testar novas regras em ambiente real!                    │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

### 3. Auto-Discovery de Componentes

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│  🔍 AUTO-DISCOVERY DE COMPONENTES                                           │
│  ─────────────────────────────────                                          │
│                                                                             │
│  O sistema sugere componentes baseado em padrões industriais:               │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                                                                     │   │
│  │   VOCÊ CRIOU: "Bomba Centrífuga"                                    │   │
│  │                                                                     │   │
│  │   💡 SUGESTÕES AUTOMÁTICAS:                                         │   │
│  │   ─────────────────────────                                         │   │
│  │                                                                     │   │
│  │   ┌─────────────────────────────────────────────────────────────┐  │   │
│  │   │                                                             │  │   │
│  │   │   🌡️ Sensor de Temperatura do Motor                         │  │   │
│  │   │      Recomendação: 95% das bombas possuem                   │  │   │
│  │   │      [ ✅ Adicionar ] [ ❌ Ignorar ]                        │  │   │
│  │   │                                                             │  │   │
│  │   │   📊 Sensor de Pressão de Saída                             │  │   │
│  │   │      Recomendação: Essencial para monitoramento             │  │   │
│  │   │      [ ✅ Adicionar ] [ ❌ Ignorar ]                        │  │   │
│  │   │                                                             │  │   │
│  │   │   📳 Sensor de Vibração                                     │  │   │
│  │   │      Recomendação: Manutenção preditiva                     │  │   │
│  │   │      [ ✅ Adicionar ] [ ❌ Ignorar ]                        │  │   │
│  │   │                                                             │  │   │
│  │   │   📊 Sensor de Vazão                                        │  │   │
│  │   │      Recomendação: Controle de processo                     │  │   │
│  │   │      [ ✅ Adicionar ] [ ❌ Ignorar ]                        │  │   │
│  │   │                                                             │  │   │
│  │   │   🔌 Inversor de Frequência                                 │  │   │
│  │   │      Recomendação: Eficiência energética                    │  │   │
│  │   │      [ ✅ Adicionar ] [ ❌ Ignorar ]                        │  │   │
│  │   │                                                             │  │   │
│  │   └─────────────────────────────────────────────────────────────┘  │   │
│  │                                                                     │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ✨ IA sugere componentes baseada em milhares de templates industriais!    │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

### 4. Versionamento de Templates com Rollback

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│  📚 VERSIONAMENTO DE TEMPLATES                                              │
│  ─────────────────────────────                                              │
│                                                                             │
│  Cada alteração no template é versionada com possibilidade de rollback:     │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                                                                     │   │
│  │   HISTÓRICO: Gerador Diesel 500KVA                                  │   │
│  │   ───────────────────────────────────                               │   │
│  │                                                                     │   │
│  │   v3.2.1 (ATUAL) ───────────────────────────────────────────────   │   │
│  │   │  📅 2025-12-19 14:30                                           │   │
│  │   │  👤 eng.silva                                                  │   │
│  │   │  📝 Ajustado threshold de temperatura para 110°C               │   │
│  │   │                                                                 │   │
│  │   │                                        [ 🔄 Rollback ]          │   │
│  │   │                                                                 │   │
│  │   v3.2.0 ───────────────────────────────────────────────────────   │   │
│  │   │  📅 2025-12-15 09:15                                           │   │
│  │   │  👤 eng.santos                                                 │   │
│  │   │  📝 Adicionada regra de pressão de óleo                        │   │
│  │   │                                                                 │   │
│  │   │                                        [ 🔄 Rollback ]          │   │
│  │   │                                                                 │   │
│  │   v3.1.0 ───────────────────────────────────────────────────────   │   │
│  │   │  📅 2025-12-01 16:45                                           │   │
│  │   │  👤 eng.oliveira                                               │   │
│  │   │  📝 Novo componente: sistema de arrefecimento                  │   │
│  │   │                                                                 │   │
│  │   │                                        [ 🔄 Rollback ]          │   │
│  │   │                                                                 │   │
│  │   v3.0.0 (MAJOR) ───────────────────────────────────────────────   │   │
│  │      📅 2025-11-15 10:00                                           │   │
│  │      👤 arq.pereira                                                │   │
│  │      📝 Reestruturação completa do template                        │   │
│  │                                                                     │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  PROPAGAÇÃO DE VERSÃO:                                                      │
│  ─────────────────────                                                      │
│                                                                             │
│  ○ Aplicar automaticamente a todas as instâncias                           │
│  ● Aplicar apenas a novas instâncias                                       │
│  ○ Perguntar para cada instância                                           │
│                                                                             │
│  ✨ Nunca perca uma configuração que funcionava!                           │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## ⚔️ Comparativo de Mercado

```
╔═══════════════════════════════════════════════════════════════════════════════╗
║                                                                               ║
║   ⚔️ CORGRID vs CONCORRÊNCIA - DEVICES COMPOSTOS                             ║
║                                                                               ║
║   ┌─────────────────────────────────────────────────────────────────────┐    ║
║   │ Feature                   │ CorGrid │ Siemens │ Ignition │ ThingsB.│    ║
║   ├───────────────────────────┼─────────┼─────────┼──────────┼─────────┤    ║
║   │ Devices Compostos Nativos │   ✅    │   ⚠️    │    ⚠️    │   ❌    │    ║
║   │ Aninhamento Ilimitado     │   ✅    │   ⚠️    │    ⚠️    │   ❌    │    ║
║   │ Regras Embutidas          │   ✅    │   ⚠️    │    ✅    │   ❌    │    ║
║   │ Herança de Templates      │   ✅    │   ❌    │    ❌    │   ❌    │    ║
║   │ Propagação de Status      │   ✅    │   ⚠️    │    ⚠️    │   ❌    │    ║
║   │ Shadow Testing            │   ✅    │   ❌    │    ❌    │   ❌    │    ║
║   │ Auto-Discovery            │   ✅    │   ⚠️    │    ❌    │   ❌    │    ║
║   │ Versionamento Templates   │   ✅    │   ⚠️    │    ⚠️    │   ⚠️    │    ║
║   │ Modelo 3D Integrado       │   ✅    │   ❌    │    ❌    │   ❌    │    ║
║   │ API Criptografada E2E     │   ✅    │   ✅    │    ✅    │   ⚠️    │    ║
║   └─────────────────────────────────────────────────────────────────────┘    ║
║                                                                               ║
║   ✅ = Completo   ⚠️ = Parcial/Limitado   ❌ = Não disponível                ║
║                                                                               ║
║   💡 DIFERENCIAIS ÚNICOS DO CORGRID:                                         ║
║   ──────────────────────────────────                                          ║
║                                                                               ║
║   1. Template DNA - Herança comportamental entre templates                   ║
║   2. Shadow Testing - Teste regras em sombra sem afetar produção             ║
║   3. Composição Visual - Arraste componentes da biblioteca                   ║
║   4. Modelo 3D Composto - Auto-geração a partir de componentes               ║
║   5. Regras Viajantes - Automação embedded no template                       ║
║                                                                               ║
╚═══════════════════════════════════════════════════════════════════════════════╝
```

---

## 🚀 Começando em 10 Minutos

### Passo 1: Acesse o Device Composer

```
Dashboard → Biblioteca → Device Template Composer
```

### Passo 2: Escolha "Novo Device Composto"

```
[ + Novo Template ] → Device Composto
```

### Passo 3: Arraste Componentes da Biblioteca

```
Biblioteca (esquerda) → Área de Composição (direita)
```

### Passo 4: Defina Conexões e Regras

```
Aba "Conexões" → Aba "Regras" → Configure automações
```

### Passo 5: Publique e Instancie

```
[ ✅ Validar ] → [ 📤 Publicar ] → [ 🏭 Criar Instância ]
```

---

## 📞 Suporte e Recursos

```
╔═══════════════════════════════════════════════════════════════════════════════╗
║                                                                               ║
║   📧 Enterprise Support:     enterprise@corgrid.io                           ║
║   📖 Documentação:           https://docs.corgrid.io/device-composer         ║
║   📹 Vídeo Tutorial:         https://learn.corgrid.io/composite-devices      ║
║   💬 Community:              https://community.corgrid.io                    ║
║   🐙 GitHub:                 https://github.com/corgrid                      ║
║                                                                               ║
╚═══════════════════════════════════════════════════════════════════════════════╝
```

---

**Versão do Documento:** 1.0
**Última Atualização:** Dezembro 2025
**Mantido por:** CorGrid Industrial Engineering Team

---

```
╔═══════════════════════════════════════════════════════════════════════════════╗
║                                                                               ║
║               🔧 CORGRID OS - DEVICE TEMPLATE COMPOSER                        ║
║                                                                               ║
║        "Modele equipamentos complexos como eles realmente são.                ║
║         Regras e monitoramento incluídos. Replique infinitamente."            ║
║                                                                               ║
╚═══════════════════════════════════════════════════════════════════════════════╝
```
