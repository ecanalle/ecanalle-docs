# AppKit em Swift

**AppKit** é o framework principal da Apple para construir interfaces de usuário em aplicações macOS. Assim como UIKit para iOS, AppKit fornece componentes prontos (widgets, controles, janelas) para criar interfaces desktop nativas e responsivas.

### Características Principais

- **Nativo do macOS:** Framework oficial e totalmente integrado com o SO.
- **Poderoso:** Oferece controle fino sobre a interface e comportamento da aplicação.
- **Maduro:** Décadas de desenvolvimento e refinamento.
- **Integração com Cocoa:** Funciona perfeitamente com outras APIs macOS.
- **Suporta tanto Swift quanto Objective-C**

### Componentes Principais

#### Janelas e Views

```swift
import AppKit

let window = NSWindow(
    contentRect: NSRect(x: 100, y: 100, width: 400, height: 300),
    styleMask: [.titled, .closable, .miniaturizable, .resizable],
    backing: .buffered,
    defer: false
)
window.title = "Minha Aplicação macOS"

let contentView = NSView()
contentView.wantsLayer = true
contentView.layer?.backgroundColor = NSColor.white.cgColor
window.contentView = contentView

window.makeKeyAndOrderFront(nil)
```

#### Botões e Controles

```swift
let button = NSButton(title: "Clique Aqui", target: self, action: #selector(buttonClicked))
button.frame = NSRect(x: 50, y: 50, width: 100, height: 30)
contentView.addSubview(button)

let textField = NSTextField()
textField.placeholderString = "Digite algo..."
contentView.addSubview(textField)
```

### AppDelegate

Toda aplicação macOS tem um AppDelegate que gerencia o ciclo de vida:

```swift
import AppKit

class AppDelegate: NSObject, NSApplicationDelegate {
    var window: NSWindow?
    
    func applicationDidFinishLaunching(_ notification: Notification) {
        // Configurar a janela principal
        self.window = NSWindow()
        self.window?.makeKeyAndOrderFront(nil)
    }
    
    func applicationShouldTerminateAfterLastWindowClosed(_ sender: NSApplication) -> Bool {
        return true
    }
}
```

### Layouts em AppKit

AppKit usa AutoLayout para posicionamento de elementos:

```swift
let label = NSTextField()
label.stringValue = "Olá, macOS!"
label.translatesAutoresizingMaskIntoConstraints = false
contentView.addSubview(label)

NSLayoutConstraint.activate([
    label.centerXAnchor.constraint(equalTo: contentView.centerXAnchor),
    label.centerYAnchor.constraint(equalTo: contentView.centerYAnchor)
])
```

### Vantagens do AppKit

- ✅ Framework oficial e maduro
- ✅ Total controle sobre a interface
- ✅ Excelente performance
- ✅ Integração profunda com macOS
- ✅ Grande comunidade

### Desvantagens

- ❌ API verbosa comparada a SwiftUI
- ❌ Curva de aprendizado mais íngreme
- ❌ Menos documentação moderna que UIKit/SwiftUI

### AppKit vs SwiftUI

**AppKit:**
- Mais controle e customização
- APIs mais detalhadas
- Ideal para aplicações complexas

**SwiftUI:**
- Mais moderno e intuitivo
- Menos código
- Ideal para novos projetos

---

**Para mais informações:** [AppKit Documentation](https://developer.apple.com/documentation/appkit)
