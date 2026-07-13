# Roadmap

> Este documento contém **apenas direções futuras já discutidas de fato** para o projeto — nenhum item aqui foi inventado. Nada listado abaixo está implementado ou tem data de entrega definida; a inclusão aqui não é um compromisso, é um registro de possibilidades já levantadas.

## Em estudo (diagnóstico já concluído, decisão pendente)

### Renderizador Inteligente de Quadros Elétricos
Um módulo componentizado para desenhar o quadro elétrico de forma reutilizável entre tela, conferência, relatório e PDF — com uma interface de dados padronizada (`ElectricalPanel`) cobrindo identificação, tipo, fabricante, modelo, módulos DIN, barramentos, tensão, fases, corrente geral, posição de DR/DPS, lista de disjuntores, espaços de reserva, etiquetas, localização e observações.

- Já existe um **diagnóstico completo de arquitetura** mapeando o que precisaria mudar.
- Já existe um **estudo técnico de migração para Vite + TypeScript** (arquivos afetados, dependências, riscos, tempo estimado, plano de migração sem interromper o desenvolvimento).
- **Nenhuma decisão de seguir foi tomada.** A migração exigiria abandonar a característica atual de "arquivo único, zero build, funciona offline sem instalar nada".

### Testes automatizados versionados
Hoje não existe nenhuma suíte de testes no repositório. Foi apontado como risco em mais de uma auditoria, especialmente para o fluxo de impressão/PDF (histórico de bugs reais) e para a checklist de cômodos.

### Pipeline de build/CI-CD
O deploy hoje é manual, via linha de comando (`netlify-cli`). Automatizar isso dependeria de decisões ainda não tomadas sobre build tooling (ver item do Renderizador acima).

## Explicitamente fora do escopo atual

### Módulo Industrial
Foi cogitado em algum momento, mas **excluído deliberadamente** do escopo do projeto por decisão já tomada. Não há biblioteca de circuitos, nem cálculo, nem tela para esse perfil. Só volta ao roadmap mediante nova decisão explícita.

---

Novos itens só devem ser adicionados a este roadmap quando forem, de fato, discutidos como direção futura real do projeto — nunca como suposição do que "poderia ser interessante".
