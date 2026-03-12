# Hummingbird em Swift

**Hummingbird** é um framework web moderno, leve e focado em performance para desenvolvimento Server-Side Swift. Construído sobre **SwiftNIO** (engine de rede assíncrona), oferece uma base sólida para APIs e aplicações web escaláveis.

## Características Principais

* **Leve:** Dependências mínimas, arquitetura modular
* **Rápido:** Built on SwiftNIO, otimizado para performance
* **Tipo-safe:** Aproveita o sistema de tipos do Swift
* **Assíncrono:** Usa `async/await` para operações não-bloqueantes
* **Pronto para Produção:** Usado em aplicações comerciais
* **Comunidade Ativa:** Desenvolvido e mantido pela comunidade

## Instalação

### Package.swift

```swift
import PackageDescription

let package = Package(
    name: "MyHummingbirdAPI",
    dependencies: [
        .package(url: "https://github.com/hummingbird-project/hummingbird.git", from: "1.0.0"),
    ],
    targets: [
        .executableTarget(
            name: "MyHummingbirdAPI",
            dependencies: [
                .product(name: "Hummingbird", package: "hummingbird"),
            ]
        )
    ]
)
```

## Servidor Básico

```swift
import Hummingbird

@main
struct HelloApp {
    static func main() async throws {
        let app = HBApplication()
        
        // Rota simples
        app.router.get("/") { _ in
            return "Hello, Hummingbird!"
        }
        
        // Iniciar servidor
        try await app.runService()
    }
}
```

**Executar:**
```bash
swift run
# Servidor rodando em http://localhost:8080
```

## Roteamento

### Rotas HTTP Básicas

```swift
import Hummingbird

let app = HBApplication()

// GET
app.router.get("/hello") { _ in
    return "Hello!"
}

// POST
app.router.post("/data") { request in
    let body = try await request.decode(as: String.self)
    return "Recebido: \(body)"
}

// PUT
app.router.put("/users/{id}") { request in
    guard let userId = request.parameters.get("id") else {
        throw HTTPError(.badRequest)
    }
    return "Atualizando usuário \(userId)"
}

// DELETE
app.router.delete("/users/{id}") { request in
    guard let userId = request.parameters.get("id") else {
        throw HTTPError(.badRequest)
    }
    return "Deletando usuário \(userId)"
}
```

### Parâmetros Query

```swift
app.router.get("/search") { request in
    let query = request.uri.queryParameters.get("q") ?? "default"
    let limit = request.uri.queryParameters.get("limit") ?? "10"
    
    return "Buscando: \(query), Limite: \(limit)"
}

// Uso: /search?q=swift&limit=20
```

## JSON e Codables

### Decodificar Request JSON

```swift
import Hummingbird
import Foundation

struct User: Codable {
    let name: String
    let email: String
}

@main
struct APIApp {
    static func main() async throws {
        let app = HBApplication()
        
        app.router.post("/users") { request -> HTTPResponse.Status in
            let user = try await request.decode(as: User.self)
            print("Usuário recebido: \(user.name)")
            
            return .created
        }
        
        try await app.runService()
    }
}
```

### Retornar JSON

```swift
struct UserResponse: Codable {
    let id: Int
    let name: String
    let email: String
}

app.router.get("/users/{id}") { request -> UserResponse in
    guard let id = request.parameters.get("id", as: Int.self) else {
        throw HTTPError(.badRequest)
    }
    
    return UserResponse(id: id, name: "Alice", email: "alice@example.com")
}
```

## Middleware

Processe requests antes de chegar às rotas:

```swift
import Hummingbird

struct LoggingMiddleware: HBMiddleware {
    func apply(to request: HBRequest, next: HBResponder) async throws -> HBResponse {
        let startTime = Date()
        let response = try await next.respond(to: request)
        let elapsed = Date().timeIntervalSince(startTime)
        
        print("[\(request.method)] \(request.uri.path) - \(String(format: "%.3f", elapsed))s")
        
        return response
    }
}

let app = HBApplication()
app.middleware.add(LoggingMiddleware())
```

## Autenticação com Middleware

```swift
struct AuthMiddleware: HBMiddleware {
    func apply(to request: HBRequest, next: HBResponder) async throws -> HBResponse {
        guard let token = request.headers.get("Authorization") else {
            throw HTTPError(.unauthorized)
        }
        
        // Validar token
        if !token.hasPrefix("Bearer ") {
            throw HTTPError(.unauthorized)
        }
        
        return try await next.respond(to: request)
    }
}

app.middleware.add(AuthMiddleware())

app.router.get("/protected") { _ in
    return "Acesso permitido!"
}
```

## Persistência com Fluent (ORM)

### Package.swift com Fluent

```swift
.package(url: "https://github.com/vapor/fluent.git", from: "4.0.0"),
.package(url: "https://github.com/vapor/fluent-postgres-driver.git", from: "2.0.0"),
```

### Definir Modelo

