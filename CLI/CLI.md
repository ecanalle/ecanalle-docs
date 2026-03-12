# CLI (Command Line Interface) em Swift

Swift pode ser usado para criar **ferramentas de linha de comando** poderosas e eficientes, aproveitando sua sintaxe clara e tipo-segurança.

## Configuração Inicial do Projeto

### Criar Projeto CLI

```bash
# Usando Swift Package Manager
swift package init --type executable --name MyCLI

# Estrutura gerada:
# MyCLI/
# ├── Package.swift
# ├── Sources/
# │   └── MyCLI/
# │       └── main.swift
# └── Tests/
```

### Package.swift Básico

```swift
import PackageDescription

let package = Package(
    name: "MyCLI",
    dependencies: [
        .package(url: "https://github.com/apple/swift-argument-parser.git", from: "1.2.0"),
    ],
    targets: [
        .executableTarget(
            name: "MyCLI",
            dependencies: [
                .product(name: "ArgumentParser", package: "swift-argument-parser")
            ]
        )
    ]
)
```

## Argumentos e Flags Básicos

### Sem Dependências Externas

```swift
import Foundation

// main.swift
let arguments = CommandLine.arguments
// arguments[0] é sempre o nome do programa
// arguments[1...] são os argumentos

if arguments.count > 1 {
    print("Você passou: \(arguments[1])")
} else {
    print("Uso: myapp <argumento>")
}
```

### Exemplo com Flag

```swift
import Foundation

let args = CommandLine.arguments
var verbose = false
var filename = ""

var i = 1
while i < args.count {
    let arg = args[i]
    
    if arg == "-v" || arg == "--verbose" {
        verbose = true
    } else if arg == "-f" || arg == "--file" {
        i += 1
        if i < args.count {
            filename = args[i]
        }
    }
    
    i += 1
}

if verbose {
    print("Modo verbose ativo")
}
if !filename.isEmpty {
    print("Arquivo: \(filename)")
}
```

## ArgumentParser (Recomendado)

Framework oficial da Apple para CLI parsing, muito mais elegante:

### Estrutura Básica

```swift
import Foundation
import ArgumentParser

struct HelloCLI: ParsableCommand {
    @Argument(help: "A person to greet")
    var name: String
    
    @Option(name: .shortAndLong, help: "Number of greetings")
    var count: Int = 1
    
    @Flag(name: .shortAndLong, help: "Use uppercase")
    var uppercase: Bool = false
    
    mutating func run() throws {
        var greeting = "Hello, \(name)!"
        if uppercase {
            greeting = greeting.uppercased()
        }
        
        for _ in 0..<count {
            print(greeting)
        }
    }
}

HelloCLI.main()
```

**Uso:**
```bash
swift run HelloCLI Alice
swift run HelloCLI Alice --count 3
swift run HelloCLI Alice -c 3
swift run HelloCLI Alice -u  # uppercase
swift run HelloCLI Alice --uppercase
```

### Subcomandos

```swift
import ArgumentParser

struct MyCLI: ParsableCommand {
    static var configuration = CommandConfiguration(
        abstract: "Uma ferramenta versátil",
        subcommands: [Create.self, Delete.self, List.self],
        defaultSubcommand: List.self
    )
}

struct Create: ParsableCommand {
    static var configuration = CommandConfiguration(
        abstract: "Cria um novo item"
    )
    
    @Argument var name: String
    
    mutating func run() throws {
        print("Criando: \(name)")
    }
}

struct Delete: ParsableCommand {
    static var configuration = CommandConfiguration(
        abstract: "Deleta um item"
    )
    
    @Argument var id: Int
    
    mutating func run() throws {
        print("Deletando item ID: \(id)")
    }
}

struct List: ParsableCommand {
    static var configuration = CommandConfiguration(
        abstract: "Lista todos os itens"
    )
    
    @Flag(name: .shortAndLong, help: "Formato detalhado")
    var detailed: Bool = false
    
    mutating func run() throws {
        if detailed {
            print("Listando itens em formato detalhado...")
        } else {
            print("Listando itens...")
        }
    }
}

MyCLI.main()
```

**Uso:**
```bash
swift run MyCLI create "New Item"
swift run MyCLI delete 5
swift run MyCLI list --detailed
```

## Entrada/Saída (I/O)

### Ler da Entrada Padrão

```swift
import Foundation

print("Digite seu nome: ", terminator: "")
fflush(stdout)

if let name = readLine() {
    print("Olá, \(name)!")
}
```

### Cores e Formatação

```swift
// Usando ANSI codes para cores
enum ANSIColor {
    static let reset = "\u{001B}[0m"
    static let red = "\u{001B}[31m"
    static let green = "\u{001B}[32m"
    static let yellow = "\u{001B}[33m"
    static let blue = "\u{001B}[34m"
    static let bold = "\u{001B}[1m"
}

print("\(ANSIColor.green)Sucesso!\(ANSIColor.reset)")
print("\(ANSIColor.red)\(ANSIColor.bold)Erro!\(ANSIColor.reset)")
```

