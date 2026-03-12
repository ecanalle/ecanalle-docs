# Generics em Swift

**Generics** são uma das funcionalidades mais poderosas do Swift. Eles permitem que você escreva funções e tipos que podem trabalhar com qualquer tipo de dado, mantendo a segurança de tipo. Com generics, você pode escrever código reutilizável que se adapta às necessidades de diferentes tipos sem precisar repetir a mesma lógica para cada um.

### O Problema que os Generics Resolvem

Imagine que você quer escrever uma função para trocar dois valores. Sem generics, você precisaria escrever uma função separada para cada tipo (Int, String, Double, etc.):

```swift
func trocaInt(a: inout Int, b: inout Int) {
    let temp = a
    a = b
    b = temp
}

func trocaString(a: inout String, b: inout String) {
    let temp = a
    a = b
    b = temp
}
````

Isso leva a uma grande quantidade de código repetitivo. Generics oferecem uma solução elegante para esse problema.

### Funções Genéricas

Você escreve funções genéricas usando **parâmetros de tipo**. Esses parâmetros atuam como um placeholder para um tipo real, que é especificado quando a função é chamada. Parâmetros de tipo são escritos dentro de colchetes angulares (`<>`) após o nome da função.

```swift
func troca<T>(a: inout T, b: inout T) {
    let temp = a
    a = b
    b = temp
}

var num1 = 10
var num2 = 20
troca(a: &num1, b: &num2)
print("num1: \(num1), num2: \(num2)") // Saída: num1: 20, num2: 10

var str1 = "Olá"
var str2 = "Mundo"
troca(a: &str1, b: &str2)
print("str1: \(str1), str2: \(str2)") // Saída: str1: Mundo, str2: Olá
```

Neste exemplo:

- `<T>` define `T` como um parâmetro de tipo.
- A função `troca` aceita dois parâmetros do tipo `T` (que devem ser do mesmo tipo) e os troca.
- Quando a função é chamada com inteiros, `T` é inferido como `Int`. Quando chamada com strings, `T` é inferido como `String`.

### Tipos Genéricos

Swift também permite definir tipos genéricos: classes, structs e enums que podem trabalhar com qualquer tipo. Você especifica os parâmetros de tipo ao definir o tipo.

```swift
struct Pilha<Element> {
    var itens = [Element]()

    mutating func empilhar(_ item: Element) {
        itens.append(item)
    }

    mutating func desempilhar() -> Element? {
        return itens.popLast()
    }
}

var pilhaDeInteiros = Pilha<Int>()
pilhaDeInteiros.empilhar(1)
pilhaDeInteiros.empilhar(2)
print(pilhaDeInteiros.desempilhar()!) // Saída: 2

var pilhaDeStrings = Pilha<String>()
pilhaDeStrings.empilhar("maçã")
pilhaDeStrings.empilhar("banana")
print(pilhaDeStrings.desempilhar()!) // Saída: banana
```

Neste exemplo:

- `struct Pilha<Element>` define uma struct genérica chamada `Pilha` que pode armazenar elementos de qualquer tipo. `Element` é o parâmetro de tipo.
- A propriedade `itens` é um array de `Element`.
- Os métodos `empilhar` e `desempilhar` trabalham com o tipo `Element`.
- Ao criar instâncias de `Pilha`, você especifica o tipo real a ser usado (por exemplo, `Pilha<Int>` ou `Pilha<String>`).

### Constraints de Tipo

Às vezes, você pode querer restringir os tipos que podem ser usados com uma função ou tipo genérico. Você pode fazer isso definindo **constraints de tipo** em um parâmetro de tipo. Constraints de tipo especificam que um parâmetro de tipo deve herdar de uma classe específica ou conformar a um protocolo específico ou a uma composição de protocolos.

```swift
protocol TextoDescritivo {
    var descricao: String { get }
}

func imprimirDescricao<T: TextoDescritivo>(item: T) {
    print(item.descricao)
}

struct Livro: TextoDescritivo {
    let titulo: String
    var descricao: String {
        return "Livro: \(titulo)"
    }
}

let meuLivro = Livro(titulo: "Aventuras")
imprimirDescricao(item: meuLivro) // Saída: Livro: Aventuras

// A função imprimirDescricao não pode ser chamada com um tipo que não conforma a TextoDescritivo.
// func imprimirNumero(num: Int) { ... }
// imprimirDescricao(item: 123) // Erro de compilação
```

Neste exemplo:

- `<T: TextoDescritivo>` na função `imprimirDescricao` significa que `T` deve ser um tipo que conforma ao protocolo `TextoDescritivo`.
- Isso permite que a função acesse a propriedade `descricao` do item.

Você também pode usar constraints de tipo com classes:

```swift
class Animal {
    // ...
}

class Cachorro: Animal {
    func latir() {
        print("Au au!")
    }
}

func fazerBarulho<T: Animal>(animal: T) {
    if let cachorro = animal as? Cachorro {
        cachorro.latir()
    } else {
        print("O animal faz algum barulho.")
    }
}

let meuCachorro = Cachorro()
fazerBarulho(animal: meuCachorro) // Saída: Au au!
let meuAnimal = Animal()
fazerBarulho(animal: meuAnimal) // Saída: O animal faz algum barulho.
```

E com composições de protocolos:

```swift
protocol ProtocoloA { /* ... */ }
protocol ProtocoloB { /* ... */ }

func funcaoComDoisProtocolos<T: ProtocoloA & ProtocoloB>(item: T) {
    // ...
}
```

### Benefícios dos Generics

- **Reutilização de código:** Permitem escrever código que funciona com múltiplos tipos.
- **Segurança de tipo:** Garantem que você não passe o tipo errado para uma função ou armazene o tipo errado em uma coleção.
- **Desempenho:** Em muitos casos, o compilador Swift pode gerar código especializado para os tipos concretos usados com generics, o que pode levar a um melhor desempenho em comparação com o uso de tipos genéricos como `Any`.

### Tipos com Multiple Generic Parameters

```swift
class Resultado<Sucesso, Erro> {
    let valor: Sucesso?
    let erro: Erro?
    
    init(sucesso: Sucesso) {
        self.valor = sucesso
        self.erro = nil
    }
    
    init(erro: Erro) {
        self.valor = nil
        self.erro = erro
    }
}

let resultado: Resultado<String, Error> = Resultado(sucesso: "Dados carregados")
```

### Erros Comuns com Generics

```swift
// ❌ Erro: Tentar comparar sem constraint
func ehIgual<T>(a: T, b: T) -> Bool {
    return a == b   // Erro: T não conforma a Equatable
}

// ✓ Solução: Adicionar constraint
func ehIgual<T: Equatable>(a: T, b: T) -> Bool {
    return a == b   // ✓ Agora funciona
}

// Usando com tipos que conformam ao protocolo
print(ehIgual(a: 5, b: 5))      // Saída: true
print(ehIgual(a: "Hello", b: "Hello")) // Saída: true
```

[Link para a documentação oficial sobre Generics](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/generics)

---

**Observações:**

- Generics são uma ferramenta essencial para escrever código Swift flexível, reutilizável e seguro.
- Entender como definir parâmetros de tipo e aplicar constraints de tipo é crucial para aproveitar ao máximo os generics.
- Protocols com associated types fornecem ainda mais capacidade de expressão em Swift avançado.
- Generics funcionam em tempo de compilação, garantindo zero custo abstrato em runtime.

[Protocolos](../Orientação%20a%20Objetos/Protocolos.md) | [Async/Await](../Frameworks/Assincronicidade%20e%20Concorrência/Async%20Await.md)
