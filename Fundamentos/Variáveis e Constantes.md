# Variáveis e Constantes em Swift

Em Swift, **variáveis** e **constantes** são usadas para armazenar valores. A principal diferença entre elas é a capacidade de alterar o valor após a sua inicialização.

### Variáveis (`var`)

Variáveis são declaradas usando a palavra-chave `var`. O valor de uma variável pode ser alterado quantas vezes forem necessárias ao longo do programa.

```swift
var mensagem = "Olá"
print(mensagem) // Saída: Olá

mensagem = "Olá, Swift!"
print(mensagem) // Saída: Olá, Swift!
````

### Constantes (`let`)

Constantes são declaradas usando a palavra-chave `let`. Uma vez que um valor é atribuído a uma constante, ele não pode ser alterado posteriormente. Tentar modificar uma constante resultará em um erro de compilação.

```swift
let anoNascimento = 1993
print(anoNascimento) // Saída: 1993

// anoNascimento = 2024 // Isso geraria um erro de compilação
```

### Inferência de Tipo e Declaração Explícita

Em Swift, você pode declarar variáveis e constantes com ou sem especificar o tipo explicitamente. Se você fornecer um valor inicial, o Swift tentará inferir o tipo automaticamente (inferência de tipo).

```swift
var nome = "João" // O tipo String é inferido
let idade = 30    // O tipo Int é inferido
```

Você também pode declarar o tipo explicitamente usando a sintaxe `: Tipo` após o nome da variável ou constante. Isso é útil quando você quer deixar claro o tipo ou quando não está fornecendo um valor inicial imediatamente (no caso de variáveis).

```swift
var sobrenome: String
sobrenome = "Silva"

let altura: Double = 1.75
```

### Escolhendo entre `var` e `let`

É uma boa prática usar `let` para declarar constantes sempre que o valor não precisar ser alterado. Isso torna o código mais seguro e fácil de entender, pois indica claramente quais valores são imutáveis. Use `var` apenas quando a necessidade de alterar o valor for clara.
#### Exemplo Comparativo

```swift
// Bom: Uso apropriado de let e var
let pi = 3.14159  // Valor constante, nunca muda
var contador = 0   // Valor que será incrementado
while contador < 5 {
    print(contador)
    contador += 1
}

// Menos direto: let para algo que deveria ser var
let statusUsuario = "ativo"
// statusUsuario = "inativo" // Erro - você pretendia modificar isso
```

### Escopo e Ciclo de Vida

```swift
func demonstrarEscopo() {
    let nomeGlobal = "Visível em toda a função"
    
    if true {
        let nomeLocal = "Visível apenas neste bloco"
        print(nomeGlobal) // ✓ Funciona
        print(nomeLocal)  // ✓ Funciona
    }
    
    // print(nomeLocal) // ✗ Erro: nomeLocal não está acessível aqui
}
```
[Link para a documentação oficial sobre Variáveis e Constantes](https://www.google.com/search?q=https://docs.swift.org/swift-book/documentation/the-swift-programming-language/variables-and-constants)

---

**Observações:**

- A escolha entre `var` e `let` afeta a mutabilidade do valor, não o tipo. Uma variável sempre terá o mesmo tipo, mesmo que seu valor mude.
- Constantes precisam ser inicializadas com um valor no momento de sua declaração ou antes de serem usadas pela primeira vez.- Seguidores da filosofia funcional preferem maximizar o uso de `let` e minimizar o de `var` para reduzir estado mutável.
- Variáveis e constantes com escopo mais reduzido (em blocos menores) são mais fáceis de rastrear e depurar.
[Tipos](Tipos.md) | [Optionals](Optionals.md)
