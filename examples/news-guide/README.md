# News Guide (Guia de Notícias)

## 📰 Sobre o Projeto

Assistente de redação Foxhollow Dispatch demonstrando fluxos ChatKit pesados em recuperação e widgets ricos.

**Nível de dificuldade:** 🟡 Intermediário

**O que você aprenderá:**
- Recuperação avançada de dados e busca
- Respostas conscientes de página/contexto
- Entity tags (@-mentions) com previews
- Composer tool choice (seleção explícita de agentes)
- Progress streaming para feedback ao usuário
- Múltiplos agentes especializados

---

## 🚀 Início Rápido

```bash
# Exporte variáveis de ambiente
export OPENAI_API_KEY="sua-chave-aqui"
export VITE_CHATKIT_API_DOMAIN_KEY="domain_pk_local_dev"

# Da raiz do repositório
npm run news-guide

# OU do diretório do exemplo
cd examples/news-guide
npm install
npm run start
```

✅ Acesse: **http://localhost:5172**

---

## 💬 Prompts de Exemplo

### Busca Básica
- **"O que está em alta agora?"**
  - 🎯 Demonstra: Busca de artigos + widget de lista
  - Mostra artigos populares em cartões clicáveis

### Consciência de Página  
- **"Resuma esta página para mim."**
  - 🎯 Demonstra: Ferramenta `get_current_page`
  - O agente sabe qual artigo você está vendo
  - Precisa ter um artigo aberto!

### Busca por Tags
- **"Mostre-me tudo marcado com parques e eventos ao ar livre."**
  - 🎯 Demonstra: Recuperação de informação com múltiplas tags
  - Usa ferramentas `search_articles_by_tags`

### @-Mentions (Entity Tags)
- **"@Elowen últimas histórias?"**
  - 🎯 Demonstra: Lookup de autor com @-mention
  - **⚠️ Importante:** Digite @ manualmente, não funciona com copiar/colar!

### Ferramentas do Compositor (Composer Tools)
- **"Que eventos estão acontecendo neste sábado?"**
  - 🎯 Primeiro selecione a ferramenta "Event finder" no menu do compositor
  - Depois faça a pergunta
  - Agente especializado em eventos processará

- **"Me dê uma pausa rápida de quebra-cabeça."**
  - 🎯 Selecione a ferramenta "Coffee break puzzle" no menu
  - Receba um desafio mental rápido

---

## 🎓 Funcionalidades Explicadas

Este exemplo é intermediário e demonstra conceitos mais avançados de recuperação de dados e roteamento de agentes.

---

### 1. 🔍 Suite de Ferramentas de Recuperação

Um conjunto completo de ferramentas para buscar informação no banco de dados de artigos.

#### Ferramentas Disponíveis:

```python
# backend/app/agents/news_agent.py

def list_available_tags_and_keywords() -> dict:
    """Lista todas as tags e palavras-chave disponíveis"""
    return {
        "tags": ["technology", "sports", "politics", "culture"],
        "keywords": ["AI", "election", "climate", "innovation"]
    }

def search_articles_by_tags(tags: list[str]) -> list[Article]:
    """Busca artigos que contenham qualquer uma das tags"""
    # Busca no banco de dados
    return database.find_articles_with_tags(tags)

def search_articles_by_keywords(keywords: list[str]) -> list[Article]:
    """Busca artigos que contenham palavras-chave no título ou conteúdo"""
    return database.full_text_search(keywords)

def search_articles_by_exact_text(text: str) -> list[Article]:
    """Busca exata por texto específico"""
    return database.exact_match(text)

def search_articles_by_author(author: str) -> list[Article]:
    """Busca todos os artigos de um autor"""
    return database.find_by_author(author)

def get_article_by_id(article_id: str) -> Article:
    """Obtém artigo completo por ID"""
    return database.get(article_id)
```

