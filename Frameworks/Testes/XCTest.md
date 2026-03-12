# XCTest em Swift

**XCTest** é o framework oficial de testes da Apple, integrado nativamente no Xcode e Swift. Fornece todas as ferramentas necessárias para escribir testes unitários, de integração e de UI para suas aplicações iOS, macOS, watchOS e tvOS.

### Estrutura Básica de um Teste

```swift
import XCTest

class CalculatorTests: XCTestCase {
    var calculator: Calculator!
    
    override func setUp() {
        super.setUp()
        // Configuração antes de cada teste
        calculator = Calculator()
    }
    
    override func tearDown() {
        // Limpeza após cada teste
        calculator = nil
        super.tearDown()
    }
    
    func testAddition() {
        let result = calculator.add(5, 3)
        XCTAssertEqual(result, 8)
    }
}
```

### Assertions Disponíveis

#### Comparações Básicas

```swift
XCTAssertEqual(5, 5)              // Igual
XCTAssertNotEqual(5, 3)           // Não igual
XCTAssertIdentical(obj1, obj2)    // Mesma referência
XCTAssertNotIdentical(obj1, obj2) // Referências diferentes
```

#### Valores Booleanos

```swift
XCTAssertTrue(result)             // Verdadeiro
XCTAssertFalse(!result)           // Falso
```

#### Nil

```swift
XCTAssertNil(value)               // É nil
XCTAssertNotNil(value)            // Não é nil
```

#### Lançamento de Erros

```swift
XCTAssertThrowsError(try riskyFunction()) // Lança erro
XCTAssertNoThrow(try safeFunction())      // Não lança erro
```

#### Comparações com Mensagem

```swift
XCTAssertEqual(5, 5, "Os números devem ser iguais")
```

### Exemplo Completo

```swift
import XCTest

class LoginViewModelTests: XCTestCase {
    var viewModel: LoginViewModel!
    var mockAPIClient: MockAPIClient!
    
    override func setUp() {
        super.setUp()
        mockAPIClient = MockAPIClient()
        viewModel = LoginViewModel(apiClient: mockAPIClient)
    }
    
    func testSuccessfulLogin() {
        // Arrange
        let email = "test@example.com"
        let password = "password123"
        
        // Act
        viewModel.login(email: email, password: password)
        
        // Assert
        XCTAssertTrue(viewModel.isLoggedIn)
        XCTAssertNil(viewModel.errorMessage)
    }
    
    func testInvalidEmail() {
        // Arrange
        let email = "invalid-email"
        let password = "password123"
        
        // Act
        viewModel.login(email: email, password: password)
        
        // Assert
        XCTAssertFalse(viewModel.isLoggedIn)
        XCTAssertNotNil(viewModel.errorMessage)
        XCTAssertEqual(viewModel.errorMessage, "Email inválido")
    }
    
    func testEmptyPassword() {
        // Arrange
        let email = "test@example.com"
        let password = ""
        
        // Act
        viewModel.login(email: email, password: password)
        
        // Assert
        XCTAssertFalse(viewModel.isLoggedIn)
        XCTAssertEqual(viewModel.errorMessage, "Senha não pode estar vazia")
    }
}
```

### Testes Assíncronos

```swift
func testAsyncDataFetching() {
    let expectation = XCTestExpectation(description: "Dados carregados")
    
    apiClient.fetchUser(1) { user in
        XCTAssertNotNil(user)
        XCTAssertEqual(user?.name, "John")
        expectation.fulfill()
    }
    
    wait(for: [expectation], timeout: 5.0)
}
```

### Performance Testing

```swift
func testCalculationPerformance() {
    measure {
        _ = fibonacci(20)
    }
}
```

### Mocking com XCTest

```swift
protocol APIClientProtocol {
    func fetchUser(id: Int, completion: @escaping (User?) -> Void)
}

class MockAPIClient: APIClientProtocol {
    var fetchUserCalled = false
    var mockUser: User?
    
    func fetchUser(id: Int, completion: @escaping (User?) -> Void) {
        fetchUserCalled = true
        completion(mockUser)
    }
}

func testMocking() {
    let mock = MockAPIClient()
    mock.mockUser = User(id: 1, name: "John")
    
    let viewModel = UserViewModel(apiClient: mock)
    viewModel.loadUser(1)
    
    XCTAssertTrue(mock.fetchUserCalled)
}
```

### Rodando Testes

#### No Xcode
```
Product → Test (⌘U)
```

#### Na Linha de Comando
```bash
xcodebuild test -scheme MyApp
```

#### Com Cobertura
```bash
xcodebuild test -scheme MyApp -enableCodeCoverage YES
```

### Vantagens do XCTest

- ✅ Framework oficial da Apple
- ✅ Integrado nativamente no Xcode
- ✅ Sem dependências externas
- ✅ Suporte completo para testes async
- ✅ Performance testing incluído
- ✅ UI testing nativamente

### Desvantagens

- ❌ Sintaxe mais verbosa que alguns frameworks
- ❌ Menos expressivo que BDD frameworks

---

**Para mais informações:** [XCTest Documentation](https://developer.apple.com/documentation/xctest)
