# Metro Map (Mapa de Metrô)

## 🗺️ Sobre o Projeto

Atualizações de GUI orientadas por chat para um mapa de metrô usando um canvas React Flow que permite ao usuário estender linhas com novas estações.

**Nível de dificuldade:** 🔴 Avançado

**O que você aprenderá:**
- Ferramentas do cliente (client tools) para buscar estado da UI
- Sincronização bidirecional entre chat e visualização
- Anotações inline de entidades
- Bloqueio de interação durante streaming
- Manipulação de estado de canvas complexo
- Actions personalizadas no header

---

## 🚀 Início Rápido

```bash
# Exporte variáveis de ambiente
export OPENAI_API_KEY="sua-chave-aqui"
export VITE_CHATKIT_API_DOMAIN_KEY="domain_pk_local_dev"

# Da raiz do repositório
npm run metro-map

# OU do diretório do exemplo
cd examples/metro-map
npm install
npm run start
```

✅ Acesse: **http://localhost:5173**

---

## 💬 Prompts de Exemplo

### Adicionar Estações
- **"Adicione uma nova estação chamada Aurora"**
  - 🎯 Demonstra: Widget de seleção de linha aparecerá
  - Você escolhe em qual linha adicionar
  - Clica no mapa onde deseja colocar a estação
  - Estação é adicionada e canvas foca nela

### Planejamento de Rotas
- **"Planeje uma rota de Titan Border até Lyra Verge."**
  - 🎯 Demonstra: Ferramenta `plan_route` com anotações
  - Cada estação na rota vira uma anotação clicável
  - Clicar na anotação move o mapa para aquela estação

### @-Mentions de Estações
- **"Me fale sobre a estação @Cinderia."**
  - 🎯 Demonstra: Entity tags de estações
  - **⚠️ Importante:** Digite @ manualmente, não funciona com copy/paste!
  - Clicar na @ mention foca a estação no mapa

### Consultas Baseadas em Seleção
- **"Me fale sobre as estações que selecionei."**
  - 🎯 Demonstra: Client tool `get_selected_stations`
  - Primeiro: Use laço no canvas para selecionar estações
  - Depois: Faça a pergunta
  - Agente busca seleção e responde sobre elas

---

## 🎓 Funcionalidades Explicadas

Este é o exemplo mais avançado, demonstrando sincronização sofisticada entre chat e visualização interativa.

---

### 1. 🗺️ Sincronização de Mapa

Ferramentas que mantêm o agente atualizado sobre o estado do mapa.

#### Arquitetura do Estado:

```
┌──────────────────────────────────┐
│        Metro Map State           │
│                                  │
│  Lines: [                        │
│    {id: "blue", stations: [...]} │
│    {id: "red", stations: [...]}  │
│  ]                               │
│                                  │
│  Stations: [                     │
│    {id: "s1", name: "Aurora"}    │
│    {id: "s2", name: "Cinderia"}  │
│  ]                               │
└──────────────────────────────────┘
         ↕️ Sincronização
┌──────────────────────────────────┐
│     React Flow Canvas            │
│                                  │
│  [Nodes e Edges renderizados]   │
└──────────────────────────────────┘
```

#### Ferramentas de Sincronização:

```python
# backend/app/agents/metro_map_agent.py

def get_map() -> dict:
    """Obtém o mapa completo com todas as linhas e estações"""
    yield ProgressUpdateEvent(title="Carregando mapa...")
    
    map_data = database.get_full_map()
    
    return {
        "lines": map_data.lines,
        "stations": map_data.stations,
        "connections": map_data.connections
    }

def list_lines() -> list[Line]:
    """Lista todas as linhas do metrô"""
    return [
        {"id": "blue", "name": "Linha Azul", "stations": 8},
        {"id": "red", "name": "Linha Vermelha", "stations": 6}
    ]

def list_stations() -> list[Station]:
    """Lista todas as estações"""
    return database.all_stations()

def get_line_route(line_id: str) -> list[Station]:
    """Obtém todas as estações de uma linha em ordem"""
    return database.get_line_stations(line_id)

def get_station(station_id: str) -> Station:
    """Obtém detalhes completos de uma estação"""
    return database.get_station(station_id)
```

**Quando são usadas:**
- `get_map`: Início da conversa, quando agente precisa visão geral
- `list_lines`: Quando usuário pergunta sobre linhas disponíveis
- `get_station`: Quando usuário pergunta sobre estação específica
- `get_line_route`: Para calcular rotas entre estações

---

### 2. 💻 Client Tools (Ferramentas do Cliente)

