---
RFC: 001
Título: Technical Rendering Engine — Evolução do Renderizador do Quadro Elétrico
Status: Em revisão (aguardando aprovação)
Sprint: 1 (NextObra Engineering Platform)
Documentos relacionados: docs/TECHNICAL_RENDERING_ENGINE.md, docs/ARCHITECTURE.md, docs/ROADMAP.md, CLAUDE.md
Autor: Assistente de engenharia (Claude), sob revisão do responsável técnico do projeto
Data: 2026-07-13
---

# 1. Objetivo

Formalizar, em padrão de RFC corporativa, a evolução do renderizador do quadro elétrico já em andamento neste projeto (`gerarSVGQuadro()` e funções auxiliares `svg*`, em `index.html`), consolidando em um único documento de especificação o que já foi diagnosticado, planejado, implementado e testado até aqui (Etapas 1 e 2), e o que resta especificar para as etapas seguintes.

O Technical Rendering Engine existe para que a plataforma gere uma **ilustração técnica profissional do quadro elétrico** — fiel aos dados reais do levantamento e do dimensionamento NBR 5410 — reutilizável em qualquer lugar que a plataforma precise mostrar esse quadro: tela, conferência, relatório e PDF, sempre a partir do mesmo motor, sem duplicação.

Esta RFC não introduz um motor novo. Ela **formaliza o motor existente** e especifica como ele deve continuar evoluindo.

# 2. Problema Atual

O renderizador de quadro elétrico, antes desta RFC, tinha as seguintes limitações (diagnosticadas em `TECHNICAL_RENDERING_ENGINE.md` e confirmadas por revisão crítica):

- O gabinete era desenhado como dois retângulos decorativos simples, sem título, sem legenda de capacidade e sem distinção visual entre contorno externo, moldura e área útil (**corrigido na Etapa 1**).
- Os trilhos eram uma barra cinza chapada, sem acabamento nem sensação de profundidade (**corrigido na Etapa 2**).
- Não existe, até o momento, uma vista frontal externa (gabinete fechado) — só a vista interna.
- Não existem módulos de reserva desenhados, mesmo quando o cálculo de capacidade (`escolherQuadro()`/`QUADRO_TAMANHOS`) já indica sobra de espaço.
- Não existe identificação de circuito além do número sequencial (`C1`, `C2`…) — a descrição completa só aparece em tooltip e na tabela técnica, nunca no desenho.
- A capacidade de módulos por trilho não corresponde a nenhuma referência física real — é hoje um limite arbitrário de legibilidade (`ROW_CAP = 8`), e coexiste com uma segunda regra de agrupamento por quantidade de circuitos (grupos de DR, limite de 4) — duas unidades diferentes de corte, sem uma regra normativa unificadora.
- O modelo de dados de circuito individual (`params.sistema`) só reconhece `'monofasico'` (1 polo) ou `'trifasico'` (3 polos) — **não existe circuito bipolar (2 polos)** como categoria, uma lacuna estrutural relevante para instalações bifásicas reais.
- `gerarSVGQuadro()` não tem acesso, no seu próprio escopo, ao total de módulos ocupados/capacidade de gabinete — esse cálculo (`totalModulos`, `escolherQuadro()`) só acontece dentro de `gerarBOMConsolidada()`, chamada depois, num fluxo separado.

# 3. Objetivos da RFC

Esta RFC pretende:

1. Especificar formalmente a arquitetura do renderizador (existente + proposta), como referência única para as próximas etapas de implementação.
2. Definir os **componentes lógicos** do renderizador (seção 7), mapeando cada um a uma função já existente, já implementada nesta RFC, ou ainda pendente.
3. Especificar o fluxo de dados completo, do levantamento técnico até o SVG final exibido/impresso.
4. Deixar explícitas as regras de compatibilidade que qualquer etapa futura precisa respeitar (projetos antigos, PDF, impressão, `localStorage`).
5. Consolidar, num único lugar, os riscos técnicos já identificados (incluindo os ainda não resolvidos, como o gap de circuito bipolar e a ambiguidade de trilhos).
6. Definir critérios de aceitação objetivos para considerar o Technical Rendering Engine concluído.

# 4. Escopo

**Faz parte desta RFC:**

- A especificação da arquitetura de renderização do quadro elétrico (vista interna e vista frontal externa).
- A especificação de todos os componentes lógicos listados na seção 7.
- O fluxo de dados entre o levantamento técnico, o motor de cálculo NBR 5410 e o renderizador.
- As regras de compatibilidade com impressão, PDF, `localStorage` e projetos salvos.
- A estratégia de implementação em etapas pequenas (já em andamento desde a Sprint 0).

