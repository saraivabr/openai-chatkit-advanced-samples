# Cat Lounge (Lounge do Gato)

## 🐱 Sobre o Projeto

Demonstração de cuidador virtual de gato construída com ChatKit (backend FastAPI + frontend Vite/React).

**Nível de dificuldade:** 🟢 Iniciante

**O que você aprenderá:**
- Conceitos básicos de agentes com estado
- Ferramentas do servidor que leem e modificam dados
- Widgets interativos com ações
- Efeitos do cliente para sincronizar a UI
- Geração de imagens com IA

---

## 🚀 Início Rápido

### Opção 1: A partir da raiz do repositório (mais fácil)

```bash
npm run cat-lounge
```

### Opção 2: A partir do diretório do exemplo

```bash
cd examples/cat-lounge
npm install
npm run start
```

**Pré-requisitos:**
1. Exporte sua chave da API OpenAI: `export OPENAI_API_KEY="sk-proj-..."`
2. Tenha `uv` instalado (gerenciador de pacotes Python)

✅ Acesse: **http://localhost:5170**

---

## 💬 Prompts de Exemplo

Experimente estas interações para ver diferentes funcionalidades:

### Básico - Interação com o gato
- **"Alimente o gato com um petisco de atum."**
  - 🎯 Demonstra: Ferramenta `feed_cat` + atualização de status
  
- **"O gato parece um pouco sujo—dê um banho nele."**
  - 🎯 Demonstra: Ferramenta `clean_cat` + efeito visual

- **"Olá, gato! Como você está se sentindo?"**
  - 🎯 Demonstra: Ferramenta `speak_as_cat` + balão de fala

### Intermediário - Widgets e nomeação
- **"Que nome devo dar ao gato?"**
  - 🎯 Demonstra: Widget de sugestões de nomes com botões clicáveis
  - O agente mostrará 3 opções de nomes em cartões
  - Você pode clicar para escolher ou pedir mais sugestões

- **"Posso ver o cartão de perfil do gato?"**
  - 🎯 Demonstra: Widget de apresentação (só visualização)
  - Mostra todas as estatísticas do gato de forma bonita

### Avançado - Estado e contexto
- **"Brinque com o gato e depois alimente com salmão."**
  - 🎯 Demonstra: Múltiplas ações em sequência
  - O agente lembrará de ambas as ações com contexto oculto

---

## 🎓 Funcionalidades Explicadas

Este exemplo é perfeito para iniciantes porque demonstra conceitos fundamentais de forma simples.

### 1. 🔧 Ferramentas do Servidor (Server Tools)

Ferramentas são funções que o agente pode chamar para obter ou modificar dados.

#### `get_cat_status` - Ler informações
```python
def get_cat_status() -> dict:
    """Obtém o status atual do gato (energia, felicidade, limpeza)"""
    return {
        "energy": 75,
        "happiness": 60,
        "cleanliness": 80,
        "name": "Whiskers"
    }
```
**Quando é usado:** Toda vez que o agente precisa saber como o gato está antes de responder.

#### `feed_cat` - Modificar dados
```python
def feed_cat(food_type: str) -> str:
    """Alimenta o gato e aumenta sua energia"""
    # Atualiza o estado interno
    cat_state.energy += 20
    # Dispara efeito visual
    emit_effect("update_cat_status", cat_state)
    return f"Gato alimentado com {food_type}!"
```
**Quando é usado:** Quando você pede para alimentar o gato.

#### Outras ferramentas disponíveis:
- `play_with_cat` - Aumenta felicidade, reduz energia
- `clean_cat` - Aumenta limpeza
- `set_cat_name` - Define nome do gato e atualiza título da thread
- `speak_as_cat` - Faz o gato "falar" com o usuário

**💡 Conceito importante:** O agente **decide sozinho** quando usar cada ferramenta baseado na conversa!

---

### 2. 🎨 Widgets Interativos

Widgets são componentes visuais ricos que aparecem no chat.

#### Widget de Sugestões de Nome (com ações)

**Como funciona:**
1. Você pergunta "Que nome devo dar ao gato?"
2. Agente chama `suggest_cat_names()` 
3. Backend cria um widget com 3 sugestões
4. Cada nome tem um botão "Escolher"
5. Há também um botão "Mais nomes"

**Código simplificado:**
```python
def suggest_cat_names():
    return Widget(
        type="name-suggestions",
        names=["Whiskers", "Shadow", "Luna"],
        actions=[
            {"id": "cats.select_name", "label": "Escolher"},
            {"id": "cats.more_names", "label": "Mais nomes"}
        ]
    )
```

**Fluxo de ação:**
```
Usuário clica "Escolher Whiskers"
    ↓
Frontend captura ação (handleWidgetAction)
    ↓
Envia para servidor (chatkit.sendAction)
    ↓
Servidor processa ação (cats.select_name)
    ↓
Atualiza nome do gato
    ↓
Envia widget atualizado
    ↓
UI mostra confirmação
```

