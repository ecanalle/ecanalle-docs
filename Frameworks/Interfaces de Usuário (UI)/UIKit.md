# UIKit em Swift

**UIKit** é o framework de interface de usuário tradicional para construir aplicativos para iOS, iPadOS, tvOS e watchOS (embora watchOS tenha seu próprio subset e framework relacionado, WatchKit/SwiftUI). Introduzido com a primeira versão do iOS, UIKit fornece as classes base para gerenciar e coordenar a interface gráfica do usuário, os eventos do sistema e a interação do usuário com o aplicativo.

### Abordagem Imperativa

Ao contrário do SwiftUI, UIKit usa uma abordagem **imperativa** para a construção de interfaces. Isso significa que você descreve passo a passo como a UI deve ser criada, configurada e atualizada em resposta a eventos e mudanças de estado.

### Componentes Fundamentais do UIKit

* **`UIView`**: A classe base para todas as views visuais na interface do usuário. É responsável por renderizar conteúdo na tela e responder a eventos de toque.
* **`UIViewController`**: Gerencia uma hierarquia de views (`UIView`s) e controla o ciclo de vida dessas views. Cada tela ou parte da interface geralmente é controlada por um `UIViewController`.
* **`UIWindow`**: A janela na qual as views do seu aplicativo são desenhadas. Geralmente, cada aplicativo tem pelo menos uma `UIWindow`.
* **Controles**: Elementos interativos como `UIButton`, `UILabel`, `UITextField`, `UISwitch`, `UISlider`, etc., que permitem a interação do usuário.
* **Layout**: Mecanismos para organizar e posicionar views dentro de seus contêineres, incluindo Auto Layout (com `NSLayoutConstraint` e `UIStackView`) e Size Classes.
* **Navegação**: Classes como `UINavigationController` (para navegação hierárquica) e `UITabBarController` (para interfaces com abas).
* **Eventos e Gestos**: Mecanismos para lidar com toques, gestos (swipes, pinches, etc.) e outros eventos do sistema.

### Exemplo Básico de um `UIViewController`

```swift
import UIKit

class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
        // Chamado após a view do controller ser carregada na memória.
        print("ViewController carregado!")

        // Criando uma UILabel programaticamente
        let label = UILabel()
        label.text = "Olá, UIKit!"
        label.frame = CGRect(x: 20, y: 100, width: 200, height: 30)
        label.textAlignment = .center
        view.addSubview(label)

        // Criando um UIButton programaticamente
        let button = UIButton(type: .system)
        button.setTitle("Clique Aqui", for: .normal)
        button.frame = CGRect(x: 20, y: 150, width: 200, height: 44)
        button.addTarget(self, action: #selector(botaoTocado), for: .touchUpInside)
        view.addSubview(button)
    }

    @objc func botaoTocado() {
        print("Botão foi tocado!")
    }
}
````

Neste exemplo:

- `ViewController` é uma subclasse de `UIViewController`.
- `viewDidLoad()` é um método do ciclo de vida chamado após a view do controller ser carregada. Aqui, configuramos a UI programaticamente.
- Uma `UILabel` e um `UIButton` são criados, configurados (texto, frame, alinhamento, título) e adicionados à view principal do `ViewController` (`view`).
- `addTarget(_:action:for:)` é usado para associar uma ação (o método `botaoTocado`) ao evento de toqueUpInside do botão.
- `@objc` é necessário para expor o método `botaoTocado` para o Objective-C runtime, que é a base do UIKit.

### Layout com Auto Layout

Auto Layout é um sistema poderoso para definir relações entre as views, permitindo que a interface se adapte a diferentes tamanhos de tela e orientações. Você define **constraints** (restrições) que ditam a posição e o tamanho das views.

```swift
import UIKit

class LayoutViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .white

        let quadradoVermelho = UIView()
        quadradoVermelho.backgroundColor = .red
        quadradoVermelho.translatesAutoresizingMaskIntoConstraints = false // Desativa as constraints automáticas baseadas em frame
        view.addSubview(quadradoVermelho)

        let quadradoAzul = UIView()
        quadradoAzul.backgroundColor = .blue
        quadradoAzul.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(quadradoAzul)

        // Constraints para o quadrado vermelho
        NSLayoutConstraint.activate([
            quadradoVermelho.widthAnchor.constraint(equalToConstant: 100),
            quadradoVermelho.heightAnchor.constraint(equalToConstant: 100),
            quadradoVermelho.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 20),
            quadradoVermelho.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 20)
        ])

        // Constraints para o quadrado azul
        NSLayoutConstraint.activate([
            quadradoAzul.widthAnchor.constraint(equalToConstant: 100),
            quadradoAzul.heightAnchor.constraint(equalToConstant: 100),
            quadradoAzul.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -20),
            quadradoAzul.bottomAnchor.constraint(equalTo: view.safeAreaLayoutGuide.bottomAnchor, constant: -20)
        ])
    }
}
```

Neste exemplo, dois `UIView`s (vermelho e azul) são criados e suas posições e tamanhos são definidos usando Auto Layout constraints. `translatesAutoresizingMaskIntoConstraints = false` é crucial ao usar Auto Layout programaticamente. `safeAreaLayoutGuide` ajuda a posicionar as views de forma segura, evitando sobreposição com a barra de status, a barra de navegação e a barra inferior.

### Ciclo de Vida do `UIViewController`

Os `UIViewController`s passam por um ciclo de vida bem definido, com métodos que são chamados em diferentes estágios:

- `viewDidLoad()`: Chamado após a view do controller ser carregada na memória.
- `viewWillAppear(_:)`: Chamado pouco antes da view do controller se tornar visível.
- `viewDidAppear(_:)`: Chamado depois que a view do controller se torna visível.
- `viewWillDisappear(_:)`: Chamado pouco antes da view do controller deixar de ser visível.
- `viewDidDisappear(_:)`: Chamado depois que a view do controller deixa de ser visível.
- `deinit`: Chamado antes que o objeto `UIViewController` seja desalocado da memória.

### Integração com Storyboards e Interface Builder

UIKit também permite construir interfaces visualmente usando Storyboards e o Interface Builder dentro do Xcode. Você pode arrastar e soltar views, configurar suas propriedades e definir constraints visualmente, o que pode acelerar o desenvolvimento da UI para muitos casos. O código Swift é então conectado aos elementos visuais por meio de **outlets**(`@IBOutlet`) e **actions** (`@IBAction`).

### Considerações Finais sobre UIKit

- UIKit é um framework maduro e poderoso com uma vasta gama de recursos e componentes.
- Embora SwiftUI seja a tecnologia de UI mais recente da Apple, muitos aplicativos ainda são construídos e mantidos com UIKit.
- Compreender o paradigma imperativo, o ciclo de vida dos View Controllers e o Auto Layout são essenciais para o desenvolvimento com UIKit.

[Link para a documentação oficial sobre UIKit](https://developer.apple.com/documentation/uikit)

---

**Observações:**

- UIKit tem sido a principal ferramenta para o desenvolvimento de interfaces iOS por muitos anos e ainda é amplamente utilizado.
- Para desenvolvedores entrando no ecossistema da Apple, entender os fundamentos do UIKit pode ser valioso, especialmente para trabalhar em projetos existentes ou para cenários onde SwiftUI ainda não oferece todas as funcionalidades necessárias.

Relacionado: [[SwiftUI]]
