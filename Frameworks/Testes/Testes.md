# Testes em Swift

**Testes** são uma parte crucial do desenvolvimento de software, garantindo que sua aplicação funciona corretamente, é confiável e mantém a qualidade ao longo do tempo. Swift oferece várias ferramentas e frameworks para implementar testes efetivos.

### Tipos de Testes

#### 1. **Testes Unitários**

Testam componentes individuais (functions, classes) isoladamente:

```swift
func testAddition() {
    let result = 2 + 2
    XCTAssertEqual(result, 4)
}
```

#### 2. **Testes de Integração**

Testam como múltiplos componentes trabalham juntos:

```swift
func testUserFetching() {
    let apiClient = APIClient()
    let userID = 1
    
    let user = apiClient.fetchUser(userID)
    XCTAssertNotNil(user)
    XCTAssertEqual(user?.name, "John")
}
```

#### 3. **Testes de UI**

Testam fluxos de usuário e interações na interface:

```swift
func testButtonTap() {
    let app = XCUIApplication()
    app.launch()
    
    let button = app.buttons["submitButton"]
    button.tap()
    
    let label = app.staticTexts["successMessage"]
    XCTAssertTrue(label.exists)
}
```

### Frameworks de Testes

#### **XCTest** (Padrão Apple)

Framework nativo e poderoso que vem com Xcode:

```swift
class CalculatorTests: XCTestCase {
    var calculator: Calculator!
    
    override func setUp() {
        super.setUp()
        calculator = Calculator()
    }
    
    func testAddition() {
        let result = calculator.add(2, 3)
        XCTAssertEqual(result, 5)
    }
    
    func testDivisionByZero() {
        let result = calculator.divide(10, 0)
        XCTAssertNil(result)
    }
}
```

#### **Quick & Nimble** (BDD Style)

Framework para testes em estilo BDD (Behavior-Driven Development):

```swift
import Quick
import Nimble

class CalculatorSpec: QuickSpec {
    override func spec() {
        describe("Calculator") {
            context("addition") {
                it("returns the sum of two numbers") {
                    let calc = Calculator()
                    expect(calc.add(2, 3)).to(equal(5))
                }
            }
        }
    }
}
```

#### **RxTest** (para APIs Reativas)

Para testar código com RxSwift:

```swift
func testObservable() {
    let scheduler = TestScheduler(initialClock: 0)
    let observable = scheduler.createHotObservable([
        .next(210, 1),
        .next(220, 2),
        .completed(230)
    ])
}
```

### Mocking e Stubs

```swift
protocol APIClientProtocol {
    func fetchUser(_ id: Int) -> User?
}

class MockAPIClient: APIClientProtocol {
    func fetchUser(_ id: Int) -> User? {
        return User(id: 1, name: "Mock User")
    }
}

func testWithMock() {
    let mockClient = MockAPIClient()
    let viewModel = UserViewModel(apiClient: mockClient)
    // Testar com dados controlados
}
```

### Assertion Comuns

```swift
XCTAssertEqual(a, b)           // Igualdade
XCTAssertNotEqual(a, b)        // Desigualdade
XCTAssert(condition)           // Verdadeiro
XCTAssertTrue(condition)       // Verdadeiro
XCTAssertFalse(condition)      // Falso
XCTAssertNil(value)            // Nulo
XCTAssertNotNil(value)         // Não nulo
XCTAssertThrowsError(expr)     // Lança erro
XCTAssertNoThrow(expr)         // Não lança erro
XCTFail(message)               // Falha força
```

### Coverage de Testes

No Xcode:
1. Product → Scheme → Edit Scheme
2. Test → Options → Code Coverage → ✓
3. Run tests e veja o relatório

### Boas Práticas

- ✅ **AAA Pattern:** Arrange, Act, Assert
- ✅ **Nomes descritivos:** `testUserCanLoginWithValidCredentials()`
- ✅ **Um conceito por teste**
- ✅ **Testes independentes**
- ✅ **Fixtures compartilhadas com cuidado**
- ✅ **Mock objetos caros (API, BD)**
- ✅ **Code coverage > 70%**

### Estrutura de Projeto Recomendada

```
MyApp/
├── Sources/
│   └── MyApp/
│       ├── Models/
│       ├── ViewModels/
│       └── Services/
└── Tests/
    └── MyAppTests/
        ├── ModelsTests/
        ├── ViewModelsTests/
        └── ServicesTests/
```

---

**Relacionados:**
- [XCTest](XCTest.md)
- [Quick & Nimble](Quick%20-%20Nimble.md)