#### Widget de Perfil (só visualização)

**Como funciona:**
1. Você pede "Mostre o perfil do gato"
2. Agente chama `show_cat_profile()`
3. Backend cria um widget bonito com todas as informações
4. **Sem botões** - só para visualizar

**💡 Diferença importante:**
- **Widgets com ações**: Usuário pode interagir (clicar, escolher)
- **Widgets sem ações**: Só mostram informação

---

### 3. 🎬 Efeitos do Cliente

Efeitos são comandos do servidor para atualizar a UI sem esperar resposta.

#### `update_cat_status` - Sincronizar estatísticas

**Quando acontece:** Após QUALQUER ação que mude o estado do gato

```python
# No servidor (backend)
def feed_cat(food_type):
    cat_state.energy += 20
    emit_effect("update_cat_status", {
        "energy": cat_state.energy,
        "happiness": cat_state.happiness,
        "cleanliness": cat_state.cleanliness
    })
```

```typescript
// No cliente (frontend)
onEffect: (effect) => {
    if (effect.name === "update_cat_status") {
        // Atualiza as barras de progresso na UI
        updateCatBars(effect.data)
    }
}
```

**Por que usar efeitos?**
- ✅ Atualização imediata da UI
- ✅ Não bloqueia a resposta do agente
- ✅ Mantém UI sincronizada com o estado

#### `cat_say` - Balão de fala

**Quando acontece:** Quando o agente chama `speak_as_cat()`

```python
# No servidor
emit_effect("cat_say", {
    "message": "Miau! Obrigado pela comida! 😺"
})
```

```typescript
// No cliente
onEffect: (effect) => {
    if (effect.name === "cat_say") {
        // Mostra balão de fala temporário
        showSpeechBubble(effect.data.message, 3000)
    }
}
```

---

### 4. 📝 Contexto Oculto (Hidden Context)

Contexto oculto são tags invisíveis ao usuário que o agente usa para lembrar do histórico.

**Como funciona:**
```python
# Após alimentar o gato
add_hidden_context("<FED_CAT timestamp='14:30'>tuna</FED_CAT>")

# Após brincar
add_hidden_context("<PLAYED_WITH_CAT timestamp='14:35'>ball</PLAYED_WITH_CAT>")

# Após nomear
add_hidden_context("<CAT_NAME_SELECTED>Whiskers</CAT_NAME_SELECTED>")
```

**Por que é útil:**
- O agente lembra ações recentes
- Pode referenciá-las em conversas futuras
- Evita ações duplicadas (ex: não alimenta o gato 2x seguidas)

**Exemplo de uso inteligente:**
```
Você: "Alimente o gato"
Agente: [vê <FED_CAT timestamp='5 minutos atrás'>]
Agente: "O gato já comeu há 5 minutos! Que tal brincar com ele?"
```

---

### 5. ⚡ Ações Rápidas (Quick Actions)

Botões fora do chat que enviam mensagens predefinidas.

**No código (App.tsx):**
```typescript
<button onClick={() => chatkit.sendUserMessage("Alimente o gato")}>
    🍽️ Alimentar
</button>
<button onClick={() => chatkit.sendUserMessage("Brinque com o gato")}>
    🎾 Brincar
</button>
<button onClick={() => chatkit.sendUserMessage("Dê banho no gato")}>
    🛁 Banho
</button>
```

**Benefício:** Usuários podem realizar ações comuns com um clique, sem digitar.

---

### 6. 🎨 Geração de Imagens com IA

O agente pode gerar imagens do gato durante a conversa!

**Como funciona:**
```python
cat_agent = Agent(
    instructions="...",
    tools=[ImageGenerationTool()],  # Adiciona capacidade de gerar imagens
)

# Ao processar resposta
stream_agent_response(
    agent=cat_agent,
    converter=ResponseStreamConverter(partial_images=3)  # Mostra progresso
)
```

**Fluxo de geração:**
1. Usuário: "Mostre como meu gato está"
2. Agente decide gerar imagem
3. Chama DALL-E com descrição (ex: "gato fofo feliz")
4. Mostra 3 imagens parciais durante geração (progresso visual)
5. Exibe imagem final no chat

**💡 Dica:** As imagens parciais fazem com que a espera pareça mais rápida!

---

## 🏗️ Arquitetura do Código

### Backend (Python)

```
backend/
├── app/
│   ├── main.py              # FastAPI app, rotas HTTP
│   ├── server.py            # Lógica do ChatKit, handlers
│   ├── cat_agent.py         # Definição do agente e ferramentas
│   ├── cat_state.py         # Gerenciamento de estado do gato
│   ├── profile_card_widget.py  # Widget de perfil
│   └── name_suggestions_widget.py  # Widget de nomes
```

**Fluxo de uma requisição:**
```
1. Frontend envia mensagem
2. main.py recebe (rota /chatkit)
3. server.py processa
4. cat_agent.py executa ferramentas
5. cat_state.py atualiza dados
6. server.py emite efeitos
7. Resposta volta para frontend
```

