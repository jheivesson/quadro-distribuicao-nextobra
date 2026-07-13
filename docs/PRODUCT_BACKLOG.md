# Product Backlog — NextObra Engineering Platform

> Este documento organiza toda a plataforma em Épicos, com base **exclusivamente no que já foi verificado no código real** (`index.html`) e no que já foi formalmente estudado/aprovado nos documentos existentes (`ARCHITECTURE.md`, `PROJECT_OVERVIEW.md`, `TECHNICAL_RENDERING_ENGINE.md`, `RFC-001`). Onde uma funcionalidade não existe e não há decisão tomada sobre ela, isso está declarado explicitamente como "não existe / sem decisão" — nunca inventado como plano.

## Como ler este documento

Cada Épico contém: Objetivo, Valor para o usuário, Funcionalidades (cada uma marcada como **Existe** / **Parcial** / **Não existe**), Dependências e Prioridade. A tabela consolidada (seção final) resume tudo para leitura rápida.

---

## EPIC 01 — Levantamento Técnico

**Objetivo:** capturar, em campo ou em escritório, todos os dados de uma instalação elétrica (identificação, perfil, cômodos/pontos, circuitos) necessários para o dimensionamento NBR 5410.

**Valor para o usuário:** elimina anotação em papel/planilha solta; o técnico monta o levantamento direto na ferramenta, já vendo o resultado do dimensionamento em tempo real.

**Funcionalidades:**

| Funcionalidade | Objetivo | Estado |
|---|---|---|
| Seleção de perfil (Residencial/Condominial/Comercial) | Definir qual biblioteca de ambientes e regras se aplicam | **Existe** |
| Identificação do empreendimento (cliente, CPF/RG, endereço, CEP com geocodificação automática) | Capturar dados cadastrais do cliente/obra | **Existe** |
| Configuração da instalação (tipo de entrada mono/bi/trifásica) | Definir parâmetros gerais do quadro | **Existe** |
| Documentação e situação do projeto elétrico (com upload de anexos) | Registrar o que já existe de documentação técnica | **Existe** |
| Cômodos e pontos com dimensionamento ao vivo | Entrada rápida por ambiente, com cálculo NBR 5410 imediato | **Existe** |
| Circuito personalizado (avançado) | Cobrir casos fora da biblioteca padrão | **Existe** |
| Casa inteligente / automação residencial | Capturar requisitos de automação que viram circuitos | **Existe** |
| Circuito bipolar (2 polos) como categoria de dado | Representar corretamente cargas 220V entre duas fases (comum em instalação bifásica real) | **Não existe** — hoje `params.sistema` só reconhece monofásico (1 polo) ou trifásico (3 polos) |
| Balanceamento de fases entre circuitos monofásicos (R/S/T) | Distribuir carga monofásica entre as fases disponíveis num quadro trifásico | **Não existe** |

**Dependências:** EPIC 02 (Motor NBR5410) — todo cálculo exibido no levantamento vem de lá.

**Prioridade:** Alta (núcleo do produto, já entregue; gaps conhecidos ficam em manutenção evolutiva).

---

## EPIC 02 — Motor NBR5410

**Objetivo:** executar o dimensionamento elétrico normativo (ampacidade, quedas de tensão, disjuntores, condutor de proteção) conforme a ABNT NBR 5410:2004.

**Valor para o usuário:** garante que todo circuito do levantamento tenha um resultado técnico correto e auditável, sem depender de tabela consultada manualmente.

**Funcionalidades:**

| Funcionalidade | Objetivo | Estado |
|---|---|---|
| Ampacidade base do condutor (Tabela 36) | Base do dimensionamento de seção | **Existe** |
| Fator de correção por temperatura (Tabela 40) | Corrigir ampacidade pela temperatura ambiente | **Existe** |
| Fator de correção por agrupamento (Tabela 42) | Corrigir ampacidade por agrupamento de circuitos | **Existe** |
| Seção do condutor de proteção/PE (Tabela 58) | Dimensionar o condutor terra | **Existe** |
| Escolha do disjuntor (corrente nominal) | Selecionar disjuntor comercial compatível | **Existe** |
| Cálculo de queda de tensão | Verificar conformidade de queda % | **Existe** |
| Escolha de IDR/DR | Selecionar disjuntor diferencial por grupo | **Existe** |
| Escolha de tamanho de gabinete (`escolherQuadro`/`QUADRO_TAMANHOS`) | Recomendar o gabinete comercial compatível com a capacidade calculada | **Existe** (mas hoje só usado na lista de materiais, não no desenho do quadro — ver EPIC 03) |
| Suporte a circuito bipolar (2 polos) no cálculo | Complementar o gap do EPIC 01 no próprio motor | **Não existe** |
| Tabela real de módulos por trilho por fabricante | Basear fisicamente a divisão de trilhos do quadro | **Não existe** — não pode ser inventada sem referência normativa/comercial real |

