# Carthage em Swift

**Carthage** é um gerenciador de dependências descentralizado para projetos Cocoa (iOS, macOS, tvOS e watchOS). Ao contrário do CocoaPods, Carthage não altera a estrutura do seu projeto nem requer integração com o build system. Em vez disso, Carthage constrói as dependências de forma independente e deixa você integrar os frameworks compilados manualmente.

### Características Principais

- **Descentralizado:** Carthage não requer um servidor central, funcionando diretamente com repositórios Git.
- **Não invasivo:** Não modifica seu projeto Xcode, deixando você no controle total da configuração.
- **Rápido:** Carthage constrói os frameworks de dependência de forma paralela.
- **Flexível:** Suporta múltiplas plataformas e permite controle fino sobre versões.
- **Simples:** Usa um arquivo `Cartfile` para especificar dependências.

### Instalação

Cartbage pode ser instalado via Homebrew:

```bash
brew install carthage
```

### Começando com Carthage

#### 1. Criar um arquivo `Cartfile`

Na raiz do seu projeto, crie um arquivo chamado `Cartfile`:

```
github "Alamofire/Alamofire" ~> 5.0
github "ReactiveX/RxSwift" ~> 6.0
```

#### 2. Instalar as dependências

```bash
carthage update --platform iOS
```

#### 3. Integrar aos targets

Arraste os frameworks gerados (em `Carthage/Build/iOS`) para o seu target no Xcode.

### Exemplo de Uso

```swift
import Alamofire

AF.request("https://api.exemplo.com/dados")
    .responseJSON { response in
        print(response)
    }
```

### Vantagens do Carthage

- ✅ Maior controle sobre o projeto
- ✅ Build mais rápido para projetos grandes
- ✅ Não requer configuração de workspace
- ✅ Compatível com Swift Package Manager

### Comparação com outras ferramentas

| Aspecto | Carthage | CocoaPods | SPM |
|---------|----------|-----------|-----|
| Invasivo | Não | Sim | Não |
| Setup | Simples | Complexo | Muito simples |
| Suporte | Descentralizado | Centralizado | Apple |

---

**Para mais informações:** [Carthage Documentation](https://github.com/Carthage/Carthage)
