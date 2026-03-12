# Classes e Structs em Swift

Swift oferece duas construções fundamentais para modelar dados e comportamentos relacionados: **classes** e **structs** (estruturas). Ambas podem definir propriedades para armazenar valores e métodos para fornecer funcionalidades. No entanto, existem diferenças importantes entre elas que influenciam a escolha de qual usar em diferentes situações.

### Semelhanças entre Classes e Structs

Classes e structs em Swift podem:

* Definir propriedades para armazenar valores.
* Definir métodos para fornecer funcionalidades.
* Definir inicializadores para configurar seus estados iniciais.
* Ser estendidas para adicionar funcionalidades além de sua implementação padrão.
* Conformar a protocolos para fornecer funcionalidades de um determinado tipo.

### Diferenças Fundamentais

As principais diferenças entre classes e structs em Swift são:

#### 1. Herança

* **Classes** podem herdar características de outras classes. Isso permite criar uma hierarquia de classes onde subclasses herdam propriedades e métodos de suas superclasses e podem adicionais ou sobrescrevê-los.
* **Structs** não suportam herança.

#### 2. Tipo por Valor vs. Tipo por Referência

* **Structs** são **tipos por valor**. Quando uma struct é atribuída a uma nova variável ou constante, ou quando é passada para uma função, uma cópia do seu valor é criada. As alterações na cópia não afetam a struct original.
* **Classes** são **tipos por referência**. Quando uma instância de uma classe é atribuída a uma nova variável ou constante, ou quando é passada para uma função, uma referência à mesma instância é passada. Isso significa que múltiplas variáveis e constantes podem referenciar a mesma instância na memória, e as alterações em uma referência afetam todas as outras referências à mesma instância.

#### 3. Inicializadores

* **Classes** não recebem um inicializador membro a membro (memberwise initializer) automático se não definirem nenhum inicializador personalizado. Se uma classe fornece um ou mais inicializadores personalizados, ela é responsável por garantir que todas as propriedades armazenadas não opcionais sejam inicializadas. Classes também podem definir deinitializers, que são chamados antes que uma instância de classe seja desalocada.
* **Structs** recebem um inicializador membro a membro automático para todas as suas propriedades armazenadas se não definirem nenhum inicializador personalizado. Mesmo que você defina inicializadores personalizados para uma struct, ela ainda pode reter seu inicializador membro a membro. Structs não podem definir deinitializers.

#### 4. Mutabilidade

* Para modificar as propriedades de uma instância de struct dentro de seus métodos, você precisa marcar o método como `mutating`. Métodos de classe podem modificar suas propriedades sem essa palavra-chave.
* A mutabilidade de uma instância de struct depende de como ela é declarada: se declarada como uma variável (`var`), suas propriedades podem ser modificadas; se declarada como uma constante (`let`), suas propriedades não podem ser modificadas (mesmo que as propriedades sejam variáveis dentro da struct). Para instâncias de classe declaradas como `let`, suas propriedades variáveis ainda podem ser modificadas.

#### 5. Detecção de Identidade

* Como classes são tipos por referência, é possível verificar se duas constantes ou variáveis se referem à mesma instância de classe usando o operador de identidade (`===` e `!==`).
* Não há um equivalente para structs porque elas são tipos por valor.

### Exemplo Básico

```swift
// Struct
struct Ponto {
    var x: Int
    var y: Int
}

var ponto1 = Ponto(x: 10, y: 20)
var ponto2 = ponto1 // ponto2 recebe uma cópia de ponto1
ponto2.x = 30
print(ponto1.x) // Saída: 10 (ponto1 não foi alterado)
print(ponto2.x) // Saída: 30

// Classe
class Pessoa {
    var nome: String
    init(nome: String) {
        self.nome = nome
    }
}

let pessoa1 = Pessoa(nome: "João")
let pessoa2 = pessoa1 // pessoa2 e pessoa1 referenciam a mesma instância
pessoa2.nome = "Maria"
print(pessoa1.nome) // Saída: Maria (pessoa1 também foi alterado)
print(pessoa2.nome) // Saída: Maria
```

### Escolhendo entre Classes e Structs

A escolha entre classes e structs depende da natureza dos dados que você precisa modelar e do comportamento associado. A Apple recomenda usar structs por padrão e usar classes quando a herança ou a identidade de referência são necessárias.

**Use structs quando:**

- O propósito principal da estrutura é encapsular alguns valores de dados relacionados simples.
- Você espera que os valores encapsulados sejam copiados em vez de referenciados quando você atribui ou passa essa estrutura.
- Quaisquer propriedades armazenadas pela estrutura são elas próprias tipos por valor.
- Você não precisa que a estrutura herde comportamento ou identidade de outra estrutura ou classe existente.

**Use classes quando:**

- Você precisa de herança.
- A identidade da instância é importante (comparar se duas referências se referem à mesma instância).
- Você precisa compartilhar e modificar o estado de um objeto entre múltiplas partes do seu código.
- Você precisa usar deinitializers para liberar recursos.

[Link para a documentação oficial sobre Classes e Structs](https://www.google.com/search?q=https://docs.swift.org/swift-book/documentation/the-swift-programming-language/classes-and-structures)

---

**Observações:**

- A escolha entre classes e structs é uma decisão fundamental no design do seu código Swift.
- Compreender as implicações de tipo por valor e tipo por referência é crucial para evitar comportamentos inesperados.

### Mutabilidade em Structs

Um aspecto importante das structs é sua mutabilidade. Para modificar propriedades de uma struct dentro de um método, você precisa marcar o método como `mutating`:

```swift
struct Ponto {
    var x = 0.0, y = 0.0
    
    mutating func mover(dx: Double, dy: Double) {
        x += dx
        y += dy
    }
}

var ponto = Ponto(x: 0.0, y: 0.0)
ponto.mover(dx: 5, dy: 10)
print("Ponto agora está em (\(ponto.x), \(ponto.y))") // Saída: Ponto agora está em (5.0, 10.0)

// Se ponto fosse let (constante), não poderia chamar métodos mutating
let pontoCostante = Ponto()
// pontoCostante.mover(dx: 1, dy: 1) // Erro!
```

### Comparação Prática

```swift
// STRUCT (Tipo por Valor)
struct Configuração {
    var tema = "claro"
    var tamanhoFonte = 14
}

var config1 = Configuração()
var config2 = config1  // Cópia
config2.tema = "escuro"
print(config1.tema)    // Saída: claro (não afetado)

// CLASSE (Tipo por Referência)
class UsuarioDeslogado {
    var email = ""
    var logado = true
}

let user1 = UsuarioDeslogado()
let user2 = user1      // Mesma referência
user2.email = "novo@email.com"
print(user1.email)     // Saída: novo@email.com (afetado!)
```

### Quando Usar Cada Uma

| Critério | Struct | Classe |
|----------|--------|--------|
| Tipo por Valor | ✓ | ✗ |
| Herança | ✗ | ✓ |
| Referência de Identidade | ✗ | ✓ |
| Deinitializer | ✗ | ✓ |
| Inicializador automático | ✓ | ✗ |
| Dados Simples | ✓ | ✗ |
| Hierarquia Complexa | ✗ | ✓ |
| Thread-safe por natureza | ✓ | ✗ |

Relacionado: [[Herança]]
