# Tipos em Swift

Em Swift, os **tipos de dados** são fundamentais para entender como manipular e armazenar valores. Swift é uma linguagem com tipagem estática, o que significa que o tipo de cada variável e constante é conhecido em tempo de compilação. Isso ajuda a garantir a segurança do tipo e a prevenir erros.

### Tipos Básicos Comuns

Aqui estão alguns dos tipos de dados básicos mais comuns em Swift:

* **Int**: Usado para representar **números inteiros**, tanto positivos quanto negativos, sem casas decimais. O tamanho de um `Int` depende da arquitetura do sistema (geralmente 32 ou 64 bits). Swift também oferece tipos como `Int8`, `Int16`, `Int32`, `Int64`, `UInt8`, `UInt16`, `UInt32`, `UInt64` para tamanhos específicos.

    ```swift
    let numeroInteiro: Int = 42
    let numeroNegativo: Int = -10
    ```

    [Link para a documentação oficial sobre Inteiros](https://www.google.com/search?q=https://docs.swift.org/swift-book/documentation/the-swift-programming-language/integers)

* **Double**: Usado para representar **números de ponto flutuante** de alta precisão com 64 bits. É adequado para valores que podem ter casas decimais. Swift também oferece o tipo `Float` para números de ponto flutuante de menor precisão (32 bits).

    ```swift
    let numeroDecimal: Double = 3.14159
    let outraDecimal: Double = -0.5
    ```

    [Link para a documentação oficial sobre Números de Ponto Flutuante](https://www.google.com/search?q=https://docs.swift.org/swift-book/documentation/the-swift-programming-language/floating-point-numbers)

* **String**: Usado para representar **sequências de caracteres**, ou seja, texto. As strings em Swift são Unicode-compliant.

    ```swift
    let mensagem: String = "Olá, Swift!"
    let nome: String = "Carlos"
    ```

    [Link para a documentação oficial sobre Strings](https://www.google.com/search?q=https://docs.swift.org/swift-book/documentation/the-swift-programming-language/strings-and-characters)

* **Bool**: Usado para representar **valores booleanos**, que podem ser `true` (verdadeiro) ou `false` (falso). São essenciais para lógica condicional.

    ```swift
    let ehVerdadeiro: Bool = true
    let ehFalso: Bool = false
    ```

    [Link para a documentação oficial sobre Booleanos](https://www.google.com/search?q=https://docs.swift.org/swift-book/documentation/the-swift-programming-language/booleans)

### Conversão entre Tipos

Frequentemente, você precisará converter valores de um tipo para outro. Swift fornece inicializadores para fazer essas conversões:

```swift
// Convertendo String para Int e Double
let numeroString = "42"
let numeroInteiro = Int(numeroString)      // Retorna Int?
let numeroDecimal = Double("3.14")         // Retorna Double?

// Convertendo Int para Double
let inteiro = 10
let decimal: Double = Double(inteiro)      // Sem casting automaticamente
print(decimal) // Saída: 10.0

// Convertendo tipos numéricos
let valor: Double = 5.5
let inteirizado = Int(valor)               // Saída: 5 (trunca a parte decimal)

// Convertendo para String
let numero = 42
let textoNumero = String(numero)           // Saída: "42"
let booleano = true
let textoBooleano = String(booleano)       // Saída: "true"
```

### Tipos Compostos

Além dos tipos básicos, Swift oferece tipos mais complexos:

```swift
// Array (lista de elementos do mesmo tipo)
let numeros: [Int] = [1, 2, 3, 4, 5]

// Dictionary (pares chave-valor)
let usuario: [String: Any] = ["nome": "Ana", "idade": 28]

// Tuple (agrupamento de valores)
let coordenadas: (x: Int, y: Int) = (x: 10, y: 20)
print(coordenadas.x) // Saída: 10
```

-----

**Observações:**

* Swift também possui outros tipos de dados importantes, como `Character`, `Array`, `Dictionary`, `Set`, `Tuple`, `Enum`, `Struct` e `Class`, que exploraremos em outros momentos.
* A inferência de tipo em Swift permite que você omita o tipo ao declarar uma variável ou constante se o compilador puder inferir o tipo a partir do valor inicial.
* Conversões entre tipos nem sempre têm sucesso (por ex: `Int("abc")` retorna `nil`), então é importante tratar erros de conversão apropriadamente.

```swift
let outroNumeroInteiro = 100 // Inferido como Int
let outraDecimal = 2.71828 // Inferido como Double
let outraMensagem = "Inferencia de tipo" // Inferido como String
let outroBooleano = true // Inferido como Bool
```
[[Variáveis e Constantes]] | [[Optionals]]

