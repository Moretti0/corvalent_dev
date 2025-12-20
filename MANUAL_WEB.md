# MANUAL DE ARQUITETURA WEB: COROUTINES, MIDDLEWARE & ENTERPRISE TRANSPORT (THE SOURCE OF TRUTH)

**Document ID:** ENG-WEB-TRUTH-V6
**Data:** 20/12/2025
**Escopo:** `src/web/` + `include/corgrid/web/` + `include/corgrid/interfaces/web/`
**Autoridade:** Core Architecture Team
**Stack:** Boost.Asio (Coroutines C++20) / Boost.Beast / Zstandard / C++23
**Modelo:** Async Coroutine-based Reactor (Single-Threaded Event Loop)
**Status:** Source of Truth - Baseado em Análise Real do Código

---

## ÍNDICE

1.  [Filosofia: The Coroutine Revolution](#1-filosofia-the-coroutine-revolution)
2.  [Inventário do Módulo Web (Dados Reais)](#2-inventário-do-módulo-web-dados-reais)
3.  [Transport Engine - O Coração do I/O](#3-transport-engine---o-coração-do-io)
4.  [HttpCoroutineSession - O Worker Assíncrono](#4-httpcoroutinesession---o-worker-assíncrono)
5.  [Sistema de Middleware (Arquitetura Completa)](#5-sistema-de-middleware-arquitetura-completa)
6.  [Middlewares Builtin](#6-middlewares-builtin)
7.  [Middleware Manager (Factory Pattern)](#7-middleware-manager-factory-pattern)
8.  [Middleware Statistics (Observer Pattern)](#8-middleware-statistics-observer-pattern)
9.  [Middleware Chain Factory (SOLID Components)](#9-middleware-chain-factory-solid-components)
10. [Sistema de Roteamento (RefactoredRouteRegistry)](#10-sistema-de-roteamento-refactoredrouteregistry)
11. [Realtime: SSE & WebSocket Enterprise Hubs](#11-realtime-sse--websocket-enterprise-hubs)
12. [Gerenciamento de Memória (Pools)](#12-gerenciamento-de-memória-pools)
13. [Interfaces Web (ISP - Interface Segregation)](#13-interfaces-web-isp---interface-segregation)
14. [C++23 Features em Uso](#14-c23-features-em-uso)
15. [Roadmap de Engenharia (C++23/26)](#15-roadmap-de-engenharia-c2326)
16. [Debugging e Troubleshooting](#16-debugging-e-troubleshooting)

---

## 1. FILOSOFIA: THE COROUTINE REVOLUTION

A arquitetura Web do CorGrid abandonou o modelo de *Callback Hell* em favor de **C++20/23 Coroutines**. Isso permite código assíncrono de alta performance que parece síncrono e linear.

### 1.1. Callback vs. Coroutine

| Paradigma | Código | Complexidade |
|-----------|--------|--------------|
| **Legacy (Callbacks)** | `async_read(socket, buf, [](ec, n){ ... });` | Alta - callback hell |
| **Enterprise (Coroutines)** | `auto n = co_await async_read(socket, buf);` | Baixa - código linear |

### 1.2. Padrão de Erro: as_tuple (No-Throw)

O código real usa `net::as_tuple` para evitar try/catch ao redor de cada `co_await`:

```cpp
// src/web/http/coroutine/HttpCoroutineSession.cpp:52-54
auto [ec, bytes] = co_await std::visit([this](auto& stream) -> net::awaitable<std::tuple<beast::error_code, size_t>> {
    return http::async_read(stream, buffer_, *parser_, net::as_tuple(net::use_awaitable));
}, socket_);
```

### 1.3. Princípios Arquiteturais

| Princípio | Implementação |
|-----------|---------------|
| **Single Responsibility** | Cada arquivo < 500 linhas |
| **Interface Segregation** | 72+ interfaces web segregadas |
| **Dependency Inversion** | Factories injetam dependências |
| **Composition over Inheritance** | RefactoredRouteRegistry usa composição |

---

## 2. INVENTÁRIO DO MÓDULO WEB (DADOS REAIS)

### 2.1. Arquivos de Implementação (`src/web/`)

| Diretório | Arquivos | Linhas | Responsabilidade |
|-----------|----------|--------|------------------|
| `http/coroutine/` | 1 | 413 | HttpCoroutineSession (worker assíncrono) |
| `http/webserver/` | 4 | ~400 | Factories, Config, Static Files |
| `http/` | 2 | ~350 | MemoryPool, EnterpriseSSEHub |
| `middleware/` | 5 | ~500 | CORS, Logging, Zstd, MiddlewareChain |
| `middleware/core/` | 5 | ~800 | Pipeline, Registry, Statistics, Factory |
| `routing/registry/` | 6 | ~800 | RouteRegistry, Discovery, Validator, Metrics |
| `routing/handlers/` | 3 | ~300 | VersionPrefix, Manual, AutoDiscovery |
| `transport/` | 1 | 360 | TransportEngine (Accept Loops) |
| `transport/http/` | 1 | ~150 | HttpCoroutineSessionFactory |
| `transport/ws/` | 1 | ~100 | BeastWebSocketAdapterFactory |
| `websocket/` | 1 | 212 | EnterpriseWebSocketHub |
| **TOTAL** | **34** | **~4.124** | |

### 2.2. Headers de Interface (`include/corgrid/interfaces/web/`)

| Categoria | Quantidade | Exemplos |
|-----------|------------|----------|
| **Core** | 12 | IHttpServer, IHttpRouter, IRouteProvider |
| **HTTP** | 14 | IHttpSession, IConnectionLifecycleManager, IMemoryPool |
| **Middleware** | 18 | IMiddlewareExecutor, IMiddlewarePipeline, IMiddlewareRegistry |
| **Routing** | 10 | IRouteDiscovery, IRouteValidator, IRouteMetrics |
| **WebServer** | 8 | IServerConfiguration, IMiddlewareManager, IStaticFileHandler |
| **Transport** | 4 | ITransportEngine, ISessionFactory |
| **Realtime** | 6 | IRealtimeHandler, IWebSocketHandler |
| **TOTAL** | **72** | |

---

## 3. TRANSPORT ENGINE - O CORAÇÃO DO I/O

**Arquivo:** `src/web/transport/TransportEngine.cpp` (360 linhas)
**Header:** `include/corgrid/web/transport/TransportEngine.hpp`

### 3.1. Arquitetura

O TransportEngine é o orquestrador central de I/O, gerenciando:
- Accept loops para HTTP e HTTPS
- Detecção de protocolo via PEEK
- Fábricas de sessão por protocolo
- Telemetria de conexões

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         TRANSPORT ENGINE                                │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌──────────────────┐              ┌──────────────────┐                │
│  │   HTTP Acceptor  │              │  HTTPS Acceptor  │                │
│  │   (Port 8080)    │              │   (Port 8443)    │                │
│  └────────┬─────────┘              └────────┬─────────┘                │
│           │                                 │                           │
│           │    co_spawn (Coroutine)         │    + TLS Handshake       │
│           ▼                                 ▼                           │
│  ┌──────────────────────────────────────────────────────────────┐      │
│  │                      PROTOCOL DETECTION                       │      │
│  │                                                               │      │
│  │    PEEK 1024 bytes → detectProtocolFromPeek()                │      │
│  │    • "upgrade: websocket" → protocol = "ws"                  │      │
│  │    • default → protocol = "http"                              │      │
│  └──────────────────────────────────────────────────────────────┘      │
│           │                                                             │
│           ▼                                                             │
│  ┌──────────────────────────────────────────────────────────────┐      │
│  │                    SESSION FACTORY DISPATCH                   │      │
│  │                                                               │      │
│  │    factories_[protocol]->makeSession(socket, ctx)            │      │
│  │    • "http" → HttpCoroutineSessionFactory                    │      │
│  │    • "ws"   → BeastWebSocketAdapterFactory                   │      │
│  └──────────────────────────────────────────────────────────────┘      │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 3.2. Construtor e Configuração

```cpp
// src/web/transport/TransportEngine.cpp:38-75
TransportEngine::TransportEngine(boost::asio::io_context& io_ctx,
                                 const tcp::endpoint& http_endpoint,
                                 std::optional<tcp::endpoint> https_endpoint,
                                 std::shared_ptr<ssl::context> ssl_ctx)
    : io_ctx_(io_ctx),
      http_acceptor_(io_ctx),
      https_acceptor_(https_endpoint ? std::make_optional<tcp::acceptor>(io_ctx) : std::nullopt),
      ssl_ctx_(std::move(ssl_ctx)) {

    // Inicializar contexto base com defaults de corgrid::constants::defaults
    base_ctx_.protocol = "http";
    base_ctx_.read_timeout = corgrid::constants::defaults::web::REQUEST_TIMEOUT;
    base_ctx_.write_timeout = corgrid::constants::defaults::web::REQUEST_TIMEOUT;
    base_ctx_.handshake_timeout = corgrid::constants::defaults::web::HANDSHAKE_TIMEOUT;
    base_ctx_.max_body_size = corgrid::constants::defaults::web::MAX_BODY_SIZE;
    base_ctx_.enable_keep_alive = corgrid::constants::defaults::web::KEEP_ALIVE;
    base_ctx_.realtime_max_message_bytes = corgrid::constants::defaults::web::RT_MAX_MESSAGE_BYTES;
    base_ctx_.realtime_max_queue_depth = corgrid::constants::defaults::web::RT_MAX_QUEUE_DEPTH;
    base_ctx_.realtime_idle_timeout = corgrid::constants::defaults::web::RT_IDLE_TIMEOUT;
    base_ctx_.realtime_ws_enabled = true;
    base_ctx_.realtime_sse_enabled = true;

    // Abrir acceptor HTTP
    http_acceptor_.open(http_endpoint.protocol());
    http_acceptor_.set_option(tcp::acceptor::reuse_address(true));
    http_acceptor_.bind(http_endpoint);
    http_acceptor_.listen();
}
```

### 3.3. Accept Loop HTTP (Coroutine)

```cpp
// src/web/transport/TransportEngine.cpp:175-262
awaitable<void> TransportEngine::httpAcceptLoop() {
    auto executor = co_await boost::asio::this_coro::executor;
    while (running_) {
        tcp::socket socket(executor);
        boost::system::error_code ec;
        co_await http_acceptor_.async_accept(socket, use_awaitable);

        // Detecção de protocolo via PEEK (não consome o socket)
        std::array<char, 1024> peek_buf{};
        std::size_t peek_len = socket.receive(
            boost::asio::buffer(peek_buf),
            tcp::socket::message_peek, ec);

        std::string protocol = detectProtocolFromPeek(peek_buf.data(), peek_len);
        SessionContext ctx = makeDefaultContext(protocol);

        // Dispatch para factory apropriada
        auto it = factories_.find(protocol);
        if (it != factories_.end()) {
            boost::beast::tcp_stream stream(std::move(socket));
            auto result = it->second->makeSession(TransportSocket{std::move(stream)}, ctx);
        }
    }
    co_return;
}
```

### 3.4. Detecção de Protocolo

```cpp
// src/web/transport/TransportEngine.cpp:163-173
static std::string detectProtocolFromPeek(const char* data, std::size_t len) {
    std::string_view sv(data, len);
    auto lower = std::string(sv);
    std::transform(lower.begin(), lower.end(), lower.begin(), ::tolower);

    if (lower.find("upgrade: websocket") != std::string::npos) {
        return "ws";
    }
    // SSE trafega como HTTP normal
    return "http";
}
```

### 3.5. SessionContext (Configuração por Sessão)

```cpp
// include/corgrid/web/transport/TransportTypes.hpp
struct SessionContext {
    std::string session_id;           // UUID único
    std::string protocol;             // "http", "ws", "sse"
    std::string route_path;           // "/api/v1/..."
    std::string method;               // "GET", "POST", etc.

    // Timeouts
    std::chrono::milliseconds read_timeout;
    std::chrono::milliseconds write_timeout;
    std::chrono::milliseconds handshake_timeout;

    // Limites
    std::size_t max_body_size;
    std::size_t realtime_max_message_bytes;
    std::size_t realtime_max_queue_depth;
    std::chrono::seconds realtime_idle_timeout;

    // Flags
    bool enable_keep_alive;
    bool realtime_ws_enabled;
    bool realtime_sse_enabled;

    // Serviços injetados
    corgrid::utils::Logger* logger;
    IMiddlewareManager* middleware_manager;
    services::RouterService* router_service;
};
```

### 3.6. std::expected para Erros

```cpp
// src/web/transport/TransportEngine.cpp:77-89
std::expected<void, SessionError> TransportEngine::registerSessionFactory(
    const std::string& protocol,
    std::shared_ptr<ISessionFactory> factory) {

    if (!factory || protocol.empty()) {
        return std::unexpected(SessionError{
            .stage = "register",
            .code = "invalid_factory",
            .message = "Factory ou protocolo inválido",
            .protocol = protocol
        });
    }
    factories_[protocol] = std::move(factory);
    return {};
}
```

---

## 4. HTTPCOROUTINESESSION - O WORKER ASSÍNCRONO

**Arquivo:** `src/web/http/coroutine/HttpCoroutineSession.cpp` (413 linhas)

### 4.1. Ciclo de Vida

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    HTTP COROUTINE SESSION LIFECYCLE                     │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  1. SPAWN (Factory cria sessão)                                         │
│     └── HttpCoroutineSessionFactory::makeSession()                      │
│         └── co_spawn(executor, session->run(), detached)                │
│                                                                         │
│  2. RUN LOOP (Coroutine principal)                                      │
│     ┌─────────────────────────────────────────────────────────────┐    │
│     │ for (;;) {                                                   │    │
│     │     parser_.emplace();                                       │    │
│     │     parser_->body_limit(ctx_.max_body_size);                │    │
│     │                                                              │    │
│     │     // Read request (as_tuple para no-throw)                 │    │
│     │     auto [ec, bytes] = co_await async_read(...);            │    │
│     │                                                              │    │
│     │     if (ec == http::error::end_of_stream) break;            │    │
│     │                                                              │    │
│     │     // Process request                                       │    │
│     │     co_await handle_request();                              │    │
│     │                                                              │    │
│     │     // Clear buffer                                          │    │
│     │     buffer_.consume(buffer_.size());                        │    │
│     │                                                              │    │
│     │     // Check keep-alive                                      │    │
│     │     if (!parser_->get().keep_alive()) break;                │    │
│     │ }                                                            │    │
│     └─────────────────────────────────────────────────────────────┘    │
│                                                                         │
│  3. SHUTDOWN (RAII cleanup)                                             │
│     └── socket.shutdown(tcp::socket::shutdown_send)                     │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 4.2. handle_request() - Pipeline Completo

```cpp
// src/web/http/coroutine/HttpCoroutineSession.cpp:90-258
net::awaitable<void> HttpCoroutineSession::handle_request() {
    auto beast_req = parser_->release();

    // 1. CONVERTER Beast Request -> Corgrid Request
    corgrid::web::HttpRequest req;
    req.method = corgrid::web::stringToHttpMethod(beast_req.method_string());
    req.target = std::string(beast_req.target());
    req.body = beast_req.body();
    for(auto const& field : beast_req) {
        req.headers[std::string(field.name_string())] = std::string(field.value());
    }

    // 2. CORS MIDDLEWARE (Preflight tratado aqui)
    static auto cors_middleware = corgrid::services::middleware::createBasicCorsMiddleware();
    corgrid::web::HttpResponse cors_response;
    if (!cors_middleware->process(req, cors_response)) {
        // OPTIONS request tratado - responder diretamente
        co_await send_response(cors_response, beast_req.version());
        co_return;
    }

    // 3. MIDDLEWARE PIPELINE
    corgrid::web::HttpResponse res;
    if (ctx_.middleware_manager) {
        bool ok = ctx_.middleware_manager->executeMiddlewarePipeline(req, res);
        if (!ok) {
            co_await send_error_response(res.status_code, res.body, beast_req.version());
            co_return;
        }
    }

    // 4. ROTEAMENTO
    if (ctx_.router_service) {
        auto match = ctx_.router_service->match(req);
        if (match) {
            // Executar middleware chain da rota
            for(auto& mw : match->middleware_chain) {
                if (!mw(req, res)) {
                    break; // Auth falhou, etc.
                }
            }

            // Executar handler
            if (match->type == RouteType::HTTP && match->http_handler) {
                match->http_handler(req, res);

                // 5. PÓS-PROCESSAMENTO: Compressão Zstd
                corgrid::web::middleware::ZstdCompressionMiddleware::process(req, res);
            }
            else if (match->type == RouteType::SSE) {
                co_await handle_sse_request(*match, ...);
                co_return;
            }
        } else {
            res = StandardResponse::error(404, "not_found", "Route not found");
        }
    }

    // 6. ENVIAR RESPOSTA
    co_await send_response(res, beast_req.version());
}
```

### 4.3. SSE Handler (Server-Sent Events)

O sistema implementa SSE nativo com chunked transfer encoding:

```cpp
// src/web/http/coroutine/HttpCoroutineSession.cpp:260-378
net::awaitable<void> HttpCoroutineSession::handle_sse_request(
    const RouteMatch& match,
    const std::unordered_map<std::string, std::string>& extra_headers,
    unsigned http_version) {

    // 1. Enviar Headers SSE Iniciais (Raw HTTP)
    std::string headers = fmt::format(
        "HTTP/1.1 200 OK\r\n"
        "Server: CorgridEdge/3.0\r\n"
        "Content-Type: text/event-stream\r\n"
        "Cache-Control: no-cache\r\n"
        "Connection: keep-alive\r\n"
        "X-Accel-Buffering: no\r\n"
        "Transfer-Encoding: chunked\r\n"
        "\r\n");

    co_await net::async_write(stream, net::buffer(headers), use_awaitable);

    // 2. Estado Compartilhado (Thread-Safe)
    auto sse_state = std::make_shared<SseState>();
    sse_state->executor = get_executor();
    sse_state->timer = std::make_shared<net::steady_timer>(sse_state->executor);

    // 3. Callback de Push (chamado pela thread externa)
    auto push_callback = [sse_state](const std::string& data) -> bool {
        std::lock_guard<std::mutex> lock(sse_state->mutex);
        if (!sse_state->active.load()) return false;

        sse_state->queue.push(data);

        // Acordar corrotina no executor correto (thread-safe)
        net::post(sse_state->executor, [timer = sse_state->timer]() {
            timer->cancel();
        });
        return true;
    };

    // 4. Registrar no handler SSE
    match.sse_handler(push_callback);

    // 5. Loop de Eventos (Coroutine)
    while (sse_state->active.load()) {
        std::string payload;
        {
            std::lock_guard<std::mutex> lock(sse_state->mutex);
            if (!sse_state->queue.empty()) {
                payload = sse_state->queue.front();
                sse_state->queue.pop();
            }
        }

        if (!payload.empty()) {
            // Formatar SSE: data: payload\n\n
            const std::string event = "data: " + payload + "\n\n";
            const std::string chunk = fmt::format("{:x}\r\n{}\r\n", event.size(), event);

            auto [ec, sz] = co_await net::async_write(stream, net::buffer(chunk), as_tuple(use_awaitable));
            if (ec) {
                sse_state->active.store(false);
                break;
            }
        } else {
            // Fila vazia, dormir até novo dado
            sse_state->timer->expires_after(std::chrono::seconds(1));
            co_await sse_state->timer->async_wait(as_tuple(use_awaitable));
        }
    }
}
```

---

## 5. SISTEMA DE MIDDLEWARE (ARQUITETURA COMPLETA)

### 5.1. Componentes do Sistema

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      MIDDLEWARE SYSTEM ARCHITECTURE                     │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                    MiddlewareRegistry                            │   │
│  │  • registerMiddleware(MiddlewareDefinition)                      │   │
│  │  • getMiddleware(name) -> MiddlewareDefinition                   │   │
│  │  • enableMiddleware(name, bool)                                  │   │
│  │  • validateMiddlewareDefinition()                                │   │
│  └───────────────────────────────────────────────────────────────────┘  │
│                              │                                          │
│                              ▼                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                    MiddlewarePipeline                            │   │
│  │  • execute(route_path, request, response) -> bool                │   │
│  │  • determineMiddlewaresToExecute(route_path)                     │   │
│  │  • shouldExecuteMiddleware(definition, route_path)               │   │
│  │  • recordExecutionInfo()                                         │   │
│  └───────────────────────────────────────────────────────────────────┘  │
│                              │                                          │
│      ┌───────────────────────┼───────────────────────┐                 │
│      ▼                       ▼                       ▼                 │
│  ┌──────────────┐   ┌──────────────────┐   ┌──────────────────┐       │
│  │ Ordering     │   │ RouteMatching    │   │ Middleware       │       │
│  │ Strategy     │   │ Strategy         │   │ Statistics       │       │
│  │              │   │                  │   │                  │       │
│  │ Default:     │   │ Regex:           │   │ • recordExec()   │       │
│  │ base_order + │   │ std::regex_match │   │ • getStats()     │       │
│  │ conditional  │   │ + cache          │   │ • Prometheus     │       │
│  └──────────────┘   └──────────────────┘   └──────────────────┘       │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 5.2. MiddlewareDefinition (Estrutura)

```cpp
// include/corgrid/web/middleware/core/MiddlewareRegistry.hpp
struct MiddlewareDefinition {
    std::string name;                    // "basic_cors_middleware"
    std::string type;                    // "cors", "auth", "compression"
    int priority;                        // 0-1000 (maior = executado primeiro)
    bool enabled;                        // true/false

    // Handler function
    std::function<bool(HttpRequest&, HttpResponse&)> handler;

    // Padrões de rota
    std::vector<std::string> route_patterns;     // ["/api/*"]
    std::vector<std::string> excluded_patterns;  // ["/health", "/metrics"]
};
```

### 5.3. MiddlewareRegistry (Registro)

**Arquivo:** `src/web/middleware/core/MiddlewareRegistry.cpp` (213 linhas)

```cpp
// src/web/middleware/core/MiddlewareRegistry.cpp:21-39
bool MiddlewareRegistry::registerMiddleware(const MiddlewareDefinition& middleware) {
    std::unique_lock<std::shared_mutex> lock(registry_mutex_);

    try {
        validateMiddlewareDefinition(middleware);

        if (middlewares_.find(middleware.name) != middlewares_.end()) {
            Logger::instance().warn(fmt::format(
                "MiddlewareRegistry: Middleware '{}' já está registrado",
                middleware.name));
            return false;
        }

        middlewares_[middleware.name] = middleware;
        Logger::instance().info(fmt::format(
            "MiddlewareRegistry: Middleware '{}' (tipo: {}) registrado com sucesso",
            middleware.name, middleware.type));
        return true;
    } catch (const std::exception& e) {
        Logger::instance().error(fmt::format(
            "MiddlewareRegistry: Erro ao registrar middleware '{}': {}",
            middleware.name, e.what()));
        return false;
    }
}
```

### 5.4. Validação de Middleware

```cpp
// src/web/middleware/core/MiddlewareRegistry.cpp:181-209
void MiddlewareRegistry::validateMiddlewareDefinition(const MiddlewareDefinition& middleware) const {
    if (middleware.name.empty()) {
        throw std::invalid_argument("Nome do middleware não pode estar vazio");
    }

    if (middleware.type.empty()) {
        throw std::invalid_argument("Tipo do middleware não pode estar vazio");
    }

    if (!middleware.handler) {
        throw std::invalid_argument("Handler do middleware não pode ser null");
    }

    if (middleware.priority < 0 || middleware.priority > 1000) {
        throw std::invalid_argument("Prioridade do middleware deve estar entre 0 e 1000");
    }

    // Validar padrões de rota
    for (const auto& pattern : middleware.route_patterns) {
        if (pattern.empty()) {
            throw std::invalid_argument("Padrões de rota não podem estar vazios");
        }
    }
}
```

### 5.5. MiddlewarePipeline (Execução)

**Arquivo:** `src/web/middleware/core/MiddlewarePipeline.cpp` (265 linhas)

```cpp
// src/web/middleware/core/MiddlewarePipeline.cpp:93-140
bool MiddlewarePipeline::execute(const std::string& route_path,
                                  web::HttpRequest& request,
                                  web::HttpResponse& response) {
    std::unique_lock<std::shared_mutex> lock(execution_mutex_);
    last_execution_info_.clear();

    try {
        // Template Method Pattern - determinar middlewares a executar
        auto middlewares_to_execute = determineMiddlewaresToExecute(route_path);

        Logger::instance().debug(fmt::format(
            "MiddlewarePipeline: Executando {} middlewares para rota '{}'",
            middlewares_to_execute.size(), route_path));

        // Executar cada middleware na ordem
        for (const auto& middleware_name : middlewares_to_execute) {
            auto middleware = registry_->getMiddleware(middleware_name);

            if (middleware.name.empty()) {
                recordExecutionInfo(middleware_name, route_path, false, false,
                                  std::chrono::microseconds(0), "Middleware não encontrado");
                continue;
            }

            // Verificar se deve executar
            if (!shouldExecuteMiddleware(middleware, route_path)) {
                recordExecutionInfo(middleware_name, route_path, false, true,
                                  std::chrono::microseconds(0), "Pulado por condições");
                continue;
            }

            // Executar
            bool result = executeMiddleware(middleware, route_path, request, response);

            // Se false, parar cadeia
            if (!result) {
                Logger::instance().debug(fmt::format(
                    "MiddlewarePipeline: Cadeia interrompida pelo middleware '{}'",
                    middleware_name));
                return false;
            }
        }

        return true;
    } catch (const std::exception& e) {
        Logger::instance().error(fmt::format(
            "MiddlewarePipeline: Erro durante execução: {}", e.what()));
        return false;
    }
}
```

### 5.6. Route Matching Strategy (Regex com Cache)

```cpp
// src/web/middleware/core/MiddlewarePipeline.cpp:48-77
class RegexRouteMatchingStrategy : public IRouteMatchingStrategy {
public:
    bool matches(const std::string& route_path, const std::string& pattern) {
        try {
            std::shared_lock<std::shared_mutex> lock(cache_mutex_);

            // Verificar cache primeiro
            auto it = regex_cache_.find(pattern);
            if (it != regex_cache_.end()) {
                return std::regex_match(route_path, it->second);
            }

            // Cache miss - criar novo regex (upgrade do lock)
            lock.unlock();
            std::unique_lock<std::shared_mutex> write_lock(cache_mutex_);

            // Double-check após obter write lock
            it = regex_cache_.find(pattern);
            if (it != regex_cache_.end()) {
                return std::regex_match(route_path, it->second);
            }

            // Criar e cachear novo regex
            std::regex compiled_regex(pattern);
            regex_cache_[pattern] = compiled_regex;

            return std::regex_match(route_path, compiled_regex);
        } catch (const std::regex_error& e) {
            Logger::instance().error(fmt::format(
                "RegexRouteMatchingStrategy: Erro de regex para padrão '{}': {}",
                pattern, e.what()));
            return false;
        }
    }

private:
    mutable std::shared_mutex cache_mutex_;
    mutable std::unordered_map<std::string, std::regex> regex_cache_;
};
```

### 5.7. Middleware Execution Info (Telemetria)

```cpp
// src/web/middleware/core/MiddlewarePipeline.cpp:250-262
void MiddlewarePipeline::recordExecutionInfo(
    const std::string& middleware_name,
    const std::string& route_path,
    bool executed,
    bool result,
    std::chrono::microseconds execution_time,
    const std::string& error_message) {

    MiddlewareExecutionInfo info;
    info.name = middleware_name;
    info.route_pattern = route_path;
    info.executed = executed;
    info.result = result;
    info.execution_time = execution_time;
    info.error_message = error_message;

    last_execution_info_.push_back(info);
}
```

---

## 6. MIDDLEWARES BUILTIN

### 6.1. BasicCorsMiddleware

**Arquivo:** `src/web/middleware/BasicCorsMiddleware.cpp` (200 linhas)

| Configuração | Default | Descrição |
|--------------|---------|-----------|
| `allowed_origins` | `["*"]` | Origens permitidas |
| `allowed_methods` | `["GET", "POST", "PUT", "DELETE", "OPTIONS"]` | Métodos permitidos |
| `allowed_headers` | `["Content-Type", "Authorization", "X-Requested-With"]` | Headers permitidos |
| `allow_credentials` | `false` | Permitir cookies |
| `max_age` | `86400` (24h) | Cache de preflight |

```cpp
// src/web/middleware/BasicCorsMiddleware.cpp:25-50
bool BasicCorsMiddleware::process(web::HttpRequest& request, web::HttpResponse& response) {
    if (!enabled_) return true;

    std::string origin;
    auto origin_it = request.headers.find("Origin");
    if (origin_it != request.headers.end()) {
        origin = origin_it->second;
    }

    // Se não é CORS, continuar
    if (origin.empty() && request.method != HttpMethod::OPTIONS) {
        return true;
    }

    // Handle preflight (OPTIONS)
    if (request.method == HttpMethod::OPTIONS) {
        handlePreflightRequest(request, response);
        return false; // Interromper pipeline - OPTIONS tratado
    }

    // Handle simple CORS
    return handleSimpleRequest(request, response);
}
```

### 6.2. BasicLoggingMiddleware

**Arquivo:** `src/web/middleware/BasicLoggingMiddleware.cpp` (248 linhas)

O BasicLoggingMiddleware implementa logging estruturado com múltiplos níveis e métricas de performance.

```cpp
// src/web/middleware/BasicLoggingMiddleware.cpp:24-39
class BasicLoggingMiddleware : public IMiddlewareProcessor {
public:
    enum class LogLevel {
        Debug,   // Log tudo
        Info,    // Log requests e responses
        Warn,    // Log apenas slow requests (>10ms)
        Error    // Log apenas very slow requests (>50ms)
    };

    BasicLoggingMiddleware() : enabled_(true), priority_(1000), log_level_(LogLevel::Info) {
        log_requests_ = true;
        log_responses_ = true;
        log_headers_ = false;   // Desabilitado por default (pode conter tokens)
        log_body_ = false;      // Desabilitado por default (payload grande)
        log_performance_ = true;
    }
};
```

#### Thresholds de Performance

```cpp
// src/web/middleware/BasicLoggingMiddleware.cpp:177-200
void logPerformance(const web::HttpRequest& request, std::chrono::microseconds duration) {
    switch (log_level_) {
        case LogLevel::Info:
            if (duration.count() > 10000) { // > 10ms
                Logger::instance().info(fmt::format("Slow Request: {} {} - {} μs",
                    method, target, duration.count()));
            }
            break;
        case LogLevel::Warn:
            if (duration.count() > 50000) { // > 50ms
                Logger::instance().warn(fmt::format("Very Slow Request: {} {} - {} μs",
                    method, target, duration.count()));
            }
            break;
        case LogLevel::Error:
            if (duration.count() > 100000) { // > 100ms
                Logger::instance().error(fmt::format("Extremely Slow Request: {} {} - {} μs",
                    method, target, duration.count()));
            }
            break;
    }
}
```

#### Estatísticas Atômicas

```cpp
// src/web/middleware/BasicLoggingMiddleware.cpp:237-241
mutable std::atomic<size_t> requests_logged_{0};
mutable std::atomic<size_t> responses_logged_{0};
mutable std::atomic<size_t> errors_logged_{0};
```

### 6.3. ZstdCompressionMiddleware

**Arquivo:** `src/web/middleware/ZstdCompressionMiddleware.cpp` (81 linhas)

```cpp
// src/web/middleware/ZstdCompressionMiddleware.cpp:19-41
bool ZstdCompressionMiddleware::process(HttpRequest& req, HttpResponse& res) {
    if (shouldCompress(req, res) && !res.body.empty()) {
        try {
            std::string compressed = compress(res.body);

            // Só aplicar se valer a pena (menor que original)
            if (compressed.size() < res.body.size()) {
                res.body = std::move(compressed);
                res.headers["Content-Encoding"] = "zstd";
                res.headers["Content-Length"] = std::to_string(res.body.size());
            }
        } catch (const std::exception& e) {
            Logger::instance().error(fmt::format(
                "ZstdMiddleware: Falha na compressão: {}", e.what()));
        }
    }
    return true;
}

bool ZstdCompressionMiddleware::shouldCompress(const HttpRequest& req, const HttpResponse& res) {
    // 1. Cliente aceita zstd?
    auto it = req.headers.find("Accept-Encoding");
    if (it == req.headers.end() || it->second.find("zstd") == std::string::npos) {
        return false;
    }

    // 2. Já comprimido?
    if (res.headers.count("Content-Encoding")) {
        return false;
    }

    // 3. Tamanho mínimo (1KB)
    if (res.body.size() < 1024) {
        return false;
    }

    return true;
}
```

### 6.4. Ordem de Execução Default

| Prioridade | Middleware | Função |
|------------|------------|--------|
| 100 | **CORS** | Valida origem e trata preflight |
| 200 | **Logging** | Mede tempo e loga requisição |
| 300 | **ErrorHandling** | Captura exceções |
| 400 | **Auth** | Valida JWT (se configurado) |
| 500 | **RateLimit** | Limita requisições por IP |
| 900 | **Compression** | Comprime resposta (Zstd) |

---

## 7. MIDDLEWARE MANAGER (FACTORY PATTERN)

**Arquivo:** `src/web/http/webserver/MiddlewareManager.cpp` (507 linhas)

### 7.1. Arquitetura

O MiddlewareManager é o orquestrador de alto nível que configura e executa o pipeline de middlewares usando Factory Pattern para criar middlewares builtin.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      MIDDLEWARE MANAGER ARCHITECTURE                     │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                     MiddlewareConfig                              │   │
│  │  cors_enabled, auth_enabled, logging_enabled, rate_limiting...  │   │
│  └───────────────────────────────────────────────────────────────────┘  │
│                              │                                          │
│                              ▼                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                  configureMiddlewares(config)                    │   │
│  │                                                                  │   │
│  │  if (config.cors_enabled)                                        │   │
│  │      addMiddleware(createCORSMiddleware(), HIGHEST)             │   │
│  │  if (config.security_headers_enabled)                            │   │
│  │      addMiddleware(createSecurityHeadersMiddleware(), HIGHEST)  │   │
│  │  if (config.auth_enabled)                                        │   │
│  │      addMiddleware(createAuthenticationMiddleware(), HIGH)      │   │
│  │  if (config.logging_enabled)                                     │   │
│  │      addMiddleware(createLoggingMiddleware(), MEDIUM)           │   │
│  │  if (config.rate_limiting_enabled)                               │   │
│  │      addMiddleware(createRateLimitingMiddleware(), MEDIUM)      │   │
│  │  if (config.compression_enabled)                                 │   │
│  │      addMiddleware(createCompressionMiddleware(), LOW)          │   │
│  └───────────────────────────────────────────────────────────────────┘  │
│                              │                                          │
│                              ▼                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │         middleware_pipeline_ (std::map<int, vector<Entry>>)      │   │
│  │                                                                  │   │
│  │  Priority 0 (HIGHEST): [CORS, SecurityHeaders]                  │   │
│  │  Priority 1 (HIGH):    [Authentication]                         │   │
│  │  Priority 2 (MEDIUM):  [Logging, RateLimiting]                  │   │
│  │  Priority 3 (LOW):     [Compression]                            │   │
│  └───────────────────────────────────────────────────────────────────┘  │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 7.2. MiddlewareEntry (Estrutura Interna)

```cpp
// src/web/http/webserver/MiddlewareManager.cpp (conceptual)
struct MiddlewareEntry {
    MiddlewareFunction function;     // std::function<bool(HttpRequest&, HttpResponse&)>
    MiddlewarePriority priority;     // HIGHEST, HIGH, MEDIUM, LOW, LOWEST
    std::string name;                // "CORS", "Authentication", etc.
    bool enabled;                    // Hot-toggle sem rebuild
    size_t execution_count{0};       // Telemetria
    size_t failure_count{0};         // Telemetria de erros
};
```

### 7.3. executeMiddlewarePipeline (Execução Thread-Safe)

```cpp
// src/web/http/webserver/MiddlewareManager.cpp:152-191
bool MiddlewareManager::executeMiddlewarePipeline(web::HttpRequest& request,
                                                  web::HttpResponse& response) {
    std::lock_guard<std::mutex> lock(pipeline_mutex_);

    auto start_time = std::chrono::steady_clock::now();

    // Executar middlewares em ordem de prioridade (menor valor = maior prioridade)
    for (auto& [priority, middleware_list] : middleware_pipeline_) {
        for (auto& entry : middleware_list) {
            if (!entry.enabled) continue;

            try {
                entry.execution_count++;
                bool continue_pipeline = entry.function(request, response);

                if (!continue_pipeline) {
                    auto duration = std::chrono::duration_cast<std::chrono::microseconds>(
                        std::chrono::steady_clock::now() - start_time);
                    Logger::instance().debug(fmt::format(
                        "Pipeline interrompido por '{}' após {}μs",
                        entry.name, duration.count()));
                    return false;
                }
            } catch (const std::exception& e) {
                entry.failure_count++;
                Logger::instance().error(fmt::format(
                    "Exceção no middleware '{}': {}", entry.name, e.what()));
                return false; // Segurança: interromper em caso de exceção
            }
        }
    }

    return true;
}
```

### 7.4. Authentication Middleware (JWT + Query Token)

O middleware de autenticação suporta dois modos de token:
1. **Header Authorization**: `Bearer <token>`
2. **Query String**: `?token=<token>` (para SSE)

```cpp
// src/web/http/webserver/MiddlewareManager.cpp:267-406
MiddlewareFunction MiddlewareManager::createAuthenticationMiddleware() const {
    return [this](web::HttpRequest& request, web::HttpResponse& response) -> bool {
        // Ignorar autenticação para extensões estáticas
        const std::string ext = std::filesystem::path(clean_path).extension().string();
        static const std::unordered_set<std::string> static_exts = {
            ".js", ".css", ".map", ".woff2", ".woff", ".ttf", ".otf", ".eot",
            ".svg", ".png", ".jpg", ".jpeg", ".gif", ".webp", ".ico", ".html"
        };
        if (static_exts.count(ext)) return true; // Bypass

        // Verificar rotas públicas do conf.json
        for (const auto& route : current_config_.public_routes) {
            if (clean_path == route || clean_path.find(route) == 0) {
                return true;
            }
        }

        // 1. Tentar Header Authorization
        std::string token;
        auto auth_it = request.headers.find("Authorization");
        if (auth_it != request.headers.end() && auth_it->second.find("Bearer ") == 0) {
            token = auth_it->second.substr(7);
        }

        // 2. Fallback: Query String (para SSE)
        if (token.empty()) {
            std::regex token_regex("token=([^&]+)");
            std::smatch match;
            if (std::regex_search(request.target, match, token_regex)) {
                token = urlDecode(match[1]);
            }
        }

        // Validar com TokenManager
        if (token_manager_ && !token_manager_->validateToken(token)) {
            response.status_code = 401;
            response.body = R"({"error": "Invalid or expired token"})";
            return false;
        }

        return true;
    };
}
```

### 7.5. Rate Limiting Middleware (IP-Based)

```cpp
// src/web/http/webserver/MiddlewareManager.cpp:409-461
MiddlewareFunction MiddlewareManager::createRateLimitingMiddleware() const {
    static std::unordered_map<std::string, std::pair<size_t, time_point>> rate_limits;
    static std::mutex rate_limit_mutex;

    return [](web::HttpRequest& request, web::HttpResponse& response) -> bool {
        // Bypass para arquivos estáticos
        if (static_exts.count(ext)) return true;

        std::lock_guard<std::mutex> lock(rate_limit_mutex);

        const size_t max_requests = 100;       // 100 req/min por IP
        const auto time_window = std::chrono::minutes(1);

        auto& [count, window_start] = rate_limits[client_ip];

        if (now - window_start > time_window) {
            count = 0;
            window_start = now;
        }

        count++;

        if (count > max_requests) {
            response.status_code = 429;
            response.body = R"({"error": "Rate limit exceeded"})";
            response.headers["Retry-After"] = "60";
            return false;
        }

        response.headers["X-RateLimit-Limit"] = std::to_string(max_requests);
        response.headers["X-RateLimit-Remaining"] = std::to_string(max_requests - count);
        return true;
    };
}
```

### 7.6. Security Headers Middleware (OWASP)

```cpp
// src/web/http/webserver/MiddlewareManager.cpp:484-499
MiddlewareFunction MiddlewareManager::createSecurityHeadersMiddleware() const {
    return [](web::HttpRequest&, web::HttpResponse& response) -> bool {
        response.headers["X-Content-Type-Options"] = "nosniff";
        response.headers["X-Frame-Options"] = "SAMEORIGIN";
        response.headers["X-XSS-Protection"] = "1; mode=block";
        response.headers["Strict-Transport-Security"] = "max-age=31536000; includeSubDomains";
        response.headers["Referrer-Policy"] = "strict-origin-when-cross-origin";
        response.headers["Content-Security-Policy"] =
            "default-src 'self'; "
            "script-src 'self' 'unsafe-inline' 'unsafe-eval' unpkg.com; "
            "style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; "
            "img-src 'self' data: https:; "
            "font-src 'self' https://fonts.gstatic.com;";
        return true;
    };
}
```

---

## 8. MIDDLEWARE STATISTICS (OBSERVER PATTERN)

**Arquivo:** `src/web/middleware/core/MiddlewareStatistics.cpp` (309 linhas)

### 8.1. Arquitetura

O MiddlewareStatistics implementa o **Observer Pattern** para notificar sistemas externos sobre execuções de middleware em tempo real.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    MIDDLEWARE STATISTICS ARCHITECTURE                    │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                   recordExecution(name, success, time)           │   │
│  └───────────────────────────────────────────────────────────────────┘  │
│                              │                                          │
│                              ▼                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                     updateStatistics()                           │   │
│  │                                                                  │   │
│  │  statistics_[name].total_executions++                           │   │
│  │  statistics_[name].successful_executions++  (if success)        │   │
│  │  statistics_[name].total_execution_time += time                 │   │
│  │  statistics_[name].min/max_execution_time = min/max             │   │
│  │  statistics_[name].avg_execution_time = total / count           │   │
│  └───────────────────────────────────────────────────────────────────┘  │
│                              │                                          │
│                              ▼                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                  notifyObservers(event)                          │   │
│  │                                                                  │   │
│  │  for (auto& observer : observers_) {                            │   │
│  │      observer->onMiddlewareExecution(event);                    │   │
│  │  }                                                               │   │
│  └───────────────────────────────────────────────────────────────────┘  │
│                              │                                          │
│          ┌───────────────────┴───────────────────┐                     │
│          ▼                                       ▼                     │
│  ┌──────────────────┐                   ┌──────────────────┐          │
│  │ PrometheusExporter│                   │ AlertManager     │          │
│  └──────────────────┘                   └──────────────────┘          │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 8.2. MiddlewareStats (Estrutura)

```cpp
// src/web/middleware/core/MiddlewareStatistics.cpp (conceptual)
struct MiddlewareStats {
    size_t total_executions{0};
    size_t successful_executions{0};
    size_t failed_executions{0};

    std::chrono::microseconds total_execution_time{0};
    std::chrono::microseconds avg_execution_time{0};
    std::chrono::microseconds min_execution_time{max};
    std::chrono::microseconds max_execution_time{0};

    std::chrono::system_clock::time_point first_execution;
    std::chrono::system_clock::time_point last_execution;

    std::string last_error;
};
```

### 8.3. Observer Pattern (Notificação em Tempo Real)

```cpp
// src/web/middleware/core/MiddlewareStatistics.cpp:21-37
void MiddlewareStatistics::recordExecution(const std::string& middleware_name,
                                           bool success,
                                           std::chrono::microseconds execution_time,
                                           const std::string& error_message) {
    updateStatistics(middleware_name, success, execution_time, error_message);

    // Observer Pattern - Notificar observadores
    MiddlewareExecutionEvent event;
    event.middleware_name = middleware_name;
    event.success = success;
    event.execution_time = execution_time;
    event.error_message = error_message;
    event.timestamp = std::chrono::system_clock::now();

    notifyObservers(event);
}
```

### 8.4. Rankings de Performance

```cpp
// src/web/middleware/core/MiddlewareStatistics.cpp:121-144
std::vector<std::string> MiddlewareStatistics::getTopPerformingMiddlewares(size_t limit) const {
    return getTopMiddlewares(limit, [](const auto& a, const auto& b) {
        // Ordenar por taxa de sucesso (descendente) e tempo médio (ascendente)
        double success_rate_a = static_cast<double>(a.second.successful_executions)
                              / a.second.total_executions;
        double success_rate_b = static_cast<double>(b.second.successful_executions)
                              / b.second.total_executions;

        if (success_rate_a != success_rate_b) {
            return success_rate_a > success_rate_b;
        }
        return a.second.avg_execution_time < b.second.avg_execution_time;
    });
}

std::vector<std::string> MiddlewareStatistics::getSlowestMiddlewares(size_t limit) const {
    return getTopMiddlewares(limit, [](const auto& a, const auto& b) {
        return a.second.avg_execution_time > b.second.avg_execution_time;
    });
}

std::vector<std::string> MiddlewareStatistics::getMostFailedMiddlewares(size_t limit) const {
    return getTopMiddlewares(limit, [](const auto& a, const auto& b) {
        return a.second.failed_executions > b.second.failed_executions;
    });
}
```

### 8.5. Métricas JSON (Prometheus-Ready)

```cpp
// src/web/middleware/core/MiddlewareStatistics.cpp:264-281
nlohmann::json MiddlewareStatistics::statsToJson(const MiddlewareStats& stats) const {
    return {
        {"total_executions", stats.total_executions},
        {"successful_executions", stats.successful_executions},
        {"failed_executions", stats.failed_executions},
        {"total_execution_time_us", stats.total_execution_time.count()},
        {"avg_execution_time_us", stats.avg_execution_time.count()},
        {"min_execution_time_us", stats.min_execution_time.count()},
        {"max_execution_time_us", stats.max_execution_time.count()},
        {"last_error", stats.last_error},
        {"success_rate", stats.total_executions > 0 ?
            static_cast<double>(stats.successful_executions) / stats.total_executions : 0.0}
    };
}
```

---

## 9. MIDDLEWARE CHAIN FACTORY (SOLID COMPONENTS)

**Arquivo:** `src/web/middleware/core/MiddlewareChainFactory.cpp` (84 linhas)

### 9.1. Factory Pattern para Componentes SOLID

O MiddlewareChainFactory é responsável por criar e compor todos os componentes do sistema de middleware seguindo os princípios SOLID.

```cpp
// src/web/middleware/core/MiddlewareChainFactory.cpp:23-42
MiddlewareChainFactory::Components MiddlewareChainFactory::createAllComponents(
    std::shared_ptr<IConfigService> config_service,
    std::shared_ptr<TokenManager> token_manager) {

    Logger::instance().info("MiddlewareChainFactory: Criando todos os componentes SOLID");

    Components components;

    // 1. Criar componentes base (sem dependências)
    components.config_manager = createConfigurationManager(config_service);
    components.registry = createRegistry();
    components.statistics = createStatistics();
    components.builtin_factory = createBuiltinFactory(token_manager);

    // 2. Criar pipeline (depende dos outros componentes)
    components.pipeline = createPipeline(
        components.config_manager,
        components.registry,
        components.statistics
    );

    return components;
}
```

### 9.2. Diagrama de Dependências

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    MIDDLEWARE CHAIN FACTORY                             │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  createAllComponents(config_service, token_manager)                     │
│                              │                                          │
│      ┌───────────────────────┼───────────────────────┐                 │
│      │                       │                       │                 │
│      ▼                       ▼                       ▼                 │
│  ┌──────────────┐   ┌──────────────┐   ┌──────────────────┐           │
│  │ Config       │   │ Registry     │   │ Statistics       │           │
│  │ Manager      │   │              │   │                  │           │
│  │              │   │              │   │                  │           │
│  │ IConfigSvc   │   │ (standalone) │   │ (standalone)     │           │
│  └──────┬───────┘   └──────┬───────┘   └────────┬─────────┘           │
│         │                  │                    │                      │
│         └──────────────────┼────────────────────┘                      │
│                            ▼                                           │
│                   ┌──────────────────┐                                 │
│                   │ MiddlewarePipeline│                                 │
│                   │                  │                                 │
│                   │ • ConfigManager  │                                 │
│                   │ • Registry       │                                 │
│                   │ • Statistics     │                                 │
│                   └──────────────────┘                                 │
│                                                                         │
│  createBuiltinFactory(token_manager)                                    │
│                   │                                                     │
│                   ▼                                                     │
│          ┌──────────────────┐                                          │
│          │ BuiltinMiddleware│                                          │
│          │ Factory          │                                          │
│          │                  │                                          │
│          │ • TokenManager   │                                          │
│          └──────────────────┘                                          │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 10. SISTEMA DE ROTEAMENTO (REFACTOREDROUTEREGISTRY)

**Arquivo:** `src/web/routing/registry/RefactoredRouteRegistry.cpp` (284 linhas)

### 10.1. Arquitetura Composition over Inheritance

```cpp
// src/web/routing/registry/RefactoredRouteRegistry.cpp:64-80
RefactoredRouteRegistry::RefactoredRouteRegistry(std::shared_ptr<ServiceRegistry> service_registry)
    : service_registry_(std::move(service_registry))
    , route_discovery_(createBasicRouteDiscovery(service_registry_))
    , route_validator_(createBasicRouteValidator())
    , middleware_manager_(createBasicRouteMiddlewareManager())
    , metrics_collector_(createBasicRouteMetricsCollector())
    , running_(false)
{
    if (!service_registry_) {
        throw std::invalid_argument("ServiceRegistry não pode ser nulo");
    }

    loadConfiguration();
    Logger::instance().info("RefactoredRouteRegistry: Inicializado com Composition over Inheritance");
}
```

### 10.2. Auto-Discovery de Rotas

```cpp
// src/web/routing/registry/RefactoredRouteRegistry.cpp:226-253
void RefactoredRouteRegistry::autoDiscoverRoutes() {
    if (!route_discovery_) return;

    try {
        // Descobrir provedores de rotas
        auto providers = route_discovery_->discoverProviders();
        Logger::instance().info(fmt::format(
            "RefactoredRouteRegistry: {} provedores descobertos", providers.size()));

        // Registrar rotas de cada provedor
        for (const auto& provider : providers) {
            auto routes = route_discovery_->extractRoutesFromProvider(provider);

            for (const auto& [method, path] : routes) {
                Logger::instance().debug(fmt::format(
                    "RefactoredRouteRegistry: Rota descoberta: {} {}", method, path));
            }
        }
    } catch (const std::exception& e) {
        Logger::instance().error(fmt::format(
            "RefactoredRouteRegistry: Erro na descoberta de rotas: {}", e.what()));
    }
}
```

### 10.3. Métricas Prometheus

```cpp
// src/web/routing/registry/RefactoredRouteRegistry.cpp:184-189
std::string RefactoredRouteRegistry::exportPrometheusMetrics() const {
    if (!metrics_collector_) {
        return "";
    }
    return metrics_collector_->exportPrometheusMetrics();
}
```

---

## 11. REALTIME: SSE & WEBSOCKET ENTERPRISE HUBS

### 11.1. EnterpriseSSEHub

**Arquivo:** `src/web/http/EnterpriseSSEHub.cpp` (188 linhas)

```cpp
// src/web/http/EnterpriseSSEHub.cpp:22-29
EnterpriseSSEHub::EnterpriseSSEHub(std::shared_ptr<connections::IConnectionManager> connection_manager)
    : connection_manager_(std::move(connection_manager)) {
    if (!connection_manager_) {
        throw std::invalid_argument("EnterpriseSSEHub: ConnectionManager não pode ser nullptr");
    }
    Logger::instance().info("EnterpriseSSEHub: Inicializado com ConnectionManager");
}

// Registrar rota SSE
void EnterpriseSSEHub::registerRoutes(web::IHttpRouter& router) {
    router.AddSSE("/stream/events", [this](std::function<bool(const std::string&)> push) {
        handleSSEConnection("/stream/events", std::move(push));
    });
    Logger::instance().info("EnterpriseSSEHub: Rota SSE registrada: /stream/events");
}

// Broadcast para path
void EnterpriseSSEHub::broadcastToPath(const std::string& path, const nlohmann::json& message) {
    auto connections = connection_manager_->getConnectionsByPath(path);
    const std::string payload = message.dump();

    for (const auto& conn_id : connections) {
        auto context = connection_manager_->getConnection(conn_id);
        if (context) {
            context->send(payload);
        }
    }
}
```

### 11.2. EnterpriseWebSocketHub

**Arquivo:** `src/web/websocket/EnterpriseWebSocketHub.cpp` (212 linhas)

```cpp
// src/web/websocket/EnterpriseWebSocketHub.cpp:61-68
void EnterpriseWebSocketHub::registerRoutes(web::IHttpRouter& router) {
    router.AddWebSocket("/stream/ws", [this](web::WebSocketContext& ctx, const std::string& message) {
        handleWebSocketMessage(ctx, message);
    });
    Logger::instance().info("EnterpriseWebSocketHub: Rota WebSocket registrada: /stream/ws");
}

// Dispatch por canal
void EnterpriseWebSocketHub::handleWebSocketMessage(web::WebSocketContext& ctx, const std::string& message) {
    if (!running_.load()) {
        ctx.close();
        return;
    }

    // Criar/obter conexão no ConnectionManager
    const std::string endpoint_key = ctx.remote_address + "|" + ctx.path;
    std::string connection_id;
    {
        std::lock_guard<std::mutex> lock(connections_mutex_);
        auto it = connection_id_by_endpoint_.find(endpoint_key);
        if (it != connection_id_by_endpoint_.end()) {
            connection_id = it->second;
        } else {
            connection_id = connection_manager_->createConnection("WebSocket", ctx.path);
            connection_id_by_endpoint_[endpoint_key] = connection_id;
        }
    }

    // Vincular callbacks
    auto connection_ctx = connection_manager_->getConnection(connection_id);
    connection_ctx->setSendCallback([send = ctx.send](const std::string& data) {
        send(data);
    });

    // Dispatch para handler registrado
    EnterpriseWebSocketHandler channel_handler;
    {
        std::lock_guard<std::mutex> lock(channels_mutex_);
        auto it = channels_.find(ctx.path);
        if (it != channels_.end()) {
            channel_handler = it->second;
        }
    }

    if (channel_handler) {
        channel_handler(connection_ctx, message);
    }
}
```

### 11.3. Comparativo SSE vs WebSocket

| Característica | SSE | WebSocket |
|----------------|-----|-----------|
| **Direção** | Unidirecional (Server → Client) | Bidirecional |
| **Protocolo** | HTTP padrão | Upgrade HTTP → WS |
| **Reconexão** | Automática (browser) | Manual |
| **Firewall** | Friendly | Pode bloquear |
| **Overhead** | Menor | Maior (handshake) |
| **Uso** | Telemetria, logs, status | Chat, controle remoto |

---

## 12. GERENCIAMENTO DE MEMÓRIA (POOLS)

**Arquivo:** `src/web/http/BasicMemoryPool.cpp`

### 12.1. BasicMemoryPool

```cpp
// include/corgrid/interfaces/web/http/BasicMemoryPool.hpp
class BasicMemoryPool : public IMemoryPool {
public:
    void* allocate(std::size_t size);
    void deallocate(void* ptr, std::size_t size);
    void reset();
    std::size_t getPoolSize() const;
    std::size_t getUsedSize() const;

private:
    std::vector<std::unique_ptr<char[]>> blocks_;
    std::size_t block_size_;
    std::size_t used_size_;
    std::mutex mutex_;
};
```

### 12.2. Uso em Sessões

```cpp
// Cada sessão pode ter seu próprio memory pool
SessionContext ctx;
ctx.memory_pool = std::make_shared<BasicMemoryPool>(64 * 1024); // 64KB blocks

// Buffers alocados do pool
auto buffer = ctx.memory_pool->allocate(4096);
```

---

## 13. INTERFACES WEB (ISP - INTERFACE SEGREGATION)

### 13.1. Composição de IHttpServer

```
IHttpServer (interface completa)
├── IWebServerLifecycle      → start(), stop(), isRunning()
├── IWebServerRouting        → registerRoute(), getRoutes()
├── IWebServerConfiguration  → getConfig(), setConfig()
├── IWebServerMonitoring     → getMetrics(), getHealth()
└── IMiddlewareManager       → executeMiddlewarePipeline()
```

### 13.2. Interfaces de Middleware

| Interface | Responsabilidade |
|-----------|------------------|
| `IMiddlewareProcessor` | Processar request/response |
| `IMiddlewareRegistry` | Registrar/gerenciar middlewares |
| `IMiddlewarePipeline` | Executar cadeia de middlewares |
| `IMiddlewareStatistics` | Métricas de execução |
| `IMiddlewareConfigLoader` | Carregar config de arquivo |

### 13.3. Interfaces de Roteamento

| Interface | Responsabilidade |
|-----------|------------------|
| `IRouteDiscovery` | Auto-descoberta de rotas |
| `IRouteValidator` | Validar conflitos de rotas |
| `IRouteMetrics` | Métricas por rota |
| `IRouteMiddleware` | Middleware específico de rota |

---

## 14. C++23 FEATURES EM USO

| Feature | Onde é Usado | Exemplo |
|---------|--------------|---------|
| **co_await** | HttpCoroutineSession, TransportEngine | `co_await http::async_read(...)` |
| **net::as_tuple** | Toda sessão HTTP | `auto [ec, sz] = co_await ...` |
| **std::expected** | TransportEngine | `std::expected<void, SessionError>` |
| **std::visit** | HttpCoroutineSession | Polimorfismo estático de socket |
| **std::format** | Logging (via fmt) | `fmt::format("Error: {}", msg)` |
| **std::shared_mutex** | MiddlewareRegistry, Pipeline | Reader-writer locks |
| **Structured bindings** | Todo o código | `auto [ec, bytes] = ...` |

---

## 15. ROADMAP DE ENGENHARIA (C++23/26)

### 15.1. Já Implementado

| Feature | Status | Arquivo |
|---------|--------|---------|
| C++20 Coroutines | Completo | HttpCoroutineSession, TransportEngine |
| std::expected | Completo | TransportEngine |
| Zstd Compression | Completo | ZstdCompressionMiddleware |
| SSE Enterprise | Completo | EnterpriseSSEHub |
| WebSocket Enterprise | Completo | EnterpriseWebSocketHub |
| Middleware Pipeline | Completo | MiddlewarePipeline |

### 15.2. Próximas Iterações

#### 15.2.1. io_uring (Linux 5.10+)
- **Alvo:** Substituir epoll por io_uring para I/O de rede
- **Benefício:** Zero syscall overhead
- **Suporte Boost:** `BOOST_ASIO_HAS_IO_URING`
- **Prioridade:** Alta

#### 15.2.2. HTTP/3 (QUIC)
- **Alvo:** Suporte nativo a HTTP/3 via QUIC
- **Benefício:** Menor latência, sem head-of-line blocking
- **Prioridade:** Média

#### 15.2.3. std::generator (C++23)
- **Alvo:** Usar para streaming de SSE
- **Benefício:** Código mais limpo para iteradores assíncronos
- **Prioridade:** Média

#### 15.2.4. Coroutine Executors (C++26)
- **Alvo:** Substituir boost::asio::any_io_executor
- **Benefício:** Padrão nativo de executors
- **Prioridade:** Futura

---

## 16. DEBUGGING E TROUBLESHOOTING

### 16.1. Conexões Travando

**Sintoma:** Requests não respondem

1. Verificar timeout configurado:
   ```cpp
   ctx.read_timeout = std::chrono::seconds(30);
   ctx.write_timeout = std::chrono::seconds(30);
   ```

2. Verificar middleware pipeline:
   ```cpp
   auto info = pipeline->getLastExecutionInfo();
   for (const auto& exec : info) {
       Logger::instance().debug(fmt::format(
           "Middleware '{}': executed={}, result={}, time={}μs",
           exec.name, exec.executed, exec.result, exec.execution_time.count()));
   }
   ```

### 16.2. CORS Falhando

**Sintoma:** Requests bloqueados no browser

1. Verificar headers enviados:
   ```bash
   curl -v -X OPTIONS http://localhost:8080/api/v1/test \
     -H "Origin: http://localhost:3000" \
     -H "Access-Control-Request-Method: POST"
   ```

2. Verificar logs do BasicCorsMiddleware:
   ```cpp
   // Habilitar log de stats
   auto stats = cors_middleware->getStats();
   Logger::instance().info(stats.dump());
   ```

### 16.3. Compressão Não Funcionando

**Sintoma:** Responses não comprimidas

1. Verificar header Accept-Encoding:
   ```bash
   curl -v http://localhost:8080/api/v1/test -H "Accept-Encoding: zstd"
   ```

2. Verificar tamanho mínimo (1KB):
   ```cpp
   // Response precisa ter > 1024 bytes para comprimir
   if (res.body.size() < 1024) {
       // Compressão pulada
   }
   ```

### 16.4. SSE Desconectando

**Sintoma:** Stream SSE fecha inesperadamente

1. Verificar timeout de idle:
   ```cpp
   ctx.realtime_idle_timeout = std::chrono::seconds(60);
   ```

2. Verificar estado do SseState:
   ```cpp
   if (!sse_state->active.load()) {
       // Cliente desconectou
   }
   ```

---

**Fim do Documento.**

Este manual é a **Fonte da Verdade** para o módulo Web do CorGrid OS. Qualquer alteração em `src/web/` deve ser refletida aqui.

**Última atualização:** 20/12/2025
**Baseado em análise de:** 34 arquivos .cpp, 72 interfaces .hpp (~5.500 linhas de código)
**Seções:** 16 capítulos cobrindo TransportEngine, HttpCoroutineSession, Middleware System, MiddlewareManager, MiddlewareStatistics, Factory Pattern, Roteamento, SSE/WebSocket, Memory Pools, Interfaces ISP, C++23 Features, Roadmap e Troubleshooting
**Mantenedor:** Core Architecture Team
