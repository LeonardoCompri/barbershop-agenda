# Regras do projeto para o frontend-next

## O que ler na especificação
Seções 3 (stack e versões), 5 (autenticação), 7 (fluxo de agendamento), 9 (avaliações), 10 (site público), 11 (responsividade) e 12 (backoffice).

## Bibliotecas extras deste projeto
Além da stack base do agente: `@react-oauth/google`, e os componentes shadcn `Calendar`, `InputOTP` e `Chart`.

## Rotas
- `src/app/(site)/`: `/`, `/agendar`, `/privacidade` e `/minha-conta/**` (cliente, cookie `customer_token`).
- `src/app/admin/`: backoffice (cookie `staff_token`), com `/admin/login` público.
- `src/proxy.ts`: redireciona `/minha-conta/**` para o login Google e `/admin/**` para `/admin/login` quando o cookie não existe.

## Fluxo de agendamento (seção 7.1)
- Etapas: Serviço → Profissional → Dia → Horário → Identificação (Google + WhatsApp) → Revisão → Confirmado.
- **O visitante navega até o horário sem login.** O login Google só é pedido na etapa 5.
- Estado: um único objeto `{ serviceId, professionalId, date, time, notes }` em `useState`, espelhado em `sessionStorage`. O login e o recarregamento não podem perder a seleção.
- Trocar uma etapa anterior limpa as seguintes.
- `?serviceId=` e `?professionalId=` na URL pré-selecionam e pulam etapas.
- Calendário: dias sem horário vêm desabilitados pela API (`/public/availability/days`).
- **409 ao confirmar:** volte para a etapa de horários, recarregue a lista e mostre "Esse horário acabou de ser reservado. Escolha outro."
- No celular: uma etapa por tela, stepper compacto ("Etapa 3 de 6") e botão principal fixo no rodapé. No desktop: etapas à esquerda e resumo fixo à direita.

## Login Google do cliente
`GoogleOAuthProvider` no layout do site e o componente `GoogleLogin` (popup, sem redirecionar). Envie o `credential` para `POST /auth/customer/google`. Na primeira vez, peça o WhatsApp.

## Login do backoffice
Tela em 2 passos: (1) e-mail ou WhatsApp + canal; (2) `InputOTP` de 6 dígitos, com contador de 10 min e "Reenviar" liberado após 60 s. Esconda a opção WhatsApp quando a API disser que ela está desativada. Mostre sempre a mensagem genérica de envio.

## Avaliações
- Home: média, total, carrossel das aprovadas e botão "Deixar avaliação" (pede login; sem atendimento concluído, mostra "Você poderá avaliar após seu primeiro atendimento").
- `/minha-conta/agendamentos`: botão "Avaliar atendimento" nos concluídos sem avaliação.
- Depois de enviar: "Obrigado! Sua avaliação será publicada após aprovação."
- Backoffice `/admin/avaliacoes`: abas Pendentes/Aprovadas, aprovar, apagar (com confirmação), "Remover avaliações de exemplo" e contador de pendentes na navegação.

## Outros
- Dados da barbearia (nome, endereço, horários, WhatsApp) **sempre vêm da API**, nunca fixos no código.
- Fuso `America/Sao_Paulo` em todas as datas exibidas.
- JSON-LD `BarberShop` na home, com endereço, telefone e horários vindos da API.
- Botão flutuante de WhatsApp (`https://wa.me/5516996058945`) no site.
