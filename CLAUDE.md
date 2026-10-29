# CLAUDE.md — Maestria Barbearia

Lido automaticamente pelo Claude Code em toda sessão. Mantido curto de propósito: os detalhes ficam em `docs/`.

## O projeto
Site de divulgação, agendamento online (login Google) e backoffice (login por código) para **uma única barbearia**.
Fluxo de agendamento inspirado no Booksy, porém simples. **Todo o sistema é responsivo (mobile first).**
Especificação completa: `docs/ESPECIFICACAO.md`.

## Quem é o dev
Sênior em PHP/Laravel e Vue, **iniciante em NestJS e Next.js**. O projeto também é uma ferramenta de aprendizado: código simples, comentários didáticos em português e pontes com Laravel/Vue.

## Estrutura
```
api/                # NestJS 12 + Prisma 7 + PostgreSQL 18
web/                # Next.js 16 + Tailwind 4 + shadcn/ui
assets/             # logos e imagens provisórias (copiar para web/public na Fase 0)
docs/
├── ESPECIFICACAO.md
├── agentes/        # regras deste projeto para cada agente
├── contratos/      # contratos de API por módulo (criados pelo tech-lead)
├── aprendizado/    # notas didáticas por fase
├── PROGRESSO.md    # checklist das fases
└── DECISOES.md     # decisões fora da especificação
.claude/agents      # link para a pasta maestria/ do repositório de agentes (não versionado)
```

## Regras dos agentes neste projeto
Os agentes são da stack e não conhecem a barbearia. **Cada agente deve ler o seu arquivo abaixo antes de trabalhar.** As regras do projeto vencem as do agente em caso de conflito.

| Agente | Arquivo de regras do projeto |
|---|---|
| `tech-lead` | `docs/agentes/tech-lead.md` |
| `backend-nest` | `docs/agentes/backend.md` |
| `frontend-next` | `docs/agentes/frontend.md` |
| `ui-designer` | `docs/agentes/ui.md` |
| `qa-tester` | `docs/agentes/qa.md` |
| `mentor` | `docs/agentes/mentor.md` |

Fluxo por tarefa: o `tech-lead` planeja; `backend-nest`, `frontend-next` e `ui-designer` implementam; o `qa-tester` valida; o `mentor` revisa e escreve a nota de aprendizado.

## Regras de ouro (valem para todos)
1. **Simples primeiro:** caminho padrão da documentação oficial, sem abstrações desnecessárias nem bibliotecas fora da stack sem justificativa.
2. **O backend nunca confia no frontend:** disponibilidade, permissões, preços e elegibilidade de avaliação são validados na API.
3. **Datas** em UTC no banco; conversão para `America/Sao_Paulo` só em `api/src/common/time.ts`.
4. **Dinheiro em centavos.**
5. **Tema escuro é o padrão**, com o claro em todas as telas.
6. **Responsivo:** sem rolagem horizontal, testado em 375px, 768px e 1280px.
7. **Dados da barbearia** vêm do seed/API, nunca fixos no frontend.
8. **Uma fase por vez.** Ao terminar uma tarefa: lint, testes e `docs/PROGRESSO.md`. Ao fechar uma fase: `docs/aprendizado/fase-N.md`.
9. Código em inglês, interface e comentários em português.

## Versões
Node 24 LTS · NestJS 12 (ESM, Vitest, oxlint) · Prisma 7 (fixado) · PostgreSQL 18 · Next.js 16 · Tailwind 4. Detalhes na seção 3.1 da especificação.

⚠️ **Nunca rode `npx prisma` sem versão:** isso instala o CLI do Prisma 8 (RC), que não funciona aqui. Use os scripts `npm run db:*` ou `npx prisma@7`.

## Comandos
```bash
nvm use                       # Node do .nvmrc
docker compose up -d          # PostgreSQL + Mailpit (e-mails em http://localhost:8025)
cd api && npm run start:dev   # API em http://localhost:3333/api
cd api && npm run db:migrate  # migrations (Prisma 7)
cd api && npm run db:seed     # dados de exemplo
cd api && npm run db:studio   # ver o banco no navegador
cd api && npm test            # testes (Vitest)
cd web && npm run dev         # site em http://localhost:3000
```
