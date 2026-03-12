# Protocolos em Swift

**Protocolos** em Swift são como um contrato que define um blueprint de métodos, propriedades e outros requisitos que um tipo específico deve fornecer. Protocolos em si não implementam nenhuma funcionalidade, eles apenas descrevem o que a conformidade a um protocolo significa. Classes, structs e enums podem adotar protocolos para fornecer uma implementação concreta dos requisitos definidos pelo protocolo.

### Sintaxe de Protocolo

Protocolos são definidos de maneira muito semelhante a classes, structs e enums:

```swift
protocol NomeDoProtocolo {
    // Definições de propriedades e métodos
}
````

### Requisitos de Propriedade

Protocolos podem especificar que tipos conformes devem fornecer propriedades de instância ou de tipo com um nome e tipo específicos. O protocolo especifica se cada propriedade deve ser `gettable` (somente leitura) ou `gettable` e `settable` (leitura e escrita).

```swift
protocol Identificavel {
    var id: String { get }
}

protocol Nomeavel {
    var nome: String { get set }
}
```

- `Identificavel` requer que qualquer tipo conforme tenha uma propriedade de instância chamada `id` que seja `gettable`.
- `Nomeavel` requer que qualquer tipo conforme tenha uma propriedade de instância chamada `nome` que seja `gettable` e `settable`.

Tipos conformes devem implementar essas propriedades como propriedades armazenadas variáveis (se `settable`) ou propriedades computadas (se `gettable`, podendo ser implementadas como armazenadas se também forem `settable`).

```swift
struct Usuario: Identificavel, Nomeavel {
    var id: String
    var nome: String
}

class Produto: Identificavel {
    let id: String
    var nome: String

    init(id: String, nome: String) {
        self.id = id
        self.nome = nome
    }
}
```

### Requisitos de Método

Protocolos podem declarar métodos de instância e de tipo que os tipos conformes devem implementar. A sintaxe é quase idêntica à declaração de funções normais, mas sem o corpo.

```swift
protocol Acionavel {
    func ligar()
    func desligar()
}

protocol Ajustavel {
    mutating func ajustar(para valor: Double)
}
```

- `Acionavel` requer métodos `ligar()` e `desligar()`.
- `Ajustavel` requer um método `ajustar(para:)`. A palavra-chave `mutating` indica que um struct ou enum que conforma a este protocolo pode precisar modificar suas propriedades internas ao implementar este método.

```swift
class Interruptor: Acionavel {
    var isOn = false
    func ligar() {
        isOn = true
        print("Interruptor ligado")
    }
    func desligar() {
        isOn = false
        print("Interruptor desligado")
    }
}

struct Volume: Ajustavel {
    var nivel: Double = 0.5
    mutating func ajustar(para valor: Double) {
        nivel = max(0.0, min(1.0, valor))
        print("Volume ajustado para \(nivel)")
    }
}
```

### Requisitos de Inicializador

Protocolos podem especificar inicializadores que os tipos conformes devem implementar.

```swift
protocol InicializavelComNome {
    init(nome: String)
}

class ClasseComNome: InicializavelComNome {
    var nome: String
    required init(nome: String) {
        self.nome = nome
    }
}

struct StructComNome: InicializavelComNome {
    var nome: String
    init(nome: String) {
        self.nome = nome
    }
}
```

Se uma classe conforma a um protocolo que requer um inicializador, a classe deve implementar esse inicializador com o modificador `required`. Isso garante que todas as subclasses também implementem o requisito do inicializador.

### Protocolos como Tipos

Protocolos em si são tipos de primeira classe em Swift. Você pode usá-los como tipos de constantes, variáveis e parâmetros de função.

```swift
func exibirIdentificacao(item: Identificavel) {
    print("ID: \(item.id)")
}

let usuario = Usuario(id: "user123", nome: "Alice")
let produto = Produto(id: "prod456", nome: "Televisão")

exibirIdentificacao(item: usuario) // Saída: ID: user123
exibirIdentificacao(item: produto) // Saída: ID: prod456
```

### Delegação

Um padrão de design comum que usa protocolos é a **delegação**. Um protocolo define os métodos opcionais ou obrigatórios que um delegado (outro objeto) deve implementar para ser notificado sobre eventos ou para fornecer funcionalidade personalizada para um objeto.

```swift
protocol ProcessadorDeDadosDelegate: AnyObject {
    func dadosProcessados(resultado: String)
    optional func erroNoProcessamento(erro: Error)
}

class ProcessadorDeDados {
    weak var delegate: ProcessadorDeDadosDelegate?

    func processar(dados: String) {
        // ... processamento ...
        let resultado = "Dados processados com sucesso"
        delegate?.dadosProcessados(resultado: resultado)
        // ... em caso de erro ...
        // let erro = NSError(domain: "Erro", code: 123, userInfo: nil)
        // delegate?.erroNoProcessamento?(erro: erro)
    }
}

class ViewController: ProcessadorDeDadosDelegate {
    let processador = ProcessadorDeDados()

    init() {
        processador.delegate = self
    }

    func dadosProcessados(resultado: String) {
        print("Resultado do processamento: \(resultado)")
    }

    func erroNoProcessamento(erro: Error) {
        print("Erro no processamento: \(erro.localizedDescription)")
    }
}
```

### Conformidade a Múltiplos Protocolos

Um tipo pode conformar a múltiplos protocolos, listando os nomes dos protocolos separados por vírgulas após os dois pontos na definição do tipo.

```swift
struct MinhaEstrutura: Identificavel, Nomeavel, Acionavel {
    var id: String
    var nome: String
    var isOn: Bool = false

    func ligar() {
        isOn = true
        print("\(nome) ligado")
    }

    func desligar() {
        isOn = false
        print("\(nome) desligado")
    }
}
```

[Link para a documentação oficial sobre Protocolos](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/protocols)

### Extensões de Protocolo e Implementação Padrão

Você pode fornecer implementação padrão para métodos de protocolo usando extensions:

```swift
extension Acionavel {
    func ligarEDesligar() {
        ligar()
        // ... após um tempo ...
        desligar()
    }
}

// Qualquer tipo que conforma a Acionavel agora tem acesso a ligarEDesligar()
```

### Protocolos como Tipos (Type Erasure)

Às vezes, você pode precisar trabalhar com coleções de diferentes tipos que conformam ao mesmo protocolo:

```swift
let dispositivos: [Acionavel] = [
    Interruptor(),
    MinhaEstrutura(id: "dev1", nome: "Dispositivo1")
]

for dispositivo in dispositivos {
    dispositivo.ligar()
}
```

---

**Observações:**

- Protocolos são uma ferramenta poderosa para alcançar polimorfismo e flexibilidade no seu código Swift.
- Eles permitem escrever código mais genérico e desacoplado, facilitando a manutenção e a testabilidade.
- A delegação é um padrão essencial que se baseia em protocolos para comunicação entre objetos.
- Extensions de protocolo permitem fornecer implementações padrão, reduzindo código duplicado.
- Protocolos com associated types (tipos associados) permitem maior expressividade mas requerem type erasure em casos de coleções polimórficas.

Relacionado: [[Classes e Structs]]
