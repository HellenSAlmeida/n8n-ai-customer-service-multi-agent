# n8n-ai-customer-service-multi-agent
Multi-agent AI customer service workflow built with N8N — 4 specialized AI agents, dynamic LLM fallback (OpenAI/Groq), PostgreSQL memory, and WhatsApp integration.

# 🤖 AI Customer Service — Multi-Agent Workflow (N8N)

> Atendimento 100% automatizado via WhatsApp com orquestração de múltiplos agentes de IA, memória persistente em PostgreSQL e roteamento dinâmico por etapa do cliente.

---

## 📌 Visão Geral

Este workflow implementa um sistema de atendimento ao cliente totalmente automatizado, construído no **N8N**, capaz de conduzir um cliente desde o primeiro contato até a localização de projetos existentes — sem intervenção humana.

A arquitetura utiliza **4 agentes de IA especializados** que atuam em sequência conforme o contexto da conversa, com **fallback automático entre modelos de linguagem** (OpenAI ↔ Groq) para garantir disponibilidade e otimização de custo.

---

## 🏗️ Arquitetura

```
WhatsApp (Webhook Inbound)
  │
  ├─► Validação de Headers + ACK 200 imediato
  │
  ├─► Switch: Tipo de Mensagem
  │     ├── Texto
  │     ├── Imagem       ──► OpenAI Vision (análise visual)
  │     ├── Áudio
  │     └── Mídia
  │
  ├─► Switch: Tipo de Evento
  │     └── Gate: IA Ativa? (StaticData Policy)
  │
  └─► Switch: Etapa do Projeto
        │
        ├─► [AGENTE 1] Atendimento Inicial
        │     Primeiro contato, coleta de contexto,
        │     classificação: novo cliente × cliente com projeto
        │
        ├─► [AGENTE 2] Qualificação
        │     Perguntas de qualificação, identificação de necessidade,
        │     decisão de transferência de etapa
        │
        ├─► [AGENTE 3] Localização de Projeto
        │     Consulta à API interna via Tools,
        │     seleção e confirmação do projeto correto
        │
        └─► [AGENTE 4] Atendimento com Visão
              Interpretação de imagens enviadas pelo cliente,
              resposta contextualizada
```

---

## 🤖 Agentes de IA

| Agente | Responsabilidade | Ferramentas |
|--------|-----------------|-------------|
| **Atendimento Inicial** | Acolhimento, coleta de contexto, classificação do intent | Memória Postgres |
| **Qualificação** | Perguntas de qualificação, scoring, decisão de etapa | Memória Postgres |
| **Localização de Projeto** | Consulta de projetos existentes via API | `lista_projetos`, `projeto_id`, `projeto_cliente`, Memória Postgres |
| **Atendimento Visual** | Interpretação de imagens (fotos de ambientes, referências) | OpenAI Vision, Memória Postgres |

---

## ⚙️ Stack Técnica

| Componente | Tecnologia |
|-----------|-----------|
| Orquestração | N8N (self-hosted) |
| LLM Principal | OpenAI GPT-4o |
| LLM Fallback | Groq (Llama 3) |
| Seleção de Modelo | N8N ModelSelector (dinâmico) |
| Memória Conversacional | PostgreSQL (por sessão) |
| Análise de Imagem | OpenAI Vision API |
| Canal de Entrada | WhatsApp (Webhook) |
| Autenticação | JWT via API interna |

### ModelSelector — Fallback Dinâmico
O workflow utiliza o nó `ModelSelector` do N8N para alternar automaticamente entre OpenAI e Groq com base em disponibilidade e configuração. Isso garante resiliência e permite otimização de custo por agente.

---

## 🔐 Segurança

- Validação de headers customizados (`X-App-Token`) em toda requisição inbound
- Autenticação JWT via login de colaborador antes de qualquer operação na API
- Gate de ativação de IA por `StaticData` — permite ligar/desligar o atendimento automatizado sem alterar o workflow
- Audit log de eventos: `chat.agent_assumed` e `chat.agent_released`
- Resposta imediata com ACK 200 (evita timeout do WhatsApp) + processamento assíncrono

---

## 🗄️ Banco de Dados

O workflow utiliza PostgreSQL para:
- **Memória conversacional** por sessão (cada agente tem seu próprio contexto isolado)
- **Histórico de atendimento** por etapa (Atendimento Inicial, Qualificação, Localização)
- **Busca da última etapa** para retomada de conversa sem perda de contexto

---

## 📦 Estrutura do Repositório

```
├── workflow/
│   └── ai-customer-service-multi-agent.json   # Workflow N8N (pronto para importar)
├── docs/
│   └── architecture.md                        # Detalhamento da arquitetura
└── README.md
```

---

## 🚀 Como Usar

### Pré-requisitos
- N8N v1.x (self-hosted ou cloud)
- PostgreSQL 14+
- Conta OpenAI com acesso à API (GPT-4o)
- Conta Groq (opcional, para fallback)
- Integração WhatsApp configurada (Evolution API ou similar)

### Setup

**1. Importar o workflow no N8N**
```
N8N → Workflows → Import → selecionar ai-customer-service-multi-agent.json
```

**2. Configurar credenciais**

| Credencial | Onde configurar |
|-----------|----------------|
| `OPENAI_API_KEY` | N8N Credentials → OpenAI |
| `GROQ_API_KEY` | N8N Credentials → Groq |
| `POSTGRES_*` | N8N Credentials → PostgreSQL |
| `YOUR_ADMIN_PASSWORD` | N8N Credentials → HTTP Basic Auth |

**3. Configurar variáveis de ambiente**
```env
INTERNAL_API_BASE_URL=http://your-internal-api:3000/api
APP_BASE_URL=https://your-app.example.com
WEBHOOK_TOKEN=your-secure-token
```

**4. Ativar o workflow**

Após configurar todas as credenciais, ativar o workflow no N8N. O webhook estará disponível em:
```
POST https://your-n8n-instance.com/webhook/webhook-inbound
```

---

## 📊 Resultados (Caso Real)

> Métricas obtidas em produção com volume real de atendimento:

- ✅ **100% dos atendimentos** conduzidos por IA sem intervenção humana
- ⚡ Tempo de resposta médio: **< 8 segundos**
- 🎯 Taxa de qualificação automática: **78% dos leads**
- 📉 Redução do TMA (Tempo Médio de Atendimento): de **20 min → 10 min**

---

## 🧩 Extensibilidade

O workflow foi desenhado para ser modular. Novos agentes podem ser adicionados criando:
1. Um novo `Switch` de roteamento na etapa correspondente
2. Um nó `Agent` com system prompt específico
3. Conexão com `Postgres Chat Memory` isolado
4. Integração com o fluxo de `Audit Log` existente

---

## 👤 Autor

**Hellen Santos**
Engenheira de Automação & AI | HS Technology

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Hellen%20Santos-blue?logo=linkedin)](https://linkedin.com/in/seu-perfil)
[![GitHub](https://img.shields.io/badge/GitHub-hellensantos-black?logo=github)](https://github.com/hellensantos)

---

*Este repositório contém uma versão sanitizada do workflow — todas as credenciais, URLs e dados de clientes foram substituídos por placeholders genéricos.*
