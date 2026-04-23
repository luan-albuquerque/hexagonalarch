# HexagonalArch

Projeto de referência em **C# / ASP.NET Core 8** que demonstra a **Arquitetura Hexagonal** (Ports & Adapters) aplicada a um CRUD de usuários.

---

## Diagrama da Arquitetura

```mermaid
graph LR
    Client(["👤 Cliente\nHTTP"])

    subgraph PrimaryAdapters["Adaptadores Primários (Inbound)"]
        direction TB
        Controller["UsersController\nGET · POST · PUT · DELETE /api/users"]
    end

    subgraph Core["⬡  N Ú C L E O  ⬡"]
        direction TB
        subgraph Domain["Domain"]
            Ports["Portas (Ports)\nIUserService\nIUserRepository\nIEmailService"]
            Entity["User (Entidade)"]
            VO["EmailVo · NameVo\n(Value Objects)"]
            Exc["InvalidEmailException\nInvalidNameException"]
        end
        subgraph Application["Application"]
            AppSvc["UserServiceManager\nimplements IUserService"]
        end
    end

    subgraph SecondaryAdapters["Adaptadores Secundários (Outbound)"]
        direction TB
        Repo["UserRepository\nEF Core InMemory"]
        EmailSvc["FakeEmailAdapter\nIEmailService"]
    end

    Client -->|"HTTP Request"| Controller
    Controller -->|"IUserService"| AppSvc
    AppSvc -->|"IUserRepository"| Repo
    AppSvc -->|"IEmailService"| EmailSvc
    AppSvc -. "opera em" .-> Entity
    AppSvc -. "valida via" .-> VO
```

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
