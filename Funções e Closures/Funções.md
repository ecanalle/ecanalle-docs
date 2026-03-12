# Funções em Swift

**Funções** são blocos de código auto-contidos que realizam uma tarefa específica. Você declara e define uma função uma vez e pode chamá-la várias vezes em diferentes partes do seu código para reutilizar essa lógica. Funções podem receber valores como entrada (chamados de parâmetros) e podem retornar um valor como saída.

### Declaração de Funções

A sintaxe básica para declarar uma função em Swift é a seguinte:

```swift
func nomeDaFuncao(parametro1: Tipo1, parametro2: Tipo2, ...) -> TipoDeRetorno {
    // Corpo da função - código a ser executado
    return valorDeRetorno
}
````

- **`func`**: Palavra-chave que indica a declaração de uma função.
- **`nomeDaFuncao`**: Um nome descritivo que identifica a função.
- **`(parametro1: Tipo1, parametro2: Tipo2, ...)`**: A lista de parâmetros que a função recebe. Cada parâmetro tem um nome e um tipo.
- **`-> TipoDeRetorno`**: Indica o tipo de valor que a função retorna. Se a função não retornar nenhum valor, o tipo de retorno é `Void` (ou você pode omitir completamente a `->` e o tipo de retorno).
- **`{ ... }`**: O corpo da função, que contém as instruções a serem executadas.
- **`return valorDeRetorno`**: Se a função tem um tipo de retorno não-`Void`, ela deve retornar um valor desse tipo usando a palavra-chave `return`.

### Exemplo Básico

```swift
func saudacao(nome: String) -> String {
    return "Olá, \(nome)!"
}

let mensagem = saudacao(nome: "Carlos")
print(mensagem) // Saída: Olá, Carlos!
```

Neste exemplo:

- `saudacao` é o nome da função.
- `nome: String` declara um parâmetro chamado `nome` do tipo `String`.
- `-> String` indica que a função retorna um valor do tipo `String`.
- O corpo da função cria uma string de saudação usando o valor do parâmetro `nome` e a retorna.

### Funções sem Parâmetros

Funções podem ser definidas sem receber nenhum parâmetro.

Swift

```swift
func mensagemDeBoasVindas() -> String {
    return "Bem-vindo!"
}

let boasVindas = mensagemDeBoasVindas()
print(boasVindas) // Saída: Bem-vindo!
```

### Funções sem Valor de Retorno

Funções também podem ser definidas para não retornar nenhum valor. Nesses casos, o tipo de retorno é `Void` ou pode ser omitido.

Swift

```swift
func exibirMensagem(texto: String) {
    print(texto)
}

exibirMensagem(texto: "Esta é uma mensagem.") // Saída: Esta é uma mensagem.

func outraMensagem(texto: String) -> Void {
    print(texto)
    return // Opcional em funções Void
}

outraMensagem(texto: "Outra mensagem.") // Saída: Outra mensagem.
```

### Parâmetros de Função

Funções podem ter múltiplos parâmetros. Cada parâmetro é especificado com um nome e um tipo, separados por vírgulas.

```swift
func apresentar(nome: String, idade: Int) {
    print("Meu nome é \(nome) e eu tenho \(idade) anos.")
}

apresentar(nome: "Ana", idade: 25) // Saída: Meu nome é Ana e eu tenho 25 anos.
```

#### Rótulos de Argumento e Nomes de Parâmetro

Cada parâmetro de função tem um **rótulo de argumento** e um **nome de parâmetro**. O rótulo de argumento é usado ao chamar a função; o nome de parâmetro é usado dentro do corpo da função. Por padrão, o primeiro parâmetro não tem um rótulo de argumento, e os parâmetros subsequentes usam seus nomes como rótulos.

Você pode especificar um rótulo de argumento explícito antes do nome do parâmetro:

```swift
func cumprimentar(para pessoa: String, com mensagem: String) {
    print("\(mensagem), \(pessoa)!")
}

cumprimentar(para: "Pedro", com: "Saudações") // Saída: Saudações, Pedro!
```

Você pode usar um sublinhado (`_`) para omitir o rótulo de argumento:

```swift
func somar(_ a: Int, _ b: Int) -> Int {
    return a + b
}

let resultado = somar(5, 3)
print(resultado) // Saída: 8
```

#### Valores Padrão para Parâmetros

Você pode definir valores padrão para os parâmetros de uma função. Se um valor não for fornecido ao chamar a função, o valor padrão será usado.

```swift
func exibirDetalhes(nome: String, cidade: String = "Joinville") {
    print("\(nome) mora em \(cidade).")
}

