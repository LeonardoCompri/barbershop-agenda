# Maestria Barbearia — Especificação do Sistema

> Documento de referência do projeto. Todo agente deve ler este arquivo antes de implementar qualquer coisa.
> Instagram da barbearia: https://www.instagram.com/maestria.barbearia/

---

## 1. Visão geral

Sistema **100% responsivo** para **uma única barbearia** (não é marketplace), com três partes:

1. **Site público**: divulgação da marca, com apresentação, serviços, equipe, galeria, avaliações e localização.
2. **Agendamento online**: o cliente escolhe serviço, profissional, dia e horário, com validação de disponibilidade. Para confirmar, **entra com a conta Google**. O profissional é avisado por e-mail e/ou WhatsApp.
3. **Backoffice**: a barbearia configura horário de funcionamento, serviços e profissionais e modera avaliações. Cada profissional gerencia seus dados, horários e folgas. Há um painel com números de atendimentos. O **acesso é sem senha**: a pessoa recebe um código por e-mail ou WhatsApp.

O fluxo de agendamento é inspirado no **Booksy**, mas propositalmente mais simples.

### Fora do escopo (por enquanto)
- Pagamento online / sinal
- Login do cliente por outro meio além do Google
- Várias unidades / várias barbearias
- App mobile nativo (o site responsivo cobre o celular)
- Programa de fidelidade, pacotes, assinaturas
- Lembretes automáticos (ficam para a fase de melhorias)

---

## 2. Dados da barbearia

Itens marcados com **(fictício)** são provisórios e editáveis depois pela tela de Configurações. Entram no seed.

| Campo | Valor |
|---|---|
| Nome | Maestria Barbearia |
| Slogan | "Precisão em cada detalhe" **(fictício)** |
| Sobre | Texto curto sobre tradição + estilo moderno **(fictício)** |
| Endereço | Rua Porfirio Marques de Andrade, 123 - Jd. Imperador |
| Cidade/UF | Araraquara/SP |
| WhatsApp | +55 16 99605-8945 (gravar em E.164: `+5516996058945`) |
| E-mail | contato@maestriabarbearia.com.br **(fictício)** |
| Instagram | @maestria.barbearia |
| Horário de funcionamento | **Configurado no backoffice.** Seed de exemplo: ter–sex 09:00–20:00, sáb 08:00–18:00, dom e seg fechado |
| Duração e preço dos serviços | **Configurados no backoffice.** Seed de exemplo na tabela abaixo |
| Logo | `assets/brand/` (ver seção 10.3) |
| Galeria | Imagens provisórias em `assets/placeholders/galeria-1..6.svg`; o admin troca pelo backoffice |
| Fotos dos profissionais | Provisória `assets/placeholders/profissional.svg` |
| Imagem do hero | Provisória `assets/placeholders/hero.svg` |
| Avaliações | 6 avaliações de exemplo no seed, marcadas como `isSample = true` (ver seção 9) |

### Serviços de exemplo (fictício)
| Serviço | Duração | Preço |
|---|---|---|
| Corte | 40 min | R$ 45,00 |
| Barba | 30 min | R$ 35,00 |
| Corte + Barba | 70 min | R$ 70,00 |
| Pezinho / acabamento | 15 min | R$ 15,00 |
| Sobrancelha | 15 min | R$ 15,00 |
| Pigmentação de barba | 30 min | R$ 40,00 |
| Platinado | 120 min | R$ 150,00 |

### Profissionais de exemplo (fictício)
Rafael Souza, Bruno Lima e Diego Martins. Todos fazem todos os serviços, com os horários de funcionamento do seed e intervalo de almoço de 12:00 a 13:00.

### Acesso inicial ao backoffice
O seed cria o usuário ADMIN a partir das variáveis `ADMIN_NAME`, `ADMIN_EMAIL` e `ADMIN_PHONE` do `.env`.

---

## 3. Stack

### 3.1 Versões (verificadas em 27/10/2026)

| Item | Versão | Observação |
|---|---|---|
| Node.js | **24 LTS** (arquivo `.nvmrc` na raiz) | O Node 26 está virando LTS agora. Ficamos no 24 por compatibilidade |
| NestJS | **12.x** | Projeto novo nasce em **ESM**, com **Vitest** (testes) e **oxlint** (lint) |
| Prisma ORM | **7.x, fixado** (`prisma@^7`, `@prisma/client@^7`) | O Prisma 8 está em RC e `npx prisma` sem versão instala o CLI 8, que **não lê o `schema.prisma`**. Nunca rode `npx prisma` solto |
| PostgreSQL | **18** (`postgres:18-alpine`) | O 19 só sai em 19/11/2026 |
| Next.js | **16.x** | React 19 e Turbopack por padrão. O antigo `middleware.ts` agora se chama **`proxy.ts`** |
| Tailwind CSS | **4.x** | Configuração feita no CSS (`@theme`), sem `tailwind.config.js` |
| shadcn/ui | versão mais recente do CLI (`npx shadcn@latest`) | Compatível com Tailwind 4 e React 19 |
| Mailpit | `axllent/mailpit:latest` | Só em desenvolvimento |

Antes de instalar algo, o agente confere a versão com `npm view <pacote> version`. Se uma major nova tiver saído, ele pergunta antes de trocar.