Ferramentas executadas no navegador que retornam estado da UI ao agente.

#### O que são Client Tools?

**Server tools:**
```
Agente → Servidor → Banco de dados → Resultado → Agente
```

**Client tools:**
```
Agente → Servidor → Cliente (navegador) → Estado da UI → Servidor → Agente
```

#### Implementação de `get_selected_stations`:

```python
# backend/app/agents/metro_map_agent.py
get_selected_stations_tool = {
    "type": "client_tool",
    "name": "get_selected_stations",
    "description": "Obtém estações atualmente selecionadas no canvas",
    "timeout": 5000  # 5 segundos de timeout
}
```

```typescript
// frontend - ChatKitPanel.tsx
onClientTool: async (tool) => {
    if (tool.name === "get_selected_stations") {
        // Busca nós selecionados no React Flow
        const selectedNodes = reactFlowInstance.getNodes()
            .filter(node => node.selected)
        
        // Retorna IDs das estações selecionadas
        return {
            station_ids: selectedNodes.map(n => n.id)
        }
    }
}
```

**Fluxo completo:**

```
1. Usuário usa laço no canvas para selecionar 3 estações
2. Usuário: "Qual a distância entre estas estações?"
3. Agente decide chamar get_selected_stations
4. Servidor envia requisição de client tool ao frontend
5. Frontend busca estações selecionadas no React Flow
6. Frontend retorna: {station_ids: ["s1", "s2", "s3"]}
7. Servidor recebe resultado
8. Agente processa: "Você selecionou: Aurora, Cinderia, Lyra..."
9. Agente calcula distâncias entre elas
```

**Com progress update:**

```python
def handle_client_tool_call():
    yield ProgressUpdateEvent(
        title="Aguardando seleção...",
        status="waiting"
    )
    
    result = await call_client_tool("get_selected_stations")
    
    yield ProgressUpdateEvent(
        title="Seleção recebida",
        status="complete"
    )
    
    return result
```

**💡 Por que é poderoso:**
- Agente "vê" o que usuário fez na UI
- Conversa consciente de interações visuais
- Não precisa pedir ao usuário "diga quais estações"

---

### 3. 🎯 Anotações Inline (Inline Annotations)

Entidades clicáveis dentro das mensagens do agente.

#### Como funcionam:

**Backend cria anotação:**
```python
# backend/app/agents/metro_map_agent.py
def plan_route(from_station, to_station):
    # Calcula rota
    route = calculate_route(from_station, to_station)
    
    # Cria anotações para cada estação
    annotations = []
    for station in route:
        annotations.append({
            "type": "entity",
            "entity_id": station.id,
            "entity_type": "station",
            "text": station.name,
            "metadata": {
                "line": station.line,
                "position": station.position
            }
        })
    
    # Monta resposta
    response = f"Rota sugerida: {' → '.join([s.name for s in route])}"
    
    return {
        "text": response,
        "annotations": annotations
    }
```

**Frontend renderiza:**
```typescript
// Mensagem do agente:
"Rota sugerida: Titan Border → Midway → Cinderia → Lyra Verge"

// Cada nome de estação é renderizado como:
<EntityAnnotation
    onClick={() => focusStation("titan-border")}
    onHover={() => showStationPreview("titan-border")}
>
    Titan Border
</EntityAnnotation>
```

**Interação do usuário:**

```
[Mensagem do agente com rota]
"Pegue a linha azul: Titan Border → Midway → Cinderia"
                      ↑ clicável    ↑ clicável  ↑ clicável

Usuário clica em "Cinderia"
    ↓
ChatKitPanel.tsx captura evento
    ↓
onEntityClick: (entity) => {
    reactFlowInstance.fitView({
        nodes: [{ id: entity.id }],
        duration: 500
    })
}
    ↓
Canvas anima movimento para a estação
    ↓
Estação fica no centro e destacada
```

**Lista de fontes:**

Além das anotações inline, as entidades também aparecem em uma lista lateral:

```
Sources (4)
├─ 📍 Titan Border (Blue Line)
├─ 📍 Midway (Blue Line)
├─ 📍 Cinderia (Blue & Red Line)
└─ 📍 Lyra Verge (Blue Line)
```

Clicar em qualquer fonte também foca a estação no mapa.

---

### 4. 🔒 Bloqueio de Interação Durante Streaming

Previne conflitos enquanto o agente atualiza o mapa.

**Problema sem bloqueio:**
```
Agente está adicionando estação "Aurora"
    ↓
Usuário arrasta canvas
    ↓
Posição da nova estação fica incorreta
    ↓
Estado inconsistente
```

**Solução:**

