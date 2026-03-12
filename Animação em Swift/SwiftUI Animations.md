SwiftUI oferece um sistema de animação declarativo e elegante, construído sobre a mesma base do Core Animation, mas com uma sintaxe muito mais concisa e integrada ao framework.

Em SwiftUI, você geralmente define o _estado_ da sua view e, quando esse estado muda, você pode especificar como essa mudança deve ser animada. O framework cuida da interpolação dos valores e da renderização da animação para você.

### Animando Propriedades com `withAnimation`

A maneira mais comum de animar mudanças de estado em SwiftUI é usando o bloco `withAnimation`. Qualquer alteração de estado dentro desse bloco será animada.

**Exemplo 1: Animando a Opacidade**

Vamos criar um `Rectangle` e animar sua opacidade ao tocar nele.

```swift
import SwiftUI

struct ContentView: View {
    @State private var isOpaque = true

    var body: some View {
        Rectangle()
            .fill(.red)
            .frame(width: 100, height: 100)
            .opacity(isOpaque ? 1.0 : 0.3)
            .onTapGesture {
                withAnimation {
                    isOpaque.toggle()
                }
            }
    }
}
```

**Explicação:**

1. Declaramos uma variável de estado `@State private var isOpaque = true` para controlar a opacidade do retângulo.
2. A propriedade `.opacity()` do `Rectangle` é ligada ao valor de `isOpaque`.
3. Usamos `.onTapGesture` para detectar quando o retângulo é tocado.
4. Dentro do bloco do `onTapGesture`, envolvemos a alteração do estado (`isOpaque.toggle()`) com `withAnimation {}`. Isso instrui o SwiftUI a animar a mudança na opacidade.
5. Por padrão, `withAnimation` usa uma animação suave com duração padrão.

### Personalizando Animações

Você pode personalizar a animação dentro do bloco `withAnimation` passando um objeto `Animation`. O SwiftUI oferece várias animações predefinidas:

- `.easeInOut(duration:)`: Começa e termina a animação lentamente, acelerando no meio.
- `.easeIn(duration:)`: Começa a animação lentamente.
- `.easeOut(duration:)`: Termina a animação lentamente.
- `.linear(duration:)`: Anima em velocidade constante.
- `.spring(response:dampingFraction:blendDuration:)`: Cria uma animação com efeito de mola.
- `.interactiveSpring(response:dampingFraction:blendDuration:)`: Similar a `.spring`, mas otimizado para interações.

**Exemplo 2: Animando a Escala com Efeito de Mola**

```swift
import SwiftUI

struct ContentView: View {
    @State private var isScaled = false

    var body: some View {
        Circle()
            .fill(.blue)
            .frame(width: 100, height: 100)
            .scaleEffect(isScaled ? 1.5 : 1.0)
            .onTapGesture {
                withAnimation(.spring(response: 0.5, dampingFraction: 0.5)) {
                    isScaled.toggle()
                }
            }
    }
}
```

**Explicação:**

- Usamos a propriedade `.scaleEffect()` para controlar a escala do círculo, ligada ao estado `isScaled`.
- Dentro do `withAnimation`, especificamos uma animação de mola com um tempo de resposta de 0.5 segundos e um fator de amortecimento de 0.5.

### Animações Implícitas

No SwiftUI, as animações geralmente são implícitas. Você declara o estado desejado e o SwiftUI anima a transição desse estado se você o envolver em um bloco `withAnimation`.

**Exemplo 3: Animando a Posição e a Rotação Simultaneamente**

```swift
import SwiftUI

struct ContentView: View {
    @State private var isMoved = false

    var body: some View {
        RoundedRectangle(cornerRadius: 20)
            .fill(.green)
            .frame(width: 150, height: 100)
            .offset(x: isMoved ? 100 : 0, y: 0)
            .rotationEffect(Angle(degrees: isMoved ? 360 : 0))
            .animation(.easeInOut(duration: 1.0), value: isMoved) // Especifica a animação para a mudança de 'isMoved'
            .onTapGesture {
                isMoved.toggle()
            }
    }
}
```

**Explicação:**

