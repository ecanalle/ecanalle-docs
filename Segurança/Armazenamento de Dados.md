# Armazenamento Seguro de Dados

**Armazenamento seguro** refere-se a como proteger dados persistidos no disco do dispositivo. Dados sensíveis devem ser criptografados e acessíveis apenas à aplicação.

## Opções de Armazenamento

### UserDefaults (❌ Para dados sensíveis)

Armazena em texto claro. Útil para preferências, não para dados sensíveis.

```swift
import Foundation

// ❌ ERRADO: Armazenar dados sensíveis
UserDefaults.standard.set("token123", forKey: "authToken")
UserDefaults.standard.set("senha123", forKey: "senha")

// ✅ CORRETO: Apenas para preferências públicas
UserDefaults.standard.set("tema_escuro", forKey: "temaSelecionado")
UserDefaults.standard.set(false, forKey: "primeiroLancamento")
```

### Keychain (✅ Recomendado para credenciais)

Criptografado, protegido pelo sistema. Ideal para tokens, senhas, chaves.

Veja [Keychain](Keychain.md) para detalhes completos.

```swift
// ✅ CORRETO: Usar Keychain para dados sensíveis
try KeychainHelper.standard.salvar("token123", para: "authToken")
let token = KeychainHelper.standard.recuperar("authToken")
```

### Core Data (Texto claro ou criptografado)

Framework de persistência orientado a objetos. Por padrão, texto claro, mas pode ser criptografado.

```swift
import CoreData

// Setup básico
let container = NSPersistentContainer(name: "MeuApp")
container.loadPersistentStores { _, error in
    if let error = error {
        print("Erro: \(error)")
    }
}

// Criar uma entidade
let usuario = NSEntityDescription.insertNewObject(forEntityName: "Usuario", into: container.viewContext)
usuario.setValue("joao@email.com", forKey: "email")
usuario.setValue("João Silva", forKey: "nome")

// Salvar
do {
    try container.viewContext.save()
} catch {
    print("Erro ao salvar: \(error)")
}

// Consultar
let requisicao: NSFetchRequest<NSFetchRequestResult> = NSFetchRequest(entityName: "Usuario")
do {
    let usuarios = try container.viewContext.fetch(requisicao)
    print("Usuários encontrados: \(usuarios.count)")
} catch {
    print("Erro ao buscar: \(error)")
}
```

### Realm (Criptografia nativa)

Banco de dados modernoo com suporte nativo a criptografia.

```swift
import RealmSwift

// Definir modelo
class Usuario: Object {
    @Persisted(primaryKey: true) var id: ObjectId
    @Persisted var email: String = ""
    @Persisted var nome: String = ""
}

// Salvar com criptografia
do {
    let config = Realm.Configuration(
        encryptionKey: NSData(SecureBytes()).bytes  // Gerar chave aleatória
    )
    let realm = try Realm(configuration: config)
    
    let usuario = Usuario()
    usuario.email = "maria@email.com"
    usuario.nome = "Maria"
    
    try realm.write {
        realm.add(usuario)
    }
} catch {
    print("Erro ao salvar: \(error)")
}

// Recuperar
do {
    let realm = try Realm()
    let usuarios = realm.objects(Usuario.self)
    usuarios.forEach { print($0.email) }
} catch {
    print("Erro: \(error)")
}
```

### FileManager (Pode ser criptografado)

Armazenar arquivos com proteção de atributos.

```swift
import Foundation

// Criar arquivo com proteção
let fileManager = FileManager.default
let documentPath = fileManager.urls(for: .documentDirectory, in: .userDomainMask)[0]
let fileURL = documentPath.appendingPathComponent("dados_sensivel.txt")

let dados = "Informação confidencial"

// Escrever arquivo
try dados.write(to: fileURL, atomically: true, encoding: .utf8)

// Proteger arquivo
var attributes = try fileManager.attributesOfItem(atPath: fileURL.path)
attributes[FileAttributeKey.protectionKey] = FileProtectionType.complete
try fileManager.setAttributes(attributes, ofItemAtPath: fileURL.path)

// Ler arquivo
let conteudo = try String(contentsOf: fileURL, encoding: .utf8)
print(conteudo)
```

### Criptografia de Arquivo com CryptoKit