### 3.2 Backend (`api/`)
- **NestJS 12** em ESM (padrão do `nest new`). Imports relativos terminam em `.js` (ex.: `import { X } from './x.js'`), que é uma regra do ESM no Node.
- **Prisma 7** com o adapter **`@prisma/adapter-pg`** + `pg`, generator `prisma-client` com `output = "../src/generated/prisma"` e configuração em `prisma.config.ts`. Seguir o guia oficial "Prisma 7 + NestJS".
- **PostgreSQL 18** (via Docker em desenvolvimento)
- **@nestjs/config** para ler o `.env`
- **class-validator / class-transformer** para validar DTOs
- **@nestjs/jwt + passport-jwt** para sessões (JWT em cookie httpOnly)
- **google-auth-library** para validar o login Google do cliente
- **@nestjs/throttler** para limitar tentativas de envio e validação de código
- **date-fns + date-fns-tz** para datas (fuso `America/Sao_Paulo`)
- **Nodemailer** para e-mail
- **WhatsApp Cloud API (Meta)** via `fetch` nativo, sem SDK
- **Vitest** (já vem com o Nest 12) para testes e **oxlint** para lint

### 3.3 Frontend (`web/`)
- **Next.js 16** (App Router, TypeScript, React 19)
- **Tailwind CSS 4**
- **shadcn/ui** (componentes, inclusive `Calendar`, `InputOTP` e `Chart`)
- **next-themes** para tema escuro (padrão) e claro
- **@react-oauth/google** para o botão "Entrar com Google"
- **react-hook-form + zod** para formulários
- **lucide-react** para ícones
- `fetch` nativo encapsulado em `web/src/lib/api.ts`

