# 🔐 Segurança em Swift

Guia completo sobre segurança em desenvolvimento Swift, iOS e aplicações Apple. Inclui boas práticas, conceitos fundamentais e padrões de segurança para proteger dados dos usuários e manter a integridade da aplicação.

A segurança é uma responsabilidade fundamental no desenvolvimento de software. Este guia cobre desde conceitos básicos até implementações avançadas de segurança em plataformas Apple.

## Tópicos Principais

- **[CryptoKit](CryptoKit.md)** - Framework moderno para operações criptográficas
- **[Keychain](Keychain.md)** - Armazenamento seguro de dados sensíveis
- **[LocalAuthentication](LocalAuthentication.md)** - Autenticação biométrica (Face ID, Touch ID)
- **[HTTPS e TLS](HTTPS%20e%20TLS.md)** - Comunicação segura na rede
- **[Autenticação](Autenticação.md)** - OAuth, JWT e padrões de autenticação
- **[Armazenamento de Dados](Armazenamento%20de%20Dados.md)** - Persistência segura com Core Data e Realm
- **[OWASP Top 10](OWASP%20Top%2010.md)** - Vulnerabilidades comuns e prevenção



## Conceitos Fundamentais 

### O que é Segurança?

Segurança no desenvolvimento de software é a prática de proteger dados, sistemas e usuários contra acessos não autorizados, modificações maliciosas ou perdas de confidencialidade. Envolve múltiplas camadas de proteção.

### Pilares da Segurança

1. **Confidencialidade** - Dados devem ser acessíveis apenas para quem é autorizado
2. **Integridade** - Dados não devem ser modificados sem autorização
3. **Disponibilidade** - Serviços devem estar disponíveis quando necessário
4. **Autenticação** - Verificar a identidade de usuários e sistemas
5. **Autorização** - Controlar o que cada usuário pode fazer

---

## Guia Prático de Segurança

### ✅ Boas Práticas Geral

- Nunca armazenar senhas ou tokens em UserDefaults ou variáveis globais
- Sempre usar HTTPS para comunicação com servidores
- Validar e sanitizar todas as entradas do usuário
- Manter dependências atualizadas
- Usar code review e testes de segurança
- Implementar logging sem expor dados sensíveis
- Usar criptografia para dados em repouso

### 🚨 Erros Comuns para Evitar

- Hardcoding de credenciais ou API keys no código
- Ignorar warnings de segurança do compilador
- Usar algoritmos criptográficos desatualizados
- Confiar apenas em client-side validation
- Expor informações sensíveis em logs ou erro messages
- Não validar certificados SSL/TLS

---

Relacionados: [CryptoKit](CryptoKit.md) | [Keychain](Keychain.md) | [LocalAuthentication](LocalAuthentication.md) | [HTTPS e TLS](HTTPS%20e%20TLS.md) | [Autenticação](Autenticação.md) | [Armazenamento de Dados](Armazenamento%20de%20Dados.md) | [OWASP Top 10](OWASP%20Top%2010.md)

**Status:** ✅ Documento Base
**Última atualização:** Março de 2026