**Como o agente usa:**
```
Usuário: "Mostre artigos sobre tecnologia e inovação"

Fluxo do agente:
1. Emite progresso: "Buscando artigos..."
2. Chama list_available_tags_and_keywords() 
3. Identifica tags relevantes: ["technology"]
4. Identifica keywords relevantes: ["innovation"]
5. Chama search_articles_by_tags(["technology"])
6. Chama search_articles_by_keywords(["innovation"])
7. Combina e deduplica resultados
8. Chama show_article_list_widget() para exibir
9. Usuário vê cartões de artigos com botões "Ver"
```

---

### 2. 📄 Respostas Conscientes de Página

O agente sabe qual página você está visualizando.

**Arquitetura:**

```
┌──────────────┐                    ┌──────────────┐
│   Frontend   │                    │   Backend    │
│              │                    │              │
│ Artigo aberto│                    │              │
│ ID: "art-123"│                    │              │
└──────┬───────┘                    └──────┬───────┘
       │                                   │
       │ Request com header:               │
       │ article-id: "art-123"             │
       │──────────────────────────────────>│
       │                                   │
       │                                   │ Salva em contexto:
       │                                   │ current_page = "art-123"
       │                                   │
       │<──────────────────────────────────│
       │                                   │
```

**Implementação:**

```typescript
// frontend - ChatKitPanel.tsx
const chatkitConfig = {
    fetch: async (url, options) => {
        // Adiciona article-id ao header
        return fetch(url, {
            ...options,
            headers: {
                ...options.headers,
                'article-id': currentArticleId  // ← Passa ID do artigo
            }
        })
    }
}
```

```python
# backend - main.py
@app.post("/chatkit")
async def chatkit_endpoint(request: Request):
    article_id = request.headers.get("article-id")
    # Salva no contexto da requisição
    request.state.article_id = article_id
```

```python
# backend - news_agent.py
def get_current_page() -> Article:
    """Obtém o artigo da página atual"""
    article_id = get_request_context().article_id
    if not article_id:
        return "Nenhum artigo aberto no momento"
    return database.get(article_id)
```

**Caso de uso:**
```
[Você está lendo artigo "IA na Educação"]

Você: "Resuma esta página"
Agente: [chama get_current_page()]
Agente: [obtém artigo "IA na Educação"]
Agente: "Este artigo discute como a inteligência artificial 
         está transformando o ensino..."
```

**💡 Vantagem:** Usuário não precisa especificar qual artigo!

---

### 3. 🏷️ Entity Tags (@-Mentions)

Menção de entidades específicas com @ para referência direta.

**Fluxo completo:**

```
1. Usuário digita "@" no compositor
2. Frontend chama GET /articles/tags?query=""
3. Backend retorna lista de artigos e autores
4. Frontend mostra dropdown com previews
5. Usuário seleciona "@Elowen" (uma autora)
6. Frontend adiciona entity tag à mensagem:
   {
     type: "entity",
     id: "author-elowen",
     name: "Elowen",
     category: "author"
   }
7. Backend converte para marcador legível pelo modelo:
   <AUTHOR_REFERENCE id="elowen">Elowen</AUTHOR_REFERENCE>
8. Agente vê a referência e busca artigos
9. Chama search_articles_by_author("elowen")
```

**Conversão de entity tags:**

```python
# backend/app/thread_item_converter.py
def convert_entity_tag(tag):
    if tag.category == "author":
        return f'<AUTHOR_REFERENCE id="{tag.id}">{tag.name}</AUTHOR_REFERENCE>'
    elif tag.category == "article":
        return f'<ARTICLE_REFERENCE id="{tag.id}">{tag.name}</ARTICLE_REFERENCE>'
```

**Por que conversão é necessária:**
- O modelo não entende objetos JSON nativamente
- Marcadores XML são claros e estruturados
- Facilitam instruções como "quando ver ARTICLE_REFERENCE, chame get_article_by_id"

**Preview de entidades:**

```typescript
// Ao fazer hover sobre @tag, mostra preview
<EntityPreview>
  <Avatar src={author.photo} />
  <Name>{author.name}</Name>
  <Bio>{author.bio}</Bio>
  <Stats>42 artigos</Stats>
</EntityPreview>
```

