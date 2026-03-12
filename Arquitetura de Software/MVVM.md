O padrão de arquitetura Model-View-ViewModel (MVVM) surgiu como uma evolução do Model-View-Controller (MVC), buscando mitigar algumas de suas desvantagens, como o acoplamento excessivo entre View e Controller e os "Massive View Controllers". O MVVM é especialmente popular no desenvolvimento de aplicações com frameworks como WPF, Xamarin e, cada vez mais, no desenvolvimento iOS com SwiftUI e até mesmo em algumas arquiteturas UIKit.

O MVVM introduz uma nova camada entre a View e o Model, chamada ViewModel. Essa camada é responsável por preparar e apresentar os dados do Model para a View, além de expor comandos que a View pode invocar para interagir com o Model.

**Os Componentes do MVVM:**

1. **Model:** Assim como no MVC, o Model no MVVM é responsável por conter os dados da aplicação e a lógica de negócios. Ele é independente da interface do usuário.
    
2. **View:** A View no MVVM é uma estrutura passiva que exibe os dados fornecidos pelo ViewModel e encaminha as ações do usuário para o ViewModel através de mecanismos de _data binding_ ou comandos. A View deve conter o mínimo de lógica possível, focando apenas na apresentação visual.
    
3. **ViewModel:** O ViewModel atua como um intermediário entre a View e o Model, mas ao contrário do Controller no MVC, ele não possui uma referência direta à View. Em vez disso, ele expõe propriedades e comandos aos quais a View pode se _bindar_ (conectar). O ViewModel busca os dados do Model, formata-os para exibição e implementa a lógica de apresentação. Ele também lida com as ações do usuário recebidas da View, geralmente invocando métodos no Model.
    

**Diagrama Simplificado:**

![[DiagramaMVVM.svg]]

**Fluxo de Interação Típico:**

1. A **View** se _binda_ às propriedades expostas pelo **ViewModel** para exibir os dados.
2. O usuário interage com elementos na **View** (por exemplo, digita em um campo de texto, pressiona um botão).
3. Essas interações são encaminhadas para o **ViewModel** através de _data binding_ (em frameworks que suportam isso nativamente, como SwiftUI) ou através de comandos (objetos que encapsulam uma ação).
4. O **ViewModel** recebe a ação do usuário, interage com o **Model** para atualizar os dados ou executar a lógica de negócios.
5. O **Model** (se necessário) notifica o **ViewModel** sobre as mudanças nos dados (geralmente através de mecanismos como delegates, notificações, Combine publishers ou RxSwift Observables).
6. O **ViewModel** atualiza suas propriedades de acordo com as mudanças no Model.
7. Devido ao _data binding_, a **View** é automaticamente notificada sobre as mudanças nas propriedades do ViewModel e se atualiza para refletir os novos dados.

**Exemplo Conceitual em Swift (adaptado do exemplo do contador para ilustrar o MVVM):**

**1. Model (`CounterModel.swift`):**

```swift
import Foundation

protocol CounterModelDelegate: AnyObject {
    func counterDidChange(to value: Int)
}

class CounterModel {
    private var count: Int = 0
    weak var delegate: CounterModelDelegate?

    func increment() {
        count += 1
        delegate?.counterDidChange(to: count)
    }

    func decrement() {
        count -= 1
        delegate?.counterDidChange(to: count)
    }

    func getCount() -> Int {
        return count
    }
}
```

_(O Model permanece essencialmente o mesmo, pois representa a camada de dados e lógica de negócios)_

**2. ViewModel (`CounterViewModel.swift`):**

```swift
import Foundation

protocol CounterViewModelDelegate: AnyObject {
    func counterValueChanged(newValue: String)
}

class CounterViewModel: CounterModelDelegate {
    private let model = CounterModel()
    weak var delegate: CounterViewModelDelegate?

    var counterValue: String = "0" {
        didSet {
            delegate?.counterValueChanged(newValue: counterValue)
        }
    }

    init() {
        model.delegate = self
        counterValue = "\(model.getCount())"
    }

    func incrementCounter() {
        model.increment()
    }

    func decrementCounter() {
        model.decrement()
    }

    // MARK: - CounterModelDelegate

    func counterDidChange(to value: Int) {
        counterValue = "\(value)"
    }
}
```

- O `CounterViewModel` mantém uma referência ao `CounterModel`.
- Ele expõe uma propriedade `counterValue` (String) que será exibida pela View.
- Ele implementa métodos (`incrementCounter`, `decrementCounter`) que a View pode invocar (através de comandos ou ações).
- Ele conforma ao `CounterModelDelegate` para ser notificado sobre as mudanças no Model e, em seguida, atualiza sua propriedade `counterValue`, notificando a View através de seu próprio delegate (neste exemplo simplificado com delegate, em cenários reais com data binding, essa notificação seria automática).

**3. View (`CounterView.swift` - adaptado para usar o ViewModel):**

