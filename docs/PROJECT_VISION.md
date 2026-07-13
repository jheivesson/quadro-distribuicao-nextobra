# Visão do Projeto — NextObra Engineering Platform

> Este documento é declaratório (propósito, missão, visão, valores) — diferente dos demais documentos em `docs/`, que descrevem só o que já existe e é verificável no código. Onde este documento faz referência a algo já implementado, o fato é real e conferido; onde descreve intenção/aspiração, isso é sinalizado explicitamente como tal, nunca como funcionalidade existente.

## Propósito da plataforma

A **NextObra Engineering Platform** existe para dar à equipe técnica da NextObra Construções uma ferramenta própria, confiável e tecnicamente rigorosa para o trabalho de engenharia elétrica em campo — substituindo cálculo manual, planilhas soltas e memória de cálculo dispersa por um fluxo único de levantamento, dimensionamento e emissão de relatório técnico.

O módulo atual da plataforma é o **Levantamento Técnico e Dimensionamento Elétrico conforme NBR 5410**, cobrindo instalações Residenciais, Condominiais e Comerciais (ver `PROJECT_OVERVIEW.md` para o que já está implementado).

## Missão

Padronizar e agilizar o levantamento técnico elétrico da NextObra, garantindo que todo dimensionamento entregue à equipe, ao escritório e ao cliente esteja **auditável, rastreável e conforme a ABNT NBR 5410** — sem depender de planilhas paralelas ou de conhecimento não documentado de quem calculou.

## Visão

Ser a plataforma interna de engenharia de referência da NextObra — nascida do módulo elétrico, mas **nomeada e estruturada para eventualmente comportar outros módulos de engenharia** da empresa, caso isso venha a ser decidido no futuro. Até o momento, **nenhum módulo além do elétrico foi definido, aprovado ou iniciado** — esta visão descreve uma direção possível, não um roteiro fechado.

## Valores técnicos

- **Rigor normativo.** Nenhum dado normativo é inventado; toda tabela/fator usado no motor de cálculo remete à NBR 5410:2004 (Tabelas 33–48) ou a uma fonte real e citável.
- **Transparência.** O relatório técnico sempre mostra o veredito de conformidade e os motivos de qualquer não conformidade — nunca esconde um circuito fora da norma.
- **Simplicidade deliberada.** A arquitetura de arquivo único, sem build, sem dependências, é uma escolha consciente enquanto atender às necessidades reais da equipe — não uma limitação a ser eliminada por padrão.
- **Incrementalidade.** Mudanças pequenas, testadas e aprovadas etapa a etapa — nunca uma reescrita silenciosa de uma vez só (ver `DEVELOPMENT_RULES.md`).
- **Auditabilidade.** Documentação viva (`docs/`), changelog real baseado em histórico de commits, e processo formal de diagnóstico → plano → aprovação → implementação → teste → relatório para qualquer mudança relevante (`CLAUDE.md`).

## Público-alvo

- **Equipe técnica de campo da NextObra** (eletricistas, técnicos e o responsável técnico) — uso principal, em campo ou em escritório.
- **Escritório técnico e clientes da NextObra** — como destinatários do relatório técnico e do PDF gerado, mesmo sem acesso direto à ferramenta.

## Diferenciais competitivos

Comparado ao método anterior (planilhas soltas e cálculo manual):

- Dimensionamento NBR 5410 calculado **ao vivo**, por cômodo/circuito, sem etapa manual de conferência de tabela.
- Bibliotecas próprias de circuitos por perfil de instalação (Residencial/Comercial/Condominial), já parametrizadas com potências e cosφ típicos do setor.
- Geração automática, no mesmo fluxo, de: quadro elétrico ilustrativo (SVG), tabela técnica detalhada, lista de materiais (BOM) e relatório em PDF — sem alternar entre ferramentas diferentes.
- Funciona **100% offline**, sem instalação, em qualquer computador com navegador — importante para uso em campo com conectividade instável.
- Construída sob medida para o fluxo de trabalho real da NextObra, não adaptada de um software genérico de terceiros.

## Princípios de engenharia

Detalhados e mantidos vivos em `DEVELOPMENT_RULES.md` e `CLAUDE.md`; resumo dos mais centrais:

- Analisar o código existente antes de implementar; nunca remover funcionalidade sem autorização explícita.
- Nunca alterar o motor de cálculo NBR 5410 sem apresentar impacto técnico; nunca inventar dado normativo.
- Preservar compatibilidade com projetos já salvos no `localStorage`.
- Toda alteração incremental, com plano apresentado e aprovado antes do código, e relatório de teste real depois.
- Não instalar dependências, não migrar arquitetura, não criar backend — sem aprovação explícita prévia.

## Objetivos de longo prazo

Refletem só o que já foi de fato discutido e estudado para este projeto (detalhado por versão em `ROADMAP.md`):

- Concluir a evolução do Renderizador SVG do Quadro Elétrico (gabinete, trilhos, disjuntores, DR/DPS, barramentos, reserva, etiquetas e vista frontal), mantendo o mesmo motor de renderização em tela e PDF.
- Avaliar, com decisão explícita do responsável do projeto, se e quando migrar para uma arquitetura componentizada (Vite + TypeScript) — estudo técnico já concluído, decisão pendente.
- Introduzir testes automatizados versionados e, eventualmente, um pipeline de build/CI-CD — hoje inexistentes.
- Manter a possibilidade de expansão modular da plataforma para além do módulo elétrico, sem comprometer o rigor normativo, a simplicidade ou a auditabilidade que a definem hoje.
