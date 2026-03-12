# Design Patterns em Swift

**Design Patterns** (Padrões de Projeto) são soluções reutilizáveis para problemas comuns na programação. Em Swift, adaptamos padrões clássicos para aproveitar características modernas da linguagem.

## Padrões Principais

## 1. Creational Patterns (Criação)

### Singleton

Garante que uma classe tenha apenas uma instância em toda a aplicação.

```swift
class DatabaseManager {
    static let shared = DatabaseManager()
    
    private init() {}  // Impede inicialização externa
    
    func connect() {
        print("Conectado ao banco de dados")
    }
}

// Uso
DatabaseManager.shared.connect()
```

**Casos de Uso:** Logger, Network Client, UserDefaults manager.

### Factory Pattern

Cria objetos sem especificar suas classes concretas.

```swift
protocol Animal {
    func speak() -> String
}

class Dog: Animal {
    func speak() -> String { "Woof!" }
}

class Cat: Animal {
    func speak() -> String { "Meow!" }
}

class AnimalFactory {
    static func createAnimal(type: String) -> Animal {
        switch type {
        case "dog": return Dog()
        case "cat": return Cat()
        default: return Dog()
        }
    }
}

// Uso
let dog = AnimalFactory.createAnimal(type: "dog")
```

**Casos de Uso:** API Response handling, View creation, Service initialization.

### Builder Pattern

Constrói objetos complexos passo a passo.

```swift
struct User {
    let name: String
    let email: String
    let age: Int?
    let phone: String?
}

class UserBuilder {
    private var name: String = ""
    private var email: String = ""
    private var age: Int?
    private var phone: String?
    
    func setName(_ name: String) -> UserBuilder {
        self.name = name
        return self
    }
    
    func setEmail(_ email: String) -> UserBuilder {
        self.email = email
        return self
    }
    
    func setAge(_ age: Int) -> UserBuilder {
        self.age = age
        return self
    }
    
    func setPhone(_ phone: String) -> UserBuilder {
        self.phone = phone
        return self
    }
    
    func build() -> User {
        User(name: name, email: email, age: age, phone: phone)
    }
}

// Uso (Method Chaining)
let user = UserBuilder()
    .setName("João")
    .setEmail("joao@example.com")
    .setAge(30)
    .build()
```

## 2. Structural Patterns (Estrutura)

### Adapter Pattern

Compatibiliza interfaces diferentes.

```swift
// Interface antiga
protocol OldApiDelegate {
    func didReceiveData(_ json: String)
}

// Interface nova
protocol NewApiDelegate {
    func didReceive(model: [String: Any])
}

// Adapter
class ApiAdapter: OldApiDelegate {
    var newDelegate: NewApiDelegate?
    
    func didReceiveData(_ json: String) {
        // Converte JSON string para dicionário
        if let data = json.data(using: .utf8),
           let dict = try? JSONSerialization.jsonObject(with: data) as? [String: Any] {
            newDelegate?.didReceive(model: dict)
        }
    }
}
```

### Decorator Pattern

Adiciona funcionalidades a objetos dinamicamente.

```swift
protocol Component {
    func operation() -> String
}

class ConcreteComponent: Component {
    func operation() -> String { "ConcreteComponent" }
}

class Decorator: Component {
    let wrappedComponent: Component
    
    init(_ component: Component) {
        self.wrappedComponent = component
    }
    
    func operation() -> String { wrappedComponent.operation() }
}

class ConcreteDecoratorA: Decorator {
    override func operation() -> String {
        "DecoratorA(" + wrappedComponent.operation() + ")"
    }
}

class ConcreteDecoratorB: Decorator {
    override func operation() -> String {
        "DecoratorB(" + wrappedComponent.operation() + ")"
    }
}

// Uso
let component = ConcreteComponent()
let decorated = ConcreteDecoratorA(ConcreteDecoratorB(component))
print(decorated.operation())  // DecoratorA(DecoratorB(ConcreteComponent))
```

### Facade Pattern

Oferece interface simplificada para subsistema complexo.

```swift
// Subsistemas complexos
class PaymentProcessor {
    func process(amount: Double) -> Bool { true }
}

class ShippingService {
    func calculateShipping(address: String) -> Double { 10.0 }
}

class InventoryManager {
    func reserveItems(_ items: [String]) -> Bool { true }
}

// Facade
class OrderFacade {
    let paymentProcessor = PaymentProcessor()
    let shippingService = ShippingService()
    let inventoryManager = InventoryManager()
    
    func placeOrder(items: [String], address: String, amount: Double) -> Bool {
        guard inventoryManager.reserveItems(items) else { return false }
        guard paymentProcessor.process(amount: amount) else { return false }
        let shipping = shippingService.calculateShipping(address: address)
        print("Pedido realizado! Frete: \(shipping)")
        return true
    }
}

// Uso (simples!)
let order = OrderFacade()
order.placeOrder(items: ["Camera"], address: "Rua A, 123", amount: 500)
```

## 3. Behavioral Patterns (Comportamento)

### Observer Pattern

Notifica múltiplos objetos sobre mudanças de estado.

