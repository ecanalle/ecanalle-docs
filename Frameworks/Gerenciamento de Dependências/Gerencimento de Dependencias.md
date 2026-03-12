# Gerenciamento de Dependências em Swift

**Gerenciamento de Dependências** refere-se ao processo de adicionar, controlar e manter bibliotecas externas (third-party) em seus projetos. Swift oferece várias ferramentas para facilitar esse processo, cada uma com suas vantagens e casos de uso específicos.

### Por que Gerenciar Dependências?

- **Evitar reinventar a roda:** Usar bibliotecas testadas e confiáveis
- **Economizar tempo:** Reduza desenvolvimento manual
- **Melhorar qualidade:** Bibliotecas maduras costumam ser bem testadas
- **Facilitar manutenção:** Controla versões e atualizações
- **Reduzir bugs:** Menos código = menos bugs

### Ferramentas Disponíveis

#### 1. **Swift Package Manager (SPM)** ⭐

A opção **oficial e recomendada** da Apple:

- Integrado nativamente no Swift
- Sem dependências externas
- Funciona perfeitamente com Xcode
- Crescimento rápido de suporte

👉 [Ver SPM em detalhes](Swift%20Package%20Manager%20(SPM).md)

#### 2. **CocoaPods**

A ferramenta mais **tradicional** e com maior repositório:

- Milhares de bibliotecas disponíveis
- Excelente documentação
- Comunidade muito grande
- Requer Ruby e workspace

👉 [Ver CocoaPods em detalhes](CocoaPods.md)

#### 3. **Carthage**

Uma opção **descentralizada e não-invasiva**:

- Não modifica seu projeto
- Controle total sobre configuração
- Direto com repositórios Git
- Requer integração manual

👉 [Ver Carthage em detalhes](Carthage.md)

### Comparação Rápida

| Aspecto | SPM | CocoaPods | Carthage |
|---------|-----|-----------|----------|
| **Setup** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| **Integração Xcode** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐ |
| **Invasivo** | ❌ | ✅ (workspace) | ❌ |
| **Repositório** | Git | Centralizado | Git |
| **Suporte** | Crescente | Alto | Médio |
| **Requer Ruby** | ❌ | ✅ | ❌ |
| **iOS 13+** | ✅ | ✅ | ✅ |

### Como Escolher?

#### Use **SPM** se:
- ✅ É um projeto novo
- ✅ Está em iOS 13+
- ✅ Quer a solução oficial
- ✅ Prefere zero dependências externas

#### Use **CocoaPods** se:
- ✅ Precisa de uma biblioteca específica que só está lá
- ✅ Tem projeto legado
- ✅ Trabalha em times que já usam
- ✅ Precisa de configuração muito customizada

#### Use **Carthage** se:
- ✅ Quer máximo controle
- ✅ Não quer modificação de projeto
- ✅ Trabalha multiplataforma (iOS + macOS)
- ✅ Prefere descentralização

### Fluxo de Trabalho Típico

```
1. Adicionar dependência ao arquivo de config
2. Instalar/atualizar dependências
3. Integrar ao projeto (auto com SPM/CocoaPods, manual com Carthage)
4. Importar e usar bibliotecas
5. Versionamento no controle de versão
```

### Versionamento Semântico

A maioria dos gerenciadores segue **Semantic Versioning**:

```
MAJOR.MINOR.PATCH
  1.   2.     3

1.x.x = Mudanças breaking (incompatível)
x.1.x = Novas features (backward compatible)
x.x.1 = Bug fixes (backward compatible)
```

Exemplos de requerimentos:

```
~> 2.0   # Compatível com 2.x (mas não 3.0+)
>= 2.0   # Versão 2.0 ou superior
2.0...3.0  # Entre 2.0 e 3.0
= 2.0.1  # Exatamente 2.0.1
```

### Melhores Práticas

- ✅ **Versionamento:** Sempre especifique versões
- ✅ **Lock files:** Commit Podfile.lock / Cartfile.resolved
- ✅ **Auditoria:** Revise dependências regularmente
- ✅ **Updates:** Mantenha dependências atualizadas
- ✅ **Documentation:** Documente why, not just what
- ✅ **Security:** Monitore vulnerabilidades
- ✅ **Performance:** Avite dependências desnecessárias

### Segurança

- Sempre revise o código de novas dependências
- Use `--pin` ou lock files para reproducibilidade
- Monitore vulnerabilidades com ferramentas como:
  - [OWASP Dependency-Check](https://owasp.org/www-project-dependency-check/)
  - [Snyk](https://snyk.io)

---

**Relacionados:**
- [Swift Package Manager (SPM)](Swift%20Package%20Manager%20(SPM).md)
- [CocoaPods](CocoaPods.md)
- [Carthage](Carthage.md)