# HTTPS e TLS

**HTTPS** (HyperText Transfer Protocol Secure) é o protocolo HTTP combinado com **TLS** (Transport Layer Security) para criptografar a comunicação entre cliente e servidor.

## Por que HTTPS é Importante?

- 🔒 **Confidencialidade** - Dados criptografados, não visíveis a terceiros
- ✅ **Integridade** - Detecta modificação de dados em trânsito
- 🆔 **Autenticação** - Verifica que está falando com o servidor correto
- 🛡️ **Proteção contra MITM** - Previne ataques man-in-the-middle

## TLS 1.2 vs TLS 1.3

- **TLS 1.2** - Estável, amplamente suportado
- **TLS 1.3** - Mais rápido, mais seguro (recomendado)

Apple recomenda TLS 1.2+. Versões antigas (SSL 3.0, TLS 1.0, 1.1) devem ser evitadas.

## Requisitos de Segurança de Transporte (ATS)

O iOS aplica automaticamente restrições de segurança:

```xml
<!-- Info.plist -->
<key>NSAppTransportSecurity</key>
<dict>
    <!-- Exigir HTTPS por padrão -->
    <key>NSAllowsArbitraryLoads</key>
    <false/>
    
    <!-- Exceções específicas (use raramente) -->
    <key>NSExceptionDomains</key>
    <dict>
        <key>exemplo.local</key>
        <dict>
            <key>NSIncludesSubdomains</key>
            <true/>
            <key>NSTemporaryExceptionAllowsInsecureHTTPLoads</key>
            <true/>
            <key>NSTemporaryExceptionMinimumTLSVersion</key>
            <string>TLSv1.2</string>
        </dict>
    </dict>
</dict>
```

**Importante:** Deixe ATS habilitado em produção. Desabilitar só para desenvolvimento local.

## Requisição HTTP Básica com HTTPS

```swift
import Foundation

func buscarDadosSeguro(url: URL) {
    var request = URLRequest(url: url)
    request.httpMethod = "GET"
    request.addValue("application/json", forHTTPHeaderField: "Content-Type")
    
    let session = URLSession.shared
    
    let task = session.dataTask(with: request) { data, response, error in
        if let error = error {
            print("Erro: \(error.localizedDescription)")
            return
        }
        
        // Verificar se é HTTPS
        if let httpResponse = response as? HTTPURLResponse {
            let protocolo = httpResponse.url?.scheme ?? "desconhecido"
            print("Protocolo: \(protocolo)")
            
            if protocolo != "https" {
                print("⚠️ Aviso: Não está usando HTTPS!")
                return
            }
        }
        
        if let data = data {
            print("Dados recebidos com segurança")
        }
    }
    
    task.resume()
}
```

## Certificate Pinning

Certificate Pinning garante que você está conectando ao servidor correto, mesmo se um certificado foi roubado.

### Pinning de Chave Pública

```swift
import Foundation

class PinningDelegate: NSObject, URLSessionDelegate {
    private let certificadoHash = "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=" // Hash base64
    
    func urlSession(_ session: URLSession,
                    didReceive challenge: URLAuthenticationChallenge,
                    completionHandler: @escaping (URLSession.AuthChallengeDisposition, URLCredential?) -> Void) {
        
        guard challenge.protectionSpace.authenticationMethod == NSURLAuthenticationMethodServerTrust,
              let serverTrust = challenge.protectionSpace.serverTrust else {
            completionHandler(.cancelAuthenticationChallenge, nil)
            return
        }
        
        // Validar certificado
        var secResult = SecTrustResultType.invalid
        let status = SecTrustEvaluate(serverTrust, &secResult)
        
        guard status == errSecSuccess else {
            completionHandler(.cancelAuthenticationChallenge, nil)
            return
        }
        
        // Extrair chave pública
        guard let certificate = SecTrustGetCertificateAtIndex(serverTrust, 0) else {
            completionHandler(.cancelAuthenticationChallenge, nil)
            return
        }
        
        // Comparar com certificado pinado
        if validarCertificado(certificate) {
            let credential = URLCredential(trust: serverTrust)
            completionHandler(.useCredential, credential)
        } else {
            completionHandler(.cancelAuthenticationChallenge, nil)
        }
    }
    
    private func validarCertificado(_ certificate: SecCertificate) -> Bool {
        // Implementar lógica de validação de certificado pinado
        return true // Simplificado
    }
}

// Uso
let config = URLSessionConfiguration.default
let delegate = PinningDelegate()
let session = URLSession(configuration: config, delegate: delegate, delegateQueue: nil)
```