```swift
import Foundation
import CryptoKit

func criptografarArquivo(em fonte: URL, para destino: URL) throws {
    let chave = SymmetricKey(size: .bits256)
    
    // Ler arquivo original
    let dados = try Data(contentsOf: fonte)
    
    // Criptografar
    let sealedBox = try AES.GCM.seal(dados, using: chave)
    
    // Escrever arquivo criptografado
    try sealedBox.combined.write(to: destino)
    
    // ⚠️ IMPORTANTE: Guardar a chave no Keychain!
    try KeychainHelper.standard.salvar(
        chave.withUnsafeBytes { Data($0).base64EncodedString() },
        para: "chaveArquivo"
    )
}

func descriptografarArquivo(em fonte: URL) throws -> Data {
    // Recuperar chave do Keychain
    guard let chaveBase64 = KeychainHelper.standard.recuperar("chaveArquivo"),
          let chaveData = Data(base64Encoded: chaveBase64) else {
        throw NSError(domain: "Erro", code: -1, userInfo: nil)
    }
    
    let chave = SymmetricKey(data: chaveData)
    
    // Ler arquivo criptografado
    let dados = try Data(contentsOf: fonte)
    
    // Descriptografar
    let sealedBox = try AES.GCM.SealedBox(combined: dados)
    return try AES.GCM.open(sealedBox, using: chave)
}
```

## Comparação de Métodos

| Armazenamento | Criptografia | Uso Ideal | Sincroniza iCloud |
|---|---|---|---|
| **UserDefaults** | ❌ | Preferências públicas | ✅ |
| **Keychain** | ✅✅ | Credenciais, tokens | ✅ |
| **Core Data** | ❌¹ | Dados estruturados | ⚠️ |
| **Realm** | ✅ | Dados estruturados | ❌ |
| **FileManager** | ❌² | Arquivos genéricos | ⚠️ |

¹ Core Data pode ser criptografado com extensões  
² Rquire criptografia customizada

## Proteção de Dados no Disco (Data Protection)

iOS automaticamente criptografa dados nos seguintes casos:

- 🔒 **Complete** - Acessível apenas quando o dispositivo está desbloqueado
- 🔒 **Complete Unless Open** - Criptografado, mas acessível durante upload se foi aberto
- 🔒 **Complete Until First User Authentication** - Criptografado até primeiro desbloqueio após boot

```swift
// Verificar nível de proteção
let fileManager = FileManager.default
let attributes = try fileManager.attributesOfItem(atPath: filePath)
if let protecao = attributes[FileAttributeKey.protectionKey] {
    print("Nível: \(protecao)")
}
```

## Boas Práticas

✅ **Recomendado:**
- Usar **Keychain** para credenciais e tokens
- Usar **Core Data** ou **Realm** para dados estruturados
- Criptografar dados sensíveis antes de armazenar em arquivo
- Definir níveis apropriados de proteção de acesso
- Limpar dados ao fazer logout
- Usar FileProtectionType.complete para arquivos sensíveis

❌ **Evitar:**
- Armazenar senhas em UserDefaults
- Usar texto claro para dados sensíveis
- Adicionar dados sensíveis a iCloud sem criptografia
- Ignorar avisos de proteção de dados
- Usar criptografia fraca ou implementada manualmente

## Limpeza de Dados

```swift
// Ao fazer logout, limpar todos os dados
func limparDadosUsuario() {
    KeychainHelper.standard.deletar("authToken")
    KeychainHelper.standard.deletar("refreshToken")
    UserDefaults.standard.removeObject(forKey: "usuarioId")
    
    // Deletar Core Data
    let fetchRequest: NSFetchRequest<NSFetchRequestResult> = NSFetchRequest(entityName: "Usuario")
    let deleteRequest = NSBatchDeleteRequest(fetchRequest: fetchRequest)
    
    let context = NSManagedObjectContext(concurrencyType: .mainQueueConcurrencyType)
    try? context.execute(deleteRequest)
    try? context.save()
}
```

## Requisitos de Entitlements

Para certas operações de segurança, adicione ao Info.plist:

```xml
<dict>
    <key>com.apple.security.application-groups</key>
    <array>
        <string>group.com.seuapp</string>
    </array>
    
    <key>com.apple.security.keychain-access-groups</key>
    <array>
        <string>$(AppIdentifierPrefix)com.seuapp</string>
    </array>
</dict>
```

---

Relacionados: [Keychain](Keychain.md) | [CryptoKit](CryptoKit.md) | [Segurança](Segurança.md)

**Status:** ✅ Verificado
**Última atualização:** Março de 2026
