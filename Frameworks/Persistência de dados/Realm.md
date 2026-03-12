# Realm em Swift

**Realm** é um banco de dados móvel open-source, rápido e poderoso, projetado como alternativa ao Core Data. É extremamente popular em aplicações iOS e Android pela sua performance, simplicidade e capacidade de sincronização em tempo real.

### Por que Realm?

- **Rápido:** Até 10x mais rápido que Core Data em certas operações
- **Simples:** API muito mais intuitiva que Core Data
- **Multiplataforma:** Funciona em iOS, Android e Web
- **Sincronização:** Suporta sincronização em tempo real com Realm Cloud
- **Open Source:** Comunidade ativa e código disponível

### Instalação

Realm é instalado via CocoaPods ou SPM:

```bash
pod 'RealmSwift'
```

### Definir um Modelo

```swift
import RealmSwift

class Dog: Object {
    @Persisted(primaryKey: true) var id: ObjectId
    @Persisted var name: String = ""
    @Persisted var breed: String?
    @Persisted var age: Int = 0
    @Persisted var owner: Person?
}

class Person: Object {
    @Persisted(primaryKey: true) var id: ObjectId
    @Persisted var name: String = ""
    @Persisted(backlinks: \.owner) var dogs: [Dog]
}
```

### Operações Básicas

#### Criar (Create)

```swift
let myDog = Dog()
myDog.name = "Rex"
myDog.breed = "Golden Retriever"
myDog.age = 3

let realm = try! Realm()
try! realm.write {
    realm.add(myDog)
}
```

#### Ler (Read)

```swift
let realm = try! Realm()

// Buscar todos
let allDogs = realm.objects(Dog.self)

// Buscar com filtro
let goldens = allDogs.where { $0.breed == "Golden Retriever" }

// Primeiro objeto
if let firstDog = allDogs.first {
    print(firstDog.name)
}
```

#### Atualizar (Update)

```swift
try! realm.write {
    myDog.age = 4
    myDog.name = "Max"
}
```

#### Deletar (Delete)

```swift
try! realm.write {
    realm.delete(myDog)
}
```

### Queries Avançadas

```swift
let realm = try! Realm()

// Filtro com predicado
let yellowDogs = realm.objects(Dog.self)
    .where { $0.breed == "Labrador" && $0.age > 2 }

// Ordenação
let sortedDogs = realm.objects(Dog.self)
    .sorted(byKeyPath: "age", ascending: false)

// Agregação
let count = realm.objects(Dog.self).count
let avgAge = realm.objects(Dog.self).avg(ofProperty: "age")
```

### SwiftUI Integration

```swift
import SwiftUI
import RealmSwift

struct ContentView: View {
    @ObservedRealmResults(Dog.self) var dogs
    
    var body: some View {
        List {
            ForEach(dogs) { dog in
                VStack(alignment: .leading) {
                    Text(dog.name)
                    Text(dog.breed ?? "Unknown")
                        .font(.caption)
                }
            }
            .onDelete { indexSet in
                $dogs.remove(atOffsets: indexSet)
            }
        }
    }
}
```

### Relacionamentos

```swift
class User: Object {
    @Persisted(primaryKey: true) var id: ObjectId
    @Persisted var name: String = ""
    @Persisted var posts: [Post]  // One-to-Many
}

class Post: Object {
    @Persisted(primaryKey: true) var id: ObjectId
    @Persisted var title: String = ""
    @Persisted var author: User?  // Many-to-One
}
```

### Vantagens do Realm

- ✅ Performance excepcional
- ✅ API simples e intuitiva
- ✅ Sincronização em tempo real (Realm Cloud)
- ✅ Type-safe
- ✅ Excelente documentação
- ✅ Comunidade grande

### Desvantagens

- ❌ Arquivo single-file (menos flexível que SQL)
- ❌ Requer dependência externa
- ❌ Menos popular que Core Data para iOS nativo

### Quando Usar Realm

- ✅ Aplicações que precisam sincronização
- ✅ Projetos multiplataforma
- ✅ Performance é crítica
- ✅ Preferência por simplicidade
- ❌ Apps que querem zero dependências externas

---

**Para mais informações:** [Realm Documentation](https://www.mongodb.com/docs/realm/sdk/swift/)
