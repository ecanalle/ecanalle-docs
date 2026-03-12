Core Animation é um framework poderoso da Apple para criar animações ricas e fluidas em aplicativos para iOS, macOS, tvOS e watchOS. Ele opera em um nível abaixo do UIKit (ou AppKit no macOS), permitindo um alto grau de controle e otimização para animações complexas.

Em Swift, você interage com Core Animation principalmente através da camada `CALayer`. Cada `UIView` que você vê na tela tem uma camada subjacente (`layer` property) que é responsável por sua renderização e animações.

### A Camada `CALayer`

Pense na `CALayer` como a "tela" onde o conteúdo visual de uma view é desenhado. Ela gerencia propriedades visuais como:

- `bounds`: O tamanho e a posição da camada dentro de seu próprio sistema de coordenadas.
- `position`: A posição da âncora da camada no sistema de coordenadas de seu superlayer.
- `anchorPoint`: Um ponto dentro da camada que é usado como ponto de referência para transformações e posicionamento (normalizado para o intervalo [0, 1] em ambos os eixos). O padrão é (0.5, 0.5), o centro da camada.
- `transform`: Uma matriz de transformação 3D que permite rotação, escala, tradução e outras transformações geométricas.
- `opacity`: A opacidade da camada (de 0.0 a 1.0).
- `backgroundColor`: A cor de fundo da camada.
- `borderColor`: A cor da borda da camada.
- `borderWidth`: A largura da borda da camada.
- `cornerRadius`: O raio dos cantos da camada.
- `contents`: O conteúdo visual da camada (geralmente uma imagem `CGImage`).

### Criando Animações com `CABasicAnimation`

A forma mais comum de animar propriedades de uma `CALayer` é usando a classe `CABasicAnimation`. Ela permite animar uma propriedade de um valor inicial para um valor final ao longo de uma duração especificada.

**Exemplo 1: Animando a Posição**

Vamos criar uma view simples e animar sua posição horizontalmente.

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
        animatePosition()
    }

    func animatePosition() {
        let animation = CABasicAnimation(keyPath: "position.x")
        animation.fromValue = animatedView.layer.position.x
        animation.toValue = view.bounds.width - animatedView.frame.width / 2 - 50 // Mover para a direita
        animation.duration = 2.0
        animation.timingFunction = CAMediaTimingFunction(name: .easeInEaseOut)
        animation.autoreverses = true // Anima de volta ao valor inicial
        animation.repeatCount = .infinity // Repete a animação indefinidamente

        animatedView.layer.add(animation, forKey: "positionAnimation")
    }
}
```

**Explicação:**

1. Criamos uma `UIView` vermelha.
2. Na função `animatePosition`, criamos uma instância de `CABasicAnimation`.
3. `keyPath`: Especifica qual propriedade da camada será animada. Aqui, estamos animando a coordenada `x` da propriedade `position`.
4. `fromValue`: O valor inicial da propriedade.
5. `toValue`: O valor final da propriedade.
6. `duration`: A duração da animação em segundos.
7. `timingFunction`: Controla a velocidade da animação ao longo do tempo. `.easeInEaseOut` começa e termina a animação lentamente, acelerando no meio.
8. `autoreverses`: Se `true`, a animação será reproduzida na direção oposta ao atingir o valor final.
9. `repeatCount`: O número de vezes que a animação será repetida. `.infinity` faz com que a animação se repita indefinidamente.
10. `animatedView.layer.add(animation, forKey: "positionAnimation")`: Adiciona a animação à camada da nossa view. A `forKey` é uma string opcional que você pode usar para identificar e remover a animação posteriormente.

### Animando Outras Propriedades

Você pode animar várias outras propriedades da mesma forma, apenas alterando o `keyPath` e os valores `fromValue` e `toValue`.

**Exemplo 2: Animando a Escala e a Opacidade**

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
        animateScaleAndOpacity()
    }

    func animateScaleAndOpacity() {
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

**Explicação:**

- Para animar a escala, usamos o `keyPath` `"transform.scale"`. Os valores são fatores de escala (1.0 é o tamanho original).
- Para animar a opacidade, usamos o `keyPath` `"opacity"`. Os valores variam de 0.0 (totalmente transparente) a 1.0 (totalmente opaco).
- Adicionamos ambas as animações à mesma camada. Elas ocorrerão simultaneamente.

### `CAAnimationGroup`

Para coordenar várias animações para que ocorram sequencialmente ou simultaneamente com controles mais precisos (como diferentes inícios e durações), você pode usar `CAAnimationGroup`.

**Exemplo 3: Animando Posição e Rotação Simultaneamente**

```swift
import UIKit

class ViewController: UIViewController {

    let animatedView = UIView(frame: CGRect(x: 50, y: 350, width: 80, height: 80))

