# LocalAuthentication

**LocalAuthentication** é um framework Apple que permite usar autenticação biométrica (Face ID, Touch ID) e código de acesso para autenticar usuários localmente no dispositivo.

## Visão Geral

LocalAuthentication oferece:

- **Face ID** - Reconhecimento facial
- **Touch ID** - Reconhecimento de impressão digital
- **Código de Acesso** - Fallback para PIN/padrão do dispositivo
- **Integração segura** - Não requer envio de dados ao servidor

### Quando Usar

- Desbloquear recursos do app
- Confirmar ações sensíveis
- Autenticação local rápida
- Substituir autenticação por senha

**Importante:** LocalAuthentication não substitui a autenticação no servidor. Use para segurança local.

## Checando Disponibilidade

```swift
import LocalAuthentication

let context = LAContext()
var error: NSError?

// Verificar se o dispositivo suporta autenticação biométrica
let canEvaluate = context.canEvaluatePolicy(
    .deviceOwnerAuthenticationWithBiometrics,
    error: &error
)

if canEvaluate {
    // Verificar qual tipo de biometria está disponível
    switch context.biometryType {
    case .faceID:
        print("Face ID disponível")
    case .touchID:
        print("Touch ID disponível")
    case .none:
        print("Nenhuma biometria disponível")
    @unknown default:
        print("Tipo desconhecido")
    }
} else if let erro = error {
    print("Erro: \(erro.localizedDescription)")
}
```

## Autenticação Simples com Face ID / Touch ID

```swift
import LocalAuthentication

func autenticarComBiometria(motivo: String) {
    let context = LAContext()
    var error: NSError?
    
    // Verificar se é possível usar biometria
    guard context.canEvaluatePolicy(
        .deviceOwnerAuthenticationWithBiometrics,
        error: &error
    ) else {
        print("Biometria não disponível")
        return
    }
    
    // Executar autenticação
    context.evaluatePolicy(
        .deviceOwnerAuthenticationWithBiometrics,
        localizedReason: motivo
    ) { success, error in
        DispatchQueue.main.async {
            if success {
                print("✅ Autenticado com sucesso!")
                // Prosseguir com a ação
            } else if let error = error as? LAError {
                switch error.code {
                case .authenticationFailed:
                    print("❌ Falha na autenticação")
                case .userCancel:
                    print("❌ Usuário cancelou")
                case .userFallback:
                    print("❌ Usuário tapped fallback")
                case .systemCancel:
                    print("❌ Sistema cancelou")
                case .passcodeNotSet:
                    print("❌ Código de acesso não configurado")
                case .biometryNotAvailable:
                    print("❌ Biometria não disponível")
                case .biometryLocked:
                    print("❌ Biometria bloqueada (muitas tentativas)")
                case .biometryNotEnrolled:
                    print("❌ Biometria não registrada")
                default:
                    print("❌ Erro: \(error.localizedDescription)")
                }
            }
        }
    }
}

// Uso
autenticarComBiometria(motivo: "Necessário para acessar dados sensíveis")
```

## Autenticação com Fallback para Código de Acesso

```swift
import LocalAuthentication

func autenticarComBiometriaOuCodigo(motivo: String) {
    let context = LAContext()
    context.localizedFallbackTitle = "Usar Código de Acesso"
    
    var error: NSError?
    
    // Usar biometria se disponível, senão permitir fallback
    let policy: LAPolicy = .deviceOwnerAuthenticationWithBiometrics
    
    guard context.canEvaluatePolicy(policy, error: &error) else {
        // Se biometria não está disponível, tentar código de acesso
        let policyFallback: LAPolicy = .deviceOwnerAuthentication
        guard context.canEvaluatePolicy(policyFallback, error: &error) else {
            print("Nenhuma forma de autenticação disponível")
            return
        }
    }
    
    context.evaluatePolicy(.deviceOwnerAuthentication, 
                          localizedReason: motivo) { success, error in
        DispatchQueue.main.async {
            if success {
                print("✅ Autenticado com sucesso!")
            } else if let error = error as? LAError {
                print("❌ Falha: \(error.localizedDescription)")
            }
        }
    }
}
```

