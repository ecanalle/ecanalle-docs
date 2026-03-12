# Vapor em Swift

**Vapor** é um framework web popular, rápido e extensível escrito em Swift. Ele permite que desenvolvedores Swift construam aplicações backend robustas, APIs e microsserviços utilizando a mesma linguagem que eles usam para desenvolver aplicativos para as plataformas da Apple (iOS, macOS, etc.). Vapor adota muitos dos melhores padrões e práticas de outros frameworks web modernos, tornando o desenvolvimento backend em Swift eficiente e agradável.

### Principais Características do Vapor

* **Fácil de usar:** Vapor oferece uma API expressiva e uma sintaxe clara, facilitando o início e o desenvolvimento de aplicações web.
* **Rápido e eficiente:** Vapor é construído com foco em performance, utilizando o framework de rede NIO da Apple para fornecer alta velocidade e escalabilidade.
* **Extensível:** Vapor possui um sistema de pacotes robusto (Vapor Toolbox e Vapor Fluent) que permite adicionar facilmente funcionalidades como ORM (Object-Relational Mapping), autenticação, templates de visualização e muito mais.
* **Seguro:** Vapor incentiva boas práticas de segurança e oferece recursos para ajudar a proteger suas aplicações contra vulnerabilidades comuns.
* **Comunidade ativa:** Vapor possui uma comunidade de desenvolvedores engajada e uma vasta documentação, facilitando a obtenimento de ajuda e recursos de aprendizado.
* **Multiplataforma:** Embora seja escrito em Swift, Vapor pode ser executado em diversas plataformas, incluindo macOS e Linux, tornando-o adequado para implantações em servidores.

### Estrutura Básica de um Projeto Vapor

Um projeto Vapor geralmente segue uma estrutura organizada que inclui:

* **`Sources/App`**: Contém o código principal da sua aplicação, incluindo definições de rotas, controladores, modelos e middleware.
* **`Sources/Run`**: Contém o ponto de entrada principal para executar a aplicação.
* **`Package.swift`**: O arquivo de manifesto do Swift Package Manager, que descreve as dependências do projeto.
* **`Public`**: Diretório para arquivos estáticos como CSS, JavaScript e imagens.
* **`Resources`**: Diretório para outros recursos como templates de visualização e arquivos de configuração.
* **`Tests`**: Contém os testes unitários e de integração para a sua aplicação.

### Definição de Rotas

Em Vapor, você define as rotas da sua aplicação (os URLs que ela responde) dentro de uma função, geralmente chamada `routes(_:)`.

```swift
import Vapor

func routes(_ app: Application) throws {
    app.get("hello") { req in
        return "Hello, Vapor!"
    }

    app.post("data") { req -> String in
        struct Input: Content {
            let message: String
        }
        let input = try req.content.decode(Input.self)
        return "Você enviou: \(input.message)"
    }
}
````

- `app.get("hello") { req in ... }` define uma rota GET no caminho `/hello`. Quando um cliente faz uma requisição GET para este URL, o closure é executado, e neste caso, retorna a string "Hello, Vapor!".
- `app.post("data") { req -> String in ... }` define uma rota POST no caminho `/data`. Ele demonstra como decodificar dados JSON do corpo da requisição (`req.content.decode(Input.self)`) e retornar uma resposta baseada nesses dados. A struct `Input` conforma ao protocolo `Content` do Vapor, facilitando a codificação e decodificação de dados.

### Controladores

Para organizar melhor o código das suas rotas, é comum usar **controladores**. Um controlador é uma classe que contém métodos (action handlers) para lidar com requisições para rotas específicas.

```swift
import Vapor

struct HelloController: RouteCollection {
    func boot(routes: RoutesBuilder) throws {
        routes.get("greet", use: greet)
        routes.get("greet", ":name", use: greetName)
    }

    func greet(_ req: Request) throws -> String {
        return "Olá do Controlador!"
    }

    func greetName(_ req: Request) throws -> String {
        guard let name = req.parameters.get("name", as: String.self) else {
            throw Abort(.badRequest)
        }
        return "Olá, \(name) do Controlador!"
    }
}

func routes(_ app: Application) throws {
    try app.register(collection: HelloController())
}
```

- A struct `HelloController` conforma ao protocolo `RouteCollection`.
- O método `boot(routes:)` é usado para definir as rotas associadas a este controlador.
- `routes.get("greet", use: greet)` associa a rota GET `/greet` ao método `greet`.
- `routes.get("greet", ":name", use: greetName)` define uma rota GET `/greet/:name`, onde `:name` é um parâmetro dinâmico que pode ser acessado através de `req.parameters`.

### Modelo (Model) e ORM (Fluent)

Para interagir com bancos de dados, o Vapor frequentemente utiliza o framework **Fluent**, que é um ORM desenvolvido pela mesma equipe do Vapor. Fluent permite definir modelos Swift que correspondem a tabelas no seu banco de dados e fornece uma API para realizar operações de leitura e escrita de forma segura e concisa.

```swift
import Vapor
import Fluent

final class Todo: Model, Content {
    static let schema = "todos"

    @ID(key: .id)
    var id: UUID?

    @Field(key: "title")
    var title: String

    init() { }

    init(id: UUID? = nil, title: String) {
        self.id = id
        self.title = title
    }
}

func routes(_ app: Application) throws {
    app.get("todos") { req async throws -> [Todo] in
        return try await Todo.query(on: req.db).all()
    }

    app.post("todos") { req async throws -> Todo in
        let todo = try req.content.decode(Todo.self)
        try await todo.save(on: req.db)
        return todo
    }
}
```

- A classe `Todo` conforma aos protocolos `Model` (do Fluent) e `Content` (do Vapor).
- `@ID` define a chave primária da tabela.
- `@Field` define um campo na tabela.
- As rotas demonstram como buscar todos os "todos" do banco de dados e como criar e salvar um novo "todo" a partir dos dados da requisição.

### Middleware

**Middleware** são componentes que interceptam as requisições HTTP de entrada e as respostas HTTP de saída. Eles podem realizar tarefas como autenticação, logging, compressão e modificação de headers. Vapor possui um sistema de middleware flexível que permite adicionar e configurar middleware globalmente ou para rotas específicas.

### Templates de Visualização (Leaf)

Para gerar HTML dinamicamente, Vapor geralmente usa o template engine **Leaf**. Leaf permite criar arquivos de template com sintaxe simples para exibir dados da sua aplicação.

### Benefícios de Usar Vapor

- **Desenvolvimento Full-Stack em Swift:** Permite usar a mesma linguagem tanto no frontend (com SwiftUI/UIKit) quanto no backend.
- **Performance:** Ideal para aplicações que exigem alta performance e escalabilidade.
- **Ecossistema Rico:** O Vapor Toolbox e o Fluent, juntamente com outros pacotes da comunidade, fornecem muitas funcionalidades prontas para uso.
- **Ótima Integração com o Ecossistema Apple:** Facilita a criação de backends para aplicativos iOS, macOS, etc.

[Link para a documentação oficial do Vapor](https://docs.vapor.codes/)

---

**Observações:**

- Vapor é uma excelente escolha para desenvolvedores Swift que desejam construir aplicações backend eficientes e escaláveis.
- Aprender os conceitos de rotas, controladores, modelos (com Fluent) e middleware é fundamental para desenvolver com Vapor.
- A comunidade Vapor é acolhedora e oferece muitos recursos para ajudar novos desenvolvedores.
