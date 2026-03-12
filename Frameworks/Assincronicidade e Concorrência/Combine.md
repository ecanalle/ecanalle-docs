# Combine em Swift

O framework **Combine** fornece uma abordagem declarativa para processar eventos ao longo do tempo. Ele permite trabalhar com streams de valores assíncronos, como eventos de interface do usuário, notificações de rede e dados que mudam ao longo do tempo. Combine se inspira em conceitos de programação reativa e oferece uma maneira unificada e consistente de lidar com a programação assíncrona e orientada a eventos em Swift.

### Principais Conceitos do Combine

Combine gira em torno de três principais tipos de entidades:

* **Publishers**: Representam uma fonte de valores ao longo do tempo. Eles emitem valores para um ou mais Subscribers.
* **Subscribers**: Recebem os valores emitidos por um Publisher e reagem a esses eventos.
* **Operators**: Métodos que modificam os valores produzidos por um Publisher antes de serem recebidos por um Subscriber. Operators permitem transformar, filtrar, combinar e manipular streams de dados.

### Exemplo Básico

```swift
import Combine

let publisher = Just("Hello, Combine!")
    .sink { value in print(value) }
````

Neste exemplo:

- `Just("Hello, Combine!")` cria um Publisher que emite um único valor ("Hello, Combine!") e depois completa.
- `.sink { value in print(value) }` é um Subscriber. O método `sink` anexa um bloco de código (o closure) que será executado quando o Publisher emitir um valor. Neste caso, ele simplesmente imprime o valor recebido.

### Publishers

Publishers são responsáveis por emitir valores ao longo do tempo. Combine fornece vários Publishers integrados para diferentes propósitos, como:

- **`Just`**: Emite um único valor e, em seguida, completa.
- **`Future`**: Representa o resultado de uma operação assíncrona que pode produzir um único valor no futuro ou falhar com um erro.
- **`PassthroughSubject`**: Um Publisher que pode emitir novos valores a qualquer momento chamando seu método `send(_:)`.
- **`CurrentValueSubject`**: Semelhante a `PassthroughSubject`, mas mantém o valor mais recente emitido e envia esse valor imediatamente para qualquer novo Subscriber.
- **Publishers de Foundation**: Combine também integra-se com tipos da Foundation, como `NotificationCenter`(para eventos de notificação) e `URLSession` (para tarefas de rede), fornecendo Publishers para seus eventos assíncronos.

### Subscribers

Subscribers recebem os valores emitidos por um Publisher. O Subscriber mais comum é criado usando o método `sink(receiveCompletion:receiveValue:)` anexado a um Publisher.

```swift
let subject = PassthroughSubject<String, Never>()

let subscription = subject
    .sink(receiveCompletion: { completion in
        switch completion {
        case .finished:
            print("Stream concluído")
        case .failure(let error):
            print("Ocorreu um erro: \(error)")
        }
    }, receiveValue: { value in
        print("Valor recebido: \(value)")
    })

subject.send("Primeiro valor")
subject.send("Segundo valor")
subject.send(completion: .finished)

// Saída:
// Valor recebido: Primeiro valor
// Valor recebido: Segundo valor
// Stream concluído
```

- `receiveCompletion`: Um closure que é chamado quando o Publisher completa com sucesso (`.finished`) ou falha com um erro (`.failure`).
- `receiveValue`: Um closure que é chamado cada vez que o Publisher emite um novo valor.

A `subscription` retornada por `sink` representa a conexão entre o Publisher e o Subscriber. É importante armazenar essa subscription (geralmente em uma propriedade de instância) para manter a assinatura ativa. Quando a subscription é desalocada, a assinatura é cancelada.

### Operators

Operators são métodos encadeados entre um Publisher e um Subscriber para modificar o stream de dados. Combine oferece uma vasta gama de operadores para realizar diversas tarefas:

- **Transformação**: `map`, `flatMap`, `scan`.
- **Filtragem**: `filter`, `removeDuplicates`, `compactMap`.
- **Combinação**: `combineLatest`, `merge`, `zip`.
- **Tratamento de tempo**: `debounce`, `throttle`, `delay`.
- **Tratamento de erros**: `catch`, `retry`.

```swift
let numbers = [1, 2, 3, 4, 5].publisher

let evenNumbers = numbers
    .filter { $0 % 2 == 0 }
    .map { $0 * 2 }
    .sink { value in
        print("Número par multiplicado por 2: \(value)")
    }

// Saída:
// Número par multiplicado por 2: 4
// Número par multiplicado por 2: 8
```

Neste exemplo, o array de números é transformado em um Publisher. O operador `filter` seleciona apenas os números pares, e o operador `map` multiplica cada número par por 2. Finalmente, o `sink` imprime os valores resultantes.

### Canceláveis (`Cancellable`)

As assinaturas em Combine retornam um objeto do tipo `AnyCancellable`. É crucial armazenar essas instâncias para manter a assinatura ativa. Geralmente, você as armazena em um `Set<AnyCancellable>` e a assinatura é automaticamente cancelada quando o `AnyCancellable` é desalocado.

Swift

```
var subscriptions = Set<AnyCancellable>()

let publisher = Timer.publish(every: 1, on: .main, in: .common)
    .autoconnect()
    .sink { _ in
        print("Timer disparou!")
    }
    .store(in: &subscriptions) // Armazena a assinatura no set
```

### Benefícios do Combine

- **Reatividade:** Permite construir fluxos de dados reativos onde as mudanças se propagam automaticamente.
- **Composição:** Fornece um rico conjunto de operadores que podem ser combinados de forma declarativa para construir lógica complexa de processamento de eventos.
- **Consistência:** Oferece uma maneira unificada de lidar com diferentes tipos de eventos assíncronos em todo o seu aplicativo.
- **Segurança de tipo:** O sistema de tipos do Combine ajuda a prevenir erros em tempo de execução relacionados ao fluxo de dados.

[Link para a documentação oficial sobre Combine](https://developer.apple.com/documentation/combine)

---

**Observações:**

- Combine é um framework poderoso que pode simplificar significativamente o tratamento de eventos assíncronos e reativos em aplicativos Swift.
- Entender os conceitos de Publishers, Subscribers e Operators é fundamental para utilizar o Combine de forma eficaz.
- O gerenciamento de assinaturas com `AnyCancellable` é essencial para evitar vazamentos de memória.

Relacionado: [[Async Await]]
