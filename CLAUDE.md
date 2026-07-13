# Identidade do projeto

Nome oficial:
NextObra Engineering Platform

Nome do módulo atual:
Levantamento Técnico e Dimensionamento Elétrico conforme NBR 5410

# Estado tecnológico atual

- HTML
- CSS
- JavaScript puro
- Aplicação concentrada atualmente em `index.html`
- SVG dinâmico para representação do quadro
- `window.print()` para impressão e PDF
- `localStorage` para persistência
- sem backend
- sem banco de dados remoto
- sem framework
- sem sistema de build

# Regras obrigatórias

1. Antes de implementar, analisar o código existente.
2. Nunca remover funcionalidades existentes sem autorização explícita.
3. Nunca modificar o motor de cálculo NBR 5410 sem apresentar impacto técnico.
4. Nunca alterar fórmulas, tabelas ou fatores normativos silenciosamente.
5. Nunca inventar dados normativos.
6. Nunca substituir o SVG atual por imagens PNG ou JPG.
7. A mesma representação do quadro deve funcionar na tela e na impressão.
8. Não instalar dependências sem aprovação.
9. Não migrar para React, Vite, Next.js ou TypeScript sem aprovação.
10. Não criar backend ou banco de dados sem aprovação.
11. Toda alteração deve ser incremental.
12. Toda tarefa deve informar arquivos que serão criados e modificados.
13. Antes de escrever código, apresentar plano de implementação.
14. Depois de implementar, apresentar relatório das alterações.
15. Não alterar credenciais, configurações de deploy ou Netlify sem aprovação.
16. Preservar compatibilidade com projetos já salvos no localStorage.
17. Não alterar o formato do JSON dos projetos sem estratégia de migração.
18. Evitar números mágicos.
19. Manter regras de negócio separadas da renderização sempre que possível.
20. Não duplicar funções existentes.
21. Não afirmar que algo foi testado sem realmente executar o teste.
22. Em caso de dúvida, parar e pedir aprovação.

# Processo obrigatório por tarefa

Fase 1 — Diagnóstico
Fase 2 — Plano
Fase 3 — Aprovação
Fase 4 — Implementação
Fase 5 — Testes
Fase 6 — Relatório final

# Segurança

- Não expor senhas no código.
- Não registrar credenciais em documentação.
- Não colocar segredos em commits.
- Alertar quando encontrar credenciais hardcoded.
- Não modificar autenticação sem autorização.

# Convenções

- Nomes de funções e variáveis em português podem ser mantidos no código atual.
- Novos nomes devem ser claros e consistentes.
- Funções devem possuir responsabilidade única.
- Comentários devem explicar decisões, não repetir o código.
- Novas constantes devem ser centralizadas quando possível.
