# OWASP Top 10

O **OWASP Top 10** é uma lista das 10 vulnerabilidades de segurança mais críticas em aplicações web e mobile, mantida pela Open Web Application Security Project.

## 1. Injection (Injeção)

**Descrição:** Código malicioso é injetado em comandos ou consultas interpretados.

### Exemplos:
- **SQL Injection** - Injetar código SQL
- **Code Injection** - Injetar código JavaScript/Swift
- **Command Injection** - Injetar comandos do sistema

### ❌ Vulnerable:
```swift
let userInput = "'; DROP TABLE usuarios; --"
let query = "SELECT * FROM usuarios WHERE email = '\(userInput)'"
// Resultado: Querymalicioso deleta a tabela!
```

### ✅ Seguro:
```swift
import SQLite

// Usar prepared statements
let usuarios = db.table("usuarios").filter(email == userInput)

// Ou usar frameworks como Realm/Core Data que evitam injeção
let request: NSFetchRequest<Usuario> = Usuario.fetchRequest()
request.predicate = NSPredicate(format: "email == %@", userInput)
```

## 2. Broken Authentication (Autenticação Quebrada)

**Descrição:** Falhas em implementação de autenticação falham em proteger contas.

### Problemas Comuns:
- Senhas fracas ou sem proteção
- Sessões não expiram
- Credenciais em URLs ou logs

### ✅ Práticas Seguras:
```swift
// Usar biometria
LocalAuthentication().authenticate()

// Ou OAuth/OpenID
AuthenticationServices().authorize()

// Expirar sessões
Timer.scheduledTimer(withTimeInterval: 3600, repeats: false) { _ in
    fazerLogout()
}
```

## 3. Sensitive Data Exposure (Exposição de Dados Sensíveis)

**Descrição:** Dados sensíveis são expostos inadvertidamente.

### Problemas:
- Armazenar dados sensíveis em texto claro
- Transmitir sem HTTPS
- Expor em logs

### ✅ Proteção:
```swift
// ✅ Usar Keychain para dados sensíveis
try KeychainHelper.standard.salvar(token, para: "authToken")

// ✅ Sempre usar HTTPS
https://api.exemplo.com/usuarios

// ✅ Remover dados sensíveis do logs
print("Login para: \(email)") // ✓ OK
print("Senha: \(senha)") // ✗ Nunca
```

## 4. XML External Entities (XXE)

**Descrição:** Processandor XML não seguro permite acesso a recursos do sistema.

### Swift raramente é afetado, mas cuidado com:
```swift
// ❌ Evitar parsear XML de fontes não confiáveis
let xmlParser = XMLParser(data: xmlData)

// ✅ Usar bibliotecas seguras e validar entrada
guard xmlData.count < 1000000 else { return } // Limitar tamanho
```

## 5. Broken Access Control (Controle de Acesso Quebrado)

**Descrição:** Usuários podem acessar recursos que não deveriam.

### Problemas:
- Sem verificação de autorização
- Modificar token/ID de usuário
- Endpoints não protegidos

### ✅ Práticas Seguras:
```swift
// ✅ Validar permissões no servidor
func buscarDadosUsuario(para usuarioId: String) async throws {
    // Sempre verificar no servidor se usuário autenticado tem permissão
    let dados = try await servidor.buscar("/usuarios/\(usuarioId)")
    
    // Validação no cliente é bônus, nunca a única proteção
    guard let meusDados = try obterMeusDados(), meusDados.id == usuarioId else {
        throw AccessError.acesNãoPermitido
    }
    
    return dados
}
```

## 6. Security Misconfiguration (Configuração Insegura)

**Descrição:** Configuração insegura do app, servidor ou framework.

### Problemas em Mobile:
- Debug habilitado em produção
- ATS desabilitado desnecessariamente
- Dependências com vulnerabilidades conhecidas

### ✅ Práticas:
```swift
import os

// ✅ Debug apenas em development
#if DEBUG
    os_log("Mensagem de debug", type: .debug)
#endif

// ✅ Manter ATS ativado (padrão)
// Info.plist: NSAllowsArbitraryLoads = false

// ✅ Atualizar dependências regularmente
// Usar ferramentas: SPM, CocoaPods, Carthage
```

