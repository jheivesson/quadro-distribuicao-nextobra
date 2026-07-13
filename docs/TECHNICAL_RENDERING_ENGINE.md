# Renderizador Técnico do Quadro Elétrico — Planejamento de Evolução

> Documento de planejamento (Fase 1 — Diagnóstico + Fase 2 — Plano, conforme `CLAUDE.md`). **Nenhum código foi alterado para produzir este documento.** Todas as referências de linha e comportamento foram verificadas diretamente em `index.html` no momento da escrita.

---

## 1. Estado atual do renderizador

O quadro elétrico é desenhado por **um único gerador SVG** (`gerarSVGQuadro()`, `index.html` L2759–2847), chamado a partir de dois pontos:

- `btn-gerar-quadro` (L3104), para o quadro geral;
- `subQuadroHTML()` (L3083), reaproveitando a mesma função para os sub-quadros de CFTV e Casa Inteligente (L3074–3081, `detectarSubQuadros()`).

Isso já atende ao requisito central deste planejamento: **existe um único motor de renderização**, reaproveitado em tela, sub-relatórios e impressão — não há um segundo caminho de desenho a ser criado do zero, apenas evoluído.

O SVG é **gerado dinamicamente por concatenação de string** (não é imagem, não é captura de tela) e inserido via `innerHTML`. Isso é compatível com o requisito "mesma representação na tela e na impressão", porque o nó SVG exibido na tela é literalmente o mesmo que vai para o PDF via `window.print()`.

O layout de hoje é **puramente algorítmico** (calculado a partir da quantidade e do tipo dos circuitos), não são coordenadas fixas — o que já satisfaz a diretriz de posicionamento calculado, e não hardcoded.

O resultado visual atual é uma **vista interna simplificada**: barras de fase, barramento de neutro, disjuntor geral + DPS, grupos de DR com seus disjuntores protegidos, disjuntores sem DR agrupados por linha, barramento de PE. Não existe hoje uma vista frontal externa (gabinete fechado) — ver seção 5.

## 2. Funções existentes e responsabilidade de cada uma

| Função | Linha | Responsabilidade única |
|---|---|---|
| `svgToggle(cx, topY, corBase, corClaro)` | L2661 | Desenha a alavanca liga/desliga de um disjuntor/DR/DPS, em dois tons |
| `svgSombraFalsa(x, y, w, h, rx)` | L2674 | Retângulo escuro deslocado, simulando sombra — **nunca via `<filter>`/gradiente**, por causa do bug de impressão já documentado no próprio código (comentário L2670–2673) |
| `svgBreaker(x, y, w, h, label1, label2, conforme, titulo)` | L2678 | Desenha um disjuntor termomagnético (geral ou de circuito); borda vermelha se `conforme === false`; `<title>` acessível com a descrição do circuito |
| `svgRCD(x, y, w, h, label1, label2)` | L2694 | Desenha um dispositivo DR (diferencial residual) |
| `svgDPS(x, y, w, h, label1, label2)` | L2706 | Desenha o DPS (proteção contra surtos) |
| `svgRail(x, y, w)` | L2717 | Desenha o trilho DIN (uma barra cinza por linha de dispositivos) |
| `svgFaseBars(x, y, w, fases)` | L2732 | Desenha as barras de fase (1, 2 ou 3, conforme o tipo de entrada), com rótulo F/F1/F2/F3 |
| `svgTerminalBar(x, y, w, color, label)` | L2748 | Desenha uma barra de terminais genérica — reaproveitada tanto para o barramento de **Neutro** (azul) quanto para o de **PE/Terra** (verde) |
| `gerarSVGQuadro(circuitos, tipoEntrada)` | L2759 | **Orquestrador** — decide quantas linhas existem, agrupa disjuntores por DR (`agruparPorDR`, L2650), calcula o disjuntor geral, monta o SVG completo chamando as funções acima, define `width`/`height` explícitos no `<svg>` raiz |
| `subQuadroHTML(titulo, lista, tipoEntrada, idSufixo)` | L3083 | Reaproveita `gerarSVGQuadro()` para desenhar um sub-quadro (CFTV/Automação) e monta o bloco de relatório ao redor dele |