### Frontend (React)

```
frontend/
├── src/
│   ├── App.tsx              # Componente principal, layout
│   ├── components/
│   │   ├── ChatKitPanel.tsx # Componente do chat
│   │   ├── CatDisplay.tsx   # Visualização do gato
│   │   └── StatsBar.tsx     # Barras de energia/felicidade
│   └── lib/
│       └── config.ts        # Configuração do ChatKit
```

**Fluxo de uma ação:**
```
1. Usuário clica em widget
2. ChatKitPanel.tsx captura (handleWidgetAction)
3. Envia para backend (chatkit.sendAction)
4. Backend processa
5. Efeito volta (onEffect)
6. CatDisplay.tsx atualiza visual
```

---

## 🔍 Entendendo o Código Passo a Passo

### Como adicionar uma nova ação para o gato

**Passo 1:** Crie a ferramenta no `cat_agent.py`
```python
def pet_cat() -> str:
    """Faz carinho no gato, aumenta felicidade levemente"""
    cat_state.happiness += 5
    emit_effect("update_cat_status", cat_state.to_dict())
    emit_effect("cat_say", {"message": "Purrrr... 😸"})
    return "Você fez carinho no gato!"
```

**Passo 2:** Registre a ferramenta
```python
cat_agent = Agent(
    instructions="...",
    tools=[
        get_cat_status,
        feed_cat,
        play_with_cat,
        pet_cat,  # Nova ferramenta
    ]
)
```

**Passo 3:** Teste!
```
Você: "Faça carinho no gato"
Agente: [chama pet_cat automaticamente]
UI: [atualiza felicidade] [mostra balão "Purrrr..."]
```

**Pronto!** O agente agora sabe fazer carinho no gato. ✨

---

## 💡 Dicas de Aprendizado

### Para Iniciantes:
1. **Execute primeiro**, entenda depois
   - Rode o exemplo e brinque com ele
   - Observe o que acontece no terminal
   - Use as ferramentas de desenvolvedor do navegador (F12)

2. **Comece pequeno**
   - Mude apenas uma coisa por vez
   - Teste cada mudança
   - Se quebrar, desfaça (Ctrl+Z)

3. **Leia o código em ordem**
   - `cat_agent.py` - veja as ferramentas
   - `server.py` - veja como processa
   - `ChatKitPanel.tsx` - veja como renderiza

### Exercícios Práticos:

#### Exercício 1: Modificar valores
Mude quanto de energia o gato ganha ao comer:
```python
# Em cat_agent.py, linha ~XX
cat_state.energy += 20  # Mude para 30
```

#### Exercício 2: Nova ferramenta simples
Crie uma ferramenta que dá água ao gato:
```python
def give_water() -> str:
    """Dá água ao gato"""
    cat_state.hydration += 15  # Você precisará adicionar este atributo!
    return "Gato bebeu água!"
```

#### Exercício 3: Novo efeito visual
Faça o gato piscar quando estiver com muita energia:
```python
if cat_state.energy > 90:
    emit_effect("cat_excited", {})
```

---

## 🐛 Problemas Comuns

### O gato não responde
**Causa:** Backend pode não estar rodando
**Solução:** 
```bash
cd backend
uv run uvicorn app.main:app --reload
```

### Ações não funcionam
**Causa:** Ações não foram registradas no `handleWidgetAction`
**Solução:** Verifique em `ChatKitPanel.tsx`:
```typescript
handleWidgetAction: async (action) => {
    if (action.type === "cats.select_name") {
        // Handler aqui
    }
}
```

### Estado não sincroniza
**Causa:** Efeito não está sendo processado
**Solução:** Adicione log no `onEffect`:
```typescript
onEffect: (effect) => {
    console.log("Efeito recebido:", effect)
    // ...
}
```

---

## 🎯 Próximos Passos

Depois de dominar o Cat Lounge:

1. **Experimente Customer Support** (nível intermediário)
   - Aprenda sobre widgets mais complexos
   - Veja anexos de imagens
   - Entenda estado mais sofisticado

2. **Explore News Guide** (nível intermediário)
   - Aprenda sobre recuperação de dados
   - Veja @-mentions em ação
   - Entenda composer tools

3. **Desafie-se com Metro Map** (nível avançado)
   - Ferramentas do cliente
   - Visualizações complexas
   - Sincronização bidirecional

---

## 📚 Recursos Relacionados

- [Documentação do ChatKit](https://platform.openai.com/docs/chatkit)
- [Guia de Integração](../../agents.md) - Documento principal de integração
- [FastAPI Docs](https://fastapi.tiangolo.com/) - Framework do backend
- [React Docs](https://react.dev/) - Framework do frontend

---

**🎉 Divirta-se cuidando do seu gato virtual!** 🐱✨

Se tiver dúvidas, leia o código ou consulte a [documentação principal](../../README.md).
