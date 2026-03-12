Quando falamos de "animação em Swift" de forma mais geral, podemos nos referir tanto ao Core Animation (mais de baixo nível e com maior controle) quanto às animações oferecidas pelo SwiftUI (mais declarativas e de alto nível).

**Animação em Swift: Uma Visão Geral dos Dois Casos Principais**

Em Swift, você tem principalmente duas abordagens para criar animações em suas aplicações Apple (iOS, macOS, watchOS, tvOS):

1. **Core Animation:** Um framework poderoso e fundamental que opera diretamente nas camadas (`CALayer`) subjacentes às views. Ele oferece um controle granular sobre as propriedades animáveis e a linha do tempo da animação.
    
2. **SwiftUI Animations:** Um sistema de animação declarativo e mais recente, construído sobre o Core Animation. Ele simplifica a criação de animações ao permitir que você declare como as mudanças de estado devem ser animadas.
    

Vamos revisitar os exemplos anteriores, agora com a perspectiva de "animação em Swift" abrangendo ambos os casos.

### Caso 1: Animação de Posição (Core Animation vs. SwiftUI)

**Core Animation (Swift)**

```swift
import UIKit

class ViewController: UIViewController {

    let animatedView = UIView(frame: CGRect(x: 50, y: 100, width: 100, height: 100))

    override func viewDidLoad() {
        super.viewDidLoad()
        animatedView.backgroundColor = .red
        view.addSubview(animatedView)
    }

    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        animatePositionWithCoreAnimation()
    }

    func animatePositionWithCoreAnimation() {
        let animation = CABasicAnimation(keyPath: "position.x")
        animation.fromValue = animatedView.layer.position.x
        animation.toValue = view.bounds.width - animatedView.frame.width / 2 - 50
        animation.duration = 2.0
        animation.autoreverses = true
        animation.repeatCount = .infinity
        animatedView.layer.add(animation, forKey: "positionAnimation")
    }
}
```

**SwiftUI**

```swift
import SwiftUI

struct ContentView: View {
    @State private var isMoved = false

    var body: some View {
        Rectangle()
            .fill(.red)
            .frame(width: 100, height: 100)
            .offset(x: isMoved ? 200 : 0)
            .animation(.easeInOut(duration: 2.0).repeatForever(autoreverses: true), value: isMoved)
            .onTapGesture {
                isMoved.toggle()
            }
    }
}
```

**Comparação:**

- **Core Animation:** Envolve a criação de um objeto `CABasicAnimation`, a especificação da propriedade a ser animada (`keyPath`), os valores inicial e final, a duração e outros parâmetros. A animação é então adicionada à camada da view.
- **SwiftUI:** Você declara o estado (`isMoved`) que afeta a propriedade visual (`offset`). Ao alterar o estado dentro de um bloco `.animation()`, o SwiftUI interpola a mudança automaticamente. O modificador `.repeatForever(autoreverses: true)` simplifica a repetição da animação.

### Caso 2: Animação de Escala e Opacidade (Core Animation vs. SwiftUI)

**Core Animation (Swift)**

```swift
import UIKit

class ViewController: UIViewController {

    let animatedView = UIView(frame: CGRect(x: 150, y: 200, width: 100, height: 100))

    override func viewDidLoad() {
        super.viewDidLoad()
        animatedView.backgroundColor = .blue
        view.addSubview(animatedView)
    }

    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        animateScaleAndOpacityWithCoreAnimation()
    }

    func animateScaleAndOpacityWithCoreAnimation() {
        // Animação de Escala
        let scaleAnimation = CABasicAnimation(keyPath: "transform.scale")
        scaleAnimation.fromValue = 1.0
        scaleAnimation.toValue = 1.5
        scaleAnimation.duration = 1.5
        scaleAnimation.autoreverses = true
        scaleAnimation.repeatCount = .infinity

        // Animação de Opacidade
        let opacityAnimation = CABasicAnimation(keyPath: "opacity")
        opacityAnimation.fromValue = 1.0
        opacityAnimation.toValue = 0.5
        opacityAnimation.duration = 1.5
        opacityAnimation.autoreverses = true
        opacityAnimation.repeatCount = .infinity

        // Adiciona as animações à camada
        animatedView.layer.add(scaleAnimation, forKey: "scaleAnimation")
        animatedView.layer.add(opacityAnimation, forKey: "opacityAnimation")
    }
}
```

**SwiftUI**

```swift
import SwiftUI

struct ContentView: View {
    @State private var isAnimating = false

    var body: some View {
        Circle()
            .fill(.blue)
            .frame(width: 100, height: 100)
            .scaleEffect(isAnimating ? 1.5 : 1.0)
            .opacity(isAnimating ? 0.5 : 1.0)
            .animation(.easeInOut(duration: 1.5).repeatForever(autoreverses: true), value: isAnimating)
            .onAppear {
                isAnimating = true // Inicia a animação quando a view aparece
            }
    }
}
```

**Comparação:**

- **Core Animation:** Requer a criação de múltiplas instâncias de `CABasicAnimation` (uma para cada propriedade), configurando seus respectivos `keyPath`, valores e parâmetros, e adicionando cada animação à camada.
- **SwiftUI:** Você aplica múltiplos modificadores de estado (`.scaleEffect()`, `.opacity()`) à view, ligados a uma única variável de estado (`isAnimating`). A animação para todas as mudanças de estado é controlada por um único modificador `.animation()`. O `.onAppear` é usado aqui para iniciar a animação assim que a view é exibida.

**Escolhendo entre Core Animation e SwiftUI Animations:**

- **SwiftUI Animations:** Geralmente são preferíveis para a maioria das necessidades de animação devido à sua sintaxe concisa, facilidade de uso e integração com o ciclo de vida das views baseadas em estado. São ótimas para animações simples e médias, transições de interface e efeitos visuais que dependem do estado da view.
    
- **Core Animation:** Ainda é essencial para cenários mais complexos que exigem controle preciso sobre a linha do tempo da animação, animações personalizadas que não são facilmente expressas com as animações predefinidas do SwiftUI, ou integração com código UIKit/AppKit existente onde você já está trabalhando com `CALayer`. Core Animation oferece mais poder e flexibilidade, mas com um custo de maior complexidade e verbosidade no código.
    

Em resumo, ao falar de "animação em Swift", é importante distinguir entre as abordagens oferecidas pelo Core Animation (mais fundamental e controlada) e pelo SwiftUI (mais declarativa e integrada), cada uma com seus próprios casos de uso e vantagens. Para novos projetos em SwiftUI, a abordagem nativa do framework é geralmente recomendada pela sua simplicidade e eficiência.
Relacionados: [Core Animation](Animação%20em%20Swift/Core%20Animation.md) [SwiftUI Animations](Animação%20em%20Swift/SwiftUI%20Animations.md)