**NÃO faz parte desta RFC:**

- Qualquer alteração ao motor de cálculo NBR 5410 (`dimensionar()` e funções de apoio) — regra permanente de `CLAUDE.md`.
- Migração de arquitetura (React, TypeScript, Vite, build tooling) — segue como item "em estudo" no `ROADMAP.md`, sem decisão tomada.
- Resolver, nesta RFC, o gap estrutural de circuito bipolar (2 polos) — é registrado como risco (seção 11) e decisão pendente, não como algo a implementar aqui.
- Definir uma tabela normativa real de "módulos por trilho por fabricante" — não pode ser inventada; permanece como decisão pendente.
- Qualquer mudança de backend, banco de dados ou autenticação.
- Implementação de código — esta RFC é apenas especificação.

# 5. Estado Atual

## Arquitetura existente

O renderizador vive inteiramente dentro de `index.html`, como um conjunto de funções JavaScript que retornam fragmentos de string SVG, concatenados por uma função orquestradora (`gerarSVGQuadro()`) e inseridos via `innerHTML`. Não há componentização por arquivo, não há framework, não há build — consistente com a arquitetura geral da plataforma (`ARCHITECTURE.md`).

O SVG resultante é o **mesmo nó DOM** usado na tela, na conferência e na impressão/PDF (via `window.print()`) — não há segunda renderização nem captura de tela.

## Funções existentes (estado atual, pós Etapas 1 e 2)

| Função | Linha | Responsabilidade |
|---|---|---|
| `svgToggle()` | L2661 | Alavanca liga/desliga de disjuntor/DR/DPS |
| `svgSombraFalsa()` | L2674 | Sombra falsa em retângulo sólido (sem `<filter>`/gradiente) |
| `svgBreaker()` | L2678 | Disjuntor termomagnético (geral ou de circuito) |
| `svgRCD()` | L2694 | Dispositivo DR |
| `svgDPS()` | L2706 | Dispositivo DPS |
| `svgRail()` | L2717 | Trilho DIN — **evoluído na Etapa 2** (sombra de encaixe, realce, perfurações) |
| `svgFaseBars()` | L2743 | Barras de fase (1/2/3, conforme tipo de entrada) |
| `svgTerminalBar()` | L2759 | Barramento genérico (reaproveitado para Neutro e PE) |
| `svgGabinete()` | L2774 | Gabinete — **novo na Etapa 1** (contorno, moldura, área útil, título, legenda de capacidade) |
| `gerarSVGQuadro()` | L2790 | Orquestrador — monta o SVG completo |
| `detectarSubQuadros()` | L3117 | Agrupa circuitos elegíveis a sub-quadro (CFTV/Automação) |
| `subQuadroHTML()` | L3126 | Reaproveita `gerarSVGQuadro()` para desenhar sub-quadros |

## Fluxo atual

1. Usuário monta o levantamento técnico (cômodos + formulário avançado) → `circuitos[]`.
2. Ao clicar "Gerar Quadro", `gerarSVGQuadro(circuitos, tipoEntrada)` monta o SVG (gabinete → barras de fase → barramento N → linhas de trilho+dispositivos → barramento PE).
3. O SVG é injetado em `#quadro-conteudo`, junto com ficha técnica, veredito, tabela técnica e BOM.
4. "Gerar PDF" aciona `window.print()` sobre o mesmo DOM.

# 6. Arquitetura Proposta

A arquitetura proposta **preserva o motor único** (`gerarSVGQuadro()` + funções `svg*`), evoluindo-o por composição, não por substituição:

- Cada componente lógico (seção 7) permanece uma função dedicada, de responsabilidade única, que desenha só a própria peça — nenhuma função desenha o que é responsabilidade de outra (regra já seguida desde antes desta RFC, mantida como princípio arquitetural formal).
- `gerarSVGQuadro()` continua como único orquestrador, responsável por: montar a lista de linhas/dispositivos, calcular dimensões do canvas, e chamar os componentes na ordem correta.
- A **vista frontal externa** (`ExternalView`, ainda não implementada) é especificada como um componente que **compõe** o resultado de `gerarSVGQuadro()`, análogo a como `subQuadroHTML()` hoje compõe o resultado para sub-quadros — nunca como um segundo motor paralelo.
- Nenhum dado novo (nome do quadro, localização, etiquetas) é obrigatório — todo componente que depende de um dado ainda inexistente no projeto deve omitir graciosamente o elemento correspondente, nunca quebrar o desenho.

# 7. Componentes

