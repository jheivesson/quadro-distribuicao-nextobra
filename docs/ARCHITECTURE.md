# Arquitetura

## Como o projeto funciona hoje

O projeto é um **arquivo HTML único e autocontido** (`index.html`, ~3.200 linhas), sem framework, sem bundler e sem backend. Ele se divide em três blocos dentro do mesmo arquivo:

- **`<style>`** — CSS da aplicação: design tokens (cores, tipografia), layout das telas, regras de impressão (`@media print`).
- **corpo HTML** — todas as telas da aplicação, como `<section>`s que ficam visíveis ou escondidas (`style.display`) conforme a navegação do usuário. Não há roteamento nem URLs distintas por tela.
- **`<script>`** — toda a lógica: cálculo NBR 5410, renderização de listas/checklists, persistência, autenticação e geração do relatório/quadro.

Não existe estado gerenciado por framework (sem Context/Redux/Zustand). O estado vive em **variáveis globais do módulo**, e a atualização da tela é feita **manualmente**: cada função que muda um dado chama, na sequência, a função de renderização correspondente (não há reatividade automática).

## Fluxo de dados

1. O usuário interage com um campo/checkbox → um `addEventListener` captura o evento.
2. O dado é lido do formulário e, quando aplicável, passa pelo motor de cálculo NBR 5410 (`dimensionar()` e funções de apoio).
3. O resultado é guardado em uma variável global (principalmente o array `circuitos[]`) ou refletido diretamente na tela via `innerHTML`.
4. A cada mudança relevante, `salvarProjeto()` serializa o estado inteiro (via `lerProjeto()`) e grava no `localStorage` (autosave contínuo).
5. Ao carregar a página, `carregarProjeto()` lê o `localStorage` e `aplicarProjeto()` repõe os valores nos campos e recria as listas (circuitos, cômodos marcados, anexos etc.).
6. Ao gerar o quadro, os dados de `circuitos[]` + tipo de entrada são usados para montar o desenho SVG e o relatório — nada disso é persistido separadamente; é recalculado a cada clique em "Gerar Quadro".

### Principais variáveis de estado global

| Variável | Papel |
|---|---|
| `circuitos` | Lista de todos os circuitos do levantamento atual |
| `proximoId` | Contador incremental de ID de circuito |
| `quadroDesatualizado` | Sinaliza que o quadro já gerado ficou desatualizado após uma mudança |
| `anexosDocumentos` | Anexos de documentos técnicos, por índice |
| `AUTH_KEY` / `PROJETO_KEY` / `BIBLIOTECA_KEY` | Chaves usadas no `localStorage` |

## Fluxo das telas

A navegação é sequencial, dentro da mesma página:

```
Login
  └─ Menu (sidebar) ──> Biblioteca de Projetos (opcional, a qualquer momento)
  └─ Identificação do Empreendimento
  └─ Selecione o perfil da instalação (Residencial / Condominial / Comercial)
  └─ Configuração da instalação (tipo de entrada)
  └─ Documentação e situação do projeto elétrico
  └─ Cômodos e pontos atendidos  ──┐
  └─ Circuito personalizado (avançado) ──┴──> alimentam circuitos[]
  └─ Levantamento Técnico (lista de circuitos adicionados)
  └─ Casa inteligente e automação residencial
  └─ Quadro de Distribuição / Relatório ──> Gerar PDF (window.print())
```

A troca de perfil (Residencial/Condominial/Comercial) troca a biblioteca de ambientes usada tanto no checklist de cômodos quanto no formulário avançado, sem recarregar a página.

## Principais módulos

Não há módulos em arquivos separados — o agrupamento abaixo é **funcional**, todo dentro de `index.html`:

| Módulo | Responsabilidade |
|---|---|
| **Persistência do projeto** | Serializar/restaurar o projeto atual (`lerProjeto`, `aplicarProjeto`, `salvarProjeto`, `carregarProjeto`, `novoProjeto`, `exportarProjeto`, `importarProjeto`) |
| **Biblioteca de projetos** | Gerenciar múltiplos projetos nomeados (`carregarBiblioteca`, `persistirBiblioteca`, `renderBiblioteca`, `salvarProjetoNaBiblioteca`, `abrirProjetoDaBiblioteca`, `excluirProjetoDaBiblioteca`) |
| **Geocodificação/CEP** | Endereço e coordenadas automáticos via ViaCEP e Nominatim (`processarCep`, `buscarCoordenadas`, `fetchComTimeout`) |
| **Autenticação** | Portão de acesso e wrapper seguro de `localStorage` (`checkAuth`, `entrarNoApp`, `safeStorageGet/Set/Remove`) |
| **Motor de cálculo NBR 5410** | Todo o dimensionamento elétrico normativo (`ampacidadeBase` até `dimensionar`, `escolherIDR`, `escolherQuadro`, `materiaisCircuito`) |
| **Geração de relatório/quadro** | Consolida o cálculo em relatório e desenho SVG (`gerarSVGQuadro`, `gerarBOMConsolidada`, `detectarSubQuadros`, `subQuadroHTML`) |

## Principais funções

| Função | Papel |
|---|---|
| `dimensionar()` | Função central do motor de cálculo — retorna seção do condutor, disjuntor, corrente de projeto (Ib), Iz corrigida, seção de proteção (PE), % de queda de tensão e conformidade |
| `gerarSVGQuadro()` | Monta o SVG completo do quadro elétrico a partir da lista de circuitos e do tipo de entrada |
| `renderComodos()` | Renderiza o checklist de cômodos/pontos, com cálculo ao vivo por linha |
| `renderListaCircuitos()` | Renderiza a lista de circuitos já adicionados ao levantamento |
| `renderBiblioteca()` | Renderiza os cards de projetos salvos na Biblioteca |
| `fichaTecnicaHTML()` | Monta a ficha técnica exibida no relatório/PDF |
| `gerarBOMConsolidada()` | Consolida a lista de materiais (proteção, disjuntores, condutores, quadro) |

Para o inventário completo — todas as 13 telas, as 24 funções de renderização ("componentes"), os 6 agrupamentos de serviço, as variáveis globais e as regras conhecidas de impressão/PDF — ver a auditoria técnica já realizada (`Auditoria_Calculadora_Eletrica_NBR5410.html`, na raiz do projeto NextObra).

## Regras de impressão conhecidas (histórico de bugs reais)

O fluxo de "Gerar PDF" (`window.print()` + `@media print`) já quebrou três vezes no histórico real do projeto. As correções viraram regras permanentes do CSS:

1. Nenhum gradiente/filtro SVG via `url()` — soma silenciosamente no export de PDF do Chrome.
2. Nenhum `border-radius` em elemento que possa quebrar entre páginas — gerava página em branco fantasma.
3. `width`/`height` explícitos no `<svg>` raiz do quadro — evita estouro/corte do desenho.

Qualquer mudança futura no desenho do quadro ou no CSS de impressão precisa respeitar essas três regras.
