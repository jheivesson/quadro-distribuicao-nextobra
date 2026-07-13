# Roadmap — NextObra Engineering Platform

> Este documento contém **apenas direções já discutidas de fato** para o projeto — nenhum item aqui foi inventado. Itens em "planejado"/"em estudo"/"longo prazo" não têm data de entrega definida; a inclusão aqui não é um compromisso, é um registro organizado do que já foi levantado, decidido ou aprovado.

## v1.0 — Em produção

### Funcionalidades existentes

- Três perfis de instalação completos: Residencial, Condominial e Comercial, cada um com biblioteca própria de circuitos.
- Motor de cálculo NBR 5410 (Tabelas 33–48): ampacidade, fator de temperatura, fator de agrupamento, seção de proteção, escolha de disjuntor, queda de tensão.
- Levantamento por cômodo/ponto com dimensionamento calculado ao vivo, mais formulário avançado para casos fora da biblioteca padrão.
- Quadro de Distribuição gerado em SVG (gabinete, trilhos, disjuntor geral, DR, DPS, disjuntores de circuito, barramentos de neutro/PE), com detalhamento técnico e lista de materiais (BOM).
- Relatório técnico completo exportável em PDF via impressão do navegador.
- Identificação do empreendimento (cliente, CPF/RG, endereço com geocodificação automática por CEP).
- Documentação e situação do projeto elétrico, com upload de anexos por tipo de documento.
- Casa inteligente / automação residencial como requisito que alimenta a biblioteca de circuitos.
- Biblioteca de Projetos (salvar, nomear, abrir, excluir múltiplos projetos) e controle de projeto (Novo/Exportar/Importar).
- Autosave contínuo em `localStorage`, com indicador visível de status.
- Menu lateral retrátil e barra flutuante de salvar/abrir.

## v1.1 — Em andamento (Sprint 0 aprovada pelo CTO, com ajustes)

### Funcionalidades planejadas (aprovadas)

**Evolução do Renderizador SVG do Quadro Elétrico** — evolui o mesmo motor existente (`gerarSVGQuadro()`), sem criar um segundo renderizador, conforme `TECHNICAL_RENDERING_ENGINE.md`:

| Etapa | Escopo | Status |
|---|---|---|
| 1 | Gabinete (contorno, moldura, área útil, título, capacidade) | Implementada, aguardando aprovação/deploy |
| 2 | Trilhos e área interna | Implementada, aguardando aprovação/deploy |
| 3 | Disjuntor geral | Planejada |
| 4 | DR | Planejada |
| 5 | DPS | Planejada |
| 6 | Disjuntores dos circuitos | Planejada |
| 7 | Barramentos de neutro e proteção | Planejada |
| 8 | Módulos de reserva | Planejada |
| 9 | Etiquetas e identificação | Planejada |
| 10 | Vista frontal externa | Planejada |
| 11 | Integração completa no relatório | Planejada |
| 12 | Testes de impressão e PDF | Planejada |

Cada etapa segue o processo de `CLAUDE.md` (diagnóstico → plano → aprovação → implementação → testes → relatório), uma de cada vez.

## Em estudo (sem decisão do responsável do projeto/CTO)

- **Renderizador Inteligente de Quadros Elétricos (arquitetura componentizada)** — módulo em React + TypeScript com interface de dados `ElectricalPanel`. Diagnóstico de arquitetura e estudo técnico de migração para Vite + TypeScript já concluídos; decisão de seguir **pendente**, porque exigiria abandonar a característica atual de "arquivo único, zero build, funciona offline".
- **Testes automatizados versionados** — apontado como lacuna em mais de uma auditoria; sem decisão de adoção.
- **Pipeline de build/CI-CD** — depende de decisão sobre build tooling (item acima).

## Longo prazo

- **Expansão modular da NextObra Engineering Platform** — o nome da plataforma foi escolhido para eventualmente comportar outros módulos de engenharia da NextObra além do elétrico. Até o momento, **nenhum outro módulo foi definido, aprovado ou iniciado** — esta é uma direção possível, não um roteiro fechado.
- **Módulo Industrial (elétrico)** — cogitado em algum momento, **excluído deliberadamente** do escopo por decisão já tomada. Não há biblioteca de circuitos, cálculo ou tela para esse perfil. Só retorna ao roadmap mediante nova decisão explícita.

---

Novos itens só devem ser adicionados a este roadmap quando forem, de fato, discutidos/decididos como direção real do projeto — nunca como suposição do que "poderia ser interessante".