Cada função desenha apenas a própria peça — já seguem o princípio de responsabilidade única (regra 19 de `CLAUDE.md`), só que como funções de string, não como componentes de arquivo.

**Observação técnica (não é bug, é um ponto de atenção para a evolução):** tanto `subQuadroHTML()` quanto o handler de `btn-gerar-quadro` chamam `gerarSVGQuadro(lista, tipoEntrada, idSufixo)`/`gerarSVGQuadro(circuitos, tipoEntrada, 'geral')` passando um 3º argumento — mas a assinatura real da função é `gerarSVGQuadro(circuitos, tipoEntrada)`, só 2 parâmetros. O 3º argumento é silenciosamente ignorado. Hoje isso não causa problema (a função não gera nenhum `id=""` que precise ser único), mas se uma evolução futura precisar de IDs únicos por instância de quadro (por exemplo, para `<title>`/acessibilidade, ou para permitir múltiplos quadros na mesma página com navegação/zoom independentes), esse parâmetro já existe nas chamadas e só precisa passar a ser usado dentro da função.

## 3. Dados atualmente disponíveis para montar o quadro

A partir de `circuitos[]` (cada item com `id`, `ambiente`, `ambienteId`, `categoria`, `descricao`, `exigeDR`, `requerNeutroNoPonto`, `emergencia`, `params`, `resultado`) e de `tipoEntrada` (mono/bi/trifásica, painel "Configuração da instalação"), já é possível derivar, sem nenhum dado novo:

- quantidade de fases (via `FASES_POR_ENTRADA`, L2725);
- disjuntor geral e seus polos (via `escolherDisjuntor`/`POLOS_GERAL_POR_ENTRADA`, L2730);
- presença e agrupamento de DR (via `c.exigeDR` + `agruparPorDR`);
- disjuntor de cada circuito (`resultado.inom`) e seus polos (`params.sistema === 'trifasico'` → 3, senão 1);
- conformidade por circuito (`resultado.conforme`) — já usada para colorir a borda;
- identificação numérica de cada circuito (`C1`, `C2`… — índice na lista, não um código do usuário);
- descrição do circuito (usada hoje só como tooltip `<title>`, não como rótulo visível);
- **quantidade total de módulos DIN** e **tamanho de gabinete recomendado** (`totalModulos` + `escolherQuadro()`/`QUADRO_TAMANHOS = [12, 18, 24, 32, 40, 54, 72]`, L2886–2890) — **já calculado hoje, mas só usado na tabela de materiais (BOM), nunca no desenho SVG**.

## 4. Dados que ainda não existem

Confirmado por leitura direta do projeto (nenhum desses campos existe hoje em `lerProjeto()`/`aplicarProjeto()` nem em nenhuma tela):

- **Identificação/nome do quadro** (ex.: "QGBT", "QDC-1") — os sub-quadros têm um título fixo por tipo ("Quadro de CFTV", "Casa Inteligente"), mas o quadro geral não tem um campo de nome/tag editável.
- **Localização física do quadro** dentro da edificação (ex.: "Térreo, corredor de serviço").
- **Fabricante e modelo do gabinete/disjuntores** — não existe em nenhum lugar do projeto (nem deveria copiar marca comercial, conforme a diretriz do produto).
- **Etiquetas de circuito definidas pelo usuário** — hoje o rótulo visível é só `C{n}` (posição na lista); não há campo para o usuário nomear/codificar um circuito (ex.: "C1 — Chuveiro Suíte").
- **Quantidade de trilhos DIN como conceito explícito.** Hoje, o número de "linhas" desenhadas é um efeito colateral do agrupamento por DR + um limite de 8 módulos por linha (`ROW_CAP`, L2771) — não existe um conceito normativo/físico de "quantos módulos cabem por trilho neste gabinete real".
- **Distribuição de fases entre circuitos monofásicos (balanceamento de carga R/S/T)** — hoje todo circuito monopolar usa a mesma cor/posição visual de fase; não há lógica que distribua circuitos monofásicos entre F1/F2/F3 para balancear a carga trifásica.
- **Espaços de reserva como conceito de dado.** Hoje o SVG desenha exatamente os dispositivos existentes — não há campo "quantidade de reservas desejadas" nem lógica que desenhe módulos vazios.
- **Vista frontal externa (gabinete fechado)** — não existe; hoje só existe a vista interna.