## 7. Cross-Site Scripting (XSS)

**Descrição:** Código malicioso executado no navegador/app do usuário.

### Raro em apps Swift nativas, maior risco em WebViews:
```swift
// ❌ VULNERÁVEL: Carregar HTML não confiável
let html = """
<img src=x onerror="alert('XSS')">
"""
webView.loadHTMLString(html, baseURL: nil)

// ✅ SEGURO: Escapar HTML e usar sandboxing
let escapedHTML = html.replacingOccurrences(of: "<", with: "&lt;")
webView.loadHTMLString(escapedHTML, baseURL: safeURL)
```

## 8. Insecure Deserialization (Desserialização Insegura)

**Descrição:** Deserializar dados não confiáveis executa código malicioso.

### Problema em Swift:
```swift
// ❌ VULNERÁVEL: Deserializar dados não confiáveis
let dados = try JSONDecoder().decode(MyClass.self, from: untrustedData)

// ✅ SEGURO: Validar e desserializar com caution
let dados = try JSONDecoder().decode(MyClass.self, from: validatedData)
// Validar estrutura antes de usar
```

## 9. Using Components with Known Vulnerabilities (Componentes com Vulnerabilidades Conhecidas)

**Descrição:** Usar bibliotecas desatualizadas com falhas de segurança.

### ✅ Prevenção:
```shell
# Verificar dependências por vulnerabilidades
swift package show-dependencies

# Usar ferramentas de scanning
# - GitHub Dependabot
# - OWASP Dependency-Check
# - Snyk

# Manter atualizado
swift package update
```

## 10. Insufficient Logging & Monitoring (Logging e Monitoramento Insuficiente)

**Descrição:** Falha em detectar e responder a ataques.

### ✅ Logging Seguro:
```swift
import os

let logger = Logger(subsystem: "com.seu.app", category: "security")

// ✅ Log de eventos importantes (sem dados sensíveis)
logger.info("Usuário fazendo login")
logger.error("Falha na autenticação para: \(email)")
logger.warning("Muitas tentativas de login falhas")

// ❌ Nunca logar
// logger.debug("Senha: \(senha)")
// logger.info("Token: \(authToken)")
```

### ✅ Monitoramento:
```swift
// Monitorar eventos de segurança
func monitoarAcesNãoAutorizado(usuarioId: String) {
    // Enviar para analytics/monitoring
    Analytics.log("security_event", parameters: [
        "tipo": "acesso_não_autorizado",
        "usuario_id": usuarioId, // Hash ou ID, nunca dados pessoais
        "timestamp": Date()
    ])
}
```

## Checklist de Segurança

### Antes de Lançar em Produção

- ✅ Código foi revisado por olhos diferentes
- ✅ Testes de segurança foram realizados
- ✅ Dependências foram auditadas
- ✅ Debug está desabilitado
- ✅ ATS está ativado
- ✅ HTTPS é obrigatório
- ✅ Dados sensíveis são criptografados
- ✅ Logging não expõe dados sensíveis
- ✅ Autenticação e autorização foram testadas
- ✅ Proteção contra força bruta implementada

## Recursos Adicionais

- [OWASP Mobile Top 10](https://owasp.org/www-project-mobile-top-10/)
- [Apple Security](https://developer.apple.com/security/)
- [CWE/SANS Top 25](https://cwe.mitre.org/top25/)

## Boas Práticas Gerais

✅ **Recomendado:**
- Fazer security reviews regularmente
- Manter frameworks e dependências atualizados
- Implementar logging e monitoramento
- Testes de penetração (se possível)
- Estudar OWASP regularmente
- Participar de bug bounty programs

❌ **Evitar:**
- Confiar apenas em validação client-side
- Usar dependências obsoletas
- Implementar criptografia customizada
- Ignorar avisos de segurança
- Colocar em produção sem testes de segurança

---

Relacionados: [Segurança](Segurança.md) | [Autenticação](Autenticação.md) | [Armazenamento de Dados](Armazenamento%20de%20Dados.md)

**Status:** ✅ Verificado
**Última atualização:** Março de 2026
