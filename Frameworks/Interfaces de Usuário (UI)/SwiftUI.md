# SwiftUI em Swift

**SwiftUI** é um framework de interface de usuário declarativo e moderno da Apple para construir interfaces de aplicativos em todas as plataformas da Apple: iOS, iPadOS, macOS, watchOS e tvOS. Ele permite que os desenvolvedores criem UIs de forma mais intuitiva e com menos código em comparação com o framework UIKit (para iOS, iPadOS e tvOS) e AppKit (para macOS) baseados em paradigma imperativo.

### Abordagem Declarativa

Uma das principais características do SwiftUI é sua abordagem **declarativa** para a construção de interfaces. Em vez de descrever passo a passo como a UI deve ser atualizada em resposta a eventos, você declara o estado desejado da UI para um determinado estado dos seus dados, e o sistema se encarrega de fazer as transições e atualizações necessárias de forma eficiente.

### Estrutura Básica de uma View em SwiftUI

A base da construção de interfaces em SwiftUI é a **View**. Uma view é um protocolo que descreve a estrutura, o layout e a aparência de uma parte da sua interface de usuário. Você cria suas próprias views conformando a esse protocolo.

```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        Text("Hello, World!")
    }
}
````

Neste exemplo:

- `import SwiftUI` importa o framework SwiftUI.
- `struct ContentView: View` declara uma nova struct chamada `ContentView` que conforma ao protocolo `View`.
- `var body: some View` é uma propriedade **requerida** para qualquer tipo que conforme a `View`. Ela descreve o conteúdo da view. `some View` é um tipo opaco, o que permite que o compilador Swift otimize a estrutura da view sem expor detalhes de implementação específicos.
- `Text("Hello, World!")` é uma view que exibe o texto "Hello, World!".

### Composindo Views

As interfaces em SwiftUI são construídas compondo views menores em estruturas mais complexas. Você usa **containers de layout** como `VStack` (pilha vertical), `HStack` (pilha horizontal) e `ZStack` (pilha com elementos sobrepostos) para organizar outras views.

```swift
struct ExemploLayout: View {
    var body: some View {
        VStack {
            Text("Título Principal")
                .font(.largeTitle)
                .padding()

            HStack {
                Image(systemName: "sun.max.fill")
                    .foregroundColor(.yellow)
                Text("Ensolarado")
            }
            .padding()

            Button("Clique Aqui") {
                print("Botão foi clicado!")
            }
        }
    }
}
```

Neste exemplo:

- Uma `VStack` organiza verticalmente um `Text`, um `HStack` e um `Button`.
- O `HStack` organiza horizontalmente uma `Image` e outro `Text`.
- `.font(.largeTitle)` e `.padding()` são **modificadores** que alteram a aparência ou o comportamento da view à qual são aplicados.

### Modificadores de View

Modificadores são métodos que retornam uma nova versão da view com a modificação aplicada. Você pode encadear múltiplos modificadores a uma única view. A ordem dos modificadores pode importar, pois alguns dependem do resultado de modificadores anteriores.

```swift
Text("Texto com Estilo")
    .font(.headline)
    .foregroundColor(.blue)
    .padding(10)
    .border(Color.gray)
```

### Gerenciamento de Estado

Em SwiftUI, a interface de usuário é uma representação do **estado** do seu aplicativo. Quando o estado muda, a UI é automaticamente atualizada para refletir essas mudanças. SwiftUI fornece várias propriedades wrapper para gerenciar diferentes tipos de estado:

- **`@State`**: Usado para gerenciar estado local mutável dentro de uma única view. Quando uma propriedade `@State`muda, a view é redesenhada.
- **`@Binding`**: Usado para criar uma conexão bidirecional entre uma propriedade `@State` em uma view pai e uma propriedade em uma view filha. Isso permite que as mudanças em uma view reflitam na outra.
- **`@ObservedObject`**: Usado para se conectar a um objeto que conforma ao protocolo `ObservableObject`. Esse objeto geralmente contém dados que podem mudar e que afetam múltiplas views. Quando uma propriedade publicada (`@Published`) em um `ObservableObject` muda, todas as views que o observam são atualizadas.
- **`@StateObject`**: Semelhante a `@ObservedObject`, mas usado para criar e gerenciar a instância do `ObservableObject` dentro da própria view. Isso garante que o objeto persista durante o ciclo de vida da view.
- **`@EnvironmentObject`**: Usado para acessar um `ObservableObject` que foi injetado no ambiente de visualização por uma view ancestral.

Swift

```swift
struct ContadorView: View {
    @State private var contador = 0

    var body: some View {
        VStack {
            Text("Contador: \(contador)")
                .font(.title)
                .padding()

            Button("Incrementar") {
                contador += 1
            }
            .padding()
        }
    }
}
```

Neste exemplo, `@State private var contador = 0` cria uma variável de estado local. Quando o botão é clicado, o valor de `contador` é incrementado, e a view é automaticamente redesenhada para exibir o novo valor.
### Exemplo Prático: ViewModel com ObservableObject

```swift
// ViewModel - ObservableObject
class ContadorViewModel: ObservableObject {
    @Published var contagem = 0
    
    func incrementar() {
        contagem += 1
    }
    
    func decrementar() {
        contagem -= 1
    }
}

// View usando o ViewModel
struct ContadorViewAvancada: View {
    @StateObject private var viewModel = ContadorViewModel()

    var body: some View {
        VStack {
            Text("Contador: \(viewModel.contagem)")
                .font(.title)

            HStack {
                Button("-") {
                    viewModel.decrementar()
                }
                .padding()
                
                Button("+") {
                    viewModel.incrementar()
                }
                .padding()
            }
        }
    }
}
```
### Navegação

SwiftUI oferece várias formas de implementar a navegação entre diferentes views, como `NavigationView` (para uma pilha de navegação), `TabView` (para uma interface com abas) e `NavigationStack` (a forma mais recente e recomendada para navegação hierárquica).

```swift
struct ContentView: View {
    var body: some View {
        NavigationStack {
            List {
                NavigationLink(destination: DetailView(item: "Item 1")) {
                    Text("Ir para Detalhe")
                }
            }
            .navigationTitle("Lista")
        }
    }
}

struct DetailView: View {
    let item: String
    
    var body: some View {
        VStack {
            Text("Detalhes de: \(item)")
                .font(.headline)
        }
        .navigationTitle("Detalhe")
    }
}
```

### Concorrência e SwiftUI com Async/Await

### Benefícios do SwiftUI

- **Desenvolvimento mais rápido:** A sintaxe declarativa e os recursos de visualização ao vivo (Canvas) aceleram o processo de desenvolvimento.
- **Menos código:** Geralmente, você precisa escrever menos código para criar a mesma interface em comparação com frameworks imperativos.
- **Consistência entre plataformas:** Compartilhe grande parte do seu código de interface de usuário entre diferentes plataformas da Apple.
- **Integração com as últimas tecnologias da Apple:** SwiftUI é projetado para funcionar bem com outros frameworks e recursos modernos da Apple.

[Link para a documentação oficial sobre SwiftUI](https://developer.apple.com/documentation/swiftui)

---

**Observações:**

- SwiftUI é o futuro do desenvolvimento de interfaces de usuário nas plataformas da Apple.
- Compreender a abordagem declarativa e o gerenciamento de estado é fundamental para construir aplicativos SwiftUI eficazes.
- Explore os vários containers de layout e modificadores para criar interfaces complexas e personalizadas.

Relacionado: [UIKit](UIKit.md)
