# Plano: notificações seletivas por chat

## Objetivo

Permitir que cada conta do ZapFast gere notificações de mensagens somente para
chats escolhidos pela pessoa usuária. Os demais chats continuam recebendo e
arquivando mensagens normalmente, mas não geram aviso na área de trabalho nem
som de notificação.

## Comportamento proposto

- Manter o controle global atual de notificações como opção principal.
- Oferecer dois modos por conta: **Todos os chats** e **Somente chats
  selecionados**.
- No segundo modo, permitir selecionar conversas individuais e grupos. Não
  impor um limite artificial de três seleções.
- Uma lista vazia no modo seletivo significa nenhuma notificação de mensagem.
- Manter o modo **Todos os chats** como padrão para configurações existentes e
  novas instalações.
- Aplicar a seleção antes de qualquer caminho que exiba uma notificação, inclusive
  o aviso genérico usado quando o conteúdo está oculto ou o app está bloqueado.
- Aplicar a seleção também a menções e respostas em grupos. Essas mensagens
  podem continuar furando o limite de repetição atual, mas somente quando o grupo
  estiver selecionado.
- Preservar os filtros existentes: conta com notificações desligadas, tela
  desligada, chat aberto, mensagem antiga, chat arquivado, silenciado ou
  bloqueado continuam sem notificar.
- Não alterar histórico, contagem de mensagens não lidas ou badges. Chamadas
  recebidas permanecem fora deste filtro.

## Desenho técnico

1. **Persistir a preferência por conta.** Em `src/settings.rs`, adicionar ao
   `AccountSettings` o modo de notificação e a lista de IDs de chat selecionados.
   Usar os IDs locais/canônicos já usados pelo modelo de chat. A configuração
   não deve ser sincronizada com o telefone nem armazenada no arquivo do
   histórico.
2. **Preservar compatibilidade.** Usar os defaults de desserialização já
   existentes para que arquivos antigos, sem os novos campos, continuem no modo
   **Todos os chats**. Não migrar nem reescrever arquivos de sessão ou do
   arquivo de mensagens.
3. **Adicionar a seleção na interface.** Em `src/ui/settings.rs`, incluir um
   seletor de modo na seção Notificações e, no modo seletivo, uma lista
   pesquisável de pessoas e grupos com seleção múltipla. A lista deve mostrar
   claramente quantos chats estão selecionados. Respeitar o fluxo existente de
   `Action` e aplicação de mudanças em `src/app.rs`.
4. **Filtrar no ponto de decisão.** Em `App::maybe_notify` (`src/app.rs`),
   verificar se o chat está permitido antes de aplicar limite de repetição,
   resolver conteúdo, som ou solicitar atenção da janela. Usar a mesma decisão
   para notificações com conteúdo e notificações genéricas. Não alterar o fluxo
   separado de chamadas.
5. **Manter o filtro estrito.** No modo seletivo, uma menção ou resposta não
   deve incluir automaticamente um grupo não selecionado. As preferências de
   som por chat e de limite de notificações continuam independentes da lista.

Não deve ser necessário alterar o protocolo, o backend, o esquema do arquivo de
mensagens ou adicionar dependências: a regra é local e usa mensagens já
entregues ao app.

## Testes de regressão

- Desserializar uma configuração antiga sem os novos campos e confirmar o modo
  **Todos os chats**.
- Confirmar que o modo **Todos os chats** mantém o comportamento atual.
- Confirmar que, no modo seletivo, somente IDs selecionados de conversas
  individuais e grupos passam pelo filtro.
- Confirmar que menções e respostas não furam a seleção, embora continuem
  respeitando a exceção existente ao limite de repetição nos grupos permitidos.
- Confirmar que um chat não selecionado não gera notificação genérica quando o
  conteúdo está oculto ou o app está bloqueado.
- Confirmar isolamento entre contas, persistência após salvar/reabrir e
  comportamento para seleção vazia.
- Confirmar que os filtros existentes para chat aberto, arquivado, silenciado,
  bloqueado e tela desligada continuam funcionando, e que chamadas não são
  afetadas.

## Documentação e validação

- Atualizar a documentação do site que descreve as notificações e a nova
  preferência.
- Adicionar as strings da interface pelo caminho de tradução existente; deixar
  `assets/i18n/` para a regeneração do mantenedor.
- Atualizar o demo ou seus testes de layout se necessário. Para a mudança visual,
  registrar capturas antes/depois com conteúdo sintético nos temas claro e escuro
  na descrição do pull request, sem adicionar as capturas ao repositório.
- Executar os testes focados e os checks completos definidos em `AGENTS.md`:
  formatação, Clippy, testes e documentação Rust, incluindo a configuração com
  todas as features.
- Verificar compilação nos alvos Linux, macOS e Windows, relatando com precisão
  quais plataformas foram compiladas ou executadas.

## Critérios de conclusão

- A pessoa pode selecionar conversas individuais e grupos para receber avisos.
- Nenhuma mensagem de chat não selecionado produz popup, som ou pedido de atenção
  da janela, inclusive com o app bloqueado ou conteúdo oculto.
- Mensagens não selecionadas continuam no histórico e na contagem de não lidas.
- Configurações antigas mantêm o comportamento atual e as seleções são isoladas
  por conta.
- Chamadas, privacidade, silêncio e filtros de elegibilidade permanecem intactos.