```swift
import UIKit

protocol CounterViewDelegate: AnyObject {
    func incrementButtonTapped()
    func decrementButtonTapped()
}

class CounterView: UIView {
    weak var delegate: CounterViewDelegate?
    private let viewModel: CounterViewModel

    let counterLabel: UILabel = {
        let label = UILabel()
        label.textAlignment = .center
        label.font = .systemFont(ofSize: 24)
        return label
    }()

    let incrementButton: UIButton = {
        let button = UIButton(type: .system)
        button.setTitle("+", for: .normal)
        button.titleLabel?.font = .systemFont(ofSize: 36)
        return button
    }()

    let decrementButton: UIButton = {
        let button = UIButton(type: .system)
        button.setTitle("-", for: .normal)
        button.titleLabel?.font = .systemFont(ofSize: 36)
        return button
    }()

    init(viewModel: CounterViewModel) {
        self.viewModel = viewModel
        super.init(frame: .zero)
        setupUI()
        incrementButton.addTarget(self, action: #selector(incrementTapped), for: .touchUpInside)
        decrementButton.addTarget(self, action: #selector(decrementTapped), for: .touchUpInside)
        viewModel.delegate = self // Para receber atualizações do ViewModel (em um cenário real com binding, isso não seria necessário)
        counterLabel.text = viewModel.counterValue // Inicializa o label com o valor do ViewModel
    }

    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    private func setupUI() {
        // ... (layout da UI como antes) ...
    }

    @objc private func incrementTapped() {
        viewModel.incrementCounter()
    }

    @objc private func decrementTapped() {
        viewModel.decrementCounter()
    }
}

extension CounterView: CounterViewModelDelegate {
    func counterValueChanged(newValue: String) {
        counterLabel.text = newValue
    }
}
```

**4. ViewController (`CounterViewController.swift` - agora responsável por criar a View e o ViewModel):**

```swift
import UIKit

class CounterViewController: UIViewController {

    private let viewModel = CounterViewModel()
    private var counterView: CounterView!

    override func loadView() {
        counterView = CounterView(viewModel: viewModel)
        view = counterView
    }

    override func viewDidLoad() {
        super.viewDidLoad()
        // O ViewController agora tem um papel menor, principalmente na composição.
    }
}
```

**Vantagens do MVVM:**

- **Maior Testabilidade:** O ViewModel não depende da View (não tem referência direta a ela), o que facilita a criação de testes unitários para a lógica de apresentação.
- **Melhor Separação de Responsabilidades:** A View se concentra na exibição, o ViewModel na lógica de apresentação e o Model nos dados e lógica de negócios.
- **Redução do Acoplamento:** A View e o ViewModel se comunicam principalmente através de _data binding_ ou comandos, reduzindo o acoplamento direto.
- **Reusabilidade do ViewModel:** Um mesmo ViewModel pode ser usado por diferentes Views que precisam exibir os mesmos dados.
- **Facilita o Desenvolvimento UI-Driven:** Os desenvolvedores de UI (View) e de lógica de apresentação (ViewModel) podem trabalhar de forma mais independente.

**Desvantagens do MVVM:**

- **Pode Introduzir Mais Código:** Para projetos simples, a criação da camada ViewModel pode parecer excessiva.
- **Complexidade do Data Binding:** A implementação e o gerenciamento do _data binding_ podem adicionar complexidade, especialmente em frameworks que não o suportam nativamente.
- **Pode Levar a ViewModels Grandes:** Em aplicações complexas, os ViewModels podem se tornar grandes se não forem bem gerenciados.

**O MVVM no Contexto do Desenvolvimento Apple:**

- **SwiftUI:** O SwiftUI adota uma arquitetura que se alinha muito bem com os princípios do MVVM. As `@State`, `@Binding`, `@ObservedObject`, `@EnvironmentObject` facilitam o _data binding_ entre a View e os objetos que atuam como ViewModels.
- **UIKit:** Embora o UIKit seja tradicionalmente MVC, o padrão MVVM tem sido cada vez mais adotado por desenvolvedores que buscam os benefícios de maior testabilidade e melhor separação de responsabilidades. A implementação do _data binding_ em UIKit geralmente envolve o uso de frameworks de terceiros (como RxSwift ou ReactiveSwift) ou abordagens manuais com delegates e closures. O Combine, introduzido pela Apple, também facilita a implementação de padrões reativos que se encaixam bem com o MVVM.

**Conclusão sobre MVVM:**

O MVVM é um padrão de arquitetura poderoso que oferece várias vantagens em relação ao MVC, especialmente em termos de testabilidade e separação de responsabilidades. Sua adoção tem crescido significativamente no desenvolvimento Apple, especialmente com a ascensão do SwiftUI. Ao entender os papéis do Model, View e ViewModel e como eles se comunicam através do _data binding_ ou comandos, você estará bem equipado para construir aplicações mais robustas e fáceis de manter.