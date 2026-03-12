# Swift Package Manager (SPM)

**Swift Package Manager** é o gerenciador oficial de dependências do Swift, mantido diretamente pela Apple. É integrado nativamente no Swift e no Xcode, eliminando a necessidade de ferramentas externas como CocoaPods ou Carthage.

### Características Principais

- **Nativo:** Parte oficial do Swift desde a versão 3.0.
- **Integrado:** Funciona perfeitamente com Xcode.
- **Simples:** Sintaxe clara e direta em Swift (sem Ruby ou outras linguagens).
- **Multiplataforma:** Funciona em iOS, macOS, watchOS, tvOS e Linux.
- **Rápido:** Compilação eficiente e sem overhead.
- **Descentralizado:** Usa Git como base, sem servidor central obrigatório.

### Criando um Package

```bash
mkdir MeuPackage
cd MeuPackage
swift package init --type library
```

Isso cria a estrutura:

```
MeuPackage/
├── Package.swift
├── Sources
│   └── MeuPackage
│       └── MeuPackage.swift
└── Tests
    └── MeuPackageTests
        └── MeuPackageTests.swift
```

### Arquivo Package.swift

```swift
// swift-tools-version:5.5
import PackageDescription

let package = Package(
    name: "MeuPackage",
    platforms: [
        .iOS(.v13),
        .macOS(.v10_15)
    ],
    products: [
        .library(
            name: "MeuPackage",
            targets: ["MeuPackage"]
        )
    ],
    dependencies: [
        .package(url: "https://github.com/Alamofire/Alamofire.git", from: "5.0.0")
    ],
    targets: [
        .target(
            name: "MeuPackage",
            dependencies: ["Alamofire"]
        ),
        .testTarget(
            name: "MeuPackageTests",
            dependencies: ["MeuPackage"]
        )
    ]
)
```

### Adicionando SPM a um Projeto Xcode

#### Método 1: Via Xcode

1. **File → Add Packages**
2. Cole a URL do repositório Git
3. Escolha a versão/branch
4. Selecione targets e clique em **Add Package**

#### Método 2: Via Terminal (Xcode 13+)

```bash
xcode-select --install  # Se necessário
```

### Exemplo de Uso

```swift
import MeuPackage
import Alamofire

let cliente = APIClient()
cliente.buscarDados()
```

### Versionamento em SPM

SPM suporta múltiplas estratégias de versionamento:

```swift
// Versão exata
.package(url: "https://...", exact: "1.0.0")

// A partir de uma versão
.package(url: "https://...", from: "1.0.0")

// Range
.package(url: "https://...", "1.0.0"..<"2.0.0")

// Branch específica
.package(url: "https://...", branch: "main")

// Commit específico
.package(url: "https://...", revision: "abc123...")
```

### Vantagens do SPM

- ✅ Suporte oficial da Apple
- ✅ Sem dependências externas
- ✅ Sintaxe Swift clara
- ✅ Integração perfeita com Xcode
- ✅ Rápido e eficiente
- ✅ Crescimento exponencial de suporte

### Migração para SPM

A maioria dos packages agora suporta SPM. Se uma biblioteca que você usa não suporta, você pode:

1. Fazer um request na comunidade
2. Submeter um PR ao projeto
3. Usar CocoaPods/Carthage como fallback

---

**Para mais informações:** [Swift.org Package Manager](https://swift.org/package-manager/)
