# Regras do projeto para o tech-lead

## O que ler
- `docs/ESPECIFICACAO.md` inteiro, com atenção às seções 3.1 (versões), 15 (fases) e 16 (critérios de aceite).
- `docs/PROGRESSO.md`. Se não existir, crie-o com o checklist das fases da seção 15.

## Particularidades da Maestria
- O `docker-compose.yml` (PostgreSQL 18 + Mailpit), o `.nvmrc` (Node 24) e o `.gitignore` **já existem** na raiz. A Fase 0 não deve recriá-los, só usá-los.
- Na Fase 0, copiar `assets/brand/` para `web/public/brand/` e `assets/placeholders/` para `web/public/placeholders/`.
- O envio de e-mail entra na **Fase 1**, porque o login do backoffice por código depende dele.
- O login Google do cliente entra na **Fase 2**. Antes disso, o usuário precisa criar o OAuth Client, então avise na hora de planejar essa fase.
- O WhatsApp (Cloud API da Meta) é **opcional** e só entra na **Fase 5**, atrás da flag `WHATSAPP_ENABLED`.
- Antes da Fase 1, confira se a cidade foi preenchida na seção 2 da especificação, porque o seed usa esse dado. Se ainda estiver `[CONFIRMAR]`, pergunte ao usuário.
- Contratos de API: use os módulos da seção 13 como base para os arquivos de `docs/contratos/`.