---

### 4. 📊 Progress Streaming

Feedback visual enquanto operações longas acontecem.

**Como implementar:**

```python
# backend/app/agents/news_agent.py
def search_articles_by_tags(tags: list[str]):
    # Emite evento de progresso
    yield ProgressUpdateEvent(
        title="Buscando artigos...",
        status="in_progress"
    )
    
    # Faz a busca (pode demorar)
    results = database.search(tags)
    
    # Emite progresso de conclusão
    yield ProgressUpdateEvent(
        title="Busca concluída",
        status="complete"
    )
    
    # Retorna resultados
    return results
```

**No frontend:**

```typescript
// ChatKitPanel.tsx
onProgress: (progress) => {
    if (progress.status === "in_progress") {
        showLoadingIndicator(progress.title)
    } else if (progress.status === "complete") {
        hideLoadingIndicator()
    }
}
```

**Estados visuais:**

```
Estado 1: "Buscando artigos..." [🔄 spinner]
Estado 2: "Carregando página..." [🔄 spinner]
Estado 3: "Escaneando datas..." [🔄 spinner]
Estado 4: "Busca concluída" [✓ checkmark]
```

**💡 Por que é importante:**
- Usuário sabe que o sistema está trabalhando
- Reduz ansiedade de espera
- Melhora percepção de performance
- Profissional e polido

---

### 5. 🛠️ Tool Choice (Composer Menu)

Seleção explícita de agentes especializados.

**Arquitetura de múltiplos agentes:**

```
┌─────────────────────────────────┐
│     News Guide Agent            │  ← Agente padrão
│     (artigos, busca geral)      │
└─────────────────────────────────┘
         
┌─────────────────────────────────┐
│     Event Finder Agent          │  ← Especialista em eventos
│     (calendário, datas)         │
└─────────────────────────────────┘

┌─────────────────────────────────┐
│     Puzzle Agent                │  ← Especialista em puzzles
│     (quebra-cabeças, jogos)     │
└─────────────────────────────────┘
```

**Configuração do compositor:**

```typescript
// frontend/src/lib/config.ts
const composerTools = [
    {
        id: "event_finder",
        label: "🗓️ Event Finder",
        description: "Buscar eventos por data"
    },
    {
        id: "puzzle",
        label: "🧩 Coffee Break Puzzle",
        description: "Obter um quebra-cabeça"
    }
]
```

```typescript
// frontend - ChatKitPanel.tsx
<ChatKit
    composer={{
        tools: composerTools
    }}
/>
```

**Roteamento no backend:**

```python
# backend/app/server.py
async def handle_message(message, tool_choice):
    if tool_choice == "event_finder":
        return await event_finder_agent.run(message)
    elif tool_choice == "puzzle":
        return await puzzle_agent.run(message)
    else:
        return await news_agent.run(message)  # Padrão
```

**Fluxo de uso:**

```
1. Usuário clica no ícone de ferramentas no compositor
2. Menu aparece com opções: "Event Finder", "Puzzle"
3. Usuário seleciona "Event Finder"
4. Botão fica destacado
5. Usuário digita "Que eventos neste fim de semana?"
6. Requisição inclui tool_choice: "event_finder"
7. Backend roteia para event_finder_agent
8. Agente especializado processa
```

**Agente especializado em eventos:**

```python
# backend/app/agents/event_finder_agent.py
event_finder_agent = Agent(
    instructions="""
    Você é especialista em encontrar eventos na agenda da cidade.
    
    Habilidades especiais:
    - Entende datas naturais ("este sábado", "próxima semana")
    - Filtra por categoria de evento
    - Ordena por relevância e popularidade
    
    Sempre mostre eventos em um widget de timeline.
    """,
    tools=[
        scan_event_calendar,
        parse_natural_date,
        filter_by_category,
        create_timeline_widget
    ]
)
```