## 5. Diferença entre vista frontal externa e vista interna

| | Vista interna (existe hoje) | Vista frontal externa (não existe) |
|---|---|---|
| O que mostra | Trilhos, barras de fase/neutro/PE, disjuntores, DR, DPS — como se a tampa estivesse aberta | O gabinete fechado, como alguém vê o quadro na parede antes de abri-lo |
| Função no produto | Documentação técnica: o que tem dentro, como está organizado, se está conforme | Identificação de campo: qual quadro é esse, onde fica, como abrir com segurança |
| Elementos típicos | Disjuntores, barramentos, trilhos, numeração de circuito | Porta/tampa, fecho, etiqueta de identificação (nome do quadro, tensão, cuidado/aviso), eventual visor de disjuntor geral |
| Está no escopo pedido? | Sim, é a evolução principal (já existe, será refinada) | Sim, pedida como proposta nova (seção 7) |

Ambas as vistas são **ilustrativas**, não fotorrealistas nem CAD — a diferença é o que cada uma revela, não o nível de realismo.

## 6. Proposta visual para a vista interna

Evolução do que já existe, mantendo a mesma função `gerarSVGQuadro()` como base:

- Manter os componentes atuais (disjuntor, DR, DPS, trilho, barras de fase/neutro/PE) com o estilo visual já validado (cores planas, sombra falsa, cantos arredondados) — não redesenhar do zero.
- Acrescentar um **rótulo de identificação do quadro** no topo do SVG (nome/tag), condicional à existência do dado (ver seção 4 — precisa de decisão do PO sobre criar esse campo).
- Acrescentar **módulos de reserva** desenhados de forma visualmente distinta (ex.: contorno tracejado, sem rótulo, cor neutra) ao final da última linha ou como linha adicional — só quando esse dado existir (ver seção 10).
- Mostrar a **etiqueta de cada circuito** (hoje só em tooltip) de forma mais legível diretamente no desenho, sem quebrar o limite de espaço de cada módulo (52×78 px hoje) — precisa de teste de legibilidade com nomes longos.
- Indicar visualmente a que **gabinete padrão** (12/18/24/32/40/54/72 módulos) aquele desenho corresponde, reaproveitando o mesmo `escolherQuadro()` já usado na BOM — hoje esse número já é calculado, só não chega ao SVG.

## 7. Proposta visual para a vista frontal

Novo componente (não existe hoje), a ser somado ao renderizador, não paralelo a ele:

- Um retângulo externo representando a porta/gabinete fechado, com cantos levemente arredondados (dentro do limite de impressão — nunca `border-radius` real do CSS em elemento que quebra página, apenas `rx`/`ry` do próprio SVG, que não tem esse bug).
- Uma etiqueta frontal central com: nome/tag do quadro (se existir), tipo de entrada (mono/bi/trifásica), disjuntor geral (corrente nominal).
- Um indicador simples da alavanca do disjuntor geral, visível através de um "visor" — reaproveitando `svgToggle()` já existente, em vez de criar um novo desenho de alavanca.
- Fecho/dobradiça representados de forma simples e esquemática (não fotorrealista).
- **Não** deve tentar reproduzir a aparência de um gabinete comercial específico (Schneider/WEG/Steck/Siemens) — apenas sugerir a linguagem visual genérica de um quadro elétrico (retângulo robusto, etiqueta central, indicador de disjuntor).

