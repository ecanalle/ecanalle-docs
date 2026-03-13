# Autenticação

**Autenticação** é o processo de verificar a identidade de um usuário ou sistema. É diferente de **Autorização**, que determina o que um usuário autenticado pode fazer.

## Tipos de Autenticação

### Autenticação por Senha

O método mais antigo. Requer:
- Hash seguro (bcrypt, Argon2, PBKDF2)
- Sal (salt) aleatório para cada usuário
- Proteção contra força bruta

```swift
// ❌ ERRADO: Nunca faça isto
let senha = "senha123"
if usuario.senha == senha {
    print("Autenticado")
}

// ✅ CORRETO: Usar hash seguro
import CryptoKit

func verificarSenha(_ senha: String, com hash: String) -> Bool {
    // Usar biblioteca como bcrypt do lado servidor
    // Nunca implementar hashing de senha no cliente!
    return false
}
```

### OAuth 2.0

Protocolo para autenticação/autorização delegada. Comum em "Login with Google", "Login with Apple", etc.

**Fluxo:**
1. App redireciona para provedor (Google, Apple, etc)
2. Usuário autentica com provedor
3. Provedor retorna código/token ao app
4. App troca código por token de acesso
5. App usa token para acessar recursos

```swift
// Pseudo código
import AuthenticationServices

func fazerLoginComApple() {
    let request = ASAuthorizationAppleIDProvider().createRequest()
    request.requestedScopes = [.fullName, .email]
    
    let controller = ASAuthorizationController(authorizationRequests: [request])
    controller.delegate = self
    controller.performRequests()
}

// Receber resultado
extension MeuController: ASAuthorizationControllerDelegate {
    func authorizationController(controller: ASAuthorizationController,
                               didCompleteWithAuthorization authorization: ASAuthorization) {
        if let credential = authorization.credential as? ASAuthorizationAppleIDCredential {
            let usuarioId = credential.user
            let email = credential.email
            let nomeCompleto = credential.fullName
            
            // Enviar para servidor para validação
            verificarComServidor(usuarioId: usuarioId)
        }
    }
}
```

### JWT (JSON Web Tokens)

Token auto-contido que pode ser verificado sem consultar o servidor. Útil para autenticação stateless.

**Estrutura:** `header.payload.signature`

```swift
// Exemplo de JWT
let token = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c"

// Decodificar JWT (sem validar assinatura)
func decodificarJWT(_ token: String) -> [String: Any]? {
    let partes = token.split(separator: ".")
    guard partes.count == 3 else { return nil }
    
    let payloadString = String(partes[1])
    let padding = String(repeating: "=", count: (4 - payloadString.count % 4) % 4)
    let base64String = payloadString.replacingOccurrences(of: "-", with: "+")
                                    .replacingOccurrences(of: "_", with: "/") + padding
    
    guard let dados = Data(base64Encoded: base64String),
          let json = try? JSONSerialization.jsonObject(with: dados) as? [String: Any] else {
        return nil
    }
    
    return json
}

// Validar assinatura JWT (deve ser feito no servidor!)
func validarJWT(_ token: String, comChave chave: String) -> Bool {
    // Implementação completa requer HMAC
    // Em produção, validar no servidor/backend
    return false
}
```

### Autenticação Multifator (MFA / 2FA)

Requer múltiplos fatores de autenticação:
- **Algo que você sabe** - Senha, PIN
- **Algo que você tem** - Telefone, token
- **Algo que você é** - Biometria (Face ID, Touch ID)

```swift
// Exemplo: Biometria + Senha

class AutenticadorMFA {
    func autenticar(email: String, senha: String) async throws {
        // 1. Verificar senha com servidor
        let resultadoSenha = try await verificarSenhaComServidor(email: email, senha: senha)
        guard resultadoSenha else { throw AutenticacaoError.senhaInvalida }
        
        // 2. Requer biometria local
        try await verificarBiometria("Confirme sua identidade")
        
        // 3. Gerar e armazenar token
        let token = try await obterTokenDoServidor()
        try KeychainHelper.standard.salvar(token, para: "authToken")
    }
    
    private func verificarBiometria(_ motivo: String) async throws {
        let context = LAContext()
        
        return try await withCheckedThrowingContinuation { continuation in
            context.evaluatePolicy(.deviceOwnerAuthentication, 
                                  localizedReason: motivo) { success, error in
                if success {
                    continuation.resume()
                } else {
                    continuation.resume(throwing: error ?? AutenticacaoError.biometriaFalhou)
                }
            }
        }
    }
}

enum AutenticacaoError: Error {
    case senhaInvalida
    case biometriaFalhou
    case tokenInvalido
}
```