exibirDetalhes(nome: "Maria") // Saída: Maria mora em Joinville.
exibirDetalhes(nome: "João", cidade: "São Paulo") // Saída: João mora em São Paulo.
```

#### Parâmetros Variádicos

Um parâmetro variádico aceita zero ou mais valores de um tipo específico. Você indica um parâmetro variádico inserindo três pontos (`...`) após o nome do tipo do parâmetro. Os valores passados para um parâmetro variádico são disponibilizados dentro do corpo da função como um array do tipo apropriado.

```swift
func calcularMedia(numeros: Double...) -> Double {
    var total = 0.0
    for numero in numeros {
        total += numero
    }
    return total / Double(numeros.count)
}

let media1 = calcularMedia(numeros: 1.0, 2.0, 3.0, 4.0, 5.0) // media1 é 3.0
let media2 = calcularMedia(numeros: 2.5, 7.5) // media2 é 5.0
let media3 = calcularMedia() // media3 é NaN (divisão por zero)
```

#### Parâmetros `inout`

Por padrão, os parâmetros de função são constantes. Se você quiser que uma função modifique o valor de um parâmetro e que essas alterações persistam fora da chamada da função, você pode declarar o parâmetro como um parâmetro `inout`.

Você passa um parâmetro `inout` para uma função precedendo o nome da variável com um e comercial (`&`) ao chamar a função.

```swift
func trocar(_ a: inout Int, _ b: inout Int) {
    let temp = a
    a = b
    b = temp
}

var num1 = 10
var num2 = 20
trocar(&num1, &num2)
print("num1 agora é \(num1), e num2 agora é \(num2)") // Saída: num1 agora é 20, e num2 agora é 10
```

### Funções como Tipos

Em Swift, funções são tipos de primeira classe, o que significa que você pode atribuir funções a variáveis e passá-las como parâmetros para outras funções.

```swift
func siteAdição(a: Int, b: Int) -> Int {
    return a + b
}

func subtracao(a: Int, b: Int) -> Int {
    return a - b
}

// Atribuindo uma função a uma variável
var operacao: (Int, Int) -> Int = siteAdição
print(operacao(5, 3)) // Saída: 8

operacao = subtracao
print(operacao(5, 3)) // Saída: 2

// Passando função como parâmetro
func executar(_ operacao: (Int, Int) -> Int, com a: Int, e b: Int) {
    print("Resultado: \(operacao(a, b))")
}

executar(siteAdição, com: 10, e: 5) // Saída: Resultado: 15
```

### Funções Aninhadas

Você pode definir funções dentro de outras funções. As funções aninhadas têm acesso às variáveis da função que as contém.

```swift
func escolherOperacao(tipo: String) -> (Int, Int) -> Int {
    func somar(a: Int, b: Int) -> Int {
        return a + b
    }
    
    func multiplicar(a: Int, b: Int) -> Int {
        return a * b
    }
    
    if tipo == "soma" {
        return somar
    } else {
        return multiplicar
    }
}

let minhaSoma = escolherOperacao(tipo: "soma")
print(minhaSoma(3, 4)) // Saída: 7

let meuProduto = escolherOperacao(tipo: "multiplicar")
print(meuProduto(3, 4)) // Saída: 12
```

### Retornando Múltiplos Valores com Tuples

Às vezes, você precisa retornar múltiplos valores de uma função. Você pode usar tuples para isso:

```swift
func buscarInfoUsuario() -> (nome: String, idade: Int, ativo: Bool) {
    return (nome: "Carlos", idade: 30, ativo: true)
}

let info = buscarInfoUsuario()
print("Nome: \(info.nome), Idade: \(info.idade)") // Saída: Nome: Carlos, Idade: 30

// Ou usando desestruturação
let (nome, idade, _) = buscarInfoUsuario()
print("Usuário: \(nome), com \(idade) anos") // Saída: Usuário: Carlos, com 30 anos
```

[Link para a documentação oficial sobre Funções](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/functions)

---

**Observações:**

- Funções são blocos de construção essenciais para organizar e reutilizar código em Swift.
- Compreender os diferentes tipos de parâmetros e retornos permite criar funções flexíveis e poderosas.
- Funções aninhadas são úteis para encapsular lógica complexa e manter o escopo das variáveis locais.
- Retornar tuples é uma forma idiomática de retornar múltiplos valores sem criar estruturas ou classes extras.
- Em Swift, as funções são consideradas "cidadãos de primeira classe", permitindo programação funcional elegante.

[[Optionals]] | [[Closures]]