## 8. Regras de posicionamento dos componentes

Regras já em vigor hoje em `gerarSVGQuadro()` (a preservar, não a reinventar):

1. Barras de fase sempre no topo, quantidade = tipo de entrada (1/2/3).
2. Barramento de Neutro logo abaixo das barras de fase.
3. Disjuntor geral + DPS sempre na primeira linha de dispositivos.
4. Cada grupo de até 4 circuitos com DR forma sua própria linha, com o disjuntor IDR à esquerda do grupo.
5. Circuitos sem DR são agrupados em linhas de até 8 módulos (`ROW_CAP`), na ordem em que aparecem em `circuitos[]`.
6. Barramento de PE sempre por último, após a última linha de dispositivos.
7. Largura do SVG (`W`) e altura (`H`) são calculadas a partir do conteúdo — nunca fixas — e o `<svg>` sempre recebe `width`/`height` explícitos (não só `viewBox`), regra já documentada no próprio código (L2813–2815) para evitar o bug de encolhimento no PDF.

Regra nova proposta (dependente de decisão do PO): se a vista frontal (seção 7) for implementada, ela deve ficar **antes** (acima ou à esquerda, a decidir) da vista interna no mesmo bloco de relatório, nunca substituindo-a.

## 9. Regras para distribuição em trilhos DIN

**Estado atual:** não existe um conceito real de "capacidade por trilho". O limite de 8 módulos por linha (`ROW_CAP = 8`, L2771) é uma escolha de legibilidade do desenho, não uma regra vinda de um gabinete real. Cada "linha" desenhada equivale visualmente a um trilho, mas sem relação com os tamanhos reais de `QUADRO_TAMANHOS` (12/18/24/32/40/54/72).

**Proposta para evolução** (requer decisão do PO — ver seção "dúvidas"):
- Definir uma capacidade de módulos por trilho **por tamanho de gabinete** (ex.: um quadro de 12 módulos normalmente tem 1 trilho de 12; um de 24 pode ter 2 trilhos de 12; um de 40 pode ter trilhos de tamanhos variados conforme o fabricante) — isso não pode ser inventado sem uma referência normativa/comercial real, e hoje o projeto não tem essa tabela.
- Alternativa mais conservadora: manter o `ROW_CAP` como está (puramente visual), mas deixar claro no relatório que a divisão em "linhas" do desenho é ilustrativa e não prescreve fisicamente quantos trilhos o eletricista deve instalar.

## 10. Regras para módulos de reserva

**Estado atual:** não desenhado. O SVG contém exatamente os dispositivos que existem em `circuitos[]}` mais geral/DPS/DR — nada além disso.

**Proposta:**
- Se o total de módulos ocupados for menor que o tamanho do gabinete escolhido (`escolherQuadro(totalModulos) - totalModulos`), desenhar essa diferença como módulos de reserva vazios, com contorno tracejado e sem rótulo (ou rótulo "reserva").
- Isso não exige nenhum dado novo do usuário — pode ser inteiramente derivado do que já existe (`totalModulos` + `QUADRO_TAMANHOS`), o que o torna o item de menor risco de implementação desta lista.

## 11. Regras para identificação dos circuitos

**Estado atual:** cada disjuntor mostra `C{n}`, onde `n` é a posição do circuito em `circuitos[]` (L2776, `numeroPorId`) — não é um código escolhido pelo usuário. A descrição completa (`descricao`) só aparece como tooltip `<title>` e na tabela de detalhamento técnico abaixo do desenho, nunca diretamente no SVG.

**Proposta:**
- Manter `C{n}` como identificador primário (é estável e já usado como referência cruzada com a tabela técnica — mudar isso quebraria essa referência).
- Avaliar mostrar uma versão curta da descrição do circuito diretamente no módulo do disjuntor, truncada com a função já existente `truncarTexto()` (L2656), sujeita a teste de legibilidade dentro da largura de 52 px por módulo.

