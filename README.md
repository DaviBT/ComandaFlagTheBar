# Comanda Digital

Sistema de comandas digitais para bares, baseado em QR code. Cada comanda física recebe um QR code único: ao ser escaneado, o cliente acessa o cardápio digital do bar, faz o pedido pelo próprio celular, e o pedido é automaticamente vinculado à comanda — sem intervenção manual.

> **Status:** em desenvolvimento (pré-MVP). Nenhum bar real está usando o sistema ainda.

## Sobre o projeto

Ao fechar a conta no caixa, os pedidos daquela comanda são zerados. Se a comanda física for perdida durante o uso, os pedidos continuam salvos em banco de dados — esse é o principal diferencial do produto. Cozinha e garçons visualizam os pedidos em tempo real.

Projeto conduzido por dois desenvolvedores, com foco inicial em bares de uma única cidade.

## Funcionalidades

**MVP (em construção):**
- [ ] Cardápio digital acessível via QR code por comanda
- [ ] Pedido do cliente cai automaticamente na fila da cozinha
- [ ] Fechamento de comanda pelo caixa (zera os pedidos, calcula o total)
- [ ] Persistência dos pedidos mesmo com a comanda física perdida
- [ ] Painel administrativo para o dono editar o cardápio

**Fora do escopo do MVP** (planejado para depois):
- Pagamento integrado
- Emissão de nota fiscal (NFC-e)
- Aplicativo dedicado para garçons
- Relatórios avançados
- Integração com plataformas de delivery

## Arquitetura

```
Cliente (celular)               →  escaneia QR, acessa o cardápio (PWA)
        │
        ▼
API backend                     →  recebe e processa pedidos
        │
        ├──▶ Banco de dados     →  persiste os pedidos (Postgres)
        ├──▶ Painel da cozinha  →  recebe pedidos em tempo real
        └──▶ Painel do caixa    →  fecha a comanda
```

Todas as tabelas relevantes carregam uma coluna `bar_id` — o sistema é multi-tenant desde o início, mesmo operando com um único bar no começo.

## Stack tecnológica

| Camada | Tecnologia |
|---|---|
| Banco de dados | PostgreSQL (via Supabase ou Neon) |
| Backend | Node.js + TypeScript (Fastify/NestJS) ou Next.js API routes |
| Frontend (cliente) | React/Next.js como PWA |
| Tempo real | Supabase Realtime ou WebSockets (Socket.io) |
| Hospedagem backend | Railway ou Render |
| Hospedagem frontend | Vercel |
| Monitoramento | Sentry + UptimeRobot |
| CI/CD | GitHub Actions |

## Estrutura de pastas (proposta)

```
comanda-digital/
├── apps/
│   ├── web/              # PWA do cliente (cardápio + pedido)
│   ├── admin/             # painel do dono (cardápio, itens)
│   ├── kitchen/            # painel da cozinha (KDS)
│   └── cashier/            # painel do caixa (fechamento)
├── backend/
│   ├── src/
│   │   ├── routes/         # endpoints da API
│   │   ├── db/             # schema e migrations
│   │   └── realtime/       # canal de tempo real
│   └── docker-compose.yml
├── docs/
│   └── Plano_Projeto_Comanda_Digital.docx
├── DECISOES.md             # decisões técnicas registradas
└── README.md
```

## Como rodar localmente

Pré-requisitos: Node.js, Docker e Docker Compose instalados.

```bash
# 1. Clonar o repositório
git clone <url-do-repositorio>
cd comanda-digital

# 2. Subir o banco de dados local
docker compose up -d

# 3. Instalar dependências
npm install

# 4. Configurar variáveis de ambiente
cp .env.example .env
# preencher com as credenciais locais/de desenvolvimento

# 5. Rodar as migrations e popular dados de teste
npm run migrate
npm run seed

# 6. Iniciar o backend e o frontend
npm run dev
```

> Nunca commitar o arquivo `.env` — segredos e credenciais ficam fora do controle de versão em qualquer ambiente (local, staging ou produção).

## Modelo de dados (resumo)

| Tabela | Descrição |
|---|---|
| `bares` | Tabela raiz (tenant); toda tabela abaixo referencia `bar_id` |
| `comandas` | Uma linha por comanda física ativa (`status`: aberta/fechada) |
| `itens_cardapio` | Itens editáveis pelo dono via painel administrativo |
| `pedidos` | Vinculado à comanda (`status`: pendente/preparando/entregue) |
| `pedido_itens` | Detalhe de cada item dentro de um pedido |
| `usuarios` | Dono, garçom ou cozinha — controla o que cada um acessa |

## Roadmap

O roadmap completo, dividido em fases (preparação → fundamentos técnicos → MVP → preparação para produção → lançamento com o primeiro bar → expansão), está documentado em [`docs/Plano_Projeto_Comanda_Digital.docx`](./docs/Plano_Projeto_Comanda_Digital.docx).

Ordem geral adotada: o sistema é construído e testado por completo **antes** de qualquer contato comercial com bares.

## Time

Dois desenvolvedores, ambos com formação em desenvolvimento de sistemas (TI), conciliando o projeto com faculdade e trabalho.