## Exemplo Completo em SwiftUI

```swift
import SwiftUI
import LocalAuthentication

@main
struct BiometriaApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}

struct ContentView: View {
    @State private var autenticado = false
    @State private var mensagem = ""
    
    var body: some View {
        VStack(spacing: 20) {
            if autenticado {
                Text("✅ Acesso Liberado!")
                    .font(.title)
                    .foregroundColor(.green)
                
                Button("Fazer Logout") {
                    autenticado = false
                    mensagem = ""
                }
                .buttonStyle(.bordered)
            } else {
                Text("🔐 Autenticação Biométrica")
                    .font(.title2)
                
                Button("Autenticar") {
                    autenticarComBiometria()
                }
                .buttonStyle(.borderedProminent)
                
                if !mensagem.isEmpty {
                    Text(mensagem)
                        .foregroundColor(.red)
                        .font(.caption)
                }
            }
        }
        .padding()
    }
    
    private func autenticarComBiometria() {
        let context = LAContext()
        var error: NSError?
        
        guard context.canEvaluatePolicy(.deviceOwnerAuthentication, error: &error) else {
            mensagem = "Autenticação não disponível"
            return
        }
        
        context.evaluatePolicy(.deviceOwnerAuthentication, 
                              localizedReason: "Acesse sua conta privada") 
        { success, erro in
            DispatchQueue.main.async {
                if success {
                    autenticado = true
                    mensagem = ""
                } else {
                    mensagem = "Autenticação falhou"
                }
            }
        }
    }
}
```

## Tipos de Erro (LAError.Code)

| Erro | Descrição |
|------|-----------|
| `.authenticationFailed` | A autenticação falhou (impressão não reconhecida) |
| `.userCancel` | O usuário cancelou a autenticação |
| `.userFallback` | Usuário selecionou fallback (código de acesso) |
| `.systemCancel` | O sistema cancelou (ex: outro app abriu) |
| `.passcodeNotSet` | Dispositivo sem código de acesso configurado |
| `.biometryNotAvailable` | Biometria não disponível no dispositivo |
| `.biometryLocked` | Muitas tentativas falhas, bloqueado temporariamente |
| `.biometryNotEnrolled` | Nenhuma biometria registrada no dispositivo |

## Boas Práticas

✅ **Recomendado:**
- Sempre verificar `canEvaluatePolicy` antes de chamar `evaluatePolicy`
- Usar `DispatchQueue.main.async` para atualizar a UI
- Fornecer mensagens de erro claras ao usuário
- Oferecer fallback para código de acesso
- Testar em dispositivo real (simulador tem limitações)
- Armazenar dados sensíveis no Keychain após autenticação

❌ **Evitar:**
- Confiar apenas em LocalAuthentication para segurança
- Ignorar erros de biometria
- Usar em thread UI principal
- Armazenar credenciais após autenticação bem-sucedida
- Chamar `evaluatePolicy` repetidamente sem verificar

## Integração com Keychain

Padrão comum: Usar LocalAuthentication para desbloquear acesso ao Keychain

```swift
import LocalAuthentication

func acessarDadosSensivel(chave: String) {
    let context = LAContext()
    
    context.evaluatePolicy(.deviceOwnerAuthentication, 
                          localizedReason: "Necessário para acessar dados") 
    { success, _ in
        if success {
            // Agora pode acessar dados do Keychain
            let dados = KeychainHelper.standard.recuperar(chave)
            print("Dados: \(dados ?? "")")
        }
    }
}
```

## Suporte de Plataforma

- **iOS:** 8.0+
- **macOS:** 10.12.1+
- **tvOS:** 10.0+
- **watchOS:** 3.0+

Face ID: iOS 11.0+
Touch ID: iOS 8.0+

---

Relacionados: [Keychain](Keychain.md) | [CryptoKit](CryptoKit.md) | [Segurança](Segurança.md)

**Status:** ✅ Verificado
**Última atualização:** Março de 2026