```typescript
// frontend - ChatKitPanel.tsx
const [isResponseActive, setIsResponseActive] = useState(false)

<ChatKit
    onResponseStart={() => {
        setIsResponseActive(true)
        // Desabilita interações
        setCanvasInteractive(false)
    }}
    
    onResponseEnd={() => {
        setIsResponseActive(false)
        // Re-habilita interações
        setCanvasInteractive(true)
    }}
/>

<ReactFlow
    nodesDraggable={!isResponseActive}
    nodesConnectable={!isResponseActive}
    elementsSelectable={!isResponseActive}
>
```

**Estados visuais:**

```css
/* Durante resposta do agente */
.canvas.locked {
    cursor: wait;
    opacity: 0.7;
    pointer-events: none;
}

/* Normal */
.canvas.unlocked {
    cursor: grab;
    opacity: 1;
    pointer-events: all;
}
```

**Feedback ao usuário:**

```typescript
{isResponseActive && (
    <OverlayMessage>
        🔄 Agente está atualizando o mapa...
    </OverlayMessage>
)}
```

---

### 5. 🎨 Fluxo de Criação de Estação

Workflow complexo com múltiplas etapas.

**Passo a passo completo:**

#### Etapa 1: Usuário solicita
```
Usuário: "Adicione uma estação chamada Aurora"
```

#### Etapa 2: Agente mostra seletor de linha
```python
def show_line_selector(station_name: str):
    lines = list_lines()
    
    return {
        "type": "line-selector",
        "question": f"Em qual linha adicionar {station_name}?",
        "options": [
            {
                "id": "blue",
                "label": "Linha Azul",
                "action": {
                    "type": "line.select",
                    "line_id": "blue",
                    "station_name": station_name
                }
            },
            # Mais linhas...
        ]
    }
```

#### Etapa 3: Usuário escolhe linha (ação de servidor)
```python
# backend/app/server.py
@app.post("/actions")
async def handle_action(action):
    if action.type == "line.select":
        # 1. Adiciona contexto oculto
        add_hidden_context(
            f"<LINE_SELECTED>{action.line_id}</LINE_SELECTED>"
        )
        
        # 2. Atualiza widget (marca linha escolhida)
        update_widget(action.widget_id, {
            "selected_line": action.line_id
        })
        
        # 3. Stream mensagem do assistente
        yield assistant_message(
            "Ótimo! Agora clique no mapa onde deseja adicionar "
            f"{action.station_name} na Linha {action.line_id}."
        )
        
        # 4. Dispara modo de seleção no cliente
        yield client_effect({
            "type": "location_select_mode",
            "enabled": True,
            "line_id": action.line_id,
            "station_name": action.station_name
        })
```

#### Etapa 4: Cliente entra em modo de seleção
```typescript
// frontend - ChatKitPanel.tsx
onEffect: (effect) => {
    if (effect.type === "location_select_mode") {
        if (effect.enabled) {
            // Muda cursor
            setMapCursor("crosshair")
            
            // Mostra instrução
            showInstruction(
                "Clique onde deseja adicionar a estação"
            )
            
            // Registra handler de clique
            setOnMapClick((position) => {
                // Cria estação no canvas
                addStationToCanvas({
                    id: generateId(),
                    name: effect.station_name,
                    line: effect.line_id,
                    position: position
                })
                
                // Notifica servidor
                sendStationCreated({
                    station_name: effect.station_name,
                    line_id: effect.line_id,
                    position: position
                })
            })
        }
    }
}
```

#### Etapa 5: Agente confirma e foca
```python
# Quando servidor recebe notificação de estação criada
def on_station_created(station_data):
    # Salva no banco de dados
    new_station = database.create_station(station_data)
    
    # Emite efeito para focar estação
    yield client_effect({
        "type": "add_station",
        "station_id": new_station.id,
        "focus": True
    })
    
    # Mensagem de confirmação
    return f"Estação {new_station.name} adicionada com sucesso!"
```

```typescript
// Cliente processa efeito add_station
onEffect: (effect) => {
    if (effect.type === "add_station") {
        // Adiciona estação ao mapa (se ainda não estiver)
        if (!hasStation(effect.station_id)) {
            addStationToCanvas(effect.station_id)
        }
        
        // Foca na nova estação
        if (effect.focus) {
            reactFlowInstance.fitView({
                nodes: [{ id: effect.station_id }],
                duration: 800,
                padding: 0.3
            })
        }
    }
}
```

**Fluxo visual:**

