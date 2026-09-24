# AUDITORIA DE TAGUEAMENTO — GOOGLE ANALYTICS (iOS, Android e Web/React)

## 1. Objetivo
Analisar o código do pacote/repositório indicado no ESCOPO abaixo e
mapear todas as chamadas de tagueamento do Google Analytics (Firebase
Analytics / gtag), diretas ou via wrappers customizados, verificando
se os eventos e parâmetros estão implementados corretamente. O prompt
deve funcionar em projetos iOS (Swift), Android (Kotlin/Java) e
Web/React (JavaScript/TypeScript).

## 2. ESCOPO DA ANÁLISE  ← EDITAR AQUI
Tipo de escopo: TELA/PÁGINA (localizada via evento screen_view ou page_view)
- Valor do atributo de tela a localizar: [ex: "home", "checkout_review", "ProductDetails"]
  (esse é o valor enviado no parâmetro de tela do evento screen_view
  no iOS/Android — ex: screen_name, screen_class, firebase_screen —
  ou no evento page_view na Web — ex: page_title, page_path, page_location)
- Plataforma(s) a considerar: [iOS | Android | Web (React) | TODAS]

Passos que devem ser seguidos, nesta ordem:
1. Localizar, no(s) repositório(s)/pacote(s) indicado(s) para a(s)
   plataforma(s) selecionada(s), o(s) evento(s) screen_view/page_view
   cujo atributo de tela seja igual ao valor informado acima.
2. A partir do ponto onde esse evento é disparado, identificar o
   arquivo e o componente correspondente a essa tela/página (ex:
   ViewController/View/Coordinator no iOS; Activity/Fragment/Compose
   Screen no Android; Page/Component/hook no React).
3. Definir como escopo de análise esse arquivo e os arquivos
   diretamente relacionados a ele (ex: ViewModel, Presenter,
   Coordinator, hooks, subcomponentes renderizados nessa tela) onde
   outros eventos possam ser disparados dentro do contexto dessa
   mesma tela/página.
4. Se o valor do atributo de tela informado não for encontrado em
   nenhuma plataforma/escopo indicado, reportar isso explicitamente
   e não prosseguir com suposições sobre qual arquivo seria.

## 3. Wrappers customizados de Analytics
Além de chamadas diretas ao SDK nativo de cada plataforma:
- iOS: Analytics.logEvent(_:parameters:) (Firebase)
- Android: firebaseAnalytics.logEvent(name, bundle) ou
  logEvent(name) { param(...) } (Firebase KTX)
- Web/React: analytics.logEvent(...) (Firebase JS SDK) ou
  gtag('event', ...)

considere como tagueamento válido também as chamadas feitas através
das seguintes classes/funções/hooks facilitadores (se existirem no
projeto):
- Nome da(s) classe(s)/função(ões)/hook(s): [ex: AnalyticsManager, AppTracker, EventLogger, useAnalytics]
- Método(s) usado(s) para disparo: [ex: track(event:), send(_:), logEvent(name:params:), trackEvent()]
- Caso não saiba os nomes, primeiro faça uma varredura no escopo
  definido acima e identifique automaticamente possíveis
  classes/funções "wrapper" (padrões: nome contendo
  Analytics/Tracker/Logger/Event/Tracking, ou funções que chamem
  Analytics.logEvent/gtag internamente) antes de prosseguir com a
  análise principal.

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
  valor/origem (hardcoded, variável de instância, singleton, contexto
  global/store, etc.), e se ele é adicionado incondicionalmente ou
  sob alguma condição.
- Alertar caso o mesmo parâmetro extra seja adicionado de forma
  inconsistente entre diferentes wrappers ou métodos (ex: um método
  adiciona region e outro não), inclusive entre plataformas
  diferentes (ex: iOS injeta region e Android/Web não).

## 4. Eventos específicos a procurar (opcional)
Se preenchido, restrinja (ou destaque) a busca aos eventos abaixo;
se vazio, liste TODOS os eventos encontrados no escopo (a tela/página
identificada na seção 2).
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
  booleano), mesmo que ele venha de uma constante, enum, sealed
  class/object (Kotlin), enum/const (TypeScript/JavaScript), static
  let, variável local ou propriedade. Nunca reporte a referência de
  código não resolvida (ex: NÃO escrever
  "TagsAuthorizationDetails.tapBackAction"; em vez disso, localizar a
  definição dessa constante/case e reportar
  "content_type = \"tap_back_action\"" — o valor final, não o caminho
  do código).
- Se o valor vier de um enum/sealed class, resolva o raw value
  (String/Int) do case usado nessa chamada específica.
- Se o valor for dinâmico (calculado em runtime, vindo de API, input
  do usuário, estado do componente, etc.) e não houver literal
  possível de resolver, reporte no formato
  nome_do_parametro = "<dinâmico>" e explique a origem (de onde vem o
  valor) na coluna de Observações — nunca deixe o caminho do código
  bruto na coluna de Parâmetros.
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

## 7. Instrução final
Salve tudo em um arquivo nomeado "analise_eventos".
