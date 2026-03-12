# Async/Await em Swift

O modelo de programação **async/await** introduzido no Swift facilita significativamente o trabalho com código assíncrono. Ele fornece uma maneira mais legível e estruturada de escrever código que executa tarefas em segundo plano sem bloquear a thread principal, tornando a experiência do usuário mais responsiva.

### O Problema com Callbacks

Tradicionalmente, operações assíncronas em Swift eram frequentemente tratadas com callbacks (funções que são passadas para serem executadas após a conclusão de uma tarefa assíncrona). No entanto, código com muitos callbacks pode se tornar complexo e difícil de ler e manter, um fenômeno conhecido como "callback hell" ou "pyramid of doom".

### Introdução ao Async/Await

O `async/await` oferece uma alternativa mais linear e semelhante a código síncrono para lidar com operações assíncronas.

* **`async`**: Uma função marcada com `async` indica que ela pode pausar a execução enquanto espera por uma operação assíncrona ser concluída. Ela só pode ser chamada de dentro de outra função `async` ou de um contexto que suporte concorrência estruturada (como uma `Task`).
* **`await`**: A palavra-chave `await` é usada dentro de uma função `async` para pausar a execução até que a operação assíncrona à sua direita seja concluída e retorne um resultado. Enquanto a função está em espera, a thread atual (geralmente uma thread de um pool de threads) é liberada para executar outras tarefas, evitando o bloqueio da thread principal.

### Exemplo Básico

```swift
async func carregarDados() -> String {
    // Simula uma operação assíncrona (por exemplo, uma requisição de rede)
    try? await Task.sleep(nanoseconds: 2_000_000_000) // Espera por 2 segundos
    return "Dados carregados"
}

func processarDados(dados: String) {
    print("Dados processados: \(dados)")
}

func iniciarProcessamento() {
    Task { // Cria um contexto para executar código assíncrono
        print("Iniciando o carregamento de dados...")
        let resultado = await carregarDados()
        processarDados(dados: resultado)
    }
    print("A thread principal não foi bloqueada.")
}

iniciarProcessamento()
// Saída (aproximadamente):
// Iniciando o carregamento de dados...
// A thread principal não foi bloqueada.
// Dados processados: Dados carregados (após 2 segundos)
````

Neste exemplo:

- `carregarDados()` é uma função `async` que simula uma operação assíncrona com uma espera de 2 segundos e, em seguida, retorna uma string.
- Dentro da função `iniciarProcessamento()`, uma `Task` é criada para executar o código assíncrono.
- A palavra-chave `await` é usada para pausar a execução dentro da `Task` até que `carregarDados()` termine de executar e retorne o valor.
- Enquanto `carregarDados()` está em espera, a thread principal continua executando, como demonstrado pela mensagem "A thread principal não foi bloqueada." sendo impressa antes do resultado dos dados carregados.

### Tratamento de Erros com `async/await`

Você pode usar `try` com `await` para lidar com funções assíncronas que podem lançar erros.

```swift
enum ErroDeCarregamento: Error {
    case falhaNaRede
}

async func carregarDadosComErro() throws -> String {
    try? await Task.sleep(nanoseconds: 1_000_000_000)
    throw ErroDeCarregamento.falhaNaRede
}

func iniciarCarregamentoComTratamentoDeErro() {
    Task {
        do {
            let resultado = try await carregarDadosComErro()
            print("Resultado: \(resultado)")
        } catch {
            print("Ocorreu um erro: \(error)")
        }
    }
}

iniciarCarregamentoComTratamentoDeErro()
// Saída (após 1 segundo):
// Ocorreu um erro: falhaNaRede
```

### Concorrência Estruturada

O `async/await` no Swift é construído sobre o conceito de **concorrência estruturada**. Isso significa que as tarefas assíncronas são executadas dentro de um escopo bem definido, geralmente associado à estrutura de chamadas de função. Isso ajuda a garantir que as tarefas sejam gerenciadas corretamente e que os recursos sejam liberados adequadamente. A `Task` no exemplo anterior é um exemplo de como criar um contexto para concorrência estruturada.

### Principais Benefícios do Async/Await

- **Código mais legível:** O código assíncrono se parece mais com código síncrono, reduzindo a complexidade visual dos callbacks.
- **Melhor tratamento de erros:** O uso de `try/catch` torna o tratamento de erros em código assíncrono mais intuitivo.
- **Sequencialização lógica:** Facilita a escrita de código que executa operações assíncronas em uma ordem específica.
- **Concorrência estruturada:** Ajuda a gerenciar o ciclo de vida das tarefas assíncronas de forma mais segura e previsível.

[Link para a documentação oficial sobre Concorrência (incluindo Async/Await)](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/concurrency)

---

**Observações:**

- `async/await` é uma das adições mais significativas ao Swift nos últimos anos, simplificando muito a programação assíncrona.
- É importante entender como usar `Task` para criar contextos onde funções `async` podem ser chamadas.
- A concorrência estruturada é um conceito chave para escrever código assíncrono robusto e seguro.

Relacionado: [Generics](../../Swift%20Avançado/Generics.md)
