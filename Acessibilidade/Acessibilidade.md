# Acessibilidade em iOS/macOS

**Acessibilidade** garante que aplicativos sejam usáveis por pessoas com deficiências (visuais, auditivas, motoras, cognitivas). É tanto ético quanto legal (WCAG, ADA).

## Por Que Acessibilidade Importa

- **~15% da população** tem alguma deficiência
- Apps acessíveis têm maior alcance de mercado
- Melhor UX beneficia *todos* (idosos, ambiente barulhento, etc.)
- Obrigatório em muitos contextos legais

## VoiceOver (Leitor de Tela)

O principal assistivo no iOS. Descreve a interface para usuários cegos/com baixa visão.

### Habilitar VoiceOver

**iOS:** Configurações → Acessibilidade → VoiceOver
**macOS:** Preferências de Sistema → Acessibilidade → VoiceOver

### Propriedade `accessibilityLabel`

```swift
import SwiftUI

Button(action: {}) {
    Image(systemName: "trash")
}
.accessibilityLabel("Deletar item")  // VO lerá "Delete item" ao invés de "trash"
```

### Propriedade `accessibilityHint`

Dica adicional para contexto:

```swift
Button("Salvar") {}
    .accessibilityLabel("Salvar documento")
    .accessibilityHint("Salva as alterações atuais")  // VO lê após pausa
```

### Agrupar Elementos (accessibilityElement)

```swift
VStack {
    HStack {
        Image(systemName: "checkmark.circle.fill")
        Text("Tarefa Completa")
    }
    .accessibilityElement(children: .combine)
    .accessibilityLabel("Tarefa completa")
}
```

## Tipos de Trait (Tipo de Elemento)

Informam o tipo do componente ao VoiceOver:

```swift
Button("Enviar") {}
    .accessibilityAddTraits(.isButton)
    .accessibilityRemoveTraits(.isStatic)

Text("Status: Ativo")
    .accessibilityAddTraits(.isHeader)  // Marca como título

Image("icon")
    .accessibilityAddTraits(.isImage)
    .accessibilityHidden(true)  // Se decorativa
```

**Traits Comuns:**
- `.isButton` — Botão
- `.isHeader` — Cabeçalho
- `.isImage` — Imagem
- `.isSelected` — Selecionado
- `.isLink` — Link
- `.updatesFrequently` — Muda frequentemente

## UIKit + Accessibility

### UILabel com Acessibilidade

```swift
let label = UILabel()
label.text = "99 mensagens"
label.isAccessibilityElement = true
label.accessibilityLabel = "99 novas mensagens"
label.accessibilityTraits = .staticText
```

### UIButton com Acessibilidade

```swift
let button = UIButton()
button.setTitle("Curtir", for: .normal)
button.isAccessibilityElement = true
button.accessibilityLabel = "Curtir postagem"
button.accessibilityHint = "Duplo toque para curtir"  // Instruções de uso
button.accessibilityTraits = .button
```

## Contraste de Cor

Mínimo WCAG AA: 4.5:1 para texto, 3:1 para gráficos.

```swift
// SwiftUI com contraste verificado
Text("Importante")
    .foregroundColor(.white)  // #FFFFFF
    .background(Color.blue)    // #0000FF (contraste 8.59:1 ✅)

// Usar ferramentas:
// - Figma: Plugin "Accessible Colors"
// - WebAIM: Contrast Checker (webaim.org/resources/contrastchecker/)
```

## Tamanho de Fonte e Escalabilidade

### DinamicType (Tamanho Ajustável)

```swift
struct ContentView: View {
    @Environment(\.font) var font
    
    var body: some View {
        Text("Título")
            .font(.system(.title, design: .default))
            .allowsTightening(true)
    }
}
```

### Mínimo de 12pt

```swift
Text("Pequeno texto")
    .font(.system(size: 12))  // Mínimo para ler
```

### Não Desabilitar Escalabilidade

```swift
// ❌ NÃO faça:
Text("Texto")
    .font(.system(size: 14))  // Tamanho fixo

// ✅ Sim:
Text("Texto")
    .font(.body)  // Responde a Dynamic Type
    .environment(\.sizeCategory, .large)  // Teste com tamanho grande
```

## Cores Não São Únicas

Não comunique informação usando cor únicamente:

```swift
// ❌ Ruim: comunicar status apenas com cor
ZStack {
    Circle().fill(Color.red)
    Text("X")
}

// ✅ Bem: cor + ícone + texto
ZStack {
    Circle().fill(Color.red)
    Image(systemName: "xmark")
        .foregroundColor(.white)
}
.accessibilityLabel("Erro")
```

## Estrutura de Hierarquia

Use headers/títulos corretamente:

