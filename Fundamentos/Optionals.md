# Optionals em Swift

Em Swift, um **optional** é um tipo que permite que uma variável ou constante contenha um valor de um determinado tipo, ou a ausência de valor (`nil`). Optionals são cruciais para lidar com a possibilidade de um valor não existir, evitando erros em tempo de execução.

### Declaração de Optionals

Para declarar uma variável ou constante como optional, você adiciona um ponto de interrogação (`?`) após o tipo. 

```swift
var nome: String? = "João"
var idade: Int? // Inicialmente, idade é nil
````

### Unwrapping Optionals

Como um optional pode ou não conter um valor, você precisa realizar o processo de "**unwrapping**" para acessar o valor dentro dele. Existem várias maneiras de fazer isso de forma segura:

#### Forced Unwrapping (Evite ao máximo)

Você pode usar o operador de exclamação (`!`) para forçar o unwrapping de um optional. No entanto, se o optional for `nil` neste momento, seu programa irá gerar um erro em tempo de execução.

```swift
var sobrenome: String? = "Silva"
// Tenha certeza de que sobrenome não é nil antes de forçar o unwrapping
let sobrenomeNaoOptional: String = sobrenome!
print("Sobrenome: \(sobrenomeNaoOptional)") // Saída: Sobrenome: Silva

var endereco: String? // endereco é nil
// let enderecoNaoOptional: String = endereco! // Isso causaria um erro em tempo de execução
```

**Recomendação:** Evite o forced unwrapping a menos que você tenha certeza absoluta de que o optional contém um valor.

#### Optional Binding (`if let` e `guard let`)

O **optional binding** é uma forma segura e idiomática de verificar se um optional contém um valor e, se sim, torná-lo disponível como uma constante ou variável temporária dentro de um escopo específico.

##### `if let`

```swift
var email: String? = "joao.silva@example.com"

if let emailNaoOptional = email {
    print("O email é: \(emailNaoOptional)") // Executado apenas se email não for nil
} else {
    print("Nenhum email fornecido.") // Executado se email for nil
}
```

##### `guard let`

O `guard let` também é usado para optional binding, mas é frequentemente usado para sair de um escopo (como uma função) se o optional for `nil`.

```swift
func exibirNomeCompleto(nome: String?, sobrenome: String?) {
    guard let nomeNaoOptional = nome else {
        print("Nome é obrigatório.")
        return
    }

    guard let sobrenomeNaoOptional = sobrenome else {
        print("Sobrenome é obrigatório.")
        return
    }

    print("Nome completo: \(nomeNaoOptional) \(sobrenomeNaoOptional)")
}

exibirNomeCompleto(nome: "Maria", sobrenome: "Souza") // Saída: Nome completo: Maria Souza
exibirNomeCompleto(nome: nil, sobrenome: "Oliveira") // Saída: Nome é obrigatório.
```

#### Nil-Coalescing Operator (`??`)

O **nil-coalescing operator** fornece uma forma concisa de desembrulhar um optional ou fornecer um valor padrão caso o optional seja `nil`.

```swift
var apelido: String?
let nomeParaExibir = apelido ?? "Nome Padrão"
print("Nome a ser exibido: \(nomeParaExibir)") // Saída: Nome a ser exibido: Nome Padrão

apelido = "Juninho"
let outroNomeParaExibir = apelido ?? "Outro Nome Padrão"
print("Outro nome a ser exibido: \(outroNomeParaExibir)") // Saída: Outro nome a ser exibido: Juninho
```

#### Optional Chaining

O **optional chaining** permite chamar métodos e acessar propriedades de valores que podem ser `nil`. Se o valor for `nil`, a chamada ou acesso retorna `nil`.

```swift
class Veiculo {
    var proprietario: String?
}

var meuVeiculo: Veiculo? = Veiculo()
meuVeiculo?.proprietario = "João" // Só atribui se meuVeiculo não for nil
print(meuVeiculo?.proprietario ?? "Desconhecido") // Saída: João

meuVeiculo = nil
print(meuVeiculo?.proprietario ?? "Desconhecido") // Saída: Desconhecido
```

### Padrão de Segurança com Optionals

```swift
// ❌ Evite: Forçar unwrapping perigoso
let numero: Int? = nil
print(numero!) // CRASH em tempo de execução!

// ✓ Prefira: Optional binding
if let numero = numero {
    print(numero)
} else {
    print("Número não disponível")
}

// ✓ Ou: Nil-coalescing com valor padrão
print(numero ?? 0) // Usa 0 como padrão
```

[Link para a documentação oficial sobre Optionals](https://www.google.com/search?q=https://docs.swift.org/swift-book/documentation/the-swift-programming-language/optionals)

---

**Observações:**

- Optionals são um conceito fundamental em Swift para a segurança de tipo e para lidar com a ausência de valores de forma explícita.
- O uso correto de optional binding e do nil-coalescing operator é altamente recomendado para escrever código Swift seguro e legível.
- Evite forced unwrapping (`!`) a menos que você tenha certeza absoluta. A maioria das crashes em Swift relacionadas a optionals vêm de forced unwrapping desnecessário.
- O optional é um dos maiores diferenciais de segurança do Swift em comparação com linguagens como C ou Objective-C.

Relacionado: [[Tipos]] | [[Funções]]