## 12. Regras para quadros monofásicos, bifásicos e trifásicos

Já implementado e a preservar tal como está:

- **Monofásico:** 1 barra de fase (`F`), disjuntor geral bipolar (2 polos).
- **Bifásico:** 2 barras de fase (`F1`, `F2`), disjuntor geral bipolar (2 polos).
- **Trifásico:** 3 barras de fase (`F1`, `F2`, `F3`), disjuntor geral tetrapolar (4 polos).
- Disjuntores de circuito individuais usam 1 polo, exceto quando `params.sistema === 'trifasico'` para aquele circuito específico (3 polos) — isto é, um quadro trifásico na entrada pode ter circuitos individuais tanto monofásicos quanto trifásicos, e isso já é tratado corretamente hoje.

Nenhuma mudança é proposta aqui — está correto e não deve ser tocado sem justificativa técnica (regra 3/4 do `CLAUDE.md`).

## 13. Compatibilidade com impressão

Três regras já descobertas e corrigidas neste projeto, que qualquer evolução do SVG **precisa continuar respeitando**:

1. **Nunca** usar `<linearGradient>`/`<filter>` referenciado via `url(#id)` — some silenciosamente no export de PDF do Chrome quando o SVG é inserido via `innerHTML` (por isso `svgSombraFalsa()` existe como alternativa em retângulo sólido, L2670–2673).
2. **Nunca** usar `border-radius` (CSS) em elementos que possam quebrar entre páginas na impressão — já causou página em branco fantasma. Cantos arredondados **dentro do próprio SVG** (atributo `rx`/`ry`) não têm esse problema e podem continuar sendo usados livremente (é o que já é feito hoje nos disjuntores, gabinete etc.).
3. **Sempre** definir `width`/`height` explícitos no `<svg>` raiz, além do `viewBox` — evita que o pipeline de impressão calcule altura errada e encolha o conteúdo.

Qualquer novo elemento visual (vista frontal, módulos de reserva, etiquetas maiores) deve ser testado especificamente gerando um PDF real (não só visualizado na tela), como já foi feito no histórico deste projeto.

## 14. Compatibilidade com projetos antigos

- `gerarSVGQuadro()` é uma função **derivada** — não lê nem grava nada em `localStorage`. Ela recebe `circuitos[]` e `tipoEntrada` (já existentes no formato salvo hoje) e devolve um SVG novo a cada chamada.
- Isso significa que qualquer evolução puramente visual (adicionar reserva, rótulo de identificação, vista frontal) **não exige mudança no formato JSON salvo**, desde que os novos dados visuais (nome do quadro, localização, etiquetas de circuito) sejam **opcionais** — ausentes em projetos antigos, o desenho simplesmente omite esses elementos (ex.: sem rótulo de nome se o campo não existir).
- Se, no futuro, for decidido acrescentar campos novos ao projeto (ex.: "nome do quadro"), a regra 17 do `CLAUDE.md` exige estratégia de migração — mas como esses campos seriam **novos e opcionais**, projetos antigos continuam abrindo normalmente com o campo simplesmente vazio/ausente, sem necessidade de transformação de dados existentes.

## 15. Riscos técnicos

1. **Confundir "evoluir o desenho" com "mudar o cálculo".** O SVG só consome `resultado` e `params` já calculados — nenhuma mudança visual deve tocar em `dimensionar()` ou funções de apoio (regras 3/4 do `CLAUDE.md`).
2. **Regressão no bug de impressão já resolvido.** É o risco mais concreto e com histórico real neste projeto — qualquer elemento novo precisa ser validado com geração de PDF real, não só inspeção visual em tela.
3. **Densidade visual.** Adicionar vista frontal + reserva + etiquetas maiores no mesmo relatório pode tornar o desenho muito grande para caber bem numa página impressa — precisa de teste com casos reais de muitos circuitos (quadros de 40+ módulos).
4. **Ambiguidade de trilhos DIN.** Como descrito na seção 9, hoje não existe uma fonte de verdade real para "módulos por trilho" — qualquer regra criada aqui sem uma referência técnica real corre o risco de "inventar dado normativo" (proibido pela regra 5 do `CLAUDE.md`).
5. **O parâmetro `idSufixo` não utilizado** (seção 2) pode gerar confusão se um novo desenvolvedor assumir que IDs já são únicos entre múltiplos quadros na mesma página.