| Componente | Mapeamento na implementação atual | Status |
|---|---|---|
| **Cabinet** | `svgGabinete()`, L2774 — contorno externo, moldura interna, área útil, título genérico, legenda de capacidade | ✅ Implementado (Etapa 1) |
| **Rails** | `svgRail()`, L2717 — trilho DIN com sombra de encaixe, realce e perfurações | ✅ Implementado (Etapa 2) |
| **Busbars** | `svgTerminalBar()` (L2759, Neutro/PE) + `svgFaseBars()` (L2743, fases) | Existente, não evoluído nesta RFC ainda — previsto na Etapa 7 |
| **Breakers** | `svgBreaker()`, L2678 — usado tanto para o disjuntor geral quanto para os disjuntores de circuito | Existente, evolução prevista nas Etapas 3 (geral) e 6 (circuitos) |
| **DR** | `svgRCD()`, L2694 | Existente, evolução prevista na Etapa 4 |
| **DPS** | `svgDPS()`, L2706 | Existente, evolução prevista na Etapa 5 |
| **Labels** | Hoje distribuído dentro de cada função (`label1`/`label2` em `svgBreaker`/`svgRCD`/`svgDPS`, título/legenda em `svgGabinete`) — não é uma função dedicada ainda | Não existe como componente próprio — previsto na Etapa 9 |
| **ReserveModules** | Não existe | Não implementado — previsto na Etapa 8 |
| **ExternalView** | Não existe | Não implementado — previsto na Etapa 10 |
| **InternalView** | É o que `gerarSVGQuadro()` já produz como um todo hoje (vista de tampa aberta) | Existente, formalizado por esta RFC como conceito — integração final prevista na Etapa 11 |

Nenhum destes componentes é um arquivo separado ou um componente de framework — são funções JavaScript dentro de `index.html`, consistente com a arquitetura atual da plataforma (ver seção 4, "não faz parte desta RFC").

# 8. Fluxo de Dados

```
Levantamento Técnico (cômodos + formulário avançado)
        │
        ▼
   circuitos[] (id, ambiente, categoria, descrição, exigeDR, params, resultado)
        │
        ▼
   Motor de cálculo NBR 5410 (dimensionar() e funções de apoio)
        │  → já executado no momento em que o circuito é adicionado a circuitos[]
        ▼
   gerarSVGQuadro(circuitos, tipoEntrada)
        │  → agrupa por DR, monta linhas, calcula módulos ocupados,
        │    chama Cabinet → Busbars(N) → linhas de Rails+Breakers/DR/DPS → Busbars(PE)
        ▼
   String SVG (mesmo nó para tela e impressão)
        │
        ├──► Tela (innerHTML em #quadro-conteudo)
        └──► PDF (window.print() sobre o mesmo DOM, via @media print)
```

Nenhum dado é buscado fora de `circuitos[]`/`tipoEntrada` — o renderizador não acessa `localStorage` nem faz nenhuma chamada de rede.

# 9. Compatibilidade

- **Projetos antigos:** `gerarSVGQuadro()` é uma função derivada, sem leitura/escrita de `localStorage`. Qualquer novo dado visual (nome do quadro, etiquetas, localização) deve ser **opcional**: ausente em projeto antigo → elemento correspondente simplesmente omitido no desenho, nunca gerando erro nem texto `undefined` literal (guard explícito obrigatório em cada novo trecho).
- **PDF:** o mesmo SVG exibido na tela é o que vai para o PDF via `window.print()` — nenhuma segunda renderização. Toda nova peça deve ser validada com geração real de PDF, não só inspeção em tela (como já foi feito nas Etapas 1 e 2).
- **Impressão:** três regras permanentes, já testadas e reafirmadas nesta RFC:
  1. Nunca `<linearGradient>`/`<filter>` via `url()` em conteúdo SVG.
  2. Nunca `border-radius` (CSS) em elemento sujeito a quebra de página — `rx`/`ry` do próprio SVG são seguros e devem continuar sendo usados.
  3. Sempre `width`/`height` explícitos no `<svg>` raiz, além do `viewBox`.
- **`localStorage`:** o formato do projeto salvo (`lerProjeto()`/`aplicarProjeto()`) não é alterado por esta RFC. Qualquer campo novo necessário para um componente futuro (ex.: nome do quadro para `Labels`) precisa ser opcional e retrocompatível, com estratégia de migração explícita se algum dia deixar de ser opcional (regra permanente de `CLAUDE.md`).

# 10. Estratégia de Implementação