```
┌──────────────────────────────────┐
│ Chat: "Adicione estação Aurora"  │
└──────────────────────────────────┘
            ↓
┌──────────────────────────────────┐
│ Widget: Selecione a linha        │
│  [ ] Linha Azul                  │
│  [ ] Linha Vermelha              │
└──────────────────────────────────┘
            ↓ (usuário clica Azul)
┌──────────────────────────────────┐
│ Mapa: Cursor vira crosshair      │
│ "Clique onde adicionar"          │
└──────────────────────────────────┘
            ↓ (usuário clica no mapa)
┌──────────────────────────────────┐
│ Nova estação aparece             │
│ Câmera anima para ela            │
│ Chat: "Adicionada com sucesso!"  │
└──────────────────────────────────┘
```

---

### 6. 🏷️ Entity Tags de Estações

@-mentions de estações com conversão inteligente.

**Frontend:**
```typescript
// ChatKitPanel.tsx - Entity search
onEntitySearch: async (query) => {
    // Busca estações que correspondem
    const stations = await searchStations(query)
    
    return stations.map(s => ({
        id: s.id,
        name: s.name,
        category: "station",
        preview: {
            title: s.name,
            subtitle: `${s.line} Line`,
            metadata: `${s.connections.length} connections`
        }
    }))
}

// Quando @ mention é clicada
onEntityClick: (entity) => {
    if (entity.category === "station") {
        focusStationOnCanvas(entity.id)
    }
}
```

**Backend - Conversão:**
```python
# backend/app/thread_item_converter.py
def convert_entity_tag(tag):
    if tag.category == "station":
        # Busca informações completas
        station = database.get_station(tag.id)
        
        # Converte em marcador rico
        return f"""
        <STATION_TAG id="{station.id}">
          <NAME>{station.name}</NAME>
          <LINE>{station.line}</LINE>
          <CONNECTIONS>{', '.join(station.connections)}</CONNECTIONS>
          <POSITION x="{station.x}" y="{station.y}" />
        </STATION_TAG>
        """
```

**Instruções do agente:**
```python
instructions = """
Quando vir <STATION_TAG>, você já tem todas as informações da estação.
NÃO chame get_station() novamente - os dados já estão no tag.

Exemplo:
Usuário: "Fale sobre @Cinderia"
Você vê: <STATION_TAG id="cinderia">
         <NAME>Cinderia</NAME>
         <LINE>Blue</LINE>
         <CONNECTIONS>Midway, Lyra Verge</CONNECTIONS>
         </STATION_TAG>

Resposta: "Cinderia é uma estação na Linha Azul, conectando 
          Midway e Lyra Verge. [informações detalhadas]"
"""
```

**💡 Otimização:** Tag já tem todos os dados, economiza chamada de ferramenta!

---

### 7. 🎨 Ações Personalizadas no Header

Botão de tema no cabeçalho do chat.

```typescript
// frontend - ChatKitPanel.tsx
<ChatKit
    header={{
        actions: [
            {
                id: "theme-toggle",
                icon: isDark ? "light-mode" : "dark-mode",
                label: isDark ? "Modo Claro" : "Modo Escuro",
                onClick: () => {
                    setIsDark(!isDark)
                    applyTheme(!isDark ? "dark" : "light")
                }
            }
        ]
    }}
/>
```

**Aplicação de tema:**
```typescript
function applyTheme(theme: "dark" | "light") {
    // Atualiza CSS variables
    document.documentElement.setAttribute("data-theme", theme)
    
    // Atualiza cores do React Flow
    setReactFlowStyle({
        background: theme === "dark" ? "#1a1a1a" : "#ffffff",
        edge: theme === "dark" ? "#666" : "#ccc"
    })
    
    // Salva preferência
    localStorage.setItem("theme", theme)
}
```

**Temas disponíveis:**
```css
/* Modo claro */
[data-theme="light"] {
    --bg-primary: #ffffff;
    --text-primary: #000000;
    --station-color: #2563eb;
}

/* Modo escuro */
[data-theme="dark"] {
    --bg-primary: #1a1a1a;
    --text-primary: #ffffff;
    --station-color: #60a5fa;
}
```

---

## 🏗️ Arquitetura do Código

### Backend (Python)

```
backend/
├── app/
│   ├── main.py                      # FastAPI app
│   ├── server.py                    # Handlers de ações
│   ├── agents/
│   │   ├── metro_map_agent.py      # Agente principal
│   │   └── title_agent.py          # Agente de títulos
│   ├── widgets/
│   │   └── line_select_widget.py   # Widget seletor
│   ├── thread_item_converter.py    # Conversão de tags
│   └── data/
│       └── metro_map.json          # Dados do mapa
```

