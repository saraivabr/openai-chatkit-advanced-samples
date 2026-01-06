# ChatKit Frontend

Este cliente Vite + React envolve o componente web ChatKit em uma UI de lista simples para que você possa focar em iterar com o agente backend. Ele espelha o tom do README raiz enquanto apresenta os caminhos de projeto e configuração que você precisa no dia a dia.

## 📚 Referência Rápida

- **Ponto de entrada da aplicação**: `src/main.tsx`
- **Helper de configuração do ChatKit**: `src/lib/config.ts`
- **UI do dashboard do gato**: `src/App.tsx` e `src/components`
- **Estilização**: `src/index.css` (camadas Tailwind)

## ✅ Requisitos

- **Node.js** 20 ou superior
- **API Backend** rodando localmente (padrão: `http://127.0.0.1:8000`)

## 🔧 Variáveis de Ambiente

Substituições opcionais incluem:
- `VITE_CHATKIT_API_URL` - URL da API do ChatKit
- `VITE_CAT_STATE_API_URL` - URL da API de estado do gato
- `VITE_CHATKIT_API_DOMAIN_KEY` - Chave de domínio do ChatKit

**⚠️ Importante:** Se você mudá-las, reinicie `npm run dev` para que o Vite recarregue os novos valores.

## 🚀 Instalar e Executar

```bash
npm install    # Instala dependências
npm run dev    # Inicia servidor de desenvolvimento
```

✅ O servidor de desenvolvimento está disponível em `http://127.0.0.1:5170`, que funciona para desenvolvimento local.

### Testando Acesso Remoto

Para testar fluxos de acesso remoto, você pode expor temporariamente a aplicação com um túnel (por exemplo `ngrok http 5170`) após adicionar esse hostname à lista de permissões.

### Implantação em Produção

Para implantações em produção:

1. **Hospede a aplicação** em infraestrutura que você controla atrás de um domínio gerenciado
2. **Registre esse domínio** na [página da lista de permissões de domínios](https://platform.openai.com/settings/organization/security/domain-allowlist)
3. **Adicione-o** em `frontend/vite.config.ts` em `server.allowedHosts`
4. **Defina a chave resultante** via `VITE_CHATKIT_API_DOMAIN_KEY`

## 🤔 Precisa de ajuda com o backend?

Veja o README raiz para passos de configuração do FastAPI e lista de permissões de domínio.

---

**💡 Dica:** Este é apenas o frontend. O "cérebro" da aplicação (agente de IA, ferramentas, lógica) está no backend!