```swift
import Fluent
import FluentPostgresDriver

final class User: Model {
    static let schema = "users"
    
    @ID(custom: .id) var id: Int?
    @Field(key: "name") var name: String
    @Field(key: "email") var email: String
    
    init() {}
    
    init(id: Int? = nil, name: String, email: String) {
        self.id = id
        self.name = name
        self.email = email
    }
}

struct CreateUsersTable: Migration {
    func prepare(on database: Database) async throws {
        try await database.schema("users")
            .id()
            .field("name", .string, .required)
            .field("email", .string, .required, .unique)
            .create()
    }
    
    func revert(on database: Database) async throws {
        try await database.schema("users").delete()
    }
}
```

### Usar Banco de Dados

```swift
import Fluent

@main
struct AppWithDB {
    static func main() async throws {
        let app = HBApplication()
        
        // Configurar banco de dados
        let database = Database(postgres: .init(
            hostname: "localhost",
            username: "postgres",
            password: "password",
            database: "myapp"
        ))
        
        app.extensions.set(\.database, to: database)
        
        // Rotas CRUD
        app.router.post("/users") { request -> User in
            let newUser = try await request.decode(as: User.self)
            try await newUser.save(on: database)
            return newUser
        }
        
        app.router.get("/users") { _ in
            return try await User.query(on: database).all()
        }
        
        app.router.get("/users/{id}") { request -> User? in
            guard let id = request.parameters.get("id", as: Int.self) else {
                throw HTTPError(.badRequest)
            }
            return try await User.find(id, on: database)
        }
        
        try await app.runService()
    }
}
```

## Exemplo Completo: API de Tarefas

```swift
import Hummingbird
import Foundation

struct Task: Codable {
    var id: Int
    var title: String
    var completed: Bool = false
}

@main
struct TaskAPI {
    static func main() async throws {
        let app = HBApplication()
        
        // Dados em memória (substitua por banco de dados em produção)
        var tasks: [Task] = [
            Task(id: 1, title: "Aprender Hummingbird", completed: false),
            Task(id: 2, title: "Criar API REST", completed: true)
        ]
        
        // GET /tasks
        app.router.get("/tasks") { _ -> [Task] in
            return tasks
        }
        
        // GET /tasks/{id}
        app.router.get("/tasks/{id}") { request -> Task in
            guard let id = request.parameters.get("id", as: Int.self),
                  let task = tasks.first(where: { $0.id == id }) else {
                throw HTTPError(.notFound)
            }
            return task
        }
        
        // POST /tasks
        app.router.post("/tasks") { request -> Task in
            var newTask = try await request.decode(as: Task.self)
            newTask.id = (tasks.max(by: { $0.id < $1.id })?.id ?? 0) + 1
            tasks.append(newTask)
            return newTask
        }
        
        // PUT /tasks/{id}
        app.router.put("/tasks/{id}") { request -> Task in
            guard let id = request.parameters.get("id", as: Int.self),
                  let index = tasks.firstIndex(where: { $0.id == id }) else {
                throw HTTPError(.notFound)
            }
            
            var updatedTask = try await request.decode(as: Task.self)
            updatedTask.id = id
            tasks[index] = updatedTask
            return updatedTask
        }
        
        // DELETE /tasks/{id}
        app.router.delete("/tasks/{id}") { request in
            guard let id = request.parameters.get("id", as: Int.self),
                  let index = tasks.firstIndex(where: { $0.id == id }) else {
                throw HTTPError(.notFound)
            }
            
            tasks.remove(at: index)
            return HTTPResponse(status: .noContent)
        }
        
        try await app.runService()
    }
}
```

**Testar:**
```bash
# GET
curl http://localhost:8080/tasks

# POST
curl -X POST http://localhost:8080/tasks \
  -H "Content-Type: application/json" \
  -d '{"title":"Nova tarefa"}'

# PUT
curl -X PUT http://localhost:8080/tasks/1 \
  -H "Content-Type: application/json" \
  -d '{"title":"Tarefa atualizada","completed":true}'

# DELETE
curl -X DELETE http://localhost:8080/tasks/1
```

## Performance e Escalabilidade

Hummingbird é otimizado para:
- **Concorrência:** Usa `async/await` para operações simultâneas
- **Memória:** Gerenciamento eficiente de conexões
- **Throughput:** Pode processar milhares de requests por segundo

## Deployment

### Compilar Release

```bash
swift build -c release
./.build/release/MyHummingbirdAPI
```

### Docker

```dockerfile
FROM swiftlang:latest

WORKDIR /app
COPY . .

RUN swift build -c release

EXPOSE 8080

CMD ["./.build/release/MyHummingbirdAPI"]
```

## Comparação com Outros Frameworks

| Aspecto | Hummingbird | Vapor | Perfect |
|---|---|---|---|
| Performance | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| Facilidade | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| Comunidade | Crescente | Grande | Pequena |
| Async/Await | Nativo | Suportado | Limitado |
| Documentação | Boa | Excelente | Média |

## Recursos

- [Documentação Oficial](https://hummingbird.codes/)
- [GitHub Repository](https://github.com/hummingbird-project/hummingbird)
- [SwiftNIO](https://github.com/apple/swift-nio)

**Relacionados:** [[Backend]] [[Vapor]] [[Perfect]] [[Kitura]]
