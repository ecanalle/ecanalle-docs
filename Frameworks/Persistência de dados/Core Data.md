# Core Data em Swift

**Core Data** é um framework poderoso da Apple para persistência de dados, object-relational mapping (ORM) e gerenciamento de grafo de objetos. É especialmente útil para aplicações que precisam de queries complexas, relacionamentos entre entidades e sincronização de dados.

### Conceitos Principais

#### 1. **Modelo de Dados**

Core Data usa um modelo visual (`.xcdatamodeld`) para definir entidades e relacionamentos:

- **Entidades:** Equivalentes a tabelas em banco de dados
- **Atributos:** Propriedades de cada entidade
- **Relacionamentos:** Conexões entre entidades

#### 2. **Managed Object Context**

Gerencia os objetos em memória:

```swift
let context = NSManagedObjectContext(concurrencyType: .mainQueueConcurrencyType)
context.persistentStoreCoordinator = coordinator
```

#### 3. **Persistent Store Coordinator**

Conecta o contexto ao armazenamento de dados (banco SQLite):

```swift
let coordinator = NSPersistentStoreCoordinator(managedObjectModel: model)
```

### Exemplo Básico

#### Definir uma Entidade (User)

```swift
@NSManaged public var id: NSNumber
@NSManaged public var name: String
@NSManaged public var email: String
@NSManaged public var posts: NSSet
```

#### Criar um Objeto

```swift
let user = NSEntityDescription.insertNewObject(
    forEntityName: "User",
    into: context
) as! User

user.id = 1
user.name = "John Doe"
user.email = "john@example.com"

try context.save()
```

#### Buscar Objetos

```swift
let request = NSFetchRequest<User>(entityName: "User")
request.predicate = NSPredicate(format: "name == %@", "John Doe")

let results = try context.fetch(request)
```

#### Atualizar um Objeto

```swift
let user = results.first
user?.email = "newemail@example.com"
try context.save()
```

#### Deletar um Objeto

```swift
if let user = results.first {
    context.delete(user)
    try context.save()
}
```

### Relacionamentos

```swift
// Entidade User com relacionamento para Posts
@NSManaged public var posts: NSSet

// Entidade Post com relacionamento para User
@NSManaged public var author: User
```

### Predicados para Buscas Avançadas

```swift
// Buscar por email contendo "example"
let predicate = NSPredicate(format: "email CONTAINS %@", "example")

// Operador AND
let andPredicate = NSCompoundPredicate(
    andPredicateWithSubpredicates: [pred1, pred2]
)

// Ordenação
request.sortDescriptors = [NSSortDescriptor(key: "name", ascending: true)]
```

### Vantagens do Core Data

- ✅ Queries complexas e filtros avançados
- ✅ Relacionamentos automáticos e cascata
- ✅ Sincronização com iCloud
- ✅ Performance otimizada
- ✅ Integração profunda com Xcode

### Desvantagens

- ❌ Curva de aprendizado acentuada
- ❌ API verbosa
- ❌ Debugging mais complexo

### Alternativas Modernas

- **SwiftData** (iOS 17+): Versão moderna de Core Data
- **Realm**: Banco de dados móvel alternativo

---

**Para mais informações:** [Core Data Documentation](https://developer.apple.com/documentation/coredata)