**Dependências:** nenhuma (é o núcleo de cálculo, consumido por todos os outros épicos).

**Prioridade:** Alta (núcleo do produto, já entregue e validado; qualquer mudança exige análise de impacto formal, conforme `CLAUDE.md`).

---

## EPIC 03 — Technical Rendering Engine

**Objetivo:** gerar a ilustração técnica do quadro elétrico (SVG), reutilizável em tela, conferência, relatório e PDF, a partir do mesmo motor de renderização — conforme especificado em `RFC-001`.

**Valor para o usuário:** visualização profissional do quadro elétrico, sem depender de desenho manual externo ou de imagem estática.

**Funcionalidades (mapeadas aos 10 componentes da RFC-001):**

| Funcionalidade | Objetivo | Estado |
|---|---|---|
| Cabinet (gabinete: contorno, moldura, área útil, título, capacidade) | Moldura visual profissional do quadro | **Existe** (Etapa 1, Sprint 1) |
| Rails (trilhos DIN) | Representar o trilho de fixação dos dispositivos | **Existe** (Etapa 2, Sprint 1) |
| Busbars (barramentos de neutro e proteção) | Representar os barramentos N e PE | **Parcial** — existe (`svgTerminalBar`/`svgFaseBars`), ainda não evoluído nesta iniciativa (Etapa 7 planejada) |
| Breakers (disjuntor geral e de circuitos) | Representar os disjuntores termomagnéticos | **Parcial** — existe (`svgBreaker`), evolução prevista nas Etapas 3 e 6 |
| DR | Representar o dispositivo diferencial residual | **Parcial** — existe (`svgRCD`), evolução prevista na Etapa 4 |
| DPS | Representar a proteção contra surtos | **Parcial** — existe (`svgDPS`), evolução prevista na Etapa 5 |
| Labels (etiquetas e identificação de circuito) | Identificação legível de cada circuito no próprio desenho | **Não existe** como componente dedicado — hoje é texto embutido em cada função (Etapa 9 planejada) |
| ReserveModules (módulos de reserva) | Mostrar visualmente a sobra de capacidade do gabinete | **Não existe** (Etapa 8 planejada) |
| ExternalView (vista frontal externa) | Mostrar o gabinete fechado, como visto em campo | **Não existe** (Etapa 10 planejada) |
| InternalView (vista interna consolidada) | Consolidar a vista de "tampa aberta" já existente | **Existe** como conceito (é o que `gerarSVGQuadro()` já produz); integração final prevista na Etapa 11 |

**Dependências:** EPIC 01 (dados de `circuitos[]`), EPIC 02 (resultado do cálculo por circuito).

**Prioridade:** Alta — Sprint 1 aprovada pelo CTO, em andamento (2 de 12 etapas concluídas).

---

## EPIC 04 — Relatórios

**Objetivo:** consolidar o levantamento, o cálculo e o desenho do quadro num relatório técnico completo, exportável em PDF.

**Valor para o usuário:** um único documento pronto para enviar ao escritório ou ao cliente, sem montagem manual.

**Funcionalidades:**

| Funcionalidade | Objetivo | Estado |
|---|---|---|
| Ficha técnica (identificação, situação do projeto, automação) | Resumo cadastral no relatório | **Existe** |
| Veredito de conformidade geral | Indicar se todos os circuitos estão dentro da norma | **Existe** |
| Detalhamento técnico por circuito (tabela) | Mostrar todos os parâmetros e resultados calculados | **Existe** |
| Lista de materiais (BOM) | Consolidar quantitativos de disjuntores, condutores, quadro | **Existe** |
| Sub-relatórios de sub-quadros (CFTV, Automação) | Detalhar circuitos especiais separadamente | **Existe** |
| Assinatura do responsável técnico | Formalizar o documento | **Existe** |
| Exportação em PDF (`window.print()`) | Gerar arquivo entregável | **Existe** |
| Envio automático do relatório (e-mail, etc.) | Distribuir o relatório sem passo manual de download/envio | **Não existe** |
| Versionamento de relatório (histórico de revisões) | Rastrear mudanças entre gerações do mesmo relatório | **Não existe** |

**Dependências:** EPIC 01, EPIC 02, EPIC 03.

**Prioridade:** Alta (já entregue; evolui organicamente junto com o EPIC 03).

