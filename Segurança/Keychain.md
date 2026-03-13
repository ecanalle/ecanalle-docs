# Keychain

**Keychain** é o sistema de gerenciamento de credenciais do iOS, macOS e outros sistemas Apple. É o lugar seguro e criptografado para armazenar senhas, tokens, certificados e outros dados sensíveis.

## Por que usar Keychain?

O Keychain oferece:

- **Criptografia automática** - Dados são criptografados no disco
- **Proteção de acesso** - Controle fino sobre quem pode acessar os dados
- **Sincronização iCloud** - Dados podem sincronizar entre dispositivos (com permissão do usuário)
- **Proteção contra ataques** - Resistência contra reversão de engenharia
- **Interface segura** - Integrada ao sistema operacional

### ❌ Nunca use para dados sensíveis:

- `UserDefaults` - Texto claro, facilmente acessível
- Variáveis globais - Acessíveis na memória
- Arquivos no Documents - Texto claro (exceto com encriptação customizada)

## Operações Básicas com Keychain no iOS

### Estrutura Simples com GenericPassword

```swift
import Foundation

// MARK: - Salvar dados no Keychain

func salvarNoKeychain(chave: String, valor: String) -> Bool {
    guard let dados = valor.data(using: .utf8) else { return false }
    
    let query: [String: Any] = [
        kSecClass as String: kSecClassGenericPassword,
        kSecAttrAccount as String: chave,
        kSecValueData as String: dados
    ]
    
    // Deletar se já existe
    SecItemDelete(query as CFDictionary)
    
    // Adicionar novo
    let status = SecItemAdd(query as CFDictionary, nil)
    return status == errSecSuccess
}

// MARK: - Recuperar dados do Keychain

func recuperarDoKeychain(chave: String) -> String? {
    let query: [String: Any] = [
        kSecClass as String: kSecClassGenericPassword,
        kSecAttrAccount as String: chave,
        kSecReturnData as String: true
    ]
    
    var resultado: AnyObject?
    let status = SecItemCopyMatching(query as CFDictionary, &resultado)
    
    guard status == errSecSuccess,
          let dados = resultado as? Data,
          let valor = String(data: dados, encoding: .utf8) else {
        return nil
    }
    
    return valor
}

// MARK: - Deletar dados do Keychain

func deletarDoKeychain(chave: String) -> Bool {
    let query: [String: Any] = [
        kSecClass as String: kSecClassGenericPassword,
        kSecAttrAccount as String: chave
    ]
    
    let status = SecItemDelete(query as CFDictionary)
    return status == errSecSuccess
}

// MARK: - Uso

let token = "abc123xyz789"
salvarNoKeychain(chave: "authToken", valor: token)

if let tokenRecuperado = recuperarDoKeychain(chave: "authToken") {
    print("Token: \(tokenRecuperado)")
}
```

## Usando KeychainHelper (Wrapper Simplificado)

Para código mais limpo, crie um wrapper:

```swift
import Foundation

struct KeychainHelper {
    static let standard = KeychainHelper()
    
    private let service = Bundle.main.bundleIdentifier ?? "com.app.keychain"
    
    // MARK: - Salvar
    
    func salvar(_ valor: String, para chave: String) throws {
        guard let dados = valor.data(using: .utf8) else {
            throw NSError(domain: "Erro", code: -1, userInfo: nil)
        }
        
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrService as String: service,
            kSecAttrAccount as String: chave,
            kSecValueData as String: dados,
            kSecAttrAccessible as String: kSecAttrAccessibleWhenUnlockedThisDeviceOnly
        ]
        
        SecItemDelete(query as CFDictionary)
        
        let status = SecItemAdd(query as CFDictionary, nil)
        guard status == errSecSuccess else {
            throw NSError(domain: "Keychain", code: Int(status), userInfo: nil)
        }
    }
    
    // MARK: - Recuperar
    
    func recuperar(_ chave: String) -> String? {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrService as String: service,
            kSecAttrAccount as String: chave,
            kSecReturnData as String: true
        ]
        
        var resultado: AnyObject?
        let status = SecItemCopyMatching(query as CFDictionary, &resultado)
        
        guard status == errSecSuccess,
              let dados = resultado as? Data,
              let valor = String(data: dados, encoding: .utf8) else {
            return nil
        }
        
        return valor
    }
    
    // MARK: - Deletar
    
    func deletar(_ chave: String) -> Bool {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrService as String: service,
            kSecAttrAccount as String: chave
        ]
        
        let status = SecItemDelete(query as CFDictionary)
        return status == errSecSuccess
    }
}

// MARK: - Uso

let helper = KeychainHelper.standard

do {
    try helper.salvar("meu_token_secreto", para: "authToken")
    
    if let token = helper.recuperar("authToken") {
        print("Token resgatado: \(token)")
    }
    
    helper.deletar("authToken")
} catch {
    print("Erro no Keychain: \(error)")
}
```

## Níveis de Proteção (Accessibility)

O Keychain oferece diferentes níveis de proteção:

```swift
// Acessível apenas quando o dispositivo está desbloqueado
kSecAttrAccessibleWhenUnlockedThisDeviceOnly

// Acessível quando o dispositivo está desbloqueado (sincroniza com iCloud)
kSecAttrAccessibleWhenUnlocked

// Sempre acessível (menos seguro)
kSecAttrAccessibleAlways
```

**Recomendado:** Use `kSecAttrAccessibleWhenUnlockedThisDeviceOnly` para máxima segurança.

## Dados Sensíveis Comuns para Armazenar

- 🔐 **Tokens de autenticação** (JWT, OAuth tokens)
- 🔐 **Senhas** (raramente, melhor usar Face ID/Touch ID)
- 🔐 **Chaves API privadas**
- 🔐 **Certificados** (cliente)
- 🔐 **Dados biométricos** (não armazenar, apenas usar para autenticação)

## Sincronização com iCloud

```swift
// Permitir sincronização com iCloud
let query: [String: Any] = [
    kSecClass as String: kSecClassGenericPassword,
    kSecAttrAccount as String: chave,
    kSecValueData as String: dados,
    kSecAttrSynchronizable as String: true  // iCloud sync
]
```

## Restauração de Dados após Reinstalação

O Keychain **não sincroniza automaticamente** quando você desinstala e reinstala o app, a menos que tenha ativado iCloud Keychain explicitamente.

Considere fazer backup de dados críticos em sua backend de forma segura.

## Boas Práticas

✅ **Recomendado:**
- Use CryptoKit para dados extra-sensíveis que requerem controle de acesso mais fino
- Sempre defina um nível apropriado de proteção (Accessibility)
- Delete dados do Keychain quando fazer logout
- Trate erros de Keychain explicitamente
- Use serviço identificado para evitar conflitos

❌ **Evitar:**
- Armazenar senhas em plain text em UserDefaults
- Compartilhar dados do Keychain entre apps (diferente entitlements)
- Ignorar erros de acesso ao Keychain
- Usar `kSecAttrAccessibleAlways` sem motivo séria

## Bibliotecas de Terceiros

Se preferir um wrapper mais completo:
- **Locksmith** - Wrapper simples para Keychain
- **Valet** - Wrapper type-safe para Keychain
- **KeychainAccess** - Ponte simples com Swift

---

Relacionados: [CryptoKit](CryptoKit.md) | [LocalAuthentication](LocalAuthentication.md) | [Segurança](Segurança.md)

**Status:** ✅ Verificado
**Última atualização:** Março de 2026
