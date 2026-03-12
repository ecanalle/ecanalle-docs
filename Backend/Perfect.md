# Perfect em Swift

**Perfect** é um framework web para desenvolvimento Server-Side Swift, permitindo criar aplicações backend robustas usando a linguagem Swift. Embora esteja em manutenção (não desenvolvimento ativo), ainda oferece funcionalidades sólidas.

## Características do Perfect

* **Completo:** Inclui servidor HTTP, ORM, routing, autenticação e mais
* **Maduro:** Um dos primeiros frameworks Swift para web
* **Extensível:** Grande número de bibliotecas e módulos
* **Cross-platform:** Funciona em macOS e Linux
* **Type-safe:** Aproveita o sistema de tipos do Swift

## Instalação e Setup

### Criar Projeto Perfect

```bash
# Via git clone (recomendado)
git clone https://github.com/PerfectlySoft/Perfect-Template.git MyAPI
cd MyAPI

# Via SwiftPM manualmente
mkdir MyAPI && cd MyAPI
swift package init --type executable
```

### Package.swift

```swift
import PackageDescription

let package = Package(
    name: "MyAPI",
    dependencies: [
        .package(url: "https://github.com/PerfectlySoft/Perfect.git", from: "3.1.0"),
        .package(url: "https://github.com/PerfectlySoft/Perfect-HTTPServer.git", from: "3.1.0"),
        .package(url: "https://github.com/PerfectlySoft/Perfect-Routing.git", from: "3.1.0"),
    ],
    targets: [
        .executableTarget(
            name: "MyAPI",
            dependencies: [
                .product(name: "Perfect", package: "Perfect"),
                .product(name: "PerfectHTTPServer", package: "Perfect-HTTPServer"),
                .product(name: "PerfectRouting", package: "Perfect-Routing"),
            ]
        )
    ]
)
```

## Servidor HTTP Básico

```swift
import PerfectHTTPServer

// Criar e configurar servidor
let server = HTTPServer()
server.serverPort = 8080

// Rotas simples
server.routes.add(method: .get, uri: "/") { request, response in
    response.setHeader(.contentType, value: "text/plain")
    response.appendBody(string: "Hello, Perfect!")
    response.completed()
}

server.routes.add(method: .get, uri: "/hello/{name}") { request, response in
    let name = request.pathVariables["name"] ?? "World"
    response.appendBody(string: "Hello, \(name)!")
    response.completed()
}

// Iniciar servidor
do {
    try server.start()
} catch {
    print("Erro ao iniciar servidor: \(error)")
}
```

## Roteamento

### Rotas Básicas

```swift
// GET
server.routes.add(method: .get, uri: "/users") { req, res in
    res.appendBody(string: "Lista de usuários")
    res.completed()
}

// POST
server.routes.add(method: .post, uri: "/users") { req, res in
    res.appendBody(string: "Usuário criado")
    res.completed()
}

// PUT
server.routes.add(method: .put, uri: "/users/{id}") { req, res in
    let id = req.pathVariables["id"] ?? ""
    res.appendBody(string: "Usuário \(id) atualizado")
    res.completed()
}

// DELETE
server.routes.add(method: .delete, uri: "/users/{id}") { req, res in
    let id = req.pathVariables["id"] ?? ""
    res.appendBody(string: "Usuário \(id) deletado")
    res.completed()
}
```

### Parâmetros Query

```swift
server.routes.add(method: .get, uri: "/search") { req, res in
    let query = req.param(name: "q") ?? ""
    let limit = req.param(name: "limit") ?? "10"
    
    res.appendBody(string: "Buscando: \(query), Limite: \(limit)")
    res.completed()
}

// Uso: GET /search?q=swift&limit=20
```

## JSON

### Codificar Resposta JSON

```swift
import Foundation

struct User: Codable {
    let id: Int
    let name: String
    let email: String
}

server.routes.add(method: .get, uri: "/users") { req, res in
    let users = [
        User(id: 1, name: "Alice", email: "alice@example.com"),
        User(id: 2, name: "Bob", email: "bob@example.com")
    ]
    
    do {
        let jsonData = try JSONEncoder().encode(users)
        let jsonString = String(data: jsonData, encoding: .utf8) ?? "[]"
        
        res.setHeader(.contentType, value: "application/json")
        res.appendBody(string: jsonString)
    } catch {
        res.status = .badRequest
        res.appendBody(string: "Erro ao serializar JSON")
    }
    
    res.completed()
}
```

### Decodificar JSON do Request

```swift
server.routes.add(method: .post, uri: "/users") { req, res in
    do {
        guard let bodyString = req.postBodyString,
              let bodyData = bodyString.data(using: .utf8) else {
            res.status = .badRequest
            res.appendBody(string: "Body inválido")
            res.completed()
            return
        }
        
        let user = try JSONDecoder().decode(User.self, from: bodyData)
        
        res.setHeader(.contentType, value: "application/json")
        let response = try JSONEncoder().encode(["success": true, "user": user])
        res.appendBody(data: response)
    } catch {
        res.status = .badRequest
        res.appendBody(string: "Erro: \(error.localizedDescription)")
    }
    
    res.completed()
}
```

## Middleware

### Middleware Customizado

```swift
import PerfectHTTPServer

class LoggingMiddleware: HTTPRequestFilter {
    func filterRequest(_ request: inout HTTPRequest, response: HTTPResponse, 
                     callback: @escaping (Bool) -> Void) {
        let timestamp = Date()
        let method = request.method
        let path = request.path
        
        print("[\(timestamp)] \(method) \(path)")
        
        callback(true)  // Continuar processamento
    }
    
    static var filterPriority: HTTPFilterPriority {
        return .high
    }
}

// Registrar middleware
server.setRequestFilters([LoggingMiddleware()])
```