---

## EPIC 05 — Clientes

**Objetivo:** representar os dados do cliente/empreendimento associados a um levantamento.

**Valor para o usuário:** identificação clara de para quem é o levantamento, com dados cadastrais completos no relatório.

**Funcionalidades:**

| Funcionalidade | Objetivo | Estado |
|---|---|---|
| Dados cadastrais do cliente (nome, CPF, RG, órgão emissor) | Identificar o cliente no relatório | **Existe** (dentro do projeto, não como entidade própria) |
| Endereço com geocodificação automática (CEP) | Localizar a obra automaticamente | **Existe** |
| Cadastro de clientes reutilizável entre projetos | Evitar redigitar dados de um cliente recorrente | **Não existe** — hoje cada projeto tem seus próprios dados de cliente, sem uma base de clientes separada |
| Histórico de projetos por cliente | Consultar todos os levantamentos já feitos para um cliente | **Não existe** |
| Portal/envio direto ao cliente | Cliente acessar o relatório sem intermediação da equipe | **Não existe** |

**Dependências:** EPIC 06 (Projetos) — dados de cliente hoje vivem dentro do projeto salvo.

**Prioridade:** Média (funcional hoje via projeto individual; virar uma entidade própria é uma evolução real, mas sem decisão tomada).

---

## EPIC 06 — Projetos

**Objetivo:** persistir, exportar, importar e controlar o ciclo de vida de um levantamento técnico completo.

**Valor para o usuário:** não perder trabalho em andamento; poder retomar, exportar como arquivo de backup, ou recomeçar do zero.

**Funcionalidades:**

| Funcionalidade | Objetivo | Estado |
|---|---|---|
| Autosave contínuo em `localStorage` | Nunca perder progresso não salvo manualmente | **Existe** |
| Indicador visível de autosave | Dar confiança visual de que o dado está salvo | **Existe** |
| Novo Projeto | Começar do zero, limpando o estado atual | **Existe** |
| Exportar Projeto (.json) | Backup/transferência manual de um projeto | **Existe** |
| Importar Projeto (.json) | Restaurar/transferir um projeto entre dispositivos | **Existe** |
| Barra flutuante de salvar/abrir | Acesso rápido às ações mais usadas | **Existe** |
| Versionamento de revisões dentro do mesmo projeto | Rastrear o histórico de mudanças de um levantamento | **Não existe** |
| Sincronização entre dispositivos | Acessar o mesmo projeto de computadores diferentes sem exportar/importar manualmente | **Não existe** — depende do EPIC 09 (Banco de Dados) |
| Colaboração multiusuário em tempo real | Duas pessoas editando o mesmo projeto simultaneamente | **Não existe** |

**Dependências:** EPIC 01 (dados que compõem o projeto).

**Prioridade:** Alta (já entregue e robusto para uso individual em um dispositivo).

---

## EPIC 07 — Biblioteca

**Objetivo:** oferecer bibliotecas de referência reutilizáveis — tanto de projetos salvos quanto de tipos de circuito/ambiente.

**Valor para o usuário:** reaproveitar projetos e não precisar digitar parâmetros técnicos padrão a cada novo circuito.

**Funcionalidades:**

| Funcionalidade | Objetivo | Estado |
|---|---|---|
| Biblioteca de Projetos (salvar, nomear, abrir, excluir) | Gerenciar múltiplos projetos no mesmo navegador | **Existe** |
| Biblioteca de ambientes/circuitos (162 itens: Residencial/Comercial/Condominial) | Fornecer parâmetros técnicos padrão por tipo de ponto | **Existe** (fixa no código-fonte) |
| Edição da biblioteca de ambientes pelo usuário | Permitir adicionar/ajustar itens da biblioteca sem editar código | **Não existe** — hoje é hardcoded, só um desenvolvedor pode alterar |
| Biblioteca compartilhada entre usuários/dispositivos | Acessar a mesma biblioteca de projetos em qualquer computador | **Não existe** — depende do EPIC 09 (Banco de Dados) |

**Dependências:** EPIC 01 (biblioteca de ambientes), EPIC 06 (biblioteca de projetos).

**Prioridade:** Alta para a parte já entregue (Biblioteca de Projetos); sem prioridade definida para a biblioteca de ambientes editável (sem decisão tomada).

---

## EPIC 08 — Autenticação

**Objetivo:** controlar o acesso à ferramenta.

**Valor para o usuário:** impedir uso por pessoas fora da equipe técnica.

**Funcionalidades:**

