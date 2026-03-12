# Operadores em Swift

Em Swift, operadores são símbolos especiais que realizam operações em um ou mais valores (operandos). Swift suporta uma variedade de operadores para diferentes tipos de tarefas.

## Tipos de Operadores

Podemos categorizar os operadores em Swift da seguinte forma:

### Operadores Aritméticos

São usados para realizar operações matemáticas básicas.

* **Adição (`+`)**: Soma dois valores.
    ```swift
    let soma = 5 + 3 // soma é igual a 8
    ```
* **Subtração (`-`)**: Subtrai um valor de outro.
    ```swift
    let subtracao = 10 - 4 // subtracao é igual a 6
    ```
* **Multiplicação (`*`)**: Multiplica dois valores.
    ```swift
    let multiplicacao = 2 * 6 // multiplicacao é igual a 12
    ```
* **Divisão (`/`)**: Divide um valor por outro.
    ```swift
    let divisao = 15 / 3 // divisao é igual a 5
    ```
* **Módulo (`%`)**: Retorna o resto da divisão de um valor por outro.
    ```swift
    let resto = 10 % 3 // resto é igual a 1
    ```

[Link para a documentação oficial sobre Operadores Aritméticos](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/basicoperators#Arithmetic-Operators)

### Operadores Lógicos

São usados para combinar ou modificar valores booleanos (`true` ou `false`).

* **AND Lógico (`&&`)**: Retorna `true` se e somente se ambos os operandos forem `true`.
    ```swift
    let a = true
    let b = false
    let resultadoAnd = a && b // resultadoAnd é false
    ```
* **OR Lógico (`||`)**: Retorna `true` se pelo menos um dos operandos for `true`.
    ```swift
    let a = true
    let b = false
    let resultadoOr = a || b // resultadoOr é true
    ```
* **NOT Lógico (`!`)**: Inverte o valor booleano de um operando. Se o operando for `true`, retorna `false`, e vice-versa.
    ```swift
    let verdadeiro = true
    let falso = !verdadeiro // falso é false
    ```

[Link para a documentação oficial sobre Operadores Lógicos](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/basicoperators#Logical-Operators)

### Operadores de Comparação

São usados para comparar dois valores e retornam um valor booleano (`true` ou `false`) baseado no resultado da comparação.

* **Igual a (`==`)**: Retorna `true` se os dois operandos forem iguais.
    ```swift
    let igual = 5 == 5 // igual é true
    ```
* **Não igual a (`!=`)**: Retorna `true` se os dois operandos não forem iguais.
    ```swift
    let naoIgual = 5 != 3 // naoIgual é true
    ```
* **Maior que (`>`)**: Retorna `true` se o operando da esquerda for maior que o da direita.
    ```swift
    let maiorQue = 10 > 5 // maiorQue é true
    ```
* **Menor que (`<`)**: Retorna `true` se o operando da esquerda for menor que o da direita.
    ```swift
    let menorQue = 3 < 7 // menorQue é true
    ```
* **Maior ou igual a (`>=`)**: Retorna `true` se o operando da esquerda for maior ou igual ao da direita.
    ```swift
    let maiorOuIgual = 5 >= 5 // maiorOuIgual é true
    ```
* **Menor ou igual a (`<=`)**: Retorna `true` se o operando da esquerda for menor ou igual ao da direita.
    ```swift
    let menorOuIgual = 3 <= 5 // menorOuIgual é true
    ```

[Link para a documentação oficial sobre Operadores de Comparação](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/basicoperators#Comparison-Operators)

### Operadores de Atribuição

São usados para atribuir valores e podem ser combinados com operadores aritméticos.

```swift
var x = 10
x += 5      // x = x + 5, agora x é 15
x -= 3      // x = x - 3, agora x é 12
x *= 2      // x = x * 2, agora x é 24
x /= 4      // x = x / 4, agora x é 6
x %= 5      // x = x % 5, agora x é 1
```

### Operadores de Intervalo (Range)

Usados para criar sequências de valores.

```swift
// Intervalo fechado: inclui ambos os limites
let intervaloFechado = 1...5 // [1, 2, 3, 4, 5]
for numero in intervaloFechado {
    print(numero)
}

// Intervalo semiaberto: inclui o início mas exclui o fim
let intervaloSemiAberto = 1..<5 // [1, 2, 3, 4]
for numero in intervaloSemiAberto {
    print(numero)
}

// Intervalo unilateral
let ateMais = 1...    // [1, 2, 3, ...]
let ateMenos = ..<5   // [0, 1, 2, 3, 4]
```

### Operador Ternário

Combina uma condição com valores alternativos em uma linha.

```swift
let idade = 18
let status = idade >= 18 ? "Maior de idade" : "Menor de idade"
print(status) // Saída: Maior de idade

// Equivalente a:
let status2: String
if idade >= 18 {
    status2 = "Maior de idade"
} else {
    status2 = "Menor de idade"
}
```

---

**Observações:**

* Esta é apenas uma introdução aos operadores básicos em Swift. Existem outros tipos de operadores, como operadores bitwise, operadores de intervalo customizados, etc.
* A documentação oficial da Apple é uma excelente fonte para aprender mais sobre todos os operadores em Swift e seus detalhes.
* Operadores podem ser customizados em tipos próprios através de method overloading.
* A precedência de operadores em Swift segue convenções matemáticas (multiplicação antes de adição, etc).

[[Tipos]]