    override func viewDidLoad() {
        super.viewDidLoad()
        animatedView.backgroundColor = .green
        view.addSubview(animatedView)
    }

    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        animatePositionAndRotation()
    }

    func animatePositionAndRotation() {
        // Animação de Posição
        let positionAnimation = CABasicAnimation(keyPath: "position.y")
        positionAnimation.fromValue = animatedView.layer.position.y
        positionAnimation.toValue = view.bounds.height - animatedView.frame.height / 2 - 50
        positionAnimation.duration = 2.0

        // Animação de Rotação
        let rotationAnimation = CABasicAnimation(keyPath: "transform.rotation")
        rotationAnimation.fromValue = 0
        rotationAnimation.toValue = CGFloat.pi * 2 // Uma rotação completa (360 graus)
        rotationAnimation.duration = 2.0

        // Grupo de Animações
        let animationGroup = CAAnimationGroup()
        animationGroup.animations = [positionAnimation, rotationAnimation]
        animationGroup.duration = 2.0
        animationGroup.autoreverses = true
        animationGroup.repeatCount = .infinity

        animatedView.layer.add(animationGroup, forKey: "positionAndRotationAnimation")
    }
}
```

**Explicação:**

- Criamos duas instâncias de `CABasicAnimation`: uma para a posição vertical (`position.y`) e outra para a rotação (`transform.rotation`). A rotação é especificada em radianos.
- Criamos um `CAAnimationGroup`.
- Atribuímos um array contendo as animações individuais à propriedade `animations` do grupo.
- A `duration` do grupo geralmente é definida para cobrir a duração da animação mais longa dentro do grupo.
- Adicionamos o `animationGroup` à camada.

### Delegate de Animação (`CAAnimationDelegate`)

Você pode receber callbacks quando uma animação começa ou termina implementando o protocolo `CAAnimationDelegate`.

```swift
import UIKit

class ViewController: UIViewController, CAAnimationDelegate {

    let animatedView = UIView(frame: CGRect(x: 200, y: 100, width: 80, height: 80))

    override func viewDidLoad() {
        super.viewDidLoad()
        animatedView.backgroundColor = .orange
        view.addSubview(animatedView)
    }

    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        animateWithDelegate()
    }

    func animateWithDelegate() {
        let animation = CABasicAnimation(keyPath: "opacity")
        animation.fromValue = 0.0
        animation.toValue = 1.0
        animation.duration = 1.0
        animation.delegate = self // Define o delegate
        animatedView.layer.add(animation, forKey: "fadeIn")
    }

    // MARK: - CAAnimationDelegate Methods

    func animationDidStart(_ anim: CAAnimation) {
        print("Animação começou!")
    }

    func animationDidStop(_ anim: CAAnimation, finished flag: Bool) {
        if flag {
            print("Animação terminou!")
            // Faça algo após a animação terminar, como remover a animação
            // animatedView.layer.removeAnimation(forKey: "fadeIn")
        } else {
            print("Animação interrompida.")
        }
    }
}
```

**Explicação:**

- A classe `ViewController` conforma ao protocolo `CAAnimationDelegate`.
- Definimos `self` como o `delegate` da animação.
- Implementamos os métodos `animationDidStart(_:)` e `animationDidStop(_:finished:)` para receber os callbacks.

### Dicas Importantes

- **Performance:** Animações Core Animation são geralmente muito performáticas porque ocorrem na thread de renderização separada da thread principal da sua aplicação. Isso evita o bloqueio da interface do usuário.
- **Model Layer vs. Presentation Layer:** Enquanto uma animação está em andamento, a `layer` da sua view (o _model layer_) mantém seu estado final. O que você vê na tela é a _presentation layer_, que é uma cópia do model layer que representa o estado atual da animação. Você pode interagir com a presentation layer durante uma animação, mas geralmente é melhor modificar o model layer para alterações permanentes.
- **Blocos de Animação do UIKit:** Para animações mais simples envolvendo propriedades de `UIView`, o UIKit oferece blocos de animação (`UIView.animate(withDuration:animations:)`) que são mais convenientes e constroem sobre o Core Animation. No entanto, para animações mais complexas ou para manipular propriedades diretamente na camada, o Core Animation oferece mais flexibilidade.
- **Key Paths:** A string `keyPath` é fundamental para especificar qual propriedade você deseja animar. A documentação da Apple lista os key paths animáveis para `CALayer` e suas subclasses.

Este é apenas um ponto de partida para explorar o Core Animation em Swift. Existem muitas outras possibilidades, como `CASpringAnimation` para animações com efeito de mola, `CATransition` para transições entre camadas, e `CAKeyframeAnimation` para animações mais complexas baseadas em uma série de valores-chave. Experimente e explore a documentação para descobrir todo o potencial deste poderoso framework!




Relacionados: [UIKit](Frameworks/Interfaces%20de%20Usuário%20(UI)/UIKit.md) [AppKit](Frameworks/Interfaces%20de%20Usuário%20(UI)/AppKit.md)