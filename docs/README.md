# NextObra Engineering Platform

**Módulo atual: Levantamento Técnico e Dimensionamento Elétrico conforme NBR 5410.**

Ferramenta técnica interna da **NextObra Construções**, usada em campo pela equipe técnica para dimensionamento elétrico conforme a **ABNT NBR 5410:2004** (Tabelas 33 a 48). É o primeiro módulo da NextObra Engineering Platform — o nome da plataforma comporta, em visão de longo prazo, futuros módulos de engenharia da empresa, mas nenhum outro módulo além deste foi definido ou iniciado até o momento (ver `PROJECT_VISION.md`).

> Este documento descreve o projeto **exatamente como ele existe hoje**. Nenhuma funcionalidade, tecnologia ou processo listado aqui foi planejado ou idealizado — tudo reflete o código real em `index.html`. Para propósito, missão, visão e princípios, ver `PROJECT_VISION.md`.

## Objetivo da plataforma

Apoiar o levantamento técnico e o dimensionamento elétrico de instalações **residenciais, condominiais e comerciais**, permitindo que a equipe técnica, em campo ou em escritório:

- monte o levantamento de circuitos de uma instalação (por cômodo/ponto, com biblioteca própria por perfil);
- receba o dimensionamento NBR 5410 em tempo real (seção do condutor, disjuntor, condutor de proteção/PE, corrente de projeto e queda de tensão);
- gere um quadro de distribuição ilustrativo (SVG) e um relatório técnico completo, exportável em PDF;
- salve, exporte e reabra projetos, tanto localmente quanto em uma biblioteca de múltiplos projetos.

O escopo cobre atualmente **três perfis de instalação**: Residencial, Condominial e Comercial. Um módulo Industrial foi cogitado, mas está **deliberadamente fora do escopo atual** (ver `ROADMAP.md`).

## Tecnologias atuais

| Camada | Tecnologia | Observação |
|---|---|---|
| Linguagem | HTML5 + CSS3 + JavaScript (ES6+) | Sem TypeScript |
| Framework de UI | Nenhum | Sem React/Vue/Angular — DOM manipulado diretamente |
| Bundler/build | Nenhum | Não há `npm`, `package.json` nem etapa de build |
| Roteamento | Nenhum | Página única; seções são mostradas/escondidas via JavaScript |
| Persistência | `localStorage` do navegador | Sem banco de dados, sem backend |
| Autenticação | Credencial fixa comparada em JS no cliente | Portão visual, não é autenticação real |
| Geração de PDF | `window.print()` nativo do navegador + CSS `@media print` | Sem biblioteca de PDF |
| APIs externas | ViaCEP (endereço por CEP), Nominatim/OpenStreetMap (geocodificação) | Ambas públicas e gratuitas |
| Hospedagem | Netlify (site `nextobra-calculadora-eletrica`) | Deploy manual via `netlify-cli`, sem CI/CD |
| Controle de versão | Git + GitHub (`jheivesson/quadro-distribuicao-nextobra`, público) | |
| Testes automatizados | Nenhum no repositório | Não existe suíte de testes versionada |

## Estrutura existente

```
Calculadora_Eletrica_NBR5410/
├── .git/
├── .netlify/                 (config gerada pela CLI do Netlify)
├── CLAUDE.md                  (regras de orientação para sessões de desenvolvimento)
├── docs/                     (esta pasta de documentação)
│   ├── README.md
│   ├── PROJECT_VISION.md
│   ├── PROJECT_OVERVIEW.md
│   ├── ARCHITECTURE.md
│   ├── ROADMAP.md
│   ├── CHANGELOG.md
│   ├── DEVELOPMENT_RULES.md
│   └── TECHNICAL_RENDERING_ENGINE.md
└── index.html                 (ÚNICO arquivo de código-fonte — ~3.200 linhas)
```

> Nota de nomenclatura: o diretório do repositório e o site publicado ainda usam o nome técnico anterior (`Calculadora_Eletrica_NBR5410` / `nextobra-calculadora-eletrica.netlify.app`) — renomear pasta, repositório ou site é uma mudança de infraestrutura/deploy, fora do escopo desta atualização de documentação. O nome comercial da plataforma, daqui em diante, é **NextObra Engineering Platform**.

Todo o código-fonte da aplicação vive em um único arquivo: `index.html`, contendo:
- bloco `<style>` — CSS da aplicação (design tokens, layout, impressão);
- corpo HTML — todas as telas/seções da aplicação;
- bloco `<script>` — toda a lógica (cálculo NBR 5410, renderização, persistência, autenticação).

Para um mapeamento função-a-função e linha-a-linha, ver `ARCHITECTURE.md`.

## Como executar o projeto

Não é necessário instalar nada. O arquivo funciona de forma autocontida:

1. Abra `index.html` diretamente em qualquer navegador moderno (Chrome, Edge, Firefox) — por duplo clique ou `file://`.
2. Faça login com a credencial da equipe técnica.
3. A ferramenta funciona **totalmente offline**, exceto pelo autopreenchimento de endereço/coordenadas a partir do CEP (que depende de internet).

Para acessar a versão publicada online, não é preciso nenhuma instalação: **https://nextobra-calculadora-eletrica.netlify.app**

### Publicar uma atualização (deploy)

O deploy é manual, feito via linha de comando com a CLI do Netlify:

```
netlify-cli deploy --prod --dir=. --site=<id-do-site>
```

Não há build a rodar antes — o `index.html` é publicado exatamente como está.

## Como contribuir

Como todo o código está em um único arquivo, o fluxo de contribuição hoje é:

1. Edite `index.html` diretamente.
2. **Teste manualmente no navegador** o fluxo afetado antes de considerar a mudança concluída — não há suíte de testes automatizada para se apoiar.
3. Preserve as regras conhecidas de impressão/PDF (ver `ARCHITECTURE.md` e `DEVELOPMENT_RULES.md`) — o fluxo de impressão já teve bugs reais e conhecidos.
4. Faça commit com mensagem descritiva em português, no padrão já usado no histórico do projeto (ver `CHANGELOG.md`).
5. Publique no GitHub (`git push`) e, se a mudança precisar estar visível para a equipe, faça o deploy manual no Netlify.
6. Consulte `DEVELOPMENT_RULES.md` antes de propor qualquer mudança de arquitetura (ex.: introduzir build tools, frameworks, ou dependências) — mudanças desse tipo exigem decisão explícita antes de qualquer implementação.
