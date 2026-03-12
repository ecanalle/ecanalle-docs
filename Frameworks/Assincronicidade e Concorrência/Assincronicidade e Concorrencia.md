# Assincronicidade e Concorrência no Swift

**Assincronicidade:** Executar tarefas demoradas sem bloquear a interface do usuário, mantendo o aplicativo responsivo.

**Concorrência:** Executar múltiplas tarefas *aparentemente* ao mesmo tempo, seja por paralelismo (múltiplos núcleos) ou interleaving (alternância rápida).

**Swift:**

* **`async/await`:**
    * Sintaxe moderna para escrever código assíncrono.
    * Funções `async` pausam com `await` até a conclusão de operações assíncronas.
    * Impede o bloqueio da thread principal.
    * Ideal para operações assíncronas isoladas e sequenciais.

* **Combine:**
    * Framework de programação reativa para lidar com eventos ao longo do tempo.
    * Usa `Publishers` (fontes de dados) e `Subscribers` (consumidores).
    * Permite manipular fluxos de dados assíncronos de forma declarativa.
    * Mais adequado para fluxos contínuos de eventos e lógica assíncrona complexa.

**Em resumo:** `async/await` simplifica operações assíncronas pontuais, enquanto Combine oferece uma abordagem reativa para fluxos de dados assíncronos mais complexos.

Relacionado: [Async Await](Frameworks/Assincronicidade%20e%20Concorrência/Async%20Await.md) [Combine](Frameworks/Assincronicidade%20e%20Concorrência/Combine.md)