# Customer Support (Suporte ao Cliente)

## 🎫 Sobre o Projeto

Demonstração de suporte ao cliente de nível profissional com fluxo alimentado pelo ChatKit. Este exemplo combina um backend FastAPI específico para o cenário com uma UI React, permitindo que agentes conversem com viajantes enquanto o painel lateral exibe dados de itinerário ao vivo, status de fidelidade e ações de serviço recentes.

**Nível de dificuldade:** 🟡 Intermediário

**O que você aprenderá:**
- Gerenciamento de estado por thread (sessão)
- Widgets complexos com múltiplas ações
- Anexos de imagem end-to-end
- Sincronização de dados em tempo real
- Agentes especializados (title agent)

---

## 📦 O que está Incluído

- **Serviço FastAPI** que transmite respostas de um Agente OpenAI treinado em ferramentas de companhia aérea
- **Componente Web ChatKit** embutido em uma aplicação React com painel lateral rico em contexto
- **Ferramentas** para mudanças de assento, cancelamentos de viagem e atualizações de itinerário que sincronizam com a UI em tempo real
- **Sistema de anexos** para upload e análise de imagens

---

## ✅ Pré-requisitos

Antes de começar, você precisará de:
- **Python** 3.11 ou superior
- **Node.js** 20 ou superior  
- **[uv](https://docs.astral.sh/uv/getting-started/installation/)** (recomendado) ou `pip`
- **Chave da API OpenAI** exportada como `OPENAI_API_KEY`
- **Chave de domínio ChatKit** exportada como `VITE_SUPPORT_CHATKIT_API_DOMAIN_KEY` (use qualquer placeholder não-vazio durante desenvolvimento local; forneça a chave real em produção)

---

## 🚀 Início Rápido

### Visão Geral
1. Instale dependências e inicie o backend de suporte ao cliente
2. Configure a chave de domínio e inicie o frontend React
3. Explore os cenários de suporte de ponta a ponta

Cada passo é detalhado abaixo.

---

### Passo 1: Iniciar o Backend FastAPI

O backend para esta demo está em `examples/customer-support/backend` e vem com seu próprio `pyproject.toml`.

```bash
cd examples/customer-support/backend
uv sync                                    # Instala dependências Python
export OPENAI_API_KEY="sk-proj-..."       # Sua chave da API OpenAI
uv run uvicorn app.main:app --reload --port 8001  # Inicia o servidor
```

**O que acontece aqui?**
- O backend gerencia estado de reservas de voos por thread
- Processa ferramentas de mudança de assento, cancelamento, bagagem
- Gerencia upload e processamento de imagens

✅ A API expõe ChatKit em `http://127.0.0.1:8001/support/chatkit` e endpoints auxiliares em `/support/*`

---

### Passo 2: Executar o Frontend React

```bash
cd examples/customer-support/frontend
npm install      # Instala dependências JavaScript
npm run dev      # Inicia servidor de desenvolvimento
```

**O que acontece aqui?**
- Interface do chat com painel lateral de contexto
- Mostra itinerário, status de fidelidade e linha do tempo
- Permite upload de fotos de inspiração de viagem

✅ O servidor de desenvolvimento roda em `http://127.0.0.1:5171` e faz proxy de chamadas `/support` de volta para a API, o que cobre iteração local.

---

### Atalho: Executar tudo junto

Do diretório `examples/customer-support` você também pode executar:

```bash
npm start  # Inicia backend (uv sync + Uvicorn) e frontend juntos
```

**Certifique-se de que:**
- `uv` está instalado
- Variáveis de ambiente necessárias estão exportadas (`OPENAI_API_KEY` e `VITE_SUPPORT_CHATKIT_API_DOMAIN_KEY`)

---

### Sobre a chave pública de domínio

Você pode usar qualquer string durante desenvolvimento local. No entanto, para implantações em produção:

1. **Hospede o frontend** em infraestrutura que você controla atrás de um domínio gerenciado
2. **Registre esse domínio** na [página da lista de permissões de domínios](https://platform.openai.com/settings/organization/security/domain-allowlist) e espelhe-o em `examples/customer-support/frontend/vite.config.ts` em `server.allowedHosts`
3. **Defina** `VITE_SUPPORT_CHATKIT_API_DOMAIN_KEY` com a chave retornada pela página da lista de permissões e verifique se aparece em `examples/customer-support/frontend/src/lib/config.ts`

**Para testar cenários de acesso remoto antes do lançamento:**
- Exponha temporariamente a aplicação com um túnel—por exemplo, `ngrok http 5171` ou `cloudflared tunnel --url http://localhost:5171`
- Adicione esse hostname à lista de permissões primeiro

---

## 💬 Prompts de Exemplo

Abra a URL impressa e experimente prompts como:

### Gerenciamento de Reservas
- **"Você pode me mover para o assento 14C no voo OA476?"**
  - 🎯 Demonstra: Ferramenta `change_seat` + atualização de itinerário

- **"Adicione mais uma mala despachada à minha reserva."**
  - 🎯 Demonstra: Ferramenta `add_checked_bag` + atualização de perfil

### Cancelamentos e Reembolsos
- **"Preciso cancelar minha viagem e solicitar um reembolso."**
  - 🎯 Demonstra: Ferramenta `cancel_trip` + processamento de reembolso

### Seleção de Refeições
- **"Quero refeição vegetariana no meu voo."**
  - 🎯 Demonstra: Widget de preferências de refeição + ação de seleção

### Opções de Voo
- **"Mostre-me opções de voos para Paris na próxima semana."**
  - 🎯 Demonstra: Widget de opções de voo + ações de seleção

**💡 Observe:** O agente invoca as ferramentas apropriadas e a linha do tempo atualiza automaticamente no painel lateral.

---

## 🎓 Funcionalidades Explicadas

### 1. 📋 Gerenciamento de Estado por Thread

Cada conversa (thread) tem seu próprio estado isolado.

**Como funciona:**
```python
# backend/app/airline_state.py
class AirlineStateManager:
    def __init__(self, thread_id: str):
        self.thread_id = thread_id
        self.itinerary = load_itinerary(thread_id)
        self.loyalty_status = load_loyalty(thread_id)
        self.timeline = []
    
    def change_seat(self, flight_id, new_seat):
        # Atualiza estado
        self.itinerary[flight_id].seat = new_seat
        # Adiciona à linha do tempo
        self.timeline.append({
            "action": "seat_changed",
            "flight": flight_id,
            "new_seat": new_seat,
            "timestamp": now()
        })
```

**Benefícios:**
- Múltiplos clientes podem usar o sistema simultaneamente
- Cada conversa mantém seu próprio contexto
- Histórico completo de ações

---

### 2. 📊 Perfil do Cliente (Customer Profile)

Antes de cada execução do agente, um snapshot do perfil é adicionado.

**Estrutura do perfil:**
```xml
<CUSTOMER_PROFILE>
  <ITINERARY>
    <FLIGHT id="OA476">
      <FROM>São Paulo</FROM>
      <TO>Miami</TO>
      <DATE>2024-03-15</DATE>
      <SEAT>12A</SEAT>
      <MEAL>Standard</MEAL>
    </FLIGHT>
  </ITINERARY>
  
  <LOYALTY>
    <TIER>Gold</TIER>
    <POINTS>45000</POINTS>
  </LOYALTY>
  
  <TIMELINE>
    <EVENT timestamp="14:30">Reserva criada</EVENT>
    <EVENT timestamp="14:32">Assento mudado para 12A</EVENT>
  </TIMELINE>
</CUSTOMER_PROFILE>
```

**Por que é importante:**
- O agente sempre tem contexto completo
- Pode tomar decisões informadas
- Respostas são personalizadas ao cliente

---

### 3. 🎨 Widgets de Voo e Refeição

Widgets ricos que capturam escolhas antes da reserva.

#### Widget de Opções de Voo

```python
# backend/app/flight_options.py
def create_flight_widget(flights):
    return {
        "type": "flight-options",
        "flights": [
            {
                "id": "OA476",
                "departure": "07:00",
                "arrival": "15:30",
                "price": "$450",
                "action": {
                    "type": "flight.select",
                    "flight_id": "OA476"
                }
            },
            # Mais opções...
        ]
    }
```

**Fluxo de seleção:**
```
1. Agente mostra widget com 3 opções de voo
2. Usuário clica em um cartão de voo
3. Ação `flight.select` é enviada ao servidor
4. Servidor valida disponibilidade
5. Cria reserva
6. Atualiza perfil do cliente
7. Bloqueia widget (não pode escolher novamente)
8. Mostra confirmação
```

---

### 4. 📎 Sistema de Anexos de Imagem

Suporte completo para upload e análise de imagens.

**Arquitetura:**
```
┌─────────────┐     ┌─────────────┐     ┌──────────────┐
│  Frontend   │────>│   Backend   │────>│  OpenAI API  │
│  (Upload)   │     │  (Process)  │     │  (Analyze)   │
└─────────────┘     └─────────────┘     └──────────────┘
      │                    │                     │
      │ 1. Request URL     │                     │
      │<───────────────────│                     │
      │                    │                     │
      │ 2. Upload image    │                     │
      │───────────────────>│                     │
      │                    │                     │
      │                    │ 3. Convert to       │
      │                    │    data URL         │
      │                    │                     │
      │                    │ 4. Send to model    │
      │                    │────────────────────>│
      │                    │                     │
      │                    │ 5. Get description  │
      │                    │<────────────────────│
```

**Funcionalidades:**
- Upload seguro com URLs assinadas
- Validação de tipo de imagem e tamanho
- Conversão para data URLs para o modelo
- Análise visual com GPT-4 Vision

**Caso de uso:**
```
Você: [envia foto de praia tropical]
Você: "Quero viajar para um lugar assim"
Agente: [analisa imagem]
Agente: "Vejo uma linda praia tropical com águas cristalinas. 
         Recomendo nossos destinos no Caribe: Cancún, Aruba ou Bahamas. 
         Qual te interessa mais?"
```

---

### 5. ⚙️ Ferramentas de Companhia Aérea

Conjunto completo de ferramentas para gerenciar reservas.

#### Ferramenta: `change_seat`
```python
def change_seat(
    flight_id: str,
    new_seat: str,
    state: AirlineStateManager
) -> str:
    """Muda o assento do passageiro em um voo"""
    
    # Valida se assento está disponível
    if not is_seat_available(flight_id, new_seat):
        return "Assento não disponível"
    
    # Atualiza itinerário
    state.itinerary[flight_id].seat = new_seat
    
    # Registra na linha do tempo
    state.timeline.append({
        "action": "seat_changed",
        "flight": flight_id,
        "old_seat": old_seat,
        "new_seat": new_seat
    })
    
    # Emite efeito para atualizar UI
    emit_effect("customer_profile/update", state.to_dict())
    
    return f"Assento mudado para {new_seat} com sucesso!"
```

#### Outras ferramentas disponíveis:
- `cancel_trip` - Cancela viagem e processa reembolso
- `add_checked_bag` - Adiciona bagagem despachada
- `set_meal_preference` - Define preferência de refeição
- `surface_flight_options` - Busca opções de voo
- `request_assistance` - Solicita assistência especial

---

### 6. 🏷️ Agente de Título (Title Agent)

Agente leve que nomeia a conversa automaticamente.

**Como funciona:**
```python
# backend/app/title_agent.py
title_agent = Agent(
    instructions="""
    Gere um título curto (máximo 5 palavras) para esta conversa
    baseado na primeira mensagem do usuário.
    
    Exemplos:
    - "Preciso mudar meu voo" → "Mudança de Voo"
    - "Quero cancelar" → "Cancelamento de Reserva"
    - "Qual minha fidelidade?" → "Consulta de Status"
    """,
    model="gpt-3.5-turbo"  # Mais rápido e barato
)

# backend/app/server.py
if is_first_message:
    # Executa em paralelo para não atrasar resposta
    title = await title_agent.run(first_message)
    thread.title = title
```

**Benefícios:**
- Títulos automáticos descritivos
- Não atrasa a primeira resposta (paralelo)
- Usa modelo mais barato (GPT-3.5)

---

### 7. 🔄 Sincronização de Painel Lateral

O painel lateral sempre reflete o estado mais recente.

**Fluxo de atualização:**
```
1. Usuário muda assento via chat
2. Servidor processa com change_seat
3. Servidor emite efeito customer_profile/update
4. Frontend recebe efeito
5. Frontend atualiza painel lateral
6. Mudança é visível imediatamente
```

**Código do frontend:**
```typescript
// CustomerContextPanel.tsx
onEffect: (effect) => {
    if (effect.name === "customer_profile/update") {
        setItinerary(effect.data.itinerary)
        setLoyalty(effect.data.loyalty)
        setTimeline(effect.data.timeline)
    }
}
```

**💡 Vantagem:** Usuário sempre vê estado atual sem precisar perguntar "qual meu assento?"

---

## 🏗️ Arquitetura do Código

### Backend (Python)

```
backend/
├── app/
│   ├── main.py                 # FastAPI app, rotas
│   ├── server.py               # Lógica ChatKit
│   ├── support_agent.py        # Agente principal
│   ├── title_agent.py          # Agente de títulos
│   ├── airline_state.py        # Gerenciamento de estado
│   ├── attachment_store.py     # Upload/download de imagens
│   ├── thread_item_converter.py # Conversão de mensagens
│   ├── flight_options.py       # Widget de voos
│   └── meal_preferences.py     # Widget de refeições
```

### Frontend (React)

```
frontend/
├── src/
│   ├── App.tsx                      # Layout principal
│   ├── components/
│   │   ├── ChatKitPanel.tsx         # Componente do chat
│   │   ├── CustomerContextPanel.tsx # Painel lateral
│   │   ├── ItineraryView.tsx        # Visualização de itinerário
│   │   ├── LoyaltyStatus.tsx        # Status de fidelidade
│   │   └── TimelineView.tsx         # Linha do tempo
│   └── lib/
│       └── config.ts                # Configuração
```

---

## 💡 Dicas de Aprendizado

### Para Desenvolvedores Intermediários:

1. **Entenda o ciclo de vida do estado**
   - Como o estado é carregado ao criar thread
   - Como persiste entre mensagens
   - Como é serializado/deserializado

2. **Explore o sistema de anexos**
   - Como URLs assinadas funcionam
   - Como imagens são processadas
   - Como o modelo "vê" imagens

3. **Estude os widgets complexos**
   - Como ações são definidas
   - Como são tratadas no servidor
   - Como widgets são bloqueados após ação

### Exercícios Práticos:

#### Exercício 1: Nova ferramenta
Crie uma ferramenta para solicitar upgrade de classe:
```python
def request_upgrade(
    flight_id: str,
    new_class: str,  # "business" ou "first"
    state: AirlineStateManager
) -> str:
    # Calcule preço do upgrade
    # Verifique disponibilidade
    # Retorne widget de confirmação
    pass
```

#### Exercício 2: Novo item de linha do tempo
Adicione registro quando usuário faz check-in:
```python
state.timeline.append({
    "action": "checked_in",
    "flight": flight_id,
    "timestamp": now(),
    "gate": "B12"
})
```

#### Exercício 3: Widget personalizado
Crie um widget de seleção de assento visual (mapa do avião):
```python
def create_seat_map_widget(flight_id):
    return {
        "type": "seat-map",
        "layout": generate_seat_layout(),
        "available": get_available_seats(),
        "selected": current_seat
    }
```

---

## 🐛 Problemas Comuns

### Estado não persiste entre mensagens
**Causa:** Thread ID não está sendo mantido
**Solução:** Verifique se o frontend envia o mesmo `thread_id`

### Anexos não fazem upload
**Causa:** URLs assinadas expiraram
**Solução:** URLs têm tempo de vida curto (5 min). Faça upload imediatamente após obter URL

### Widget não atualiza após ação
**Causa:** Ação não está emitindo efeito de atualização
**Solução:** Adicione `emit_effect("customer_profile/update", ...)` após processar ação

---

## 🎯 Próximos Passos

Depois de dominar Customer Support:

1. **Explore News Guide**
   - Sistemas de busca e recuperação
   - @-mentions de entidades
   - Composer tool choice

2. **Desafie-se com Metro Map**
   - Visualizações complexas
   - Ferramentas do cliente
   - Anotações inline

3. **Construa seu próprio**
   - Use Customer Support como template
   - Adapte para seu domínio (hotel, restaurante, loja)
   - Implemente ferramentas específicas do seu negócio

---

## 📚 Recursos Relacionados

- [Documentação do ChatKit](https://platform.openai.com/docs/chatkit)
- [Guia de Integração](../../agents.md)
- [FastAPI - Uploads de Arquivo](https://fastapi.tiangolo.com/tutorial/request-files/)
- [OpenAI Vision API](https://platform.openai.com/docs/guides/vision)

---

**✈️ Pronto para oferecer suporte de classe mundial!** 🎫✨

O agente invoca as ferramentas apropriadas e a linha do tempo atualiza automaticamente no painel lateral.
