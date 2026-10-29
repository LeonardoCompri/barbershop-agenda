# Regras do projeto para o backend-nest

## O que ler na especificação
Seções 3 (stack e versões), 5 (perfis e autenticação), 6 (modelo de dados), 7 (agendamento), 8 (notificações), 9 (avaliações), 13 (API) e 14 (variáveis de ambiente). Na Fase 1, também a seção 2 (dados do seed).

## Bibliotecas extras deste projeto
Além da stack base do agente: `google-auth-library`, `@nestjs/throttler`, `nodemailer` e WhatsApp Cloud API via `fetch` nativo (sem SDK). **Não há senhas no sistema**, então não instale bcrypt.

## Autenticação: duas sessões separadas
- `customer_token` (30 dias) → `CustomerAuthGuard` → rotas `/customer/*`.
- `staff_token` (12 h) → `StaffAuthGuard` + `RolesGuard` com `@Roles('ADMIN')` → rotas `/admin/*` e `/me/*`.
- Cookies httpOnly, `sameSite=lax`, `secure` em produção. CORS liberado só para `WEB_URL`.

### Login Google do cliente (seção 5.1)
- `verifyIdToken` com `audience = GOOGLE_CLIENT_ID`. Recusar se `email_verified` for falso.
- Upsert do `Customer` por `googleId`. O telefone é obrigatório antes do primeiro agendamento.
- Criar `docs/google-login.md` com o passo a passo do Google Cloud Console.

### Login do backoffice por código (seção 5.2), seguir à risca
- 6 dígitos com `crypto.randomInt`, guardando **apenas o hash sha256**.
- Validade de 10 min, uso único, máximo de 5 tentativas. Um novo pedido invalida os anteriores.
- Até 1 envio a cada 60 s e 5 por hora por usuário, mais o `@nestjs/throttler` por IP.
- `request-code` **sempre** responde 200 com a mesma mensagem. Usuário inativo não recebe código.
- Em `NODE_ENV=development`, imprimir o código no log. O e-mail cai no Mailpit.
- Expor para o frontend se o canal WhatsApp está ativo (`WHATSAPP_ENABLED`).

## Seed (seção 2)
- Endereço e WhatsApp reais; serviços, profissionais e e-mail fictícios.
- Horário de funcionamento de exemplo: ter–sex 09:00–20:00, sáb 08:00–18:00, dom/seg fechado. Profissionais com almoço de 12:00 a 13:00.
- 6 avaliações `isSample = true`, `APPROVED`. Imagens da galeria apontando para `/placeholders/galeria-N.svg` com `isSample = true`.
- ADMIN criado a partir de `ADMIN_NAME`, `ADMIN_EMAIL` e `ADMIN_PHONE`.
- O seed é idempotente (pode rodar mais de uma vez sem duplicar).

## Disponibilidade (seção 7.2), a parte mais importante
- Fica em `src/availability/availability.service.ts`, com funções pequenas e puras (`generateSlots`, `overlaps`, `clipToBusinessHours`, `removeBusySlots`…).
- Ordem: `ClosedDay`/dia fechado → `WorkingHour` **recortado** pelo `BusinessHour` → slots a cada `slotIntervalMinutes` → cabe a duração? → conflito com `Appointment CONFIRMED` → conflito com `TimeOff` → antecedência mínima → limite de dias.
- "Sem preferência" (`professionalId=any`) junta os horários dos aptos. Ao confirmar, escolhe o profissional com menos agendamentos no dia (empate → `displayOrder`).
- Fuso `America/Sao_Paulo` só via `src/common/time.ts`.

## Criação de agendamento (seção 7.3)
- Recalcular a disponibilidade, `prisma.$transaction` com isolamento `Serializable`, checar o conflito de novo → **409** se o horário foi ocupado.
- Máximo de 3 agendamentos futuros ativos por cliente.
- Copiar nome, telefone e preço para o agendamento.
- Cancelamento só até `cancelLimitHours` antes.
- Ao salvar `WorkingHour` fora do horário de funcionamento → 400.

## Avaliações (seção 9)
- Só o cliente com atendimento `COMPLETED` dele, uma vez por atendimento (`appointmentId` único). Toda avaliação nova nasce `PENDING`.
- `/public/reviews` lista só `APPROVED` e calcula média e total só com elas.
- O admin aprova, apaga uma e apaga todas com `isSample`.

## Notificações (seção 8)
- `NotificationsService` com um método por evento (`sendLoginCode`, `appointmentCreated`, `appointmentCancelled`, `reviewSubmitted`) e os canais em `email.sender.ts` e `whatsapp.sender.ts`.
- Os templates de e-mail usam o logo e a paleta monocromática.
- Na Fase 5, documentar o WhatsApp em `docs/whatsapp.md` (templates de Autenticação e de Utilidade).
