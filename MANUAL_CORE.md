# MANUAL DE ARQUITETURA DE KERNEL: CORGRID CORE (THE SOURCE OF TRUTH)

**Document ID:** ENG-CORE-TRUTH-V5
**Data:** 19/12/2025
**Escopo:** `src/core/` + `include/corgrid/core/`
**Autoridade:** Architecture Team / Engineering Lead
**Arquitetura:** Microkernel Híbrido Assíncrono (C++23)
**Status:** Source of Truth - Baseado em Análise Real do Código

---

## ÍNDICE

1.  [Filosofia Arquitetural](#1-filosofia-arquitetural)
2.  [Inventário do Core (Dados Reais)](#2-inventário-do-core-dados-reais)
3.  [Sistema de Injeção de Dependência (DIContainer)](#3-sistema-de-injeção-de-dependência-dicontainer)
4.  [Ciclo de Vida de Serviços (IServiceBase)](#4-ciclo-de-vida-de-serviços-iservicebase)
5.  [O Processo de Bootstrap (Application.cpp)](#5-o-processo-de-bootstrap-applicationcpp)
6.  [Orquestração de Serviços (ServiceOrchestrator)](#6-orquestração-de-serviços-serviceorchestrator)
7.  [Shutdown Graceful (GracefulShutdownManager)](#7-shutdown-graceful-gracefulshutdownmanager)
8.  [Hot-Reload (ProcessLifecycleManager)](#8-hot-reload-processlifecyclemanager)
9.  [Sistema de Métricas (MetricRegistry)](#9-sistema-de-métricas-metricregistry)
10. [Tratamento de Erros (ErrorHandler)](#10-tratamento-de-erros-errorhandler)
11. [Sistema de Configuração](#11-sistema-de-configuração)
12. [Sistema de Logging](#12-sistema-de-logging)
13. [Modelo de Concorrência e Threading](#13-modelo-de-concorrência-e-threading)
14. [Integração com Plugins](#14-integração-com-plugins)
15. [C++23 Features em Uso](#15-c23-features-em-uso)
16. [Roadmap de Engenharia (C++23)](#16-roadmap-de-engenharia-c23)
17. [Guia de Desenvolvimento](#17-guia-de-desenvolvimento)
18. [Debugging e Troubleshooting](#18-debugging-e-troubleshooting)

---

## 1. FILOSOFIA ARQUITETURAL

O Core do CorGrid é um **Microkernel de Aplicação** - não uma aplicação monolítica. Sua função primária é orquestrar o ciclo de vida de módulos, não executar regras de negócio diretamente.

### 1.1. Princípios Fundamentais

| Princípio | Descrição | Implementação Real |
|-----------|-----------|-------------------|
| **Imutabilidade na Inicialização** | Grafo de dependências imutável após RUNNING | `DIContainer` sela registros após boot |
| **Fail-Fast no Boot** | Aborta se config crítica faltando | `Application.cpp:138-144` - throw em initializeServices |
| **Reactor Pattern** | Sistema orientado a eventos | Thread principal dedicada ao io_context |
| **Ownership Explícita** | Smart pointers com semântica clara | `unique_ptr` para posse, `shared_ptr` para serviços compartilhados |
| **RAII Total** | Zero `new`/`delete` manuais | Todos recursos via smart pointers |

### 1.2. Design Patterns Implementados

| Pattern | Localização | Propósito |
|---------|-------------|-----------|
| **Template Method** | `IServiceBase.cpp` | start()/stop() com hooks onStart()/onStop() |
| **Strategy** | `ErrorHandler.hpp` | 5 estratégias de tratamento de erro |
| **Builder** | `DIBuilder.cpp` | Construção fluente do container DI |
| **Registry** | `ServiceRegistry`, `MetricRegistry`, `ComponentRegistry` | Registro centralizado de componentes |
| **Singleton** | `GracefulShutdownManager`, `ProcessLifecycleManager` | Instância única global |
| **Factory** | `ServiceFactory` | Criação de serviços com dependências |

---

## 2. INVENTÁRIO DO CORE (DADOS REAIS)

### 2.1. Arquivos de Implementação (`src/core/`)

| Arquivo | Linhas | Responsabilidade |
|---------|--------|------------------|
| `Application.cpp` | 539 | Bootstrap principal, main loop, lifecycle |
| `DIContainer.cpp` | 451 | Motor de DI com detecção de ciclos |
| `orchestration/ServiceOrchestrator.cpp` | 484 | Orquestração de serviços |
| `DIBuilder.cpp` | 254 | Builder fluente para container |
| `MetricRegistry.cpp` | 197 | Registry de métricas com weak_ptr |
| `lifecycle/GracefulShutdownManager.cpp` | 137 | Shutdown graceful com jthread |
| `IServiceBase.cpp` | 90 | Template Method para serviços |
| `lifecycle/ProcessLifecycleManager.cpp` | 87 | Hot-reload via execve |
| `ServiceRegistry.cpp` | 26 | Wrapper de registries especializados |
| `BasicServiceInitializer.cpp` | ~150 | Inicialização ordenada de serviços |
| `BasicConfigurationManager.cpp` | ~200 | Gestão de configuração |
| `BasicCommandLineParser.cpp` | ~180 | Parsing de CLI |
| `BasicLoggingManager.cpp` | ~100 | Configuração de logging |
| `ComponentRegistry.cpp` | ~80 | Registry de componentes |
| `ErrorHandler.cpp` | ~50 | Implementação de logging de erros |
| `ApplicationConfig.cpp` | ~60 | Estrutura de configuração |
| `BasicComponents.cpp` | ~40 | Componentes básicos |
| `orchestration/OrchestratorImpl.cpp` | ~150 | Implementação do orchestrator |
| `bootstrap/main.cpp` | ~30 | Entry point |
| **TOTAL** | **~3.300** | |

### 2.2. Headers (`include/corgrid/core/`)

| Header | Responsabilidade |
|--------|------------------|
| `DIContainer.hpp` | Container DI com Concepts C++20 |
| `DIBuilder.hpp` | Builder fluente |
| `ErrorHandler.hpp` | Strategy pattern para erros |
| `MetricRegistry.hpp` | Registry de métricas |
| `ServiceRegistry.hpp` | Registry de serviços |
| `Application.hpp` | Classe Application |
| `ApplicationConfig.hpp` | Struct de configuração |
| `ComponentRegistry.hpp` | Registry de componentes |
| `BasicComponents.hpp` | Componentes básicos |
| `BasicServiceInitializer.hpp` | Inicializador |
| `BasicConfigurationManager.hpp` | Gerenciador de config |
| `BasicLoggingManager.hpp` | Gerenciador de log |
| `BasicCommandLineParser.hpp` | Parser CLI |
| `ExceptionHandler.hpp` | Handler de exceções |
| `lifecycle/GracefulShutdownManager.hpp` | Shutdown manager |
| `lifecycle/ProcessLifecycleManager.hpp` | Hot-reload manager |
| `orchestration/OrchestratorImpl.hpp` | Orchestrator |
| `interfaces/ICommandLineParser.hpp` | Interface CLI parser |
| `interfaces/IConfigurationManager.hpp` | Interface config |
| `interfaces/ILoggingManager.hpp` | Interface logging |
| `interfaces/IServiceInitializer.hpp` | Interface inicializador |
| `interfaces/ISignalHandler.hpp` | Interface sinais |
| `interfaces/IDaemonManager.hpp` | Interface daemon |
| **TOTAL** | **23 headers** |

---

## 3. SISTEMA DE INJEÇÃO DE DEPENDÊNCIA (DIContainer)

### 3.1. Arquitetura

O DIContainer é o coração do sistema de DI, implementando:

```
IDIContainer (interface)
    └── DIContainer (implementação concreta - 451 linhas)
            └── DIBuilder (configuração fluente - 254 linhas)
```

**Arquivo:** `src/core/DIContainer.cpp` (451 linhas)
**Header:** `include/corgrid/core/DIContainer.hpp` (319 linhas)

### 3.2. Lifetimes Suportados

```cpp
// include/corgrid/interfaces/core/IDIContainer.hpp
enum class Lifetime {
    Singleton,   // Uma instância para toda a aplicação
    Transient,   // Nova instância a cada resolução
    Scoped       // Uma instância por escopo (request, thread)
};
```

### 3.3. Detecção de Dependências Circulares

O sistema usa uma **stack thread-local** para detectar ciclos em tempo de resolução:

```cpp
// src/core/DIContainer.cpp:16-38
namespace {
using ResolutionKey = std::pair<std::type_index, std::string>;
thread_local std::vector<ResolutionKey> g_resolution_stack;

class ResolutionGuard {
public:
    explicit ResolutionGuard(ResolutionKey key) : key_(std::move(key)) {
        if (std::ranges::find(g_resolution_stack, key_) != g_resolution_stack.end()) {
            throw std::runtime_error(std::format(
                "Dependência circular detectada em tempo de resolução: {}|{}",
                key_.first.name(), key_.second));
        }
        g_resolution_stack.push_back(key_);
    }
    ~ResolutionGuard() {
        if (!g_resolution_stack.empty() && g_resolution_stack.back() == key_) {
            g_resolution_stack.pop_back();
        }
    }
private:
    ResolutionKey key_;
};
} // namespace
```

### 3.4. Auto-Discovery

O DIContainer automaticamente descobre e registra:

```cpp
// include/corgrid/core/DIContainer.hpp:276-302
template<typename T>
void autoDiscoverService(const std::shared_ptr<T>& instance) {
    if constexpr (std::is_polymorphic_v<T>) {
        // Auto-discovery de IService
        if (auto service = std::dynamic_pointer_cast<IService>(instance)) {
            std::unique_lock lock(mutex_);
            orchestratableServices_.push_back(service);
        }
        // Auto-discovery de IMetricProvider
        if (auto provider = std::dynamic_pointer_cast<monitor::IMetricProvider>(instance)) {
            std::unique_lock lock(mutex_);
            metricProviders_.push_back(provider);
        }
    }
}
```

### 3.5. Concepts C++20 para Type Safety

```cpp
// include/corgrid/core/DIContainer.hpp:89-101
template<typename TService>
requires std::derived_from<TService, IService>
void registerService(const std::string& serviceName, std::shared_ptr<TService> service) {
    registerInstance<TService>(service, serviceName);
    if (auto iservice = std::dynamic_pointer_cast<IService>(service)) {
        std::unique_lock lock(mutex_);
        namedServices_[serviceName] = iservice;
    }
}
```

### 3.6. Estruturas de Dados (Performance)

```cpp
// include/corgrid/core/DIContainer.hpp:250-259
// Usa boost::container::flat_map para performance cache-friendly
boost::container::flat_map<std::pair<std::type_index, std::string>, ServiceRegistration> registrations_;
boost::container::flat_map<std::string, ScopeData> scopes_;

// Evita ciclos com weak_ptr
std::vector<std::weak_ptr<IService>> orchestratableServices_;
std::vector<std::weak_ptr<monitor::IMetricProvider>> metricProviders_;
boost::container::flat_map<std::string, std::weak_ptr<IService>> namedServices_;
```

### 3.7. Exemplo de Uso

```cpp
// Registro com dependências automáticas
container->registerWithDependencies<IAuthService, AuthService, IDatabaseService>();

// Registro de factory com dependências
container->registerFactoryWithDeps<IHttpServer, IRouter, IMiddlewareChain>(
    [](auto router, auto middleware) {
        return std::make_shared<BeastHttpServer>(router, middleware);
    },
    Lifetime::Singleton
);

// Resolução
auto authService = container->resolve<IAuthService>();
```

---

## 4. CICLO DE VIDA DE SERVIÇOS (IServiceBase)

### 4.1. Interface Base

```cpp
// include/corgrid/interfaces/core/IService.hpp
class IService {
public:
    virtual ~IService() = default;
    virtual void start() = 0;
    virtual void stop() = 0;
    virtual bool isRunning() const = 0;
};
```

### 4.2. Template Method Pattern

**Arquivo:** `src/core/IServiceBase.cpp` (90 linhas)

```cpp
// src/core/IServiceBase.cpp:10-25
void IServiceBase::start() {
    if (running_.load()) {
        logWarning("Tentativa de iniciar serviço já em execução");
        return;
    }
    try {
        logStart();
        onStart();  // Hook method - implementado pelo serviço específico
        setRunning(true);
        logInfo("Serviço iniciado com sucesso");
    } catch (const std::exception& e) {
        logError("Falha ao iniciar serviço: " + std::string(e.what()));
        throw;  // Re-throw para Application lidar
    }
}

void IServiceBase::stop() {
    if (!running_.load()) {
        logWarning("Tentativa de parar serviço já parado");
        return;
    }
    try {
        logInfo("Iniciando parada do serviço...");
        onStop();  // Hook method - implementado pelo serviço específico
        setRunning(false);
        logStop();
    } catch (const std::exception& e) {
        logError("Erro durante parada do serviço: " + std::string(e.what()));
        setRunning(false);  // Garante estado consistente mesmo com erro
    }
}
```

### 4.3. Estado Atômico Thread-Safe

```cpp
// src/core/IServiceBase.cpp:45-58
bool IServiceBase::isRunning() const {
    // Verificação thread-safe do estado base
    if (!running_.load()) {
        return false;
    }
    // Delegação para verificação específica do serviço
    try {
        return checkRunning();  // Virtual - pode ter lógica adicional
    } catch (const std::exception& e) {
        logError("Erro na verificação de estado: " + std::string(e.what()));
        return false;  // Em caso de erro, assume parado
    }
}

void IServiceBase::setRunning(bool running) {
    running_.store(running);  // std::atomic<bool>
}
```

### 4.4. Logging Padronizado Integrado

```cpp
// src/core/IServiceBase.cpp:60-84
void IServiceBase::logStart() const {
    corgrid::utils::Logger::instance().info(
        std::format("{}: Iniciando serviço...", getServiceName()));
}

void IServiceBase::logStop() const {
    corgrid::utils::Logger::instance().info(
        std::format("{}: Serviço parado com sucesso", getServiceName()));
}

void IServiceBase::logError(const std::string& message) const {
    corgrid::utils::Logger::instance().error(
        std::format("{}: {}", getServiceName(), message));
}

void IServiceBase::logWarning(const std::string& message) const {
    corgrid::utils::Logger::instance().warn(
        std::format("{}: {}", getServiceName(), message));
}
```

---

## 5. O PROCESSO DE BOOTSTRAP (Application.cpp)

**Arquivo:** `src/core/Application.cpp` (539 linhas)

### 5.1. Diagrama de Fases

```
┌─────────────────────────────────────────────────────────────────────┐
│                        BOOTSTRAP SEQUENCE                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  FASE 1: CLI Parsing                                                │
│  ├── parse(argc, argv)                                              │
│  ├── Handle --help, --version                                       │
│  └── Merge CLI config → ApplicationConfig                           │
│                                                                     │
│  FASE 2: Configuração                                               │
│  ├── loadConfiguration() - Boost.PropertyTree JSON                  │
│  └── setupLogging() - Spdlog async                                  │
│                                                                     │
│  FASE 3: Validação                                                  │
│  ├── validateConfiguration()                                        │
│  └── checkSingleInstance() - PID file                               │
│                                                                     │
│  FASE 4: Daemonização (opcional)                                    │
│  ├── daemonize() - double fork                                      │
│  └── createPidFile()                                                │
│                                                                     │
│  FASE 5: Inicialização de Serviços                                  │
│  ├── GracefulShutdownManager.initializeShutdownHandlers()           │
│  ├── initializeServices() → ServiceFactory                          │
│  └── orchestrator_->start()                                         │
│                                                                     │
│  FASE 6: Main Loop                                                  │
│  ├── Health check a cada 30s                                        │
│  ├── Hot-reload via reload_requested_ flag                          │
│  └── Circuit breaker (MAX_CONSECUTIVE_ERRORS)                       │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 5.2. Construtor - Inicialização de Registries

```cpp
// src/core/Application.cpp:26-33
Application::Application(const ApplicationConfig& config)
    : config_(config) {
    metric_registry_ = std::make_shared<MetricRegistry>();
    component_registry_ = std::make_shared<ComponentRegistry>();
    service_registry_ = std::make_shared<ServiceRegistry>(
        metric_registry_, component_registry_);
    factory_ = std::make_shared<services::ServiceFactory>(service_registry_);
    orchestrator_ = std::make_shared<corgrid::OrchestratorImpl>(service_registry_);
}
```

### 5.3. Main Loop com Circuit Breaker

```cpp
// src/core/Application.cpp:159-231
while (running_ && !shutdown_manager.isShutdownRequested()) {
    try {
        // Hot-reload handling
        if (reload_requested_.exchange(false)) {
            std::lock_guard<std::mutex> lock(config_mutex_);
            auto config_service = service_registry_->get<IConfigService>();
            if (config_service) {
                config_service->reload();
                consecutive_errors_ = 0;
            }
        }

        // Health check periódico (30 segundos)
        static auto last_health_check = std::chrono::steady_clock::now();
        auto now = std::chrono::steady_clock::now();
        if (now - last_health_check > std::chrono::seconds(30)) {
            for (const auto& service : service_registry_->getOrchestratableServices()) {
                if (service && !service->isRunning()) {
                    corgrid::utils::Logger::instance().warn(
                        "Serviço não está rodando, tentando reiniciar");
                    service->start();
                }
            }
            last_health_check = now;
        }

        // Circuit breaker
        if (consecutive_errors_ >= MAX_CONSECUTIVE_ERRORS) {
            corgrid::utils::Logger::instance().critical(
                std::format("Muitos erros consecutivos ({}), shutdown de emergência",
                    consecutive_errors_.load()));
            shutdown();
            break;
        }

        // Rate limiter para erros
        if (consecutive_errors_ > 0) {
            auto error_delay = std::chrono::milliseconds(100 * consecutive_errors_);
            std::this_thread::sleep_for(error_delay);
        } else {
            std::this_thread::sleep_for(std::chrono::seconds(1));
        }
    } catch (const std::exception& e) {
        corgrid::utils::Logger::instance().error(
            std::format("Erro no main loop: {}", e.what()));
        consecutive_errors_++;
    }
}
```

### 5.4. Daemonização (Double Fork)

```cpp
// src/core/Application.cpp:351-393
void Application::daemonize() {
    corgrid::utils::Logger::instance().info("Entrando em modo daemon...");

    // First fork
    pid_t pid = fork();
    if (pid < 0) {
        CGR_THROW(corgrid::exceptions::InternalFailureException, "Falha no primeiro fork");
    }
    if (pid > 0) exit(0);  // Parent exits

    // Create new session
    if (setsid() < 0) {
        CGR_THROW(corgrid::exceptions::InternalFailureException, "Falha ao criar nova sessão");
    }

    // Second fork
    pid = fork();
    if (pid < 0) {
        CGR_THROW(corgrid::exceptions::InternalFailureException, "Falha no segundo fork");
    }
    if (pid > 0) exit(0);  // Parent exits

    // Change to root directory
    if (chdir("/") < 0) {
        CGR_THROW(corgrid::exceptions::InternalFailureException, "Falha ao mudar diretório");
    }

    // Redirect stdio to /dev/null
    close(STDIN_FILENO);
    close(STDOUT_FILENO);
    close(STDERR_FILENO);
    open("/dev/null", O_RDONLY);  // stdin
    open("/dev/null", O_WRONLY);  // stdout
    open("/dev/null", O_WRONLY);  // stderr

    corgrid::utils::Logger::instance().info("Modo daemon ativado");
}
```

---

## 6. ORQUESTRAÇÃO DE SERVIÇOS (ServiceOrchestrator)

**Arquivo:** `src/core/orchestration/ServiceOrchestrator.cpp` (484 linhas)

### 6.1. OrchestrationResult

```cpp
// Estrutura de resultado da orquestração
struct OrchestrationResult {
    bool success;
    std::vector<std::string> initialized_services;
    size_t failed_services;
    std::chrono::milliseconds initialization_time;
    std::string error_message;
};
```

### 6.2. Fluxo de Inicialização

```cpp
// src/core/orchestration/ServiceOrchestrator.cpp:30-81
ServiceOrchestrator::OrchestrationResult
ServiceOrchestrator::initializeServices(const ApplicationConfig& config) {
    OrchestrationResult result;
    auto start_time = std::chrono::high_resolution_clock::now();

    try {
        // 1. Criar orchestrator core
        if (!createCoreOrchestrator()) {
            result.error_message = "Falha ao criar orchestrator core";
            return result;
        }

        // 2. Inicializar serviços core
        if (!initializeCoreServices(config)) {
            result.error_message = "Falha ao inicializar serviços core";
            return result;
        }

        // 3. Inicializar serviços de plugin
        if (!initializePluginServices(config)) {
            result.error_message = "Falha ao inicializar serviços de plugin";
            return result;
        }

        // 4. Validar dependências
        if (!validateServiceDependencies()) {
            result.error_message = "Falha na validação de dependências";
            return result;
        }

        auto end_time = std::chrono::high_resolution_clock::now();
        result.initialization_time =
            std::chrono::duration_cast<std::chrono::milliseconds>(end_time - start_time);
        result.success = true;
        result.initialized_services = initialized_services_;

        corgrid::utils::Logger::instance().info(
            std::format("ServiceOrchestrator: {} serviços inicializados em {}ms",
                initialized_services_.size(), result.initialization_time.count()));
        return result;
    } catch (const std::exception& e) {
        result.error_message = std::string("Erro: ") + e.what();
        return result;
    }
}
```

### 6.3. Health Monitoring

```cpp
// src/core/orchestration/ServiceOrchestrator.cpp:201-239
nlohmann::json ServiceOrchestrator::getServiceHealth() const {
    nlohmann::json health;
    auto services = registry_->getAllServices();

    health["total_services"] = services.size();
    health["timestamp"] = std::chrono::duration_cast<std::chrono::seconds>(
        std::chrono::system_clock::now().time_since_epoch()).count();

    nlohmann::json services_array = nlohmann::json::array();
    for (const auto& [name, service] : services) {
        nlohmann::json service_info;
        service_info["name"] = name;
        service_info["running"] = service ? service->isRunning() : false;
        service_info["healthy"] = checkServiceHealth(name);
        services_array.push_back(service_info);
    }

    health["services"] = services_array;
    health["summary"] = {
        {"running", getRunningServices().size()},
        {"failed", getFailedServices().size()}
    };

    return health;
}
```

---

## 7. SHUTDOWN GRACEFUL (GracefulShutdownManager)

**Arquivo:** `src/core/lifecycle/GracefulShutdownManager.cpp` (137 linhas)

### 7.1. Sinais Tratados

| Sinal | Ação |
|-------|------|
| `SIGINT` | Shutdown graceful |
| `SIGTERM` | Shutdown graceful |
| `SIGQUIT` | Shutdown graceful |
| `SIGHUP` | Reload de configuração |
| `SIGPIPE` | Ignorado (`SIG_IGN`) |

### 7.2. Uso de std::jthread (C++20)

```cpp
// src/core/lifecycle/GracefulShutdownManager.cpp:30-43
void GracefulShutdownManager::requestShutdown() {
    if (shutdown_requested_.exchange(true)) {
        return;  // Já foi solicitado
    }

    corgrid::utils::Logger::instance().info("GracefulShutdownManager: Shutdown solicitado");

    // Usa std::jthread com stop_token para cancelamento cooperativo
    if (!shutdown_thread_.joinable()) {
        shutdown_thread_ = std::jthread([this](std::stop_token stop_token) {
            performGracefulShutdown(stop_token);
        });
    }
}
```

### 7.3. Timeout de Segurança

```cpp
// src/core/lifecycle/GracefulShutdownManager.cpp:91-116
void GracefulShutdownManager::performGracefulShutdown(std::stop_token stop_token) {
    if (stop_token.stop_requested()) {
        return;
    }

    corgrid::utils::Logger::instance().info(
        "GracefulShutdownManager: Executando shutdown graceful...");

    try {
        // Aguarda flush de logs
        std::this_thread::sleep_for(std::chrono::milliseconds(100));

        if (application_shutdown_) {
            application_shutdown_();
        }

        corgrid::utils::Logger::instance().info(
            "GracefulShutdownManager: Shutdown graceful concluído");
    } catch (const std::exception& e) {
        corgrid::utils::Logger::instance().error(
            std::format("GracefulShutdownManager: Erro durante shutdown: {}", e.what()));
    }

    // TIMEOUT: Forçar exit após 5 segundos
    std::this_thread::sleep_for(std::chrono::seconds(5));
    corgrid::utils::Logger::instance().warn(
        "GracefulShutdownManager: Forçando exit após timeout");
    std::_Exit(0);
}
```

---

## 8. HOT-RELOAD (ProcessLifecycleManager)

**Arquivo:** `src/core/lifecycle/ProcessLifecycleManager.cpp` (87 linhas)

### 8.1. Mecanismo

O hot-reload é implementado via `execve()`, substituindo o processo atual por uma nova instância do mesmo binário.

```cpp
// src/core/lifecycle/ProcessLifecycleManager.cpp:47-85
void ProcessLifecycleManager::reexec() {
    std::lock_guard<std::mutex> lock(mutex_);

    if (args_.empty()) {
        corgrid::utils::Logger::instance().critical(
            "ProcessLifecycleManager: Impossível reiniciar - argumentos não capturados");
        return;
    }

    // Preparar argv para execv
    std::vector<char*> argv_c;
    argv_c.reserve(args_.size() + 1);
    for (const auto& arg : args_) {
        argv_c.push_back(const_cast<char*>(arg.c_str()));
    }
    argv_c.push_back(nullptr);

    // Definir variável de ambiente para indicar hot reload
    if (setenv("CORGRID_HOT_RELOAD", "1", 1) != 0) {
        corgrid::utils::Logger::instance().error(
            "ProcessLifecycleManager: Falha ao definir env CORGRID_HOT_RELOAD");
    }

    // TODO: Preservação de sockets (FDs) seria implementada aqui
    // Ex: fcntl(fd, F_SETFD, 0) para remover CLOEXEC de sockets de listen

    corgrid::utils::Logger::instance().info(
        std::format("ProcessLifecycleManager: Executando novo binário: {}", program_path_));

    // Substituir processo atual
    if (execv(program_path_.c_str(), argv_c.data()) == -1) {
        int err = errno;
        corgrid::utils::Logger::instance().critical(
            std::format("ProcessLifecycleManager: Falha fatal no execv: {} ({})",
                std::strerror(err), err));
        restart_requested_ = false;
        throw std::runtime_error("Falha fatal no hot reload");
    }
}
```

### 8.2. Detecção de Hot-Reload

```cpp
// src/core/lifecycle/ProcessLifecycleManager.cpp:34-37
bool ProcessLifecycleManager::isHotReload() const {
    const char* val = std::getenv("CORGRID_HOT_RELOAD");
    return val != nullptr && std::string(val) == "1";
}
```

---

## 9. SISTEMA DE MÉTRICAS (MetricRegistry)

**Arquivo:** `src/core/MetricRegistry.cpp` (197 linhas)

### 9.1. Gerenciamento com weak_ptr

Para evitar ciclos de referência, o MetricRegistry usa `weak_ptr`:

```cpp
// src/core/MetricRegistry.cpp:13-36
void MetricRegistry::registerProvider(std::shared_ptr<monitor::IMetricProvider> provider) {
    if (!provider) return;

    ErrorHandler::handleVoidOperation([this, provider]() {
        std::unique_lock lock(mutex_);
        cleanupExpiredProviders();

        // Verificar se já existe
        auto existing = std::ranges::find_if(providers_,
            [provider](const std::weak_ptr<monitor::IMetricProvider>& weak) {
                if (auto sp = weak.lock()) {
                    return sp.get() == provider.get();
                }
                return false;
            });

        if (existing == providers_.end()) {
            providers_.push_back(provider);  // weak_ptr armazena
            corgrid::utils::Logger::instance().debug(
                std::format("MetricRegistry: Provedor '{}' registrado", provider->name()));
        }
    }, ErrorHandler::ErrorConfig{...});
}
```

### 9.2. Coleta com Expected<T,E>

```cpp
// src/core/MetricRegistry.cpp:60-107
utils::Expected<nlohmann::json, std::string> MetricRegistry::collectAllMetrics() {
    try {
        nlohmann::json result;
        result["timestamp"] = std::chrono::system_clock::now().time_since_epoch().count();
        result["providers"] = nlohmann::json::object();

        auto providers = getValidProviders();
        for (const auto& provider : providers) {
            const std::string pname = provider ? std::string(provider->name()) : "";
            try {
                auto collected = provider->collect();
                if (collected.has_value()) {
                    result["providers"][pname] = collected.value();
                } else {
                    result["providers"][pname] = nlohmann::json{{"error", collected.error()}};
                }
            } catch (const std::exception& e) {
                result["providers"][pname] = nlohmann::json{{"error", e.what()}};
            }
        }

        result["total_providers"] = providers.size();
        return utils::Expected<nlohmann::json, std::string>(result);
    } catch (const std::exception& e) {
        return utils::Expected<nlohmann::json, std::string>::unexpected(e.what());
    }
}
```

### 9.3. Cleanup Automático

```cpp
// src/core/MetricRegistry.cpp:179-190
void MetricRegistry::cleanupExpiredProviders() {
    auto it = std::remove_if(providers_.begin(), providers_.end(),
        [](const std::weak_ptr<monitor::IMetricProvider>& weak) {
            return weak.expired();
        });

    if (it != providers_.end()) {
        size_t removed = std::distance(it, providers_.end());
        providers_.erase(it, providers_.end());
        corgrid::utils::Logger::instance().debug(
            std::format("MetricRegistry: {} provedores expirados removidos", removed));
    }
}
```

---

## 10. TRATAMENTO DE ERROS (ErrorHandler)

**Arquivo:** `include/corgrid/core/ErrorHandler.hpp` (399 linhas)

### 10.1. Strategy Pattern

```cpp
// include/corgrid/core/ErrorHandler.hpp:29-35
enum class Strategy {
    LogAndContinue,     // Log e continua execução
    LogAndThrow,        // Log e re-throw exception
    LogAndReturn,       // Log e retorna valor default
    Silent,             // Apenas log silenciosamente
    MetricsOnly         // Apenas incrementa métricas
};
```

### 10.2. Hierarquia de Exceções Tratadas

| Exceção | Nível de Log | Métricas |
|---------|--------------|----------|
| `ValidationException` | WARN | `validation_error` |
| `AccessDeniedException` | ERROR | `security_error` |
| `NotFoundException` | WARN | `not_found_error` |
| `InternalFailureException` | ERROR | `internal_error` |
| `std::exception` | ERROR | `generic_error` |
| Unknown (`...`) | CRITICAL | `unknown_error` |

### 10.3. Métodos Monádicos (C++23 Style)

```cpp
// include/corgrid/core/ErrorHandler.hpp:371-389
template<typename T, typename TOperation>
static utils::Expected<T, std::string> tryMonadic(
    TOperation&& operation,
    std::string_view context = ""
) {
    return tryOperation<T>(
        std::forward<TOperation>(operation),
        ErrorConfig{Strategy::LogAndReturn, std::string(context), true, false, std::nullopt}
    );
}

template<typename TOperation>
static utils::Expected<std::monostate, std::string> tryMonadicVoid(
    TOperation&& operation,
    std::string_view context = ""
) {
    return tryVoidOperation(
        std::forward<TOperation>(operation),
        ErrorConfig{Strategy::LogAndReturn, std::string(context), true, false, std::nullopt}
    );
}
```

### 10.4. Exemplo de Uso

```cpp
// Uso com Strategy
auto result = ErrorHandler::handleOperation<std::string>(
    []() { return loadConfig(); },
    ErrorHandler::ErrorConfig{
        .strategy = ErrorHandler::Strategy::LogAndReturn,
        .context = "ConfigLoader",
        .enable_metrics = true
    }
);

// Uso monádico
auto result = ErrorHandler::tryMonadic<User>(
    []() { return findUser(userId); },
    "UserRepository.find"
);

if (result.has_value()) {
    // sucesso
} else {
    // result.error() contém mensagem
}
```

---

## 11. SISTEMA DE CONFIGURAÇÃO

### 11.1. Hierarquia de Precedência

```
CLI Flags (maior prioridade)
    ↓
Environment Variables (CORGRID_*)
    ↓
Config File (conf.json)
    ↓
Hardcoded Defaults (menor prioridade)
```

### 11.2. Carregamento de Configuração

```cpp
// src/core/Application.cpp:288-318
void Application::loadConfiguration() {
    try {
        if (std::filesystem::exists(config_.config_file)) {
            boost::property_tree::ptree pt;
            boost::property_tree::read_json(config_.config_file, pt);

            // Load configuration from JSON usando Boost.PropertyTree
            config_.server_port = pt.get<int>("server.port", config_.server_port);
            config_.server_host = pt.get<std::string>("server.host", config_.server_host);
            config_.log_level = pt.get<std::string>("system.log_level", config_.log_level);

            // Load enabled plugins
            config_.enabled_plugins.clear();
            try {
                for (const auto& plugin : pt.get_child("plugins.enabled")) {
                    config_.enabled_plugins.push_back(
                        plugin.second.get_value<std::string>());
                }
            } catch (const boost::property_tree::ptree_bad_path&) {
                // plugins.enabled not found, use defaults
            }

            corgrid::utils::Logger::instance().info(
                std::format("Configuração carregada de: {}", config_.config_file));
        } else {
            corgrid::utils::Logger::instance().warn(
                std::format("Arquivo não encontrado: {}, usando padrões", config_.config_file));
        }
    } catch (const std::exception& e) {
        corgrid::utils::Logger::instance().error(
            std::format("Erro ao carregar configuração: {}", e.what()));
        throw;
    }
}
```

### 11.3. Hot-Reload de Configuração

O reload é disparado via `SIGHUP` e processado no main loop:

```cpp
// src/core/Application.cpp:162-176
if (reload_requested_.exchange(false)) {
    try {
        std::lock_guard<std::mutex> lock(config_mutex_);
        auto config_service = service_registry_->get<IConfigService>();
        if (config_service) {
            config_service->reload();
            consecutive_errors_ = 0;
        }
    } catch (const std::exception& e) {
        corgrid::utils::Logger::instance().error(
            std::format("Falha ao recarregar configuração: {}", e.what()));
        consecutive_errors_++;
    }
}
```

---

## 12. SISTEMA DE LOGGING

### 12.1. Configuração

```cpp
// src/core/Application.cpp:269-286
void Application::setupLogging() {
    auto toLevel = [](const std::string& lvl) {
        if (lvl == "trace") return corgrid::utils::LogLevel::trace;
        if (lvl == "debug") return corgrid::utils::LogLevel::debug;
        if (lvl == "warn") return corgrid::utils::LogLevel::warn;
        if (lvl == "error" || lvl == "err") return corgrid::utils::LogLevel::error;
        if (lvl == "critical") return corgrid::utils::LogLevel::critical;
        return corgrid::utils::LogLevel::info;
    };

    const auto global_level = toLevel(config_.log_level);
    corgrid::utils::Logger::instance().configure(
        config_.log_file.empty()
            ? std::string(corgrid::constants::paths::LOG_FILE)
            : config_.log_file,
        global_level,
        config_.log_file_max_size,
        config_.log_file_max_files
    );
}
```

### 12.2. Níveis de Log

| Nível | Uso |
|-------|-----|
| `trace` | Debug detalhado de fluxo |
| `debug` | Informações de desenvolvimento |
| `info` | Operações normais |
| `warn` | Situações não ideais |
| `error` | Erros recuperáveis |
| `critical` | Erros fatais |

---

## 13. MODELO DE CONCORRÊNCIA E THREADING

### 13.1. Threads do Sistema

| Thread | Responsabilidade | Implementação |
|--------|------------------|---------------|
| **Main Thread** | Main loop, health checks | `Application::run()` |
| **Shutdown Thread** | Graceful shutdown | `std::jthread` em `GracefulShutdownManager` |
| **Reload Thread** | Hot-reload de config | `std::jthread` em `GracefulShutdownManager` |
| **Worker Pool** | Tarefas CPU-intensivas | `ThreadPool` (Boost.Asio) |
| **IO Context** | Operações de I/O | `boost::asio::io_context` |

### 13.2. Primitivas de Sincronização

```cpp
// DIContainer - reader-writer lock para performance
mutable std::shared_mutex mutex_;  // Muitos leitores, poucos escritores

// Application - mutex simples para configuração
mutable std::mutex config_mutex_;

// Estado atômico
std::atomic<bool> running_{false};
std::atomic<uint32_t> consecutive_errors_{0};
std::atomic<bool> reload_requested_{false};
```

### 13.3. Padrão de Acesso

```cpp
// Leitura concorrente (múltiplos threads)
{
    std::shared_lock lock(mutex_);
    // ... leitura segura ...
}

// Escrita exclusiva
{
    std::unique_lock lock(mutex_);
    // ... modificação ...
}
```

---

## 14. INTEGRAÇÃO COM PLUGINS

### 14.1. Carregamento de Plugins

```cpp
// src/core/orchestration/ServiceOrchestrator.cpp:347-384
bool ServiceOrchestrator::initializePluginServices(const ApplicationConfig& config) {
    try {
        auto plugin_manager = registry_->get<PluginManager>();
        if (!plugin_manager) {
            corgrid::utils::Logger::instance().debug(
                "ServiceOrchestrator: PluginManager não encontrado - sem plugins");
            return true;  // Não é erro, pode não ter plugins
        }

        // Carregar plugins habilitados
        for (const auto& plugin_name : config.enabled_plugins) {
            try {
                plugin_manager->loadPluginFile(plugin_name);
                initialized_services_.push_back(plugin_name);
                corgrid::utils::Logger::instance().debug(
                    std::format("ServiceOrchestrator: Plugin {} carregado", plugin_name));
            } catch (const std::exception&) {
                failed_services_.push_back(plugin_name);
                corgrid::utils::Logger::instance().warn(
                    std::format("ServiceOrchestrator: Falha ao carregar plugin {}", plugin_name));
            }
        }

        corgrid::utils::Logger::instance().info(
            std::format("ServiceOrchestrator: {} plugins carregados",
                config.enabled_plugins.size() - failed_services_.size()));
        return true;
    } catch (const std::exception& e) {
        corgrid::utils::Logger::instance().error(
            std::format("ServiceOrchestrator: Erro ao inicializar plugins: {}", e.what()));
        return false;
    }
}
```

### 14.2. Isolamento via SDK

Plugins acessam o core via `ISandboxContext`, nunca diretamente:

```
Plugin
  └── ISandboxContext (SDK)
        ├── ctx.log()         → Logger
        ├── ctx.getConfig()   → ConfigService
        ├── ctx.httpGet()     → HttpClient
        └── ctx.mqtt()        → MqttClient
```

---

## 15. C++23 FEATURES EM USO

### 15.1. Tabela de Features

| Feature | Onde é Usado | Exemplo |
|---------|--------------|---------|
| **std::format** | Todo o core | `std::format("Error: {}", msg)` |
| **std::jthread** | GracefulShutdownManager | `std::jthread([](std::stop_token st){...})` |
| **Concepts** | DIContainer | `requires std::derived_from<T, IService>` |
| **std::ranges** | DIContainer, MetricRegistry | `std::ranges::find_if(...)` |
| **boost::container::flat_map** | DIContainer | Cache-friendly map |
| **std::source_location** | Logging (opcional) | `std::source_location::current()` |

### 15.2. Expected<T,E> Pattern

```cpp
// utils/Expected.hpp - implementação própria
template<typename T, typename E = std::string>
class Expected {
    std::variant<T, E> data_;
public:
    bool has_value() const;
    T& value();
    const E& error() const;
    static Expected<T,E> unexpected(E error);
};

// Uso real
utils::Expected<nlohmann::json, std::string> collectAllMetrics();
```

---

## 16. ROADMAP DE ENGENHARIA (C++23)

### 16.1. Já Implementado

| Feature | Status | Arquivo |
|---------|--------|---------|
| std::format | Completo | Todo o core |
| std::jthread | Completo | GracefulShutdownManager |
| Concepts | Completo | DIContainer |
| std::ranges | Completo | DIContainer, MetricRegistry |
| Expected<T,E> polyfill | Completo | utils/Expected.hpp |

### 16.2. Próximas Iterações

#### 16.2.1. C++20 Modules
- **Alvo:** Converter `ServiceRegistry` e `IService` para módulo `Core.Registry`
- **Benefício:** Redução de 40-60% no tempo de compilação incremental
- **Prioridade:** Alta

#### 16.2.2. std::expected Nativo (C++23)
- **Alvo:** Substituir polyfill `utils::Expected` por `std::expected`
- **Benefício:** Conformidade com padrão, otimizações do compilador
- **Prioridade:** Média

#### 16.2.3. Compile-Time DI
- **Alvo:** Validação de dependências em tempo de compilação
- **Conceito:** `template<Service T> class Registry`
- **Benefício:** Erro de compilação se dependência faltante
- **Prioridade:** Alta

#### 16.2.4. io_uring (Linux 5.10+)
- **Alvo:** Substituir `epoll` por `io_uring` para I/O de disco e rede
- **Benefício:** Zero syscall overhead para operações intensivas
- **Suporte Boost:** `BOOST_ASIO_HAS_IO_URING`
- **Prioridade:** Média

#### 16.2.5. Coroutines (co_await)
- **Atual:** `async_read(socket, buffer, [](ec, len){ ... })`
- **Futuro:** `co_await async_read(socket, buffer);`
- **Benefício:** Eliminação de callback hell
- **Prioridade:** Alta

---

## 17. GUIA DE DESENVOLVIMENTO

### 17.1. Adicionando um Novo Serviço Core

1. **Definir Interface**
   ```cpp
   // include/corgrid/interfaces/services/IMyService.hpp
   class IMyService {
   public:
       virtual ~IMyService() = default;
       virtual void doSomething() = 0;
   };
   ```

2. **Implementar Serviço**
   ```cpp
   // src/services/MyService.cpp
   class MyService : public IServiceBase, public IMyService {
   protected:
       void onStart() override { /* inicialização */ }
       void onStop() override { /* cleanup */ }
   public:
       void doSomething() override { /* lógica */ }
       std::string_view getServiceName() const override { return "MyService"; }
   };
   ```

3. **Registrar no DIBuilder**
   ```cpp
   // src/core/DIBuilder.cpp
   void DIBuilder::registerMyServices() {
       container_->registerSingleton<IMyService, MyService>();
   }
   ```

4. **Adicionar ao ServiceOrchestrator (se necessário)**

### 17.2. Checklist de Código

- [ ] Arquivo não excede 500 linhas
- [ ] Sem `new`/`delete` manuais (RAII)
- [ ] Usa `std::expected` ou polyfill para erros de runtime
- [ ] Usa Logger estruturado, nunca `std::cout`
- [ ] Thread-safety com mutexes apropriados
- [ ] Constantes centralizadas (sem magic numbers)

---

## 18. DEBUGGING E TROUBLESHOOTING

### 18.1. Boot Failures

**Sintoma:** Core sai imediatamente sem logs

1. Verificar permissões: `ls -la /var/log/corgrid/`
2. Rodar em foreground: `./corgrid_edge --log-level trace`
3. Verificar porta em uso: `netstat -tulpn | grep 8080`

### 18.2. Deadlocks

**Sintoma:** Boot trava (hang)

1. Provável dependência circular no `start()`
2. Debug com GDB:
   ```bash
   gdb -p $(pidof corgrid_edge)
   (gdb) thread apply all bt
   # Procurar por std::mutex::lock ou std::future::wait
   ```

### 18.3. Memory Leaks

```bash
valgrind --leak-check=full --show-leak-kinds=all ./corgrid_edge
# Foque em "definitely lost" no namespace corgrid::
# Falsos positivos são comuns em OpenSSL e drivers de DB
```

### 18.4. Circuit Breaker Ativado

**Sintoma:** Log `"Muitos erros consecutivos, shutdown de emergência"`

1. Verificar `MAX_CONSECUTIVE_ERRORS` (default: 10)
2. Analisar logs anteriores para identificar causa raiz
3. Verificar conectividade de banco de dados
4. Verificar configuração de serviços

---

**Fim do Documento.**

Este manual é a **Fonte da Verdade** para o core do CorGrid OS. Qualquer alteração em `src/core/` deve ser refletida aqui.

**Última atualização:** 19/12/2025
**Baseado em análise de:** 19 arquivos .cpp, 23 arquivos .hpp (~7.600 linhas)
**Mantenedor:** Core Architecture Team
