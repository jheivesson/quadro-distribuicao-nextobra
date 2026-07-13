# Visão Geral do Projeto

## O que existe hoje

O projeto é uma ferramenta de **levantamento técnico e dimensionamento elétrico NBR 5410**, funcionando como uma página única (`index.html`) com as seguintes telas/seções, todas mostradas ou escondidas na mesma página (não há rotas):

1. **Login** — credencial fixa de acesso da equipe.
2. **Menu (sidebar retrátil)** — acesso a Biblioteca, Novo Projeto, Exportar/Importar Projeto e Sair, além do indicador de norma e de autosave.
3. **Biblioteca de Projetos** — salvar, nomear, reabrir e excluir múltiplos projetos guardados no navegador.
4. **Identificação do Empreendimento** — dados do cliente/obra (nome, CPF, RG, órgão emissor, endereço, cidade/UF, CEP com autopreenchimento de endereço e coordenadas).
5. **Selecione o perfil da instalação** — Residencial, Condominial ou Comercial.
6. **Configuração da instalação** — tipo de entrada de serviço (mono/bi/trifásica), responsável técnico, data, revisão.
7. **Documentação e situação do projeto elétrico** — o que já existe de documentação técnica, com upload de anexos por tipo de documento.
8. **Cômodos e pontos atendidos pela demanda** — checklist por cômodo/ponto (biblioteca própria por perfil, até 62 itens), com dimensionamento NBR 5410 calculado ao vivo por linha.
9. **Circuito personalizado (avançado)** — entrada manual para casos fora da biblioteca padrão.
10. **Levantamento Técnico de Dimensionamento Elétrico** — lista consolidada dos circuitos já adicionados.
11. **Casa inteligente e automação residencial** — requisitos de automação que alimentam a biblioteca de circuitos e o relatório.
12. **Quadro de Distribuição / Relatório** — desenho SVG do quadro elétrico, veredito de conformidade, tabelas técnicas e de materiais (BOM), exportação em PDF.
13. **Serviços NextObra + Rodapé** — vitrine institucional estática.

O fluxo cobre **três perfis de instalação de forma completa**: Residencial, Condominial e Comercial — todos com bibliotecas próprias de circuitos e cálculo NBR 5410 validado.

## Limitações atuais

- **Sem backend e sem banco de dados.** Toda a persistência é local, via `localStorage` do navegador — não há sincronização entre dispositivos, nem backup automático fora da máquina do usuário.
- **Autenticação apenas simbólica.** O login é uma credencial fixa comparada no próprio JavaScript do cliente — não há sessão de servidor, hash de senha ou controle de acesso real.
- **Sem testes automatizados no repositório.** Toda validação de mudanças depende de teste manual no navegador.
- **Sem build tooling.** O projeto é um único arquivo HTML/CSS/JS — não há TypeScript, bundler ou framework de componentes.
- **Geração de PDF frágil por natureza.** Depende do `window.print()` do navegador; historicamente já apresentou bugs reais (conteúdo sumindo, orientação errada, páginas em branco) hoje corrigidos e documentados, mas o mecanismo em si continua sensível a mudanças de CSS/markup.
- **Deploy manual, sem CI/CD.** Publicar uma atualização exige rodar o deploy manualmente via linha de comando.
- **Módulo Industrial fora do escopo.** Só existem bibliotecas de circuito para Residencial, Comercial e Condominial.
- **Upload de arquivos limitado a 3 MB por anexo**, e restrito ao armazenamento local do navegador (sem nuvem/servidor).
- **Alto acoplamento de código.** Cálculo, estado e renderização de tela convivem no mesmo arquivo e, em grande parte, nas mesmas funções — não há separação formal entre domínio, estado e apresentação.

## Visão de evolução

Duas direções de evolução já foram formalmente estudadas (diagnóstico de arquitetura e estudo técnico de migração), mas **nenhuma decisão de implementação foi tomada** até o momento:

- **Renderizador Inteligente de Quadros Elétricos** — um módulo componentizado (React + TypeScript) para desenhar o quadro elétrico de forma reutilizável entre tela, conferência, relatório e PDF, com uma interface de dados padronizada (`ElectricalPanel`). Isso exigiria introduzir um passo de build (ex.: Vite), o que muda a característica atual de "arquivo único, funciona offline, sem instalar nada".
- **Testes automatizados e CI/CD** — apontados como lacunas nas auditorias já realizadas, sem compromisso de implementação definido.

Itens detalhados de evolução (sempre como possibilidades futuras, não compromissos) estão em `ROADMAP.md`.
