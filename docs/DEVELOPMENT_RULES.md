# Regras de Desenvolvimento

Regras para qualquer implementação futura neste projeto. Todas as regras abaixo vêm de decisões, correções ou padrões já estabelecidos de fato ao longo do desenvolvimento real do projeto — nenhuma foi inventada agora.

## Regras de negócio (NBR 5410 / escopo técnico)

1. **Nunca mesclar circuitos que a NBR 5410 exige serem independentes.** Ex.: chuveiro, iluminação e tomada de um mesmo banheiro são 3 circuitos separados, nunca 1 só.
2. **Não incluir conteúdo de energia solar/fotovoltaica/BESS** neste projeto. O único ponto de contato com motores/inversores é o de inversores de frequência para motores industriais, e mesmo assim fora do escopo atual (módulo Industrial não existe hoje).
3. **Módulo Industrial permanece fora do escopo** até uma decisão explícita em contrário. Não adicionar biblioteca de circuitos, cálculo ou tela para esse perfil sem essa decisão.
4. **cosφ por categoria de equipamento é uma estimativa**, não um valor normativo — deve continuar ajustável pelo usuário e sinalizado como estimativa no formulário/relatório (comentário já existente no código, linha ~1344).

## Regras técnicas (impressão / PDF)

5. **Nunca usar gradiente ou filtro SVG referenciado via `url()`** (`<linearGradient>`, `<filter feDropShadow>`) em qualquer desenho que precise aparecer no PDF — já causou conteúdo sumindo silenciosamente no export do Chrome.
6. **Nunca aplicar `border-radius` em elementos que podem quebrar entre páginas** na impressão — já causou páginas em branco fantasma.
7. **Sempre definir `width`/`height` explícitos no `<svg>` raiz** de qualquer diagrama que vá para impressão — evita estouro/corte do desenho.
8. **O quadro elétrico deve continuar sendo desenhado dinamicamente** (SVG gerado a partir dos dados), nunca via captura de tela ou imagem estática.

## Regras de processo

9. **Testar manualmente no navegador** (ou via Playwright, quando disponível) antes de considerar qualquer mudança concluída — o projeto não tem suíte de testes automatizada versionada para se apoiar.
10. **Nunca commitar segredos/tokens** diretamente em comandos ou arquivos versionados. Usar arquivo temporário fora do controle de versão e remover imediatamente após o uso.
11. **Toda mudança visível para a equipe deve ser publicada** (commit + push no GitHub) e, quando aplicável, seguida de deploy manual no Netlify para manter o site publicado sincronizado com o repositório.
12. **Pedir confirmação explícita do responsável do projeto antes de qualquer ação irreversível ou que crie exposição externa nova** — por exemplo, publicar em repositório público, habilitar hospedagem pública nova, ou qualquer ação equivalente. Uma autorização já dada não vale para uma ação de escopo diferente.
13. **Antes de qualquer migração de arquitetura** (introdução de build tools, TypeScript, framework de UI, banco de dados etc.), fazer primeiro um diagnóstico e/ou estudo técnico, sem implementar nada, até haver decisão explícita de seguir.
14. **Preservar a característica "arquivo único, zero build, funciona offline, sem instalar nada"** enquanto não houver decisão explícita em contrário — é uma característica valorizada do projeto até o momento, não um detalhe incidental.

## Regras de comunicação

15. Toda comunicação com o responsável do projeto — respostas, perguntas, relatórios — deve ser em **português do Brasil**.
16. Documentos gerados sobre o estado do projeto (auditorias, estudos, relatórios) devem se basear **apenas no código real** — quando algo pedido não existir no projeto, isso deve ser declarado explicitamente como "não localizado", nunca suposto ou inventado.