**💡 Por que múltiplos agentes:**
- Cada um tem instruções otimizadas para seu domínio
- Ferramentas especializadas
- Respostas mais precisas
- Custo otimizado (pode usar modelos diferentes)

---

### 6. 🎨 Widgets com Ações

Widgets interativos que capturam interações do usuário.

#### Widget de Lista de Artigos (Client-handled action)

```python
# backend/app/widgets/article_list_widget.py
def create_article_list_widget(articles):
    return {
        "type": "article-list",
        "items": [
            {
                "id": "art-123",
                "title": "IA na Educação",
                "author": "Elowen",
                "excerpt": "Como a IA está...",
                "action": {
                    "type": "open_article",  # ← Ação tratada no cliente
                    "article_id": "art-123"
                }
            }
            # Mais artigos...
        ]
    }
```

```typescript
// frontend - ChatKitPanel.tsx
handleWidgetAction: async (action) => {
    if (action.type === "open_article") {
        // Navega para o artigo
        navigateToArticle(action.article_id)
        // Atualiza contexto de página
        setCurrentArticleId(action.article_id)
    }
}
```

**Fluxo:**
```
1. Widget mostra 5 artigos
2. Cada um tem botão "Ver"
3. Usuário clica "Ver" no segundo artigo
4. Ação open_article é capturada no cliente
5. Página navega para o artigo
6. article-id é atualizado no contexto
7. Próximas perguntas são conscientes desta página
```

#### Widget de Timeline de Eventos (Server-handled action)

```python
# backend/app/widgets/event_list_widget.py
def create_event_timeline_widget(events):
    return {
        "type": "event-timeline",
        "events": [
            {
                "id": "evt-1",
                "title": "Festival de Jazz",
                "date": "Sábado, 15:00",
                "collapsed": True,  # Começa colapsado
                "action": {
                    "type": "view_event_details",  # ← Ação tratada no servidor
                    "event_id": "evt-1"
                }
            }
        ]
    }
```

```python
# backend/app/server.py
@app.post("/actions")
async def handle_action(action):
    if action.type == "view_event_details":
        # Expande evento no widget
        event = get_event(action.event_id)
        
        # Atualiza widget (não chama modelo)
        return {
            "widget_update": {
                "event_id": action.event_id,
                "collapsed": False,
                "details": event.full_description
            }
        }
```

**Por que server-handled:**
- Não precisa consultar modelo para expandir
- Mais rápido (atualização imediata)
- Economiza tokens
- Lógica centralizada no servidor

---

### 7. 📦 Contexto Oculto (Hidden Context)

Página de destaque é incluída como contexto invisível.

```python
# backend/app/agents/news_agent.py
def build_context():
    # Artigo featured é adicionado como contexto oculto
    featured = get_featured_article()
    
    hidden_context = f"""
    <FEATURED_ARTICLE>
      <ID>{featured.id}</ID>
      <TITLE>{featured.title}</TITLE>
      <SUMMARY>{featured.summary}</SUMMARY>
    </FEATURED_ARTICLE>
    """
    
    return hidden_context
```

**Por que é útil:**
```
Usuário: "O que está em destaque hoje?"
Agente: [vê FEATURED_ARTICLE no contexto oculto]
Agente: "O artigo em destaque é '{featured.title}'. 
         {featured.summary}"

Vs sem contexto oculto:
Agente: "Não sei qual artigo está em destaque. 
         Deixe-me buscar..." [chama ferramenta]
```

**💡 Vantagem:** Respostas mais rápidas para informações frequentes!

---

## 🏗️ Arquitetura do Código

### Backend (Python)

```
backend/
├── app/
│   ├── main.py                      # FastAPI, rotas, headers
│   ├── server.py                    # Roteamento de agentes
│   ├── agents/
│   │   ├── news_agent.py           # Agente principal
│   │   ├── event_finder_agent.py   # Agente de eventos
│   │   ├── puzzle_agent.py         # Agente de puzzles
│   │   └── title_agent.py          # Agente de títulos
│   ├── widgets/
│   │   ├── article_list_widget.py  # Widget de artigos
│   │   └── event_list_widget.py    # Widget de eventos
│   ├── thread_item_converter.py    # Conversão de entidades
│   └── database.py                  # Mock de banco de dados
```