```swift
protocol Observer: AnyObject {
    func update(_ value: Int)
}

class Subject {
    private var observers: [Observer] = []
    private var value: Int = 0
    
    func attach(_ observer: Observer) {
        observers.append(observer)
    }
    
    func detach(_ observer: Observer) {
        observers.removeAll { $0 === observer }
    }
    
    func setValue(_ newValue: Int) {
        value = newValue
        notifyObservers()
    }
    
    private func notifyObservers() {
        observers.forEach { $0.update(value) }
    }
}

class ConcreteObserver: Observer {
    let id: Int
    
    init(_ id: Int) { self.id = id }
    
    func update(_ value: Int) {
        print("Observer \(id) recebeu: \(value)")
    }
}

// Uso
let subject = Subject()
let obs1 = ConcreteObserver(1)
let obs2 = ConcreteObserver(2)
subject.attach(obs1)
subject.attach(obs2)
subject.setValue(42)  // Ambos são notificados
```

**Em Swift Moderno:** Use `@Published` com Combine ou `@ObservedObject` em SwiftUI.

### Strategy Pattern

Define família de algoritmos intercambiáveis.

```swift
protocol PaymentStrategy {
    func pay(amount: Double)
}

class CreditCardPayment: PaymentStrategy {
    func pay(amount: Double) {
        print("Pagando \(amount) com cartão de crédito")
    }
}

class PayPalPayment: PaymentStrategy {
    func pay(amount: Double) {
        print("Pagando \(amount) via PayPal")
    }
}

class CryptoPayment: PaymentStrategy {
    func pay(amount: Double) {
        print("Pagando \(amount) em criptomoedas")
    }
}

class PaymentProcessor {
    var strategy: PaymentStrategy
    
    init(_ strategy: PaymentStrategy) {
        self.strategy = strategy
    }
    
    func processPayment(amount: Double) {
        strategy.pay(amount: amount)
    }
}

// Uso
let processor = PaymentProcessor(CreditCardPayment())
processor.processPayment(amount: 100)

processor.strategy = PayPalPayment()  // Trocar estratégia
processor.processPayment(amount: 50)
```

### State Pattern

Permite objeto alterar comportamento conforme estado.

```swift
protocol OrderState {
    func next(context: Order)
    func description() -> String
}

class PendingState: OrderState {
    func next(context: Order) {
        print("Processando pagamento...")
        context.state = ProcessingState()
    }
    
    func description() -> String { "Pendente" }
}

class ProcessingState: OrderState {
    func next(context: Order) {
        print("Preparando envio...")
        context.state = ShippedState()
    }
    
    func description() -> String { "Processando" }
}

class ShippedState: OrderState {
    func next(context: Order) {
        print("Pedido entregue!")
        context.state = DeliveredState()
    }
    
    func description() -> String { "Enviado" }
}

class DeliveredState: OrderState {
    func next(context: Order) {
        print("Pedido já foi entregue")
    }
    
    func description() -> String { "Entregue" }
}

class Order {
    var state: OrderState = PendingState()
    
    func nextStep() {
        state.next(context: self)
    }
    
    func status() -> String {
        state.description()
    }
}

// Uso
let order = Order()
print(order.status())  // Pendente
order.nextStep()  // Processando pagamento...
print(order.status())  // Processando
order.nextStep()  // Preparando envio...
print(order.status())  // Enviado
```

### Command Pattern

Encapsula pedido como objeto.

```swift
protocol Command {
    func execute()
    func undo()
}

class Light {
    var isOn = false
    
    func on() {
        isOn = true
        print("Luz ligada")
    }
    
    func off() {
        isOn = false
        print("Luz desligada")
    }
}

class LightOnCommand: Command {
    let light: Light
    
    init(_ light: Light) { self.light = light }
    
    func execute() { light.on() }
    func undo() { light.off() }
}

class RemoteControl {
    var commands: [Command] = []
    
    func executeCommand(_ command: Command) {
        command.execute()
        commands.append(command)
    }
    
    func undoLastCommand() {
        guard !commands.isEmpty else { return }
        commands.removeLast().undo()
    }
}

// Uso
let light = Light()
let remote = RemoteControl()
remote.executeCommand(LightOnCommand(light))
remote.undoLastCommand()
```

## Padrões Swift-Específicos

### Delegate Pattern

Muito usado em Swift/iOS (UITableViewDelegate, etc).

```swift
protocol DataSourceDelegate: AnyObject {
    func didLoadData(_ data: [String])
}

class DataManager {
    weak var delegate: DataSourceDelegate?
    
    func fetchData() {
        let data = ["Item 1", "Item 2", "Item 3"]
        delegate?.didLoadData(data)
    }
}

class ViewController: DataSourceDelegate {
    let dataManager = DataManager()
    
    func viewDidLoad() {
        dataManager.delegate = self
        dataManager.fetchData()
    }
    
    func didLoadData(_ data: [String]) {
        print("Dados recebidos: \(data)")
    }
}
```

## Quando Usar Cada Padrão

| Padrão | Quando Usar | Exemplos |
|---|---|---|
| Singleton | Um único ponto de acesso | Logger, Database, UserDefaults |
| Factory | Criar objetos sem conhecer classe | API responses, Views |
| Builder | Objetos complexos | User config, Request building |
| Observer | Notificar múltiplos interessados | Event systems, Binding |
| Strategy | Múltiplos algoritmos | Payment methods, Sorting |
| State | Comportamento muda com estado | Order workflow, State machines |
| Decorator | Adicionar responsabilidades | Middleware, Wrappers |
| Facade | Simplificar interface | Realmente qualquer subsistema |

## Recursos Adicionais

- [Refactoring Guru - Design Patterns em Swift](https://refactoring.guru/pt-br/design-patterns/swift)
- [GitHub: Design Patterns in Swift](https://github.com/ochococo/Design-Patterns-In-Swift)
- [iOS Good Practices](https://github.com/futurice/ios-good-practices)

**Relacionados:** [[Arquitetura de Software]] [[Orientação a Objetos]]