## 16. Plano de implementação em etapas pequenas (proposto — aguardando aprovação)

Cada etapa abaixo é independente e pode ser aprovada/rejeitada separadamente, sem depender das seguintes:

1. **Módulos de reserva** — menor risco, nenhum dado novo necessário (deriva de `totalModulos` + `escolherQuadro()`, já calculados). Só toca em `gerarSVGQuadro()`.
2. **Uso do `idSufixo`** para gerar IDs únicos por instância de quadro (correção de uma inconsistência já existente, sem mudança visual perceptível).
3. **Etiqueta de identificação do quadro no SVG** — depende de decisão do PO sobre criar o campo "nome do quadro" na tela de Configuração da instalação (mudança de UI fora do escopo deste documento, a ser tratada como tarefa própria se aprovada).
4. **Legibilidade da descrição do circuito no disjuntor** (hoje só em tooltip) — mudança visual isolada em `svgBreaker()`.
5. **Vista frontal externa** — maior escopo desta lista; deve vir por último, depois de validar que as mudanças menores acima não quebraram impressão.
6. Validação de impressão real (PDF) após cada etapa, não só ao final.

Nenhuma etapa cria um segundo motor de renderização — todas evoluem `gerarSVGQuadro()` e as funções `svg*` já existentes.

## 17. Critérios de aceitação

- O quadro geral e os sub-quadros (CFTV/Automação) continuam sendo gerados pela mesma função (`gerarSVGQuadro`) — nenhuma duplicação de motor.
- Nenhuma mudança em `dimensionar()` ou nas demais funções do motor de cálculo NBR 5410.
- Nenhuma mudança no formato JSON de projetos salvos sem que os campos novos sejam opcionais e retrocompatíveis.
- PDF gerado via `window.print()` continua sem conteúdo cortado, sem página em branco, sem elemento sumindo — validado com geração real de PDF, não só inspeção em tela.
- Nenhum gradiente/filtro SVG via `url()` introduzido.
- Nenhum `border-radius` (CSS) introduzido em elemento sujeito a quebra de página.
- Nenhuma biblioteca externa instalada; nenhuma migração de arquitetura realizada.
- Todo dado novo exibido no desenho (nome do quadro, reserva etc.) é opcional — a ausência do dado não quebra o desenho, apenas omite o elemento correspondente.

## 18. Testes necessários

Como o projeto não tem suíte de testes automatizada versionada (ver `PROJECT_OVERVIEW.md`), a validação de qualquer etapa deste plano precisa ser feita manualmente (ou via script Playwright avulso), cobrindo pelo menos:

1. Quadro monofásico, bifásico e trifásico, cada um gerado e conferido visualmente.
2. Quadro pequeno (poucos circuitos, sem DR) e quadro grande (muitos circuitos, múltiplos grupos de DR) — testar o limite de módulos por linha.
3. Geração de PDF real (não só tela) para cada um dos casos acima, verificando: nenhuma página em branco, nenhum elemento cortado, orientação Paisagem correta.
4. Sub-quadros de CFTV e Casa Inteligente, confirmando que continuam reaproveitando a mesma função sem duplicação.
5. Reabertura de um projeto salvo **antes** desta evolução (formato antigo), confirmando que o quadro continua sendo desenhado normalmente mesmo sem os campos novos (nome do quadro, etiquetas etc.).
6. Se implementados módulos de reserva: conferir visualmente contra o valor de `escolherQuadro()` já usado na BOM, para os 7 tamanhos de gabinete existentes (12/18/24/32/40/54/72).