## Fluxo de Login Seguro

```swift
class LoginManager {
    func fazerLogin(email: String, senha: String) async throws {
        // 1. Validar entrada
        guard !email.isEmpty && !senha.isEmpty else {
            throw LoginError.entradaInvalida
        }
        
        // 2. Usar HTTPS para enviar credenciais
        var request = URLRequest(url: URL(string: "https://api.exemplo.com/login")!)
        request.httpMethod = "POST"
        request.addValue("application/json", forHTTPHeaderField: "Content-Type")
        
        let body = ["email": email, "password": senha]
        request.httpBody = try JSONEncoder().encode(body)
        
        // 3. Receber tokens
        let (dados, response) = try await URLSession.shared.data(for: request)
        
        guard (response as? HTTPURLResponse)?.statusCode == 200 else {
            throw LoginError.credenciaisInvalidas
        }
        
        // 4. Decodificar resposta
        let loginResponse = try JSONDecoder().decode(LoginResponse.self, from: dados)
        
        // 5. Armazenar tokens com segurança
        try KeychainHelper.standard.salvar(loginResponse.accessToken, para: "accessToken")
        try KeychainHelper.standard.salvar(loginResponse.refreshToken, para: "refreshToken")
        
        UserDefaults.standard.set(loginResponse.usuarioId, forKey: "usuarioId")
    }
    
    func fazerLogout() {
        KeychainHelper.standard.deletar("accessToken")
        KeychainHelper.standard.deletar("refreshToken")
        UserDefaults.standard.removeObject(forKey: "usuarioId")
    }
}

struct LoginResponse: Codable {
    let accessToken: String
    let refreshToken: String
    let usuarioId: String
}

enum LoginError: Error {
    case entradaInvalida
    case credenciaisInvalidas
    case rede
}
```

## Renovação de Token (Token Refresh)

```swift
class TokenManager {
    func renovarAccessToken() async throws -> String {
        guard let refreshToken = KeychainHelper.standard.recuperar("refreshToken") else {
            throw TokenError.refreshTokenNotFound
        }
        
        var request = URLRequest(url: URL(string: "https://api.exemplo.com/refresh")!)
        request.httpMethod = "POST"
        request.addValue("application/json", forHTTPHeaderField: "Content-Type")
        request.addValue("Bearer \(refreshToken)", forHTTPHeaderField: "Authorization")
        
        let (dados, _) = try await URLSession.shared.data(for: request)
        let resposta = try JSONDecoder().decode(RefreshResponse.self, from: dados)
        
        // Armazenar novo token
        try KeychainHelper.standard.salvar(resposta.accessToken, para: "accessToken")
        
        return resposta.accessToken
    }
}

struct RefreshResponse: Codable {
    let accessToken: String
}

enum TokenError: Error {
    case refreshTokenNotFound
    case renovacaoFalhou
}
```

## Armazenamento de Credenciais

✅ **Fazer:**
- Armazenar tokens no Keychain
- Usar HTTPS para transmissão
- Implementar expiração de tokens
- Usar refresh tokens

❌ **Não fazer:**
- Armazenar credenciais em UserDefaults
- Armazenar senhas no app
- Transmitir sobre HTTP
- Incluir credenciais em logs

## Boas Práticas

✅ **Recomendado:**
- Sempre usar HTTPS
- Implementar MFA quando possível
- Usar biometria como fator adicional
- Armazenar tokens no Keychain
- Implementar timeout de sessão
- Usar certificados pinados para APIs críticas
- Fazer logout ao desinstalar o app

❌ **Evitar:**
- Armazenar senhas no cliente
- Implementar seu próprio sistema de autenticação
- Usar cookies para APIs mobile
- Enviar credenciais em parâmetros de URL
- Ignorar expiração de tokens

---

Relacionados: [LocalAuthentication](LocalAuthentication.md) | [Keychain](Keychain.md) | [HTTPS e TLS](HTTPS%20e%20TLS.md) | [Segurança](Segurança.md)

**Status:** ✅ Verificado
**Última atualização:** Março de 2026
