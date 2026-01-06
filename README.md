# Exemplos do OpenAI ChatKit

Este repositório reúne demonstrações práticas do ChatKit organizadas por cenários de uso. Cada exemplo combina um backend FastAPI com um frontend Vite + React, implementando um backend personalizado usando o SDK Python do ChatKit e integrando-o com o ChatKit.js no lado do cliente.

## 📚 O que é o ChatKit?

O ChatKit é uma plataforma da OpenAI que permite criar interfaces de chat inteligentes com agentes de IA. Ele oferece componentes prontos para uso tanto no frontend (JavaScript) quanto no backend (Python), facilitando a criação de aplicações conversacionais avançadas.

## 🎯 Exemplos Disponíveis

Você pode executar os seguintes exemplos (do mais simples ao mais complexo):

- [**Cat Lounge**](examples/cat-lounge) - **[Iniciante]** Cuidador virtual de um gato que ajuda a melhorar as estatísticas de energia, felicidade e limpeza. Ótimo para aprender os conceitos básicos!
- [**Customer Support**](examples/customer-support) - **[Intermediário]** Concierge de companhia aérea com dados de itinerário em tempo real, sincronização de linha do tempo e ferramentas específicas do domínio.
- [**News Guide**](examples/news-guide) - **[Intermediário]** Assistente da redação Foxhollow Dispatch com busca de artigos, menções @ e respostas conscientes de página.
- [**Metro Map**](examples/metro-map) - **[Avançado]** Planejador de metrô orientado por chat com uma rede React Flow de linhas e estações.

## 🚀 Início Rápido