- Usamos `.offset()` para controlar a posição horizontal e `.rotationEffect()` para controlar a rotação do retângulo arredondado, ambos ligados ao estado `isMoved`.
- Aplicamos o modificador `.animation()` diretamente à view. O argumento `value: isMoved` informa ao SwiftUI para animar sempre que o valor de `isMoved` mudar, usando a animação `.easeInOut(duration: 1.0)`.

### Transições

SwiftUI também oferece transições para animar a entrada e a saída de views da tela. Você pode usar o modificador `.transition()` em uma view dentro de um bloco condicional (`if`).

**Exemplo 4: Animando a Entrada e Saída de uma View com Transição de Deslizamento**

```swift
import SwiftUI

struct ContentView: View {
    @State private var isShowing = false

    var body: some View {
        VStack {
            Button("Mostrar/Esconder") {
                withAnimation {
                    isShowing.toggle()
                }
            }

            if isShowing {
                Text("Olá, Mundo!")
                    .font(.largeTitle)
                    .transition(.slide) // Aplica a transição de deslizamento
            }
        }
    }
}
```

**Explicação:**

- Quando `isShowing` se torna `true`, a `Text` view aparece com uma transição de deslizamento. Quando se torna `false`, ela desaparece com a mesma transição.
- SwiftUI oferece várias transições predefinidas como `.slide`, `.opacity`, `.scale`, e você também pode criar transições personalizadas.

### Animações de Fase (`PhaseAnimator`)

Para animações mais complexas que envolvem uma sequência de estados ou fases, o SwiftUI introduziu o `PhaseAnimator`. Ele permite definir uma série de fases com valores associados e anima a transição entre essas fases.

**Exemplo 5: Animação de Pulso com `PhaseAnimator`**

```swift
import SwiftUI

struct ContentView: View {
    enum PulsePhase: CaseIterable {
        case initial, scaled
    }

    var body: some View {
        PhaseAnimator(PulsePhase.allCases) { phase in
            Circle()
                .fill(.purple)
                .frame(width: 100, height: 100)
                .scaleEffect(phase == .scaled ? 1.2 : 1.0)
                .opacity(phase == .scaled ? 0.8 : 1.0)
        } animation: { phase in
            switch phase {
            case .initial:
                return .easeInOut(duration: 0.5)
            case .scaled:
                return .easeInOut(duration: 0.5).delay(0.2)
            }
        }
        .onAppear {
            // Inicia a animação repetidamente
            Timer.scheduledTimer(withTimeInterval: 1.4, repeats: true) { _ in
                // No PhaseAnimator, a animação geralmente é controlada pelas fases
                // Neste exemplo, podemos não precisar de um Timer externo se a lógica
                // da animação estiver totalmente dentro do PhaseAnimator.
                // Para um pulso contínuo, o PhaseAnimator fará isso automaticamente.
            }
        }
    }
}
```

**Explicação:**

- Definimos um enum `PulsePhase` com as fases `.initial` e `.scaled`.
- O `PhaseAnimator` itera sobre os casos de `PulsePhase`. Para cada fase, ele aplica modificadores à `Circle`.
- O closure `animation:` define a animação para cada transição de fase.
- O `onAppear` com um `Timer` é uma forma de iniciar um ciclo repetitivo de fases (embora o `PhaseAnimator` possa ser configurado para repetir automaticamente em cenários mais complexos).

### Dicas Importantes para Animações em SwiftUI

- **Declarativo:** As animações em SwiftUI são declarativas. Você descreve o estado final e como a transição deve ocorrer, e o framework cuida dos detalhes da animação.
- **Estado é a Chave:** As animações em SwiftUI são geralmente acionadas por mudanças no estado (`@State`, `@Binding`, `@ObservedObject`, etc.).
- **Modificadores:** Use os modificadores `.animation()` e `.transition()` para especificar as animações e transições.
- **Performance:** O SwiftUI é otimizado para performance, e as animações geralmente são suaves e eficientes.
- **Composição:** Você pode combinar várias animações e transições para criar efeitos complexos.

SwiftUI simplifica muito a criação de animações expressivas em suas aplicações, aproveitando o poder do Core Animation por baixo dos panos. Ao entender os conceitos básicos de `withAnimation`, `Animation`, e `transition`, você pode adicionar facilmente movimento e vida à sua interface de usuário.

Relacionados: [SwiftUI](Frameworks/Interfaces%20de%20Usuário%20(UI)/SwiftUI.md)