### Frontend (React + React Flow)

```
frontend/
├── src/
│   ├── App.tsx                      # Layout principal
│   ├── components/
│   │   ├── ChatKitPanel.tsx        # Chat + handlers
│   │   ├── MetroMap.tsx            # Canvas React Flow
│   │   ├── StationNode.tsx         # Nó de estação
│   │   ├── LineEdge.tsx            # Edge de linha
│   │   └── EntityPreview.tsx       # Preview de @ mention
│   ├── hooks/
│   │   ├── useMetroData.ts         # Hook de dados
│   │   └── useStationSelection.ts  # Hook de seleção
│   └── lib/
│       ├── config.ts                # Config ChatKit
│       └── routing.ts               # Cálculo de rotas
```

---

## 💡 Dicas de Aprendizado

### Para Desenvolvedores Avançados:

1. **Entenda React Flow profundamente**
   - Como nós e edges são gerenciados
   - Como animações de câmera funcionam
   - Como seleção múltipla é implementada

2. **Domine client tools**
   - Quando usar vs server tools
   - Como lidar com timeouts
   - Como fornecer feedback durante espera

3. **Estude sincronização bidirecional**
   - Chat → Canvas (efeitos)
   - Canvas → Chat (client tools, entity clicks)
   - Como prevenir loops infinitos

### Exercícios Práticos:

#### Exercício 1: Nova ferramenta do cliente
Crie tool para obter zoom atual do mapa:
```typescript
onClientTool: async (tool) => {
    if (tool.name === "get_viewport") {
        return {
            zoom: reactFlowInstance.getZoom(),
            center: reactFlowInstance.getViewport()
        }
    }
}
```

#### Exercício 2: Anotações avançadas
Adicione tempo de viagem nas anotações de rota:
```python
annotations.append({
    "type": "entity",
    "entity_id": station.id,
    "metadata": {
        "travel_time": calculate_time(prev, station),
        "distance_km": calculate_distance(prev, station)
    }
})
```

#### Exercício 3: Modo de edição
Crie modo onde usuário pode arrastar estações:
```typescript
const [editMode, setEditMode] = useState(false)

<ReactFlow
    nodesDraggable={editMode}
    onNodeDragStop={(event, node) => {
        // Salva nova posição
        updateStationPosition(node.id, node.position)
        
        // Notifica agente
        chatkit.sendSystemMessage(
            `Station ${node.data.name} moved to ${node.position}`
        )
    }}
/>
```

---

## 🐛 Problemas Comuns

### Client tool timeout
**Causa:** Frontend não respondeu em 5 segundos
**Solução:** 
1. Verifique se onClientTool está implementado
2. Adicione logs para debug
3. Aumente timeout se operação é lenta

### Estação não foca após criação
**Causa:** ID da estação não corresponde entre backend e frontend
**Solução:** Use ID consistente gerado no backend e passado via efeito

### Interação travada
**Causa:** `onResponseEnd` não foi chamado (erro no streaming)
**Solução:** Adicione timeout de segurança:
```typescript
useEffect(() => {
    if (isResponseActive) {
        const timeout = setTimeout(() => {
            setIsResponseActive(false)
        }, 30000) // 30s timeout
        
        return () => clearTimeout(timeout)
    }
}, [isResponseActive])
```

### Anotações não clicáveis
**Causa:** `onEntityClick` não configurado
**Solução:** Verifique configuração do ChatKit

---

## 🎯 Próximos Passos

Você dominou o exemplo mais avançado! Agora pode:

1. **Construir visualizações complexas**
   - Gráficos interativos com D3.js
   - Dashboards com atualizações em tempo real
   - Editores visuais com IA

2. **Implementar sincronização sofisticada**
   - Múltiplos usuários colaborando
   - Undo/redo com histórico
   - Conflito resolution

3. **Criar aplicações profissionais**
   - CAD/Modeling tools com IA
   - Planejamento de projetos visual
   - Editores de diagramas inteligentes

---

## 📚 Recursos Relacionados

- [Documentação do ChatKit](https://platform.openai.com/docs/chatkit)
- [React Flow Docs](https://reactflow.dev/) - Biblioteca de visualização
- [Guia de Integração](../../agents.md)
- [Client Tools Best Practices](https://platform.openai.com/docs/chatkit/client-tools)

---

**🗺️ Parabéns por completar o exemplo mais avançado!** ✨

Você agora entende:
- Sincronização bidirecional complexa
- Client tools para estado da UI
- Anotações interativas
- Visualizações sofisticadas com IA

Use este conhecimento para criar experiências visuais incríveis com ChatKit!
