# SwiftData em Swift

**SwiftData** é o novo framework de persistência de dados da Apple, introduzido no iOS 17. É uma alternativa moderna e simplificada ao Core Data, aproveitando as vantagens do Swift moderno com menos complexidade e mais segurança de tipos.

### Por que SwiftData?

- Syntaxe mais simples que Core Data
- Totalmente escrito em Swift (não em Objective-C)
- Suporte nativo a Codable e structs
- Melhor integração com SwiftUI
- Type-safe queries

### Começando com SwiftData

#### 1. Definir um Modelo

```swift
import SwiftData

@Model
final class User {
    var id: UUID = UUID()
    var name: String
    var email: String
    var createdAt: Date = Date()
    
    @Relationship(deleteRule: .cascade) var posts: [Post] = []
    
    init(name: String, email: String) {
        self.name = name
        self.email = email
    }
}

@Model
final class Post {
    var id: UUID = UUID()
    var title: String
    var content: String
    var author: User?
    
    init(title: String, content: String, author: User? = nil) {
        self.title = title
        self.content = content
        self.author = author
    }
}
```

#### 2. Criar e Salvar Dados

```swift
@Environment(\.modelContext) var modelContext

let user = User(name: "John", email: "john@example.com")
modelContext.insert(user)

try? modelContext.save()
```

#### 3. Buscar Dados com @Query

```swift
import SwiftUI

struct ContentView: View {
    @Query var users: [User]
    
    var body: some View {
        List(users) { user in
            VStack(alignment: .leading) {
                Text(user.name)
                Text(user.email).font(.caption)
            }
        }
    }
}
```

#### 4. Filtrar e Ordenar

```swift
@Query(
    sort: [SortDescriptor(\.name, order: .forward)],
    predicate: #Predicate { $0.email.contains("example") }
) var filteredUsers: [User]
```

#### 5. Atualizar Dados

```swift
let user = users.first
user?.email = "newemail@example.com"
try? modelContext.save()
```

#### 6. Deletar Dados

```swift
modelContext.delete(user)
try? modelContext.save()
```

### Configurar o Container

```swift
@main
struct MyApp: App {
    let container: ModelContainer
    
    init() {
        do {
            let config = ModelConfiguration(isStoredInMemoryOnly: false)
            container = try ModelContainer(
                for: User.self, Post.self,
                configurations: config
            )
        } catch {
            fatalError("Could not initialize ModelContainer: \(error)")
        }
    }
    
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(container)
    }
}
```

### Vantagens do SwiftData

- ✅ Sintaxe limpa e moderna
- ✅ Type-safe
- ✅ Perfeito para SwiftUI
- ✅ Menos boilerplate que Core Data
- ✅ Melhor performance
- ✅ Seguro de thread

### Quando Usar SwiftData

- ✅ Projetos novos em iOS 17+
- ✅ Aplicações SwiftUI
- ✅ Dados estruturados simples a médios
- ❌ Apps que precisam suportar iOS < 17
- ❌ Queries extremamente complexas (use Core Data)

### Comparação: Core Data vs SwiftData

| Aspecto | Core Data | SwiftData |
|---------|-----------|-----------|
| Complexidade | Alta | Baixa |
| Curva de aprendizado | Íngreme | Suave |
| Type-safety | Média | Alta |
| SwiftUI integration | Boa | Excelente |
| Compatibilidade | iOS 10+ | iOS 17+ |

---

**Para mais informações:** [SwiftData Documentation](https://developer.apple.com/documentation/swiftdata)