Mantém a divisão já aprovada e em andamento desde a Sprint 0, uma etapa por vez, cada uma com diagnóstico → plano → aprovação → implementação → testes → relatório (processo de `CLAUDE.md`):

| Etapa | Componente(s) | Status |
|---|---|---|
| 1 — Gabinete | Cabinet | ✅ Implementada |
| 2 — Trilhos e área interna | Rails | ✅ Implementada |
| 3 — Disjuntor geral | Breakers (geral) | Planejada |
| 4 — DR | DR | Planejada |
| 5 — DPS | DPS | Planejada |
| 6 — Disjuntores dos circuitos | Breakers (circuitos) | Planejada |
| 7 — Barramentos de neutro e proteção | Busbars | Planejada |
| 8 — Módulos de reserva | ReserveModules | Planejada |
| 9 — Etiquetas e identificação | Labels | Planejada |
| 10 — Vista frontal externa | ExternalView | Planejada |
| 11 — Integração completa no relatório | InternalView (consolidação) | Planejada |
| 12 — Testes de impressão e PDF | Todos (validação final) | Planejada |

Cada etapa é independente na entrega, mas respeita as dependências naturais já identificadas (ex.: `ReserveModules` depende de acesso ao total de módulos hoje só calculado em `gerarBOMConsolidada()` — ver risco correspondente na seção 11).

# 11. Riscos

1. **Gap estrutural de circuito bipolar.** `params.sistema` só reconhece 1 ou 3 polos — um circuito real de 2 polos (comum em instalações bifásicas) não tem representação hoje. Fora do escopo desta RFC resolver, mas é um risco relevante para qualquer etapa que prometa "cobrir todos os tipos de circuito".
2. **Ambiguidade de capacidade por trilho.** Duas unidades de corte diferentes coexistem (grupos de DR por quantidade de circuitos vs. linhas sem DR por soma de módulos) — qualquer regra nova de "módulos por trilho" precisa decidir o que fazer com essa inconsistência antes de ser implementada.
3. **Acoplamento entre `gerarSVGQuadro()` e `gerarBOMConsolidada()`.** O total de módulos/capacidade de gabinete só existe hoje na função da BOM, chamada depois do SVG no fluxo real — a Etapa 8 (Módulos de reserva) precisa resolver essa dependência de ordem de chamada nos dois pontos de uso (quadro geral e sub-quadros).
4. **Legibilidade de texto em espaço fixo.** Módulos de disjuntor têm ~46px úteis de largura — qualquer etiqueta mais longa (Etapa 9) corre risco real de ficar ilegível ou sobrepor elementos vizinhos.
5. **Regressão no bug de impressão já resolvido.** Histórico real neste projeto (conteúdo sumindo, página em branco, orientação errada) — toda nova peça deve ser validada com PDF real, não só tela.
6. **Densidade visual em quadros grandes.** Vista frontal + reserva + etiquetas maiores, somadas num quadro de 40+ módulos, podem estourar um número de páginas aceitável — precisa de teste com o pior caso real antes de considerar a Etapa 12 concluída.
7. **Ponto de integração da vista frontal ainda não decidido tecnicamente.** Precisa ficar claro, antes da Etapa 10, se `ExternalView` é parte do mesmo `<svg>` de `gerarSVGQuadro()` ou uma função que compõe o resultado ao lado dele — para não criar acidentalmente um segundo motor.

# 12. Critérios de Aceitação

- Todos os 10 componentes da seção 7 implementados e mapeados a uma função dedicada, cada uma com responsabilidade única.
- `gerarSVGQuadro()` continua como único orquestrador — nenhuma duplicação de motor de renderização.
- Nenhuma alteração em `dimensionar()` ou em qualquer função do motor de cálculo NBR 5410.
- PDF gerado via `window.print()` sem conteúdo cortado, sem página em branco, sem elemento sumindo — validado com geração real de PDF para os casos mono/bi/trifásico e para quadro pequeno/grande.
- Nenhum `<filter>`/gradiente via `url()`; nenhum `border-radius` (CSS) em elemento sujeito a quebra de página.
- Todo dado novo exibido no desenho é opcional — ausência do dado não quebra o desenho de um projeto salvo antes desta RFC.
- Nenhuma biblioteca externa instalada; nenhuma migração de arquitetura realizada como parte desta RFC.
- Os riscos 1, 2 e 3 da seção 11, mesmo que não resolvidos, estão explicitamente documentados como decisão pendente antes do fechamento da Etapa 8/9 correspondente.

---

Esta RFC não implementa nada — é apenas especificação. Toda implementação segue etapa a etapa, com plano e aprovação prévios, conforme já estabelecido em `CLAUDE.md`.
