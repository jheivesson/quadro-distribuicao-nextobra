# Changelog

Formato inspirado em [Keep a Changelog](https://keepachangelog.com/pt-BR/1.1.0/). O projeto ainda não usa versionamento semântico (não há arquivo de versão) — as entradas abaixo foram reconstruídas a partir do histórico real de commits do Git, agrupadas por data, da mais recente para a mais antiga.

A partir de hoje, toda nova mudança deve ser registrada em **[Não lançado]** e movida para uma nova data quando publicada.

## [Não lançado]

- Criação da pasta `docs/` com a documentação inicial do projeto (este conjunto de arquivos).

## 2026-07-13

### Adicionado
- Bibliotecas de circuitos reconstruídas: cada cômodo passou a gerar circuitos independentes por exigência da NBR 5410 (ex.: chuveiro, iluminação e tomada de um banheiro viram 3 circuitos, não 1).
- Controle de projeto: "Novo Projeto", "Exportar Projeto" e "Importar Projeto".
- Biblioteca de Projetos: salvar, nomear, abrir e excluir múltiplos projetos guardados no navegador.
- Campo de Estado (UF) em dropdown (27 estados + DF); CEP passou a preencher endereço e coordenadas automaticamente (ViaCEP + Nominatim).
- Indicador visível de autosave e seção "Arquivos salvos" na Biblioteca.
- Alerta bloqueante antes de imprimir, pedindo para conferir a orientação Paisagem.
- Barra flutuante fixa com "Salvar Projeto" / "Abrir Projeto", sempre visível na tela.
- Campos de CPF, RG e órgão emissor do cliente.
- "Documentos disponíveis" passou a ser um dropdown com checkboxes (antes era uma lista fixa, ocupando muito espaço vertical).
- Menu lateral (sidebar) reunindo norma/status/ações de projeto, depois transformado em menu retrátil (comandos ficam ocultos até clicar em "☰ Menu").

### Corrigido
- PDF perdia conteúdo do quadro elétrico em determinadas situações (causa raiz: gradientes/filtros SVG não renderizados no export do Chrome).
- Orientação de impressão saindo em retrato em vez de paisagem.
- Páginas em branco na impressão, causadas por `border-radius` fragmentando conteúdo entre páginas.
- Posição do card "Selecione o perfil da instalação", que precisou ser reordenado duas vezes até ficar corretamente acima de "Tipo de entrada de serviço".

### Alterado
- Quadro elétrico ganhou aparência visual mais profissional (SVG desenhado com sombras simuladas, cores planas, numeração de disjuntores).

## 2026-07-12

### Adicionado
- Primeira versão funcional do Quadro de Distribuição NextObra, com dimensionamento conforme NBR 5410.
- Fase 1 do formulário: identificação do empreendimento, situação do projeto, perfil da instalação e automação residencial (incluindo integração conceitual com Alexa).
- Bibliotecas completas de circuitos para os perfis Comercial e Condominial — os 3 perfis (Residencial, Comercial, Condominial) tornaram-se totalmente funcionais.
- Checklist de cômodos com dimensionamento de queda de tensão calculado ao vivo, conforme NBR 5410.
- Sub-quadros dedicados de CFTV e de comando inteligente (automação).

### Alterado
- Quadro passou a se adaptar ao tipo de entrada de serviço (monofásica/bifásica/trifásica); perfil Industrial removido/sem aviso de disponibilidade.
- Renomeação do painel de circuitos e do nome do aplicativo para "Levantamento Técnico de Dimensionamento Elétrico" — o termo "Quadro de Distribuição" ficou reservado apenas para a seção que representa o quadro elétrico físico (diagrama e botão de gerar/atualizar).
- Removidos os campos de UC/concessionária (fora do escopo elétrico do levantamento); senha de acesso trocada; adicionado anexo de documentos e opção "levantamento as built".
