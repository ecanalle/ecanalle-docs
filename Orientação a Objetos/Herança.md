# Herança em Swift

A **herança** é uma característica fundamental da programação orientada a objetos que permite que uma classe (conhecida como **subclasse** ou **classe filha**) herde propriedades e métodos de outra classe (conhecida como **superclasse** ou **classe pai**). A herança permite que você crie novas classes baseadas em classes existentes, reaproveitando o código e estabelecendo uma hierarquia de classes relacionadas.

### Classe Base (Superclasse)

Qualquer classe que não herda de outra é conhecida como uma **classe base**.

```swift
class Veiculo {
    var velocidadeAtual = 0.0
    var descricao: String {
        return "Viajando a \(velocidadeAtual) km/h"
    }
    func acelerar(por delta: Double) {
        velocidadeAtual += delta
    }
    func frear(por delta: Double) {
        velocidadeAtual -= delta
        if velocidadeAtual < 0 {
            velocidadeAtual = 0
        }
    }
}

let carro = Veiculo()
print(carro.descricao) // Saída: Viajando a 0.0 km/h
carro.acelerar(por: 50.0)
print(carro.descricao) // Saída: Viajando a 50.0 km/h
```

Neste exemplo, `Veiculo` é uma classe base com propriedades como `velocidadeAtual` e métodos como `acelerar` e `frear`.

### Subclasse (Classe Filha)

Uma classe que herda de outra é chamada de **subclasse**. A sintaxe para definir uma subclasse é especificar a superclasse após o nome da subclasse, separada por dois pontos (`:`).

```swift
class Bicicleta: Veiculo {
    var temCesta = false
}

let minhaBicicleta = Bicicleta()
print(minhaBicicleta.velocidadeAtual) // Saída: 0.0 (herda da superclasse Veiculo)
minhaBicicleta.acelerar(por: 20.0) // Herda o método acelerar
print(minhaBicicleta.descricao) // Saída: Viajando a 20.0 km/h (herda a propriedade computada descricao)
minhaBicicleta.temCesta = true // Adiciona uma nova propriedade específica para Bicicleta
```

A classe `Bicicleta` herda todas as propriedades e métodos de `Veiculo`. Ela também adiciona sua própria propriedade, `temCesta`.

### Sobrescrita (Overriding)

Uma subclasse pode fornecer sua própria implementação para um método, propriedade ou subscript que herda de sua superclasse. Isso é conhecido como **sobrescrita**. Você indica uma sobrescrita usando a palavra-chave `override`.

```swift
class Trem: Veiculo {
    override func acelerar(por delta: Double) {
        velocidadeAtual += 2 * delta // Trem acelera mais rápido
    }
}

let meuTrem = Trem()
meuTrem.acelerar(por: 50.0)
print(meuTrem.descricao) // Saída: Viajando a 100.0 km/h
```

A subclasse `Trem` sobrescreve o método `acelerar(por:)` para modificar o comportamento de aceleração.

#### Sobrescrita de Propriedades

Uma subclasse pode sobrescrever uma propriedade herdada para fornecer seu próprio getter e setter (para propriedades armazenadas ou computadas) ou para adicionar observadores de propriedade (para propriedades armazenadas).

```swift

class Carro: Veiculo {
    var marcha = 1
    override var descricao: String {
        return super.descricao + ", na marcha \(marcha)"
    }
}

let meuCarro = Carro()
meuCarro.acelerar(por: 30.0)
meuCarro.marcha = 2
print(meuCarro.descricao) // Saída: Viajando a 30.0 km/h, na marcha 2
```

A subclasse `Carro` sobrescreve a propriedade computada `descricao` para incluir informações sobre a marcha atual. Ela usa `super.descricao` para acessar a implementação da propriedade `descricao` na superclasse.

##### Sobrescrita de Observadores de Propriedade

Você também pode adicionar observadores (`willSet` e `didSet`) a uma propriedade herdada (armazenada).

```swift
class MeuCarroEspecial: Carro {
    override var velocidadeAtual: Double {
        willSet {
            print("Preparando para mudar a velocidade para \(newValue)")
        }
        didSet {
            print("Velocidade mudou de \(oldValue) para \(velocidadeAtual)")
        }
    }
}

let meuCarroEspecial = MeuCarroEspecial()
meuCarroEspecial.velocidadeAtual = 60.0
// Saída: Preparando para mudar a velocidade para 60.0
// Saída: Velocidade mudou de 0.0 para 60.0
```

### Impedindo Sobrescritas com `final`

Você pode impedir que um método, propriedade ou subscript seja sobrescrito marcando-o com o modificador `final`. Qualquer tentativa de sobrescrever um membro `final` em uma subclasse resultará em um erro de compilação.

Você também pode marcar uma classe inteira como `final` para impedir que ela seja subclassificada.

```swift
final class ClasseInalteravel {
    // ...
}

// class SubClasseInalteravel: ClasseInalteravel { // Isso geraria um erro de compilação
//     ...
// }

class ClasseComMetodoFinal {
    final func metodoInalteravel() {
        // ...
    }
}

class SubClasseComMetodoFinal: ClasseComMetodoFinal {
    // override func metodoInalteravel() { // Isso geraria um erro de compilação
    //     ...
    // }
}
```

Relacionado: [Classes e Structs](Classes%20e%20Structs.md)