## Persistência com Perfect ORM

### Configuração Básica

```swift
import PerfectORM
import PerfectPostgreSQL  // ou PerfectMySQL

// Configurar conexão
var connInfo = [
    "host": "localhost",
    "port": "5432",
    "user": "postgres",
    "password": "password",
    "database": "myapp"
]

// Inicializar banco
let database = Database(PostgreSQL(), connInfo: connInfo)
database.open()  // Abre conexão
```

### Definir Modelo

```swift
import PerfectORM

class User: Codable, Persistable {
    var id: Int?
    var name: String = ""
    var email: String = ""
    
    static let tableName = "users"
    
    enum CodingKeys: String, CodingKey {
        case id, name, email
    }
}
```

### CRUD Básico

```swift
// CREATE
var user = User()
user.name = "Alice"
user.email = "alice@example.com"

do {
    try user.create(db: database)
    print("Usuário criado com ID: \(user.id ?? -1)")
} catch {
    print("Erro ao criar: \(error)")
}

// READ
do {
    let users = try User.fetchAll(db: database, where: "name LIKE %@", ["Alice"])
    users.forEach { print($0.name) }
} catch {
    print("Erro ao buscar: \(error)")
}

// UPDATE
user.email = "alice.new@example.com"
do {
    try user.update(db: database)
} catch {
    print("Erro ao atualizar: \(error)")
}

// DELETE
do {
    try user.delete(db: database)
} catch {
    print("Erro ao deletar: \(error)")
}
```

## Exemplo Completo: API REST de Tarefas

```swift
import PerfectHTTPServer
import Foundation

struct Task: Codable {
    let id: Int
    var title: String
    var completed: Bool
}

var tasks: [Task] = [
    Task(id: 1, title: "Aprender Swift", completed: false),
    Task(id: 2, title: "Criar API", completed: true)
]

let server = HTTPServer()
server.serverPort = 8080

// GET /tasks
server.routes.add(method: .get, uri: "/tasks") { req, res in
    do {
        let data = try JSONEncoder().encode(tasks)
        res.setHeader(.contentType, value: "application/json")
        res.appendBody(data: data)
    } catch {
        res.status = .internalServerError
    }
    res.completed()
}

// GET /tasks/{id}
server.routes.add(method: .get, uri: "/tasks/{id}") { req, res in
    guard let idStr = req.pathVariables["id"],
          let id = Int(idStr),
          let task = tasks.first(where: { $0.id == id }) else {
        res.status = .notFound
        res.completed()
        return
    }
    
    do {
        let data = try JSONEncoder().encode(task)
        res.setHeader(.contentType, value: "application/json")
        res.appendBody(data: data)
    } catch {
        res.status = .internalServerError
    }
    res.completed()
}

// POST /tasks
server.routes.add(method: .post, uri: "/tasks") { req, res in
    guard let bodyData = req.postBodyBytes,
          var task = try? JSONDecoder().decode(Task.self, from: Data(bytes: bodyData)) else {
        res.status = .badRequest
        res.completed()
        return
    }
    
    task.id = (tasks.max(by: { $0.id < $1.id })?.id ?? 0) + 1
    tasks.append(task)
    
    do {
        let data = try JSONEncoder().encode(task)
        res.status = .created
        res.setHeader(.contentType, value: "application/json")
        res.appendBody(data: data)
    } catch {
        res.status = .internalServerError
    }
    res.completed()
}

// PUT /tasks/{id}
server.routes.add(method: .put, uri: "/tasks/{id}") { req, res in
    guard let idStr = req.pathVariables["id"],
          let id = Int(idStr),
          let index = tasks.firstIndex(where: { $0.id == id }),
          let bodyData = req.postBodyBytes else {
        res.status = .badRequest
        res.completed()
        return
    }
    
    do {
        var updatedTask = try JSONDecoder().decode(Task.self, from: Data(bytes: bodyData))
        updatedTask.id = id
        tasks[index] = updatedTask
        
        let data = try JSONEncoder().encode(updatedTask)
        res.setHeader(.contentType, value: "application/json")
        res.appendBody(data: data)
    } catch {
        res.status = .internalServerError
    }
    res.completed()
}

// DELETE /tasks/{id}
server.routes.add(method: .delete, uri: "/tasks/{id}") { req, res in
    guard let idStr = req.pathVariables["id"],
          let id = Int(idStr),
          let index = tasks.firstIndex(where: { $0.id == id }) else {
        res.status = .notFound
        res.completed()
        return
    }
    
    tasks.remove(at: index)
    res.status = .noContent
    res.completed()
}

do {
    try server.start()
} catch {
    print("Erro: \(error)")
}
```

## Deployment

### Compile em Release

```bash
swift build -c release
# Executável em: .build/release/MyAPI
```

### Servidor Linux

```bash
# Ubuntu/Debian
sudo apt-get install swift

# Compile
swift build -c release

# Execute
./.build/release/MyAPI
```

## Comparação com Alternativas

| Framework | Manutenção | Facilidade | Performance | Comunidade |
|---|---|---|---|---|
| **Perfect** | Manutenção | ⭐⭐⭐ | ⭐⭐⭐ | Pequena |
| **Vapor** | Ativa | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | Grande |
| **Hummingbird** | Ativa | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | Crescente |
| **Kitura** | Limitada | ⭐⭐⭐ | ⭐⭐⭐ | Pequena |

## Conclusão

Perfect é funcional e estável, mas para **novos projetos**, considere [[Vapor]] ou [[Hummingbird]] por comunidade mais ativa e desenvolvimento contínuo.

**Relacionados:** [[Backend]] [[Vapor]] [[Hummingbird]] [[Kitura]]
