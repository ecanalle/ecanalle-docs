# Extensions em Swift

**Extensions** são um recurso poderoso em Swift que permite adicionar novas funcionalidades a um tipo existente de classe, struct, enum ou protocolo. Isso inclui tipos para os quais você não tem o código-fonte original (como tipos fornecidos por bibliotecas ou frameworks do sistema). As extensions são semelhantes a categorias em Objective-C, mas não têm nomes.

### Capacidades das Extensions

As extensions em Swift podem:

* Adicionar propriedades computadas de instância e de tipo.
* Definir métodos de instância e de tipo.
* Fornecer novos inicializadores.
* Definir subscripts.
* Definir e usar novos tipos aninhados.
* Fazer um tipo existente conformar a um protocolo.

### Sintaxe de uma Extension

A sintaxe para declarar uma extension é a seguinte:

```swift
extension NomeDoTipo {
    // Novas funcionalidades a serem adicionadas ao NomeDoTipo
}
````

Uma extension pode estender um tipo para fazê-lo adotar um ou mais protocolos:

```swift
extension NomeDoTipo: NomeDoProtocolo, OutroProtocolo {
    // Implementação dos requisitos do protocolo
}
```

### Exemplos

#### Adicionando um Método de Instância

```swift
extension String {
    func saudacao() -> String {
        return "Olá, \(self)"
    }
}

let nome = "Alice"
let cumprimento = nome.saudacao()
print(cumprimento) // Saída: Olá, Alice
```

Neste exemplo, uma extension adiciona um novo método de instância chamado `saudacao()` ao tipo `String`. Agora, qualquer instância de `String` pode chamar esse método.

#### Adicionando uma Propriedade Computada de Instância

```swift
extension Double {
    var aoQuadrado: Double {
        return self * self
    }
}

let numero = 3.0
let quadrado = numero.aoQuadrado
print(quadrado) // Saída: 9.0
```

Aqui, uma extension adiciona uma propriedade computada de instância chamada `aoQuadrado` ao tipo `Double`. Ela fornece uma maneira concisa de obter o quadrado de um `Double`.

#### Fornecendo Novos Inicializadores

Extensions podem adicionar novos inicializadores a tipos existentes. Isso permite estender outros tipos para aceitar seus próprios tipos personalizados como parâmetros de inicialização ou fornecer opções de inicialização adicionais.

```swift
struct Tamanho {
    var largura = 0.0, altura = 0.0
}

extension Tamanho {
    init(lado: Double) {
        self.largura = lado
        self.altura = lado
    }
}

let quadrado = Tamanho(lado: 5.0)
print("Largura: \(quadrado.largura), Altura: \(quadrado.altura)") // Saída: Largura: 5.0, Altura: 5.0
```

Esta extension adiciona um novo inicializador à struct `Tamanho` que permite criar uma instância com um único valor para ambos, largura e altura.

#### Adicionando um Método de Tipo

Extensions também podem adicionar métodos de tipo (métodos chamados no próprio tipo, em vez de em uma instância do tipo). Você indica um método de tipo escrevendo a palavra-chave `static` antes da palavra-chave `func`.

```swift
extension Int {
    static func vezesDois(valor: Int) -> Int {
        return valor * 2
    }
}

let resultado = Int.vezesDois(valor: 7)
print(resultado) // Saída: 14
```

#### Fazendo um Tipo Existente Conformar a um Protocolo

Extensions podem ser usadas para fazer um tipo existente conformar a um protocolo. Isso é particularmente útil para adotar protocolos em tipos que você não escreveu.

```swift
protocol TextoDescritivo {
    var descricao: String { get }
}

extension Int: TextoDescritivo {
    var descricao: String {
        return "Este é o número \(self)"
    }
}

let numeroInteiro = 10
print(numeroInteiro.descricao) // Saída: Este é o número 10
```

Neste exemplo, uma extension faz o tipo `Int` conformar ao protocolo `TextoDescritivo` fornecendo uma implementação para a propriedade requerida `descricao`.

### Observações Importantes sobre Extensions

- Extensions podem adicionar novas funcionalidades a um tipo, mas não podem sobrescrever funcionalidades existentes.
- Extensions podem adicionar propriedades computadas, mas não podem adicionar propriedades armazenadas a tipos existentes. No entanto, você pode adicionar propriedades armazenadas declarando-as com associações de objetos em Objective-C e expondo-as para Swift.
- Extensions podem adicionar novos inicializadores convenientes a uma classe, mas não podem adicionar novos inicializadores designados. Eles também não podem adicionar deinitializers a classes.

### Casos de Uso Comuns para Extensions

```swift
// Adicionar validação a String
extension String {
    var eValido: Bool {
        return self.count >= 3
    }
    
    var ehVazioBranco: Bool {
        return self.trimmingCharacters(in: .whitespaces).isEmpty
    }
}

let email = "abc"
print(email.eValido) // Saída: true

// Estender Array com método customizado
extension Array {
    func printElements() {
        for (index, element) in self.enumerated() {
            print("Índice \(index): \(element)")
        }
    }
}

let numeros = [1, 2, 3]
numeros.printElements()
```

### Organizando Código com Extensions

```swift
// Ao invés de um arquivo gigante, você pode organizar assim:
protocol PaymentProcessing {
    func processPayment()
}

extension PaymentProcessing {
    func processPaymentWithTracking() {
        print("Iniciando processamento...")
        processPayment()
        print("Processamento concluído")
    }
}
```

[Link para a documentação oficial sobre Extensions](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/extensions)

---

**Observações:**

- Extensions são uma ferramenta poderosa para estender e adaptar tipos existentes para atender às necessidades do seu código.
- Elas promovem a organização e a legibilidade do código, permitindo separar funcionalidades adicionais da definição original do tipo.
- Use extensions para conformar tipos existentes a protocolos, melhorando a arquitetura e a modularidade do código.
- Organizando extensions por funcionalidade em arquivos ou blocos separados melhora significativamente a legibilidade do código em projetos grandes.

Relacionado: [[Classes e Structs]]