| Funcionalidade | Objetivo | Estado |
|---|---|---|
| Login com credencial fixa | Barreira visual de acesso | **Existe** — mas é simbólico: credencial fixa comparada no JavaScript do cliente, sem backend |
| Sessão persistida (`localStorage`) | Manter login entre acessos até "Sair" | **Existe** |
| Autenticação real (hash de senha, sessão de servidor) | Segurança de acesso de fato | **Não existe** |
| Múltiplos usuários com credenciais próprias | Rastrear quem fez cada levantamento | **Não existe** |
| Papéis/permissões (ex.: técnico vs. administrador) | Diferenciar níveis de acesso | **Não existe** |
| Recuperação de senha | Autoatendimento em caso de esquecimento | **Não existe** |

**Dependências:** nenhuma hoje; qualquer evolução para autenticação real dependeria do EPIC 09 (Banco de Dados/backend).

**Prioridade:** Baixa — funciona como portão simbólico hoje; evoluir para autenticação real não tem decisão tomada.

---

## EPIC 09 — Banco de Dados

**Objetivo:** persistir dados da plataforma fora do navegador local.

**Valor para o usuário:** acesso multi-dispositivo, backup automático, e base para autenticação real e biblioteca compartilhada.

**Funcionalidades:**

| Funcionalidade | Objetivo | Estado |
|---|---|---|
| Persistência local via `localStorage` | Guardar projeto atual e Biblioteca no navegador | **Existe** (não é um banco de dados — é armazenamento local do navegador) |
| Banco de dados remoto (servidor) | Persistir dados fora do navegador do usuário | **Não existe** |
| Sincronização multi-dispositivo | Acessar os mesmos dados de qualquer computador | **Não existe** |
| Backup automático em nuvem | Proteger contra perda de dados do navegador local | **Não existe** |
| Controle de acesso a dados por usuário/papel | Restringir quem vê o quê | **Não existe** |

**Dependências:** nenhuma tecnicamente, mas é pré-requisito para evoluções reais dos EPIC 05, 06, 07 e 08.

**Prioridade:** Não definida — sem decisão do responsável do projeto/CTO. Introduzir banco de dados muda a arquitetura "zero-build, funciona offline" da plataforma (ver `PROJECT_OVERVIEW.md`).

---

## EPIC 10 — Inteligência Artificial

**Objetivo:** *não definido.*

**Valor para o usuário:** *não definido.*

**Funcionalidades:** **nenhuma existe, nenhuma é parcial, e nenhuma foi sequer formalmente discutida ou estudada até o momento.** Este épico é criado aqui como um placeholder estrutural do backlog, sem escopo, sem funcionalidades levantadas e sem qualquer decisão tomada. Nenhuma funcionalidade de IA foi implementada, planejada ou aprovada nesta plataforma até a data deste documento.

**Dependências:** não aplicável — sem escopo definido, não é possível apontar dependências reais.

**Prioridade:** Não definida — sem decisão do responsável do projeto/CTO sobre se, quando ou como este épico será escopado.

---

## Tabela consolidada

| ID | Nome | Prioridade | Complexidade | Dependências | Status |
|---|---|---|---|---|---|
| EPIC-01 | Levantamento Técnico | Alta | Média | EPIC-02 | Concluído (com gaps conhecidos documentados) |
| EPIC-02 | Motor NBR5410 | Alta | Alta | — | Concluído |
| EPIC-03 | Technical Rendering Engine | Alta | Alta | EPIC-01, EPIC-02 | Em andamento (2/12 etapas) |
| EPIC-04 | Relatórios | Alta | Média | EPIC-01, EPIC-02, EPIC-03 | Concluído |
| EPIC-05 | Clientes | Média | Média | EPIC-06 | Parcial |
| EPIC-06 | Projetos | Alta | Baixa | EPIC-01 | Concluído |
| EPIC-07 | Biblioteca | Alta / Não definida* | Média | EPIC-01, EPIC-06 | Parcial |
| EPIC-08 | Autenticação | Baixa | Alta (se evoluir) | EPIC-09 (se evoluir) | Concluído como portão simbólico / Não iniciado como autenticação real |
| EPIC-09 | Banco de Dados | Não definida | Alta | — | Não iniciado |
| EPIC-10 | Inteligência Artificial | Não definida | Não estimável (sem escopo) | Não aplicável | Sem decisão / Não iniciado |

*EPIC-07: prioridade Alta para a parte já entregue (Biblioteca de Projetos); Não definida para a biblioteca de ambientes editável.

---

Nenhum código foi alterado para produzir este documento. Onde uma funcionalidade não existe e não há decisão tomada sobre ela, isso foi declarado explicitamente — nenhum item deste backlog foi inventado.
