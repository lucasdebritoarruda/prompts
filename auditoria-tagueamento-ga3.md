# AUDITORIA DE TAGUEAMENTO — GOOGLE ANALYTICS (iOS / Swift)

## 1. Objetivo
Analisar o código Swift indicado no ESCOPO abaixo e mapear todas as
chamadas de tagueamento do Google Analytics (Firebase Analytics),
diretas ou via wrappers customizados, verificando se os eventos e
parâmetros estão implementados corretamente.

## 2. ESCOPO DA ANÁLISE  ← EDITAR AQUI
Tipo de escopo: [ARQUIVO | PASTA | PROJETO_INTEIRO]
- Se ARQUIVO: nome(s) do(s) arquivo(s): [ex: LoginViewController.swift, HomeViewModel.swift]
- Se PASTA: caminho da pasta: [ex: Sources/Features/Checkout]
- Se PROJETO_INTEIRO: analisar todos os arquivos .swift do repositório

## 3. Wrappers customizados de Analytics
Além de chamadas diretas (ex: Analytics.logEvent, FirebaseAnalytics),
considere como tagueamento válido as chamadas feitas através das
seguintes classes/protocolos facilitadores (se existirem no projeto):
- Nome da(s) classe(s)/interface(s): [ex: AnalyticsManager, AppTracker, EventLogger]
- Método(s) usado(s) para disparo: [ex: track(event:), send(_:), logEvent(name:params:)]
- Caso não saiba os nomes, primeiro faça uma varredura no escopo
  definido acima e identifique automaticamente possíveis classes
  "wrapper" (padrões: nome contendo Analytics/Tracker/Logger/Event,
  ou classes que chamem Analytics.logEvent internamente) antes de
  prosseguir com a análise principal.

### 3.1 Parâmetros injetados internamente pelo wrapper
Para cada wrapper identificado, verifique também se, DENTRO do corpo
da função (e não apenas nos parâmetros recebidos na chamada do
call site), há inserção adicional de parâmetros antes do disparo real
ao SDK — ex: enriquecimento automático do evento com dados como
region, user_id, session_id, app_version, environment, etc.
- Nome(s) do(s) parâmetro(s) a verificar: [ex: region]
- Se vazio, reporte TODOS os parâmetros extras encontrados dentro do
  corpo da função que não vieram diretamente dos argumentos passados
  pelo chamador.
- Para cada parâmetro extra encontrado, informar: nome do parâmetro,
  valor/origem (hardcoded, variável de instância, singleton, etc.),
  e se ele é adicionado incondicionalmente ou sob alguma condição.
- Alertar caso o mesmo parâmetro extra seja adicionado de forma
  inconsistente entre diferentes wrappers ou métodos (ex: um método
  adiciona region e outro não).

## 4. Eventos específicos a procurar (opcional)
Se preenchido, restrinja (ou destaque) a busca aos eventos abaixo;
se vazio, liste TODOS os eventos encontrados no escopo.
- Nome(s) do(s) evento(s): [ex: purchase_completed, screen_view, login_success]
- Parâmetro(s) esperado(s) por evento (opcional): [ex: purchase_completed → item_id, value, currency]

## 5. O que verificar para cada chamada encontrada
- Arquivo e linha onde a chamada ocorre
- Se é chamada direta ao SDK ou via wrapper (citar qual)
- Nome do evento disparado
- Parâmetros enviados e seus tipos/valores
- Se o evento é disparado condicionalmente (ex: dentro de if/guard) —
  nesse caso, descrever a condição
- Inconsistências: nomes de evento com typo, parâmetros faltando,
  valores hardcoded suspeitos, eventos duplicados/disparados 2x
- Quando a chamada for via wrapper: parâmetros extras injetados
  internamente na função (ver seção 3.1), com origem de cada um

### 5.1 Formato obrigatório dos valores de parâmetro
Todo parâmetro reportado na tabela DEVE seguir o padrão:
  nome_do_parametro = "valor_literal"
- Sempre resolva o valor até chegar a um literal (string, número ou
  booleano), mesmo que ele venha de uma constante, enum, static let,
  variável local ou propriedade. Nunca reporte a referência de código
  não resolvida (ex: NÃO escrever "TagsAuthorizationDetails.tapBackAction";
  em vez disso, localizar a definição dessa constante/case e reportar
  "content_type = \"tap_back_action\"" — o valor final, não o caminho
  do código).
- Se o valor vier de um enum, resolva o raw value (String/Int) do case
  usado nessa chamada específica.
- Se o valor for dinâmico (calculado em runtime, vindo de API, input
  do usuário, etc.) e não houver literal possível de resolver, reporte
  no formato nome_do_parametro = "<dinâmico>" e explique a origem
  (de onde vem o valor) na coluna de Observações — nunca deixe o
  caminho do código bruto na coluna de Parâmetros.
- Se não for possível localizar a definição do valor dentro do escopo
  analisado, reporte nome_do_parametro = "<não resolvido: caminho.do.codigo>"
  e sinalize isso como pendência na coluna de Observações.

## 6. Formato de saída
Tabela com colunas: Arquivo | Linha | Evento | Origem (direta/wrapper) |
Parâmetros (recebidos + injetados internamente) | Observações/Inconsistências

Ao final, um resumo com: total de eventos encontrados, eventos da
lista da seção 4 que NÃO foram encontrados (se aplicável), lista
de wrappers identificados automaticamente (se aplicável), e lista
de parâmetros extras injetados por wrapper (seção 3.1), destacando
inconsistências entre wrappers/métodos.
