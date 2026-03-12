# Persistência de Dados em Swift

**Persistência de dados** é a capacidade de salvar informações de forma permanente em disco, permitindo que dados continuem disponíveis mesmo após o encerramento da aplicação. iOS e macOS oferecem várias opções para implementar persistência, cada uma com seus próprios casos de uso.

### Opções de Persistência

#### 1. **UserDefaults**

A forma mais simples de salvar pequeños dados (preferências, configurações):

```swift
// Salvar dados
UserDefaults.standard.set("John", forKey: "userName")
UserDefaults.standard.set(25, forKey: "age")

// Recuperar dados
let name = UserDefaults.standard.string(forKey: "userName")
let age = UserDefaults.standard.integer(forKey: "age")
```

#### 2. **Arquivo (Codable)**

Salvar estruturas complexas como JSON ou arquivo binário:

```swift
struct User: Codable {
    let id: Int
    let name: String
    let email: String
}

let user = User(id: 1, name: "John", email: "john@example.com")
let data = try JSONEncoder().encode(user)

// Salvar em arquivo
let url = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask)[0]
try data.write(to: url.appendingPathComponent("user.json"))

// Recuperar do arquivo
let savedData = try Data(contentsOf: url.appendingPathComponent("user.json"))
let decodedUser = try JSONDecoder().decode(User.self, from: savedData)
```

#### 3. **Core Data**

Framework poderoso para persistência orientada a objetos com suporte a queries complexas:

```swift
// Criar um objeto
let newUser = NSEntityDescription.insertNewObject(
    forEntityName: "User",
    into: managedObjectContext
) as! User
newUser.name = "John"
newUser.email = "john@example.com"

// Salvar
try managedObjectContext.save()

// Buscar
let request = NSFetchRequest<User>(entityName: "User")
let results = try managedObjectContext.fetch(request)
```

#### 4. **SwiftData** (Novo em iOS 17+)

Moderna alternativa ao Core Data:

```swift
@Model
final class User {
    var name: String
    var email: String
}

// Usar com @Query
@Query var users: [User]
```

#### 5. **Banco de Dados SQLite**

Usando bibliotecas como FMDB ou simples SQL:

```swift
let query = "SELECT * FROM users WHERE id = ?"
let results = db.executeQuery(query, withArgumentsIn: [1])
```

#### 6. **Realm**

Banco de dados móvel popular:

```swift
let user = User()
user.name = "John"
user.email = "john@example.com"

let realm = try! Realm()
try realm.write {
    realm.add(user)
}
```

### Comparação de Opções

| Opção | Simplicidade | Poder | Complexidade |
|-------|-------------|-------|-------------|
| UserDefaults | ⭐⭐⭐⭐⭐ | ⭐ | Muito baixa |
| Codable | ⭐⭐⭐⭐ | ⭐⭐⭐ | Baixa |
| Core Data | ⭐⭐ | ⭐⭐⭐⭐⭐ | Alta |
| SwiftData | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | Média |
| SQLite | ⭐⭐⭐ | ⭐⭐⭐⭐ | Média |
| Realm | ⭐⭐⭐ | ⭐⭐⭐⭐ | Média |

### Quando Usar Cada Uma

- **UserDefaults:** Configurações e preferências simples
- **Codable:** Dados até alguns MB, estruturas simples
- **Core Data:** Aplicações complexas com queries avançadas
- **SwiftData:** Projetos novos (iOS 17+)
- **SQLite:** Performance crítica com grandes volumes
- **Realm:** Aplicações mobile com sincronização

---

**Relacionados:** 
- [Core Data](Core%20Data.md) 
- [SwiftData](SwiftData.md)
- [Realm](Realm.md)