# Kitura em Swift

**Kitura** é um framework web para Server-Side Swift, originalmente desenvolvido pela IBM em 2016. Embora tenha tido desenvolvimento limitado nos últimos anos, permanece funcional e estável para aplicações web e APIs.

## Histórico

* **2016:** Lançado pela IBM como framework Swift para web
* **2019:** IBM parou desenvolvimento ativo
* **Atual:** Mantido pela comunidade como projeto open-source

## Características

* **Maduro:** Está em uso desde 2016
* **Simples:** API direta e fácil de entender
* **Multiplataforma:** Funciona em macOS, Linux e outros sistemas
* **Extensível:** Suporte para middleware e extensões
* **ORM Integrado:** Inclui ferramentas de persistência

## Instalação

### Package.swift

```swift
import PackageDescription

let package = Package(
    name: "MyKituraAPI",
    dependencies: [
        .package(url: "https://github.com/IBM-Swift/Kitura.git", from: "2.9.0"),
    ],
    targets: [
        .executableTarget(
            name: "MyKituraAPI",
            dependencies: [
                .product(name: "Kitura", package: "Kitura"),
            ]
        )
    ]
)
```

## Servidor Básico

```swift
import Kitura
import Foundation

let router = Router()

// Rota simples
router.get("/") { request, response, next in
    response.send("Hello from Kitura!")
    next()
}

// Iniciar servidor
Kitura.addRoutes(router)
Kitura.start()
```

**Executar:**
```bash
swift run
# Servidor em http://localhost:8080
```

## Roteamento

### Rotas HTTP

```swift
import Kitura

let router = Router()

// GET
router.get("/hello") { request, response, next in
    response.send("Hello!")
    next()
}

// POST
router.post("/users") { request, response, next in
    response.send("Usuário criado")
    next()
}

// PUT
router.put("/users/:id") { request, response, next in
    let userId = request.parameters["id"] ?? "unknown"
    response.send("Atualizando usuário \(userId)")
    next()
}

// DELETE
router.delete("/users/:id") { request, response, next in
    let userId = request.parameters["id"] ?? "unknown"
    response.send("Usuário \(userId) deletado")
    next()
}
```

### Parâmetros de Rota

```swift
router.get("/users/:id/posts/:postId") { request, response, next in
    let userId = request.parameters["id"] ?? ""
    let postId = request.parameters["postId"] ?? ""
    
    response.send("Usuário: \(userId), Post: \(postId)")
    next()
}

// Uso: GET /users/123/posts/456
```

### Query Parameters

```swift
router.get("/search") { request, response, next in
    let query = request.queryParameters["q"] ?? "default"
    let limit = request.queryParameters["limit"] ?? "10"
    
    response.send("Buscando: \(query), Limite: \(limit)")
    next()
}

// Uso: GET /search?q=swift&limit=20
```

## JSON

### Trabalhar com JSON

```swift
import Kitura
import Foundation

struct User: Codable {
    let id: Int
    let name: String
    let email: String
}

let router = Router()

// Retornar JSON
router.get("/users") { request, response, next in
    let users = [
        User(id: 1, name: "Alice", email: "alice@example.com"),
        User(id: 2, name: "Bob", email: "bob@example.com")
    ]
    
    response.send(json: users)
    next()
}

// Decodificar JSON do request
router.post("/users") { request, response, next in
    let decoder = JSONDecoder()
    
    guard let body = request.body?.asJSON as? [String: Any],
          let name = body["name"] as? String,
          let email = body["email"] as? String else {
        response.status(.badRequest)
        return next()
    }
    
    let newUser = User(id: 3, name: name, email: email)
    response.send(json: newUser)
    next()
}
```

## Middleware

### Middleware Customizado

```swift
import Kitura

class LoggingMiddleware: RouterMiddleware {
    func handle(request: RouterRequest, response: RouterResponse, next: @escaping () -> Void) {
        let timestamp = Date()
        let method = request.method.rawValue
        let path = request.path
        
        print("[\(timestamp)] \(method) \(path)")
        
        next()
    }
}

let router = Router()
router.all(middleware: LoggingMiddleware())

// Todas as rotas usarão o middleware
```

### Middleware para Autenticação

```swift
class AuthMiddleware: RouterMiddleware {
    func handle(request: RouterRequest, response: RouterResponse, next: @escaping () -> Void) {
        guard let authHeader = request.headers["Authorization"] else {
            response.status(.unauthorized)
            response.send("Não autorizado")
            return
        }
        
        if authHeader.hasPrefix("Bearer ") {
            next()
        } else {
            response.status(.unauthorized)
            response.send("Token inválido")
        }
    }
}

let router = Router()
router.all("/protected", middleware: AuthMiddleware())

router.get("/protected/data") { request, response, next in
    response.send("Dados protegidos!")
    next()
}
```