```swift
// SwiftUI
struct ContentView: View {
    var body: some View {
        VStack(alignment: .leading) {
            Text("Seção Principal")
                .font(.title)
                .accessibilityAddTraits(.isHeader)
            
            Text("Conteúdo da seção...")
            
            Text("Subseção")
                .font(.headline)
                .accessibilityAddTraits(.isHeader)
        }
    }
}
```

## Navegação por Teclado

Importante para motricidade limitada:

```swift
// SwiftUI
Button("Ação") {
    // Ação
}
.keyboardShortcut(.defaultAction)  // Enter/Return

// UIKit
let button = UIButton()
button.addKeyCommand(
    UIKeyCommand(input: "s", modifierFlags: .control, action: #selector(handleSave))
)
```

## Focus Externo

Indicar qual elemento tem foco (teclado/VoiceOver):

```swift
@State private var focusedField: String?

VStack {
    TextField("Nome", text: $name)
        .focused($focusedField, equals: "name")
        .accessibilityLabel("Nome do usuário")
    
    Button("Enviar") {
        focusedField = nil  // Move foco
    }
}
```

## Teste de Acessibilidade

### Ativar Inspetor de Acessibilidade

**Xcode:** Produto → Scheme → Edit Scheme → Diagnósticos → Marcar "Accessibility Inspector"

### Checar com VoiceOver

1. Ativar VoiceOver (Configurações → Acessibilidade)
2. Navegar com swipes
3. Certificar que:
   - Todos os elementos tem label
   - Labels são descritivos
   - Ordem de leitura faz sentido
   - Botões têm ações claras

### Teste de Contraste

Use Color Contrast Analyzer ou WebAIM Contrast Checker.

### Rodar Testes Automáticos

```swift
import XCTest

class AccessibilityTests: XCTestCase {
    func testButtonHasAccessibilityLabel() {
        let app = XCUIApplication()
        app.launch()
        
        let button = app.buttons["Delete"]
        XCTAssertNotNil(button.label)
    }
}
```

## Exemplo Completo: Card Acessível

```swift
import SwiftUI

struct AccessibleCard: View {
    let title: String
    let description: String
    let isCompleted: Bool
    let onToggle: () -> Void
    
    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            HStack {
                Image(systemName: isCompleted ? "checkmark.circle.fill" : "circle")
                    .foregroundColor(isCompleted ? .green : .gray)
                    .accessibilityHidden(true)  // Decorativo
                
                VStack(alignment: .leading) {
                    Text(title)
                        .font(.headline)
                        .accessibilityAddTraits(.isHeader)
                    
                    Text(description)
                        .font(.caption)
                        .foregroundColor(.gray)
                }
            }
            
            Button(action: onToggle) {
                Text(isCompleted ? "Desmarcar" : "Marcar como feito")
                    .frame(maxWidth: .infinity)
                    .padding(.vertical, 8)
            }
            .buttonStyle(.bordered)
        }
        .padding()
        .border(Color.gray.opacity(0.3))
        .accessibilityElement(children: .contain)
        .accessibilityLabel(title)
        .accessibilityValue(isCompleted ? "Completo" : "Pendente")
        .accessibilityHint("Duplo toque para alternar status")
    }
}
```

## Checklist de Acessibilidade

- [ ] Todos os elementos interativos têm `accessibilityLabel`
- [ ] Contraste de cor > 4.5:1 para texto
- [ ] Tamanho mínimo de fonte: 12pt
- [ ] Informação não depende apenas de cor
- [ ] Navegação por teclado funciona
- [ ] VoiceOver pode ler a interface
- [ ] Ordem de leitura é lógica
- [ ] Imagens decorativas têm `accessibilityHidden(true)`
- [ ] Testes com inspetor de acessibilidade passam
- [ ] Documentação de recursos acessíveis

## Recursos Oficiais

- [Apple Accessibility Documentation](https://developer.apple.com/accessibility/)
- [WCAG 2.1 Guidelines](https://www.w3.org/WAI/WCAG21/quickref/)
- [iOS Accessibility Testing Guide](https://developer.apple.com/videos/play/wwdc2021/254/)
- [Inclusive Design Principles](https://inclusivedesignprinciples.org/)

## Leitura Recomendada

- "Inclusive Components" (Heydon Pickering)
- "Accessible Web Design" (Jeremy Sydik)

**Relacionados:** [Interface de Usuário (UI)](../Frameworks/Interfaces%20de%20Usuário%20(UI)/Interface%20de%20Usuário%20(UI).md) [SwiftUI](../Frameworks/Interfaces%20de%20Usuário%20(UI)/SwiftUI.md) [UIKit](../Frameworks/Interfaces%20de%20Usuário%20(UI)/UIKit.md) [Internacionalização e Localização](../Internacionalização%20e%20Localização/Internacionalização%20e%20Localização.md)
