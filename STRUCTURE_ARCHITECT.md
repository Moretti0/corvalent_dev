# CorGrid OS - Structure Architect

**Document ID:** STRUCTURE-ARCHITECT-V2
**Data:** 20/12/2025
**Propósito:** Source of Truth para modelagem hierárquica de instalações
**Autoridade:** Core Architecture Team

---

## Índice

1. [Visão Geral](#1-visão-geral)
2. [Hierarquia de Entidades](#2-hierarquia-de-entidades)
3. [Interface StructureTemplate](#3-interface-structuretemplate)
4. [Presets de Categorias](#4-presets-de-categorias)
5. [Wizard de Criação](#5-wizard-de-criação)
6. [Verticais de Aplicação](#6-verticais-de-aplicação)
7. [Sistema de Slots](#7-sistema-de-slots)
8. [Geolocalização](#8-geolocalização)
9. [Regras Cross-Zone](#9-regras-cross-zone)
10. [API REST](#10-api-rest)
11. [Persistência](#11-persistência)
12. [Roadmap](#12-roadmap)

---

## 1. Visão Geral

O **Structure Architect** permite modelar instalações físicas completas através de uma hierarquia de entidades. A abordagem é multicamadas, onde cada nível agrega o nível inferior:

```
Structure → Zone → Cell → Device → Sensor/Actuator
```

Cada camada:
- Herda configurações do nível superior
- Propaga status para o nível superior
- Pode ter modelo 3D/2D próprio
- Suporta geolocalização (GPS outdoor ou posição relativa indoor)

---

## 2. Hierarquia de Entidades

### 2.1. Pirâmide de Abstração

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│                           HIERARQUIA CORGRID                            │
│                                                                         │
│                              ┌───────────┐                              │
│                              │ STRUCTURE │                              │
│                              │ (Edifício,│                              │
│                              │  Planta)  │                              │
│                              └─────┬─────┘                              │
│                                    │                                    │
│                    ┌───────────────┼───────────────┐                    │
│                    ▼               ▼               ▼                    │
│               ┌────────┐     ┌────────┐     ┌────────┐                  │
│               │  ZONE  │     │  ZONE  │     │  ZONE  │                  │
│               │ (Andar,│     │ (Área, │     │(Setor) │                  │
│               │ Bloco) │     │  Sala) │     │        │                  │
│               └───┬────┘     └───┬────┘     └───┬────┘                  │
│                   │              │              │                       │
│            ┌──────┴──────┐      ...            ...                      │
│            ▼             ▼                                              │
│       ┌────────┐   ┌────────┐                                           │
│       │  CELL  │   │  CELL  │                                           │
│       │(Sala,  │   │(Posto) │                                           │
│       │Estação)│   │        │                                           │
│       └───┬────┘   └────────┘                                           │
│           │                                                             │
│    ┌──────┴──────┐                                                      │
│    ▼             ▼                                                      │
│ ┌──────┐    ┌──────┐                                                    │
│ │DEVICE│    │DEVICE│                                                    │
│ │      │    │      │                                                    │
│ └──┬───┘    └──────┘                                                    │
│    │                                                                    │
│ ┌──┴──┐                                                                 │
│ │SENS.│                                                                 │
│ └─────┘                                                                 │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 2.2. Definição de Cada Nível

| Nível | Descrição | Exemplos |
|-------|-----------|----------|
| **Structure** | Instalação física completa | Edifício, Campus, Planta Industrial |
| **Zone** | Subdivisão lógica/física | Andar, Ala, Setor, Área Externa |
| **Cell** | Unidade operacional | Sala, Estação de Trabalho, Posto |
| **Device** | Equipamento/Máquina | Gateway, Controlador, Medidor |
| **Sensor/Actuator** | Ponto de medição/controle | Temperatura, Relé, Válvula |

---

## 3. Interface StructureTemplate

### 3.1. Definição

**Fonte:** `www/src/stores/useStructureWizardStore.ts:15-33`

```typescript
export interface StructureTemplate {
  id: string;
  metadata: {
    name: string;
    description?: string;
    category?: string;  // 'building', 'floor', 'room', 'outdoor'
    specs?: Record<string, any>;
  };
  assets: {
    image_2d?: string;  // Planta baixa
    model_3d?: string;  // Modelo BIM/GLTF
    manual?: string;    // Documentação
  };
  camera?: {
    position: [number, number, number];
    target: [number, number, number];
  };
  slots: StructureSlot[];  // Pontos de montagem
}
```

### 3.2. StructureSlot

**Fonte:** `www/src/stores/useStructureWizardStore.ts:5-13`

```typescript
export interface StructureSlot {
  id: string;
  name: string;
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

---

## 4. Presets de Categorias

### 4.1. Categorias Disponíveis

**Fonte:** `www/src/constants/structurePresets.ts`

O sistema oferece 4 categorias base de structures:

| ID | Label | Descrição | Ícone |
|----|-------|-----------|-------|
| `building` | Building / Site | Edifício ou campus completo | Building |
| `floor` | Floor / Level | Andar dentro de um edifício | Layers |
| `room` | Room / Space | Espaço fechado (sala, servidor) | Box |
| `outdoor` | Outdoor Zone | Área externa (estacionamento, jardim) | Sun |

### 4.2. Preset: Building

```typescript
{
  id: 'building',
  label: 'Building / Site',
  description: 'Entire physical building or campus',
  icon: 'Building',
  defaultSlots: [
    { id: 'main_entrance', name: 'Main Entrance' },
    { id: 'lobby', name: 'Lobby / Reception' },
    { id: 'roof', name: 'Rooftop Area' },
    { id: 'basement', name: 'Basement / Utility' }
  ],
  specs: [
    { key: 'total_area', label: 'Total Area', type: 'number', unit: 'm²', required: true },
    { key: 'floors', label: 'Number of Floors', type: 'number', required: true },
    { key: 'address', label: 'Address', type: 'string', required: true },
    { key: 'type', label: 'Building Type', type: 'enum',
      options: ['Office', 'Industrial', 'Residential', 'Data Center', 'Mixed'] },
    { key: 'construction_year', label: 'Construction Year', type: 'number' }
  ]
}
```

### 4.3. Preset: Floor

```typescript
{
  id: 'floor',
  label: 'Floor / Level',
  description: 'A single level within a building',
  icon: 'Layers',
  defaultSlots: [
    { id: 'corridor_main', name: 'Main Corridor' },
    { id: 'elevator_lobby', name: 'Elevator Lobby' },
    { id: 'restrooms', name: 'Restrooms Area' },
    { id: 'utility_room', name: 'Utility / Electrical Room' }
  ],
  specs: [
    { key: 'level_number', label: 'Level Number', type: 'number', required: true },
    { key: 'floor_area', label: 'Floor Area', type: 'number', unit: 'm²', required: true },
    { key: 'ceiling_height', label: 'Ceiling Height', type: 'number', unit: 'm' },
    { key: 'access_control', label: 'Access Control', type: 'enum',
      options: ['Public', 'Restricted', 'High Security'] }
  ]
}
```

### 4.4. Preset: Room

```typescript
{
  id: 'room',
  label: 'Room / Space',
  description: 'Enclosed space (Office, Server Room, Storage)',
  icon: 'Box',
  defaultSlots: [
    { id: 'ceiling_center', name: 'Ceiling Center' },
    { id: 'door_entry', name: 'Main Door' },
    { id: 'window_1', name: 'Window Area 1' },
    { id: 'wall_north', name: 'North Wall' }
  ],
  specs: [
    { key: 'capacity', label: 'Max Capacity', type: 'number', unit: 'people' },
    { key: 'area', label: 'Area', type: 'number', unit: 'm²', required: true },
    { key: 'function', label: 'Function', type: 'enum',
      options: ['Office', 'Meeting', 'Server Room', 'Storage', 'Utility', 'Corridor'] },
    { key: 'hvac_zone', label: 'HVAC Zone ID', type: 'string' }
  ]
}
```

### 4.5. Preset: Outdoor

```typescript
{
  id: 'outdoor',
  label: 'Outdoor Zone',
  description: 'Parking lot, Garden, Loading Dock',
  icon: 'Sun',
  defaultSlots: [
    { id: 'gate_main', name: 'Main Gate' },
    { id: 'parking_a', name: 'Parking Section A' },
    { id: 'perimeter_fence', name: 'Perimeter Fence' }
  ],
  specs: [
    { key: 'area', label: 'Zone Area', type: 'number', unit: 'm²' },
    { key: 'surface_type', label: 'Surface', type: 'enum',
      options: ['Asphalt', 'Grass', 'Concrete', 'Gravel'] },
    { key: 'lighting', label: 'Lighting Coverage', type: 'enum',
      options: ['Full', 'Partial', 'None'] }
  ]
}
```

---

## 5. Wizard de Criação

### 5.1. Store do Wizard

**Fonte:** `www/src/stores/useStructureWizardStore.ts:71-184`

```typescript
export const useStructureWizardStore = create<WizardState>()(
  devtools(
    persist(
      (set) => ({
        currentStep: 1,
        template: INITIAL_TEMPLATE,
        localBlobs: {},
        localFiles: {},
        detectedModelFormat: 'glb',
        errors: {},

        setStep: (step) => set({ currentStep: step }),
        updateMetadata: (data) => set((state) => ({...})),
        updateAssets: (data) => set((state) => ({...})),
        addSlot: (slot) => set((state) => ({...})),
        removeSlot: (id) => set((state) => ({...})),
        populateSlotsFromPreset: (categoryId) => set((state) => ({...})),
        resetWizard: () => set({...}),
      }),
      { name: 'structure-wizard-storage' }
    )
  )
);
```

### 5.2. Actions Disponíveis

| Action | Descrição |
|--------|-----------|
| `setStep(step)` | Navegar para passo do wizard |
| `updateMetadata(data)` | Atualizar metadados do template |
| `updateAssets(data)` | Atualizar assets (imagem, modelo 3D) |
| `updateCamera(camera)` | Configurar posição da câmera 3D |
| `addSlot(slot)` | Adicionar novo slot |
| `updateSlot(id, updates)` | Atualizar slot existente |
| `removeSlot(id)` | Remover slot |
| `populateSlotsFromPreset(categoryId)` | Carregar slots padrão da categoria |
| `resetWizard()` | Reiniciar wizard |

---

## 6. Verticais de Aplicação

O sistema de hierarquia Structure → Zone → Cell → Device suporta diversas verticais. A seguir, exemplos de como organizar cada vertical usando a hierarquia existente.

### 6.1. Edifícios Comerciais

```
Structure: Edifício
├── Zone: Andar Térreo
│   ├── Cell: Recepção
│   │   └── Device: Gateway Zigbee
│   │       ├── Sensor: Presença
│   │       ├── Sensor: Temperatura
│   │       └── Atuador: Iluminação
│   ├── Cell: Sala Reunião 01
│   │   └── Device: Controlador HVAC
│   │       ├── Sensor: CO2
│   │       └── Atuador: Ar Condicionado
│   └── Cell: Sala Elétrica
│       └── Device: Medidor de Energia
│           ├── Sensor: Tensão
│           ├── Sensor: Corrente
│           └── Sensor: Potência
├── Zone: 1º Andar
│   └── Cell: Escritório Open Space
│       └── Device: Gateway LoRaWAN
│           ├── Sensor: Ocupação
│           └── Sensor: Luminosidade
└── Zone: Cobertura
    └── Cell: Casa de Máquinas
        └── Device: Chiller
            ├── Sensor: Temperatura Água
            └── Sensor: Pressão
```

### 6.2. Hospitais e Centros de Saúde

```
Structure: Hospital
├── Zone: Emergência
│   ├── Cell: Triagem
│   │   └── Device: Gateway
│   │       └── Sensor: Presença
│   ├── Cell: Sala Observação 01
│   │   └── Device: Monitor Ambiental
│   │       ├── Sensor: Temperatura
│   │       ├── Sensor: Umidade
│   │       └── Sensor: CO2
│   └── Cell: Sala Medicação
│       └── Device: Refrigerador Controlado
│           └── Sensor: Temperatura Interna
├── Zone: Centro Cirúrgico
│   ├── Cell: Sala 01
│   │   └── Device: Controlador HVAC Classe A
│   │       ├── Sensor: Pressão Diferencial
│   │       ├── Sensor: Partículas
│   │       └── Sensor: Temperatura
│   └── Cell: Sala 02
│       └── (mesma estrutura)
├── Zone: Internação
│   └── Cell: Quarto 101..150
│       └── Device: Painel Leito
│           └── Sensor: Chamada Enfermagem
└── Zone: Utilidades
    ├── Cell: Central de Gases
    │   └── Device: Manifold
    │       ├── Sensor: Pressão O2
    │       └── Sensor: Nível Tanque
    └── Cell: Subestação
        └── Device: Medidor
            └── Sensor: Demanda
```

### 6.3. Subestações e Energia

```
Structure: Subestação 138kV
├── Zone: Pátio Alta Tensão
│   ├── Cell: Bay Entrada L1
│   │   └── Device: Disjuntor SF6
│   │       ├── Sensor: Pressão SF6
│   │       ├── Sensor: Temperatura
│   │       └── Sensor: Contador Operações
│   ├── Cell: Bay Transformador T1
│   │   └── Device: Transformador
│   │       ├── Sensor: Temperatura Óleo
│   │       ├── Sensor: Temperatura Enrolamento
│   │       ├── Sensor: Nível Óleo
│   │       └── Sensor: Cromatografia Gases
│   └── Cell: Bay Saída
│       └── Device: Chave Seccionadora
│           └── Sensor: Posição
├── Zone: Casa de Comando
│   └── Cell: Sala Controle
│       └── Device: CLP Proteção
│           ├── Sensor: Corrente Fase A
│           ├── Sensor: Corrente Fase B
│           └── Sensor: Corrente Fase C
└── Zone: Perímetro
    └── Cell: Cerca
        └── Device: Sensor Perimetral
            └── Sensor: Vibração
```

### 6.4. Cidades Inteligentes

```
Structure: Distrito Centro
├── Zone: Avenida Principal
│   ├── Cell: Cruzamento 01
│   │   └── Device: Controlador Semáforo
│   │       ├── Sensor: Contagem Veículos
│   │       └── Atuador: Tempo Fase
│   ├── Cell: Poste 15
│   │   └── Device: Gateway Iluminação
│   │       ├── Sensor: Luminosidade
│   │       └── Atuador: Dimmer LED
│   └── Cell: Ponto de Ônibus 03
│       └── Device: Painel Informativo
│           └── Sensor: Presença Passageiros
├── Zone: Praça Central
│   └── Cell: Quiosque
│       └── Device: Estação Ambiental
│           ├── Sensor: Qualidade Ar (PM2.5)
│           ├── Sensor: Ruído
│           └── Sensor: Temperatura
└── Zone: Estacionamento Público
    └── Cell: Setor A
        └── Device: Gateway LoRa
            └── Sensor: Vaga Ocupada (x50)
```

### 6.5. Fazendas e Agro

```
Structure: Propriedade Rural
├── Zone: Talhão Norte
│   ├── Cell: Pivô 01
│   │   └── Device: Controlador Irrigação
│   │       ├── Sensor: Pressão Entrada
│   │       ├── Sensor: Vazão
│   │       ├── Sensor: Posição Angular
│   │       └── Atuador: Bomba
│   └── Cell: Ponto Solo 01
│       └── Device: Estação Solo
│           ├── Sensor: Umidade 10cm
│           ├── Sensor: Umidade 30cm
│           └── Sensor: Temperatura Solo
├── Zone: Armazenagem
│   └── Cell: Silo 01
│       └── Device: Controlador Aeração
│           ├── Sensor: Temperatura Cabo (x8)
│           ├── Sensor: Umidade Grãos
│           └── Atuador: Ventilador
├── Zone: Estação Meteorológica
│   └── Cell: Torre Met
│       └── Device: Datalogger
│           ├── Sensor: Temperatura Ar
│           ├── Sensor: Umidade Relativa
│           ├── Sensor: Velocidade Vento
│           ├── Sensor: Radiação Solar
│           └── Sensor: Pluviômetro
└── Zone: Sede
    └── Cell: Escritório
        └── Device: Gateway
            └── Sensor: Consumo Energia
```

---

## 7. Sistema de Slots

### 7.1. Conceito

Slots são **pontos de montagem** dentro de uma Structure onde Zones ou Devices podem ser conectados. Funcionam como "encaixes" pré-definidos na planta.

### 7.2. Populando Slots a partir de Preset

**Fonte:** `www/src/stores/useStructureWizardStore.ts:130-152`

```typescript
populateSlotsFromPreset: (categoryId) => set((state) => {
  const preset = STRUCTURE_PRESETS.find(p => p.id === categoryId);
  if (!preset || !preset.defaultSlots) return {};

  const existingIds = new Set(state.template.slots.map(s => s.id));
  const newSlots = preset.defaultSlots
    .filter(def => !existingIds.has(def.id))
    .map(def => ({
      id: def.id,
      name: def.name,
      geometry: null,
      ui2d: null
    }));

  if (newSlots.length === 0) return {};

  return {
    template: {
      ...state.template,
      slots: [...state.template.slots, ...newSlots]
    }
  };
}),
```

### 7.3. Slots Padrão por Categoria

| Categoria | Slots Padrão |
|-----------|--------------|
| Building | main_entrance, lobby, roof, basement |
| Floor | corridor_main, elevator_lobby, restrooms, utility_room |
| Room | ceiling_center, door_entry, window_1, wall_north |
| Outdoor | gate_main, parking_a, perimeter_fence |

---

## 8. Geolocalização

### 8.1. Coordenadas Suportadas

O sistema suporta dois tipos de coordenadas:

| Tipo | Uso | Formato |
|------|-----|---------|
| **GPS (WGS84)** | Outdoor | Lat, Lon, Altitude |
| **Local (metros)** | Indoor | X, Y, Z relativos |

### 8.2. Estrutura de Posição

```typescript
interface GeoPosition {
  type: 'gps' | 'local';
  coordinates: {
    // GPS
    latitude?: number;
    longitude?: number;
    altitude?: number;
    // Local
    x?: number;
    y?: number;
    z?: number;
  };
  reference?: string;  // ID da Structure pai para coordenadas locais
}
```

### 8.3. Integração com Modelos 3D

Quando um modelo BIM/IFC é carregado:
- `IfcBuilding` → Structure
- `IfcBuildingStorey` → Zone
- `IfcSpace` → Cell
- `IfcDistributionElement` → Device

---

## 9. Regras Cross-Zone

### 9.1. Escopo de Regras

Regras podem operar em diferentes escopos:

| Escopo | Descrição |
|--------|-----------|
| `STRUCTURE` | Afeta todas as Zones |
| `ZONE` | Afeta todas as Cells da Zone |
| `CELL` | Afeta todos os Devices da Cell |
| `DEVICE` | Afeta apenas o Device específico |

### 9.2. Propagação de Status

O status de uma Structure é o **pior status** entre seus filhos:

```
HEALTHY < WARNING < DEGRADED < CRITICAL < OFFLINE
```

Se qualquer Zone estiver CRITICAL, a Structure também será CRITICAL.

### 9.3. Herança de Configurações

Configurações definidas em nível superior são herdadas:

```
Structure (timezone: America/Sao_Paulo)
└── Zone (herda timezone)
    └── Cell (herda timezone)
        └── Device (herda timezone)
```

Override é possível em qualquer nível.

---

## 10. API REST

### 10.1. Endpoints de Structures

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| GET | `/api/v1/structures` | Listar structures |
| GET | `/api/v1/structures/:id` | Obter structure |
| POST | `/api/v1/structures` | Criar structure |
| PUT | `/api/v1/structures/:id` | Atualizar structure |
| DELETE | `/api/v1/structures/:id` | Excluir structure |
| GET | `/api/v1/structures/:id/zones` | Listar zones |
| POST | `/api/v1/structures/:id/zones` | Criar zone |

### 10.2. Endpoints de Zones

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| GET | `/api/v1/zones/:id` | Obter zone |
| PUT | `/api/v1/zones/:id` | Atualizar zone |
| DELETE | `/api/v1/zones/:id` | Excluir zone |
| GET | `/api/v1/zones/:id/cells` | Listar cells |
| POST | `/api/v1/zones/:id/cells` | Criar cell |

### 10.3. Endpoints de Cells

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| GET | `/api/v1/cells/:id` | Obter cell |
| PUT | `/api/v1/cells/:id` | Atualizar cell |
| DELETE | `/api/v1/cells/:id` | Excluir cell |
| GET | `/api/v1/cells/:id/devices` | Listar devices |
| POST | `/api/v1/cells/:id/devices` | Vincular device |

---

## 11. Persistência

### 11.1. Formato JSON

```json
{
  "id": "struct_hospital_central",
  "version": "1.0.0",
  "created_at": "2025-12-20T10:00:00Z",
  "metadata": {
    "name": "Hospital Central",
    "category": "building",
    "specs": {
      "total_area": 15000,
      "floors": 5,
      "address": "Av. Principal, 1000",
      "type": "Hospital"
    }
  },
  "assets": {
    "image_2d": "/uploads/structures/hospital_planta.png",
    "model_3d": "/uploads/structures/hospital.glb"
  },
  "camera": {
    "position": [50, 30, 50],
    "target": [0, 0, 0]
  },
  "slots": [
    { "id": "main_entrance", "name": "Entrada Principal" },
    { "id": "emergency", "name": "Acesso Emergência" }
  ],
  "zones": [
    {
      "id": "zone_emergencia",
      "name": "Emergência",
      "category": "floor",
      "cells": [...]
    }
  ]
}
```

### 11.2. Zustand com Persistência

O wizard persiste automaticamente no localStorage:

```typescript
{
  name: 'structure-wizard-storage',
  partialize: (state) => ({
    currentStep: state.currentStep,
    template: state.template,
    detectedModelFormat: state.detectedModelFormat
  })
}
```

---

## 12. Roadmap

### 12.1. Funcionalidades Atuais

| Feature | Status |
|---------|--------|
| Hierarquia Structure → Zone → Cell → Device | ✅ Implementado |
| 4 Presets de Categoria (building, floor, room, outdoor) | ✅ Implementado |
| Sistema de Slots | ✅ Implementado |
| Persistência do Wizard | ✅ Implementado |
| Upload de Modelo 3D (GLB/GLTF) | ✅ Implementado |

### 12.2. Funcionalidades Planejadas

| Feature | Status | Prioridade |
|---------|--------|------------|
| Importação BIM/IFC | 📋 Planejado | Alta |
| Visualização 3D Navegável | 🔄 Em Progresso | Alta |
| Regras Cross-Zone | 📋 Planejado | Alta |
| Propagação de Status | 📋 Planejado | Média |
| Templates de Verticais | 📋 Planejado | Média |
| Geolocalização GPS | 📋 Planejado | Média |

---

## Referências de Código

| Arquivo | Linhas | Descrição |
|---------|--------|-----------|
| `www/src/stores/useStructureWizardStore.ts` | 185 | Store do Structure Wizard |
| `www/src/constants/structurePresets.ts` | 99 | Presets de categorias |

---

**Versão:** 2.0
**Última Atualização:** 20/12/2025
**Mantenedor:** Core Architecture Team
