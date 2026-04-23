# HexagonalArch

Projeto de referência em **C# / ASP.NET Core 8** que demonstra a **Arquitetura Hexagonal** (Ports & Adapters) aplicada a um CRUD de usuários.

---

## Diagrama da Arquitetura Hexagonal

```mermaid
%%{init: {'theme': 'dark', 'flowchart': {'curve': 'basis'}}}%%
flowchart LR

    %% ── ATOR EXTERNO ──────────────────────────────────────────────────────
    Client(["👤  Cliente\n     HTTP"])

    %% ── ADAPTADOR PRIMÁRIO  ❰ Lado Condutor — Inbound ❱ ─────────────────
    subgraph PRI["🔵  ADAPTADOR PRIMÁRIO"]
        CTRL{{"UsersController\nPOST · GET\nPUT · DELETE\n/api/users"}}
    end

    %% ── HEXÁGONO — NÚCLEO ────────────────────────────────────────────────
    subgraph HEX["⬡ ─────────────  N Ú C L E O  ───────────── ⬡"]
        direction TB

        subgraph APP["Application Layer"]
            SVC{{"UserServiceManager\n〈 implements IUserService 〉"}}
        end

        subgraph PORTS["Ports  —  Domain Layer"]
            direction LR
            PSVC{{"⬡  IUserService"}}
            PREPO{{"⬡  IUserRepository"}}
            PEMAIL{{"⬡  IEmailService"}}
        end

        subgraph DOM["Domain Layer"]
            direction LR
            ENT(["User\n(Entidade)"])
            VO(["EmailVo · NameVo\n(Value Objects)"])
            EXC(["InvalidEmailException\nInvalidNameException"])
        end
    end

    %% ── ADAPTADORES SECUNDÁRIOS  ❰ Lado Conduzido — Outbound ❱ ──────────
    subgraph SEC["🟠  ADAPTADORES SECUNDÁRIOS"]
        direction TB
        REPO["🗄  UserRepository\nEF Core · InMemory\nInfra.Data"]
        MAIL["📧  FakeEmailAdapter\nInfra.Email"]
    end

    %% ── FLUXO DE COMUNICAÇÃO ─────────────────────────────────────────────
    Client    --   "HTTP Request"    -->  CTRL
    CTRL      --   "▶  usa"         -->  PSVC
    PSVC      --   "implementa"     -->  SVC
    SVC       --   "usa  ▶"         -->  PREPO
    SVC       --   "usa  ▶"         -->  PEMAIL
    PREPO     --   "implementa"     -->  REPO
    PEMAIL    --   "implementa"     -->  MAIL
    SVC       -. "opera em"     .-> ENT
    SVC       -. "valida com"   .-> VO
    SVC       -. "lança"        .-> EXC

    %% ── ESTILOS POR CAMADA ───────────────────────────────────────────────
    classDef coreStyle   fill:#0f3460,stroke:#e94560,color:#ffffff,stroke-width:3px
    classDef portStyle   fill:#1b4332,stroke:#52b788,color:#ffffff,stroke-width:2px
    classDef priStyle    fill:#2d1b4e,stroke:#c77dff,color:#ffffff,stroke-width:2px
    classDef secStyle    fill:#1a2744,stroke:#f4a261,color:#ffffff,stroke-width:2px
    classDef entityStyle fill:#0d1117,stroke:#a8dadc,color:#a8dadc,stroke-width:1px
    classDef clientStyle fill:#3d0000,stroke:#e63946,color:#ffffff,stroke-width:2px

    class SVC coreStyle
    class PSVC,PREPO,PEMAIL portStyle
    class CTRL priStyle
    class REPO,MAIL secStyle
    class ENT,VO,EXC entityStyle
    class Client clientStyle
```

### Legenda de cores

| Cor | Camada | Papel na Arquitetura |
|---|---|---|
| 🔴 Vermelho | Cliente HTTP | Ator externo que dispara requisições |
| 🟣 Roxo | Adaptador Primário | Traduz HTTP → chamada ao núcleo (`UsersController`) |
| 🟢 Verde | Portas (Ports) | Interfaces que isolam o núcleo do mundo externo |
| 🔵 Azul | Núcleo / Application | Orquestra a lógica de negócio (`UserServiceManager`) |
| ⬛ Escuro | Domain | Entidades, Value Objects e Exceções puras |
| 🟠 Laranja | Adaptadores Secundários | Implementam as portas de saída (dados e e-mail) |

---

## Estrutura de Projetos

```
HexagonalArch.sln
├── Domain/               # Núcleo puro — sem dependências externas
│   ├── Entities/         # User
│   ├── ValueObjects/     # EmailVo, NameVo
│   ├── Exceptions/       # InvalidEmailException, InvalidNameException
│   └── Ports/            # IUserService, IUserRepository, IEmailService
│
├── Application/          # Orquestração dos casos de uso
│   └── Services/         # UserServiceManager (implements IUserService)
│
├── API/                  # Adaptador primário — REST
│   └── Controllers/      # UsersController
│
├── Infra.Data/           # Adaptador secundário — persistência
│   ├── Context/          # InMemoryContext (EF Core)
│   └── Repositories/     # UserRepository (implements IUserRepository)
│
└── Infra.Email/          # Adaptador secundário — e-mail
    └── Operations/       # FakeEmailAdapter (implements IEmailService)
```

---

## Conceitos da Arquitetura Hexagonal

| Conceito | Descrição | Exemplos no projeto |
|---|---|---|
| **Núcleo (Core)** | Lógica de negócio isolada, sem dependência de frameworks | `Domain`, `Application` |
| **Portas (Ports)** | Interfaces que definem os contratos de entrada e saída | `IUserService`, `IUserRepository`, `IEmailService` |
| **Adaptadores Primários** | Recebem requisições externas e as traduzem para o núcleo | `UsersController` |
| **Adaptadores Secundários** | Implementam as portas de saída (persistência, e-mail, etc.) | `UserRepository`, `FakeEmailAdapter` |

---

## Stack

- **Linguagem:** C# com .NET 8
- **Framework:** ASP.NET Core 8 (Web API)
- **ORM:** Entity Framework Core (InMemory para desenvolvimento)
- **Documentação API:** Swagger / OpenAPI (Swashbuckle)
- **Injeção de Dependência:** Microsoft.Extensions.DependencyInjection

---

## Como Executar

```bash
# Restaurar dependências e executar a API
dotnet run --project API
```

A API estará disponível em `https://localhost:{porta}` com Swagger em `/swagger`.

### Endpoints disponíveis

| Método | Rota | Descrição |
|---|---|---|
| `GET` | `/api/users` | Lista todos os usuários |
| `POST` | `/api/users` | Cadastra novo usuário |
| `PUT` | `/api/users/{id}` | Atualiza um usuário |
| `DELETE` | `/api/users/{id}` | Remove um usuário |