### 3.4 Infra
- `docker-compose.yml` na raiz (já incluído no pacote) com **PostgreSQL 18** e **Mailpit** (caixa de e-mail falsa para desenvolvimento, em http://localhost:8025)
- Monorepo simples: duas pastas (`api/` e `web/`), **sem** Turborepo/Nx

---

## 4. Regras de simplicidade (o projeto é de aprendizado)

O dono do projeto é desenvolvedor experiente em PHP/Laravel e Vue, mas este é o **primeiro contato com NestJS e Next.js**. Então:

1. **Código simples > código esperto.** Nada de abstrações genéricas, repositórios genéricos, factories, decorators customizados além do `@Roles` e do usuário logado, ou padrões que fujam do "caminho padrão" da documentação oficial.
2. **Um módulo Nest por assunto** (`services`, `professionals`, `appointments`...), cada um com `module`, `controller`, `service` e `dto/`. Só isso.
3. **Comentários em português** explicando *conceitos do framework* na primeira vez em que aparecem. Exemplo: "O `@Injectable()` permite que o Nest injete esta classe em outras, parecido com o service container do Laravel".
4. **Nomes de código em inglês**, textos da interface em português.
5. **Funções pequenas**, com nomes que dizem o que fazem.
6. Sempre que possível, fazer uma **ponte com Laravel/Vue** nos comentários e nas notas de aprendizado.
7. **Não adicionar bibliotecas** além das listadas sem justificar.
8. Ao terminar cada fase, gerar uma nota em `docs/aprendizado/` explicando o que foi construído e quais conceitos apareceram.

---

## 5. Perfis e autenticação

| Perfil | Quem é | Como entra | O que faz |
|---|---|---|---|
| **Visitante** | Qualquer pessoa | — | Vê o site e navega pelas etapas 1 a 4 do agendamento |
| **Cliente** | Quem agenda | **Login com Google** | Confirma agendamentos, vê e cancela os próprios, envia avaliações |
| **ADMIN** | Dono/gerente | **Código por e-mail ou WhatsApp** | Tudo no backoffice |
| **PROFESSIONAL** | Barbeiro | **Código por e-mail ou WhatsApp** | Próprio perfil, horários, folgas, agenda e números |

Um ADMIN também pode atender clientes. Para isso, ele tem um registro de `Professional` ligado ao seu usuário.

São **duas sessões separadas**, cada uma com seu cookie httpOnly e seu guard:
- `customer_token` → `CustomerAuthGuard` (rotas `/customer/*`), validade de 30 dias
- `staff_token` → `StaffAuthGuard` + `RolesGuard` (rotas `/admin/*` e `/me/*`), validade de 12 horas

No Next.js, o redirecionamento de quem não está logado fica em `web/src/proxy.ts` (o antigo `middleware.ts`), que só confere se o cookie existe. A validação de verdade é sempre na API.

Assim o cliente e o backoffice nunca se misturam, e o código de cada guard fica curto.

### 5.1 Login do cliente com Google
1. O frontend mostra o botão oficial "Entrar com Google" (`@react-oauth/google`, componente `GoogleLogin`). Ele abre um popup, sem redirecionar a página e sem perder a seleção do agendamento.
2. O Google devolve um **ID token** (`credential`) ao frontend.
3. O frontend envia `POST /api/auth/customer/google { credential }`.
4. O backend valida o token com `google-auth-library` (`verifyIdToken`, conferindo `audience = GOOGLE_CLIENT_ID` e `email_verified`).
5. O backend cria ou atualiza o `Customer` (por `googleId`) com nome, e-mail e foto, e grava o cookie `customer_token`.
6. Na primeira vez, o cliente informa o **WhatsApp**, que é obrigatório para agendar. Ele fica salvo no perfil e vem preenchido nas próximas vezes.

O agente deve criar `docs/google-login.md` com o passo a passo no Google Cloud Console: criar o OAuth Client do tipo "Web", cadastrar as origens JavaScript autorizadas (`http://localhost:3000` e o domínio de produção) e preencher `GOOGLE_CLIENT_ID` no `.env` da api e `NEXT_PUBLIC_GOOGLE_CLIENT_ID` no `.env` do web.

### 5.2 Login do backoffice por código (sem senha)
**Tela `/admin/login`:**
1. A pessoa digita **e-mail ou WhatsApp** e escolhe por onde quer receber o código (e-mail / WhatsApp).
2. `POST /api/auth/staff/request-code { identifier, channel }`
3. A tela muda para 6 campos de código (`InputOTP` do shadcn), com contador de 10 minutos e o botão "Reenviar código" liberado após 60 segundos.
4. `POST /api/auth/staff/verify-code { identifier, code }` → grava o cookie `staff_token` e redireciona para `/admin`.

**Regras de segurança (backend):**
- Código de **6 dígitos** gerado com `crypto.randomInt`. Guardar apenas o **hash** (sha256), nunca o código puro.
- Validade de **10 minutos**, uso único. Um novo pedido invalida os códigos anteriores do mesmo usuário.
- No máximo **5 tentativas** por código. Depois disso, o código é invalidado.
- No máximo **1 envio a cada 60 segundos** e **5 envios por hora** por usuário, além do `@nestjs/throttler` por IP nas duas rotas.
- `request-code` **sempre responde 200 com a mesma mensagem** ("Se o contato estiver cadastrado, você receberá um código"), para não revelar quem tem acesso.
- Usuário inativo não recebe código.
- Em desenvolvimento (`NODE_ENV=development`), o código também é impresso no log da API. O e-mail chega no Mailpit.
- Se o canal WhatsApp ainda não estiver configurado (`WHATSAPP_ENABLED=false`), a tela mostra só a opção e-mail.

---

## 6. Modelo de dados (Prisma, referência)

```prisma
enum Role { ADMIN PROFESSIONAL }
enum AppointmentStatus { CONFIRMED CANCELLED COMPLETED NO_SHOW }
enum ReviewStatus { PENDING APPROVED }
enum CodeChannel { EMAIL WHATSAPP }

model Barbershop {            // registro único com as configurações
  id                   Int     @id @default(1)
  name                 String
  slogan               String?
  about                String?
  address              String?
  city                 String?
  whatsapp             String?
  email                String?
  instagram            String?
  mapsEmbedUrl         String?
  logoUrl              String?
  heroImageUrl         String?
  slotIntervalMinutes  Int     @default(15)  // de quanto em quanto tempo os horários aparecem
  minAdvanceMinutes    Int     @default(60)  // antecedência mínima para agendar
  maxAdvanceDays       Int     @default(30)  // até quantos dias à frente pode agendar
  cancelLimitHours     Int     @default(2)   // cliente cancela até X horas antes
}

model BusinessHour {          // horário de funcionamento da barbearia (configurado no backoffice)
  id        String  @id @default(cuid())
  weekday   Int     @unique // 0 = domingo ... 6 = sábado
  isOpen    Boolean @default(true)
  openTime  String  // "09:00"
  closeTime String  // "20:00"
}

model ClosedDay {             // feriados e dias em que a barbearia não abre
  id     String   @id @default(cuid())
  date   DateTime @unique @db.Date
  reason String?
}

model User {                  // quem acessa o backoffice (sem senha)
  id           String   @id @default(cuid())
  name         String
  email        String   @unique
  phone        String?  @unique // E.164, usado para receber código por WhatsApp
  role         Role
  active       Boolean  @default(true)
  professional Professional?
  loginCodes   LoginCode[]
  createdAt    DateTime @default(now())
}

model LoginCode {
  id        String      @id @default(cuid())
  userId    String
  user      User        @relation(fields: [userId], references: [id])
  codeHash  String
  channel   CodeChannel
  attempts  Int         @default(0)
  expiresAt DateTime
  usedAt    DateTime?
  createdAt DateTime    @default(now())
  @@index([userId, createdAt])
}

model Customer {              // cliente que agenda (login Google)
  id           String   @id @default(cuid())
  googleId     String   @unique
  email        String   @unique
  name         String
  avatarUrl    String?
  phone        String?  // WhatsApp, obrigatório antes do primeiro agendamento
  createdAt    DateTime @default(now())
  appointments Appointment[]
  reviews      Review[]
}

model Professional {
  id               String   @id @default(cuid())
  userId           String?  @unique
  user             User?    @relation(fields: [userId], references: [id])
  name             String
  bio              String?
  photoUrl         String?
  phone            String?  // WhatsApp do profissional, E.164
  email            String?
  notifyByEmail    Boolean  @default(true)
  notifyByWhatsapp Boolean  @default(false)
  active           Boolean  @default(true)
  displayOrder     Int      @default(0)
  services         ProfessionalService[]
  workingHours     WorkingHour[]
  timeOffs         TimeOff[]
  appointments     Appointment[]
}

model Service {
  id              String  @id @default(cuid())
  name            String
  description     String?
  durationMinutes Int     // configurado no backoffice
  priceInCents    Int     // dinheiro sempre em centavos
  active          Boolean @default(true)
  displayOrder    Int     @default(0)
  professionals   ProfessionalService[]
  appointments    Appointment[]
}

model ProfessionalService {   // quais serviços cada profissional faz
  professionalId String
  serviceId      String
  professional   Professional @relation(fields: [professionalId], references: [id])
  service        Service      @relation(fields: [serviceId], references: [id])
  @@id([professionalId, serviceId])
}

model WorkingHour {           // várias faixas por dia permitem intervalo de almoço
  id             String @id @default(cuid())
  professionalId String
  professional   Professional @relation(fields: [professionalId], references: [id])
  weekday        Int    // 0 = domingo ... 6 = sábado
  startTime      String // "09:00"
  endTime        String // "12:00"
}

model TimeOff {               // folgas, férias, bloqueios pontuais do profissional
  id             String   @id @default(cuid())
  professionalId String
  professional   Professional @relation(fields: [professionalId], references: [id])
  startsAt       DateTime
  endsAt         DateTime
  reason         String?
}

model Appointment {
  id             String   @id @default(cuid())
  customerId     String
  professionalId String
  serviceId      String
  customer       Customer     @relation(fields: [customerId], references: [id])
  professional   Professional @relation(fields: [professionalId], references: [id])
  service        Service      @relation(fields: [serviceId], references: [id])
  customerName   String   // cópia no momento do agendamento
  customerPhone  String   // cópia no momento do agendamento
  notes          String?
  startsAt       DateTime
  endsAt         DateTime
  priceInCents   Int      // cópia do preço no momento do agendamento
  status         AppointmentStatus @default(CONFIRMED)
  createdBy      String   @default("CUSTOMER") // "CUSTOMER" ou "STAFF" (encaixe feito no balcão)
  createdAt      DateTime @default(now())
  review         Review?
  @@index([professionalId, startsAt])
  @@index([customerId, startsAt])
}

model GalleryImage {
  id           String  @id @default(cuid())
  url          String
  caption      String?
  isSample     Boolean @default(false) // imagens provisórias do seed
  displayOrder Int     @default(0)
}

model Review {
  id            String       @id @default(cuid())
  customerId    String?      // nulo nas avaliações de exemplo
  customer      Customer?    @relation(fields: [customerId], references: [id])
  appointmentId String?      @unique // uma avaliação por atendimento
  appointment   Appointment? @relation(fields: [appointmentId], references: [id])
  customerName  String
  rating        Int          // 1 a 5
  comment       String
  status        ReviewStatus @default(PENDING)
  isSample      Boolean      @default(false) // avaliações fictícias do seed
  createdAt     DateTime     @default(now())
  approvedAt    DateTime?
}
```

**Encaixe no balcão:** quando a equipe cria um agendamento pelo backoffice para alguém sem login Google, o sistema cria (ou reaproveita, pelo telefone) um `Customer` com `googleId = "manual:<telefone>"` e e-mail fictício `<telefone>@manual.local`. Isso mantém `customerId` sempre preenchido e o código simples.

**Datas:** gravar sempre em UTC no banco. Toda conversão para o horário da barbearia (`America/Sao_Paulo`) fica concentrada em **um único arquivo**: `api/src/common/time.ts`.

**Imagens:** na primeira versão, guardar uploads em `api/uploads/` e servir como estático. Trocar por S3/Cloudinary só no deploy. As imagens provisórias ficam em `web/public/placeholders/`.

---

## 7. Fluxo de agendamento (fluxo fechado, inspirado no Booksy)

### 7.1 Ordem das etapas

No Booksy o cliente escolhe **primeiro o serviço**, porque a duração do serviço define quais horários cabem na agenda. O visitante navega livremente e só precisa entrar com Google na hora de confirmar. Isso também é como o Booksy faz, e reduz desistências.

```
1. Serviço → 2. Profissional → 3. Dia → 4. Horário → 5. Entrar com Google + WhatsApp → 6. Revisão → 7. Confirmado
```

| Etapa | O que o cliente vê | Regras |
|---|---|---|
| 1. Serviço | Lista de serviços ativos com nome, duração e preço | Só serviços ativos e com pelo menos 1 profissional ativo |
| 2. Profissional | Profissionais que fazem aquele serviço + **"Sem preferência"** | Só profissionais ativos ligados ao serviço |
| 3. Dia | Calendário do mês | Dias sem horário livre aparecem desabilitados, assim como dias passados, dias além de `maxAdvanceDays`, dias fechados e `ClosedDay` |
| 4. Horário | Grade de horários livres agrupada em manhã / tarde / noite | Calculado pelo backend (7.2) |
| 5. Identificação | Se não estiver logado: botão "Entrar com Google". Depois: nome e foto da conta, campo WhatsApp (obrigatório, preenchido se já salvo) e observação (opcional) | Telefone validado e normalizado para E.164 e salvo no perfil |
| 6. Revisão | Resumo com serviço, profissional, data, hora, duração e preço | Botão "Confirmar agendamento" |
| 7. Confirmado | `/minha-conta/agendamentos/[id]` | Botões para adicionar à agenda (.ics), abrir no Maps, falar no WhatsApp da barbearia e cancelar |

- O cliente pode voltar a qualquer etapa pelo stepper. Ao trocar algo, as etapas seguintes são limpas.
- A seleção fica salva em `sessionStorage`, então recarregar a página ou fazer o login não perde nada.
- Se o cliente já estiver logado, a etapa 5 mostra só a confirmação dos dados.

**"Sem preferência":** o backend junta os horários livres de todos os profissionais aptos. Ao confirmar, escolhe o profissional livre com **menos agendamentos naquele dia** (empate → `displayOrder`).

### 7.2 Cálculo de disponibilidade (backend)

Arquivo: `api/src/availability/availability.service.ts`. Deve ser **a parte mais bem testada do sistema**.

Para um profissional, um serviço e uma data:

1. Se a data for `ClosedDay` ou o `BusinessHour` daquele dia estiver fechado → nenhum horário.
2. Pegar os `WorkingHour` do profissional para o dia da semana e **recortar** cada faixa pelo horário de funcionamento (o profissional nunca atende fora do horário da barbearia).
3. Para cada faixa, gerar inícios de `slotIntervalMinutes` em `slotIntervalMinutes`.
4. Descartar o horário se `início + duração do serviço` passar do fim da faixa.
5. Descartar se conflitar com um `Appointment` `CONFIRMED`. Há conflito quando os intervalos se sobrepõem: `inicioA < fimB && inicioB < fimA`.
6. Descartar se conflitar com um `TimeOff`.
7. Descartar se estiver antes de `agora + minAdvanceMinutes`.
8. Descartar se a data estiver além de `hoje + maxAdvanceDays`.

Endpoints derivados:
- **Dias disponíveis do mês**: roda o cálculo para cada dia e devolve os dias com pelo menos 1 horário.
- **Horários do dia**: devolve a lista de horários livres.

Validação no backoffice: ao salvar um `WorkingHour` fora do horário de funcionamento, responder 400 com mensagem clara. Ao reduzir o horário de funcionamento, avisar quantos agendamentos futuros ficam fora do novo horário (eles **não** são cancelados automaticamente).

### 7.3 Confirmação e proteção contra agendamento duplo

`POST /api/customer/appointments` (exige `customer_token`):

1. Validar o DTO. Exigir que o cliente tenha telefone salvo.
2. Verificar se serviço e profissional existem, estão ativos e se o profissional faz o serviço.
3. **Recalcular a disponibilidade** daquele horário, sem confiar no frontend.
4. Criar dentro de `prisma.$transaction` com isolamento `Serializable`, checando conflito de novo antes de inserir.
5. Se o horário foi ocupado nesse meio tempo → **409 Conflict**. O frontend volta para a etapa de horários, recarrega a lista e mostra: "Esse horário acabou de ser reservado. Escolha outro."
6. Limite anti-abuso: no máximo **3 agendamentos futuros ativos por cliente**. Acima disso → 400 com o WhatsApp da barbearia.
7. Depois de salvar, disparar as notificações (seção 8). **Falha na notificação nunca desfaz o agendamento**, só registra o erro no log.

### 7.4 Área do cliente
- `/minha-conta/agendamentos`: próximos e anteriores, com status.
- Cancelamento permitido até `cancelLimitHours` antes. Depois disso, mostrar o botão do WhatsApp da barbearia.
- Status vira `CANCELLED`, o horário volta a ficar livre e o profissional é avisado.
- Em agendamentos `COMPLETED` sem avaliação: botão **"Avaliar atendimento"**.
- `/minha-conta`: nome, foto e e-mail (do Google, somente leitura), WhatsApp (editável) e sair.

### 7.5 Casos de borda que precisam funcionar
- Serviço que termina exatamente no fim do expediente → permitido.
- Serviço que atravessaria o intervalo de almoço → bloqueado.
- Faixa do profissional maior que o horário da barbearia → recortada.
- Feriado cadastrado → dia desabilitado no calendário.
- Profissional desativado com agendamentos futuros → agendamentos continuam, mas ele some do site.
- Serviço desativado → some do site; agendamentos já feitos continuam.
- Dois clientes confirmando o mesmo horário ao mesmo tempo → só um consegue (409 para o outro).
- Sessão Google expirada na etapa 6 → pedir login de novo sem perder a seleção.
- Fuso: tudo calculado em `America/Sao_Paulo`.
- Data passada, data além do limite e horário dentro da antecedência mínima → recusados no backend, mesmo que o frontend deixe passar.

---

## 8. Notificações

`NotificationsModule` com um `NotificationsService` simples, com **um método por evento**:
- `sendLoginCode(user, code, channel)`
- `appointmentCreated(appointment)` → profissional (pelos canais que ele ativou) + cliente (e-mail do Google)
- `appointmentCancelled(appointment)` → profissional + cliente
- `reviewSubmitted(review)` → e-mail para os ADMINs ("Nova avaliação aguardando aprovação")

Canais (cada um em um arquivo pequeno: `email.sender.ts` e `whatsapp.sender.ts`):

1. **E-mail (desde a Fase 1, porque o login por código depende dele):** Nodemailer + SMTP via `.env`. Em desenvolvimento, aponta para o Mailpit. Templates HTML simples com o logo e as cores da marca.
2. **WhatsApp (Fase 5):** WhatsApp Cloud API da Meta, chamada com `fetch`, ativada por `WHATSAPP_ENABLED=true`. Exige:
   - conta WhatsApp Business e número verificado;
   - template de categoria **Autenticação** para o código de login;
   - template de categoria **Utilidade** para avisar o profissional de agendamento/cancelamento.

   O agente deve documentar tudo em `docs/whatsapp.md`. Enquanto não estiver configurado, o login usa só e-mail, e a página de confirmação do cliente mostra o botão "Falar no WhatsApp" (`https://wa.me/5516996058945?text=<resumo>`).

Mensagem mínima ao profissional: nome e WhatsApp do cliente, serviço, data, hora, duração e observação.

---

## 9. Avaliações

- **Seed:** 6 avaliações fictícias (`isSample = true`, `status = APPROVED`, notas entre 4 e 5, textos realistas e variados, nomes fictícios). Aparecem no site até a barbearia ter avaliações reais.
- **Quem pode avaliar:** cliente logado com Google que tenha **pelo menos um atendimento `COMPLETED`**. Uma avaliação por atendimento. Isso evita avaliações falsas sem complicar.
- **Onde avaliar:**
  - na área do cliente, botão "Avaliar atendimento" nos atendimentos concluídos;
  - na home, botão "Deixar avaliação" na seção de avaliações. Se não estiver logado, pede login Google. Se não tiver atendimento concluído, mostra "Você poderá avaliar após seu primeiro atendimento."
- **Formulário:** nota de 1 a 5 estrelas (obrigatória) e comentário (obrigatório, 10 a 500 caracteres). O nome vem da conta Google, exibido como "Nome S." (primeiro nome + inicial).
- **Moderação:** toda avaliação nova entra como `PENDING` e **só aparece no site depois de aprovada**. O cliente vê a mensagem "Obrigado! Sua avaliação será publicada após aprovação."
- **Backoffice (`/admin/avaliacoes`):** abas Pendentes / Aprovadas, com ações **Aprovar** e **Apagar** (com confirmação). Na aba de aprovadas, botão **"Remover avaliações de exemplo"** apaga todas com `isSample = true`. Contador de pendentes na sidebar e card no painel.
- **Site:** a média e o total consideram só as avaliações aprovadas.

---

## 10. Site público (`web/`)

### 10.1 Rotas
| Rota | Conteúdo | Acesso |
|---|---|---|
| `/` | Página inicial | público |
| `/agendar` | Fluxo de agendamento (stepper) | público até a etapa 4; login Google a partir da 5 |
| `/minha-conta` | Perfil do cliente | cliente |
| `/minha-conta/agendamentos` | Lista de agendamentos | cliente |
| `/minha-conta/agendamentos/[id]` | Detalhe / confirmação / cancelar / avaliar | cliente (só o dono) |
| `/privacidade` | Política de privacidade curta (LGPD): quais dados do Google e o telefone são usados, para quê, e como pedir exclusão | público |

### 10.2 Seções da página inicial (nesta ordem)
1. **Header fixo**: logo, links âncora (Serviços, Equipe, Galeria, Avaliações, Contato), alternador de tema, avatar do cliente quando logado (menu com "Meus agendamentos" e "Sair") e botão "Agendar". No celular, vira menu hambúrguer com drawer.
2. **Hero**: imagem de fundo escurecida (placeholder), logo grande, slogan, CTAs "Agendar horário" e "Ver serviços".
3. **Sobre**: texto curto e 3 diferenciais com ícones.
4. **Serviços**: cards com nome, descrição, duração, preço e botão que leva a `/agendar?serviceId=` (pula a etapa 1).
5. **Equipe**: cards com foto, nome, bio curta e "Agendar com ele" (`/agendar?professionalId=`).
6. **Galeria**: grid com lightbox (imagens provisórias até a troca).
7. **Avaliações**: média em estrelas, total, carrossel de depoimentos aprovados e botão "Deixar avaliação".
8. **Localização e horários**: mapa (iframe do Google Maps), endereço, **horário de funcionamento vindo do backoffice** (com destaque "Aberto agora" / "Fechado agora"), WhatsApp e Instagram.
9. **CTA final**: faixa "Pronto para o próximo corte?" com botão.
10. **Footer**: logo, redes, contato, link para `/privacidade` e link discreto "Área do profissional" (`/admin/login`).
11. **Botão flutuante de WhatsApp** no canto inferior direito (no celular, fica acima da barra de ação do stepper quando estiver em `/agendar`).

Páginas públicas são **Server Components** (bom para SEO). Só o que tem interação vira Client Component. Incluir metadados (title, description, Open Graph com o logo) e JSON-LD `BarberShop` com endereço, telefone e horários.

### 10.3 Identidade visual: "barbearia futurista" a partir do logo

**O logo é monocromático (preto e branco)**, com um "M" geométrico de traço grosso e um corte diagonal. A identidade parte disso:

- **Arquivos** (copiar de `assets/brand/` para `web/public/brand/` na Fase 0):
  - `logo-branco.png`: logo completo branco com fundo transparente, para o **tema escuro**
  - `logo-preto.png`: logo completo preto com fundo transparente, para o **tema claro**
  - `icone-original.png`: ícone "M" no círculo, usado em avatar, favicon e e-mails
  - `favicon-32.png` e `apple-icon.png`: ícones do site
  - Trocar o logo conforme o tema com duas `<Image>` e as classes `dark:block hidden` / `dark:hidden block`
  - Pedir à barbearia o logo em **SVG** quando possível, para melhor nitidez
- **Paleta monocromática com toque metálico.** Todas as cores viram tokens CSS do shadcn:
  - Escuro (padrão): fundo `#0A0A0A`, superfície `#141414`, borda `#262626`, texto `#FAFAFA`, texto secundário `#A3A3A3`, **primário branco** `#FAFAFA` (botões brancos com texto preto)
  - Claro: fundo `#FAFAFA`, superfície `#FFFFFF`, borda `#E5E5E5`, texto `#0A0A0A`, **primário preto** `#0A0A0A`
  - Acento metálico (prata/cromo) só em detalhes: gradiente `#E5E5E5 → #737373` em linhas, bordas em destaque e no brilho do hover
  - Cores de estado discretas: sucesso `#22C55E`, erro `#EF4444`, aviso `#F59E0B`
- **Tipografia** via `next/font`: títulos em **Montserrat** (700–800, levemente espaçada, conversa com o "MAESTRIA" do logo); texto em **Inter**; rótulos pequenos "tech" opcionais em **Michroma**.
- **Motivo gráfico:** a **diagonal do "M"** se repete em divisores de seção, cantos cortados de cards e destaques do hover. Além disso: grid de linhas finas ao fundo, cards com borda de 1px e brilho prateado suave, header com vidro fosco e ícones de traço fino.
- **Movimento:** só transições CSS e entradas leves, respeitando `prefers-reduced-motion`.
- **Acessibilidade:** contraste AA nos dois temas, foco visível e labels em todos os campos.

---

## 11. Responsividade (obrigatória em todo o sistema)

**Mobile first**: o layout é pensado primeiro para o celular e depois expandido. A maioria dos clientes vai agendar pelo celular, e os barbeiros vão consultar a agenda entre um atendimento e outro.

| Breakpoint | Largura | Referência |
|---|---|---|
| base | 360–639px | celular (testar em **375px**) |
| `sm` | ≥ 640px | celular grande |
| `md` | ≥ 768px | tablet |
| `lg` | ≥ 1024px | notebook |
| `xl` | ≥ 1280px | desktop |

Regras gerais:
- **Nenhuma rolagem horizontal** em nenhuma tela, em nenhuma largura.
- Alvos de toque com no mínimo **44×44px**. Campos com fonte de pelo menos 16px, para o iPhone não dar zoom.
- Imagens com `next/image` e `sizes` corretos. Textos longos quebram, sem cortar.
- Considerar a área segura do iPhone (`env(safe-area-inset-bottom)`) nas barras fixas.

Por tela:
- **Site:** menu hambúrguer abaixo de `md`; grids de 1 → 2 → 3/4 colunas; carrossel de avaliações com swipe.
- **Agendamento:** no celular, uma etapa por tela, stepper compacto no topo ("Etapa 3 de 6") e **botão principal fixo no rodapé**; calendário ocupando a largura toda; horários em grade de 3 colunas. No desktop, as etapas ficam à esquerda e um **resumo fixo** ("Seu agendamento") à direita.
- **Backoffice:** sidebar fixa a partir de `lg`. Abaixo disso, **barra de navegação inferior** com os 4 itens principais (Painel, Agenda, Avaliações, Mais) e drawer para o resto. **Tabelas viram cards** abaixo de `md`. A **agenda** mostra 1 dia e 1 profissional por vez no celular (com seletor) e colunas por profissional no desktop. Formulários em 1 coluna no celular e 2 no desktop. Gráficos do painel com largura fluida.

---

## 12. Backoffice (`web/src/app/admin`)

Layout com navegação responsiva (seção 11), header com nome do usuário, alternador de tema e sair. **Tema escuro padrão**, claro disponível.

| Rota | Perfil | Conteúdo |
|---|---|---|
| `/admin/login` | — | Login por código (seção 5.2), com o logo centralizado |
| `/admin` | ambos | Painel (abaixo) |
| `/admin/agenda` | ambos | Visão de dia/semana por profissional (ADMIN vê todos, profissional vê só a sua). Ao clicar, abre os detalhes, com mudança de status (concluído, não compareceu, cancelado) e botão de WhatsApp do cliente. Botão "Novo agendamento" para encaixe |
| `/admin/funcionamento` | ADMIN | **Horário de funcionamento** por dia da semana (aberto/fechado, abre às, fecha às) e **feriados/dias fechados** |
| `/admin/servicos` | ADMIN | Tabela e formulário: nome, descrição, **duração (minutos)**, preço, ativo, ordem |
| `/admin/profissionais` | ADMIN | Tabela e formulário: dados, foto, serviços que faz, e-mail e WhatsApp de acesso, papel (admin/profissional), ativo |
| `/admin/galeria` | ADMIN | Upload, legenda, ordenar, excluir e "substituir imagens provisórias" |
| `/admin/avaliacoes` | ADMIN | Moderação: abas Pendentes/Aprovadas, aprovar, apagar, remover exemplos (seção 9) |
| `/admin/clientes` | ADMIN | Lista simples de clientes (nome, e-mail, WhatsApp, nº de atendimentos, último atendimento) com busca |
| `/admin/configuracoes` | ADMIN | Dados da barbearia e regras de agendamento (intervalo, antecedência, limite de cancelamento, dias à frente) |
| `/admin/meu-perfil` | ambos | Foto, bio, WhatsApp, e-mail e preferências de notificação |
| `/admin/meus-horarios` | profissional | Editor semanal: para cada dia, ligar/desligar e adicionar faixas (ex.: 09:00–12:00 e 13:00–19:00), limitado ao horário de funcionamento |
| `/admin/minhas-folgas` | profissional | Lista e criação de bloqueio (dia inteiro ou intervalo) |

### Painel (`/admin`)
Filtro de período (hoje, 7 dias, 30 dias, mês atual). Cards e gráficos:
- Atendimentos concluídos no período e agendamentos futuros
- Faturamento estimado (soma de `priceInCents` dos concluídos)
- Taxa de cancelamento e de não comparecimento
- Novos clientes no período
- Nota média das avaliações e **avaliações pendentes** (com link para moderação)
- Gráfico de atendimentos por dia
- Serviços mais vendidos
- Atendimentos por profissional (só ADMIN)
- Horários de pico (por hora do dia)
- Lista "Próximos agendamentos de hoje"

O profissional vê os mesmos números filtrados para ele (sem faturamento geral, clientes e avaliações pendentes).

---

## 13. API (referência)

Prefixo global: `/api`. Respostas em JSON. Erros no padrão do Nest (`{ statusCode, message }`) com mensagens em português.

### Público (sem login)
| Método | Rota | Descrição |
|---|---|---|
| GET | `/public/barbershop` | Dados da barbearia + horário de funcionamento |
| GET | `/public/services` | Serviços ativos |
| GET | `/public/professionals?serviceId=` | Profissionais ativos (filtra por serviço) |
| GET | `/public/availability/days?serviceId=&professionalId=&month=2026-11` | Dias com horário livre (`professionalId=any` para "Sem preferência") |
| GET | `/public/availability/slots?serviceId=&professionalId=&date=2026-11-03` | Horários livres do dia |
| GET | `/public/gallery` | Galeria |
| GET | `/public/reviews` | Avaliações aprovadas + média + total |

### Autenticação
| Método | Rota | Descrição |
|---|---|---|
| POST | `/auth/customer/google` | `{ credential }` → cookie `customer_token` |
| POST | `/auth/customer/logout` | Limpa o cookie |
| POST | `/auth/staff/request-code` | `{ identifier, channel }` → envia código (resposta sempre genérica) |
| POST | `/auth/staff/verify-code` | `{ identifier, code }` → cookie `staff_token` |
| POST | `/auth/staff/logout` | Limpa o cookie |

### Cliente (`customer_token`)
| Método | Rota | Descrição |
|---|---|---|
| GET / PATCH | `/customer/me` | Perfil (PATCH só altera o telefone) |
| GET | `/customer/appointments` | Meus agendamentos |
| POST | `/customer/appointments` | Criar agendamento |
| GET | `/customer/appointments/:id` | Detalhe (só o dono) |
| POST | `/customer/appointments/:id/cancel` | Cancelar |
| GET | `/customer/appointments/:id/ics` | Arquivo de calendário |
| GET | `/customer/review-eligibility` | Se pode avaliar e quais atendimentos estão pendentes de avaliação |
| POST | `/customer/reviews` | `{ appointmentId, rating, comment }` → cria como `PENDING` |

### Backoffice: ADMIN (`staff_token` + `@Roles('ADMIN')`)
CRUD em `/admin/services`, `/admin/professionals` (criar profissional cria também o `User`), `/admin/gallery`, `/admin/closed-days`;
`GET/PUT /admin/business-hours` (salva a semana inteira de uma vez);
`GET/PUT /admin/barbershop`;
`GET /admin/appointments?date=&professionalId=&status=`, `PATCH /admin/appointments/:id/status`, `POST /admin/appointments` (encaixe);
`GET /admin/reviews?status=`, `PATCH /admin/reviews/:id/approve`, `DELETE /admin/reviews/:id`, `DELETE /admin/reviews/samples`;
`GET /admin/customers?search=`;
`GET /admin/dashboard?from=&to=`.

### Backoffice: profissional (`staff_token`, também acessível ao ADMIN que é profissional)
`GET /me`, `GET/PUT /me/profile`, `GET/PUT /me/working-hours` (salva a semana inteira), `GET/POST/DELETE /me/time-off`, `GET /me/appointments?from=&to=`, `PATCH /me/appointments/:id/status`, `GET /me/dashboard`.

Um profissional nunca acessa dados de outro. As rotas `/me/*` filtram sempre pelo usuário do token, nunca por um id enviado pelo cliente.

---

## 14. Variáveis de ambiente

`api/.env.example`:
```
DATABASE_URL=postgresql://maestria:maestria@localhost:5432/maestria
PORT=3333
WEB_URL=http://localhost:3000
JWT_SECRET=troque-isto
GOOGLE_CLIENT_ID=
SMTP_HOST=localhost
SMTP_PORT=1025
SMTP_USER=
SMTP_PASS=
MAIL_FROM="Maestria Barbearia <nao-responda@maestriabarbearia.com.br>"
WHATSAPP_ENABLED=false
WHATSAPP_TOKEN=
WHATSAPP_PHONE_NUMBER_ID=
WHATSAPP_TEMPLATE_LOGIN_CODE=
WHATSAPP_TEMPLATE_APPOINTMENT=
ADMIN_NAME=Administrador
ADMIN_EMAIL=admin@maestriabarbearia.com.br
ADMIN_PHONE=+5516996058945
```

`web/.env.example`:
```
NEXT_PUBLIC_API_URL=http://localhost:3333/api
NEXT_PUBLIC_GOOGLE_CLIENT_ID=
```

---

## 15. Fases de implementação

Cada fase termina com código funcionando, testes da fase passando, a nota `docs/aprendizado/fase-N.md` e um resumo do que testar manualmente.

| Fase | Entrega | Agentes principais |
|---|---|---|
| 0 | Estrutura: `docker-compose` (Postgres + Mailpit), projeto Nest, projeto Next, Prisma conectado, `.env.example`, assets da marca e placeholders copiados para `web/public/`, README de como rodar | tech-lead, backend-nest, frontend-next |
| 1 | Backend base: schema completo, seed (dados da seção 2), envio de e-mail, **login do backoffice por código**, guards, CRUD de funcionamento, feriados, serviços, profissionais, horários e folgas | backend-nest, qa-tester, mentor |
| 2 | Backend do cliente: **login Google**, disponibilidade, criação e cancelamento de agendamento com proteção contra duplo, **avaliações** (criar, elegibilidade, moderar) e testes do cálculo | backend-nest, qa-tester, mentor |
| 3 | Frontend do site: tema, layout responsivo, home completa, login Google, fluxo `/agendar`, área `/minha-conta`, enviar avaliação, `/privacidade` | ui-designer, frontend-next, qa-tester |
| 4 | Frontend do backoffice: login por código, navegação responsiva, agenda, funcionamento, cadastros, horários, folgas, galeria, **moderação de avaliações**, clientes, configurações | frontend-next, ui-designer, qa-tester |
| 5 | Notificações: e-mails de agendamento/cancelamento/avaliação, **WhatsApp Cloud API** (código de login + aviso ao profissional), arquivo .ics, `docs/whatsapp.md` | backend-nest, qa-tester |
| 6 | Painel com métricas e gráficos | backend-nest, frontend-next |
| 7 | Revisão final: responsividade em todas as telas, acessibilidade, SEO e guia de deploy (ex.: web na Vercel, api + banco em Railway/Render/Neon, SMTP real) | tech-lead, qa-tester, ui-designer, mentor |

### Melhorias futuras (não fazer agora)
Lembrete 24h antes por WhatsApp/e-mail, reagendamento pelo cliente, lista de espera, sinal via Pix, relatório de comissão por profissional, PWA (instalar o site no celular).

---

## 16. Critérios de aceite do fluxo fechado

- [ ] Cliente consegue agendar do zero no celular, incluindo o login Google, em menos de 1 minuto.
- [ ] O visitante navega até o horário sem login, e o login não perde a seleção.
- [ ] O calendário nunca mostra como disponível um dia sem horário livre, fechado ou feriado.
- [ ] Nenhum horário oferecido conflita com agendamento, folga, intervalo, fim de expediente ou horário de funcionamento.
- [ ] Dois agendamentos simultâneos para o mesmo horário → apenas um é criado.
- [ ] O profissional recebe a notificação, e uma falha de envio não perde o agendamento.
- [ ] O cliente vê e cancela os próprios agendamentos dentro do prazo; fora do prazo, vê o WhatsApp da barbearia.
- [ ] O cancelamento libera o horário imediatamente.
- [ ] Avaliação só pode ser enviada por cliente com atendimento concluído, e só aparece no site depois de aprovada.
- [ ] O admin aprova, apaga e remove as avaliações de exemplo.
- [ ] O login do backoffice funciona por código de e-mail (e de WhatsApp, quando ativado): o código expira em 10 minutos, bloqueia após 5 tentativas e não revela quem está cadastrado.
- [ ] O profissional não vê nem altera dados de outro profissional.
- [ ] Alterar o horário de funcionamento ou a duração de um serviço no backoffice muda imediatamente os horários oferecidos no site.
- [ ] Os temas escuro e claro funcionam em todas as telas sem "piscar", com o logo certo em cada um.
- [ ] Todas as telas funcionam em 375px, 768px e 1280px, sem rolagem horizontal.