### Usando a Biblioteca Alamofire (Terceiros)

```swift
import Alamofire

let certificateData = NSData(contentsOfFile: Bundle.main.path(forResource: "certificado", ofType: "cer")!)!

let serverTrustPolicy = ServerTrustPolicy.pinCertificates(
    certificates: [SecCertificateCreateWithData(nil, certificateData)!],
    validateCertificateChain: true,
    validateHost: true
)

let serverTrustPolicies = ["api.exemplo.com": serverTrustPolicy]
let manager = SessionManager(serverTrustPolicyManager: ServerTrustPolicyManager(policies: serverTrustPolicies))
```

## Headers de Segurança

Recomende ao servidor incluir estes headers:

```
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Content-Security-Policy: default-src 'self'
```

Você pode validar no client:

```swift
if let httpResponse = response as? HTTPURLResponse {
    if let hsts = httpResponse.value(forHTTPHeaderField: "Strict-Transport-Security") {
        print("✅ HSTS habilitado: \(hsts)")
    }
}
```

## Validação de Certificado Customizada

```swift
func validarCertificadoCustomizado(challenge: URLAuthenticationChallenge) -> URLCredential? {
    let serverTrust = challenge.protectionSpace.serverTrust
    let certificate = SecTrustGetCertificateAtIndex(serverTrust, 0)
    
    // 1. Validar cadeia de certificados
    var secResult = SecTrustResultType.invalid
    SecTrustEvaluate(serverTrust, &secResult)
    
    let isValido = secResult == .unspecified || secResult == .proceed
    guard isValido else { return nil }
    
    // 2. Validar hostname
    let hostname = challenge.protectionSpace.host
    guard validarHostname(hostname, com: certificate!) else { 
        return nil 
    }
    
    // 3. Validar data de expiração
    guard naoExpirou(certificate: certificate!) else { 
        return nil 
    }
    
    return URLCredential(trust: serverTrust)
}
```

## Boas Práticas

✅ **Recomendado:**
- Sempre usar HTTPS em produção
- Manter ATS ativado
- Implementar Certificate Pinning para APIs críticas
- Validar certificados e hostname
- Usar TLS 1.2 ou superior
- Implementar HSTS no servidor
- Revisar relativização de certifications periodicamente
- Usar URLSession.dataTask ao invés de requisições síncronas

❌ **Evitar:**
- Desabilitar ATS sem necessidade
- Usar HTTP para dados sensíveis
- Aceitar certificados auto-assinados em produção
- Ignorar erros de validação de certificado
- Enviar credenciais em query parameters
- Registrar dados sensíveis em logs

## Verificação de Certificado Expirado

```swift
func validarExpiracaoCertificado(trust: SecTrust) -> Bool {
    var result = SecTrustResultType.invalid
    let status = SecTrustEvaluate(trust, &result)
    
    guard status == errSecSuccess else { return false }
    
    // Verificar se não expirou
    let now = Date()
    // Certificado expirado resulta em resultado inválido
    
    return result == .unspecified || result == .proceed
}
```

---

Relacionados: [CryptoKit](CryptoKit.md) | [Segurança](Segurança.md)

**Status:** ✅ Verificado
**Última atualização:** Março de 2026
