# Guia de Integração do ChatKit

Este documento explica como personalizar o template inicial do ChatKit que está neste repositório. Ele abrange o wrapper do cliente React, a integração do lado do servidor que alimenta as respostas do agente e o sistema de widgets/ações que permite UIs mais ricas. O objetivo é manter tudo que você precisa em um só lugar para que você possa desenvolver rapidamente sem precisar caçar comentários no código.

---

## 📚 Referência Rápida

- **Ponto de entrada do Frontend**: `frontend/src/main.tsx`
- **Helper de configuração do ChatKit**: `frontend/src/lib/config.ts`
- **Ponto de entrada do FastAPI**: `backend/app/main.py`

---

## ✅ Pré-requisitos

Antes de começar, você precisará de:
- **Node.js** 20 ou superior
- **Python** 3.11 ou superior
- **[uv](https://docs.astral.sh/uv/getting-started/installation/)** (recomendado) ou `pip`
- **Chave da API OpenAI** exportada como `OPENAI_API_KEY`
- **Chave de domínio ChatKit** exportada como `VITE_CHATKIT_API_DOMAIN_KEY` (qualquer placeholder não-vazio durante desenvolvimento local; use a chave real da lista de permissões em produção)

---

## 🚀 Configuração do Projeto Local

### Passo a passo didático:

#### 1. **Configurar o Backend**

O backend é a parte que processa as requisições e se comunica com a API da OpenAI.

```bash
cd backend
uv sync                                  # Instala as dependências Python
export OPENAI_API_KEY="sk-proj-..."     # Sua chave da API OpenAI
uv run uvicorn app.main:app --reload --port 8000  # Inicia o servidor
```

**O que acontece aqui?**
- `uv sync`: Instala todas as bibliotecas Python necessárias
- `export OPENAI_API_KEY`: Define sua chave de acesso à OpenAI
- `uvicorn`: Inicia o servidor web FastAPI na porta 8000

✅ A API estará disponível em `http://127.0.0.1:8000`

**💡 Dica:** Se você não tem uma chave da API, obtenha em [platform.openai.com/api-keys](https://platform.openai.com/api-keys)

#### 2. **Configurar o Frontend**

O frontend é a interface visual que o usuário vê e interage.

```bash
cd frontend
npm install      # Instala as dependências JavaScript
npm run dev      # Inicia o servidor de desenvolvimento
```

**O que acontece aqui?**
- `npm install`: Baixa todas as bibliotecas JavaScript necessárias (React, Vite, ChatKit, etc.)
- `npm run dev`: Inicia o servidor de desenvolvimento Vite com hot-reload (atualização automática)

✅ O servidor Vite estará disponível em `http://127.0.0.1:5170`

**💡 Dica:** Com o hot-reload, toda vez que você editar um arquivo, a página atualiza automaticamente!

#### 3. **Configurar Lista de Permissões de Domínio**

O ChatKit usa uma lista de permissões de domínios para segurança.

**Para desenvolvimento local:**
```bash
export VITE_CHATKIT_API_DOMAIN_KEY=domain_pk_local_dev
```

**Para produção:**
1. Registre seu domínio de desenvolvimento ou produção em [platform.openai.com/settings/organization/security/domain-allowlist](https://platform.openai.com/settings/organization/security/domain-allowlist)
2. Substitua o placeholder com o valor `domain_pk_...` gerado
3. Adicione o domínio em `frontend/vite.config.ts` na configuração `server.allowedHosts`

**💡 Por que isso é necessário?**
A lista de permissões garante que apenas domínios autorizados possam usar sua integração ChatKit, prevenindo uso não autorizado.

---

## 🏗️ Arquitetura do Projeto

### Como tudo se conecta:

```
┌─────────────────┐         ┌──────────────────┐         ┌──────────────┐
│   Navegador     │ ◄─────► │  FastAPI Backend │ ◄─────► │  OpenAI API  │
│  (React + UI)   │  HTTP   │  (Python Logic)  │  HTTP   │   (GPT-4)    │
└─────────────────┘         └──────────────────┘         └──────────────┘
     │                              │
     │ ChatKit.js                  │ ChatKit Python SDK
     │                              │
     └──────────── Comunicação ────┘
          via WebSocket/HTTP
```

**Fluxo de uma mensagem:**
1. **Usuário** digita mensagem no chat (Frontend)
2. **ChatKit.js** envia para o Backend
3. **Backend Python** processa com ferramentas e contexto
4. **Backend** chama API da OpenAI
5. **OpenAI** retorna resposta do agente
6. **Backend** formata e envia de volta
7. **Frontend** exibe a resposta ao usuário

---

## 📁 Estrutura de Diretórios Explicada

```
project/
├── frontend/                    # Aplicação React
│   ├── src/
│   │   ├── main.tsx            # Ponto de entrada (começa aqui!)
│   │   ├── App.tsx             # Componente principal da aplicação
│   │   ├── components/         # Componentes React reutilizáveis
│   │   │   └── ChatKitPanel.tsx  # Componente do chat ChatKit
│   │   └── lib/
│   │       └── config.ts       # Configurações do ChatKit
│   ├── package.json            # Dependências do Node.js
│   └── vite.config.ts          # Configuração do Vite
│
└── backend/                     # API FastAPI
    ├── app/
    │   ├── main.py             # Ponto de entrada da API
    │   ├── server.py           # Lógica do servidor ChatKit
    │   └── agents/             # Agentes de IA especializados
    │       └── *_agent.py      # Definições de agentes
    └── pyproject.toml          # Dependências Python
```

**💡 Para entender melhor:**
- `frontend/`: Todo código que roda no navegador do usuário
- `backend/`: Todo código que roda no servidor
- `agents/`: Cérebros especializados para diferentes tarefas

---

## 🎓 Conceitos Fundamentais

### O que é um Agente?

Um **agente** é como um assistente especializado com:
- **Instruções** (system prompt): O que ele deve fazer
- **Ferramentas** (tools): O que ele pode fazer
- **Contexto** (context): O que ele sabe

**Exemplo simples:**
```python
# Um agente que ajuda com matemática
agente_matematica = Agent(
    instructions="Você é um professor de matemática paciente.",
    tools=[calculadora, graficos],
    model="gpt-4"
)
```

### O que são Ferramentas (Tools)?

**Ferramentas** são funções que o agente pode chamar para:
- Buscar dados do banco de dados
- Fazer cálculos
- Atualizar informações
- Interagir com APIs externas

**Exemplo:**
```python
def buscar_preco_produto(produto_id: str) -> float:
    """Busca o preço atual de um produto"""
    return database.query(f"SELECT preco FROM produtos WHERE id = {produto_id}")
```

O agente decide **quando** e **como** usar cada ferramenta baseado na conversa.

### O que são Widgets?

**Widgets** são componentes visuais ricos que aparecem no chat:
- Cartões informativos
- Botões clicáveis
- Listas interativas
- Formulários

**Por exemplo:** Em vez de apenas texto dizendo "Aqui estão 3 opções de voo", o agente pode mostrar um widget com cartões bonitos de cada voo com botões "Selecionar".

---

## 🔧 Personalizando Seu Projeto

### Mudando as Instruções do Agente

Abra o arquivo do agente (por exemplo, `backend/app/agents/cat_agent.py`):

```python
# Instruções originais
instructions = "Você é um cuidador de gatos virtual."

# Personalize para seu caso de uso
instructions = """
Você é um consultor financeiro especializado em investimentos.
Sempre explique conceitos complexos de forma simples.
Use exemplos práticos do dia a dia.
"""
```

### Adicionando Novas Ferramentas

```python
from typing_extensions import Annotated

def minha_nova_ferramenta(
    parametro: Annotated[str, "Descrição do parâmetro"]
) -> str:
    """Descrição clara do que a ferramenta faz"""
    # Sua lógica aqui
    return resultado

# Registre a ferramenta no agente
agente.add_tool(minha_nova_ferramenta)
```

### Criando um Novo Widget

```python
def criar_widget_personalizado():
    return {
        "type": "card",
        "title": "Título do Widget",
        "description": "Descrição detalhada",
        "actions": [
            {
                "type": "button",
                "label": "Clique Aqui",
                "action": "minha_acao"
            }
        ]
    }
```

---

## 🐛 Solução de Problemas Comuns

### Erro: "OPENAI_API_KEY not found"
**Solução:** Exporte a variável de ambiente:
```bash
export OPENAI_API_KEY="sua-chave-aqui"
```

### Erro: "Port 8000 already in use"
**Solução:** Outro processo está usando a porta. Mate o processo ou use outra porta:
```bash
uv run uvicorn app.main:app --reload --port 8001
```

### Frontend não conecta ao Backend
**Solução:** Verifique se:
1. O backend está rodando (`http://127.0.0.1:8000`)
2. O proxy está configurado corretamente em `vite.config.ts`
3. Não há erro de CORS (o backend deve permitir requisições do frontend)

### Mudanças no código não aparecem
**Frontend:** O hot-reload deve funcionar automaticamente. Se não, reinicie com `Ctrl+C` e `npm run dev`

**Backend:** Use `--reload` no uvicorn para auto-reload:
```bash
uv run uvicorn app.main:app --reload
```

---

## 📚 Recursos Adicionais

### Documentação Oficial
- [ChatKit Documentation](https://platform.openai.com/docs/chatkit)
- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [React Documentation](https://react.dev/)

### Próximos Passos

1. **Inicie com o exemplo mais simples** (Cat Lounge)
2. **Experimente modificar as instruções** do agente
3. **Adicione uma ferramenta simples** para entender o fluxo
4. **Explore os exemplos mais avançados** quando se sentir confortável

---

## 💡 Dicas de Aprendizado

### Para Iniciantes:
- Comece lendo o código de `cat_agent.py` - é o mais simples
- Execute o Cat Lounge e veja como ele funciona
- Faça pequenas mudanças e veja o resultado
- Use `console.log()` no frontend e `print()` no backend para debug

### Para Desenvolvedores Intermediários:
- Estude como os widgets são criados e renderizados
- Entenda o fluxo de ferramentas do servidor vs cliente
- Implemente uma ferramenta personalizada do zero
- Explore os diferentes tipos de efeitos do cliente

### Para Desenvolvedores Avançados:
- Implemente agentes especializados múltiplos
- Crie widgets complexos com estado
- Otimize streaming de respostas
- Integre com seus próprios sistemas de banco de dados

---

## ❓ Perguntas Frequentes

**P: Preciso pagar para usar a API da OpenAI?**
R: Sim, mas há créditos gratuitos iniciais. Veja [pricing](https://openai.com/pricing).

**P: Posso usar modelos diferentes do GPT-4?**
R: Sim! Você pode usar GPT-3.5, GPT-4, ou outros modelos compatíveis. Basta mudar o parâmetro `model`.

**P: Como faço deploy em produção?**
R: Você precisará:
1. Um servidor para o backend (ex: Railway, Render, AWS)
2. Um host para o frontend (ex: Vercel, Netlify)
3. Configurar variáveis de ambiente em produção
4. Registrar seu domínio na lista de permissões

**P: Posso usar outro framework além do React?**
R: Sim! O ChatKit.js funciona com Vue, Angular, ou JavaScript puro.

---

**🎉 Pronto para começar?** Execute `npm run cat-lounge` da raiz do repositório e comece a explorar!
