# Closures em Swift

**Closures** são blocos de código auto-contidos que podem ser passados e usados em seu código. Eles são semelhantes a funções, mas podem capturar e armazenar referências a quaisquer constantes e variáveis do contexto em que são definidos. Essa capacidade de capturar valores é conhecida como "closing over" essas constantes e variáveis.

### Sintaxe Básica de um Closure

A sintaxe geral de um closure em Swift é a seguinte:

```swift
{ (parâmetros) -> tipoDeRetorno in
    instruções
}
````

- **`(parâmetros)`**: Define a lista de parâmetros que o closure aceita. A sintaxe é a mesma que a lista de parâmetros de uma função.
- **`-> tipoDeRetorno`**: Indica o tipo de valor que o closure retorna.
- **`in`**: Palavra-chave que separa a lista de parâmetros e o tipo de retorno do corpo do closure.
- **`instruções`**: O bloco de código a ser executado quando o closure é chamado.

### Exemplo Básico

```swift
let saudacao = { (nome: String) -> String in
    return "Olá, \(nome)!"
}

let mensagem = saudacao("Ana")
print(mensagem) // Saída: Olá, Ana!
```

Neste exemplo:

- `saudacao` é uma constante que armazena um closure.
- O closure recebe um parâmetro `nome` do tipo `String` e retorna uma `String`.
- A palavra-chave `in` marca o início do corpo do closure.
- O corpo do closure retorna uma string de saudação que inclui o `nome`.

### Inferência de Tipo em Closures

Em muitos casos, o Swift pode inferir os tipos dos parâmetros e o tipo de retorno de um closure, tornando a sintaxe mais concisa.

```swift
let outraSaudacao = { nome in "Olá novamente, \(nome)!" }
let outraMensagem = outraSaudacao("Pedro")
print(outraMensagem) // Saída: Olá novamente, Pedro!
```

Aqui, os tipos `String` para o parâmetro `nome` e para o tipo de retorno foram inferidos pelo Swift.

### Closures como Argumentos de Funções

Closures são frequentemente usados como argumentos para funções, especialmente em operações assíncronas ou para definir comportamentos customizados.

```swift
func executarAposDelay(segundos: Double, completion: @escaping () -> Void) {
    DispatchQueue.main.asyncAfter(deadline: .now() + segundos) {
        completion()
    }
}

executarAposDelay(segundos: 2) {
    print("Executado após 2 segundos.")
}
```

Neste exemplo, um closure sem parâmetros e sem tipo de retorno é passado como o argumento `completion` para a função `executarAposDelay`. A anotação `@escaping` indica que o closure pode ser executado após o retorno da função.

### Shorthand Argument Names

O Swift fornece nomes abreviados para os argumentos de closures inline. Você pode se referir aos argumentos pelo nome `$0`, `$1`, `$2`, e assim por diante.

```swift
let numeros = [1, 5, 3, 9, 2]
let ordenados = numeros.sorted(by: { $0 < $1 })
print(ordenados) // Saída: [1, 2, 3, 5, 9]
```

Aqui, `$0` e `$1` são os nomes abreviados para os dois argumentos do closure de comparação passado para a função `sorted(by:)`.

### Trailing Closures

Se o último argumento de uma função for um closure, você pode usar a sintaxe de **trailing closure**. Isso significa escrever o closure após os parênteses da chamada da função.

```swift
let outrosOrdenados = numeros.sorted { $0 > $1 }
print(outrosOrdenados) // Saída: [9, 5, 3, 2, 1]
```

### Captura de Valores

Closures podem capturar valores do seu contexto circundante. Mesmo que o escopo original onde essas variáveis foram definidas não exista mais, o closure ainda pode acessar e modificar esses valores (se a variável foi definida como `var`).

```swift
func criarIncrementador(incremento: Int) -> () -> Int {
    var total = 0
    let incrementador: () -> Int = {
        total += incremento
        return total
    }
    return incrementador
}

let incrementarPorDez = criarIncrementador(incremento: 10)
print(incrementarPorDez()) // Saída: 10
print(incrementarPorDez()) // Saída: 20
print(incrementarPorDez()) // Saída: 30

let incrementarPorCinco = criarIncrementador(incremento: 5)
print(incrementarPorCinco()) // Saída: 5
print(incrementarPorDez()) // Saída: 40 (a variável 'total' é capturada separadamente para cada closure)
```

### Closures Escaping e Non-Escaping

Um closure **non-escaping** (padrão) é executado antes da função retornar. Um closure **escaping** pode ser armazenado e executado depois.

```swift
// Non-escaping (padrão)
func processar(dados: [Int], callback: (Int) -> Void) {
    for numero in dados {
        callback(numero)
    }
}

processar(dados: [1, 2, 3]) { numero in
    print(numero)
}

// Escaping - usado para callbacks assincronos
var completionHandlers: [() -> Void] = []

func armazenarCompletion(handler: @escaping () -> Void) {
    completionHandlers.append(handler)
}

armazenarCompletion {
    print("Execução posterior!")
}

// Executar depois
completionHandlers.forEach { $0() }
```

### Closures Capturando Self

Cuidado ao capturar `self` em closures para evitar ciclos de referência forte:

```swift
class Downloader {
    var tarefas: [() -> Void] = []
    
    func baixarComCiclo(url: String) {
        tarefas.append {
            // ❌ Ciclo de referência: self retém a closure, closure retém self
            print("Baixando: \(url)")
        }
    }
    
    func baixarSemCiclo(url: String) {
        tarefas.append { [weak self] in
            // ✓ Melhor: weak self quebra o ciclo
            self?.download(from: url)
        }
    }
    
    func download(from url: String) {
        print("Baixando: \(url)")
    }
}
```

[Link para a documentação oficial sobre Closures](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/closures)

---

**Observações:**

- Closures são um recurso poderoso e flexível em Swift, fundamental para muitos padrões de programação funcional e assíncrona.
- Entender a captura de valores e o ciclo de vida dos closures é importante para evitar retenções fortes (strong reference cycles) em cenários mais complexos.
- A anotação `@escaping` é necessária quando o closure pode ser executado após a função retornar.
- Use `[weak self]` ao capturar `self` em closures de longa duração para evitar ciclos de retenção de memória.

Relacionado: [[Funções]]