## Exemplo Completo: API de Tarefas

```swift
import Kitura
import Foundation

struct Task: Codable {
    var id: Int
    var title: String
    var completed: Bool = false
}

var tasks: [Task] = [
    Task(id: 1, title: "Aprender Kitura", completed: false),
    Task(id: 2, title: "Criar API REST", completed: true)
]

let router = Router()

// GET /tasks
router.get("/tasks") { _, response, next in
    response.send(json: tasks)
    next()
}

// GET /tasks/{id}
router.get("/tasks/:id") { request, response, next in
    guard let idStr = request.parameters["id"],
          let id = Int(idStr),
          let task = tasks.first(where: { $0.id == id }) else {
        response.status(.notFound)
        return next()
    }
    
    response.send(json: task)
    next()
}

// POST /tasks
router.post("/tasks") { request, response, next in
    var newTask = Task(id: 0, title: "", completed: false)
    
    do {
        guard let body = request.body?.asJSON as? [String: Any],
              let title = body["title"] as? String else {
            response.status(.badRequest)
            return next()
        }
        
        newTask.id = (tasks.max(by: { $0.id < $1.id })?.id ?? 0) + 1
        newTask.title = title
        tasks.append(newTask)
        
        response.status(.created)
        response.send(json: newTask)
    }
    
    next()
}

// PUT /tasks/{id}
router.put("/tasks/:id") { request, response, next in
    guard let idStr = request.parameters["id"],
          let id = Int(idStr),
          let index = tasks.firstIndex(where: { $0.id == id }) else {
        response.status(.notFound)
        return next()
    }
    
    guard let body = request.body?.asJSON as? [String: Any],
          let title = body["title"] as? String else {
        response.status(.badRequest)
        return next()
    }
    
    tasks[index].title = title
    if let completed = body["completed"] as? Bool {
        tasks[index].completed = completed
    }
    
    response.send(json: tasks[index])
    next()
}

// DELETE /tasks/{id}
router.delete("/tasks/:id") { request, response, next in
    guard let idStr = request.parameters["id"],
          let id = Int(idStr),
          let index = tasks.firstIndex(where: { $0.id == id }) else {
        response.status(.notFound)
        return next()
    }
    
    tasks.remove(at: index)
    response.status(.noContent)
    next()
}

Kitura.addRoutes(router)
Kitura.start()
```

## Persistência

### Conexão com PostgreSQL

```swift
import Kitura
import CloudFoundryEnv
import Foundation

// Configurar conexão
let env = CloudFoundryEnv()

do {
    let dbUrl = try env.getURL(name: "kitura-database")
    // Usar dbUrl para conectar ao banco
} catch {
    print("Erro ao configurar banco: \(error)")
}
```

## Deployment

### Executar em Release

```bash
swift build -c release
./.build/release/MyKituraAPI
```

### Cloud Foundry

Kitura foi originalmente projetado para Cloud Foundry:

```bash
cf push myapp
```

## Status Atual

Kitura ainda é funcional, mas:
- ✅ Estável para projetos existentes
- ✅ Comunidade pequena mas dedicada
- ⚠️ Desenvolvimento ativo limitado
- ❌ Não recomendado para novos projetos

## Comparação com Alternativas

| Aspecto | Kitura | Vapor | Hummingbird |
|---|---|---|---|
| Performance | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Facilidade | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Comunidade | Pequena | Grande | Crescente |
| Manutenção | Limitada | Ativa | Ativa |
| Async/Await | Parcial | Completo | Nativo |

## Conclusão

**Use Kitura se:**
- Manter projeto existente em Kitura
- Preferir API simples
- Trabalhar com infraestrutura Cloud Foundry

**Use Vapor ou Hummingbird para novos projetos** devido a:
- Comunidade mais ativa
- Desenvolvimento contínuo
- Performance superior
- Melhor suporte a async/await

## Recursos

- [GitHub: IBM-Swift/Kitura](https://github.com/IBM-Swift/Kitura)
- [Documentação](https://kitura.dev/)
- [Exemplos](https://github.com/IBM-Swift/Kitura-Sample)

**Relacionados:** [Backend](Backend.md) [Vapor](Vapor.md) [Hummingbird](Hummingbird.md) [Perfect](Perfect.md)
