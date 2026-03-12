# Quick & Nimble em Swift

**Quick** e **Nimble** são frameworks de testes para Swift que implementam a abordagem BDD (Behavior-Driven Development). Quick fornece uma sintaxe expressiva para descrever comportamentos, enquanto Nimble oferece matchers elegantes para assertions.

### Instalação

Via CocoaPods:

```ruby
target 'MyAppTests' do
  pod 'Quick'
  pod 'Nimble'
end
```

Via SPM:

```swift
.package(url: "https://github.com/Quick/Quick.git", from: "7.0.0"),
.package(url: "https://github.com/Quick/Nimble.git", from: "13.0.0")
```

### Estrutura Básica com Quick

```swift
import Quick
import Nimble

class CalculatorSpec: QuickSpec {
    override func spec() {
        describe("Calculator") {
            it("adds two numbers correctly") {
                let calculator = Calculator()
                let result = calculator.add(2, 3)
                expect(result).to(equal(5))
            }
        }
    }
}
```

### Describe, Context e It

```swift
describe("User") {
    context("quando criado com dados válidos") {
        it("deve ter nome não vazio") {
            let user = User(name: "John", email: "john@example.com")
            expect(user.name).toNot(beEmpty())
        }
        
        it("deve ter email válido") {
            let user = User(name: "John", email: "john@example.com")
            expect(user.email).to(contain("@"))
        }
    }
    
    context("quando criado com dados inválidos") {
        it("deve rejeitar nome vazio") {
            let user = User(name: "", email: "john@example.com")
            expect(user.isValid).to(beFalse())
        }
    }
}
```

### Matchers do Nimble

#### Igualdade e Comparação

```swift
expect(5).to(equal(5))
expect(5).toNot(equal(3))
expect(5).to(beLessThan(10))
expect(5).to(beGreaterThan(2))
expect(5).to(beLessThanOrEqualTo(5))
```

#### Strings

```swift
expect("Hello").to(contain("ell"))
expect("Hello").to(startWith("He"))
expect("Hello").to(endWith("lo"))
expect("hello").to(match("^[a-z]*$"))
```

#### Coleções

```swift
expect([1, 2, 3]).to(contain(2))
expect([1, 2, 3]).toNot(beEmpty())
expect([1, 2, 3]).to(haveCount(3))
```

#### Booleanos

```swift
expect(true).to(beTrue())
expect(false).to(beFalse())
```

#### Nil

```swift
expect(value).to(beNil())
expect(value).toNot(beNil())
```

#### Tipos

```swift
expect("hello").to(beAnInstanceOf(String.self))
```

### Exemplo Completo

```swift
import Quick
import Nimble

class LoginSpec: QuickSpec {
    override func spec() {
        describe("LoginViewController") {
            var viewModel: LoginViewModel!
            var mockAPIClient: MockAPIClient!
            
            beforeEach {
                mockAPIClient = MockAPIClient()
                viewModel = LoginViewModel(apiClient: mockAPIClient)
            }
            
            context("com credenciais válidas") {
                it("realiza login com sucesso") {
                    viewModel.email = "user@example.com"
                    viewModel.password = "password123"
                    viewModel.login()
                    
                    expect(viewModel.isLoggedIn).toEventually(beTrue())
                }
            }
            
            context("com email inválido") {
                it("mostrar mensagem de erro") {
                    viewModel.email = "invalid-email"
                    viewModel.password = "password123"
                    viewModel.login()
                    
                    expect(viewModel.errorMessage)
                        .toEventually(contain("Email inválido"))
                }
            }
            
            context("com senha vazia") {
                it("não permite login") {
                    viewModel.email = "user@example.com"
                    viewModel.password = ""
                    viewModel.login()
                    
                    expect(viewModel.isLoggedIn).to(beFalse())
                }
            }
        }
    }
}
```

### Asserciones Assíncronas

```swift
// Esperar que um valor mude
expect(viewModel.isLoading).toEventually(beFalse(), timeout: .seconds(2))

// Com polling customizado
expect(self.asyncValue).toEventually(
    equal(expectedValue),
    timeout: .seconds(5),
    pollInterval: .milliseconds(100)
)
```

### Closures para Setup/Teardown

```swift
beforeEach {
    // Executado antes de cada teste
    viewModel = LoginViewModel()
}

afterEach {
    // Executado após cada teste
    viewModel = nil
}

beforeSuite {
    // Executado uma vez antes de todos os testes
}

afterSuite {
    // Executado uma vez após todos os testes
}
```

### Vantagens de Quick & Nimble

- ✅ Sintaxe BDD expressiva e legível
- ✅ Matchers muito elsigantes
- ✅ Ótimo para documentação de comportamento
- ✅ Menos verboso que XCTest
- ✅ Comunidade ativa

### Desvantagens

- ❌ Requer dependência externa
- ❌ Setup mais complexo que XCTest
- ❌ Pode ser overkill para testes simples

### Combinando Quick, Nimble e XCTest

Você pode usar ambos no mesmo projeto:

```swift
// Testes simples com XCTest
// Testes comportamentais com Quick + Nimble
```

---

**Para mais informações:**
- [Quick Documentation](https://github.com/Quick/Quick/wiki)
- [Nimble Documentation](https://github.com/Quick/Nimble/blob/main/README.md)
