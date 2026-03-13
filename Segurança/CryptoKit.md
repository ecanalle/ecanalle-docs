# CryptoKit

**CryptoKit** é um framework moderno da Apple (disponível desde iOS 13) que fornece APIs seguras e fáceis de usar para operações criptográficas. Substituiu a necessidade de usar `CommonCrypto` em muitos casos.

## Visão Geral

CryptoKit oferece primitivas criptográficas bem testadas e otimizadas para performance. O framework é projetado com segurança em primeiro lugar, prevenindo erros comuns como:

- Uso de algoritmos desatualizados
- Proteção automática contra ataques de timing
- Tipo-segurança (type-safe)
- Memory safety

## Operações de Hash

### Criar um Hash SHA-256

```swift
import CryptoKit

let dados = "Olá, Mundo!".data(using: .utf8)!
let hash = SHA256.hash(data: dados)

print(hash) // Imprime o hash em formato hexadecimal
```

### Comparar Hashes

```swift
let hash1 = SHA256.hash(data: dados1)
let hash2 = SHA256.hash(data: dados2)

if hash1 == hash2 {
    print("Os dados são idênticos")
} else {
    print("Os dados são diferentes")
}
```

### Algoritmos Disponíveis

- **SHA256** - Mais comum, 256 bits de saída
- **SHA384** - 384 bits de saída
- **SHA512** - 512 bits de saída

## HMAC (Hash-based Message Authentication Code)

HMAC combina hash com uma chave secreta para verificar a integridade e autenticidade de dados.

```swift
import CryptoKit

let chave = SymmetricKey(size: .bits256)
let mensagem = "Dados importantes".data(using: .utf8)!

let assinatura = HMAC<SHA256>.authenticationCode(for: mensagem, using: chave)

// Depois, verificar a assinatura
let novaAssinatura = HMAC<SHA256>.authenticationCode(for: mensagem, using: chave)

if hmac == novaAssinatura {
    print("Mensagem autenticada com sucesso")
}
```

## Criptografia Simétrica (AES)

Usa a mesma chave para criptografar e desencriptar.

```swift
import CryptoKit

// Gerar uma chave aleatória
let chave = SymmetricKey(size: .bits256)

// Dados para criptografar
let dadosOriginais = "Informação secreta".data(using: .utf8)!

// Criptografar usando AES-GCM
let sealedBox = try! AES.GCM.seal(dadosOriginais, using: chave)

// Desencriptar
let dadosDescriptografados = try! AES.GCM.open(sealedBox, using: chave)

let resultado = String(data: dadosDescriptografados, encoding: .utf8)
print(resultado) // "Informação secreta"
```

### AES-GCM vs AES-CBC

- **GCM** - Modo autenticado (recomendado atualmente)
- **CBC** - Modo classico, requer HMAC separado para autenticação

## Criptografia Assimétrica (RSA)

Usa par de chaves (pública/privada). A chave pública pode ser compartilhada para criptografar, apenas a chave privada desencripta.

```swift
import CryptoKit

// Gerar um par de chaves RSA
let chavePrivada = try! RSA.Encryption.PrivateKey(keySize: .bits2048)
let chavePublica = chavePrivada.publicKey

// Dados para criptografar
let dados = "Mensagem secreta".data(using: .utf8)!

// Criptografar com a chave pública (qualquer um pode fazer)
let criptografado = try! chavePublica.encrypt(dados)

// Desencriptar com a chave privada (apenas o proprietário)
let descriptografado = try! chavePrivada.decrypt(criptografado)
```

## Assinatura Digital

Garante autenticidade e não-repúdio (o assinante não pode negar que assinou).

```swift
import CryptoKit

// Gerar par de chaves para assinatura
let chavePrivada = try! Curve25519.Signing.PrivateKey()
let chavePublica = chavePrivada.publicKey

let dados = "Documento importante".data(using: .utf8)!

// Assinar com a chave privada
let assinatura = try! chavePrivada.signature(for: dados)

// Verificar a assinatura com a chave pública
let valida = chavePublica.isValidSignature(assinatura, for: dados)
print(valida) // true
```

## Troca de Chaves (Key Exchange)

Permite que dois participantes estabeleçam uma chave compartilhada de forma segura.

```swift
import CryptoKit

// Participante A
let chavePrivadaA = Curve25519.KeyAgreement.PrivateKey()
let chavePublicaA = chavePrivadaA.publicKey

// Participante B
let chavePrivadaB = Curve25519.KeyAgreement.PrivateKey()
let chavePublicaB = chavePrivadaB.publicKey

// Ambos podem derivar a mesma chave compartilhada
let chaveComunhA = try! chavePrivadaA.sharedSecretFromKeyAgreement(with: chavePublicaB)
let chaveComunhB = try! chavePrivadaB.sharedSecretFromKeyAgreement(with: chavePublicaA)

// As chaves derivadas devem ser idênticas
print(chaveComunhA.withUnsafeBytes { Data($0) } == chaveComunhB.withUnsafeBytes { Data($0) })
```

## Boas Práticas

- ✅ Use `CryptoKit` em preferência a `CommonCrypto` (mais moderno)
- ✅ Sempre use o tamanho de chave recomendado (.bits256 ou maior)
- ✅ Nunca reutilize o mesmo nonce/IV em AES-GCM
- ✅ Guarde chaves privadas no Keychain, nunca em UserDefaults
- ✅ Use `try-catch` para tratar erros de criptografia apropriadamente
- ❌ Não implemente seus próprios algoritmos criptográficos

## Requisitos de Plataforma

- **iOS:** 13.0+
- **macOS:** 10.15+
- **tvOS:** 13.0+
- **watchOS:** 6.0+

---

Relacionados: [Keychain](Keychain.md) | [HTTPS e TLS](HTTPS%20e%20TLS.md) | [Segurança](Segurança.md)

**Status:** ✅ Verificado
**Última atualização:** Março de 2026
