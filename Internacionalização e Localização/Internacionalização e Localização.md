# Internacionalização e Localização em Swift

**Internacionalização (i18n)** e **Localização (l10n)** são processos complementares que permitem adaptar aplicativos para diferentes idiomas, regiões e culturas, oferecendo uma experiência personalizada aos usuários globais.

## Diferenças Entre Internacionalização e Localização

- **Internacionalização (i18n):** O processo técnico de preparar um aplicativo para suportar múltiplos idiomas e regiões. Envolve remover idioma hardcoded, usar recursos localizáveis e seguir convenções regionais.
- **Localização (l10n):** O processo de traduzir conteúdo e adaptar o aplicativo para um idioma/região específica.

## Strings Localizáveis em Swift

### Usando NSLocalizedString

A forma mais comum de marcar strings para localização:

```swift
import Foundation

// Forma básica
let greeting = NSLocalizedString("Hello, World!", comment: "Greeting message")

// Com table
let message = NSLocalizedString("Welcome", tableName: "Main", bundle: Bundle.main, value: "Welcome", comment: "Welcome message")

// Com argumentos
let formatted = String(format: NSLocalizedString("You have %d messages", comment: "Message count"), count: 5)
```

**Boas Práticas:**
- Sempre adicione um `comment` descritivo
- Use `tableName` para organizar strings em arquivos separados
- Evite concatenar strings — use formatação

### SwiftUI com Localização

Em SwiftUI, utilize localized string literals:

```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        VStack {
            Text("Hello, World!")  // Automaticamente localizável
            Text(LocalizedStringKey("greeting"))
        }
    }
}

// Com formatação
struct LocalizedText: View {
    var count: Int
    var body: some View {
        Text("You have \(count) items")
            .environment(\.locale, Locale(identifier: "pt_BR"))
    }
}
```

## Arquivos de Localização (.strings)

### Estrutura de Projeto

```
MyApp/
├── en.lproj/
│   └── Localizable.strings
├── pt-BR.lproj/
│   └── Localizable.strings
├── es.lproj/
│   └── Localizable.strings
└── Sources/
```

### Criando Arquivo Localizable.strings

**en.lproj/Localizable.strings:**
```
"greeting" = "Hello, World!";
"farewell" = "Goodbye!";
"welcome_message" = "Welcome, %@";
"message_count" = "You have %d messages";
```

**pt-BR.lproj/Localizable.strings:**
```
"greeting" = "Olá, Mundo!";
"farewell" = "Até logo!";
"welcome_message" = "Bem-vindo, %@";
"message_count" = "Você tem %d mensagens";
```

## Format Specifiers Comuns

| Especificador | Tipo | Exemplo |
|---|---|---|
| `%@` | String | `"User: %@"` → `"User: John"` |
| `%d` | Integer | `"Count: %d"` → `"Count: 42"` |
| `%f` | Float | `"Price: %.2f"` → `"Price: 19.99"` |
| `%lld` | Long Long | `"ID: %lld"` → `"ID: 123456789"` |

## Formato de Data, Hora e Número por Região

### DateFormatter com Localização

```swift
import Foundation

let date = Date()

// Formato automático para a locale atual
let dateFormatter = DateFormatter()
dateFormatter.locale = Locale.current  // ou Locale(identifier: "pt_BR")
dateFormatter.dateStyle = .long  // "March 12, 2026" (EN) ou "12 de março de 2026" (PT)
dateFormatter.timeStyle = .medium

let formatted = dateFormatter.string(from: date)
print(formatted)
```

### NumberFormatter com Localização

```swift
let formatter = NumberFormatter()
formatter.locale = Locale(identifier: "pt_BR")
formatter.numberStyle = .currency
formatter.currencyCode = "BRL"

let price = 99.99
if let formatted = formatter.string(from: NSNumber(value: price)) {
    print(formatted)  // "R$ 99,99"
}
```

### Pluralização com CLDR (Unicode CLDR)

Para mensagens que variam com quantidade:

```swift
// Usando stringsdict (iOS 8+)
// Localizable.stringsdict

// Para PT-BR
/*
<dict>
  <key>items_count</key>
  <dict>
    <key>NSStringFormatSpecTypeKey</key>
    <string>NSStringPluralRuleType</string>
    <key>NSStringFormatValueTypeKey</key>
    <string>d</string>
    <key>one</key>
    <string>%d item</string>
    <key>other</key>
    <string>%d itens</string>
  </dict>
</dict>
*/

let count = 5
let format = NSLocalizedString("items_count", comment: "Items count")
let message = String.localizedStringWithFormat(format, count)
```

## Teste de Localização em Xcode

### Schemo para Pseudo-Idiomas

Xcode oferece pseudo-idiomas para teste:
1. **Edit Scheme** → **Run** → **Arguments Passed On Launch**
2. Configure com argumentos como:
   - `-AppleLanguages (en)` — English
   - `-AppleLanguages (pt_BR)` — Português Brasileiro
   - `-AppleLanguages (en-Latn-x-accented)` — Teste de acentos
   - `-AppleLanguages (en-Long)` — Simula textos longos

### Teste de Linguagem

```bash
# Executar com linguagem específica
xcrun simctl spawn booted defaults write com.example.App AppleLanguages -array pt-BR
```

## Diretórios de Recursos por Região

### Imagens e Ativos Localizados

```swift
// Para imagens específicas por região
let image = UIImage(named: "greeting_pt_BR") ?? UIImage(named: "greeting")

// Ou usando asset catalogs com localization
```

## Boas Práticas de Localização

1. **Planeje desde o Início:** Não adicione localização no final do projeto. Estruture desde o início.
2. **Use Chaves Descritivas:** `"btn_save"` ao invés de `"save"` para evitar conflitos.
3. **Contexto para Tradutores:** Adicione `comment` para descrever onde e como a string é usada.
4. **Teste com Múltiplos Idiomas:** Não assuma que o comprimento das strings será o mesmo em todas as línguas.
5. **Evite Hardcoding:** Nunca coloque texto na interface diretamente.
6. **Pluralização Correta:** Use `.stringsdict` para pluralização em vez de concatenação.
7. **Formatação Segura:** Use `String(format:)` com especificadores seguros ao invés de concatenação.

## Recursos Adicionais

- [Apple: Internationalization and Localization Guide](https://developer.apple.com/documentation/xcode/localizing-and-varying-content)
- [NSLocalizedString Documentation](https://developer.apple.com/documentation/foundation/nslocalizedstring)
- [Unicode CLDR Plural Rules](http://cldr.unicode.org/index/cldr-spec/plural-rules)
- [Xcode Localization Help](https://help.apple.com/xcode/mac/current/#/deve8a28fcf0)

## Exemplo Completo: App com I18n

```swift
import SwiftUI

// MARK: - Localization Helper
struct L10n {
    static let greeting = NSLocalizedString("greeting", comment: "Main greeting")
    static let farewell = NSLocalizedString("farewell", comment: "Farewell message")
    
    static func items(count: Int) -> String {
        String(format: NSLocalizedString("items_count", comment: "Item count"), count)
    }
}

// MARK: - View
struct ContentView: View {
    @State private var itemCount = 5
    
    var body: some View {
        VStack(spacing: 20) {
            Text(L10n.greeting)
                .font(.title)
            
            Text(L10n.items(count: itemCount))
                .font(.body)
            
            Button(L10n.farewell) {
                // Action
            }
        }
        .padding()
    }
}
```

**Relacionados:** [Links e Materiais](etc/Links%20e%20Materiais.md) [Acessibilidade](Acessibilidade/Acessibilidade.md)