### Frontend (React)

```
frontend/
├── src/
│   ├── App.tsx                      # Layout, roteamento
│   ├── components/
│   │   ├── ChatKitPanel.tsx        # Chat com entity handling
│   │   ├── ArticleView.tsx         # Visualização de artigo
│   │   ├── ArticleList.tsx         # Lista de artigos
│   │   └── EntityPreview.tsx       # Preview de @mentions
│   └── lib/
│       └── config.ts                # Config com composer tools
```

---

## 💡 Dicas de Aprendizado

### Para Desenvolvedores Intermediários:

1. **Entenda o fluxo de recuperação**
   - Como queries são interpretadas
   - Como múltiplas ferramentas são combinadas
   - Como resultados são deduplicados

2. **Explore entity tagging**
   - Como @ triggers o dropdown
   - Como tags são convertidas
   - Como o modelo as usa

3. **Estude múltiplos agentes**
   - Quando usar agente especializado vs geral
   - Como rotear requisições
   - Como compartilhar ferramentas comuns

### Exercícios Práticos:

#### Exercício 1: Nova ferramenta de busca
Adicione busca por data de publicação:
```python
def search_articles_by_date_range(
    start_date: str,
    end_date: str
) -> list[Article]:
    """Busca artigos publicados em um período"""
    # Implementar busca por range de datas
    pass
```

#### Exercício 2: Novo agente especializado
Crie um agente para análise de sentimento:
```python
sentiment_agent = Agent(
    instructions="Analise o sentimento de artigos...",
    tools=[analyze_sentiment, categorize_emotion]
)
```

#### Exercício 3: Widget personalizado
Crie widget de estatísticas de autor:
```python
def create_author_stats_widget(author_id):
    stats = calculate_author_stats(author_id)
    return {
        "type": "author-stats",
        "total_articles": stats.total,
        "avg_views": stats.avg_views,
        "top_tags": stats.top_tags
    }
```

---

## 🐛 Problemas Comuns

### @-Mentions não funcionam
**Causa:** Digitou via copy/paste
**Solução:** Digite @ manualmente no teclado para trigger do dropdown

### "Nenhum artigo aberto" ao pedir resumo de página
**Causa:** article-id não está sendo enviado
**Solução:** 
1. Verifique se um artigo está realmente aberto
2. Confirme que custom fetch está configurado
3. Veja logs do backend para confirmar recebimento do header

### Progress não aparece
**Causa:** onProgress não configurado ou ProgressUpdateEvent não sendo emitido
**Solução:** Adicione log:
```typescript
onProgress: (p) => console.log("Progress:", p)
```

### Agente errado responde
**Causa:** tool_choice não sendo enviado ou roteamento incorreto
**Solução:** Verifique que:
1. Compositor tem ferramenta selecionada
2. tool_choice está na requisição
3. Backend tem case para esse tool_choice

---

## 🎯 Próximos Passos

Depois de dominar News Guide:

1. **Explore Metro Map** (avançado)
   - Ferramentas do cliente
   - Visualizações complexas
   - Anotações inline

2. **Construa seu próprio**
   - Sistema de busca para seu domínio
   - Múltiplos agentes especializados
   - Entity tags personalizadas

3. **Otimize performance**
   - Cache de buscas frequentes
   - Paginação de resultados
   - Busca incremental

---

## 📚 Recursos Relacionados

- [Documentação do ChatKit](https://platform.openai.com/docs/chatkit)
- [Guia de Integração](../../agents.md)
- [ElasticSearch](https://www.elastic.co/) - Para busca full-text em produção
- [React Router](https://reactrouter.com/) - Para navegação entre artigos

---

**📰 Pronto para criar experiências ricas de busca e recuperação!** ✨

Explore os diferentes agentes especializados e veja como múltiplos cérebros trabalham juntos!