## Executar Comandos do Sistema

```swift
import Foundation

// Execução simples
func runCommand(_ command: String, args: [String] = []) throws -> String {
    let process = Process()
    process.executableURL = URL(fileURLWithPath: "/usr/bin/env")
    process.arguments = [command] + args
    
    let pipe = Pipe()
    process.standardOutput = pipe
    try process.run()
    process.waitUntilExit()
    
    let data = pipe.fileHandleForReading.readDataToEndOfFile()
    return String(data: data, encoding: .utf8) ?? ""
}

// Uso
do {
    let output = try runCommand("ls", args: ["-la"])
    print(output)
} catch {
    print("Erro: \(error)")
}
```

## Estrutura de Projeto CLI Professional

```
MyCLI/
├── Package.swift
├── Sources/
│   └── MyCLI/
│       ├── main.swift              # Ponto de entrada
│       ├── Commands/
│       │   ├── CreateCommand.swift
│       │   ├── DeleteCommand.swift
│       │   └── ListCommand.swift
│       ├── Utilities/
│       │   ├── Logger.swift
│       │   ├── FileManager+Ext.swift
│       │   └── Colors.swift
│       └── Models/
│           └── Item.swift
├── Tests/
│   └── MyCLITests/
│       └── CommandTests.swift
└── README.md
```

## Exemplo Completo: Gerenciador de Tarefas

```swift
import Foundation
import ArgumentParser

// MARK: - Model
struct Task: Codable {
    let id: UUID
    var title: String
    var completed: Bool = false
}

// MARK: - Storage
class TaskStorage {
    private let filePath = FileManager.default.homeDirectoryForCurrentUser
        .appendingPathComponent(".taskscli/tasks.json")
    
    func loadTasks() -> [Task] {
        guard let data = try? Data(contentsOf: filePath),
              let tasks = try? JSONDecoder().decode([Task].self, from: data) else {
            return []
        }
        return tasks
    }
    
    func saveTasks(_ tasks: [Task]) throws {
        try FileManager.default.createDirectory(at: filePath.deletingLastPathComponent(), 
                                               withIntermediateDirectories: true)
        let data = try JSONEncoder().encode(tasks)
        try data.write(to: filePath)
    }
}

// MARK: - Commands
struct TasksCLI: ParsableCommand {
    static var configuration = CommandConfiguration(
        abstract: "Gerenciador de tarefas via CLI",
        subcommands: [Add.self, List.self, Complete.self],
        defaultSubcommand: List.self
    )
}

struct Add: ParsableCommand {
    @Argument var title: String
    
    mutating func run() throws {
        var storage = TaskStorage()
        var tasks = storage.loadTasks()
        
        let newTask = Task(id: UUID(), title: title)
        tasks.append(newTask)
        
        try storage.saveTasks(tasks)
        print("✅ Tarefa adicionada: \(title)")
    }
}

struct List: ParsableCommand {
    @Flag(name: .shortAndLong) var onlyCompleted: Bool = false
    
    mutating func run() throws {
        let storage = TaskStorage()
        let tasks = storage.loadTasks()
        
        let filtered = onlyCompleted ? tasks.filter { $0.completed } : tasks
        
        if filtered.isEmpty {
            print("Nenhuma tarefa encontrada")
            return
        }
        
        for task in filtered {
            let status = task.completed ? "✅" : "○"
            print("\(status) \(task.title)")
        }
    }
}

struct Complete: ParsableCommand {
    @Argument var index: Int
    
    mutating func run() throws {
        var storage = TaskStorage()
        var tasks = storage.loadTasks()
        
        guard index > 0 && index <= tasks.count else {
            print("❌ Índice inválido")
            return
        }
        
        tasks[index - 1].completed = true
        try storage.saveTasks(tasks)
        print("✅ Tarefa marcada como concluída")
    }
}

TasksCLI.main()
```

## Build e Distribuição

### Compilar Release

```bash
swift build -c release
# Executável em: .build/release/MyCLI
```

### Instalar Globalmente

```bash
# Copiar para /usr/local/bin
cp .build/release/MyCLI /usr/local/bin/

# Agora pode usar de qualquer lugar
mycli --help
```

## Melhores Práticas

1. **Trate Erros com Graciosidade:** Use `try-catch` e forneça mensagens úteis
2. **Oferça Help Claro:** `-h` e `--help` devem ser automáticos
3. **Use Cores com Moderação:** Para importantes (erros, sucessos)
4. **Validação de Entrada:** Sempre valide argumentos
5. **Teste Seus Comandos:** Crie testes para subcomandos
6. **Documentação:** README com exemplos de uso
7. **Controle de Versão:** Adicione versão ao comando `--version`

## Recursos

- [Apple: ArgumentParser](https://github.com/apple/swift-argument-parser)
- [Swift.org: Building for Server-Side Swift](https://www.swift.org/server/)
- [Vapor CLI Tools](https://docs.vapor.codes/basics/commands/)

**Relacionados:** [[Backend]] [[Swift Avançado]]