### Pré-requisitos
Antes de começar, certifique-se de ter instalado:
- Node.js (versão 20 ou superior)
- Python (versão 3.11 ou superior)
- [uv](https://docs.astral.sh/uv/getting-started/installation/) (gerenciador de pacotes Python recomendado)

### Configuração Inicial

**Passo 1:** Configure sua chave da API OpenAI
```bash
export OPENAI_API_KEY="sua-chave-aqui"
```
💡 **Dica:** Você pode obter sua chave em [platform.openai.com/api-keys](https://platform.openai.com/api-keys)

**Passo 2:** Certifique-se que o `uv` está instalado
```bash
# No macOS/Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Ou via pip
pip install uv
```

**Passo 3:** Execute um exemplo

Você pode iniciar qualquer exemplo de duas formas:

#### Opção A: A partir da raiz do repositório (mais fácil)
```bash
npm run cat-lounge        # Inicia o Cat Lounge
npm run customer-support  # Inicia o Customer Support
npm run news-guide        # Inicia o News Guide
npm run metro-map         # Inicia o Metro Map
```

#### Opção B: A partir do diretório do projeto
```bash
cd examples/cat-lounge
npm install
npm run start
```

### Tabela de Referência Rápida

| Exemplo          | Comando (raiz do repo)     | Comando (diretório do projeto)                             | URL de Acesso         |
| ---------------- | -------------------------- | ---------------------------------------------------------- | --------------------- |
| Cat Lounge       | `npm run cat-lounge`       | `cd examples/cat-lounge && npm install && npm run start`   | http://localhost:5170 |
| Customer Support | `npm run customer-support` | `cd examples/customer-support && npm install && npm start` | http://localhost:5171 |
| News Guide       | `npm run news-guide`       | `cd examples/news-guide && npm install && npm run start`   | http://localhost:5172 |
| Metro Map        | `npm run metro-map`        | `cd examples/metro-map && npm install && npm run start`    | http://localhost:5173 |

## 📖 Índice de Funcionalidades

Esta seção explica os diferentes recursos do ChatKit demonstrados nos exemplos. Cada recurso está vinculado ao código-fonte correspondente para facilitar o aprendizado.

### 🔧 Chamadas de ferramentas do servidor para recuperar dados da aplicação

**O que é:** Ferramentas do servidor são funções que o agente de IA pode chamar para obter informações da sua aplicação antes de responder ao usuário.

**Exemplos práticos:**

- **Cat Lounge**:
  - A ferramenta `get_cat_status` ([cat_agent.py](examples/cat-lounge/backend/app/cat_agent.py)) busca as estatísticas mais recentes do gato para o agente.
  - 💡 **Caso de uso:** Quando o usuário pergunta "Como está meu gato?", o agente primeiro chama esta ferramenta para obter os dados atuais antes de responder.

- **News Guide**:
  - O agente utiliza um conjunto de ferramentas de recuperação—`list_available_tags_and_keywords`, `get_article_by_id`, `search_articles_by_tags/keywords/exact_text`, e `get_current_page`—antes de responder, e usa `show_article_list_widget` para apresentar resultados ([news_agent.py](examples/news-guide/backend/app/agents/news_agent.py)).
  - Contextos ocultos como a página de destaque são normalizados na entrada do agente para que resumos e recomendações permaneçam fundamentados ([news_agent.py](examples/news-guide/backend/app/agents/news_agent.py)).
  - 💡 **Caso de uso:** Quando o usuário pede "Mostre artigos sobre tecnologia", o agente busca os artigos usando as ferramentas de busca.

- **Metro Map**:
  - O agente do metrô sincroniza dados do mapa com `get_map` e exibe detalhes de linhas e estações via `list_lines`, `list_stations`, `get_line_route`, e `get_station` antes de dar direções ([metro_map_agent.py](examples/metro-map/backend/app/agents/metro_map_agent.py)).
  - `show_line_selector` apresenta ao usuário uma pergunta de múltipla escolha usando um widget.
  - As respostas de planejamento de rotas anexam fontes de entidades para as estações no caminho sugerido como anotações.
  - 💡 **Caso de uso:** Quando o usuário pede "Me mostre o caminho da estação A para B", o agente busca as informações do mapa antes de calcular a rota.

- **Customer Support**:
  - O concierge adiciona um snapshot `<CUSTOMER_PROFILE>` (itinerário, fidelidade, linha do tempo recente) antes de cada execução e expõe ferramentas para mudar assentos, cancelar viagens, adicionar bagagens, definir refeições, mostrar opções de voo e solicitar assistência contra o estado `AirlineStateManager` por thread ([server.py](examples/customer-support/backend/app/server.py), [support_agent.py](examples/customer-support/backend/app/support_agent.py), [airline_state.py](examples/customer-support/backend/app/airline_state.py)).
  - 💡 **Caso de uso:** Quando o usuário pede "Mude meu assento para 14A", o agente acessa o estado atual da reserva, verifica disponibilidade e executa a mudança.

### 💻 Chamadas de ferramentas do cliente que modificam ou buscam estado da UI

**O que é:** Ferramentas do cliente permitem que o agente de IA interaja com o estado da interface do usuário diretamente no navegador.

**Exemplo prático:**

- **Metro Map**:
  - A ferramenta do cliente `get_selected_stations` obtém os nós atualmente selecionados no canvas para que o agente possa usar o estado do lado do cliente em sua resposta ([ChatKitPanel.tsx](examples/metro-map/frontend/src/components/ChatKitPanel.tsx), [metro_map_agent.py](examples/metro-map/backend/app/agents/metro_map_agent.py)).
  - 💡 **Caso de uso:** O usuário seleciona várias estações no mapa visual e pergunta "Qual a distância entre estas estações?". O agente usa a ferramenta do cliente para saber quais estações foram selecionadas.

### 🎬 Efeitos do cliente (fire-and-forget)

**O que é:** Efeitos do cliente são comandos enviados do servidor para o cliente para atualizar a interface, sem esperar resposta. São ideais para sincronizar o estado visual com as ações do servidor.

**Exemplos práticos:**

- **Cat Lounge**:
  - Os efeitos do cliente `update_cat_status` e `cat_say` são invocados por ferramentas do servidor para sincronizar o estado da UI e exibir balões de fala; tratados via `onEffect` em [ChatKitPanel.tsx](examples/cat-lounge/frontend/src/components/ChatKitPanel.tsx).
  - 💡 **Caso de uso:** Quando você alimenta o gato, o servidor envia um efeito para atualizar a barra de energia na tela e mostrar o gato dizendo "Miau! Obrigado!".

- **Metro Map**:
  - O efeito do cliente `location_select_mode` é transmitido dentro do manipulador de ação do servidor ([server.py](examples/metro-map/backend/app/server.py)) após uma linha ser escolhida e atualiza o canvas do mapa do metrô ([ChatKitPanel.tsx](examples/metro-map/frontend/src/components/ChatKitPanel.tsx)).
  - O efeito do cliente `add_station` é transmitido pelo agente após atualizações do mapa para sincronizar imediatamente o canvas e focar a parada recém-criada ([metro_map_agent.py](examples/metro-map/backend/app/agents/metro_map_agent.py), [ChatKitPanel.tsx](examples/metro-map/frontend/src/components/ChatKitPanel.tsx)).
  - 💡 **Caso de uso:** Quando uma nova estação é adicionada ao mapa, o efeito faz a câmera do canvas se mover automaticamente para mostrar a nova estação.

- **Customer Support**:
  - O servidor transmite efeitos `customer_profile/update` após ferramentas ou ações de widget para que o painel lateral espelhe os dados mais recentes de itinerário, fidelidade e linha do tempo ([support_agent.py](examples/customer-support/backend/app/support_agent.py), [server.py](examples/customer-support/backend/app/server.py)).
  - 💡 **Caso de uso:** Após mudar de assento, o painel lateral é atualizado automaticamente para mostrar o novo assento sem precisar recarregar a página.

### 📄 Respostas conscientes de página

**O que é:** O agente pode saber qual página o usuário está visualizando e dar respostas contextualizadas sobre "esta página".

**Exemplo prático:**

- **News Guide**:
  - O cliente ChatKit encaminha o id do artigo atualmente aberto em um cabeçalho `article-id` para que o backend possa limitar respostas a "esta página" ([ChatKitPanel.tsx](examples/news-guide/frontend/src/components/ChatKitPanel.tsx)).
  - O servidor lê esse contexto de requisição e expõe `get_current_page` para que o agente possa carregar o conteúdo completo sem pedir ao usuário para colá-lo ([main.py](examples/news-guide/backend/app/main.py), [news_agent.py](examples/news-guide/backend/app/agents/news_agent.py)).
  - 💡 **Caso de uso:** O usuário está lendo um artigo e pergunta "Resuma esta página". O agente sabe automaticamente qual artigo resumir.

### 📊 Atualizações de progresso

**O que é:** Mensagens que mostram ao usuário que o sistema está trabalhando em uma tarefa (como "Buscando...", "Carregando...").

**Exemplos práticos:**

- **News Guide**:
  - As ferramentas de recuperação transmitem mensagens `ProgressUpdateEvent` enquanto buscam tags, autores, palavras-chave, texto exato ou carregam a página atual para que a UI mostre estados "Buscando..."/"Carregando..." ([news_agent.py](examples/news-guide/backend/app/agents/news_agent.py)).
  - O localizador de eventos emite progresso enquanto varre datas, dias da semana ou palavras-chave para manter os usuários informados durante buscas mais longas ([event_finder_agent.py](examples/news-guide/backend/app/agents/event_finder_agent.py)).
  - 💡 **Caso de uso:** Ao buscar em milhares de artigos, o usuário vê "Buscando artigos..." em vez de uma tela em branco.

- **Metro Map**:
  - O agente do metrô emite uma atualização de progresso ao recuperar informações do mapa em `get_map`; também emite uma atualização de progresso ao aguardar a conclusão de uma chamada de ferramenta do cliente em `get_selected_stations` ([metro_map_agent.py](examples/metro-map/backend/app/agents/metro_map_agent.py)).
  - 💡 **Caso de uso:** Ao calcular uma rota complexa, o usuário vê "Analisando mapa do metrô...".

### 🔄 Estados do ciclo de vida da resposta na UI

**O que é:** Controle do comportamento da interface durante diferentes fases da resposta do agente (início, processamento, fim).

**Exemplo prático:**

- **Metro Map**:
  - O cliente bloqueia a interação do mapa no início da resposta e desbloqueia quando o stream termina para que o estado do canvas não derive durante atualizações do agente, adicionando manipuladores `onResponseStart` e `onResponseEnd` ([ChatKitPanel.tsx](examples/metro-map/frontend/src/components/ChatKitPanel.tsx)).
  - 💡 **Caso de uso:** Enquanto o agente está adicionando estações ao mapa, o usuário não pode mover o mapa, evitando conflitos.

### 🎨 Widgets sem ações

**O que é:** Componentes visuais que exibem informação mas não requerem interação do usuário.

**Exemplo prático:**

- **Cat Lounge**:
  - A ferramenta do servidor `show_cat_profile` transmite um widget de apresentação definido em [profile_card_widget.py](examples/cat-lounge/backend/app/profile_card_widget.py).
  - 💡 **Caso de uso:** Quando você pergunta "Mostre o perfil do gato", um cartão visual bonito aparece com as informações, mas sem botões para clicar.

### 🎯 Widgets com ações

**O que é:** Componentes visuais interativos que permitem ao usuário tomar decisões clicando em botões.

**Exemplos práticos:**

- **Cat Lounge**:
  - A ferramenta do servidor `suggest_cat_names` transmite um widget com configurações de ação que especificam ações tratadas pelo cliente: `cats.select_name` e `cats.more_names`.
  - Quando o usuário clica no widget, essas ações são tratadas com o callback `handleWidgetAction` em [ChatKitPanel.tsx](examples/cat-lounge/frontend/src/components/ChatKitPanel.tsx).
  - 💡 **Caso de uso:** O agente sugere 3 nomes para o gato em cartões clicáveis. Você clica em "Whiskers" e o nome é selecionado.

- **Customer Support**:
  - Widgets de voo e refeição transmitem com payloads de ação (`flight.select`, `support.set_meal_preference`) para capturar escolhas antes da reserva, construídos a partir de templates `.widget` ([flight_options.py](examples/customer-support/backend/app/flight_options.py), [meal_preferences.py](examples/customer-support/backend/app/meal_preferences.py), [support_agent.py](examples/customer-support/backend/app/support_agent.py)).
  - 💡 **Caso de uso:** O agente mostra 3 opções de voo em cartões. Você clica no voo das 15h e ele é reservado.

- **News Guide**:
  - Widgets de lista de artigos renderizam botões "Ver" que disparam ações `open_article` para navegação e engajamento do cliente ([news_agent.py](examples/news-guide/backend/app/agents/news_agent.py), [article_list_widget.py](examples/news-guide/backend/app/widgets/article_list_widget.py)).
  - O localizador de eventos transmite um widget de linha do tempo com botões `view_event_details` configurados para tratamento do servidor para que os usuários possam expandir itens inline ([event_finder_agent.py](examples/news-guide/backend/app/agents/event_finder_agent.py), [event_list_widget.py](examples/news-guide/backend/app/widgets/event_list_widget.py)).

- **Metro Map**:
  - A ferramenta do servidor `show_line_selector` transmite um widget com a ação `line.select` configurada para disparar no clique do item da lista ([metro_map_agent.py](examples/metro-map/backend/app/agents/metro_map_agent.py), [line_select_widget.py](examples/metro-map/backend/app/widgets/line_select_widget.py)).

### ⚙️ Ações de widget tratadas no servidor

**O que é:** Quando o usuário clica em um botão no widget, a ação é enviada de volta ao servidor para processamento.

**Exemplos práticos:**

- **Cat Lounge**:
  - A ação `cats.select_name` também é tratada no lado do servidor para refletir atualizações nos dados e transmitir de volta uma versão atualizada do widget de sugestões de nomes em [server.py](examples/cat-lounge/backend/app/server.py).
  - É invocada usando `chatkit.sendAction()` do callback `handleWidgetAction` em [ChatKitPanel.tsx](examples/cat-lounge/frontend/src/components/ChatKitPanel.tsx).
  - 💡 **Caso de uso:** Você escolhe um nome, o servidor salva no banco de dados e atualiza o widget para mostrar "Nome selecionado!".

- **Customer Support**:
  - Manipuladores de ação persistem reservas, refeições, upgrades e remarcações; bloqueiam widgets; registram contexto oculto; e atualizam o perfil quando os usuários clicam em `flight.select`, `support.set_meal_preference`, `booking.*`, `upsell.*`, ou `rebook.select_option` ([server.py](examples/customer-support/backend/app/server.py)).
  - 💡 **Caso de uso:** Ao selecionar um upgrade de assento, o servidor valida disponibilidade, processa pagamento e atualiza sua reserva.

- **News Guide**:
  - A ação `view_event_details` é processada no lado do servidor para atualizar o widget da linha do tempo com descrições expandidas sem uma viagem de ida e volta ao modelo ([server.py](examples/news-guide/backend/app/server.py)).

- **Metro Map**:
  - A ação `line.select` é tratada no lado do servidor para transmitir um widget atualizado, adicionar um item de contexto oculto `<LINE_SELECTED>` à thread, transmitir uma mensagem do assistente perguntando ao usuário se deseja adicionar a estação no início ou fim da linha, e acionar o efeito do cliente `location_select_mode` para a UI sincronizar ([server.py](examples/metro-map/backend/app/server.py)).

### 📎 Anexos (Attachments)

**O que é:** Capacidade de enviar arquivos (como imagens) na conversa.

**Exemplo prático:**

- **Customer Support**:
  - Anexos de imagem de ponta a ponta: o backend emite URLs de upload/download, aplica limites de imagem/tamanho e converte uploads para URLs de dados para o modelo ([attachment_store.py](examples/customer-support/backend/app/attachment_store.py), [main.py](examples/customer-support/backend/app/main.py), [thread_item_converter.py](examples/customer-support/backend/app/thread_item_converter.py)). O painel React registra `attachments.create`, faz upload via URL assinada e insere o anexo no compositor quando viajantes compartilham fotos de inspiração ([CustomerContextPanel.tsx](examples/customer-support/frontend/src/components/CustomerContextPanel.tsx)).
  - 💡 **Caso de uso:** Você envia uma foto do destino dos seus sonhos e o agente sugere pacotes de viagem baseados na imagem.

### 🏷️ Anotações (Annotations)

**O que é:** Marcadores visuais que destacam entidades específicas mencionadas na resposta do agente.

**Exemplo prático:**

- **Metro Map**:
  - A ferramenta `plan_route` renderiza cada estação em uma rota planejada como uma fonte de entidade na mensagem do assistente. A estação é renderizada como anotações inline na mensagem do assistente e também na lista de fontes.
  - O manipulador de clique de entidade do cliente move o canvas React Flow para a estação clicada ([ChatKitPanel.tsx](examples/metro-map/frontend/src/components/ChatKitPanel.tsx), [metro_map_agent.py](examples/metro-map/backend/app/agents/metro_map_agent.py)).
  - 💡 **Caso de uso:** O agente diz "Pegue a linha azul de Titan Border até Lyra Verge". Você pode clicar em "Titan Border" na mensagem e o mapa automaticamente foca nessa estação.

### 📝 Títulos de threads

**O que é:** Nomes automáticos para as conversas, facilitando encontrá-las depois.

**Exemplos práticos:**

- **Cat Lounge**:
  - Após o usuário nomear o gato, a ferramenta `set_cat_name` fixa o nome e atualiza o título da thread para `{nome}'s Lounge` antes de salvá-la ([cat_agent.py](examples/cat-lounge/backend/app/cat_agent.py)).
  - 💡 **Caso de uso:** Você nomeia seu gato "Whiskers" e a conversa automaticamente se chama "Whiskers's Lounge".

- **Customer Support**:
  - Um agente de título leve nomeia a conversa na primeira mensagem do usuário sem atrasar a primeira resposta ([title_agent.py](examples/customer-support/backend/app/title_agent.py), [server.py](examples/customer-support/backend/app/server.py)).
  - 💡 **Caso de uso:** Você escreve "Preciso remarcar meu voo" e a thread automaticamente se chama "Remarcação de voo".

- **News Guide**:
  - O `title_agent` executa na primeira mensagem do usuário para gerar um título curto amigável para redação quando nenhum existe ([server.py](examples/news-guide/backend/app/server.py), [title_agent.py](examples/news-guide/backend/app/agents/title_agent.py)).

- **Metro Map**:
  - O servidor do metrô usa um `title_agent` dedicado para definir um título breve de planejamento de metrô na primeira rodada e o persiste nos metadados da thread ([server.py](examples/metro-map/backend/app/server.py), [title_agent.py](examples/metro-map/backend/app/agents/title_agent.py)).

### @ Tags de entidade (menções @)

**O que é:** Capacidade de mencionar entidades específicas digitando @ (como @EstacaoX ou @ArtigoY) para referenciá-las diretamente.

**Exemplos práticos:**

- **News Guide**:
  - Busca e visualizações de entidades alimentam @-menções para artigos/autores no compositor e renderizam visualizações de hover via `/articles/tags` ([ChatKitPanel.tsx](examples/news-guide/frontend/src/components/ChatKitPanel.tsx), [main.py](examples/news-guide/backend/app/main.py)).
  - Entidades marcadas são convertidas em marcadores legíveis pelo modelo para que o agente possa buscar os registros certos (`<ARTICLE_REFERENCE>` / `<AUTHOR_REFERENCE>`) ([thread_item_converter.py](examples/news-guide/backend/app/thread_item_converter.py)).
  - Tags de referência de artigos são resolvidas em artigos completos via a ferramenta instruída `get_article_by_id` antes do agente citar detalhes ([news_agent.py](examples/news-guide/backend/app/agents/news_agent.py)).
  - 💡 **Caso de uso:** Você digita "@" e vê uma lista de artigos. Seleciona "@Tecnologia na Educação" e pergunta "Resuma esse artigo". O agente sabe exatamente qual artigo buscar.

- **Metro Map**:
  - A busca de entidades do compositor lista estações para que os usuários possam @-mencioná-las; clicar em uma tag também foca a estação no canvas ([ChatKitPanel.tsx](examples/metro-map/frontend/src/components/ChatKitPanel.tsx)).
  - Estações marcadas são convertidas em blocos `<STATION_TAG>` com metadados completos de linha para que o agente possa responder sem outra busca ([thread_item_converter.py](examples/metro-map/backend/app/thread_item_converter.py), [server.py](examples/metro-map/backend/app/server.py)).
  - 💡 **Caso de uso:** Digite "@Cinderia" e pergunte "Quais linhas passam por aqui?". O agente já sabe sobre qual estação você está falando.

### 🛠️ Escolha de ferramenta (menu do compositor)

**O que é:** Botões especiais no compositor que permitem forçar o uso de um agente ou ferramenta específica.

**Exemplo prático:**

- **News Guide**:
  - O cliente ChatKit é configurado com uma opção `composer.tools` que especifica opções no menu do compositor ([ChatKitPanel.tsx](examples/news-guide/frontend/src/components/ChatKitPanel.tsx))
  - Botões de ferramenta do compositor permitem que usuários forcem agentes específicos (`event_finder`, `puzzle`), configurando `tool_choice` na requisição ([config.ts](examples/news-guide/frontend/src/lib/config.ts)).
  - O backend roteia essas escolhas de ferramenta para agentes especializados antes de voltar ao agente News Guide ([server.py](examples/news-guide/backend/app/server.py)).
  - 💡 **Caso de uso:** Você clica no botão "Localizador de Eventos" antes de perguntar "O que acontece neste fim de semana?" para garantir que o agente especializado em eventos responda.

### 🎨 Ações personalizadas no cabeçalho

**O que é:** Botões ou controles personalizados na barra superior do chat.

**Exemplo prático:**

- **Metro Map**:
  - O cabeçalho do chat usa um toggle de ícone do lado direito (`dark-mode` / `light-mode`) para alternar o esquema de cores da aplicação no lado do cliente ([ChatKitPanel.tsx](examples/metro-map/frontend/src/components/ChatKitPanel.tsx)).
  - 💡 **Caso de uso:** Você clica no ícone de lua no cabeçalho e a interface muda para modo escuro instantaneamente.

### 🎨 Geração de imagens

**O que é:** Capacidade do agente de criar imagens com IA durante a conversa.

**Exemplo prático:**

- **Cat Lounge**:
  - O `cat_agent` inclui uma `ImageGenerationTool` configurada com 3 imagens parciais.
  - O método `respond` do servidor cat lounge passa `ResponseStreamConverter(partial_images=3)` ao invocar `stream_agent_response` para que o helper calcule corretamente o progresso da geração de imagem ao transmitir cada imagem parcial.
  - 💡 **Caso de uso:** Você pergunta "Mostre como meu gato está" e o agente gera uma imagem fofa do gato com base no seu estado atual (feliz, com fome, etc.).
